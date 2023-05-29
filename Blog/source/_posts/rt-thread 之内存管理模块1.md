---
title: rt-thread 内存管理模块之小内存管理算法
index_img: /img/rust.png
date: 2023-05-29 20:51:18
tags: rt-thread 
---

# 内存管理
在 rt-thread 操作系统中存在这几大模块，分别为：线程调度模块、线程间通信模块、线程间同步模块、内存管理模块、文件系统模块以及网络接口模块。
本次我们来了解一下 rt-thread 操作系统中对于内存是如何进行管理的。

## 简介
RT-Thread 是一个实时操作系统，为了适应嵌入式设备的资源限制，它提供了多个小内存管理算法。这些算法旨在高效地管理小块内存，并尽可能减少内存碎片的产生。

## 内存堆
内存堆（Memory Heap）是计算机内存中一块动态分配的内存区域，用于存储程序运行时动态分配的数据。在大多数编程语言中，包括C、C++、Java等，内存堆用于存储动态分配的对象和数据结构，这些对象在程序运行时动态创建和销毁。  

内存堆的主要特点是动态分配和释放内存空间，不同于静态分配的内存（如全局变量和局部变量）。程序可以使用堆上的内存来存储灵活大小的数据，根据需要进行动态的内存分配和释放。这使得程序能够在运行时根据需求来管理内存，并避免静态内存分配的限制。

在使用内存堆时，程序员通常使用特定的堆分配函数（如malloc、new等）来请求所需大小的内存块。当不再需要这些内存块时，应使用相应的堆释放函数（如free、delete等）将其释放回堆，以避免内存泄漏。
需要注意的是，对内存堆的不当使用可能导致内存泄漏、内存碎片等问题，因此在使用堆分配的内存时应谨慎管理和释放内存。

综上所述：内存堆是一块动态分配的内存区域，用于存储程序运行时动态分配的数据。程序可以使用堆上的内存来存储灵活大小的数据，并通过堆分配函数和堆释放函数来管理内存的分配和释放。
在 RT-Thread 操作系统中，动态内存堆的管理根据使用场景分为三种：
- small memory 管理算法：适用于小内存块的管理算法
- slab memroy 管理算法：适用于大内存块的管理算法
- memheap memory 管理算法：适用于多内存块的管理算法  

本次先来了解一下 small memory 管理算法。通过对整体源码的解读，由浅入深，加深对于 rt-thread 操作系统中内存管理模块的理解。

## small memory
小内存管理算法是一个简单的内存分配算法。初始时，它是一块大的内存。当需要分配内存块时，将从这个大的内存块上分割出相匹配的内存块，然后把分割出来的空闲内存块还回给堆管理系统中。每个内存块都包含一个管理用的数据头，通过这个头把使用块与空闲块用双向链表的方式链接起来。注意：这里的链表存放的并不是数据头绝对地址，而是相对于开始地址的偏移量。在下面的源码解析中会进行详细的说明。

### 数据结构

在小内存管理算法中，主要使用了下面的数据结构：

#### 小内存模块的内存头

在小内存管理算法中，每一块内存块都会有一个内存头，内存头的数据结构如下：

![](https://files.mdnice.com/user/44140/cd02a0f3-7508-43f4-834d-d855c9324a39.png)

参数解析：  

| 参数    | 描述    |
| --- | --- |
|  pool_ptr   | small memory object addr    |
| next    | next free item    |
| prev    | prev free item    |

#### 内存结构

rt-thread 操作系统内存管理的基本数据结构：

![](https://files.mdnice.com/user/44140/20ffb762-873f-45c2-99d5-ec36d963aba1.png)

参数解析：

| 参数    | 描述    |
| --- | --- |
| parent    | inherit from rt_object    |
|algorithm      |memory management algorithm name     |
|address     |memory start address     |
|total     |memory size    |
|used     |size used     |
|max    |maximum usage     |


#### 小内存模块对象的基本结构

在小内存管理算法中，存在的唯一一个模块对象：

![](https://files.mdnice.com/user/44140/687d0768-7c2e-43c2-ab27-c65a651a6d8f.png)

参数解析：

| 参数    | 描述    |
| --- | --- |
|parent     | inherit from rt_memory     |
|heap_ptr     |pointer to the heap     |
|heap_end     |pointer to the heap end address     |
|lfree     |pointer to free memory list     |
|mem_size_aligned     |aligned memory size     |

## API

### 字节对齐

在观看源码的之前，先来了解两个字节对齐的宏定义：
```c
#define RT_ALIGN(size, align)           (((size) + (align) - 1) & ~((align) - 1))
#define RT_ALIGN_DOWN(size, align)      ((size) & ~((align) - 1))
```

1. RT_ALIGN：在 size 的基础上向上对齐，对齐后的值为 align 的倍数。RT_ALIGN(13, 4)，对齐后的值为：16
2. RT_ALIGN_DOWN： 在 size 的基础上向下对齐，对齐后的值为 align 的倍数。RT_ALIGN_DOWN(13, 4)， 对齐后的值为：12

为什么要在嵌入式系统中进行字节对齐操作，以下是几个主要原因：

1. 访问效率：许多嵌入式系统使用总线来访问内存，而总线通常是按照字节为单位进行传输的。如果数据结构没有进行字节对齐，例如一个2字节的数据结构放置在奇数地址上，那么在访问这个数据结构时就需要进行两次内存访问操作。而如果数据结构进行了字节对齐，将其放置在合适的地址上，可以减少内存访问次数，提高数据的读取效率。
2. CPU对齐要求：许多CPU架构对于某些数据类型有对齐要求。如果数据没有按照要求对齐，可能会导致性能下降或者引发硬件异常。通过字节对齐操作，确保数据结构满足CPU的对齐要求，可以避免潜在的问题。
3. 数据可靠性：在多字节数据类型（如整型、浮点型等）的读写操作中，如果数据没有按照对齐要求存放，可能会引发未定义的行为。例如，某些CPU要求访问4字节整型数据时，其地址必须是4的倍数。如果数据没有进行对齐，读写操作可能会跨越多个字节的边界，导致数据错误或者异常。
4. 数据结构互操作性：在嵌入式系统中，可能会涉及到不同硬件模块或通信协议之间的数据交互。这些硬件或协议往往有特定的数据对齐要求。通过对数据进行字节对齐操作，可以确保数据在不同模块之间的正确传输和解析，提高数据的互操作性。

### plug_holes

合并相邻的内存块：

```c
static void plug_holes(struct rt_small_mem *m, struct rt_small_mem_item *mem)
{
    struct rt_small_mem_item *nmem;
    struct rt_small_mem_item *pmem;

    RT_ASSERT((rt_uint8_t *)mem >= m->heap_ptr);
    RT_ASSERT((rt_uint8_t *)mem < (rt_uint8_t *)m->heap_end);

    /* 判断 mem 的下一块内存块是否可以合并 */
    nmem = (struct rt_small_mem_item *)&m->heap_ptr[mem->next];
    if (mem != nmem && !MEM_ISUSED(nmem) &&
        (rt_uint8_t *)nmem != (rt_uint8_t *)m->heap_end)
    {
        /* 如果 mem->next 没有被使用，合并 mem 和 mem->next */
        if (m->lfree == nmem)
        {
            m->lfree = mem;
        }
        nmem->pool_ptr = 0;
        mem->next = nmem->next;
        ((struct rt_small_mem_item *)&m->heap_ptr[nmem->next])->prev = (rt_uint8_t *)mem - m->heap_ptr;
    }

    /* 判断 mem 的上一块内存块是否可以合并 */
    pmem = (struct rt_small_mem_item *)&m->heap_ptr[mem->prev];
    if (pmem != mem && !MEM_ISUSED(pmem))
    {
        /* 如果 mem->prev 没有被使用，合并 mem 和 mem->prev */
        if (m->lfree == mem)
        {
            m->lfree = pmem;
        }
        mem->pool_ptr = 0;
        pmem->next = mem->next;
        ((struct rt_small_mem_item *)&m->heap_ptr[mem->next])->prev = (rt_uint8_t *)pmem - m->heap_ptr;
    }
}
```

该函数的作用是将相邻空闲内存块进行合并；有以下两种情况：

1. mem 的下一块内存块为空闲内存块，并且不是内存堆结束数据头；如果此时 lfree 指向了 nmem，需要修改 lfree 指向 mem；可以用下面的图来描述：

![](https://files.mdnice.com/user/44140/0ff9e7e2-590f-44c5-850c-c0ac1d553107.png)


2. mem 的上一块内存块为空闲内存块；如果此时 lfree 指向了 mem，需要修改 lfree 的指向；该合并内存块操作可以用如下如图片来描述：

![](https://files.mdnice.com/user/44140/59498024-1652-4caf-b542-fc9fb4634287.png)

### rt_smem_init

小内存管理算法初始化：

```c
rt_smem_t rt_smem_init(const char    *name,
                     void          *begin_addr,
                     rt_size_t      size)
{
    struct rt_small_mem_item *mem;
    struct rt_small_mem *small_mem;
    rt_ubase_t start_addr, begin_align, end_align, mem_size;

    small_mem = (struct rt_small_mem *)RT_ALIGN((rt_ubase_t)begin_addr, RT_ALIGN_SIZE);
    start_addr = (rt_ubase_t)small_mem + sizeof(*small_mem);
    /* 内存堆起始地址进行字节对齐 */
    begin_align = RT_ALIGN((rt_ubase_t)start_addr, RT_ALIGN_SIZE);
    /* 内存堆结束地址进行字节对齐 */
    end_align   = RT_ALIGN_DOWN((rt_ubase_t)begin_addr + size, RT_ALIGN_SIZE);

    /* 计算对齐之后的内存堆大小， 减去头尾 2 个数据头所占空间 */
    if ((end_align > (2 * SIZEOF_STRUCT_MEM)) &&
        ((end_align - 2 * SIZEOF_STRUCT_MEM) >= start_addr))
    {
        mem_size = end_align - begin_align - 2 * SIZEOF_STRUCT_MEM;
    }
    else
    {
        rt_kprintf("mem init, error begin address 0x%x, and end address 0x%x\n",
                   (rt_ubase_t)begin_addr, (rt_ubase_t)begin_addr + size);

        return RT_NULL;
    }

    rt_memset(small_mem, 0, sizeof(*small_mem));
    /* 初始化小内存对象 */
    rt_object_init(&(small_mem->parent.parent), RT_Object_Class_Memory, name);
    small_mem->parent.algorithm = "small";
    small_mem->parent.address = begin_align;
    small_mem->parent.total = mem_size;
    small_mem->mem_size_aligned = mem_size;

    /* 内存堆起始地址 */
    small_mem->heap_ptr = (rt_uint8_t *)begin_align;

    RT_DEBUG_LOG(RT_DEBUG_MEM, ("mem init, heap begin address 0x%x, size %d\n",
                                (rt_ubase_t)small_mem->heap_ptr, small_mem->mem_size_aligned));

    /* 初始化第一个内存块数据头 */
    mem        = (struct rt_small_mem_item *)small_mem->heap_ptr;
    mem->pool_ptr = MEM_FREED();
    mem->next  = small_mem->mem_size_aligned + SIZEOF_STRUCT_MEM;
    mem->prev  = 0;

    /* 初始化内存堆结束数据头 */
    small_mem->heap_end        = (struct rt_small_mem_item *)&small_mem->heap_ptr[mem->next];
    small_mem->heap_end->pool_ptr = MEM_USED();
    small_mem->heap_end->next  = small_mem->mem_size_aligned + SIZEOF_STRUCT_MEM;
    small_mem->heap_end->prev  = small_mem->mem_size_aligned + SIZEOF_STRUCT_MEM;
#ifdef RT_USING_MEMTRACE
    rt_smem_setname(small_mem->heap_end, "INIT");
#endif /* RT_USING_MEMTRACE */

    /* 空闲指针指向第一块可用内存块 */
    small_mem->lfree = (struct rt_small_mem_item *)small_mem->heap_ptr;

    return &small_mem->parent;
}
```

初始化之后的内存堆结构如下：

![](https://files.mdnice.com/user/44140/d996b9c3-bb4a-4b7c-b755-f481f132b7c3.png)

初始化之后，当前的内存堆系统有以下几个属性：

- 此时内存对系统中有两个数据头，一个表示内存堆可用内存块，另外一个表示内存堆结束数据头；随着内存的申请，数据头会依次增加
- 此时内存堆系统中只有一整块内存块，内存块大小为：mem_size_aliged 
- lfree 永远指向第一块空闲内存块，此时指向了 first mem;
- heap_ptr 作为一个常量，始终指向内存堆起始地址 (begin_align)
- heap_end 作为一个常量，始终指向内存堆结束数据头。

假设函数入参为：begin_addr = 0x01，size = 0x4000；RT_ALIGN_SIZE = 8，rt_smem_init 函数中各个变量的地址如下：

- begin_addr = 0x01
- small_mem = 0x08 (8 字节向上对齐)
- sizeof(struct rt_small_mem) = 0x38
- start_addr =  (0x08 + 0x38) = 0x40
- begin_align = 0x40 （这里 8 字节对齐刚好是 start_addr 的地址；可能会存在 begin_align > start_addr 的情况，这里不做讨论）
- end_align =  0x4000（8字节向下对齐）
- SIZEOF_STRUCT_MEM = sizeof(struct rt_small_mem_item *mem) = 0x10
- mem_size_aligned = 0x4000 - 0x40 - 2 * 0x10 = 0x3FA0
- mem->next = 0x3FA0 + 0x10 = 0x3FB0 (为 heap end 从 begin_align 的偏移量)

### rt_smem_detach

从系统中移除一个内存块对象。这个没什么过多解释，将小内存算法对象从 RT_Object_Class_Memory 链表中删除：

```c
rt_err_t rt_smem_detach(rt_smem_t m)
{
    RT_ASSERT(m != RT_NULL);
    RT_ASSERT(rt_object_get_type(&m->parent) == RT_Object_Class_Memory);
    RT_ASSERT(rt_object_is_systemobject(&m->parent));

    rt_object_detach(&(m->parent));

    return RT_EOK;
}
```

### rt_smem_alloc

从内存块中申请 size 大小的内存块；申请成功返回内存的首地址，申请失败则返回 RT_NULL：

```c
void *rt_smem_alloc(rt_smem_t m, rt_size_t size)
{
    rt_size_t ptr, ptr2;
    struct rt_small_mem_item *mem, *mem2;
    struct rt_small_mem *small_mem;

    if (size == 0)
        return RT_NULL;

    /* 字节对齐 */
    if (size != RT_ALIGN(size, RT_ALIGN_SIZE))
    {
        RT_DEBUG_LOG(RT_DEBUG_MEM, ("malloc size %d, but align to %d\n",
                                    size, RT_ALIGN(size, RT_ALIGN_SIZE)));
    }
    else
    {
        RT_DEBUG_LOG(RT_DEBUG_MEM, ("malloc size %d\n", size));
    }

    small_mem = (struct rt_small_mem *)m;
    size = RT_ALIGN(size, RT_ALIGN_SIZE);
    if (size < MIN_SIZE_ALIGNED)
        size = MIN_SIZE_ALIGNED;

    /* 申请的内存大于总内存容量 */
    if (size > small_mem->mem_size_aligned)
    {
        return RT_NULL;
    }
    /* 从第一个空闲内存块开始遍历，找到可用的内存块 */
    for (ptr = (rt_uint8_t *)small_mem->lfree - small_mem->heap_ptr;
         ptr <= small_mem->mem_size_aligned - size;
         ptr = ((struct rt_small_mem_item *)&small_mem->heap_ptr[ptr])->next)
    {
        /* 获取内存块的数据头 */
        mem = (struct rt_small_mem_item *)&small_mem->heap_ptr[ptr];
        /* 该内存块没有被使用，并且空间大小满足需求 */
        if ((!MEM_ISUSED(mem)) && (mem->next - (ptr + SIZEOF_STRUCT_MEM)) >= size)
        {
            /* 内存申请完之后，该内存块还有剩余空间（>= SIZEOF_STRUCT_MEM + MIN_SIZE_ALIGNED） */
            if (mem->next - (ptr + SIZEOF_STRUCT_MEM) >=
                (size + SIZEOF_STRUCT_MEM + MIN_SIZE_ALIGNED))
            {
                /* 剩余的内存空间建立新的内存块 */
                ptr2 = ptr + SIZEOF_STRUCT_MEM + size;
                /* 内存块数据头 */
                mem2       = (struct rt_small_mem_item *)&small_mem->heap_ptr[ptr2];
                /* 标记为空闲内存块 */
                mem2->pool_ptr = MEM_FREED();
                mem2->next = mem->next;
                mem2->prev = ptr;
                mem->next = ptr2;

                /* 判断内存堆是否结束*/
                if (mem2->next != small_mem->mem_size_aligned + SIZEOF_STRUCT_MEM)
                {
                    ((struct rt_small_mem_item *)&small_mem->heap_ptr[mem2->next])->prev = ptr2;
                }
                small_mem->parent.used += (size + SIZEOF_STRUCT_MEM);
                if (small_mem->parent.max < small_mem->parent.used)
                    small_mem->parent.max = small_mem->parent.used;
            }
            else /* 该内存块，内存分配完之后，剩余空间不足以建立新的内存块 */
            {
                small_mem->parent.used += mem->next - ((rt_uint8_t *)mem - small_mem->heap_ptr);
                if (small_mem->parent.max < small_mem->parent.used)
                    small_mem->parent.max = small_mem->parent.used;
            }
            /* 将申请到的内存块修改为已被使用 */ 
            mem->pool_ptr = MEM_USED();
            /* 需要调整空闲内存链表起始地址 */
            if (mem == small_mem->lfree)
            {
                /* 查找 mem 之后的下一个空闲块，并更新空闲块指针 */
                while (MEM_ISUSED(small_mem->lfree) && small_mem->lfree != small_mem->heap_end)
                    small_mem->lfree = (struct rt_small_mem_item *)&small_mem->heap_ptr[small_mem->lfree->next];
            }
            /* return the memory data except mem struct */
            return (rt_uint8_t *)mem + SIZEOF_STRUCT_MEM;
        }
    }

    return RT_NULL;
}
```

假设此时内存堆的结构如下图所示：

![](https://files.mdnice.com/user/44140/a1d9d45c-0c64-4320-845c-9f84083d7dde.png)

该函数的基本流程如下：
1. 检查入参 size 参数是否合法
2. 进行字节向上对齐操作，比如入参 size 为 13，对齐后的 size 为 16（这里还是 8 字节对齐）
3. 因为 lfree 永远指向第一个空闲内存块，所以从 lfree 开始遍历链表
4. 判断此时内存块的大小是否满足申请需求
5. 判断此时内存块分配完内存之后，是否还有足够的空闲建立新的内存块
6. 将申请的内存块设置为被使用
7. 调整 lfree 的地址，指向新的空闲内存块
8. 返回内存块起始地址

#### 获取内存块大小

在结构体 rt_small_mem_item  中，并没有成员变量显示指定内存块大小，如何判断内存块大小满足申请需求？
- 在 rt_small_mem_item 结构体中，next 的值并不是一个地址，而是一个偏移量，其基地址为 begin_align 
- 在上面的源代码中，ptr 的值就是一个偏移量，其值为 lfree 的偏移量，假设为 x1；此时可以通过数组下标的访问方式，访问内存块中任意内存数据块的数据头，mem = (struct rt_small_mem_item *)&small_mem->heap_ptr[ptr] 
- 在上面源代码中，第一次进行 for 循环时，mem = lfree；mem->next 获取到了 第二块 Used Memory 数据头的偏移量。假设为 x2,
- 通过上面获取到的两个偏移量，就可以计算中 lfree 指向的内存块的大小：（x2 - x1 - SIZEOF_STRUCT_MEM)

#### 分割内存块

如果申请的内存大小，小于空闲内存块大小，并且空闲内存块分配完之后的内存空间不小于数据头 + 12 字节（32 位系统最小内存块位 12字节，64 位为 24 字节），将剩余的内存空间建立新的内存数据头：

### rt_smem_realloc

调整之前已分配的内存块大小：

```c
void *rt_smem_realloc(rt_smem_t m, void *rmem, rt_size_t newsize)
{
    rt_size_t size;
    rt_size_t ptr, ptr2;
    struct rt_small_mem_item *mem, *mem2;
    struct rt_small_mem *small_mem;
    void *nmem;

    small_mem = (struct rt_small_mem *)m;
    /* 字节对齐 */
    newsize = RT_ALIGN(newsize, RT_ALIGN_SIZE);
    if (newsize > small_mem->mem_size_aligned)
    {
        RT_DEBUG_LOG(RT_DEBUG_MEM, ("realloc: out of memory\n"));

        return RT_NULL;
    }
    else if (newsize == 0) /* 重新分配的内存块大小为0，申请该内存块 */
    {
        rt_smem_free(rmem);
        return RT_NULL;
    }

    /* 申请一个新的内存块 */
    if (rmem == RT_NULL)
        return rt_smem_alloc(&small_mem->parent, newsize);

    mem = (struct rt_small_mem_item *)((rt_uint8_t *)rmem - SIZEOF_STRUCT_MEM);

    /* 计算内存块当前大小 */
    ptr = (rt_uint8_t *)mem - small_mem->heap_ptr;
    size = mem->next - ptr - SIZEOF_STRUCT_MEM;
    if (size == newsize)
    {
        /* 当前内存空间大小和要重新分配的大小相同，不进行分配，直接返回 */
        return rmem;
    }
    /* 新的内存空间大小小于旧的内存空间大小，并且满足分离内存块的条件，将当前内存块进行分割 */
    if (newsize + SIZEOF_STRUCT_MEM + MIN_SIZE < size)
    {
        /* 分离内存块 */
        small_mem->parent.used -= (size - newsize);
        /* 分离出来的内存块标记为 free */
        ptr2 = ptr + SIZEOF_STRUCT_MEM + newsize;
        mem2 = (struct rt_small_mem_item *)&small_mem->heap_ptr[ptr2];
        mem2->pool_ptr = MEM_FREED();
        mem2->next = mem->next;
        mem2->prev = ptr;

        mem->next = ptr2;
        if (mem2->next != small_mem->mem_size_aligned + SIZEOF_STRUCT_MEM)
        {
            ((struct rt_small_mem_item *)&small_mem->heap_ptr[mem2->next])->prev = ptr2;
        }

        if (mem2 < small_mem->lfree)
        {
            /* 修改 lfree 的指向 */
            small_mem->lfree = mem2;
        }
        /* 合并内存块 */
        plug_holes(small_mem, mem2);
        /* 返回新内存块的地址 */
        return rmem;
    }

    /* 重新申请 newsize 大小的内存空间 */
    nmem = rt_smem_alloc(&small_mem->parent, newsize);
    if (nmem != RT_NULL) /* 检查新申请的内存 */
    {
        /* 将旧内存块的数据拷贝到新的内存块地址空间 */
        rt_memcpy(nmem, rmem, size < newsize ? size : newsize);
        /* 释放旧内存块 */
        rt_smem_free(rmem);
    }
    /* 返回新内存块的地址 */
    return nmem;
}
```

重新分配内存空间有以下几种可能性：

1. newsize 大于内存堆的总大小，判定为非法入参，返回错误码
2. newsize 为 0，释放 rmem 内存空间
3. rmem 为 NULL，调用 malloc 函数，申请内存空间
4. size 等于 newsize ，不进行内存分配，直接返回 rmem
5. newsize + SIZEOF_STRUCT_MEM + MIN_SIZE 小于 size，说明 rmem 内存空间可以进行内存分割；最后判断是否可以进行内存块合并。其效果如下：
6. newsize 大于 size，直接调用 rt_smem_alloc 函数，寻找新的可用内存块；如果申请成功，将旧内存块上的内容拷贝到新的内存空间

### rt_smem_free

内存释放接口，将内存归还到内存堆中：

```c
void rt_smem_free(void *rmem)
{
    struct rt_small_mem_item *mem;
    struct rt_small_mem *small_mem;

    if (rmem == RT_NULL)
        return;
    /* 获取内存块数据头 */
    mem = (struct rt_small_mem_item *)((rt_uint8_t *)rmem - SIZEOF_STRUCT_MEM);
    small_mem = MEM_POOL(mem);
    /* 修改内存块状态 */
    mem->pool_ptr = MEM_FREED();
    if (mem < small_mem->lfree)
    {
        /* 修改 lfree 的指向 */
        small_mem->lfree = mem;
    }
    /* 修改已使用内存块的大小 */
    small_mem->parent.used -= (mem->next - ((rt_uint8_t *)mem - small_mem->heap_ptr));

    /* 最后，进行内存块合并 */
    plug_holes(small_mem, mem);
}
```

该函数的基本流程如下：

1. 获取要释放的内存区域数据头
2. 将内存块设置为未使用状态
3. 如果内存块的地址小于 lfree，则修改 lfree 的指向
4. 修改已被使用的内存大小
5. 合并内存块

**注意：释放过后的内存区域并没有清理数据，会有脏数据存在。再次申请时，可以通过 memset 清理脏数据。**

## 注意事项
在小内存管理算法中，如果频繁的进行内存的申请和释放，会产生内存碎片，造成的结果是没有可用的内存空间可以申请。

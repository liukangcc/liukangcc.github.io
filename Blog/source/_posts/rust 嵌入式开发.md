---
title: Rust代码 嵌入到 rt-thread
index_img: /img/rust.png
date: 2021-09-01 10:23:00
tags: rust
---

## Rust 是什么

Rust 是一门赋予每个人 构建可靠且高效软件能力的语言。

- 高性能：速度惊人且内存利用率极高
- 可靠性：在编译期就能消除各种内存错误
- 生产力：出色的文档，友好的编译器和清晰的错误提示信息

##  为什么要用 Rust 进行嵌入式开发

Rust 的设计理念：既要安全，也要高性能。Rust 的设计理念完全是嵌入式开发所需要的。

嵌入式软件在运行过程中出现问题，大部分是由于内存引起的。Rust 语言可以说是一门面向编译器的语言。在编译期间，就能够确保你安全地使用内存。

目前，嵌入式的主流开发语言还是 C 语言，不能上来就把底层的逻辑用 Rust 重新实现一遍。但是可以在 C 代码中嵌入 Rust 语言。

## C 调用 Rust

在 C 代码中调用 Rust 代码，需要我们将 Rust 源代码打包为静态库文件。在 C 代码编译时，链接进去。

### 创建 lib 库
 1. 在 Clion 中使用 `cargo init --lib rust_to_c` 建立 lib 库。添加以下代码到 lib.rs 中，使用 Rust 语言计算两个整数的和：

    ```rust
    #![no_std]
    use core::panic::PanicInfo;
    
    #[no_mangle]
    pub extern "C" fn sum(a: i32, b: i32) -> i32 {
        a + b
    }
    
    #[panic_handler]
    fn panic(_info:&PanicInfo) -> !{
        loop{}
    }
    ```

 2. 在 Cargo.toml 文件中添加以下代码，生成静态库文件：

    ```
    [lib]
    name = "sum"
    crate-type = ["staticlib"]
    path = "src/lib.rs"
    ```

### 交叉编译

1. 安装 armv7 target:

   ```
   rustup target add armv7a-none-eabi
   ```

2. 生成静态库文件：

   ```rust
   PS C:\Users\LiuKang\Desktop\RUST\rust_to_c> cargo build --target=armv7a-none-eabi --release --verbose
          Fresh rust_to_c v0.1.0 (C:\Users\LiuKang\Desktop\RUST\rust_to_c)
       Finished release [optimized] target(s) in 0.01s
   ```

### 生成头文件

1. 安装 [cbindgen]([eqrion/cbindgen: A project for generating C bindings from Rust code (github.com)](https://github.com/eqrion/cbindgen)), cbindgen 从 rust 库生成 C/C++ 11 头文件：

    ```
    cargo install --force cbindgen
    ```

2. 在项目文件夹下新建文件 `cbindgen.toml` 文件：

3. 生成头文件：

    ```
    cbindgen --config cbindgen.toml --crate rust_to_c --output sum.h
    ```

### 调用 Rust 库文件

1. 将生成的`sum.h` 以及 `sum.a` 文件放入 `rt-thread\bsp\qemu-vexpress-a9\applications` 目录下

2. 修改 SConscript 文件，添加静态库:

   ```rust
   from building import *
   
   cwd     = GetCurrentDir()
   src     = Glob('*.c') + Glob('*.cpp')
   CPPPATH = [cwd]
   
   LIBS = ["libsum.a"]
   LIBPATH = [GetCurrentDir()]
   
   group = DefineGroup('Applications', src, depend = [''], CPPPATH = CPPPATH, LIBS = LIBS, LIBPATH = LIBPATH)
   
   Return('group')
   ```

3. 在 main 函数中调用 sum 函数, 并获取返回值

   ```rust
   #include <stdint.h>
   #include <stdio.h>
   #include <stdlib.h>
   #include <rtthread.h>
   #include "sum.h"
   
   int main(void)
   {
       int32_t tmp;
   
       tmp = sum(1, 2);
       printf("call rust sum(1, 2) = %d\n", tmp);
   
       return 0;
   }
   ```
   
4. 在 env 环境下，使用 scons 编译工程：

```rust
   LiuKang@DESKTOP-538H6DE D:\repo\github\rt-thread\bsp\qemu-vexpress-a9
   $ scons -j6
   scons: Reading SConscript files ...
   scons: done reading SConscript files.
   
   scons: warning: you do not seem to have the pywin32 extensions installed;
   parallel (-j) builds may not work reliably with open Python files.
   File "D:\software\env_released_1.2.0\env\tools\Python27\Scripts\scons.py", line 204, in <module>
   scons: Building targets ...
   scons: building associated VariantDir targets: build
   LINK rtthread.elf
   arm-none-eabi-objcopy -O binary rtthread.elf rtthread.bin
   arm-none-eabi-size rtthread.elf
  text    data     bss     dec     hex filename
628220    2148   86700  717068   af10c rtthread.elf
   scons: done building targets.
   
   LiuKang@DESKTOP-538H6DE D:\repo\github\rt-thread\bsp\qemu-vexpress-a9
   $ qemu.bat
   WARNING: Image format was not specified for 'sd.bin' and probing guessed raw.
Automatically detecting the format is dangerous for raw images, write operations on block 0 will be restricted.
Specify the 'raw' format explicitly to remove the restrictions.
   
\ | /
   - RT -     Thread Operating System
/ | \     4.0.4 build Jul 28 2021
2006 - 2021 Copyright by rt-thread team
   lwIP-2.1.2 initialized!
   [I/sal.skt] Socket Abstraction Layer initialize success.
   [I/SDIO] SD card capacity 65536 KB.
   [I/SDIO] switching card to high speed failed!
   call rust sum(1, 2) = 3
   msh />
```

### 加减乘除

1. 在 lib.rs 文件中，使用 rust 语言实现加减乘除运算：

      ```rust
        #![no_std]
        use core::panic::PanicInfo;
        
        
        #[no_mangle]
        pub extern "C" fn add(a: i32, b: i32) -> i32 {
            a + b
        }
        
        #[no_mangle]
        pub extern "C" fn subtract(a: i32, b: i32) -> i32 {
            a - b
        }
        
        #[no_mangle]
        pub extern "C" fn multiply(a: i32, b: i32) -> i32 {
            a * b
        }
        
        #[no_mangle]
        pub extern "C" fn divide(a: i32, b: i32) -> i32 {
            a / b
        }
        
        #[panic_handler]
        fn panic(_info:&PanicInfo) -> !{
            loop{}
        }
    ```

2. 生成库文件和头文件并放在 application 目录下

3. 使用 scons 编译，链接时报错，在 rust github 仓库的 issues 中找到了 [解决办法](https://github.com/rust-lang/compiler-builtins/issues/353) ：

    ```rust
       LINK rtthread.elf
       d:/software/env_released_1.2.0/env/tools/gnu_gcc/arm_gcc/mingw/bin/../lib/gcc/arm-none-eabi/5.4.1/armv7-ar/thumb\libgcc.a(_arm_addsubdf3.o): In function `__aeabi_ul2d':
       (.text+0x304): multiple definition of `__aeabi_ul2d'
       applications\libsum.a(compiler_builtins-9b744f6fddf5e719.compiler_builtins.20m0qzjq-cgu.117.rcgu.o):/cargo/registry/src/github.com-1ecc6299db9ec823/compiler_builtins-0.1.35/src/float/conv.rs:143: first defined here
       collect2.exe: error: ld returned 1 exit status
       scons: *** [rtthread.elf] Error 1
       scons: building terminated because of errors.
    ```

4. 修改 `rtconfig.py` 文件， 添加链接参数 `--allow-multiple-definition`：

   ```rust
       DEVICE = ' -march=armv7-a -marm -msoft-float'
       CFLAGS = DEVICE + ' -Wall'
       AFLAGS = ' -c' + DEVICE + ' -x assembler-with-cpp -D__ASSEMBLY__ -I.'
       LINK_SCRIPT = 'link.lds'
       LFLAGS = DEVICE + ' -nostartfiles -Wl,--gc-sections,-Map=rtthread.map,-cref,-u,system_vectors,--allow-multiple-definition'+\
                         ' -T %s' % LINK_SCRIPT
   
       CPATH = ''
       LPATH = ''
   ```

5. 编译并运行 qemu：

```
   LiuKang@DESKTOP-538H6DE D:\repo\github\rt-thread\bsp\qemu-vexpress-a9
   $ scons -j6
   scons: Reading SConscript files ...
   scons: done reading SConscript files.
   
   scons: warning: you do not seem to have the pywin32 extensions installed;
   parallel (-j) builds may not work reliably with open Python files.
   File "D:\software\env_released_1.2.0\env\tools\Python27\Scripts\scons.py", line 204, in <module>
   scons: Building targets ...
   scons: building associated VariantDir targets: build
   LINK rtthread.elf
   arm-none-eabi-objcopy -O binary rtthread.elf rtthread.bin
   arm-none-eabi-size rtthread.elf
  text    data     bss     dec     hex filename
628756    2148   86700  717604   af324 rtthread.elf
   scons: done building targets.
   
   LiuKang@DESKTOP-538H6DE D:\repo\github\rt-thread\bsp\qemu-vexpress-a9
   $ qemu.bat
   WARNING: Image format was not specified for 'sd.bin' and probing guessed raw.
    Automatically detecting the format is dangerous for raw images, write operations on block 0 will be restricted.
    Specify the 'raw' format explicitly to remove the restrictions.
   
\ | /
   - RT -     Thread Operating System
/ | \     4.0.4 build Jul 28 2021
2006 - 2021 Copyright by rt-thread team
   lwIP-2.1.2 initialized!
   [I/sal.skt] Socket Abstraction Layer initialize success.
   [I/SDIO] SD card capacity 65536 KB.
   [I/SDIO] switching card to high speed failed!
   call rust sum(1, 2) = 3
   call rust subtract(2, 1) = 1
   call rust multiply(2, 2) = 4
   call rust divide(4, 2) = 2
```

## Rust 调用 C

可以 在 C 代码中调用 Rust，那么在 Rust 中也可以调用 C 代码。我们在  Rust 代码中调用 rt_kprintf 函数：

### 修改 lib.rs 文件

    ```rust
        // 导入的 rt-thread 函数列表
        extern "C" {
            pub fn rt_kprintf(format: *const u8, ...);
        }
        
        #[no_mangle]
        pub extern "C" fn add(a: i32, b: i32) -> i32 {
            unsafe {
                rt_kprintf(b"this is from rust\n\0" as *const u8);
            }
            a + b
        }
    ```

### 生成库文件

    ```rust
        cargo build --target=armv7a-none-eabi --release --verbose
           Compiling rust_to_c v0.1.0 (C:\Users\LiuKang\Desktop\RUST\rust_to_c)
             Running `rustc --crate-name sum --edition=2018 src/lib.rs --error-format=json --json=diagnostic-rendered-ansi --crate-type staticlib --emit=dep-info,link -C opt-level=3 -C embed-bitcode=no -C metadata=a
        0723fa112c78339 -C extra-filename=-a0723fa112c78339 --out-dir C:\Users\LiuKang\Desktop\RUST\rust_to_c\target\armv7a-none-eabi\release\deps --target armv7a-none-eabi -L dependency=C:\Users\LiuKang\Desktop\RUS
        T\rust_to_c\target\armv7a-none-eabi\release\deps -L dependency=C:\Users\LiuKang\Desktop\RUST\rust_to_c\target\release\deps`
            Finished release [optimized] target(s) in 0.11s
    ```

### 运行

复制 rust 生成的库文件到 application 目录下。

    ```shell
        LiuKang@DESKTOP-538H6DE D:\repo\github\rt-thread\bsp\qemu-vexpress-a9                                                       
        $ scons -j6                                                                                                             
        scons: Reading SConscript files ...                                                                                  
        scons: done reading SConscript files.                                                                                                     
        scons: warning: you do not seem to have the pywin32 extensions installed;                                                   
                parallel (-j) builds may not work reliably with open Python files.                                                  
        File "D:\software\env_released_1.2.0\env\tools\Python27\Scripts\scons.py", line 204, in <module>                            
        scons: Building targets ...                                                                                                 
        scons: building associated VariantDir targets: build                                                                        
        LINK rtthread.elf                                                                                                           
        arm-none-eabi-objcopy -O binary rtthread.elf rtthread.bin                                                                   
        arm-none-eabi-size rtthread.elf                                                                                             
           text    data     bss     dec     hex filename                                                                            
         628812    2148   90796  721756   b035c rtthread.elf                                                                        
        scons: done building targets.                                                                                               
                                                                                                                                    
        LiuKang@DESKTOP-538H6DE D:\repo\github\rt-thread\bsp\qemu-vexpress-a9                                                       
        $ qemu.bat                                                                                                                  
        WARNING: Image format was not specified for 'sd.bin' and probing guessed raw.                                               
                 Automatically detecting the format is dangerous for raw images, write operations on block 0 will be restricted.    
                 Specify the 'raw' format explicitly to remove the restrictions.                                                    
                                                                                                                                    
         \ | /                                                                                                                      
        - RT -     Thread Operating System                                                                                          
         / | \     4.0.4 build Jul 28 2021                                                                                          
         2006 - 2021 Copyright by rt-thread team                                                                                    
        lwIP-2.1.2 initialized!                                                                                                     
        [I/sal.skt] Socket Abstraction Layer initialize success.                                                                    
        [I/SDIO] SD card capacity 65536 KB.                                                                                         
        [I/SDIO] switching card to high speed failed!                                                                               
        this is from rust                                                                                                           
        call rust sum(1, 2) = 3                                                                                           
        call rust subtract(2, 1) = 1                                                                                                
        call rust multiply(2, 2) = 4                                                                                                
        call rust divide(4, 2) = 2                                                                                     
        msh />                                                                                                                      
    ```
# STL（standard Template Library）

## STL 的诞生

- 长久以来，软件界一直希望建立一种可重复利用的东西

- c++ 的面向对象和泛型编程思想，目的就是复用性的提升

- 大多情况下，数据结构和算法都未能有一套标准，导致被迫从事大量重复工作

- 为了建立数据结构和算法的一套标准，诞生了 STL

## 基本概念

- STL 从广义上分为：容器（container），算法 (algorithm)，迭代器 (iterator)

- 容器和算法之间通过迭代器进行无缝连接

- STL 几乎所有的代码都采用了模板或者模板函数

## STL 六大组件

- 容器：各种数据结构，如：vector、list、deque、set、map 等，用来存放数据

- 算法：各种常用算法。如：sort、find、copy、for_each

- 迭代器：扮演了容器与算法之间的胶合剂

- 仿函数：行为类似函数，可作为算法的某种策略

- 适配器（配接器）：修饰容器或者仿函数或迭代器接口的东西

- 空间配置器：负责空间的配置和管理

### 容器

- 关联式容器：二叉树结构

- 序列式容器：强调值的排序

### 算法

有限的步骤，解决逻辑或者数学上的问题

- 质变算法：运算过程中会更改区间内的元素的内容。如：拷贝、替换、删除等

- 非质变算法：运算过程中不会更改区间内的元素内容。如：查找、计数、遍历等

### 迭代器

提供一种方法，使之能够依序访问某个容器所含的各个元素，而又无需暴露该容器的内部表示方法。

种类:

| 种类           | 功能 | 支持运算 |
| -------------- | ---- | -------- |
| 输入迭代器     |      |          |
| 输出迭代器     |      |          |
| 前向迭代器     |      |          |
| 双向迭代器     |      |          |
| 随机访问迭代器 |      |          |
|                |      |          |

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v;
    v.push_back(0);
    v.push_back(1);
    v.push_back(2);
    v.push_back(3);

    for (std::vector<int>::iterator i = v.begin(); i != v.end(); ++i) {
        std::cout << *i << std::endl;
    }

    /*
    for (auto &i: v) {
         std::cout << i << std::endl;
    }
    */
    return 0;
}

```

输出：

```shell
0
1
2
3
```

## std

### string

#### 初始化

```c++
std::string st1 = "hello, world";
std::string str2("hello, world");
```

#### 赋值

```c++
#include<iostream>
#include<string>

using namespace std;

int main() {
    string str1;
    str1 = "hello, world";
    cout << str1 << endl;

    string str2;
    str2 = str1;
    cout << str2 << endl;

    /* 单个字符赋值 */
    string str3;
    str3 = 'a';
    cout << str3 << endl;

    // 把字符串 s 赋值给当前字符串
    string str4;
    str4.assign("hello, c++");
    cout << str4 << endl;

    // 把当前字符串 S 的前 n 个字符赋值给当前字符串
    string str5;
    str5.assign("hello, c++", 5);

    string str6;
    str6.assign(str5);

    return 0;
}

```

输出：

```shell
hello, world
hello, world
a
hello, c++
```

#### 拼接

```c++
#include<algorithm>
#include<iostream>
#include<vector>

using namespace std;

int main() {
    string str = "hello";
    string str1 = ",world";

    str += ", world";
    cout << str << endl;

    str = "hello";
    str += ',';
    cout << str << endl;

    str = "hello";
    str += str1;
    cout << str << endl;

    str = "hello";
    str.append(",world");
    cout << str << endl;

    str = "hello";
    str.append(",world", 8);
    cout << str << endl;

    str = "hello";
    str.append(str1);
    cout << str << endl;

    str = "hello";
    str.append(str1, 0, 8);
    cout << str << endl;

    return 0;
}
```

输出：

```
hello , world
hello,
hello ,world
hello ,world
hello ,world
hello ,world
hello ,world
```

#### 查找和替换

```c++
#include <iostream>
#include <algorithm>
#include <vector>

using namespace std;

int main() {

    string str("liukangcc, hello, world!");

    cout <<str.find("hello", 0) << endl;
    cout <<str.find('c', 0) << endl;
    cout <<str.find("hello, world", 0, 5) << endl;
    cout <<str.rfind("hello", str.size()) << endl;
    cout <<str.rfind('c', str.size()) << endl;
    cout <<str.rfind("hello, world", str.size(), 5) << endl;

    string str_son = str.replace(0, 9, "iopwsopq");
    cout << str_son << endl;

    return 0;
}

```

输出

```
11
7
11
11
8
11
iopwsopq, hello, world!
```

#### 比较

```c++
#include <iostream>
#include <algorithm>

using namespace std;

int main() {

    string str("liukangcc, hello, world!");
    string str1("liukangcc, hello, world.");

    cout <<str.compare(str1) << endl;

    return 0;
}

```

输出

```shell
-1
```

#### 字符存取

```
#include <algorithm>
#include <iostream>


using namespace std;

int main() {
    string str("liukangcc, hello, world!");

    for (int i = 0; i < str.size(); ++i) {
        cout <<str[i];
    }
    cout << endl;

    for (int i = 0; i < str.size(); ++i) {
        cout <<str.at(i);
    }
    cout << endl;

    return 0;
}

```

输出

```
liukangcc, hello, world!
liukangcc, hello, world!
```

#### 插入和删除

** 功能描述：**

- 对 string 字符串进行插入和删除字符操作

** 函数原型：**

- string& insert(int pos, const char* s); // 插入字符串

- string& insert(int pos, const string& str); // 插入字符串

- string& insert(int pos, int n, char c); // 在指定位置插入 n 个字符 c

- string& erase(int pos, int n = npos); // 删除从 Pos 开始的 n 个字符

#### string 子串

** 功能描述：**

- 从字符串中获取想要的子串

** 函数原型：**

- string substr(int pos = 0, int n = npos) const;// 返回由 pos 开始的 n 个字符组成的字符串

### vector

** 功能：**

- vector 数据结构和 ** 数组非常相似 **，也称为 ** 单端数组 **

**vector 与普通数组区别：**

- 不同之处在于数组是静态空间，而 vector 可以 ** 动态扩展 **

** 动态扩展：**

- 并不是在原空间之后续接新空间，而是找更大的内存空间，然后将原数据拷贝新空间，释放原空间

![img](https://vipkshttps6.wiz.cn/editor/784db3c0-1b4e-11ec-8bf2-3759ddf0e07e/00bdc86a-e379-48e6-8130-caa4c67af74b/resources/9nFThyYcSUxOVWcufNTesMx_ihe6mOAj7Q6bBr34OYs.png?token=W.0t45nsSJjIsEtzO7zjBZHWUXJy17Nh5Ix9ch_Vy61Knaa3PJBjHO250GmTkd1aA)

- vector 容器的迭代器是支持随机访问的迭代器

#### 构造函数

- vector<T> v; // 采用模板实现类实现，默认构造函数

- vector(v.begin(), v.end()); // 将 v[begin(), end()) 区间中的元素拷贝给本身。

- vector(n, elem); // 构造函数将 n 个 elem 拷贝给本身。

- vector(const vector &vec); // 拷贝构造函数。

```c++
#include <iostream>
#include <vector>

using namespace std;

void printVector(vector<int>& v) {
    for (vector<int>::iterator it = v.begin(); it != v.end(); it++) {
        cout << *it << " ";
    }

    cout << endl;
}

void test01() {

    vector<int> v1; // 无参构造

    for (int i = 0; i < 10; ++i) {
        v1.push_back(i);
    }
    printVector(v1);

    vector<int> v2(v1.begin(), v1.end());
    printVector(v2);

    vector<int> v3(10, 100);
    printVector(v3);

    vector<int> v4(v3);
    printVector(v4);
}

int main() {
   test01();
   return 0;
}
```

输出

```
0 1 2 3 4 5 6 7 8 9
0 1 2 3 4 5 6 7 8 9
100 100 100 100 100 100 100 100 100 100
100 100 100 100 100 100 100 100 100 100
```

#### 赋值操作

** 功能描述：**

- 给 vector 容器进行赋值

```c++
#include <iostream>
#include <vector>

using namespace std;

void printVector(vector<int>& v) {
    for (auto & it : v) {
        cout << it << " ";
    }

    cout << endl;
}

int main() {
    vector<int> v1;

    for (int i = 0; i < 10; ++i) {
        v1.push_back(i);
    }
    printVector(v1);

    vector<int> v2 = v1;
    printVector(v2);

    vector<int> v3 ;
    v3.assign(v2.begin(), v2.end());
    printVector(v3);

    vector<int> v4 ;
    v4.assign(10, 100);
    printVector(v4);

    return 0;
}

```

输出

```shell
0 1 2 3 4 5 6 7 8 9
0 1 2 3 4 5 6 7 8 9
0 1 2 3 4 5 6 7 8 9
100 100 100 100 100 100 100 100 100 100
```

#### 容量和大小

** 功能描述：**

- 对 vector 容器的容量和大小操作

** 函数原型：**

- empty()                        判断容器是否为空

- capacity()                     容器的容量

- size();                            返回容器中元素的个数

- resize(int num)            重新指定容器的长度为 num，若容器变长，则以默认值填充新位置。如果容器变短，则末尾超出容器长度的元素被删除。

- resize(int num, elem)  重新指定容器的长度为 num，若容器变长，则以 elem 值填充新位置。如果容器变短，则末尾超出容器长度的元素被删除

#### 删除操作

- erase

- remove

### deque

描述：双端数组，可以对头进行插入删除操作

- deque 与 vector 区别：vector 对于头部的插入删除效率低，数据量越大，效率越低

- deque 相对而言，对头部的插入删除速度会比 vector 快，vector 访问元素时的速度会比 deque 快, 这和两者内部实现有关

- deque 容器中存储元素并不能保证所有元素都存储到连续的内存空间中。** 当需要向序列两端频繁的添加或删除元素时，应首选 deque 容器。**

### List

### mutex

互斥量。保护共享资源，任意时刻只允许一个线程访问共享资源。

- mutex.lock() 上锁

- mutex.unlock() 解锁

### lock_guard

mutex 会造成死锁。lock_guard() 只有构造函数和析构函数，当调用构造函数时，会自动传入的对象的 lock() 函数，而当调用析构函数时，自动调用 unlock() 函数。std::lock_guard 是非常巧妙的一种设计思路，利用类对象的生命周期，构造时使互斥量加锁，析构时使互斥量解锁，从而实现其 ** 作用域内 ** 对互斥量的管理。

```c++
#include <iostream>
#include <mutex>
#include <thread>
#include <chrono>

std::mutex mtx;

int number = 0;

void test_lock() {

    std::lock_guard<std::mutex> lk(mtx); // create a lock_guard class
    std::thread::id id = std::this_thread::get_id();

    std::cout << "cout ----- id :" << id  << std::endl;

    number++;
}

void test_thread(void) {

    while (1) {
        if (number>= 10) {
            break;
        }
        test_lock();
        std::cout << "thread number:" << number << std::endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
}

int main() {

    std::thread t1(test_thread);
    std::thread t2(test_thread);

    t1.join();
    t2.join();

    std::cout << "number is:" << number << std::endl;

    return 0;
}
```

输出

```
cout ----- id : 2
thread number: 1
cout ----- id : 3
thread number: 2
cout ----- id : 3
thread number: 3
cout ----- id : 2
thread number: 4
cout ----- id : 2
thread number: 5
cout ----- id : 3
thread number: 6
cout ----- id : 2
thread number: 7
cout ----- id : 3
thread number: 8
cout ----- id : 2
thread number: 9
cout ----- id : 3
thread number: 10
number is: 10
```

### map

map 是 STL 的一个关联容器，以键值对存储的数据，其类型可以自己定义，每个关键字在 map 中只能出现一次，关键字不能修改，值可以修改；map 同 set、multiset、multimap（与 map 的差别仅在于 multimap 允许一个键对应多个值）内部数据结构都是红黑树，而 java 中的 hashmap 是以 hash table 实现的。所以 map 内部有序（自动排序，单词时按照字母序排序），查找时间复杂度为 O(logn)。

#### 操作方法

##### 声明

- 头文件： `#include <map>`

- 定义：
  - <int, sting> my_map;
  - typedef map<int, string> My_Map;
  - My_Map my_map;

##### 插入一个元素

- my_map.insert()

- my_map.emplace()

##### 清空

- my_map.clear()

##### 删除一个元素

- my_map.erase()

##### 获取长度

- my_map.size()

##### 获取头部的迭代器

- my_map.begin()

##### 获取末尾的迭代器

- my_map.end()

##### 判断 map 是否为空

- my_map.empty()

##### 交换两个 map

两个 map 中所有元素都交换

- my_map.swap()

##### 插入

- 第一种：用 insert 函数插入 pair 数据：

  ```c++
  map<int,string> my_map;
  my_map.insert(pair<int,string>(1,"first"));
  my_map.insert(pair<int,string>(2,"second"));
  ```

- 第二种：用 insert 函数插入 value_type 数据：

  ```c++
  map<int,string> my_map;
  my_map.insert(map<int,string>::value_type(1,"first"));
  my_map.insert(map<int,string>::value_type(2,"second"));
  ```

- 第三种：用数组的方式直接赋值：

  ```c++
  map<int,string> my_map;
  my_map[1]="first";
  my_map[2]="second";

##### 遍历

```c++
#include <iostream>
#include <unordered_map>

using namespace std;

int main() {

    unordered_map<string, int> mp;

    mp["Li"] = 20;
    mp["Wang"] = 18;
    mp["Zhao"] = 30;

    // 方式一、迭代器
    cout << "first: itor" << endl;
    for (auto it = mp.begin(); it != mp.end(); it++) {
        cout <<it->first << " " << it->second << endl;
    }

    // 方式二、range for C++ 11 版本及以上
    cout << "second: range for" << endl;
    for (auto it : mp) {
        cout << it.first << " " << it.second << endl;
    }

    // 方法三、 C++ 17 版本及以上
    cout << "third" << endl;
    for (auto [key, val] : mp) {
        cout << key  << " " << val << endl;
    }

    return 0;
}

```

输出

```shell
first: itor
Zhao 30
Wang 18
Li 20
second: range for
Zhao 30
Wang 18
Li 20
third
Zhao 30
Wang 18
Li 20
```

### bind

std::bind 的 [头文件](https://so.csdn.net/so/search?q = 头文件 & spm=1001.2101.3001.7020) 是 ，它是一个函数适配器，接受一个可调用对象（callable object），生成一个新的可调用对象来 “适应” 原对象的参数列表。

#### 函数原型

std::bind 函数有两种函数原型，定义如下：

```c++
template<class F, class... Args>
/*unspecified*/ bind(F&& f, Args&&... args);


template<class R, class F, class... Args>
/*unspecified*/ bind(F&& f, Args&&... args);
```

std::bind 返 回一个基于 f 的函数对象，其参数被绑定到 args 上 。

f 的参数要么被绑定到值，要么被绑定到 placeholders（占位符，如_1, _2, ..., _n）。

std::bind 将可调用对象与其参数一起进行绑定，绑定后的结果可以使用 std::function 保存。std::bind 主要有以下两个作用：

将可调用对象和其参数绑定成一个防函数；

只绑定部分参数，减少可调用对象传入的参数。

#### 绑定普通函数

```c++
double callableFunc (double x, double y) {return x/y;}
auto NewCallable = std::bind (callableFunc, std::placeholders::_1,2);

std::cout <<NewCallable (10) << std::endl;
```

- bind 的第一个参数是函数名，普通函数做实参时，会隐式转换成函数指针。因此 std::bind(callableFunc,_1,2) 等价于 std::bind (&callableFunc,_1,2)；

- _1 表示占位符，位于 <functional> 中，std::placeholders::_1；

- 第一个参数被占位符占用，表示这个参数以调用时传入的参数为准，在这里调用 NewCallable 时，给它传入了 10，其实就想到于调用 callableFunc(10,2);

#### 绑定一个成员函数

```c++
class Base {
    void display_sum(int a1, int a2) {
        std::cout << a1 + a2 << '\n';
    }

    int m_data = 30;
};

int main() {
   Base base;
   auto newiFunc = std::bind(&Base::display_sum, &base, 100, std::placeholders::_1);
}
```

- bind 绑定类成员函数时，第一个参数表示对象的成员函数的指针，第二个参数表示对象的地址。

- 必须显式地指定 & Base::diplay_sum，因为编译器不会将对象的成员函数隐式转换成函数指针，所以必须在 Base::display_sum 前添加 &；

- 使用对象成员函数的指针时，必须要知道该指针属于哪个对象，因此第二个参数为对象的地址 &base；

#### 绑定一个引用参数

默认情况下，bind 的那些不是占位符的参数被拷贝到 bind 返回的可调用对象中。但是，与 lambda 类似，有时对有些绑定的参数希望以引用的方式传递，或是要绑定参数的类型无法拷贝。

```c++
#include <iostream>
#include <functional>
#include <vector>
#include <algorithm>
#include <sstream>

using namespace std::placeholders;
using namespace std;

ostream & printInfo(ostream &os, const string& s, char c) {
    os << s << c;
    return os;
}

int main() {

    vector<string> words{"welcome", "to", "C++11"};
    ostringstream os;

    char c = ' ';

    for_each(words.begin(), words.end(), [&os, c](const string & s){os << s << c;} );

    cout <<os.str() << endl;

    ostringstream os1;

    // ostream 不能拷贝，若希望传递给 bind 一个对象，
    // 而不拷贝它，就必须使用标准库提供的 ref 函数
    for_each(words.begin(), words.end(),bind(printInfo, ref(os1), _1, c));
    cout <<os1.str() << endl;

    return 0;
}

```

输出

```
welcome to C++11
welcome to C++11
```
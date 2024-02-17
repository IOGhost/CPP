# C/C++ 字节对齐

**面试高频指数：★★★★☆**

在C/C++中，字节对齐是内存分配的一种策略。

当分配内存时，编译器会自动调整数据结构的内存布局，使得数据成员的起始地址与其自然对齐边界（一般为自己大小的倍数）相匹配。

## 字节对齐的作用和好处

计算机的内存是一块用于存储数据的空间，由一系列连续的存储单元组成，就像下面这样，
![](https://cdn.how2cs.cn/gzh/0081Kckwgy1gk6p1iowcxj30t20acmz8.jpg)

由于 1 个 bit 只能表示两个状态，所以大佬们规定 8个 bit 为一组，命名为 byte。

并且将 byte 作为内存寻址的最小单元，也就是给每个 byte 一个编号，这个编号就叫内存的**地址**。

![](https://cdn.how2cs.cn/gzh/0081Kckwgy1gk6pcjje44j30qe09ggnr.jpg)

(PS: 关于内存更多理解可以看这篇文章: [内存是什么？](https://csguide.cn/cpp/memory/what_is_memory.html))

理论上，任何类型的变量都可以从任意地址开始存放。

然而实际上，访问特定类型的变量通常需要从特定对齐的内存地址开始。

**因为如果不对数据存储进行适当的对齐，可能会导致存取效率降低。**

所以，各种数据类型需要按照一定的规则在内存中排列（起始地址），而不是顺序地一个接一个排放，这种排列就是字节对齐。

例如，有些平台每次读取都是从偶数地址开始。如果一个 int 类型（假设为 32 位系统）存储在偶数地址开始的位置，那么一个读周期就可以读取这 32 位。

但如果存储在奇数地址开始的位置，则需要两个读周期，并将两次读取的结果的高低字节拼凑才能得到这 32  位数据。

显然这会显著降低读取效率。

**总结： 字节对齐有助于提高内存访问速度，因为许多处理器都优化了对齐数据的访问。但是，这可能会导致内存中的一些空间浪费。**

## 字节对齐规则

以下是字节对齐的一些基本规则：

### **1. 自然对齐边界**
对于基本数据类型，其自然对齐边界通常为其大小。

例如，`char` 类型的自然对齐边界为 1 字节，`short` 为 2 字节，`int` 和 `float` 为 4 字节，`double` 和 64 位指针为 8 字节。具体数值可能因编译器和平台而异。

### **2. 结构体对齐**
结构体内部的每个成员都根据其自然对齐边界进行对齐。

也就是可能在成员之间插入填充字节。

结构体本身的总大小也会根据其最大对齐边界的成员进行对齐（比如结构体成员包含的最长类型为int类型，那么整个结构体要按照4的倍数对齐），以便在数组中正确对齐。
### **3. 联合体对齐**
联合体的对齐边界取决于其最大对齐边界的成员。联合体的大小等于其最大大小的成员，因为联合体的所有成员共享相同的内存空间。
### **4. 编译器指令**
可以使用编译器指令（如 `#pragma pack`）更改默认的对齐规则。这个命令是全局生效的。这可以用于减小数据结构的大小，但可能会降低访问性能。
### **5. 对齐属性**
在 C++11 及更高版本中，可以使用 `alignas` 关键字为数据结构或变量指定对齐要求。这个命令是对某个类型或者对象生效的。例如，`alignas(16) int x;` 将确保 `x` 的地址是 16 的倍数。
### **6. 动态内存分配**

大多数内存分配函数（如 `malloc` 和 `new`）会自动分配足够对齐的内存，以满足任何数据类型的对齐要求。

## 实际案例分析

举例说明下:

```cpp
#include <iostream>

#pragma pack(push, 1) // 设置字节对齐为 1 字节，取消自动对齐
struct UnalignedStruct {
    char a;
    int b;
    short c;
};
#pragma pack(pop) // 恢复默认的字节对齐设置

struct AlignedStruct {
    char a;   // 本来1字节，padding 3 字节
    int b;    //  4 字节
    short c;  // 本来 short 2字节，但是整体需要按照 4 字节对齐(成员对齐边界最大的是int 4) 
              // 所以需要padding 2
   // 总共: 4 + 4 + 4
};

struct MyStruct {
 double a;    // 8 个字节
 char b;      // 本来占一个字节，但是接下来的 int 需要起始地址为4的倍数
              //所以这里也会加3字节的padding
 int c;       // 4 个字节
 // 总共:  8 + 4 + 4 = 16
};

struct MyStruct1 {
 char b;    // 本来1个字节 + 7个字节padding
 double a;  // 8 个字节
 int c;     // 本来 4 个字节，但是整体要按 8 字节对齐，所以 4个字节padding
  // 总共: 8 + 8 + 8 = 24
};


int main() {
    std::cout << "Size of unaligned struct: " << sizeof(UnalignedStruct) << std::endl; 
    // 输出：7
    std::cout << "Size of aligned struct: " << sizeof(AlignedStruct) << std::endl; 
    // 输出：12，取决于编译器和平台
    std::cout << "Size of aligned struct: " << sizeof(MyStruct) << std::endl; 
    // 输出：16，取决于编译器和平台
    std::cout << "Size of aligned struct: " << sizeof(MyStruct1) << std::endl;
     // 输出：24，取决于编译器和平台
    return 0;
}
```


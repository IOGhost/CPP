常量表达式是指值不会改变并且在编译过程就能够得到计算结果的表达式。显然，字面值属于常量表达式，用常量表达式初始化的 const 对象也是常量表达式。如下：

```cpp
const int max_num = 20;           // max_num是常量表达式
const int limit = max_num + 1;    // limit 是常量表达式
int staff_size = 2;               // staff_size 不是常量表达式，因为staff_size没有用const修饰
const int zs = get_size();        // sz 不是常量表达式，虽然sz是个常量，但它的值在运行时才能确定
```


可见：一个对象（或表达式）是不是常量表达式由它的数据类型（是否const）和初始值共同决定。


constexpr(const expression)：

const可以用来修饰常量，可是只有当其初始值是个常量表达式时，const修饰的对象才是个常量表达式。C++11 提出了 constexpr 用于定义常量表达式。一般而言，如果你认定变量是一个常量表达式，那就把它声明成 constexpr 类型。constexpr 变量在定义时必须初始化.

```cpp
constexpr int mf = 20;		// mf是常量表达式
constexpr int limit = mf + 1;	// limit 是常量表达式
constexpr int sz = get_sz();	// 只有当 get_sz() 是一个 constexpr 函数时才是一条正确的声明语句
```

指针和 constexpr：

当 constexpr 修饰指针时，constexpr 仅对指针有效，与指针所指的对象无关：

```cpp
const int *p = nullptr;		// p 是指向常量的指针
constexpr int *q = nullptr;	// q 是常指针，constexpr 仅对指针有效
```

当然，const 和 constptr 可以一起来修饰一个指针，用于表明指向常量的常指针

```cpp
constexpr const int *p = &i;	// 指向常量的常指针，注意 &i 必须是常量表达式，i可以是全局变量或静态变量等等。。这些变量的地址在编译时就确定了
```

constexpr函数：
constexpr函数是指能用于常量表达式的函数。不同于一般函数，constexpr 函数的返回值类型及所有形参的类型都是字面值类型（算术类型，引用，指针等属于字面值类型），而且函数体中有且只有一条 return 语句。为了能在编译过程中随之展开，constexpr函数被隐式地指定为内联函数。constexpr 函数的返回值在编译时就能被确定。

我们允许 constexpr 函数不一定返回常量表达式，但是我们认为：

```cpp
// 如果 arg 为常量表达式，则 scale(arg)也是常量表达式
constexpr size_t scale(size_t cnt) { return 2 * cnt;}
```

举例：

```cpp
#include <iostream>
 
constexpr size_t scale(size_t cnt) {
	return 2 * cnt;
}
 
int main () {
	constexpr size_t sz;
	constexpr size_t s = 3;
        sz = scale(s);			
	std::cout << sz << std::endl;
	return 0;
}
```
可见，constexpr 函数不一定返回常量表达式，但是在 scale(3) 可以在编译时确定。
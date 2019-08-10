---
layout:     post
title:      "More Effective C++学习笔记（2）-运算符"
date:       2017-01-08 12:00:00
author:     "邵正"
header-img: "img/home-bg-o.jpg"
tags:
    - C++
    - More effective c++
---


| 主题     | 概要                      |
| -------- | ------------------------- |
| C++      | More Effective C++ 运算符 |
| -------- | ---                       |
| **编辑** | **时间**                  |
| 新建     | 20170108                  |
| -------- | ---                       |
| **序号** | **参考资料**              |
| 1        | More effective C++        |


功力不到，总觉得C++的重载运算符是非常恶心、难以理解的存在。特别是接触过JAVA与C#以后。但不懂运算符，如何谈C++？

## Item M5：谨慎定义类型转换函数 ##
C++编译器除了像C语言支持数值转换，比如把char隐式转换为int和从short转换为double等。它还提供两种函数形式的转换，即：单参数构造函数（single-argument constructors）和隐式类型转换运算符。
### 单参数构造函数 ###
单参数构造函数是指只用一个参数即可以调用的构造函数。该函数可以是只定义了一个参数，也可以是虽定义了多个参数但第一个参数以后的所有参数都有缺省值。
通过单参数构造函数进行隐式类型转换更难消除。而且在很多情况下这些函数所导致的问题要甚于隐式类型转换运算符。

举一个例子，一个array类模板，这些数组需要调用者确定边界的上限与下限：

```c++
template<class T>
class Array {
public:
	Array(int lowBound, int highBound);
	Array(int size);    //--能做为类型转换函数使用，是无穷痛苦的源泉
	T& operator[](int index);
}

```

第一个构造函数允许调用者确定数组索引的范围，例如从10到20。它是一个两参数构造函数，所以不能做为类型转换函数。第二个构造函数让调用者仅仅定义数组元素的个数（使用方法与内置数组的使用相似），不过不同的是它能做为类型转换函数使用，能导致无穷的痛苦。
反例，

```c++
bool operator==(const Array<int>& lhs,
	const Array<int>& rhs);

Array<int> a(10);
Array<int> b(10);

for (int i = 0; i < 10; ++i)
{
	if (a == b[i]) { // 哎呦! "a" 应该是 "a[i]"
		do something for when
			a[i] and b[i] are equal;
	}
	else {
		do something for when they're not;
	}
}

```
这里定义了两个大小为10的int型数组a、b，凑巧重载了==操作符，当出现语句：
if (a == b[i]) 时，编译器并不按期望的报错。
此时编译器它把这个调用看成用Array<int>参数(对于a)和int(对于b[i])参数调用operator==函数。然而没有operator==函数是这样的参数类型，我们的编译器注意到它能通过调用Array<int>构造函数能转换int类型到Array<int>类型，这个构造函数只有一个int类型的参数。然后编译器如此去编译，生成的代码就象这样：

```c++
for (int i = 0; i < 10; ++i)
	if (a == static_cast<Array<int>>(b[i]))

```
即把int型的b[i]隐式转换成Array<int>型了。
每一次循环都把a的内容与一个大小为b[i]的临时数组（内容是未定义的）比较。这不仅不可能以正确的方法运行，而且还是效率低下的。因为每一次循环我们都必须建立和释放Array<int>对象。

有两个解决办法，
容易的方法是利用一个最新编译器的特性，explicit关键字。为了解决隐式类型转换而特别引入的这个特性，它的使用方法很好理解。构造函数用explicit声明，如果这样做，编译器会拒绝为了隐式类型转换而调用构造函数。显式类型转换依然合法。

```c++
template<class T>
class Array {
public:
	Array(int lowBound, int highBound);
	explicit Array(int size);    //注意使用explict关键字
	T& operator[](int index);
};

```
复杂的方法是使用proxy 类，代理类的作用就是支持其他对象的工作。
还是这个例子，

```c++
class Array 
	{
		public:
			class ArraySize  // 这个类是新的,代理类
			{ 
			public:
				ArraySize(int numElements) : theSize(numElements) {}
				int size() const { return theSize; }
			private:
				int theSize;
			};
			Array(int lowBound, int highBound);
			Array(ArraySize size); // 注意新的声明
	};

```
现在看使用：

```c++
Array<int>  a(10);
```
这样声明能否成功？发生了什么？
你的编译器要求用int参数调用Array<int>里的构造函数，但是没有这样的构造函数。编译器意识到它能从int参数转换成一个临时ArraySize对象，ArraySize对象只是Array<int>构造函数所需要的，这样编译器进行了转换。函数调用（及其后的对象建立）也就成功了。

```c++
if (a == b[i])
```
前面的这条语句能否成功？发生了什么？
为了调用operator＝＝函数，编译器要求Array<int>对象在”==”右侧,但是不存在一个参数为int的单参数构造函数。而且编译器无法把int转换成一个临时ArraySize对象然后通过这个临时对象建立必须的Array<int>对象，因为这将调用两个用户定义（user-defined）的类型转换，一个从int到ArraySize，一个从ArraySize到Array<int>。这种转换顺序被禁止的，所以当试图进行比较时编译器肯定会产生错误。

### 隐式类型转换运算符 ###
隐式类型转换运算符只是一个样子奇怪的成员函数：operator 关键字，其后跟一个类型符号。你不用定义函数的返回类型，因为返回类型就是这个函数的名字。
例子：
例如为了允许Rational(有理数)类隐式地转换为double类型（在用有理数进行混合类型运算时，可能有用），你可以如此声明Rational类：

```c++
class Rational {
public:	
	Rational(int realNum,int virtualNum);
	operator double() const; // 转换Rational类成
};

```
隐式调用double方法：

```c++
Rational r(1, 2); // r 的值是1/2
double d = 0.5 * r;
```
这种调用方式会引起令人恼火的灵异问题，看这个反例：

```c++
Rational r(1, 2);
std::cout << r;
```
假设忘记重载了<<操作符，期望的输出是有理数或报错。但现在编译器会会试图找到一个合适的隐式类型转换顺序以使得函数调用正常运行。编译器会发现它们能调用Rational::operator double函数来把r转换为double类型。所以上述代码打印的结果是一个浮点数，而不是一个有理数。

解决方法是用不使用语法关键字的等同的函数来替代转换运算符。就像c++  string类里面用c_str()函数替换把string隐式转换成char *。

## Item M6：自增(increment)、自减(decrement)操作符前缀形式与后缀形式的区别 ##

C++允许重载++ /-- 操作符，以前不管理解没理解，要求背牢的++i叫先增加后取值；i++叫先取值后增加。下面来看它的实现。

重载++/--时是怎么区分它们什么时候是前缀，什么时候是后缀的？实际上它们是通过参数类型上的差异进行区分。后缀形式有一个int类型参数，当函数被调用时，编译器传递一个0做为int参数的值给该函数。

```c++
class UPInt 
{ // "unlimited precision int"
public:
	UPInt& operator++(); // ++ 前缀
	const UPInt operator++(int); // ++ 后缀

	UPInt& operator--(); // -- 前缀
	const UPInt operator--(int); // -- 后缀

	UPInt& operator+=(int); // += 操作符
};

```
使用方法：

```c++
UPInt i;
++i; // 调用 i.operator++(); 
i++; // 调用 i.operator++(0); 
--i; // 调用 i.operator--(); 
i--; // 调用 i.operator--(0);
```
尤其要注意的是：这些操作符前缀与后缀形式返回值类型是不同的。前缀形式返回一个引用，后缀形式返回一个const类型。

仔细分析它们的实现：

```c++
// 前缀形式：增加然后取回值
UPInt& UPInt::operator++()
{
	*this += 1; // 增加
	return *this; // 取回值
}
// postfix form: fetch and increment
const UPInt UPInt::operator++(int)
{
	UPInt oldValue = *this; // 取回值
	++(*this); // 增加
	return oldValue; // 返回被取回的值
}

```
主要要注意两点：
1）、后缀increment必须返回一个对象（它返回的是增加前的值），但是为什么是const对象呢？

主要为了防止出现类似两次调用的代码：

```c++
UPInt i;
i++++; // 两次increment后缀

```
类比int，不支持连续的increment，而且取的值也和期望的不一样，所以用const类型加以避免。

2）、效率问题
后缀自增函数必须建立一个临时对象以做为它的返回值，上述实现代码建立了一个显示的临时对象（oldValue），这个临时对象必须被构造并在最后被析构，因而要尽量使用前缀形式。

一点思考：现代计算机，似乎这点效率问题可以忽略不计。

## Item M7：不要重载“&&”,“||”, 或“,”  ##

符号 "&&"、"||"、","总结起来有下面的两个特性：

1）支持短路求值法。
2）能保证计算顺序。

什么是逗号运算符？看下面条语句：

```c++
for (int i = 0, j = strlen(s) - 1; i < j; ++i, --j)
{
}

```
在for循环的最后一个部分里，i被增加同时j被减少。一个包含逗号的表达式首先计算逗号左边的表达式，然后计算逗号右边的表达式；整个表达式的结果是逗号右边表达式的值。所以在上述循环的最后部分里，编译器首先计算++i，然后是- -j，逗号表达式的结果是- -j。

以重载"&&"运算符为例，重载了表达式

```c++
if (expression1 && expression2)
```
实际上等同于下面的代码之一：

```c++
if (expression1.operator&&(expression2)) //--作为成员函数

if (operator&&(expression1, expression2)) //--作为全局函数

```
当表达式作为函数的参数时，两个参数都会去求值，因此不能模拟短路运算。而且，计算的顺序也是没法确定的，不能确保从左到右。

## Item M8：理解各种不同含义的new和delete ##

先来看看术语的文字游戏，new操作符（new operator）和new操作（operator new）的区别？我从汉语的角度更能理解，new 操作符是一个名词，可以理解为一个内置的关键字；而new 操作是一个动词，理解为一个运算符。

一点思考：这样也就解释了重载的问题，关键字不能重载，而运算符能重载。

在这条语句：

```c++
string *ps = new string("Memory Management");
```
中,new 是一个名词概念，实现了两部分的功能：第一部分是分配足够的内存以便容纳所需类型的对象。第二部分是它调用构造函数初始化内存中的对象。new操作符总是做这两件事情，你不能以任何方式改变它的行为。
但分配内存的过程是可以改变的。new操作符调用一个函数来完成必需的内存分配，你能够重写或重载这个函数来改变它的行为。new操作符为分配内存所调用函数的名字是operator new。
函数operator new 通常这样声明：

```c++
void * operator new(size_t size);
```
返回值类型是void*，参数size_t确定分配多少内存。
可以重载这个函数，包括改变返回值的类型和参数的个数。
与之对应的是delete操作符和delete操作。
delete操作符包括了调用析构函数和释放内存两部分。而delete操作（operator delete）用来释放内存。

同样的，对于数组操作，有operator new[]、operator delete[]的概念。

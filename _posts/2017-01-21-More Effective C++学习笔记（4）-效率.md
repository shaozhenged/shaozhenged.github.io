---
layout:     post
title:      "More Effective C++学习笔记（4）-效率"
date:       2017-01-21 12:00:00
author:     "邵正"
header-img: "img/home-bg-o.jpg"
tags:
    - C++
    - More effective c++
---


| 主题     | 概要                    |
| -------- | ----------------------- |
| C++      | More Effective C++ 效率 |
| -------- | ---                     |
| **编辑** | **时间**                |
| 新建     | 20170121                |
| -------- | ---                     |
| **序号** | **参考资料**            |
| 1        | More effective C++      |


## Item M16：牢记80－20准则（80－20 rule） ##
80－20准则说的是大约20％的代码使用了80％的程序资源；大约20%的代码耗用了大约80％的运行时间；大约20％的代码使用了80％的内存；大约20％的代码执行80％的磁盘访问；80％的维护投入于大约20％的代码上。

## Item M17：考虑使用lazy evaluation（懒惰计算法）  ##


## Item M18：分期摊还期望的计算  ##

不要被分期摊还这一高大上的字眼给迷惑了，其实我们在程序中不知不觉中已经使用过，只是没有这种理论深度。主要应用在需要频繁使用某个计算结果的场合。

### 缓存 ###
最简单也是最常用的方法是使caching(缓存)那些已经被计算出来而以后还有可能需要的值。我的经验是，这在与数据库打交道的程序中最为常用，通过map结构存放<key/value>对，如果key不在map中，才去访问数据库。如果在map中，就直接返回。

### 预提取 ###
Prefetching(预提取)是另一种方法。你可以把prefech想象成购买大批商品而获得的折扣。动态数组就是典型的例子，动态数组能够伸缩性的管理空间，当需要给数组赋值时，空间不够的时候，可能一次会扩展的空间是所需内存的两倍，这样减少分配的次数。

## Item M19：理解临时对象的来源 ##
这里的临时对象是指没有命名的非堆（non-heap）对象。这种未命名的对象通常在两种条件下产生：为了使函数成功调用而进行隐式类型转换和函数返回对象时。理解如何和为什么建立这些临时对象是很重要的，因为构造和释放它们的开销对于程序的性能来说有着不可忽视的影响。


### 使函数成功调用而建立临时对象 ###
当传送给函数的对象类型与参数类型不匹配时会产生这种情况。
例如一个函数，它用来计算一个字符在字符串中出现的次数：

```
// 返回ch在str中出现的次数
size_t countChar(const string& str, char ch);

```
在这里，输入的第一个参数是一个string类型的常量引用。然而，如果我们传递给它的参数是一个类似这样的buffer:

```
char buffer[MAX_STRING_LEN];
```
显然传入的参数类型与声明的参数类型不匹配，编译器会自动的消除这种不匹配。方法是建立一个string类型的临时对象。通过以buffer做为参数调用string的构造函数来初始化这个临时对象。countChar的参数str被绑定在这个临时的string对象上。当countChar返回时，临时对象自动释放。

这一过程很方便，但引起了不必要的开销。

需要注意的是，仅当通过传值（by value）方式传递对象或传递常量引用（reference-to-const）参数时，才会发生这些类型转换。当传递一个非常量引用（reference-to-non-const）参数对象，就不会发生。

比如这个函数：

```
void uppercasify(string& str); // 把str中所有的字符改变成大写
```
注意，这里的参数是非常量引用。同样，假设传递一个数组类型的buffer：

```
char subtleBookPlug[] = "Effective C++";
uppercasify(subtleBookPlug); // 错误!

```
在这里就不会发生自动转换。
为什么？假设建立一个临时对象，那么临时对象将被传递到upeercasify中，其会修改这个临时对象，把它的字符改成大写。但是对subtleBookPlug函数调用的真正参数没有任何影响；仅仅改变了临时从subtleBookPlug生成的string对象。

### 函数返回对象时 ###
例如operator+必须返回一个对象，以表示它的两个操作数的和。例如给定一个类型Number，这种类型的operator+被这样声明：

```
const Number operator+(const Number& lhs,const Number& rhs);
```
这个函数的返回值是临时的，因为它没有被命名；它只是函数的返回值。你必须为每次调用operator+构造和释放这个对象而付出代价。

综上所述，临时对象是有开销的，所以你应该尽可能地去除它们，然而更重要的是训练自己寻找可能建立临时对象的地方。在任何时候只要见到常量引用（reference-to-const）参数，就存在建立临时对象而绑定在参数上的可能性。在任何时候只要见到函数返回对象，就会有一个临时对象被建立（以后被释放）。


## Item M20：协助完成返回值优化 ##
一个返回对象的函数很难有较高的效率，因为传值返回会导致调用对象内的构造和析构
函数(参见条款M19)，这种调用是不能避免的。问题很简单：一个函数要么为了保证正确的行为而返回对象要么就不这么做。如果它返回了对象，就没有办法摆脱被返回的对象。

不要尝试着去用返回指针或返回引用的方法的方法来代替返回对象。

常见的错误：

```
const Rational& operator*(const Rational& lhs,
	const Rational& rhs)
{
	Rational result(lhs.numerator() * rhs.numerator(),
		lhs.denominator() * rhs.denominator());
	return result;//返回时，其指向的对象已经不存在了
}

```
正确的做法是是返回constructor argument而不是直接返回对象，你可以这样做：

```
// 一种高效和正确的方法，用来实现
// 返回对象的函数
const Rational operator*(const Rational& lhs,
	const Rational& rhs)
{
	return Rational(lhs.numerator() * rhs.numerator(),
		lhs.denominator() * rhs.denominator());
}

```
或者定义为内联函数 ：

```
inline const Rational operator*(const Rational& lhs,
	const Rational& rhs)
{
	return Rational(lhs.numerator() * rhs.numerator(),
		lhs.denominator() * rhs.denominator());
}

```

## Item M21：通过重载避免隐式类型转换 ##

假定有一个带有单参数构造函数的类和一个重载"+"符号：

```
class UPInt				// unlimited precision
{ 
public: // integers 类
	UPInt();
	UPInt(int value);
};

const UPInt operator+(const UPInt& lhs, const UPInt& rhs);

```
现在考虑下面这些语句：

```
UPInt upi1, upi2,upi3;
upi3 = upi1 + 10;
upi3 = 10 + upi2;
```
这些语句也能够成功运行。方法是通过建立临时对象把整形数10转换为UPInts。让编译器完成这种类型转换是确实是很方便，但是建立临时对象进行类型转换工作是有开销的，而我们不想承担这种开销。更好的实现方法是实现operator+混合类型的重载。

```
const UPInt operator+(const UPInt& lhs, const UPInt& rhs);// add UPInt and UPInt
	
const UPInt operator+(const UPInt& lhs, int rhs);// add UPInt and int
	 
const UPInt operator+(int lhs, const UPInt& rhs);// add int and  UPInt

```
但是不要像这样进行声明：

```
const UPInt operator+(int lhs, int rhs); // 错误!
```
在C++中有一条规则是每一个重载的operator必须带有一个用户定义类型（user-defined type）的参数。int不是用户定义类型，所以我们不能重载operator成为仅带有此[int]类型参数的函数。

## Item M22：考虑用运算符的赋值形式（op=）取代其单独形式（op） ##

大多数程序员认为如果他们能这样写代码：

```
x = x + y; x = x - y;
```
那他们也能这样写：

```
x += y; x -= y;
```
如果x和y是用户定义的类型（user-defined type），就不能确保这样。就C++来说，operator+、operator=和operator+=之间没有任何关系，因此如果你想让这三个operator同时存在并具有你所期望的关系，就必须自己实现它们。

通常的方法是operator+根据operator+=来实现：

```
template<class T>
const T operator+(const T& lhs, const T& rhs)
{
	return T(lhs) += rhs; 
}

template<class T>
const T operator-(const T& lhs, const T& rhs)
{
	return T(lhs) -= rhs; 
}

```

一点思考：如何实现operator+=？

```
T& operator+=(const T& rhs)
{
	*this=*this+rhs
	return *this;
}

```
这样似乎不行，会有先有鸡还是先有蛋的问题，只有老老实实的各个对应的成员变量相加。

## Item M23：考虑变更程序库  ##
应该选择最适合自己应用场景的程序库。


## Item M24：理解虚拟函数、多继承、虚基类和RTTI所需的代价 ##

 见另一篇博客《C++专题总结之理解虚拟函数、多继承、虚基类和RTTI》 

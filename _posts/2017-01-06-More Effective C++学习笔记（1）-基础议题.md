---
layout:     post
title:      "More Effective C++学习笔记（1）-基础议题"
date:       2017-01-06 12:00:00
author:     "邵正"
header-img: "img/home-bg-o.jpg"
tags:
    - C++
    - More effective c++
---

| 主题     | 概要                        |
| -------- | --------------------------- |
| C++      | More Effective C++ 基础议题 |
| -------- | ---                         |
| **编辑** | **时间**                    |
| 新建     | 20170106                    |
| -------- | ---                         |
| **序号** | **参考资料**                |
| 1        | More effective C++          |

## Item M1：指针与引用的区别##
指针与引用都是让你间接引用其他对象。你如何决定在什么时候使用指针，在什么时候使用引用呢？
首先，要认识到在任何情况下都不能使用指向空值的引用。一个引用必须总是指向某些对象。因此如果你使用一个变量并让它指向一个对象，但是该变量在某些时候也可能不指向任何对象，这时你应该把变量声明为指针，因为这样你可以赋空值给该变量。相反，如果变量肯定指向一个对象，例如你的设计不允许变量为空，这时你就可以把变量声明为引用。

引用必须被初始化：

```
void ItemM1_RefAndPtr_Base()
{
	string& rs; // 错误，引用必须被初始化
	string s("xyzzy");
	string& rs = s; // 正确，rs指向s
}

```
这段代码在编译阶段就会提示：error C2065: “rs”: 未声明的标识符；
不存在指向空值的引用这个事实意味着使用引用的代码效率比使用指针的要高。因为在使用引用之前不需要测试它的合法性。

```
void ItemM1_PrintDouble(const double& rd)
{
	cout << rd; // 不需要测试rd,它肯定指向一个double值

}

void ItemM1_PrintDouble(const double *pd)
{
	if (pd)
	{
		cout << *pd; // 检查是否为NULL
	}

}

```

指针与引用的另一个重要的不同是指针可以被重新赋值以指向另一个不同的对象。但是引用则总是指向在初始化时被指定的对象，以后不能改变。

```
void ItemM1_RefAndPtr_Base()
{
	string s1("Nancy");
	string s2("Clancy");
	string& rs = s1; // rs 引用 s1
	string *ps = &s1; // ps 指向 s1
	rs = s2; // rs 仍旧引用s1, 但是 s1的值现在是 "Clancy"
	ps = &s2; // ps 现在指向 s2;s1 没有改变
}

```

这里，rs,s1,s2值都变为"Clancy"。
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTA2MjMyMTM3NzA1)

总结：
一是你考虑到存在不指向任何对象的可能（在这种情况下，你能够设置指针为空），二是你需要能够在不同的时刻指向不同的对象（在这种情况下，你能改变指针的指向）。如果总是指向一个对象并且一旦指向一个对象后就不会改变指向，那么你应该使用引用。
还有一种情况，就是当你重载某个操作符时，你应该使用引用，比如操作符[]。

## Item M2：尽量使用C++风格的类型转换 ##

C风格的类型转换，一来它们过于粗鲁，能允许你在任何类型之间进行转换；二来C风格的类型转换在程序语句中难以识别。
C++通过引进四个新的类型转换操作符克服了C风格类型转换的缺点，这四个操作符是, static_cast, const_cast, dynamic_cast, 和reinterpret_cast。
在大多数情况下，对于这些操作符你只需要知道原来你习惯于这样写，
(type) expression
而现在你总应该这样写：
static_cast<type>(expression)。

### static_cast###

```
int firstNumber,secondNumber;
double result = ((double)firstNumber) / secondNumber;
double result = static_cast<double>(firstNumber) / secondNumber;

```
static_cast在功能上基本上与C风格的类型转换一样强大，含义也一样。它也有功能上限制。例如，你不能用static_cast象用C风格的类型转换一样把struct转换成int类型或者把double类型转换成指针类型，另外，static_cast不能从表达式中去除const属性，因为另一个新的类型转换操作符const_cast有这样的功能。

### const_cast###
const_cast用于类型转换掉表达式的const或volatileness属性。通过使用const_cast，你向人们和编译器强调你通过类型转换想做的只是改变一些东西的constness或者 volatileness属性。这个含义被编译器所约束。如果你试图使用const_cast来完成修改constness 或者volatileness属性之外的事情，你的类型转换将被拒绝。

```
class Widget { };
class SpecialWidget : public Widget {  };
void update(SpecialWidget *psw);

SpecialWidget sw; // sw 是一个非const 对象。
const SpecialWidget& csw = sw; // csw 是sw的一个引用,它是一个const 对象
update(&csw); // 错误!不能传递一个const SpecialWidget* 变量, 给一个处理SpecialWidget*类型变量的函数

```
上述错误在VS2015中，会报error C4430: 缺少类型说明符错误。

### dynamic_cast###
它被用于安全地沿着类的继承关系向下进行类型转换。
这就是说，你能用dynamic_cast把指向基类的指针或引用转换成指向其派生类或其兄弟类的指针或引用，而且你能知道转换是否成功。失败的转换将返回空指针（当对指针进行类型转换时）或者抛出异常（当对引用进行类型转换时）。
需要注意的是，dynamic_casts在帮助你浏览继承层次上是有限制的。它不能被用于缺乏虚函数的类型上（参见条款M24），也不能用它来转换掉constness：

```
void ItemM2_TypeCast_Base()
{
	int firstNumber,secondNumber;
	double result = dynamic_cast<double>(firstNumber) / secondNumber;
}

```
VS2015 会报，error C2680: “double”: dynamic_cast 的目标类型无效。因为int与double类型之间没有继承关系。
### reinterpret_cast###
使用这个操作符的类型转换，其的转换结果几乎都是执行期定义（implementation-defined）。因此，使用reinterpret_casts的代码很难移植。
reinterpret_casts的最普通的用途就是在函数指针类型之间进行转换。

例如，假设你有一个函数指针数组：

```
typedef void (*FuncPtr)();
FuncPtr funcPtrArray[10];

```
让我们假设你希望（因为某些莫名其妙的原因）把一个指向下面函数的指针存入funcPtrArray数组：

```
int doSomething();
```
你不能不经过类型转换而直接去做，因为doSomething函数对于funcPtrArray数组来说有一个错误的类型。在FuncPtrArray数组里的函数返回值是void类型，而doSomething函数返回值是int类型。

```
funcPtrArray[0] = &doSomething;  // 错误！类型不匹配
funcPtrArray[0] =reinterpret_cast<FuncPtr>(&doSomething);  //正确

```

## Item M3：不要对数组使用多态##
直接上例子：

```
class BST {  };
class BalancedBST : public BST {  };

void printBSTArray(ostream& s,const BST array[],int numElements)
{
	for (int i = 0; i < numElements; ) {
		s << array[i]; //假设BST类
	} //重载了操作符<<
}

BST BSTArray[10];
printBSTArray(cout, BSTArray, 10); // 运行正常

BalancedBST bBSTArray[10];
printBSTArray(cout, bBSTArray, 10); // 还会运行正常么？

```

这里有一个类BST（比如是搜索树对象）和继承自BST类的派生类BalancedBST，有一个函数，它能打印出BST类数组中每一个BST对象的内容。当你传递给该函数一个含有BST对象的数组变量时，它能够正常运行。然而，当你把含有BalancedBST对象的数组变量传递给printBSTArray函数时，会产生无法预期的错误。问题出在：

```
for (int i = 0; i < numElements; ) {
s << array[i];
}

```

这里array[i]数组中每个元素都是BST类型，因此每个元素与数组起始地址的间隔是i*sizeof(BST)。但当传递的是BalancedBST对象时，期望的大小是i*sizeof(BalancedBST)，没有人知道此时会发生什么错误。

一点思考：如果要实现类似数组来存放多态的功能，应该自己实现一个迭代器的功能。

## Item M4：避免无用的缺省构造函数  ##

这个条款主要讲是否要给类建一个缺省的构造函数。
缺省构造函数会用在哪些地方？如果没有缺省构造函数会碰到什么问题？
主要应用在3种情况：
1.建立对象数组，通常没有一种办法能在建立对象数组时给构造函数传递参数。所以在通常情况下，不可能建立EquipmentPiece对象数组：

```
EquipmentPiece bestPieces[10]; // 错误！没有正确调用 EquipmentPiece 构造函数
EquipmentPiece *bestPieces =new EquipmentPiece[10];  // 错误！与上面的问题一样

```
虽然可以使用3种方法（在非堆上初始化，利用指针数组来代替对象数组，使用operator new[]操作为数组分配raw memory）来避免这个问题，但它们都各有局限。

2.缺省构造函数所造成的第二个问题是它们无法在许多基于模板（template-based）的容器类里使用。
3.设计虚基类时所面临的要提供缺省构造函数还是不提供缺省构造函数的两难决策。如果不提供缺省构造函数，则所有子类都必须知道并理解提供给虚基类构造函数的参数的含义。派生类的作者是不会企盼和喜欢这种规定的。但如果强制提供缺省构造函数，就不再能确保对对象进行了有意义的初始化。因而，会引起更多低效的合法性检查。

一点思考：缺省构造函数有利有弊，不应该被滥用。



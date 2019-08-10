---
layout:     post
title:      "More Effective C++学习笔记（5）-技巧"
date:       2017-01-21 12:00:00
author:     "邵正"
header-img: "img/home-bg-o.jpg"
tags:
    - C++
    - More effective c++
---


| 主题     | 概要                    |
| -------- | ----------------------- |
| C++      | More Effective C++ 技巧 |
| -------- | ---                     |
| **编辑** | **时间**                |
| 新建     | 20170121                |
| -------- | ---                     |
| **序号** | **参考资料**            |
| 1        | More effective C++      |

前面书中讲了编程的指导准则，实际上段位还不够的时候，应该多掌握些实用的技巧。
## Item M25：将构造函数和非成员函数虚拟化  ##
构造函数的作用是什么？我的理解是根据初始化参数创建一个特定的对象。那怎么会有“虚拟构造函数”的概念呢？我回忆了一下，似乎脑中没有接受过“虚拟构造函数”的知识，“虚拟析构函数”到还常用。

原来这里的“虚拟构造函数”只是作者自己定义的一个概念，其实指的是一种函数，行为与构造函数相似，用来建立新对象，而且因为它能建立不同类型的对象，所以称它为虚拟构造函数。

有一种特殊种类的虚拟构造函数――虚拟拷贝构造函数――也有着广泛的用途。虚拟拷贝构造函数能返回一个指针，指向调用该函数的对象的新拷贝。

这个可以看书上的代码：

```c++
class NLComponent
{
public:
	// declaration of virtual copy constructor
	virtual NLComponent * clone() const = 0;	
};

class TextBlock : public NLComponent 
{
public:
	virtual TextBlock * clone() const // virtual copy
	{
		return new TextBlock(*this);
	} // constructor
};

class Graphic : public NLComponent 
{
public:
	virtual Graphic * clone() const // virtual copy
	{
		return new Graphic(*this);
	} // constructor
};

```
类的虚拟拷贝构造函数只是调用它们真正的拷贝构造函数。因此“拷贝”的含义与真正的拷贝构造函数相同。如果真正的拷贝构造函数只做了简单的拷贝，那么虚拟拷贝构造函数也做简单的拷贝。如果真正的拷贝构造函数做了全面的拷贝，那么虚拟拷贝构造函数也做全面的拷贝。

注意上述代码的实现利用了最近才被采纳的较宽松的虚拟函数返回值类型规则。被派生类重定义的虚拟函数不用必须与基类的虚拟函数具有一样的返回类型。如果函数的返回类型是一个指向基类的指针（或一个引用），那么派生类的函数可以返回一个指向基类的派生类的指针（或引用）。

把非成员函数虚拟化，这里也只是个宽泛的概念，指的是根据参数的不同动态类型而其行为特性也不同。
想像下面的代码：

```c++
class NLComponent 
{
public:
	virtual ostream& print(ostream& s) const = 0;	
};

class TextBlock : public NLComponent 
{
public:
	virtual ostream& print(ostream& s) const;
};

class Graphic : public NLComponent 
{
public:
	virtual ostream& print(ostream& s) const;
};

inline ostream& operator<<(ostream& s, const NLComponent& c)
{
	return c.print(s);
}

```
operator<< 根据传入的参数不同，而做出了不同的动作。使用成内联，可以减少调用的开销。

## Item M26：限制某个类所能产生的对象数量 ##
以前写过的代码最多就是用单例模式，倒没碰到过要刻意限制对象数量的情景。下面对限制建立n（n>=1）个对象数量的情况进行下总结。

### 使用友元非成员函数 ###
假定所有人都能访问打印机，但是只有一个打印机对象被建立。

```c++
class PrintJob; // forward 声明
				
class Printer 
{
public:
	void submitJob(const PrintJob& job);
	void reset();
	void performSelfTest();
	friend Printer& thePrinter();   //--友元非成员函数
private:
	Printer();
	Printer(const Printer& rhs);
};
Printer& thePrinter()
{
	static Printer p; // 单个打印机对象
	return p;
}

```
这个设计由三个部分组成，第一、Printer类的构造函数是private。这样能阻止建立对象。第二、全局函数thePrinter被声明为类的友元，让thePrinter避免私有构造函数引起的限制。最后thePrinter包含一个静态Printer对象，这意味着只有一个对象被建立。

### 使用静态成员函数 ###
这种混合了C风格与面向对象风格的代码应尽量消除，更好的方法是把打印功能放在类中，声明为一个静态函数。

```c++
class Printer
{
public:
	static Printer& thePrinter();
private:
	Printer();
	Printer(const Printer& rhs);
};
Printer& Printer::thePrinter()
{
	static Printer p;
	return p;
}

```
注意这个类里面存在的两个细微的地方。

1).唯一的Pritner对象是位于函数里的静态成员而不是在类中的静态成员
这样做是非常重要的。在类中的静态对象实际上总是被构造（和释放），即使不使用该对象。与此相反，只有第一次执行函数时，才会建立函数中的静态对象，所以如果没有调用函数，就不会建立对象。
另外，与一个函数的静态成员相比，把Printer声明为类中的静态成员还有一个缺点，它的初始化时间不确定。我们能够准确地知道函数的静态成员什么时候被初始化：“在第一次执行定义静态成员的函数时”。而没有定义一个类的静态成员被初始化的时间。C++为一个translation unit（也就是生成一个object文件的源代码的集合）内的静态成员的初始化顺序提供某种保证，但是对于在不同translation unit中的静态成员的初始化顺序则没有这种保证。

一点思考：什么是translation unit？

    According to standard C++ (wayback machine link) : A translation unit is the basic unit of compilation in C++. It consists of the contents of a single source file, plus the contents of any header files directly or indirectly included by it, minus those lines that were ignored using conditional preprocessing statements.
    A single translation unit can be compiled into an object file, library, or executable program.
    The notion of a translation unit is most often mentioned in the contexts of the One Definition Rule, and templates.

根据这个概念，我的理解是一个类，如果被编译在不同的对象文件中，就不能保证它的初始化顺序。

2). 内联与函数内静态对象的关系
再看一下thePrinter的非成员函数形式：

```c++
Printer& thePrinter()
{
	static Printer p;
	return p;
}

```
除了第一次执行这个函数时（也就是构造p时），其它时候这就是一个一行函数——它由“return p;”一条语句组成。这个函数最适合做为内联函数使用。然而它不能被声明为内联。
为什么呢？请想一想，为什么你要把对象声明为静态呢？通常是因为你只想要该对象的一个拷贝。现在再考虑“内联”意味着什么呢？从概念上讲，它意味着编译器用函数体替代该对函数的每一个调用，不过非成员函数还不只这些。非成员函数还有其它的含义。它还意味着internal linkage（内部链接）。
因此，你只需记住一件事：“带有内部链接的函数可能在程序内被复制（也就是说程序的目标（object）代码可能包含一个以上的内部链接函数的代码），这种复制也包括函数内的静态对象。”结果如何？如果建立一个包含局部静态对象的非成员函数，你可能会使程序的静态对象的拷贝超过一个！


### 限制对象数量 ###
如果想要限制建立对象的数量，一种很正常的想法是计算对象的数量：

```c++
class Printer {
public:
	class TooManyObjects {}; // 当需要的对象过多时
							 // 就使用这个异常类
	Printer();
	~Printer();

private:
	static size_t numObjects;
	Printer(const Printer& rhs); // 这里只能有一个printer，所以不允许拷贝
};

size_t Printer::numObjects = 0;
Printer::Printer()
{
	if (numObjects >= 1) 
	{
		throw TooManyObjects();
	}
	继续运行正常的构造函数;
	++numObjects;
}
Printer::~Printer()
{
	进行正常的析构函数处理;
	--numObjects;
}

```
此法的核心思想就是使用numObjects跟踪Pritner对象存在的数量。当构造类时，它的值就增加，释放类时，它的值就减少。如果试图构造过多的Printer对象，就会抛出一个TooManyObjects类型的异常。这种限制建立对象数目的方法有两个较吸引人的优点。一个是它是直观的，每个人都能理解它的用途。另一个是很容易推广它的用途，可以允许建立对象最多的数量不是一，而是其它大于一的数字。
但是也会存在像下面的问题：
1). 建立对象的环境
假设新定义了一个彩色打印机，让它继承自普通打印机：

```c++
class ColorPrinter : public Printer 
{
	...
};
```
现在假设我们系统有一个普通打印机和一个彩色打印机：

```c++
Printer p;
ColorPrinter cp;

```
这两个定义会产生多少Pritner对象？答案是两个：一个是p，一个是cp。在运行时，当构造cp的基类部分时，会抛出TooManyObjects异常。对于许多程序员来说，这可不是他们所期望的事情。

当其它对象包含Printer对象时，会发生同样的问题：

```c++
class CPFMachine { // 一种机器，可以复印，打印
private: // 发传真。
	Printer p; // 有打印能力
	FaxMachine f; // 有传真能力
	CopyMachine c; // 有复印能力
	...
};
CPFMachine m1; // 运行正常
CPFMachine m2; // 抛出 TooManyObjects异常

```
问题是Printer对象能存在于三种不同的环境中：只有它们本身；作为其它派生类的基类；被嵌入在更大的对象里。存在这些不同环境极大地混淆了跟踪“存在对象的数目”的含义，因为你心目中的“对象的存在” 的含义与编译器不一致。

解决办法是把构造函数私有，定义一个公有函数来伪造构造函数的功能。

```c++
class FSA 
{
public:
	// 伪构造函数
	static FSA * makeFSA();
	static FSA * makeFSA(const FSA& rhs);
	...
private:
	FSA();
	FSA(const FSA& rhs);
	...
};
FSA * FSA::makeFSA()
{
	return new FSA();
}
FSA * FSA::makeFSA(const FSA& rhs)
{
	return new FSA(rhs);
}

```
不象thePrinter函数总是返回一个对象的引用（引用的对象是固定的），每个makeFSA的伪构造函数则是返回一个指向对象的指针（指向的对象都是惟一的，不相同的）。也就是说允许建立的FSA对象数量没有限制。不过每个伪构造函数都调用new这个事实暗示调用者必须记住调用delete。遇到这种情况，要时刻记住需要优先使用auto_ptr。

2). 允许对象来去自由

什么是对象的来去自由？我想就是能够自由的建立和释放。

这一节没能彻底理解，待后面回头补上。


### 一个具有对象计数功能的基类 ###

。。。

## Item M27：要求或禁止在堆中产生对象  ##
这个条款的内容比较难，

### 要求在堆中建立对象 ###
如果要一个对象必须在堆中建立，即禁止new 操作以外的手段建立对象，防止被自动构造和自动释放，最简单的办法是禁止使用隐式的构造函数和析构函数，比如把构造函数和析构函数声明为private。

如果两者都声明为私有，会增加工作量，要定义自己的伪构造函数和伪析构函数，更好的办法是让析构函数成为private，让构造函数成为public。引进一个专用的伪析构函数，用来访问真正的析构函数。客户端调用伪析构函数释放他们建立的对象。

看这个例子：如果我们想仅仅在堆中建立代表unlimited precision numbers（无限精确度数字）的对象，可以这样做：

```c++
class UPNumber 
{
public:
	UPNumber();
	UPNumber(int initValue);
	UPNumber(double initValue);
	UPNumber(const UPNumber& rhs);
	// 伪析构函数 (一个const 成员函数， 因为
	// 即使是const对象也能被释放。)
	void destroy() const { delete this; }
	...
private:
	~UPNumber();
};

```
而使用方法是：

```c++
UPNumber *p = new UPNumber;
p->destroy();

```
这里显示的调用destroy()伪析构函数进行释放资源。而如果尝试像类似下面的调用，

```c++
UPNumber n; // 错误! (在这里合法， 但是当它的析构函数被隐式地调用时，就不合法了)
UPNumber *p = new UPNumber; //正确
...
delete p; // 错误! 试图调用private 析构函数

```
特别是语句：
UPNumber  n;
构造的时候没问题，但当它离开了自己的作用域，会隐式调用析构函数，就会报错。实际上VS在编译阶段就能诊测到这个错误：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIxMjAzODQ4OTM3)


### 判断一个对象是否在堆中 ###

现实项目中，似乎很少会用到需要判断一个对象是否在堆中的情况。这一节主要用来进行理论知识方面的学习，主要的路线其实比较清晰，就是不断的提出想法，然后再否定想法。

构造函数没法区分这两种情况：

```c++
NonNegativeUPNumber *n1 =new NonNegativeUPNumber; // 在堆中
NonNegativeUPNumber n2; //不再堆中

```

如果想要区分，有什么好的方法？

#### 假设使用new操作符判断 ####

```c++
class UPNumber 
{
public:
	// 如果建立一个非堆对象，抛出一个异常
	class HeapConstraintViolation {};
	static void * operator new(size_t size);
	UPNumber();
	...
private:
	static bool onTheHeap; //在构造函数内，指示对象是否被构造在堆上
};
// obligatory definition of class static
bool UPNumber::onTheHeap = false;

//--注意这里：当调用了new操作，则认为是在堆中
void *UPNumber::operator new(size_t size)
{
	onTheHeap = true;
	return ::operator new(size);
}
UPNumber::UPNumber()
{
	if (!onTheHeap) {
		throw HeapConstraintViolation();
	}
	proceed with normal construction here;
	onTheHeap = false; // 为下一个对象清除标记
}

```
这种方法利用了这样一个事实：“当在堆上分配对象时，会调用operator new来分配raw memory”。只有onTheHeap为真时，才能正确的调用构造函数。

但是考虑这样一句代码：

```c++
UPNumber *numberArray = new UPNumber[100];
```
会存在两个问题：
第一，这里调用的operator new[] 操作，所以onTheHeap不会为真，可以重载new []操作来解决；
第二：这里会调用100次构造函数，但是只在最开始的时候分配了一次内存，所以只有第一次调用构造函数前把onTheHeap设置为true。当调用第二个构造函数时，会抛出一个异常。

因此，这种方法不可行，开始寻找新方法。

#### 假设通过程序的地址空间判断 ####

忽略程序的移植性，例设利用一个在很多系统上存在的事实，程序的地址空间被做为线性地址管理，程序的栈从地址空间的顶部向下扩展，堆则从底部向上扩展：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIxMjA0MTE4ODQ2)

你可能会想能够使用下面这个函数来判断某个特定的地址是否在堆中：

```c++
// 不正确的尝试，来判断一个地址是否在堆中
bool onHeap(const void *address)
{
	char onTheStack; // 局部栈变量
	return address < &onTheStack;
}

```
这个函数背后的思想很有趣。在onHeap函数中onTheSatck是一个局部变量。因此它在堆栈上。当调用onHeap时，它的栈框架（stack frame）(也就是它的activation record)被放在程序栈的顶端，因为栈在结构上是向下扩展的（趋向低地址），onTheStack的地址肯定比任何栈中的变量或对象的地址小。如果参数address的地址小于onTheStack的地址，它就不会在栈上，而是肯定在堆上。

这种方法最根本的问题是对象可以被分配在三个地方，而不是两个。
，栈和堆能够容纳对象，但是我们忘了静态对象。静态对象是那些在程序运行时仅能初始化一次的对象。静态对象不仅仅包括显示地声明为static的对象，也包括在全局和命名空间里的对象。这些对象肯定位于某些地方，而这些地方既不是栈也不是堆。

它们的位置是依据系统而定的，但是在很多栈和堆相向扩展的系统里，它们位于堆的底端。
加上静态变量后，最新的地址空间图片如下所示：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIxMjA0MjA1MDY1)

onHeap不能工作的原因立刻变得很清楚了，它不能辨别堆对象与静态对象的区别。

#### 假设通过HeapTracked基类判断 ####
基本思路是使用一个list存放所有通过new操作得到的地址，判断函数通过遍历当前的地址是否在这个list中，如果在，则认为是存在于堆中。

下面是这个HeapTracked基类的全部实现：

```c++
class HeapTracked { // 混合类; 跟踪从operator new返回的ptr
public: 
	class MissingAddress {}; // 异常类，见下面代码
	virtual ~HeapTracked() = 0;
	static void *operator new(size_t size);
	static void operator delete(void *ptr);
	bool isOnHeap() const;
private:
	typedef const void* RawAddress;
	static list<RawAddress> addresses;
};

// mandatory definition of static class member
list<RawAddress> HeapTracked::addresses;

// HeapTracked的析构函数是纯虚函数，使得该类变为抽象类。然而析构函数必须被定义，
//所以我们做了一个空定义。.
HeapTracked::~HeapTracked() {}
void * HeapTracked::operator new(size_t size)
{
	void *memPtr = ::operator new(size);	// 获得内存
	addresses.push_front(memPtr);			// 把地址放到list的前端
	return memPtr;
}
void HeapTracked::operator delete(void *ptr)
{
	//得到一个 "iterator"，用来识别list元素包含的ptr；	
	list<RawAddress>::iterator it =	find(addresses.begin(), addresses.end(), ptr);
	if (it != addresses.end()) {	// 如果发现一个元素
		addresses.erase(it);		//则删除该元素
		::operator delete(ptr);		// 释放内存
	}
	else {							// 否则
		throw MissingAddress();		// ptr就不是用operator new
	} // 分配的，所以抛出一个异常
}
bool HeapTracked::isOnHeap() const
{
	// 得到一个指针，指向*this占据的内存空间的起始处，
	// 有关细节参见下面的讨论
	const void *rawAddress = dynamic_cast<const void*>(this);
	// 在operator new返回的地址list中查到指针
	list<RawAddress>::iterator it =
		find(addresses.begin(), addresses.end(), rawAddress);
	return it != addresses.end(); // 返回it是否被找到
}

```
注意这条语句：

```c++
const void *rawAddress = dynamic_cast<const void*>(this);
```
这里的this实际上可能是指向的派生类，通过dynamic_cast转换成const void*,并且将指针指向“原指针指向对象内存”的开始处。但是要注意dynamic_cast只能用于“指向至少具有一个虚拟函数的对象”的指针上。
如何使用这个类？
例如我们想判断Assert对象指针指向的是否是堆对象：

```c++
class Asset : public HeapTracked 
{
private:
	UPNumber value;
	...
};

```
可以通过一个非成员函数：

```c++
void inventoryAsset(const Asset *ap)
{
	if (ap->isOnHeap()) 
	{
		ap is a heap - based asset — inventory it as such;
	}
	else 
	{
		ap is a non - heap - based asset — record it that way;
	}
}

```

### 禁止堆对象 ###
可能有三种情况来建立对象：
1）、对象被直接实例化，很好理解，自己定义一个对象，会显示或隐式调用构造函数。
2）、对象做为派生类的基类被实例化，当派生类被实例化时，基类会自动被实例化。
3）、对象被嵌入到其它对象内，也会自动调用隐式构造函数。

下面看如何在这三种情况下禁止建立堆对象。
#### 禁止用户直接实例化对象 ####
比较简单，把new和delete操作定义为私有：

```c++
class UPNumber 
{
private:
	static void *operator new(size_t size);
	static void operator delete(void *ptr);
};

```
现在用户仅仅可以做允许它们做的事情：

```c++
UPNumber n1; // okay
static UPNumber n2; // also okay
UPNumber *p = new UPNumber; // error! attempt to call private operator new

```

#### 禁止做为派生类的基类被实例化 ####
如果new、delete操作在基类中被声明为私有，而在派生类中没有对其进行改写(overwrite)，则基类和派生类都不能被实例化，因为operator new和operator delete是自动继承的。

```c++
class UPNumber { ... }; // 同上
class NonNegativeUPNumber : public UPNumber//假设这个类没有声明operator new
{		
	...
};

NonNegativeUPNumber n1; // 正确
static NonNegativeUPNumber n2; // 也正确
NonNegativeUPNumber *p = new NonNegativeUPNumber;// 错误! 试图调用private operator new

```

#### 对象被嵌入到其它对象 ####
UPNumber的operator new是private这一点，不会对包含UPNumber成员对象的对象的分配产生任何影响：

```c++
class Asset {
public:
	Asset(int initValue);
	...
private:
	UPNumber value;
};
Asset *pa = new Asset(100); // 正确, 调用Asset::operator new 或						
					 // ::operator new, 不是UPNumber::operator new

```



## Item M28：灵巧（smart）指针 ##

灵巧指针是一种外观和行为都被设计成与内建指针相类似的对象，不过它能提供更多的功能。
当你使用灵巧指针替代C++的内建指针（也就是dumb pointer）,你就能控制下面这些方面的指针的行为：
构造和析构，你可以决定建立灵巧指针时应该怎么做；
拷贝和赋值，你能对拷贝灵巧指针或有灵巧指针参与的赋值操作进行控制；
提领（Dereferencing），取出指针所指东西的内容。当用户引用被灵巧指针所指的对象，会发生什么事情呢？你可以自行决定。

灵巧指针从模板中生成，因为要与内建指针类似，必须是strongly typed(强类型)的；模板参数确定指向对象的类型。大多数灵巧指针模板看起来都象这样：

```c++
template<class T>
class SmartPtr 
{
public:
	SmartPtr(T* realPtr = 0);					// 建立一个灵巧指针, 指向dumb pointer所指的对象。未初始化的指针缺省值为0(null)
							 
	SmartPtr(const SmartPtr& rhs);				// 拷贝一个灵巧指针
	~SmartPtr();								// 释放灵巧指针
				
	SmartPtr& operator=(const SmartPtr& rhs);	// make an assignment to a smart ptr
	T* operator->() const;						// dereference一个灵巧指针，以访问所指对象的成员						  
	T& operator*() const;						// dereference 灵巧指针
private:
	T *pointee;									// 灵巧指针所指的对象
};

```
有一个私有模版成员，是一个指向T对象的dumb pointer。包括了公有的构造、析构、赋值、提领操作。下面分别对这几个操作的用法进行总结：


### 灵巧指针的构造、赋值和析构 ###
灵巧指针的的构造通常很简单：找到指向的对象（一般由灵巧指针构造函数的参数给出），让灵巧指针的内部成员dumb pointer指向它。如果没有找到对象，把内部指针设为0或发出一个错误信号（可以是抛出一个异常）。
灵巧指针拷贝构造函数、赋值操作符函数和析构函数的实现由于（所指对象的）所有权的问题所以有些复杂。如果一个灵巧指针拥有它指向的对象，当它被释放时必须负责删除这个对象。

为了避免出现下面这种删除两次对象的情况，灵巧指针的拷贝和赋值操作会通过所有权转移来避免。

这是一个C++的简易的auto_ptr模板：

```c++
template<class T>
class auto_ptr {
public:
	auto_ptr(T *ptr = 0) : pointee(ptr) {}
	~auto_ptr() { delete pointee; }
	...
private:
	T *pointee;
};

```
假如auto_ptr拥有对象时，它可以正常运行。但是当auto_ptr被拷贝或被赋值时，会发生什么情况呢？

```c++
auto_ptr<TreeNode> ptn1(new TreeNode);
auto_ptr<TreeNode> ptn2 = ptn1;		// 调用拷贝构造函数
									//会发生什么情况？
auto_ptr<TreeNode> ptn3;
ptn3 = ptn2;						// 调用 operator=;
									// 会发生什么情况?

```
这里可能会导致两个auto_ptr指向一个相同的对象。这是一个灾难，因为当释放quto_ptr时每个auto_ptr都会删除它们所指的对象。这意味着一个对象会被我们删除两次。这种两次删除的结果将是不可预测的。

为了解决这个问题，使用“当auto_ptr被拷贝和赋值时，对象所有权随之被传递”的方法。

```c++
template<class T>
class auto_ptr 
{
public:
	...
	auto_ptr(auto_ptr<T>& rhs);			// 拷贝构造函数
	auto_ptr<T>&						// 赋值
	operator=(auto_ptr<T>& rhs);		// 操作符
	...
};
template<class T>
auto_ptr<T>::auto_ptr(auto_ptr<T>& rhs)
{
	pointee = rhs.pointee;		// 把*pointee的所有权传递到 *this						   
	rhs.pointee = 0;			// rhs不再拥有任何东西
} 

template<class T>
auto_ptr<T>& auto_ptr<T>::operator=(auto_ptr<T>& rhs)
{
	if (this == &rhs)		// 如果这个对象自我赋值
		return *this;		// 什么也不要做

	delete pointee;			// 删除现在拥有的对象
	pointee = rhs.pointee;	// 把*pointee的所有权
	rhs.pointee = 0;		// 从 rhs 传递到 *this
	return *this;
}

```
注释已经很好的解释了上面的代码，只是在使用的时候，需要注意不要用传值的方法传递auto_ptr对象。

比如：

```c++
// 这个函数通常会导致灾难发生
void printTreeNode(ostream& s, auto_ptr<TreeNode> p)
{
	s << *p;
}
int main()
{
	auto_ptr<TreeNode> ptn(new TreeNode);
	...
	printTreeNode(cout, ptn); //通过传值方式传递auto_ptr
	...
}

```
当printTreeNode的参数p被初始化时（调用auto_ptr的拷贝构造函数），ptn指向对象的所有权被传递到给了p。当printTreeNode结束执行后，p离开了作用域，它的析构函数删除它指向的对象（就是原来ptr指向的对象）。然而ptr已不再指向任何对象（它的dumb pointer是null），所以调用printTreeNode以后任何试图使用它的操作都将产生未定义的行为。

通常，使用引用传递来代替值传递。

```c++
// 这个函数的行为更直观一些
void printTreeNode(ostream& s,const auto_ptr<TreeNode>& p)
{
	s << *p;
}

```
在函数里，p是一个引用，而不是一个对象，所以不会调用拷贝构造函数初始化p。当ptn被传递到上面这个printTreeNode时，它还保留着所指对象的所有权，调用printTreeNode以后还可以安全地使用ptn。

灵巧指针的析构函数通常是这样的：

```c++
template<class T>
SmartPtr<T>::~SmartPtr()
{
	if (*this owns *pointee) 
	{
		delete pointee;
	}
}

```
这里的if判断，主要用在使用了引用计数时，灵巧指针必须判断是否有权删除所指对象。

### 实现Dereference 操作符 ###

让我们把注意力转向灵巧指针的核心部分，operator*和operator-> 函数。前者返回所
指的对象。理论上，这很简单：

```c++
template<class T>
T& SmartPtr<T>::operator*() const
{
	perform "smart pointer" processing;
	return *pointee;
}

```
注意，注意返回类型是一个引用。如果返回对象，尽管编译器允许这么做，却可能导致灾难性后果。
为什么不能返回对象？
因为，pointee不用必须指向T类型对象；它也可以指向T的派生类对象。如果在这种情况下operator*函数返回的是T类型对象而不是派生类对象的引用，你的函数实际上返回的是一个错误类型的对象。
在返回的这种对象上调用虚拟函数，不会触发与（原先）所指对象的动态类型相符的函数。实际上就是说你的灵巧指针将不能支持虚拟函数，象这样的指针再灵巧也没有用。

operator->的情况与operator*是相同的，考虑像下面类似的语句：

```c++
void editTuple(DBPtr<Tuple>& pt)
{
	LogEntry<Tuple> entry(*pt);
	do 
	{
		pt->displayEditDialog();
	} while (pt->isValid() == false);
}

```
语句：

```c++
pt->displayEditDialog();
```
被编译器解释为：

```c++
(pt.operator->())->displayEditDialog();
```
这意味着不论operator->返回什么，它必须在返回结果上使用member-selection operator(成员选择操作符)（->）。因此operator->仅能返回两种东西：一个指向某对象的dumb pointer或另一个灵巧指针。通常情况下，直接返回一个普通dumb pointer。

```c++
template<class T>
T* SmartPtr<T>::operator->() const
{
	perform "smart pointer" processing;
	return pointee;
}

```
灵巧指针的构造、赋值、析构、提领是最基本的东西，下面总结关于它的更深入的主题。

### 测试灵巧指针是否为NULL ###
怎么测试一个灵巧指针是否为空？下面的语句对不对？

```c++
SmartPtr<TreeNode> ptn;
...
if (ptn == 0) ...	// error!
if (ptn) ...		// error!
if (!ptn) ...		// error!

```
我们这里要判断的实际上是灵巧指针的这个dumb pointer成员为空，而不是ptn这个对象为空，一种方法是写显示的isNull函数；一种是提供隐式类型转换操作符，如果dumb pointer成员为空，就把ptn转换成void*。

```c++
template<class T>
class SmartPtr 
{
public:
	...
	operator void*();		// 如果灵巧指针为null，返回0， 否则返回非0
	... 
}; 
SmartPtr<TreeNode> ptn;
...
if (ptn == 0) ...			// 现在正确
if (ptn) ...				// 也正确
if (!ptn) ...				// 正确

```
但是像条款M5提到的，这种隐式转换经常会碰到头疼的灵异问题。

```c++
SmartPtr<Apple> pa;
SmartPtr<Orange> po;
...
if (pa == po) ... // 这能够被成功编译!

```
即使在SmartPtr<Apple> 和 SmartPtr<Orange>之间没有operator= 函数，也能够编译，因为灵巧指针被隐式地转换为void*指针。

所以要慎用void *隐式转换。

### 把灵巧指针转变成dumb指针 ###
如果原来函数的原型，参数是一个dumb指针，现在以灵巧指针作为参数调用，会发生什么情况？
比如，原来函数的原型是这样：

```c++
class Tuple { ... }; // 同上
void normalize(Tuple *pt); // 注意使用的是dumb指针

```
现在试图用指向Tuple的灵巧指针作参数调用normalize：

```c++
DBPtr<Tuple> pt;
...
normalize(pt); // 错误!

```
这种调用不能够编译，因为不能把DBPtr<Tuple>转换成Tuple*。你可以这样做，从而使该函数正常运行：

```c++
normalize(&*pt); // 繁琐, 但合法
```
可能会想到用隐式的类型转换：

```c++
template<class T> // 同上
class DBPtr 
{
public:
	...
	operator T*() { return pointee; }
	...
};

```
现在看起来很美好，能够直接这样调用：

```c++
DBPtr<Tuple> pt;
...
normalize(pt); // 能够运行

```
也能满足测试空值时的语法调用：

```c++
if (pt == 0) ...	// 正确, 把pt转变成Tuple*				
if (pt) ...		// 同上
if (!pt) ...		// 同上 (reprise)

```
但是它有类型转换函数所具有的缺点：
1）、使用户能够直接访问dump指针

```c++
void processTuple(DBPtr<Tuple>& pt)
{
	Tuple *rawTuplePtr = pt; // 把DBPtr<Tuple> 转变成
							 // Tuple*
	使用raw TuplePtr 修改 tuple;
}

```
2）、灵巧指针不能进行连续的转换
因为从灵巧指针到dumb指针的转换是“用户定义类型转换”，在同一时间编译器进行这种转换的次数不能超过一次。
还是看例子，有一个TupleAccessors类，使用的单参构造函数类型是dump指针：

```c++
class TupleAccessors 
{
public:
	TupleAccessors(const Tuple *pt); 
	... 
};

```
现在有一个使用TupleAccessors对象作为参数的函数：

```c++
TupleAccessors merge(const TupleAccessors& ta1,const TupleAccessors& ta2);
```

如果直接使用Tuple *作为参数，能够成功调用：

```c++
Tuple *pt1, *pt2;
...
merge(pt1, pt2); 		// 正确, 两个指针被转换为
				 	// TupleAccessors objects

```
如果用灵巧指针DBPtr<Tuple>进行调用，编译就会失败：

```c++
DBPtr<Tuple> pt1, pt2;
...
merge(pt1, pt2); 		// 错误!不能把 pt1 和
					// pt2转换称TupleAccessors对象

```
因为从DBPtr<Tuple>到TupleAccessors的转换要调用两次用户定义类型转换（一次从DBPtr<Tuple>到Tuple*，一次从Tuple*到TupleAccessors），编译器不会进行这种序列的转换。

3）、隐藏极深的Bug

考虑这段代码：

```c++
DBPtr<Tuple> pt = new Tuple;
...
delete pt;

```
这段代码应该不能被编译，pt不是指针，它是一个对象，你不能删除一个对象。只有指针才能被删除，对么？
但是现成却能编译通过，因为pt隐式转换为Tuple*，然后删除它。
存在这么多问题，那底线很简单：除非有一个让人非常信服的原因去这样做，否则绝对不要提供转换到dumb指针的隐式类型转换操作符。


### 灵巧指针和继承类到基类的类型转换 ###
先说结论：灵巧指针不能继承。
再来看例子：
假设我们有一个public继承层次结构，以模型化音乐商店的商品：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIxMjEwMTIyMjMw)
下面是简易代码：

```c++
class MusicProduct 
{
public:
	MusicProduct(const string& title);
	virtual void play() const = 0;
	virtual void displayTitle() const = 0;
	...
};
class Cassette : public MusicProduct 
{
public:
	Cassette(const string& title);
	virtual void play() const;
	virtual void displayTitle() const;
	...
};

class CD : public MusicProduct 
{
public:
	CD(const string& title);
	virtual void play() const;
	virtual void displayTitle() const;
	...
};

```
有一个典型的多态函数：

```c++
void displayAndPlay(const MusicProduct* pmp, int numTimes)
{
	for (int i = 1; i <= numTimes; ++i) 
	{
		pmp->displayTitle();
		pmp->play();
	}
}

```
正常的使用情况是：

```c++
Cassette *funMusic = new Cassette("Alapalooza");
CD *nightmareMusic = new CD("Disco Hits of the 70s");
displayAndPlay(funMusic, 10);
displayAndPlay(nightmareMusic, 0);

```
但是当我们用灵巧指针替代替dumb指针，会发生什么呢？
函数变为：

```c++
void displayAndPlay(const SmartPtr<MusicProduct>& pmp,int numTimes);
```
这样调用：

```c++
SmartPtr<Cassette> funMusic(new Cassette("Alapalooza"));
SmartPtr<CD> nightmareMusic(new CD("Disco Hits of the 70s"));
displayAndPlay(funMusic, 10); // 错误!
displayAndPlay(nightmareMusic, 0); // 错误!

```
不能进行编译的原因是不能把SmartPtr<CD>或SmartPtr<Cassette>转换成SmartPtr<MusicProduct>。从编译器的观点来看，这些类之间没有任何关系。
毕竟SmartPtr<CD> 或 SmartPtr<Cassette>不是从SmartPtr<MusicProduct>继承过来的，这些类之间没有继承关系，我们不可能要求编译器把一种对象转换成（完全不同的）另一种类型的对象。

可能想到的是在每一个派生类里，实现一个隐式类型转换操作符，类似这样：

```c++
operator SmartPtr<MusicProduct>()
{
	return SmartPtr<MusicProduct>(pointee);
}

```
但是这样做，破坏了模版的通用性，也有大量的重复代码。
幸运的是，可以用成员函数模版来实现:

```c++
template<class T>				// 模板类，指向T的
class SmartPtr					// 灵巧指针
{ 
public:
	SmartPtr(T* realPtr = 0);
	T* operator->() const;
	T& operator*() const;
	template<class newType>		// 模板成员函数
	operator SmartPtr<newType>() // 为了实现隐式类型转换.
	{
		return SmartPtr<newType>(pointee);
	}
	...
};

```
如果用新的灵巧指针，则下面的调用将不会是一个错误：

```c++
SmartPtr<Cassette> funMusic(new Cassette("Alapalooza"));
SmartPtr<CD> nightmareMusic(new CD("Disco Hits of the 70s"));
displayAndPlay(funMusic, 10); 
displayAndPlay(nightmareMusic, 0); 

```
拿这句代码来说：

```c++
displayAndPlay(funMusic, 10);
```
funMusic对象的类型是SmartPtr<Cassette>。函数displayAndPlay期望的参数是SmartPtr<MusicProduct>地对象。编译器侦测到类型不匹配，于是寻找把funMusic转换成SmartPtr<MusicProduct>对象的方法。它在SmartPtr<MusicProduct>类里寻找带有SmartPtr<Cassette>类型参数的单参数构造函数（参见条款M5），但是没有找到。然后它们又寻找成员函数模板，以实例化产生这样的函数。它们在SmartPtr<Cassette>发现了模板，把newType绑定到MusicProduct上，生成了所需的函数。实例化函数，生成这样的代码：

```c++
SmartPtr<Cassette>:: operator SmartPtr<MusicProduct>()
{
	return SmartPtr<MusicProduct>(pointee);
}

```
但是使用这种成员函数模板也并不是万能的，会存在二义性的情况。考虑下面的情况：
假设我们用一个新类CasSingle来扩充MusicProduct类层次，用来表示cassette singles。修改后的类层次看起来象这样：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIxMjEwNDIxMjgz)

现在考虑这段代码：

```c++
template<class T>		// 同上, 包括作为类型
class SmartPtr { ... }; // 转换操作符的成员模板
void displayAndPlay(const SmartPtr<MusicProduct>& pmp,int howMany);
void displayAndPlay(const SmartPtr<Cassette>& pc,int howMany);
SmartPtr<CasSingle> dumbMusic(new CasSingle("Achy Breaky Heart"));
displayAndPlay(dumbMusic, 1);

```
这里displayAndPlay 函数被Overload(重载)了两次，那当调用这个语句的时候：
displayAndPlay(dumbMusic, 1);
出现了二义性，因为编译器不知道该把SmartPtr<CasSingle>转换为SmartPtr<Cassette>还是SmartPtr<MusicProduct>，它们具有同样的优先级。

通过什么手段消除这种二义性？通常是在会产生二义性结果的地方使用casts。

一点延生：
Overload(重载)：在C++程序中，可以将语义、功能相似的几个函数用同一个名字表示，但参数或返回值不同（包括类型、顺序不同），即函数重载。
（1）相同的范围（在同一个类中）；
（2）函数名字相同；
（3）参数不同；
（4）virtual 关键字可有可无。
Override(覆盖)：是指派生类函数覆盖基类函数，特征是：
（1）不同的范围（分别位于派生类与基类）；
（2）函数名字相同；
（3）参数相同；
（4）基类函数必须有virtual 关键字。
Overwrite(重写)：是指派生类的函数屏蔽了与其同名的基类函数，规则如下：
（1）如果派生类的函数与基类的函数同名，但是参数不同。此时，不论有无virtual关键字，基类的函数将被隐藏（注意别与重载混淆）。
（2）如果派生类的函数与基类的函数同名，并且参数也相同，但是基类函数没有virtual关键字。此时，基类的函数被隐藏（注意别与覆盖混淆）。


### 灵巧指针和const ###
对于dumb指针来说，const既可以针对指针所指向的东西，也可以针对于指针本身，或者兼有两者的含义。

```c++
CD goodCD("Flood");
const CD *p;		// p 是一个non-const 指针
			//指向 const CD 对象

CD * const p = &goodCD;	// p 是一个const 指针，指向non-const CD 对象;
		        // 因为 p 是const, 它必须被初始化

const CD * const p = &goodCD;	// p 是一个const 指针
			       // 指向一个 const CD 对象

```
我们自然想要让灵巧指针具有同样的灵活性。不幸的是只能在一个地方放置const，并只能对指针本身起作用，而不能针对于所指对象：

```c++
const SmartPtr<CD> p = &goodCD;		 // p 是一个const 灵巧指针 
								 // 指向 non-const CD 对象

```
好像有一个简单的补救方法，就是建立一个指向cosnt CD的灵巧指针：

```c++
SmartPtr<const CD> p = &goodCD;		// p 是一个 non-const 灵巧指针
								// 指向const CD 对象

```
现在我们可以建立const和non-const对象和指针的四种不同组合：

```c++
SmartPtr<CD> p;				// non-const 对象
							// non-const 指针
SmartPtr<const CD> p;			// const 对象,
							// non-const 指针
const SmartPtr<CD> p = &goodCD;		 // non-const 对象
								// const指针
const SmartPtr<const CD> p = &goodCD;			 // const 对象
									  	// const 指针

```
但是，这里不能像dump指针一样，non-const能自动的转换成const，而应该用到前面提过的成员函数模版来实现。

## Item M29：引用计数 ##
引用计数这一节，看了至少5遍，终于大体是明白了。
引用计数其实并不难以理解，但是再加上模版、再加上smart指针，就让人看得云里雾里。

引用计数的用处？
我想主要还是为了节省内存，比如我们项目里面做参数采集的时候，参数可能有成千上万个，这些参数可能归属于不同的设备，那不应该对每个设备就去拷贝一份参数，而应该使用引用计数，达到节约内存。
### 实现引用计数 ###
整个小节都是以String对象来进行介绍的，要想达到的效果就像下面一样：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIyMjEwOTQ0ODM0)
a~e是5个String对象，共享Hello字符串，并且有一个统计字段，来维护当前共享的个数。

那现在的问题是这一个统计字段和字符串是存在什么地方的？

是在String内部，一个私有类（结构）保存，就是下面的StringValue：

```c++
class String 
{
public:
	... // the usual String member
		// functions go here
private:
	struct StringValue { ... }; // holds a reference count and a string value
	StringValue *value;         // value of this String
};

class String 
{
private:
	struct StringValue 
	{
		int refCount;
		char *data;
		StringValue(const char *initValue);
		~StringValue();
	};
	...
};
String::StringValue::StringValue(const char *initValue)
	: refCount(1)
{
	data = new char[strlen(initValue) + 1];
	strcpy(data, initValue);
}
String::StringValue::~StringValue()
{
	delete[] data;
}

```
这个私有类只提供了一个堆数组用来保存数据，一个refCount字段用来统计被引用的次数。实际上，真正的计数操作，还是在String对象身上。

看下它的构造函数：

```c++
class String 
{
public:
	String(const char *initValue = "");
	String(const String& rhs);
	...
};

```
初始化构造函数的实现：

```c++
String::String(const char *initValue)
	: value(new StringValue(initValue))
{}

```
用传入的char *字符串创建了一个新的StringValue对象，并将我们正在构造的String对象指向这个新生成的StringValue。

这样的用户代码：

```c++
String s("More Effective C++");
```
生成的数据结构是这样的：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIyMjExMTIzNjk2)
注意，这里对象是被独立构造的，两个同样初始化的值，并不会共享数据。

```c++
String s1("More Effective C++");
String s2("More Effective C++");

```
这仍然会被创建两个独立的对象：

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIyMjExMjEwODg0)
那在什么地方会实现共享？在发生拷贝的时候。

```c++
String::String(const String& rhs): value(rhs.value)
{
	++value->refCount;
}

```
发生拷贝时，直接指向同一个对象，并且引用计数加1。

当释放一个对象的时候，只有在引用计数为0时，才真正的删除：

```c++
String::~String()
{
	if (--value->refCount == 0) delete value;
}

```
当用户写下这样的代码：

```c++
s1 = s2; // s1 and s2 are both String objects
```

其结果应该是s1和s2指向相同的StringValue对象。对象的引用计数应该在赋值时被增加。并且，s1原来指向的StringValue对象的引用计数应该减少，因为s1不再具有这个值了。如果s1是拥有原来的值的唯一对象，这个值应该被销毁。

```c++
String& String::operator=(const String& rhs)
{
	if (value == rhs.value)   // do nothing if the values are already the same
	{ 
		return *this; 
	} 

	if (--value->refCount == 0) // destroy *this's value if no one else is using it
	{ 
		delete value; 
	}
	value = rhs.value; // have *this share rhs's value
	++value->refCount; 
	return *this;
}

```

### 写时拷贝 ###
所谓写时拷贝，字面意思，就是在写的时候进行拷贝。当修改一个对象的时候，这个对象可能被多个其它对象共享，避免影响到其它对象，会把这个对象拷贝出来，在拷贝对象上进行修改，并且原来共享对象的引用计数减1。

看下书中String对象，数组下标操作[]，常量方法与非常量方法：

```c++
class String 
{
public:
	const char& operator[](int index) const;	// for const Strings
	char& operator[](int index);				// for non-const Strings
	...
};

```
常量方法：

```c++
const char& String::operator[](int index) const
{
	return value->data[index];
}

```
非常量方法：

```c++
char& String::operator[](int index)
{
	// if we're sharing a value with other String objects,
	// break off a separate copy of the value for ourselves
	if (value->refCount > 1) 
	{
		--value->refCount; // decrement current value's
						   // refCount, because we won't
						   // be using that value any more
		value = new StringValue(value->data);  // make a copy of the value for ourselves
			
	}
	// return a reference to a character inside our
	// unshared StringValue object
	return value->data[index];
}

```

### 指针、引用与写时拷贝 ###

大部分情况下，写时拷贝可以同时保证效率和正确性。但是写时拷贝也有失效的地方，看上面的源码，发生写时拷贝的前提是，它与其它对象共享。
考虑下面这种情况：

```c++
String s1 = "Hello";
char *p = &s1[1];
String s2 = s1;

```
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIyMjExNjI1MTU2)

s1与s2共享"Hello"对象，p指向"Hello"对象的首地址。
对比两句操作：

```c++
s1[0] = 'x';    //--只改变s1
*p = 'x';      //--会同时修改s1和s2

```
就像注释所示的那样，当通过指针修改"Hello"对象首地址的内容时，由于"Hello"对象没有检测到p,并不会发生写时拷贝，会同时修改两个对象。
解决方法是：在每个StringValue对象中增加一个标志以指出它是否为可共享的。在最初（对象可共享时）将标志打开，在非const的operator[]被调用时将它关闭。一旦标志被设为false，它将永远保持在这个状态。
增加标志后的改进版本：

```c++
class String 
{
private:
	struct StringValue 
	{
		int refCount;
		bool shareable; // add this
		char *data;
		StringValue(const char *initValue);
		~StringValue();
	};
	...
};

String::StringValue::StringValue(const char *initValue)
	: refCount(1),
	shareable(true) // add this
{
	data = new char[strlen(initValue) + 1];
	strcpy(data, initValue);
}

String::StringValue::~StringValue()
{
	delete[] data;
}

```
拷贝构造函数需要先根据标志进行判断：

```c++
String::String(const String& rhs)
{
	if (rhs.value->shareable) 
	{
		value = rhs.value;
		++value->refCount;
	}
	else 
	{
		value = new StringValue(rhs.value->data);
	}
}

```
非const的operator[]版本是唯一将共享标志设为false的地方：

```c++
char& String::operator[](int index)
{
	if (value->refCount > 1) 
	{
		--value->refCount;
		value = new StringValue(value->data);
	}
	value->shareable = false; // add this 
	return value->data[index];
}

```



### 带引用计数的基类 ###

上面的版本中，StringValue要自己管理引用计数，而引用计数不只会用在字符串类上，还会用到很多其它类上，因此设想构建一个基类来管理引用计数，任何需要用引用计数的类都必须从它继承。
这个基类命名为RCObject，它封装了引用计数功能，如增加和减少引用计数的函数。它还包含了当这个值不再被需要时摧毁值对象的代码（也就是引用计数为0时）。最后，它包含了一个字段以跟踪这个值对象是否可共享，并提供查询这个值和将它设为false的函数。

```c++
class RCObject 
{
public:
	RCObject();
	RCObject(const RCObject& rhs);
	RCObject& operator=(const RCObject& rhs);
	virtual ~RCObject() = 0;
	void addReference();
	void removeReference();
	void markUnshareable();
	bool isShareable() const;
	bool isShared() const;
private:
	int refCount;
	bool shareable;
};

```
RCOject的实现代码：

```c++
RCObject::RCObject(): refCount(0), shareable(true) {}

RCObject::RCObject(const RCObject&)	: refCount(0), shareable(true) {}

RCObject& RCObject::operator=(const RCObject&)
{
	return *this;
}
RCObject::~RCObject() {} // virtual dtors must always be implemented

void RCObject::addReference() { ++refCount; }
void RCObject::removeReference()
{
	if (--refCount == 0) delete this;
}
void RCObject::markUnshareable()
{
	shareable = false;
}
bool RCObject::isShareable() const
{
	return shareable;
}
bool RCObject::isShared() const
{
	return refCount > 1;
}

```

这个RCObject有两个地方需要注意的，一是注意它们的构造函数，refCount被赋为0，而不是1。给refCount的赋值操作，应该在使用这个基类的派生类中来实现。

另一个地方是它的赋值操作，这里什么都没作。

```c++
RCObject& RCObject::operator=(const RCObject&)
{
	return *this;
}

```
这里比较难理解，通过例子就能说明一切：
假设有StringValue的sv1和sv2两个对象，StringValue是继承自RCObject的，那么它们之间的赋值操作：

```c++
sv1 = sv2;
```
会发生什么？

sv1的值会变为sv2，也就是共享sv1对象的所有对象的值都会发生变化，但共享sv1对象的数量仍然是这么多，没发生变化。同样，sv2的引用计数也没有发生改变。

下面看它的使用，重构StringValue，使它继承自RCObject：

```c++
String::StringValue::StringValue(const char *initValue)
{
	data = new char[strlen(initValue) + 1];
	strcpy(data, initValue);
}
String::StringValue::~StringValue()
{
	delete[] data;
}

```
现在，这个版本不再存放引用计数，放在基类中进行存放，注意这里refCount的初始值为0，需要使用它的对象进行管理。这比较笨拙，下面看下自动的引用计数处理。



### 自动的引用计数处理 ###
先回顾一下原来的String版本，我们在任何拷贝指针、给指针赋值和销毁指针的时候要自己去value->refCount。

```c++
class String 
{
public:
	... // the usual String member
		// functions go here
private:
	struct StringValue { ... }; // holds a reference count and a string value
	StringValue *value;         // value of this String
};


```

现在，我们用smart指针来代替value，让smart指针来帮我们进行引用计数处理。

这是一个模版类，实际指向的是实现了RCObject的类：

```c++
// template class for smart pointers-to-T objects. T must
// support the RCObject interface, typically by inheriting
// from RCObject
template<class T>
class RCPtr 
{
public:
	RCPtr(T* realPtr = 0);
	RCPtr(const RCPtr& rhs);
	~RCPtr();
	RCPtr& operator=(const RCPtr& rhs);
	T* operator->() const; // see Item 28
	T& operator*() const; // see Item 28
private:
	T *pointee; // dumb pointer this object is emulating
	void init(); // common initialization
};

```
分成几个部分来讲下它的实现：
看它的构造函数：

```c++
template<class T>
RCPtr<T>::RCPtr(T* realPtr) : pointee(realPtr)
{
	init();
}
template<class T>
RCPtr<T>::RCPtr(const RCPtr& rhs) : pointee(rhs.pointee)
{
	init();
}
template<class T>
void RCPtr<T>::init()
{
	if (pointee == 0) { // if the dumb pointer is null, so is the smart one
		return;     
	}
	// if the value isn't shareable,copy it
	if (pointee->isShareable() == false) { 
		pointee = new T(*pointee); 
	} 
	pointee->addReference(); // note that there is now a new reference to the value
}

```

这里会有个隐藏着的Bug：

这句语句实际上调用的是T的构造函数。

```c++
pointee = new T(*pointee);
```
我们把T假设为StringValue，前面的所有版本中，都没有实现这样的构造函数：

```c++
StringValue(const StringValue & value);
```
编译器将为我们生成一个。这个生成的拷贝构造函数遵守C++的自动生成拷贝构造函数的原则，只拷贝了StringValue的数据pointer，而没有拷贝所指向的char *字符串，是一个浅拷贝。所以使用这类模版的时候，必须要提高形如下面的深层拷贝：

```c++
String::StringValue::StringValue(const StringValue& rhs)
{
	data = new char[strlen(rhs.data) + 1];
	strcpy(data, rhs.data);
}

```
一点思考：为什么不能用浅拷贝？如果两个对象的成员变量指向同一块内存，当一个变量把内存释放后，对另外一个变量将是一个灾难。

赋值函数：

```c++
template<class T>
RCPtr<T>& RCPtr<T>::operator=(const RCPtr& rhs)
{
	if (pointee != rhs.pointee) // skip assignments where the value doesn't change
	{ 
		if (pointee) 
		{
			pointee->removeReference(); // remove reference to current value
		} 
		pointee = rhs.pointee; // point to new value
		init(); // if possible, share it
	} // else make own copy
	return *this;
}

```
析构函数：

```c++
template<class T>
RCPtr<T>::~RCPtr()
{
	if (pointee)pointee->removeReference();
}

```

最后部分是smart指针的提领操作：

```c++
template<class T>
T* RCPtr<T>::operator->() const { return pointee; }
template<class T>
T& RCPtr<T>::operator*() const { return *pointee; }

```

### 合在一起 ###

有了前面的准备知识，现在将各个部分放在一起，构造一个基于可重用的RCObject和RCPtr类的带引用计数的String类。

每个带引用计数的Sting对象被实现为这样的数据结构：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIyMjEyNjE2OTk4)
所有类的定义：

```c++
template<class T> // template class for smart
class RCPtr { // pointers-to-T objects; T must inherit from RCObject
public: 
	RCPtr(T* realPtr = 0);
	RCPtr(const RCPtr& rhs);
	~RCPtr();
	RCPtr& operator=(const RCPtr& rhs);
	T* operator->() const;
	T& operator*() const;
private:
	T *pointee;
	void init();
};

```

```c++
class RCObject { // base class for reference counted objects
public: 
	void addReference();
	void removeReference();
	void markUnshareable();
	bool isShareable() const;
	bool isShared() const;
protected:
	RCObject();
	RCObject(const RCObject& rhs);
	RCObject& operator=(const RCObject& rhs);
	virtual ~RCObject() = 0;
private:
	int refCount;
	bool shareable;
};

```

```c++
class String { // class to be used by
public: // application developers
	String(const char *value = "");
	const char& operator[](int index) const;
	char& operator[](int index);
private:
	// class representing string values
	struct StringValue : public RCObject {
		char *data;
		StringValue(const char *initValue);
		StringValue(const StringValue& rhs);
		void init(const char *initValue);
		~StringValue();
	};
	RCPtr<StringValue> value;
};

```
这里有一个重大的不同：这个String类的公有接口和本条款开始处我们使用的版本不同。拷贝构造函数在哪里？赋值运算在哪里？析构函数在哪里？这儿明显有问题。
实际上，没问题。它工作得很好。我们不再需要那些函数了！因为编译器为String自动生成的拷贝构造函数将自动调用其RCPtr成员的拷贝构造函数，而这个拷贝构造函数完成所有必须的对StringValue对象的操作，包括它的引用计数。

将所有东西放在一起，这儿是RCObject的实现：

```c++
RCObject::RCObject()
	: refCount(0), shareable(true) {}
RCObject::RCObject(const RCObject&)
	: refCount(0), shareable(true) {}
RCObject& RCObject::operator=(const RCObject&)
{
	return *this;
}
RCObject::~RCObject() {}
void RCObject::addReference() { ++refCount; }
void RCObject::removeReference()
{
	if (--refCount == 0) delete this;
}
void RCObject::markUnshareable()
{
	shareable = false;
}
bool RCObject::isShareable() const
{
	return shareable;
}
bool RCObject::isShared() const
{
	return refCount > 1;
}

```

这是RCPtr的实现：

```c++
template<class T>
RCPtr<T>::RCPtr(T* realPtr)
	: pointee(realPtr)
{
	init();
}
template<class T>
RCPtr<T>::RCPtr(const RCPtr& rhs)
	: pointee(rhs.pointee)
{
	init();
}
template<class T>
RCPtr<T>::~RCPtr()
{
	if (pointee)pointee->removeReference();
}
template<class T>
RCPtr<T>& RCPtr<T>::operator=(const RCPtr& rhs)
{
	if (pointee != rhs.pointee) {
		if (pointee) pointee->removeReference();
		pointee = rhs.pointee;
		init();
	}
	return *this;
}
template<class T>
T* RCPtr<T>::operator->() const { return pointee; }
template<class T>
T& RCPtr<T>::operator*() const { return *pointee; }

```
这是String::StringValue的实现：

```c++
void String::StringValue::init(const char *initValue)
{
	data = new char[strlen(initValue) + 1];
	strcpy(data, initValue);
}
String::StringValue::StringValue(const char *initValue)
{
	init(initValue);
}
String::StringValue::StringValue(const StringValue& rhs)
{
	init(rhs.data);
}
String::StringValue::~StringValue()
{
	delete[] data;
}

String::String(const char *initValue): value(new StringValue(initValue)) {}
const char& String::operator[](int index) const
{
	return value->data[index];
}
char& String::operator[](int index)
{
	if (value->isShared()) {
		value = new StringValue(value->data);
	}
	value->markUnshareable();
	return value->data[index];
}

```
非常优雅，完成了同样的工作，但代码量大大减少。


### 在现存类上增加引用计数 ###
到现在为止，我们所讨论的都假设我们能够访问有关类的源码。但如果我们想让一个位于支撑库中而无法修改的类获得引用计数的好处呢？不可能让它们从RCObject继承的，所以也不能对它们使用灵巧指针RCPtr。

只要对我们的设计作小小的修改，我们就可以将引用计数加到任意类型上。
计算机科学中的绝大部分问题都可以通过增加一个中间层次来解决。我们增加一个新类CountHolder以处理引用计数，它从RCObject继承。我们让CountHolder包含一个指针指向Widget。然后用等价的灵巧指针RCIPter模板替代RCPtr模板，它知道CountHolder类的存在。（名字中的“i”表示间接“indirect”。）修改后的设计为：

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIyMjEyOTQ4MDM0)

这个版本的技巧就在下面这个Holder类上：

```c++
struct CountHolder : public RCObject {
	~CountHolder() { delete pointee; }
	T *pointee;
};

```
作为一个中间类，包含一个指向真正内容的指针。

## Item M30：代理类 ##
为什么需要代理类？看下引出这个主题的问题：

这是合法的，定义一个二维数组：

```c++
int data[10][20];    // 2D array: 10 by 20
```
而相同的结构如果使用变量作维的大小的话，是不可以的：

```c++
void processInput(int dim1, int dim2)
{
	int data[dim1][dim2]; // error! array dimensions
	...                  // must be known during
}

```
甚至，在堆分配时都是不合法的：

```c++
int *data =new int[dim1][dim2];   // error!
```
### 实现二维数组 ###

我们可以定义一个类模板来实现二维数组：

```c++
template<class T>
class Array2D 
{
public:
	Array2D(int dim1, int dim2);
	...
};

```

使用方法与通常的类构造相似：

```c++
Array2D<int> data(10, 20); // fine
Array2D<float> *data =new Array2D<float>(10, 20); // fine
void processInput(int dim1, int dim2)
{
	Array2D<int> data(dim1, dim2); // fine
	...
}

```

然而，使用这些array对象并不直接了当。根据C和C++中的语法习惯，我们应该能够使用[]来索引数组：

```c++
cout << data[3][6];
```
不要妄想申明一个operator[][]函数，因为没有operator[][]这种东西。

一种方法是重载operator()，但要容忍奇怪的语法。

```c++
template<class T>
class Array2D 
{
public:
	// declarations that will compile
	T& operator()(int index1, int index2);
	const T& operator()(int index1, int index2) const;
	...
};

```
用户于是这么使用数组：

```c++
cout << data(3, 6);
```
另一种方法是引入一个Array1D的代理类：

```c++
int data[10][20];
cout << data[3][6];

```
分析上面条语句：变量data不是真正的二维数组，它是一个10元素的一维数组。其中每一个元素又都是一个20元素的数组。第一个[]返回的是一个数组，第二个[]从这个返回的数组中再去取一个元素。
我们可以通过重载Array2D类的operator[]来玩同样的把戏。Array2D的operator[]返回一个新类Array1D的对象。再重载Array1D的operator[]来返回所需要的二维数组中的元素：

```c++
Array2D
{
public:
	class Array1D 
	{
	public:
		T& operator[](int index);
		const T& operator[](int index) const;
		...
	};
	Array1D operator[](int index);
	const Array1D operator[](int index) const;
	...
};

```
现在，它合法了：

```c++
Array2D<float> data(10, 20);
...
cout << data[3][6]; // fine

```
这里，data[3]返回一个Array1d对象，在这个对象上的operator[]操作返回二维数组中(3,6)位置上的浮点数。

一点思考：
前面谈到了：

```c++
int *data =new int[dim1][dim2];   // error!
```
这个句语是错误的，那该怎么构造Array2D呢？书中没说。
应该不外乎类似下面的语句：

```c++
//动态开辟空间  
int **p = new int*[m]; //开辟行  
for (int i = 0; i < m; i++)
	p[i] = new int[n]; //开辟列

```


### 区分通过operator[]进行的是读操作还是写操作 ###

支持operator[]的string类型，允许用户些下这样的代码：

```c++
String s1, s2;      // a string-like class; the
			   // use of proxies keeps this
			   // class from conforming to
			   // the standard string interface
... 
cout << s1[5]; // read s1
s2[5] = 'x';   // write s2
s1[3] = s2[8]; // write s1, read s2

```
能够区分读和写。
怎么区分的，迷惑的是使用const属性重载operator[]，这样能不能区分？

```c++
class String 
{
public:
	const char& operator[](int index) const; // for reads
	char& operator[](int index); // for writes
	...
};

```
唉，这不能工作。编译器根据调用成员函数的对象的const属性来选择此成员函数的const和非const版本，而不考虑调用时的环境。因此：

```c++
String s1, s2;
...
cout << s1[5]; // calls non-const operator[],because s1 isn't const
s2[5] = 'x'; // also calls non-const operator[]: s2 isn't const
s1[3] = s2[8]; // both calls are to non-const operator[], because both s1
			   // and s2 are non-const objects

```
解决方法是使用一个proxy对象，由proxy对象来判断是读还是写。

```c++
class String { // reference-counted strings;
public: // see Item 29 for details
	class CharProxy { // proxies for string chars
	public:
		CharProxy(String& str, int index); // creation
		CharProxy& operator=(const CharProxy& rhs); // lvalue
		CharProxy& operator=(char c); // uses
		operator char() const; // rvalue
							   // use
	private:
		String& theString; // string this proxy pertains to
		int charIndex; // char within that string
					   // this proxy stands for
	};
	// continuation of String class
	const CharProxy operator[](int index) const; // for const Strings
	CharProxy operator[](int index); // for non-const Strings
	...
		friend class CharProxy;
private:
	RCPtr<StringValue> value;
};

```
这里面多了一个CharProxy，唯一暴露给用户的地方是重载[]的时候，构造函数传入了一个字符串对象的引用和当前字符的索引。

现在，这条语句就能正常工作了：

```c++
cout << s1[5];
```
表达式s1[5]返回的是一CharProxy对象。没有为这样的对象定义输出流操作，所以编译器努力地寻找一个隐式的类型转换以使得operator<<调用成功（见Item M5）。它们找到一个：在CahrProxy类内部申明了一个隐式转换到char的操作。于是自动调用这个转换操作，结果就是CharProxy类扮演的字符被打印输出了。
这是String的opertator[]函数的代码：

```c++
const String::CharProxy String::operator[](int index) const
{
	return CharProxy(const_cast<String&>(*this), index);
}
String::CharProxy String::operator[](int index)
{
	return CharProxy(*this, index);
}

```
每个函数都创建和返回一个proxy对象来代替字符。根本没有对那个字符作任何操作：我们将它推迟到直到我们知道是读操作还是写操作。

proxy对象记录了它属于哪个string对象以及所扮演的字符的下标：


```c++
String::CharProxy::CharProxy(String& str, int index)
	: theString(str), charIndex(index) {}

```

将proxy对象作右值使用时很简单，只需返回它所扮演的字符就可以了：

```c++
String::CharProxy::operator char() const
{
	return theString.value->data[charIndex];
}

```
回头再看CahrProxy的赋值操作的实现，这是我们必须处理proxy对象所扮演的字符作赋值的目标（即左值）使用的地方：

```c++
String::CharProxy& String::CharProxy::operator=(const CharProxy& rhs)
{
	// if the string is sharing a value with other String objects,
	// break off a separate copy of the value for this string only
	if (theString.value->isShared()) 
	{
		theString.value = new StringValue(theString.value->data);
	}
	// now make the assignment: assign the value of the char
	// represented by rhs to the char represented by *this
	theString.value->data[charIndex] =rhs.theString.value->data[rhs.charIndex];
	return *this;
}

```

第二个CharProxy的赋值操作是类似的：

```c++
String::CharProxy& String::CharProxy::operator=(char c)
{
	if (theString.value->isShared()) {
		theString.value = new StringValue(theString.value->data);
	}
	theString.value->data[charIndex] = c;
	return *this;
}

```
综上，CharProxy能自己判断是做左值还是右值，做左值的时候调用operator=，做右值的时候调用隐式类型转换。

### 局限性 ###
收回上面的，CharProxy能自己判断是做左值还是右值的话，因为它不是万能的，右值不只是出现在赋值运算的情况下，还可能出现在下面这些地方：

例外1：
```c++
String s1 = "Hello";
char *p = &s1[1]; // error!

```
表达式s1[1]返回一个CharProxy，于是“=”的右边是一个CharProxy *。没有从CharProxy *到char *的转换函数，所以p的初始化过程编译失败了。应该重载CharProxy类的取地址运算。

例外2：
如果有一个引用计数的数组：

```c++
template<class T> // reference-counted array  using proxies
class Array { 
public:
	class Proxy {
	public:
		Proxy(Array<T>& array, int index);
		Proxy& operator=(const T& rhs);
		operator T() const;
		...
	};
	const Proxy operator[](int index) const;
	Proxy operator[](int index);
	...
};

```
常见使用：

```c++
Array<int> intArray;
...
intArray[5] = 22; // fine
intArray[5] += 5; // error!
++intArray[5];    // error!

```c++
当operator[]作最简单的赋值操作的目标时，是成功的，但当它出现operator+=和operator++的左侧时，失败了。因为operator[]返回一个proxy对象，而它没有operator+=和operator++操作。同样的情况存在于其它需要左值的操作中，包括operator*=、operator<<=、operator--等等。

例外3：
不能通过proxy对象调用实际对象的成员函数。
例如，假设我们用带引用计数的数组处理有理数。我们将定义一个Rational类，然后使用前面看到的Array模板：

```
class Rational {
public:
	Rational(int numerator = 0, int denominator = 1);
	int numerator() const;
	int denominator() const;
	...
};

```
类似的调用会出错：

```c++
cout << array[4].numerator(); // error!
int denom = array[22].denominator(); // error!

```
operator[]返回一个proxy对象而不是实际的Rational对象。但成员函数numerator()和denominator()只存在于Rational对象上，而不是其proxy对象。

## Item M31：让函数根据一个以上的对象来决定怎么虚拟 ##
我们都知道虚函数，但是有时需要的是一种作用在多个对象上的虚函数，怎么实现？

看书中的例子：
有这么一个游戏，游戏的背景是发生在太空，有宇宙飞船、太空站和小行星。
在你构造的世界中的宇宙飞船、太空站和小行星，它们可能会互相碰撞。假设其规则是：
如果飞船和空间站以低速接触，飞船将泊入空间站。否则，它们将有正比于相对速度的损坏。
如果飞船与飞船，或空间站与空间站相互碰撞，参与者均有正比于相对速度的损坏。
如果小行星与飞船或空间站碰撞，小行星毁灭。如果是小行星体积较大，飞船或空间站也毁坏。
如果两个小行星碰撞，将碎裂为更小的小行星，并向各个方向溅射。

可能的继承体系：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIyMjE1OTU2ODQx)

类结构：

```c++
class GameObject { ... };
class SpaceShip : public GameObject { ... };
class SpaceStation : public GameObject { ... };
class Asteroid : public GameObject { ... };

```
可能处理碰撞的过程是这样的：

```c++
void checkForCollision(GameObject& object1,	GameObject& object2)
{
	if (theyJustCollided(object1, object2)) 
	{
		processCollision(object1, object2);
	}
	else {
		...
	}
}

```
这里首先判断它们是否碰撞，然后调用processCollision()函数来处理碰撞。这里的问题是：你知道发生的结果将取决于object1和object2的真实类型，但你并不知道其真实类型；你所知道的就只有它们是GameObject对象。如果碰撞的处理过程只取决于object1的动态类型，你可以将processCollision()设为虚函数，并调用object1.processColliion(object2)。如果只取决于object2的动态类型，也可以同样处理。但现在，取决于两个对象的动态类型。虚函数体系只能作用在一个对象身上，它不足以解决问题。

这是一种被称为“二重调度（double dispatch）”的问题。下面列举了几种解决此问题的方法：



### 用虚函数加RTTI（运行时类型识别）###
按通常的方式，虚函数实现一个单一的调度：

```c++
class GameObject 
{
public:
	virtual void collide(GameObject& otherObject) = 0;
	...
};
class SpaceShip : public GameObject 
{
public:
	virtual void collide(GameObject& otherObject);
	...
};

```
处理碰撞的过程变成了这样：

```c++
void SpaceShip::collide(GameObject& otherObject)
{
	const type_info& objectType = typeid(otherObject);
	if (objectType == typeid(SpaceShip)) 
	{
		SpaceShip& ss = static_cast<SpaceShip&>(otherObject);
		process a SpaceShip - SpaceShip collision;
	}
	else if (objectType == typeid(SpaceStation)) 
	{
		SpaceStation& ss =
			static_cast<SpaceStation&>(otherObject);
		process a SpaceShip - SpaceStation collision;
	}
	else if (objectType == typeid(Asteroid)) 
	{
		Asteroid& a = static_cast<Asteroid&>(otherObject);
		process a SpaceShip - Asteroid collision;
	}
	else 
	{
		throw CollisionWithUnknownObject(otherObject);
	}
}

```
我们需要检测的只是一个对象的类型，通过typeid来检测。另一个是*this，它的类型由虚函数体系判断。这种方法存在的问题是什么？
一是：代码重复，在每一个子类中都要维护这样一个巨大的if-else if 结构。
二是：为了防止未知的对象，最后必须加一个else子句，这一个异常让调用者很难处理。



### 只使用虚函数 ###
下面的方法可以只用虚函数，就能确定两个对象的动态类型。在基类中重载collide函数，并且每个派生类都要实现它们：

```c++
class SpaceShip; // forward declarations
class SpaceStation;
class Asteroid;
class GameObject 
{
public:
	virtual void collide(GameObject& otherObject) = 0;
	virtual void collide(SpaceShip& otherObject) = 0;
	virtual void collide(SpaceStation& otherObject) = 0;
	virtual void collide(Asteroid& otherobject) = 0;
	...
};
class SpaceShip : public GameObject 
{
public:
	virtual void collide(GameObject& otherObject);
	virtual void collide(SpaceShip& otherObject);
	virtual void collide(SpaceStation& otherObject);
	virtual void collide(Asteroid& otherobject);
	...
};

```
使用方法超级简单：

```c++
void SpaceShip::collide(GameObject& otherObject)
{
	otherObject.collide(*this);
}

```
在这里*this与otherObject的静态类型都是清楚的。这个函数在SpaceShip中，所以*this的类型就是SpaceShip。所有的collide函数都是虚函数，所以在SpaceShip::collide中调用的是otherObject真实类型中实现的collide版本。

但是这种方法有一个致命的缺陷：当有一个新的类型来时，每个类中都要新增一个collide的重载版本。但是有时候修改现存类经常是做不到的，比如原来是一个支撑库。


### 模拟虚函数表 ###
实质就是维护一个函数指针数组，这个函数指针与一类参数有一一映射的关系，使用的时候，其实就是寻找这个映射。

新的实现版本：

```c++
class GameObject 
{
public:
	virtual void collide(GameObject& otherObject) = 0;
	...
};

class SpaceShip : public GameObject 
{
public:
	virtual void collide(GameObject& otherObject);
	virtual void hitSpaceShip(SpaceShip& otherObject);
	virtual void hitSpaceStation(SpaceStation& otherObject);
	virtual void hitAsteroid(Asteroid& otherobject);
	...
};

```
这里最大的不同是，放弃了collide函数的重载，而是给每个类型的碰撞定义一个名字。这里之所以要单独另起名字，是为了要维护一个对象类型与函数指针的映射表，比如碰撞对象是SpaceShip，则返回的是hitSpaceShip函数指针，而如果碰撞对象是SpaceStation，则返回的是hitSpaceStation指针。
那现在的问题，集中到了如何创建这个映射表以及如何寻找这个映射上了。

直接看最优雅的实现，用STL中的Map数据结构：

```c++
class SpaceShip : public GameObject 
{
private:
	typedef void (SpaceShip::*HitFunctionPtr)(GameObject&);
	typedef map<string, HitFunctionPtr> HitMap;
	...
};

SpaceShip::HitFunctionPtr SpaceShip::lookup(const GameObject& whatWeHit)
{
	static HitMap collisionMap; // --下一个小节详解对它的初始化

	HitMap::iterator mapEntry =
		collisionMap.find(typeid(whatWeHit).name());

	// mapEntry == collisionMap.end() if the lookup failed;	
	if (mapEntry == collisionMap.end()) return 0;
	
	return (*mapEntry).second;
}

```


### 初始化模拟虚函数表 ###
初始化虚函数表并不是想像中这么简单，下面书中还是按“提出一种做法—然后否定这种做法”这一思路进行了说明。
#### 直接在lookup函数中 ###
像这样，在每次寻找映射时，进行初始化：

```c++
SpaceShip::HitFunctionPtr SpaceShip::lookup(const GameObject& whatWeHit)
{
	static HitMap collisionMap;
	collisionMap["SpaceShip"] = &hitSpaceShip;
	collisionMap["SpaceStation"] = &hitSpaceStation;
	collisionMap["Asteroid"] = &hitAsteroid;
	...
}

```
这种方法主要问题是每调用一次lookup，就要添加一次函数指针，虽然key值是唯一的，但还是不必要的开销。


#### 定义私有初始化函数 ###
替代方法是用一个初始化函数，过程变为这样：

```c++
class SpaceShip : public GameObject 
{
private:
	static HitMap initializeCollisionMap();
	...
};
SpaceShip::HitFunctionPtr
SpaceShip::lookup(const GameObject& whatWeHit)
{
	static HitMap collisionMap = initializeCollisionMap();
	...
}

```
为了避免拷贝开销，替换成指针，并用智能指针代替：

```c++
class SpaceShip : public GameObject 
{
private:
	static HitMap * initializeCollisionMap();
	...
};
SpaceShip::HitFunctionPtr SpaceShip::lookup(const GameObject& whatWeHit)
{
	static auto_ptr<HitMap> collisionMap(initializeCollisionMap());
	...
}

```
这个initializeCollisionMap函数可能想这样写：

```c++
SpaceShip::HitMap * SpaceShip::initializeCollisionMap()
{
	HitMap *phm = new HitMap;
	(*phm)["SpaceShip"] = &hitSpaceShip;
	(*phm)["SpaceStation"] = &hitSpaceStation;
	(*phm)["Asteroid"] = &hitAsteroid;
	return phm;
}

```
但它是不可能编译成功的，因为

```c++
typedef void (SpaceShip::*HitFunctionPtr)(GameObject&);
typedef map<string, HitFunctionPtr> HitMap;

```
Map需要的是一个GameObject作为参数的函数指针，而不是它子类作为参数。

可能想到了，直接应用reinterpret_cast进行函数类型转换：

```c++
SpaceShip::HitMap * SpaceShip::initializeCollisionMap()
{
	HitMap *phm = new HitMap;
	(*phm)["SpaceShip"] =reinterpret_cast<HitFunctionPtr>(&hitSpaceShip);
	(*phm)["SpaceStation"] =reinterpret_cast<HitFunctionPtr>(&hitSpaceStation);
	(*phm)["Asteroid"] =reinterpret_cast<HitFunctionPtr>(&hitAsteroid);
	return phm;
}

```
这样虽然能通过编译，但引起了更致命可怕的问题—传递地址错误！

注：把原文摘抄在这里，不是太理解，以下是书中的原文：
这样可以编译通过，但是个坏主意。它必然伴随一些你绝不该做的事：对你的编译器撒谎。告诉编译器，hitSpaceShip、hitSpaceStation和hitAsteroid期望一个GameObject类型的参数，而事实不是这样的。hitSpaceShip期望一个SpaceShip，hitSpaceStation期望一个SpaceStation，hitAsteroid期望一个Asteroid。这些cast说的是其它东西，它们撒谎了。
不只是违背了原则，这儿还有危险。编译器不喜欢被撒谎，当它们发现被欺骗后，它们经常会找出一个报复的方法。这此处，它们很可能通过产生错误的代码来报复你，当你通过*phm调用函数，而相应的GameObject的派生类是多重继承的或有虚基类时。如果SpaceStation。SpaceShip或Asteroid除了GameObject外还有其它基类，你可能会发现当你调用你在这儿搜索到的碰撞处理函数时，其行为非常的粗暴。
再看一下Item M24中描述的A－B－C－D的继承体系以及D的对象的内存布局。
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIyMjIwNjU2ODAx)
D中的四个类的部分，其地址都不同。这很重要，因为虽然指针和引用的行为并不相同（见Item M1），编译器产生的代码中通常是通过指针来实现引用的。于是，传引用通常是通过传指针来实现的。当一个有多个基类的对象（如D的对象）传引用时，最重要的就是编译器要传递正确的地址－－匹配于被调函数申明的形参类型的那个。
但如果你对你的编译器撒谎说你的函数期望一个GameObject而实际上要的是一个SpaceShip或一个SpaceStation时，发生什么？编译器将传给你错误的地址，导致运行期错误。

为了解决上面这个问题，将所有的函数都改为接受GameObject类型：

```c++
class GameObject 
{ // this is unchanged
public:
	virtual void collide(GameObject& otherObject) = 0;
	...
};
class SpaceShip : public GameObject 
{
public:
	virtual void collide(GameObject& otherObject);
	// these functions now all take a GameObject parameter
	virtual void hitSpaceShip(GameObject& spaceShip);
	virtual void hitSpaceStation(GameObject& spaceStation);
	virtual void hitAsteroid(GameObject& asteroid);
	...
};

```
现在，我们理解为什么这儿没有照抄而使用了一组成员函数指针。所有的碰撞处理函数都有着相同的参数类型，所以必要给它们以不同的名字。

这样，map赋值的时候不用函数指针类型转换，因为都是GameObject类型，但在每个具体的处理函数中，需要转换成具体的类型：

```c++
SpaceShip::HitMap * SpaceShip::initializeCollisionMap()
{
	HitMap *phm = new HitMap;
	(*phm)["SpaceShip"] = &hitSpaceShip;
	(*phm)["SpaceStation"] = &hitSpaceStation;
	(*phm)["Asteroid"] = &hitAsteroid;
	return phm;
}

void SpaceShip::hitSpaceShip(GameObject& spaceShip)
{
	SpaceShip& otherShip =
		dynamic_cast<SpaceShip&>(spaceShip);
	process a SpaceShip - SpaceShip collision;
}
void SpaceShip::hitSpaceStation(GameObject& spaceStation)
{
	SpaceStation& station =
		dynamic_cast<SpaceStation&>(spaceStation);
	process a SpaceShip - SpaceStation collision;
}
void SpaceShip::hitAsteroid(GameObject& asteroid)
{
	Asteroid& theAsteroid =
		dynamic_cast<Asteroid&>(asteroid);
	process a SpaceShip - Asteroid collision;
}

```


### 使用非成员的碰撞处理函数 ###

把碰撞处理函数从类中剥离出来，成为非成员函数，函数的参数是要传递的两个对象。

在一个未命名的名称空间中定义碰撞处理函数、函数指针、以及映射表：

```c++
#include "SpaceShip.h"
#include "SpaceStation.h"
#include "Asteroid.h"
namespace { // unnamed namespace — see below
			// primary collision-processing functions
	void shipAsteroid(GameObject& spaceShip,GameObject& asteroid);
	void shipStation(GameObject& spaceShip,GameObject& spaceStation);
	void asteroidStation(GameObject& asteroid,GameObject& spaceStation);
	...
		// secondary collision-processing functions that just
		// implement symmetry: swap the parameters and call a
		// primary function
		void asteroidShip(GameObject& asteroid,GameObject& spaceShip)
	{
		shipAsteroid(spaceShip, asteroid);
	}
	void stationShip(GameObject& spaceStation,GameObject& spaceShip)
	{
		shipStation(spaceShip, spaceStation);
	}
	void stationAsteroid(GameObject& spaceStation,	GameObject& asteroid)
	{
		asteroidStation(asteroid, spaceStation);
	}
	...
		// see below for a description of these types/functions
		typedef void(*HitFunctionPtr)(GameObject&, GameObject&);
	typedef map< pair<string, string>, HitFunctionPtr > HitMap;
	pair<string, string> makeStringPair(const char *s1,	const char *s2);
	HitMap * initializeCollisionMap();
	HitFunctionPtr lookup(const string& class1,	const string& class2);
} // end namespace

```
注意以前的映射表由一个对象确定，而现在要由两个对象确定，所以使用了STL中的pair数据结构。
创建pair:

```c++
pair<string, string> makeStringPair(const char *s1,	const char *s2)
{
	return pair<string, string>(s1, s2);
}

```
初始化映射表：

```c++
HitMap * initializeCollisionMap()
{
	HitMap *phm = new HitMap;
	(*phm)[makeStringPair("SpaceShip", "Asteroid")] =
		&shipAsteroid;
	(*phm)[makeStringPair("SpaceShip", "SpaceStation")] =
		&shipStation;
	...
		return phm;
}

```
查找映射表：

```c++
HitFunctionPtr lookup(const string& class1,const string& class2)
{
	static auto_ptr<HitMap>
		collisionMap(initializeCollisionMap());
	// see below for a description of make_pair
	HitMap::iterator mapEntry =
		collisionMap->find(make_pair(class1, class2));
	if (mapEntry == collisionMap->end()) return 0;
	return (*mapEntry).second;
}

```
处理碰撞函数：

```c++
void processCollision(GameObject& object1,GameObject& object2)
{
	HitFunctionPtr phf = lookup(typeid(object1).name(),typeid(object2).name());
	if (phf) phf(object1, object2);
	else throw UnknownCollision(object1, object2);
}

```
理解这部分代码比较容易，但应该要理解为什么使用非成员函数的形式？
主要的好处是：
如果增加了新的GaemObject的子类，现存类不需要重新编译；也没有了RTTI的混乱和if...then...else的不可维护。只需要在初始化映射表中增加一个映射关系，并定义一个新的碰撞处理函数。

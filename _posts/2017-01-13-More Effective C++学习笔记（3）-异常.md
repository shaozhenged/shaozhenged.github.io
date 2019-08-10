---
layout:     post
title:      "More Effective C++学习笔记（3）-异常"
date:       2017-01-13 12:00:00
author:     "邵正"
header-img: "img/home-bg-o.jpg"
tags:
    - C++
    - More effective c++
---


| 主题     | 概要                    |
| -------- | ----------------------- |
| C++      | More Effective C++ 异常 |
| -------- | ---                     |
| **编辑** | **时间**                |
| 新建     | 20170113                |
| -------- | ---                     |
| **序号** | **参考资料**            |
| 1        | More effective C++      |

C++中引进了异常机制，对比C语言程序员用错误码来标识错误，无疑更加优雅和健状。对于写出异常安全的代码，应该牢记遵循下面的准则。
## Item M9：使用析构函数防止资源泄漏 ##

对指针说再见。不用对所有的指针说再见，但是你需要对用来操纵局部资源（local resources）的指针说再见。
直接看代码：

```c++
// 从s中读去动物信息, 然后返回一个指针
// 指向新建立的某种类型对象
ALA * readALA(istream& s);

//你的程序的关键部分就是这个函数
void processAdoptions(istream& dataSource)
{
	while (dataSource) { // 还有数据时,继续循环
		ALA *pa = readALA(dataSource); //得到下一个动物
		pa->processAdoption(); //处理收容动物
		delete pa; //删除readALA返回的对象
	}
}
```
可以不用理解这个函数的具体作用，只要认识到这一点：while循环体内定义了一个局部指针变量，每次循环完后删除这个对象。问题出现在:

```c++
pa->processAdoption(); //处理收容动物
delete pa; //删除readALA返回的对象

```
这两句代码之间。想像一下，当处理函数内部出现异常并抛出，则它后面的语句
delete pa
不会得到执行，则会出现资源泄漏的风险。

可能想到的法子是，直接在函数内部捕获这个异常：

```c++
void processAdoptions(istream& dataSource)
{
	while (dataSource) 
	{
		ALA *pa = readALA(dataSource);
		try 
		{
			pa->processAdoption();
		}
		catch (...)						// 捕获所有异常
		{ 
			delete pa;					// 避免内存泄漏
										// 当异常抛出时
			throw;						// 传送异常给调用者
		}
		delete pa;						// 避免资源泄漏
	} // 当没有异常抛出时
}

```

但这里为了删除资源，要在两个地方维护，应当消除这种低质量的重复代码。

一点思考：能否依懒于 try…catch…final 块？

消除方法是把释放资源的操作放在函数体内局部对象的析构函数里，因为当函数返回时局部对象总是被释放，无论函数是如何退出的。具体方法是用一个对象代替指针pa，这个对象的行为与指针相似。当pointer-like对象（类指针对象）被释放时，我们能让它的析构函数调用delete。

C++里面和boost库里面就有这种对象，称为智能指针。

简易版的auto_ptr长这样子，在它的析构函数中释放资源。

```c++
template<class T>
class auto_ptr 
{
public:
	auto_ptr(T *p = 0) : ptr(p) {} // 保存ptr，指向对象
	~auto_ptr() { delete ptr; } // 删除ptr指向的对象
private:
	T *ptr; // raw ptr to object
};

```
现在就可以放心大胆的把上面的代码重写成这样：

```c++
void processAdoptions(istream& dataSource)
{
	while (dataSource) {
		auto_ptr<ALA> pa(readALA(dataSource));
		pa->processAdoption();
	}
}

```
这里就不存在堆对象里面资源不被释放的情况，因为即使发生异常，局部对象pa的析构函数总是会被调用。


## Item M10：在构造函数中防止资源泄漏 ##

这个条款前面部分来回看了几遍，但知道了结局真是差点眼泪掉下来。

还是看书中的例子：

这是一个具有多媒体功能的通讯录程序。这个通讯录除了能存储通常的文字信息如姓名、地址、电话号码外，还能存储照片和声音。简化一下，假设只能存放姓名，照片，和声音，则这个类可以这样设计：

```c++
class Image					// 用于图像数据
{ 
public:
	Image(const string& imageDataFileName);	
};

class AudioClip				// 用于声音数据
{ 
public:
	AudioClip(const string& audioDataFileName);	
};

class BookEntry				// 通讯录中的条目
{ 
public:
	BookEntry(const string& name,const string& imageFileName = "",const string& audioClipFileName = "");
	~BookEntry();	
private:
	string theName;			// 人的姓名	
	Image *theImage;		// 他们的图像
	AudioClip *theAudioClip; // 他们的一段声音片段
};

```
这声明部分应该比较好理解，BookEntry是这个通讯录条目类，构造函数带有参数，对姓名、图像、和声音进行初始化。

下面看它的构造函数和析构函数，并分析其中可能出现的问题。

```c++
BookEntry::BookEntry(const string& name,const string& imageFileName,Const string& audioClipFileName)	: theName(name), theImage(0), theAudioClip(0)
{
	if (imageFileName != "") {
		theImage = new Image(imageFileName);
	}
	if (audioClipFileName != "") {
		theAudioClip = new AudioClip(audioClipFileName);
	}
}
BookEntry::~BookEntry()
{
	delete theImage;
	delete theAudioClip;
}

```
在正常情况下一切安好，但当出现异常时，就存在内存泄露的风险。
试想一下，

```c++
if (audioClipFileName != "") 
{
	theAudioClip = new AudioClip(audioClipFileName);
}

```
当在进行声音初始化时，发生了异常，可能是new 操作（申请内存），也可能是声音类的构造函数时发生了异常。那这个异常会被抛出，并且会传递到BookEntry构造函数的外部，那现在没有对这个异常进行捕捉，谁来负责删除已经建立好的theImage指向的对象？可能想的是析构函数来完成，但~BookEntry根本不会被调用。因为C++仅能删除被完全构造的对象，只有一个对象的构造函数完全运行完毕，这个对象才被完全地构造。

一点思考：我们自己定义类的构造函数，完成的功能应该绝对的简单，不应该主动抛出异常。

可能会想到在调用构造函数时主动的捕获这个异常并做处理：

```c++
void testBookEntryClass()
{
	BookEntry *pb = 0;
	try 
	{
		pb = new BookEntry("Addison-Wesley Publishing Company",	"One Jacob Way, Reading, MA 01867");		
	}
	catch (...)		// 捕获所有异常
	{ 
		delete pb; // 删除pb,当抛出异常时
		throw; // 传递异常给调用者
	}
	delete pb; // 正常删除pb
}

```
你会发现在BookEntry构造函数里为Image分配的内存仍旧被丢失了，这是因为如果new操作没有成功完成，程序不会对pb进行赋值操作。如果BookEntry的构造函数抛出一个异常，pb将是一个空值，所以在catch块中删除它除了让你自己感觉良好以外没有任何作用。

那更为主动，在构造函数中处理异常：

```c++
BookEntry::BookEntry(const string& name,const string& imageFileName,const string& audioClipFileName)
	: theName(name),theImage(0), theAudioClip(0)
{
	try { // 这try block是新加入的
		if (imageFileName != "") {
			theImage = new Image(imageFileName);
		}
		if (audioClipFileName != "") {
			theAudioClip = new AudioClip(audioClipFileName);
		}
	}
	catch (...) { // 捕获所有异常
		delete theImage; // 完成必要的清除代码
		delete theAudioClip;
		throw; // 继续传递异常
	}
}

```
这似乎可行，除了构造函数和析构函数里面有少量的重复代码。

但是它没有考虑到下面这种情况。假设我们略微改动一下设计，让theImage 和theAudioClip是常量（constant）指针类型：

```c++
class BookEntry				// 通讯录中的条目
{
public:
	BookEntry(const string& name, const string& imageFileName = "", const string& audioClipFileName = "");
	~BookEntry();
private:
	string theName;			// 人的姓名	
	Image * const theImage;		// 他们的图像
	AudioClip * const theAudioClip; // 他们的一段声音片段
};

```
一点思考：这里应该是指针常量，而不是常量指针，即指针指向的位置不能变，但它的内容能改变。
附上常量指针与指针常量的概念：
1.常量指针
定义：具有只能够读取内存中数据，却不能够修改内存中数据的属性的指针，称为指向常量的指针，简称常量指针。
声明：const int * p; int const * p; 
2.指针常量
定义：指针常量是指指针所指向的位置不能改变，即指针本身是一个常量，但是指针所指向的内容可以改变。
声明：int * const p=&a;

必须通过BookEntry构造函数的成员初始化表来初始化这样的指针，因为再也没有其它地方可以给const指针赋值。通常会这样初始化theImage和theAudioClip：

```c++
BookEntry::BookEntry(const string& name,const string& imageFileName,const string& audioClipFileName)
	: theName(name), theImage(imageFileName != ""? new Image(imageFileName)	: 0),
	theAudioClip(audioClipFileName != ""? new AudioClip(audioClipFileName): 0)
{
}

```
这又回到了老问题，即抛出了异常，但是没有进行处理。

替代方法是定义私有类成员函数，来完成初始化的工作，并捕获异常。

```c++
class BookEntry				// 通讯录中的条目
{
public:
	BookEntry(const string& name, const string& imageFileName = "", const string& audioClipFileName = "");
	~BookEntry();
private:
	string theName;			// 人的姓名	
	Image * const theImage;		// 他们的图像
	AudioClip * const theAudioClip; // 他们的一段声音片段

	Image * initImage(const string& imageFileName);
	AudioClip * initAudioClip(const string&	audioClipFileName);
};

BookEntry::BookEntry(const string& name,const string& imageFileName,const string& audioClipFileName)
	: theName(name),theImage(initImage(imageFileName)),theAudioClip(initAudioClip(audioClipFileName))
{}

// theImage 被首先初始化,所以即使这个初始化失败也
// 不用担心资源泄漏，这个函数不用进行异常处理。
Image * BookEntry::initImage(const string& imageFileName)
{
	if (imageFileName != "") return new Image(imageFileName);
	else return 0;
}

// theAudioClip被第二个初始化, 所以如果在theAudioClip
// 初始化过程中抛出异常，它必须确保theImage的资源被释放。
// 因此这个函数使用try...catch 
AudioClip * BookEntry::initAudioClip(const string&audioClipFileName)
{
	try 
	{
		if (audioClipFileName != "") 
		{
			return new AudioClip(audioClipFileName);
		}
		else return 0;
	}
	catch (...) 
	{
		delete theImage;
		throw;
	}
}

```
上面的程序的确不错，也解决了令我们头疼不已的问题。不过也有缺点，在原则上应该属于构造函数的代码却分散在几个函数里，这令我们很难维护。

所以最后、最优雅、最泪奔的结论是采用条款M9的建议，把theImage 和 theAudioClip指向的对象做为一个资源，被一些局部对象管理。这个解决方法建立在这样一个事实基础上：theImage 和theAudioClip是两个指针，指向动态分配的对象，因此当指针消失的时候，这些对象应该被删除。所以我们把theImage 和 theAudioClip raw指针类型改成对应的auto_ptr类型。

```c++
class BookEntry				// 通讯录中的条目
{
public:
	BookEntry(const string& name, const string& imageFileName = "", const string& audioClipFileName = "");
	~BookEntry();
private:
	string theName;			// 人的姓名	
	const auto_ptr<Image> theImage; // 它们现在是auto_ptr对象
	const auto_ptr<AudioClip> theAudioClip;
};

BookEntry::BookEntry(const string& name,const string& imageFileName,const string& audioClipFileName)	: theName(name),theImage(imageFileName != ""? new Image(imageFileName): 0),theAudioClip(audioClipFileName != ""? new AudioClip(audioClipFileName): 0)
{
}

BookEntry::~BookEntry() //--do nothing
{
}

```
如果在初始化theAudioClip时抛出异常，theImage已经是一个被完全构造的对象，所以它能被自动删除掉，就象theName一样。而且因为theImage 和 theAudioClip现在是包含在BookEntry中的对象，当BookEntry被删除时它们能被自动地删除。因此不需要手工删除它们所指向的对象。
综上所述，如果你用对应的auto_ptr对象替代指针成员变量，就可以防止构造函数在存在异常时发生资源泄漏，你也不用手工在析构函数中释放资源，并且你还能象以前使用非const指针一样使用const指针，给其赋值。

## Item M11：禁止异常信息（exceptions）传递到析构函数外 ##
在有两种情况下会调用析构函数。第一种是在正常情况下删除一个对象，例如对象超出了作用域或被显式地delete。第二种是异常传递的堆栈辗转开解（stack-unwinding）过程中，由异常处理系统删除一个对象。
在上述两种情况下，调用析构函数时异常可能处于激活状态也可能没有处于激活状态。遗憾的是没有办法在析构函数内部区分出这两种情况。因此在写析构函数时你必须保守地假设有异常被激活。因为如果在一个异常被激活的同时，析构函数也抛出异常，并导致程序控制权转移到析构函数外，C++将调用terminate函数。这个函数的作用正如其名字所表示的：它终止你程序的运行，而且是立即终止，甚至连局部对象都没有被释放。

看书中的例子，一个Session类用来跟踪在线计算机的sessions，析构函数的作用是注销会话。

```c++
class Session {
public:
	Session();
	~Session();
private:
	static void logCreation(Session *objAddr);
	static void logDestruction(Session *objAddr);
};

```
正常情况的析构函数是：

```c++
Session::~Session()
{
	logDestruction(this);
}

```
一切看上去很好，但是如果logDestruction抛出一个异常，会发生什么事呢？异常没有被Session的析构函数捕获住，所以它被传递到析构函数的调用者那里。但是如果析构函数本身的调用就是源自于某些其它异常的抛出，那么terminate函数将被自动调用，彻底终止你的程序。
正确的做法是这样的：

```c++
Session::~Session()
{
	try {
		logDestruction(this);
	}
	catch (...) 
	{}
}

```
在析构函数中捕获这个异常，阻止任何异常被传递到析构函数以外。但要注意不要在catch块儿里面做任何事情，避免引入新的异常。

## Item M12：理解“抛出一个异常”与“传递一个参数”或“调用一个虚函数”间的差异  ##
从语法上看，在函数里声明参数与在catch子句中声明参数几乎没有什么差别。
假设有一个类：

```c++
class Widget //一个类，具体是什么类在这里并不重要
{ 
};

```
常用的参数传递形式：

```c++
void f1(Widget w); // 一些函数，其参数分别为Widget, Widget&,或Widget* 类型
void f2(Widget& w); 
void f3(const Widget& w); 
void f4(Widget *pw);
void f5(const Widget *pw);
```
常用的catch子句中的声明参数：

```c++
//一些catch 子句，用来捕获异常，异常的类型为Widget, Widget&, 或Widget*
catch (Widget w)
{
}
catch (Widget& w)
{
}
catch (const Widget& w)
{

}
catch (Widget *pw)
{

}
catch (const Widget *pw)
{

}

```
那他们之间有什么相同点与不同点？
相同点：
传递参数与异常的途径相同，可以是传值、传递引用或传递指针。
不同点：
### 程序的控制权的差异###
你调用函数时，程序的控制权最终还会返回到函数的调用处，但是当你抛出一个异常时，控制权永远不会回到抛出异常的地方。

### 异常对象的拷贝形式的差异    ###
书中有这句话：C++规范要求被做为异常抛出的对象必须被复制。
什么意思？看两个抛出异常函数的例子。
有这样一个函数，参数类型是Widget，并抛出一个Widget类型的异常：

```c++
// 一个函数，从流中读值到Widget中
istream operator >> (istream& s, Widget& w);

//--版本1
void passAndThrowWidget()
{
	Widget localWidget;
	cin >> localWidget; //传递localWidget到 operator>>
	throw localWidget; // 抛出localWidget异常
}

```
注意函数中的最后一条语句：

```c++
throw localWidget
```
这里抛出的不是localWidget本身，而是它的一个拷贝。必须这么做，因为当localWidget离开了生存空间后，其析构函数将被调用。
即使被抛出的对象不会被释放，也会进行拷贝操作。例如如果passAndThrowWidget函数声明localWidget为静态变量（static）：

```c++
//--版本2
void passAndThrowWidget()
{
	static Widget localWidget;	// 现在是静态变量（static）;
								//一直存在至程序结束
	cin >> localWidget;			// 象以前那样运行
	throw localWidget;			// 仍将对localWidget
}

```
当抛出异常时仍将复制出localWidget的一个拷贝。这表示即使通过引用来捕获异常，也不能在catch块中修改localWidget；仅仅能修改localWidget的拷贝。
当异常对象被拷贝时，拷贝操作是由对象的拷贝构造函数完成的。该拷贝构造函数是对象的静态类型（static type）所对应类的拷贝构造函数，而不是对象的动态类型（dynamic type）对应类的拷贝构造函数。

```c++
class SpecialWidget : public Widget 
{ 
};

//--版本3
void passAndThrowWidget()
{
	SpecialWidget localSpecialWidget;
	Widget& rw = localSpecialWidget; // rw 引用SpecialWidget
	throw rw;
}

```
这里抛出的异常对象是Widget，即使rw引用的是一个SpecialWidget。因为rw的静态类型是Widget，而不是SpecialWidget。

异常是其它对象的拷贝，这个事实影响到你如何在catch块中再抛出一个异常。比如下面这两个catch块，乍一看好像一样：

```c++
catch (Widget& w) // 捕获Widget异常
{
	... // 处理异常
	throw; // 重新抛出异常，让它继续传
} 
catch (Widget& w) // 捕获Widget异常
{
	... // 处理异常
	throw w; // 传递被捕获异常的
} // 拷贝
```
但需要注意两者之间的差异：第一个块中重新抛出的是当前异常（current exception）,无论它是什么类型。特别是如果这个异常开始就是做为SpecialWidget类型抛出的，那么第一个块中传递出去的还是SpecialWidget异常，即使w的静态类型（static type）是Widget。这是因为重新抛出异常时没有进行拷贝操作。第二个catch块重新抛出的是新异常，类型总是Widget，因为w的静态类型（static type）是Widget。一般来说，你应该用throw来重新抛出当前的异常，因为这样不会改变被传递出去的异常类型，而且更有效率，因为不用生成一个新拷贝。

下面看捕获异常的三种形式：

```c++
catch (Widget w)		// 通过传值捕获异常
catch (Widget& w)		// 通过传递引用捕获 异常
catch (const Widget& w) //通过传递指向const的引用捕获异常

```
实际上通过传值捕获异常和通过传递引用捕获异常最主要的不同是拷贝异常对象的次数不同。传值方式捕获异常时，会对这个异常拷贝两次。一次是抛出时都会建立临时对象进行拷贝；一次是按值传递给catch块时，会再拷贝一次。而按引用捕获异常时只会进行前面的一次拷贝。


### 参数类型匹配过程的差异 ###

这一点很好理解，主要是注意写catch块时的顺序。主要是杜绝写出下面这种形式：

```c++
try
{
	...
}
catch (logic_error& ex) 
{
	// 这个catch块 将捕获所有的logic_error异常, 包括它的派生类
	... 
} 
catch (invalid_argument& ex) 
{ // 这个块永远不会被执行因为所有的invalid_argument异常都被上面的catch子句捕获
	...
}
```
与上面这种行为相反，当你调用一个虚拟函数时，被调用的函数位于与发出函数调用的对象的动态类型（dynamic type）最相近的类里。你可以这样说虚拟函数采用最优适合法，而异常处理采用的是最先适合法。

## Item M13：通过引用（reference）捕获异常 ##
这节给我印象较深的，不是为什么要通过引用捕获异常，而是为什么不能通过指针捕获异常。

一般大家都知道了不要使用一个指向局部变量的指针。可能很少会写出下面这种代码：

```c++
void someFunction()
{
	exception ex; // 局部异常对象;
				  // 当退出函数的生存空间时
				  // 这个对象将被释放。
	...
	throw &ex; // 抛出一个指针，指向已被释放的对象
	... 
}

```
但是有JAVA、C#习惯的人不经意间，可能会写出下面的这种代码：

```c++
void someFunction()
{
	...
    throw new exception; // 抛出一个指针，指向一个在堆中建立的对象
	... 
}

```
这没问题啊，避免了捕获一个指向已被释放对象的指针的问题。
但是catch子句的作者又面临一个令人头疼的问题：他们是否应该删除他们接受的指针？如果是在堆中建立的异常对象，那他们必须删除它，否则会造成资源泄漏。如果不是在堆中建立的异常对象，他们绝对不能删除它，否则程序的行为将不可预测。该如何做呢？
这是不可能知道的。一些被调用者可能会传递全局或静态对象的地址，另一些可能传递堆中建立的异常对象的地址。通过指针捕获异常，将遇到一个哈姆雷特式的难题：是删除还是不删除？这是一个难以回答的问题。所以你最好避开它。


## Item M14：审慎使用异常规格(exception specifications) ##


## Item M15：了解异常处理的系统开销 ##
这个对现代计算机来说，应该不是什么问题。

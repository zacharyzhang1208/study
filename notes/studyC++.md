# C++

## 基本内置类型

C++定义了一组表示整数，浮点数，单个字符和布尔值的算术类型，另外还定义了一种称为void的特殊类型  
算术类型的储存空间大小依机器而定。这里的大小指的是用来表示该类型的二进制位（bit）数  
C++标准规定了每个算术类型的最小储存空间，但是它并不阻止编译器使用更大的储存空间  
实际上，几乎所有的编译器使用的储存空间都比要求的要大  

值得注意的是，字符类型有两种： `char` 类型和 `wchar_t` 类型，后者用于扩展字符集（中文和日文等，这些字符不能用单个 `char` 表示）

## 引用  

### 引用介绍

引用是C++的一种机制，它就是某一变量（目标）的一个别名，对引用的操作与对变量直接操作完全一样  
引用的声明方法：类型标识符 &引用名=目标变量名；  
如：  
```C++
int a;
int &ra=a; //定义引用ra,它是变量a的引用，即别名
```
说明：  
&在此不是求地址运算，而是起标识作用  
类型标识符是指目标变量的类型  
声明引用时，必须同时对其进行初始化  
引用声明完毕后，相当于目标变量名有两个名称，即该目标原名称和引用名，且不能再把该引用名作为其他变量名的别名 `ra=1;` 等价于 `a=1;`  
对引用求地址，就是对目标变量求地址，&ra与&a相等，这其实是因为编译器会将 `&ra` 解释为： `&(*ra)` = `&a` ，所以 `&ra` 将得到 `&a` ，也验证了对所有的ra的操作，和对a的操作等同    
不能建立数组的引用，因为数组是一个由若干个元素所组成的集合，所以无法建立一个数组的别名  
  
### 引用底层实现  

最开始，我们写过一段代码来测试引用的地址，发现引用的地址和变量的地址是一样的  
但是，在后面对引用的底层分析后发现，它本身又存放的是变量的地址，即引用的值是地址，那么这不是很冲突吗？  
从底层实现来看，引用变量的确存放的是被引用对象的地址，只不过，对于高级程序员来说是透明的，编译器屏蔽了引用和指针的差别  
很多人误以为引用不占空间，其实这个说法不正确，引用是占空间的，它的大小与与指针的大小一致，引用的底层实现就是地址，说白了引用就是C++编译器提供的一种语法，隐藏了许多特性，C++标准制定者希望你把引用当成变量的别名来看待  

## C++异常处理  

C++ 通过 `throw` 语句和 `try...catch` 语句实现对异常的处理。throw 语句的语法如下：  
`throw 表达式`  
该语句拋出一个异常，异常是一个表达式，其值的类型可以是基本类型，也可以是类  
`throw` 语句要配合 `try` 和 `catch`语句使用：  
```C++
try 
{
	//...
	throw(Typename);
    //...
	}
catch(Typename t)
{
    //异常处理代码
}
//省略...
catch(Typename t)
{
    //异常处理代码
}
catch(...)//如果捕获到你所定义的异常类型之外的异常
{
	//异常处理代码
}
```

## 类类型

### 静态成员变量

类体中的数据成员的声明前加上static关键字，该数据成员就成为了该类的静态数据成员  
和其他数据成员一样，静态数据成员也遵守 `public` / `protected` / `private` 访问规则  
同时，静态数据成员还具有以下特点：   
静态数据成员实际上是类域中的全局变量，被所有该类成员共享，所以，静态数据成员的初始化不应该被放在类的定义中 ，而应该单独拿出来初始化   
例如我定义了一个ZWindow类，里面嵌套定义了一个ZWindowBase类    
```C++
class ZWindow
{
private:
	class ZWindowBase
	{
	public:
		static LPCWSTR getName() noexcept;
		static HINSTANCE getInstance() noexcept;
	private:
		ZWindowBase() noexcept;
		~ZWindowBase();
		ZWindowBase(const ZWindowBase&) = delete;
		ZWindowBase& operator=(const ZWindowBase&) = delete;
		static const LPCWSTR wndClassName;
		static ZWindowBase zWindowBase;
		HINSTANCE hInstance;
	};
public:
	ZWindow() noexcept;
	~ZWindow();
	ZWindow(int, int,LPCWSTR) noexcept;
	ZWindow(const ZWindow&) = delete;
private:
	static LRESULT CALLBACK handleMsgSetup(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam);
	static LRESULT CALLBACK handleMsgThunk(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam);
	LRESULT handleMsg(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam) noexcept;
private:
	int width = 0;
	int height = 0;
	HWND hWnd;
};
```
然后我在类的定义外部这样初始化这两个静态变量  
```C++
const LPCWSTR ZWindow::ZWindowBase::wndClassName = L"ZEngine";
ZWindow::ZWindowBase ZWindow::ZWindowBase::zWindowBase;
```

如果代码看不懂可以先看studyWindow学习笔记

### 静态成员函数

静态成员函数属于整个类所有  
可以通过类名和对象名访问public静态成员函数  
静态成员函数只能访问静态成员变量和静态成员函数

### mutable关键字  

顾名思义，类的成员前面如果加上mutable就表示它是可变的，甚至可以突破类const成员函数的限制   

### C++11新标准  

```C++
class A
{
private:
    int a=1;
};
```
一般而言，类中的数据成员在定义类的时候是不能初始化的，因为类本身只是一个类型，并不是一个实例  
但是，在C++11新标准中，上面的代码是可行的且与下面的代码等价    
```C++  
class A
{
public:
    A()
    {
        a=1;
    }
private:
     int a;
};
```
此外
C++11允许使用 `=delete` 将拷贝构造函数和拷贝赋值运算符定义为删除的函数  
在函数参数列表后加上 `=delete` 即表明这个函数是删除的函数   

与此类似的，我们可以使用 `=defalue` 告诉编译器生成默认的构造函数

## 指针

### 普通的指针

### this指针

#### this参数介绍

首先从类成员的隐式参数this说起

```C++
struct A
{   
    int value;
    A(int param)
    {
        value=param;
    }
    int getvalue()
    {
        return value;
    }
};
```
以上代码自定义了类A，该结构包含了自定义的构造函数和getvalue()成员函数。
```C++
A a(1);
int b = a.getvalue();
```
以上代码定义了A的对象a，并通过a调用了A的成员函数getvalue()  
需要注意的是，此时的getvalue()函数实际上是对象a调用的，返回的value值实际上隐式地返回a.value。

成员函数时通过一个名为this的隐式参数来访问调用它的那个对象，当通过某个对象调用成员函数时，实际上就是将该对象的地址赋值给隐式的this参数  
所以，类A的geta()函数可以看成是如下定义
```C++
int getvalue(this)
{
   return this->value;
}
```
而通过a调用getvalue()的代码实际上就是
```C++
A::getvalue(&a);
```
this参数是指针常量  
需要注意的是，this参数是指向对象地址的，因此其值不能被改变，但是可以通过this指针修改所指对象的值，也就是说this参数本身是一个常量  

#### 常量成员函数的引入  

this是指向类的对象的，这个对象是非常量的，当类的对象是常量时，就不能将其地址赋值给this  
```C++
const A a(1);
int b = a.getvalue();
```
此时，a是常量，通过常量调用成员函数时，无法将a的地址赋值给隐藏的this参数，报错信息是  
`error C2662: “int A::geta(void)”: 不能将“this”指针从“const A”转换为“A &”`

此时，需要将this声明为指向const的指针，但是由于this是隐式参数，无法直接指定，所以就要使用常量成员函数，即在成员函数的参数列表之后添加const  
它的本质是告诉编译器将这里的this声明为指向const的指针，也就是说在常量成员函数中的隐式参数this是一个指向常量的指针常量  
```C++
int getvalue() const
{
    return value;
}
```
此时，上面提到的代码就不会报错  

实际上，const修饰指针主要是为了告诉编译器能否通过指针修改所指对象的值，并不是说指针所指的对象必须是const类型  
因此，对于非常量的对象，也能调用常量成员函数    
```C++
A a(1);
int b = a.getvalue();
```
也因为常量成员函数中的隐式参数this是一个指向常量的指针常量，所以这个函数不能修改该类里面的任何成员的值（当然你可以读取值，但是不能修改，就是这个意思）  
<br />
<br />

### 智能指针  

智能指针定义在头文件 `<memory>` 中

## 容器

### 迭代器类型

实际上我们可以把迭代器理解成一种在C++泛型编程中使用的类型，它的作用和指针很像，但是功能比指针更强大  
它实际上是C++标准库中封装的一个类，目的是为了泛型算法的使用  
每种STL容器中都有自己的迭代器类型，如vector:  
`vector<int>::iterator iter`
这条语句定义了名为iter的变量，它的数据类型是由 `vector<int>` 定义的迭代器类型  

### 迭代器的begin和end操作  

STL的每种容器都定义了一对名为begin()和end()的函数用来返回迭代器  
如果容器不为空，begin操作返回的迭代器指向容器的第一个元素  
```C++
vector<int> ivec;
vector<int>::iterator iter = ivec.begin();   
```
此时iter指向ivec的第一个元素ivec[0]  
需要注意的是，如果使用 `ivec.end();` 函数初始化iter，则iter实际上指向一个不存在的元素，因为由end操作返回的是指向容器“末端元素的下一个”，我们把它称为“超出末端迭代器”（off-the-end iterator），表明它指向一个实际不存在的元素  
如果容器为空，则begin操作返回的迭代器与end操作返回的迭代器相同  

### 迭代器运算

STL的迭代器类型定义了一些操作适用于所有类型的迭代器  

解引用： `*iter` 获取迭代器所指向的元素，我们可以使用解引用操作符 `*` 来访问迭代器所指元素，如 `*iter = 0;` 意为把iter这个迭代器所指元素赋值为0  

解引用并获取成员： `iter->mem` 对iter解引用，获取指定元素中名为mem的成员，这句话等效于 `(*iter).mem`   

自增： `iter++` / `++iter` ，使迭代器自增指向容器中下一个元素

自减： `iter--` / `--iter` ，使迭代器自减指向容器中上一个元素  

判断相等/不等： `iter1 == iter2` / `iter1 != iter2` 如果两个迭代器指向同一个元素或都指向超出末端的下一个位置时相等，否则不等  

### size_type类型  

从逻辑上讲，size()成员函数应该似乎返回整型数值，但事实上，size操作返回是string::size_type类型的值  
许多库类型都定义了一些配套类型（companion type）  
通过这些配套类型，库函数的使用就与机器无关（machine-independent）  
size_type就是这些配套类型中的一种，它被定义并使用在string类和vector类中，与unsigned型（unsigned int获unsigned long）具有相同含义，而且保证足够大的能够存储任意的string对象和vector对象的长度  
string::size_type它在不同的机器上，长度是可以不同的，并非固定的长度，但只要你使用了这个类型，就使得你的程序适合这个机器。与实际机器匹配  
下面给出一个使用它的例子：  
```C++
for(vector<int>::size_type ix = 0; ix != ivec.size(); ++ix)
{
    ivec[ix] = 0;
}
```  

### 顺序容器  

我们之前已经使用过一种容器类型：STL的vector类型，这是一种顺序容器（sequential container）  
顺序容器将单一类型元素聚集起来，根据位置来存储访问这些元素。顺序容器的元素排列次序与元素值无关，是由元素添加到容器里的次序决定的  
STL标准库为我们定义了三种顺序容器类型：  

vector  支持快速随机访问
  
list  支持快速插入/删除

deque  双端队列
  
<br />
<br />

## C++强制类型转换  

### 四种强制类型转换命令 

四种命令都遵循以下格式：  
`cast-name<type>(expression)`  
具体分以下四种  

#### dynamic_cast  



#### const_cast  

顾名思义，该命令会转换掉表达式的const性质

#### static_cast

用于完成“相关类型”之间的转换  
如从 `double` 到 `int` 之间的转换  
对比特位的截断，补齐等操作，编译器会自动完成  

#### reinterpret_cast  

该命令为操作数的二进制位提供了低层次的重新解释  
它可以处理完全的“互不相关类型”之间的转换  
本质上完全是对操作数的比特位复制  
一般我们使用它是为了欺骗编译器以达到某种目的
下面给出一个使用它的例子
```C++
LRESULT WINAPI Window::HandleMsgSetup( HWND hWnd,UINT msg,WPARAM wParam,LPARAM lParam)
{
	if (msg == WM_NCACTIVATE)
	{
		const CREATESTRUCTW* const pCreate = reinterpret_cast<CREATESTRUCTW*>(lParam);
		Window* const pWnd = static_cast<Window*>(pCreate->lpCreateParams);
		SetWindowLongPtr(hWnd, GWLP_USERDATA, reinterpret_cast<LONG_PTR>(pWnd));
		SetWindowLongPtr(hWnd, GWLP_WNDPROC, reinterpret_cast<LONG_PTR>(&Window::HandleMsgThunk));
		return pWnd->HandleMsg(hWnd, msg, wParam, lParam);
	}
	return DefWindowProc(hWnd, msg, wParam, lParam);
}
LRESULT WINAPI Window::HandleMsgThunk(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
	Z_window* const pWnd = reinterpret_cast<Z_window*>(GetWindowLongPtr(hWnd, GWLP_USERDATA));
	return pWnd->HandleMsg(hWnd,msg,wParam,lParam);
}
LRESULT Window::HandleMsg(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
	switch (msg)
	{
	case WM_CLOSE:
		PostQuitMessage(0);
		return 0;
	}
	return DefWindowProc(hWnd, msg, wParam, lParam);
}
```
这是一个对WindowsAPI的类封装实现  
在这里我们把一个ClassWndEx中的lpWndProc赋值成了HandleMsgSetup  
当系统需要调用窗口过程函数时就会根据lpWwndproc储存的值寻找到我们定义的窗口过程函数  
WM_NCACREATE是一个Windows系统消息，在窗口创建之前系统会发出这个消息  
关于lpWndproc对应的函数的四个参数：  
`HWND hWnd` 是Windows系统中的窗体句柄  
`UINT msg` 是消息值
`WPARAM wParam` 和 `LPARAM lParam` 是附加参数，当消息类型不同时wParam和lParam所代表的意义也不同  
在上面这个例子中：  
当 `msg == NCACREATE` 时
`WPARAM wParam` - 不使用
`LPARAM lParam` - 是CREATESTRUCT结构类型的指针，其中的参数是CreateWindowEx中的12个参数  
CREATESTRUCT结构定义了传递给应用程序的窗口过程的初始化参数，它定义了窗口外观相关特性，CREATESTRUCT结构具有如下形式：  
``` C++
typedef struct tagCREATESTRUCT {
    DWORD DdwExStyle, //窗口的扩展风格
    LPCTSTR lpClassName, //指向注册类名的指针
    LPCTSTR lpWindowName, //指向窗口名称的指针
    DWORD dwStyle, //窗口风格
    int x, //窗口的水平位置
    int y, //窗口的垂直位置
    int nWidth, //窗口的宽度
    int nHeight, //窗口的高度
    HWND hWndParent, //父窗口的句柄
    HMENU hMenu, //菜单的句柄或是子窗口的标识符
    HINSTANCE hInstance, //应用程序实例的句柄
    LPVOID lpParam //指向窗口的创建数据
} CREATESTRUCT;
```
其中， `LPVOID lpCreateParams` 是一个创建窗口过程中可以用来储存自定义参数的变量  
在这里我们用它来储存我们定义的窗口类对象的指针  
```C++
HWND CreateWindowEx(
DWORD DdwExStyle, //窗口的扩展风格
LPCTSTR lpClassName, //指向注册类名的指针
LPCTSTR lpWindowName, //指向窗口名称的指针
DWORD dwStyle, //窗口风格
int x, //窗口的水平位置
int y, //窗口的垂直位置
int nWidth, //窗口的宽度
int nHeight, //窗口的高度
HWND hWndParent, //父窗口的句柄
HMENU hMenu, //菜单的句柄或是子窗口的标识符
HINSTANCE hInstance, //应用程序实例的句柄
LPVOID lpParam //指向窗口的创建数据
);
```
当我们创建一个窗口时，把指向对象的指针this传入最后一个参数lpParam即可  
这样，CreateWindowEx返回的窗口句柄指向的窗口结构中就会存下lpParam这个数据  

### 旧式强制类型转换

在引入命名的强制类型转换操作符之前，显示类型转换用圆括号将类型括起来实现  
`char *pc = (char*) ip`  
效果与 `reinterpret_cast` 相同，但是这种强制类型转换的可读性较差，难以跟踪错误转换  
所以在现代C++程序中建议使用C++提供的四种命名的强制类型转换命令  

## STL常用库

### 一

`#include<string>`  
字符串
### 二

`#include<vector>`  
不定长数组  
要使用不定长数组，先要创建数组对象    
`vector<类型名> 对象名` 如 `vector<int> vec`  
下面是几种重要的vector操作:  
使用`vec.push_back(XXX);`来向vec尾部添加元素  
使用`vec.push_front(XXX);`向头部添加元素  
使用`vec.begin();`和`vec.end();`来获取头和尾迭代器   
<br />
<br />

### 三

`#include<algorithm>`
里面有一个sort排序函数很常用，经常搭配vector使用  
`sort(vec.begin(),vec.end())`  
这样排序默认是升序的  
如果想要降序排序，可以重载Comp函数  
``` C++
bool Comp(const int &a,const int &b)
{
    return a>b;
}
```
其中 a表示前一个，b表示后一个  
你甚至可以通过重载Comp函数来实现任何自定义的比较规则进行排序  
如果Comp返回true，则a在b前面  ，否则b在a前面  
<br />
<br />



# Chapter 3 如何处理大型程序

[toc]

## **3.1 作用域与生存空间**

### 作用域与标识符的有效区域

不论是在程序的什么位置，使用到的每个标识符都会指向一个特定的实体：变量、函数、类型等。

**作用域**（scope）是程序的一部分，在其中名字有特定的含义。C++中大多数作用域都以花括号分隔。

同一个标识符在不同的作用域中可能指向不同的实体。**标识符的有效区域始于标识符的声明语句，以声明语句所在的作用域末端为结束**。

```cpp
#include <iostream>

int main() {
	int sum = 0;
	for (int ind = 0; ind <= 10; ind++) {
		sum += ind;
	}
	std::cout << "Sum of 1 to 10 inclusive is " << sum << std::endl;
	return 0;
}
```

上述程序使用了三个标识符，分别为`main`,`sum`,`val`。

- 标识符`main`定义于所有花括号之外，他和其他大多数定义在函数体之外的名字一样拥有**全局作用域**。一旦声明之后，全局作用域内的名字在整个程序范围内都可以被使用。
- `sum`定义于`main`函数所限定的定义域之内，从声明`sum`开始直到`main`函数结束为止都可以访问它，但是在函数块外就无法访问了。因此，我们说`sum`具有**块作用域**。同样的`val`仅在`for`循环中可以访问。

### 嵌套作用域

作用域可以彼此包含。被包含的作用域被称为**内层作用域**，包含着别的作用域的作用域被称为**外层作用域**。

作用域中一旦声明了某个标识符，他所嵌套的所有作用域中都可以访问到该名字。

允许在内层作用域中重新定义外层作用域中已存在的名字，此时在内层作用域中该标识符表示的原对象将变得不可访问。

除此之外，还可以使用作用域操作符`::`访问某个特定类或命名空间，以覆盖默认的作用域规则。特殊的，当作用域左侧为空时，将访问全局作用域。

```cpp
#include <iostream>
using std::cout; //访问STL命名空间
using std::endl; //访问STL命名空间

int sum = 20;
int main() {
	int sum = 0; //覆盖全局变量sum
	for (int i = 0; i <= 10; i++) {
		sum += i;
	}
	::sum += sum; //全局变量sum += main中的sum变量
	cout << "Sum at main() is " << sum
		<< " and Sum with global scope is " << ::sum << endl; //使用 :: 访问全局作用域
}
```

## **3.2 分离式编译**

C++支持程序分离式编译，以应对复杂的程序。通常来说，**C++程序包含头文件`.h`和源文件`.cpp`两部分**。

- 类一般不定义在函数体内，当在函数体外定义类时，在各个指定的源文件中可能只有一处该类的定义。而且，如果要在不同文件中使用同一个类，类的定义就必须保持一致。基于上述原因，**类通常被定义在头文件中，并且类所在的头文件与类的名字相同**。例如，库类型`string`在名为`string`的头文件中定义。**类成员函数的定义一般放在与类名字相同的源文件中**。
- 除此之外，`const`、`constexpr`变量等，即**只能被定义一次，但需要在所有文件中使用的实体，也被定义在头文件中**。

> 使用实例
>
> `Contact.h`文件：
>
> ```cpp
> #ifndef CONTACT_H
> #define CONTACT_H
> #include <iostream>
> #include <fstream>
> #include <string>
> using std::string;
> using std::istream;
> using std::ostream;
> using std::ifstream;
> using std::ofstream;
> 
> class Contact
> {
> private:
> 	string name;			//联系人姓名
> 	string phoneNumber;		//联系人电话
> 	string mailAddress;		//联系人邮箱
> 	bool gender;			//联系人性别，1为男，0为女
> 	unsigned int age;		//联系人年龄
> 	string address;			//联系人地址
> public:
> 	Contact();
> 	Contact(string _name, string _phoneNumber, string _mailAddress, int _gender, unsigned int _age, string _address);
> 
> 	friend istream& operator>> (istream& is, Contact& item);
> 	friend ostream& operator<< (ostream& os, const Contact& item);
> 
> 	friend ifstream& operator>> (ifstream& ifs, Contact& item);
> 	friend ofstream& operator<< (ofstream& ofs, const Contact& item);
> 
> 	friend bool operator== (const Contact& item, string str);	//for search.
> };
> #endif // !CONTACT_H
> 
> 
> ```
>
> `Contact.cpp`文件：
>
> ```cpp
> #include "Contact.h"
> #include <iostream>
> using std::cout;
> using std::endl;
> 
> Contact::Contact(){
> 	//...
> }
> 
> Contact::Contact(string _name, string _phoneNumber, string _mailAddress, int _gender, unsigned int _age, string _address){
> 	//...
> }
> 
> istream& operator>>(istream& is, Contact& item){
> 	//...
> }
> 
> ostream& operator<<(ostream& os, const Contact& item){
> 	//...
> }
> 
> ifstream& operator>>(ifstream& ifs, Contact& item){
> 	//...
> }
> 
> ofstream& operator<<(ofstream& ofs, const Contact& item){
> 	//...
> }
> 
> bool operator==(const Contact& item, string str){
> 	//...
> }
> ```

## **3.3 预处理器**

### 头文件保护符

通常来说，一个头文件中定义的类往往要用到多个其他类中的成员变量或函数。因此，（尤其是在大型程序中），头文件的重复包含非常常见。但是，对于`const`对象等来说，在整个程序中只能被定义一次。重复包含含有此种对象的头文件会导致编译错误。**为了确保头文件多次包含后仍能安全工作，我们使用头文件保护符**。其形式如下：

```cpp
#ifndef CLASS_H
#define CLASS_H
class Class {
	//...
};
#endif // !CLASS_H
```

一般来说，我们定义的宏应该和类名相吻合，以便于在大型程序中区分。**通常的做法是将类所在的头文件全部大写，并将标点置换为下划线，如果头文件名包含多个单词，则两个单词之间也用下划线分隔**。

另一种防止头文件被重复包含的方法是使用`#progma`流程控制。上面的代码也可以写为

```cpp
#progma once
class Class {
	//...
};
```

但是，该种方式并不常见。通常我们应该选择第一种方式保护头文件不被重复包含。

### 活动预处理模块与非活动预处理模块

```cpp

#define USING_MAIN_1
//#define USING_MAIN_2
//#define USING_MAIN_3

#ifdef USING_MAIN_1	//活动预处理模块
	//...
#endif // USING_MAIN_1

#ifdef USING_MAIN_2 //非活动预处理模块

#endif // USING_MAIN_2

#ifdef USING_MAIN_3 //非活动预处理模块

#endif // USING_MAIN_3
```

处理活动模块主要为了在编译时进行有选择的挑选，**通常用来调试一些指定的代码，以达到版本控制的功能，特定模块代码调试的功能**。以下是常见的一些使用实例。

```cpp
#define DEBUG_MODE

#ifdef DEBUG_MODE
void debug() {
	cout << __DATE__ << endl << __TIME__ << endl;
}
#else
void debug() {
	return;
}
#endif // DEBUG_MODE
```

```cpp
#define VIRSION_2

#ifdef VIRSION_2
void funciton() {
	//...
}
#endif // VIRSION_2

#ifdef VIRSION_1
void function() {

}
#endif // VIRSION_1
```

### C++ 中的预定义宏

C++ 提供了下表所示的一些预定义宏：

| 宏         | 描述                                                         |
| :--------- | :----------------------------------------------------------- |
| `__LINE__` | 这会在程序编译时包含当前行号。                               |
| `__FILE__` | 这会在程序编译时包含当前文件名。                             |
| `__DATE__` | 这会包含一个形式为` month/day/year` 的字符串，它表示把源文件转换为目标代码的日期。 |
| `__TIME__` | 这会包含一个形式为` hour:minute:second `的字符串，它表示程序被编译的时间。 |

### # 和 ## 运算符

`define`预处理器的一般形式为

```cpp
#define macro-name replacement-text 
```

**`#` 运算符会把 `replacement-text `令牌转换为用引号引起来的字符串。**

请看下面的宏定义：

```cpp
#include <iostream>
using namespace std;
 
#define MKSTR( x ) #x
 
int main ()
{
    cout << MKSTR(HELLO C++) << endl;
 
    return 0;
}
```

当上面的代码被编译和执行时，它会产生下列结果：

```
HELLO C++
```

让我们来看看它是如何工作的。不难理解，C++ 预处理器把下面这行：

```
cout << MKSTR(HELLO C++) << endl;
```

转换成了：

```
cout << "HELLO C++" << endl;
```

**`##` 运算符用于连接两个令牌。下面是一个实例：**

```
#define CONCAT( x, y )  x ## y
```

当 `CONCAT` 出现在程序中时，它的参数会被连接起来，并用来取代宏。例如，程序中` CONCAT(HELLO, C++) `会被替换为 `"HELLO C++"`，如下面实例所示。

```cpp
#include <iostream>
using namespace std;
 
#define concat(a, b) a ## b
int main()
{
   int xy = 100;
   
   cout << concat(x, y);
   return 0;
}
```

当上面的代码被编译和执行时，它会产生下列结果：

```
100
```

让我们来看看它是如何工作的。不难理解，C++ 预处理器把下面这行：

```
cout << concat(x, y);
```

转换成了：

```
cout << xy;
```

### 参数宏

您可以使用 `#define` 来定义一个带有参数的宏，如下所示：

```cpp
#include <iostream>
using namespace std;
 
#define MIN(a,b) (a<b ? a : b)
 
int main ()
{
   int i, j;
   i = 100;
   j = 30;
   cout <<"较小的值为：" << MIN(i, j) << endl;
 
    return 0;
}
```

当上面的代码被编译和执行时，它会产生下列结果：

```
较小的值为：30
```

### 一些使用预处理器的例子

其他类型的条件编译。

```cpp
#define SOMETHING 50
#if SOMETHING >= 100
//...
#else
//...
#endif
```

```cpp
#if (defined DEBUG) && (defined SOMETHING)
//...
#endif
```

> **`defined SOMETHING`返回`true`当`SOMETHING`宏被定义。**

利用预处理器处理命名空间。

```cpp
#ifdef SOMETHING
namespace space1{
#endif
//...
#ifdef SOMETHING
}//space1
#endif
```

## **3.4 命名空间与`using`声明**

详见[命名空间](file:///E:/CPPGithub/Cpp/HtmlDoc/namespace.mhtml)。
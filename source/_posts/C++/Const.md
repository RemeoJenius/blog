---
title: Const
date: 2017-05-05 12:50:51
tags:
categories: C++
---
# Const
在C语言当中的，这个变量加上一个Const关键字之后，这个变量被初始化之后呢，就不能赋值。
![Alt text](/images/C++/const1.png)
但是由于指针的存在，就出现一个问题，就是这个const是修饰谁的，比如:
``` c++
#include <iostream>
using namespace std;

int main(int argc, char const *argv[])
{
	int i = 10;
	const int *p = &i;
	cout<<*p<<endl;
	*p = 2;
	return 0;
}
```
会在第9行报错，error: read-only variable is not assignable
说这个变量是只读的，变量是不能被赋值的。
说明这个指针所指的这个对象是const的是不能被修改的，但是指针变量还是可以修改的。

``` C++
#include <iostream>
using namespace std;

int main(int argc, char const *argv[])
{
	int i = 10;
	const int *p = &i;
	cout<<*p<<endl;
	p++;
	return 0;
}
```
没有报错。我们再看两个例子：
``` C++
#include <iostream>
using namespace std;

int main(int argc, char const *argv[])
{
	int i = 10;
	int *const p = &i;
	cout<<*p<<endl;
	*p = 2;
	cout<<*p<<endl;
	return 0;
}
```
这个没有报错，输出是：
10
2
```C++
#include <iostream>
using namespace std;

int main(int argc, char const *argv[])
{
	int i = 10;
	int *const p = &i;
	cout<<*p<<endl;
	p++;
	cout<<*p<<endl;
	return 0;
}
```
这个报错了，error: cannot assign to variable 'p' with const-qualified type 'int *const'
说不能对变量p赋值，因为这个指针p是被const修饰的，是类型'int *const'
这个指针p所指向的那个变量的值被修改了，但是p不能被修改，说明这个const修饰的是指针变量p，所以指针变量p不能被修改。
这就有一个问题了，怎么区分呢？其实有一个小技巧
```C++
const int *p = &i; // 看这个*号，const在星号左边说明是指针所指的那个变量是const的
int *const p = &i; // const在星号右边说明这个指针p是const的
```
当然你可以都让它们是const的，让指针是const的同时，让指针所指的那个变量（内存空间）也都是const
``` C++
#include <iostream>
using namespace std;

int main(int argc, char const *argv[])
{
	int i = 10;
	const int *const p = &i;
	p++;  // 这里会有错，error: cannot assign to variable 'p' with const-qualified type 'int *const'
	cout<<*p<<endl;
	*p = 2; // 这里会有错，error: read-only variable is not assignable
	cout<<*p<<endl;
	return 0;
}
```
这样两个变量都是const的了
接下来再试一件事情
```C++
#include <iostream>
using namespace std;

int main(int argc, char const *argv[])
{
	char *s = "hello world";
	cout<<s<<endl;
	s[0] = 'B';
	cout<<s<<endl;
	return 0;
}
```
我 g++ 了一下，有一个warning:
![Alt text](/images/C++/const2.png)
如果我先无视这个warning ./a.out 一下
![Alt text](/images/C++/const3.png)
它输出了这个 hello world 之后程序异常终止了。
前面的那个warning 的意思是说：conversion from string literal to 'char *' is deprecated
说这事已经过时了，这个意思是说你想将"hello world"这个字符串交给char *是不对的，做了修改之后程序就崩溃了。
实际上事情是这样的，s在栈里面，因为是本地变量（本地变量和函数的参数列表的变量都在栈里面）s是一个指针，它指向了一块内存，也就是说"hello world"是一块内存，内存又分为三种不同的地方，本地变量是放在，栈里面，new出来的东西是放在堆里面，然后全局变量在一个全局数据区里面。而全局变量里面的这种常量，是在代码段里的，因为"hello world"是一个常量，所以尽管我们没有在s前面加上const，实际上它是const的。它被放在了代码段里面，而代码段是不可写的。现在说有的操作系统，都是葱花1970年代出得那个Unix，现代操作系统的基本理论是从那里来的，所以不管外表是什么样，里面东西是一样的。你的程序运行起来之后，你的程序会从操作系统得到一块内存，这块内存当然是虚拟的，但是这块内存是独立的，每个进程，每个程序都有独立的内存。每块内存从0开始，在计算机的内部有一个计算机的硬件，MMU内存管理单元。由它来管这件事情，它会划分，从0开始到0x000xxx是代码段，如果你去试图去写这个代码段，就会有错：Bus error，实际上是MMU的地方出了错。
这是这件事情，假如：
```C++
#include <iostream>
using namespace std;

int main(int argc, char const *argv[])
{
	// char *s = "hello world";
	char s[] = "hello world";
	cout<<s<<endl;
	s[0] = 'B';
	cout<<s<<endl;
	return 0;
}
```
输出：
hello world
Bello world
没有warning、没有error
说明了 char[] s = "hello world";
做了一件事，将代码段里的"hello world" 复制到栈里面了，存在本地变量里面。
可以写一个程序来验证。
``` C++
#include <stdio.h>


int main(int argc, char const *argv[])
{
	const char *s1 = "hello world";
	char s2[] = "hello world";
	printf("s1  =%p\n",s1);
	printf("s2  =%p\n",s2);
    printf("main=%p\n",main;
	return 0;
}
```
之后我们编译运行一下输出：
![Alt text](/images/C++/const4.png)
明显地址是不同的，char *s1 是比较靠前的地址，所以它在代码段里的
同时也看了main函数的地址，也是比较靠前的

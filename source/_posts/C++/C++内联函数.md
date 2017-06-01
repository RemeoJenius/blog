---
title: C++内联函数
date: 2017-04-25 12:47:36
tags:
categories: C++
---
# Overhead for a function call
- **the processing time required by a device prior to the execution of a command**:
 - **Push parameters**: 将函数的参数推到栈里面去

 - **Push return address**:把返回地址推到栈里面去
 - **Prepare return values**:准备返回的值
 - **Pop all pushed**:把所有推进栈里面的东西都赶出来
# inline Function
上述这么多额外的事情，可以用一个手段来避免产生这些额外的开销
C++给我们提供了一种方式，就是内联函数。内联函数我们在调用它的时候，并不是会产生那些动作（push,pop,return,call），
而是把那个函数的代码嵌入到那个地方去，但是仍然保持函数的独立性（函数独有的空间，对参数类型的检查等）。
当然，如果你的函数是inline的，这个函数就不会出现在最终的可执行代码里面。它只会出现在编译器里面。
如果你想写函数是inline的话，在.h和.cpp必须都要在函数前面写上inline。实际上.h和.cpp是写给不同人看的，
.cpp是用来产生那个函数的，.h是用来给调用这个函数的地方看的。.cpp里面有没有inline是让编译器生成不生成这个函数的，有inline就不生成这个函数。头文件里面的inline是告诉，这个地方你不能生成调用它的那个代码，要把函数体拆进来。
## 代码 main.cpp
``` C++
#include "a.h"

int main(int argc, char const *argv[])
{
	f(10,10);
	return 0;
}
```
## 代码 a.cpp
``` C++
#include <iostream>
using namespace std;
#include "a.h"

inline void f(int i,int j){
	cout<<i<<"  "<<j<<endl;

}
```
## 代码 a.h
``` C++
inline void f(int i,int j);
```
出错信息：
![Alt text](/images/C++/inline1.png)
这个warning说这个函数是inline的但是却没有定义。linker说main.cpp要用的这个f函数不存在。
我们可以cpp main.cpp 看一下。cpp 这个命令是编译预处理器。
![Alt text](/images/C++/inline2.png)
编译器在编译的时候是按照，每一个编译单元去编译的，也就是每一个文件去编译。
通过cpp可以看到并没有定义函数体，所以是没有办法插进去的。所以只能放弃将f函数做内联。
而a.o中没有产生任何代码因为它是inline的，所以a.o中没有f函数的痕迹在的，当link的时候就出现问题了
main.cpp说我要个f函数，然后a.cpp里又没有f函数。所以我们应该把f函数的body放在a.h里面去。
## 代码 a.h
``` c++
#include <iostream>
using namespace std;
inline void f(int i,int j);
inline void f(int i,int j){
	cout<<i<<"  "<<j<<endl;

}
```
编译过了，运行也正常了
所以这是一种权衡（tradeoff）
有inline之后，对于之前的用法就会不一样，之前的用法是说你的每一个类应该会有对应的.cpp和对应的一个.h，.h里面放的是声明，而.cpp里放的是定义。
而inline函数这种函数的定义就不再是个定义，而是个声明，因此它不能放.cpp，只放在.h就够了。所以对应上面的代码来说，不需要有a.cpp有a.h就够了.h把所有inline的函数的body都放进去，就可以了。
对于inline函数而言，用inline函数就意味着，在这个函数被调用的时候，函数的代码会被插入到被调用的地方，这也意味着你的代码会变长程序会变大，因此它会牺牲代码的空间但是它会降低调用函数的overhead（额外的开销），这是典型的以空间换时间的策略。
这种方式比C的宏要好，C的宏不会进行类型检查，而inline会进行类型检查，所以inline比宏要好要安全。
## 注意
但是编译器也会做一个这样的事情，它如果发现你的inline函数过于巨大它可能会拒绝这个inline函数作为inline（inline函数内部做了很复杂的循环或者内部做了递归）
如果是成员函数
例如这个
``` C++
#include <iostream>
using namespace std;
#include "a.h"

class A{
	public :
		void f(){cout<<"f()"<<endl;};
};

```
f函数体直接写在后面了，这样就不用在写.cpp了只要写一个.h就可以了
放在头文件里
``` C++
#include <iostream>
using namespace std;
#include "a.h"

class A{
	public :
		void f();
};

inline void A::f(){
	cout<<"f()"<<endl;
}
```
所以加了inline这个函数就不是定义而是声明，然后他不在class里面，而在class外面。
这样做的好处是，class会很清爽。

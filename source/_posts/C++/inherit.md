---
title: C++继承
date: 2017-04-23 13:35:44
tags:
categories: C++
---
# name hiding
先看一下例子
``` c++
#include <iostream>
using namespace std;

class A{
	private :
		int i;
	public :
		 A():i(10){cout<<"A::A()"<<endl;}
		int getI(){return this->i;}
		int getI(int i){return i;}
};
class B:public A{
	public :
		int getI(){return 20;}
};
int main(){
	B b;
	cout<<b.getI()<<endl;
	return 0;
}
```
结果输出：
A::A()
20
这个理解上没有什么问题，这个所有的OOP的语言都有，这个叫做override，函数的重写或者叫函数的复写。
接下来再看一个例子
``` C++
#include <iostream>
using namespace std;

class A{
	private :
		int i;
	public :
		 A():i(10){cout<<"A::A()"<<endl;}
		int getI(){return this->i;}
		int getI(int i){return i;}
};
class B:public A{
	public :
		int getI(){return 20;}
};
int main(){
	B b;
	cout<<b.getI()<<endl;
	cout<<b.getI(1)<<endl;
	return 0;
}
```
报错：
![Alt text](/images/namehiding.png)
意思大概是说，函数中的参数过多，应该是0个，但是现在有一个参数，你可能想调用的是那个无参数的函数
相当于类B中没有带一个参数的getI函数。不是说B类继承A类，之后就是把所有的A类当中，不管是private的还是public的都继承过来了嘛，为什么A类有带一个参数getI但是对B类的对象调用会报错。这个是因为C++所特有的，除了C++其他的OOP语言都不是这么干的，这叫name hiding。对于C++来说，如果父类当中有overload的函数，在子类当中你出现和父类相同的函数（名字，参数列表都一样）那么子类当中就只有那一个函数了，父类的那些就都被隐藏掉了。这是因为在C++，B类中的getI函数和A类中的getI函数是没有任何关系的，其他的OOP语言会构成一种关系，叫override
# Function overloading
overload 是指的我的一些函数可以有相同的函数名，然后他们有不同的参数列表（这个是参数列表的个数和类型会不一样）。
``` C++
#include <iostream>
using namespace std;

class A{
	private :
		int i;
	public :
		A():i(0){}
		void print(){cout<< i <<endl;}
		void print(long i){cout<< i <<endl;}
		void print(int i){cout<< i <<endl;}
};

int main(){
	A a;
	a.print();
	a.print(155L);
	a.print(15);
	return 0;
}

```
这个就是overload（重载）
当然，如果一个函数的名称相同，参数表相同而返回值类型不同，构不构成overload呢？
这个显然不行的，因为这样编译器无从分辨，你到底要调用哪个函数。
有的人会说我可以这样
``` C++
int a = f();
double a = f();
```
通过返回值要交给哪个类型的变量来让编译器来判断，我要调那个函数。
但是问题在于你还可以这样写
``` C++
f();
```
这样写编译器就不会知道。
# default value
很多编程语言都支持default value
``` C++
#include <iostream>
using namespace std;

class A{
	private :
		int i;
	public :
		A():i(0){}
		void print(int i=10){cout<< i <<endl;}

};

int main(){
	A a;
	a.print();
	return 0;
}

```
不过在这里的忠告是，尽量不用default value ，当然用default value在一些情况下你可以少打一些字。
但是对软件工程来说，一般情况下凡事让你少打字的情况都是不好的。因为default value会造成你阅读上的困难。
当你看到我在调用print的函数的时候你会以为他只是一个没有参数的函数，但是你找不到。而且default value很不安全，它不一定是设计者的意图。

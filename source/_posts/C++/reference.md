---
title: reference
date: 2017-05-09 08:32:08
tags:
categories: C++
---
# Declaring references
reference中文我们叫做引用，C++复杂就复杂在于，它提供了太多的内存模型。或者说它提供了太多的两种东西，第一种东西是它提供了太多的可以放对象的地方，C++的对象可以放在栈里面，可以放在堆里面，可以放在全局数据区里面，这是第一种复杂。第二种复杂它提供了太多的让你可以访问对象的方式，你可以直接掌握那个对象，也就是说你的变量里面可以放的是对象，你可以通过指针去访问一个对象，你可以通过引用去访问一个对象。所以我们有三种放对象的地方，我们有三种访问对象的方式，把它们乘起来，我们有九种组合。
![Alt text](/images/C++/reference1.png)
``` C++
#include <iostream>
using namespace std;

int main(int argc, char const *argv[])
{
	char c;
	char* p = &c;
	char& r =  c;
	return 0;
}
```
这个时候我们用r也就是在用c，reference也可以叫alias（别名），而现在的r就是c的一个alias别名。
如果是在参数表里面或者是作为成员变量，那个时候它才没有后面的那个初始化。
![Alt text](/images/C++/reference2.png)
我们对reference有两种理解，第一种字面上的理解就是这个y就是x的另外一种名字，以后任何地方出现y你都可以将它替换成x，因此当y=18发生的时候，就是说x=18，然后输出x的值，那个值就是18了。
## Rules of references
定义reference的时候必须要初始化。
``` C++
void f(int& x);
f(y);   // 给参数列表初始化，但是如果在f函数中改变了x的值，那么外面的y的值也会改变，这就很邪恶了，
        // 因为你调用的时候不知道，传的是那个对象还是reference
```
![Alt text](/images/C++/reference3.png)
reference需要一个地方去放它，i*3有结果但是没有location，没有一个对方去放它。
![Alt text](/images/C++/reference4.png)
References和Pointers做了一对比。
实际上reference是通过pointer的方式实现的，reference就是const的pointer，当年BS为了让代码少一点*，所以做出来reference来。
在这里讲一点java，java的内存模型要比C++简单很多。java对象只有一个地方可以放，所有的对象只能放在堆里面，然后只有一种方式可以去访问那个对象，就是通过指针，但是它只有通过指针去访问对象，所以它可以把指针的那个*号取消掉，然后它对别人说我这个不叫指针叫做引用。但其实它和C++的引用是不一样的，它更像是C++的指针，因为引用是不能做引用之间的赋值的，java是可以的，所以java尽管它的文献里都叫做引用，但实际上它是指针。但是java的指针和C++的指针区别在哪里。一、外形上没有了那个*号；二、那个指针是不能做计算的。C/C++的指针最大的问题是可以做计算的，做完计算之后就不知道在哪了，你就不知道它指的是不是对的，java把这个取消掉了。java就把内存模型做的简单点，事实上内存模型并不需要这么复杂，C++能做的是java也能做，唯一一个说C++能做而java不能做的事情，不是因为复杂的内存模型，而是C++中C的特性，因为它可以直接去访问底层的那些东西。

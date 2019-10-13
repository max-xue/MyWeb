---
title: C++研究之---virtual 解析
categories:
  - C++
date: 2018-08-22 10:50:22
comments: true
tags:
  - c++
---

OO编程有三大特性：封装，继承，多态 在C++中，从绑定时间来看，可以分成静态多态和动态多态，也称为编译期多态和运行期多态。静态多态即函数重载，在同一类内相同的函数名，不同的参数列表。相对简单，现在重点分析动态多态。 **虚函数** C++中的虚函数的作用主要是实现了多态的机制。关于多态，简而言之就是用父类型别的指针指向其子类的实例，然后通过父类的指针调用实际子类的成员函数。这种技术可以让父类的指针有“多种形态”，这是一种泛型技术。所谓泛型技术，说白了就是试图使用不变的代码来实现可变的算法。比如：模板技术，RTTI技术，虚函数技术，要么是试图做到在编译时决议，要么试图做到运行时决议。 通过以上这段话，我们知道动态多态主要是通过虚函数实现，而虚函数(Virtual Function)则是通过一张虚函数表(Virtual Table)来实现的。简称为V-Table。在这个表中，主是要一个类的虚函数的地址表，这张表解决了继承、覆盖的问题。这样，在有虚函数的类的实例中这个表被分配在了这个实例的内存中，所以，当我们用父类的指针来操作一个子类的时候，这张虚函数表就显得由为重要了，它就像一个地图一样，指明了实际所应该调用的函数。我们通过一些代码来了解这个概念：

typedef void(*pFun)(void);//函数指针
 
class Base {
public:
 
    virtual void f() { cout << "Base::f" << endl; }
 
    virtual void g() { cout << "Base::g" << endl; }
 
    virtual void h() { cout << "Base::h" << endl; }
};
 
void FunTest()
{
    Base b;
 
    cout << "虚函数表地址：" << (int*)(&b) << endl;
 
    pFun* fun = (pFun*)*(int*)&b;
 
    while (*fun) {
        (*fun)();
        fun++;
    }
}
 
int main()
{
    FunTest();
    getchar();
    
    return 0;
}

在VS2013编译器win32的测试结果为：

![这里写图片描述](http://www.2cto.com/uploadfile/Collfiles/20161101/20161101092309987.png)

我们来看看虚函数表的地址0x00DF820里面存了什么?

![这里写图片描述](http://www.2cto.com/uploadfile/Collfiles/20161101/20161101092309989.png)

通过上图我们知道，通过Base类实例化的对象b里面(有3个虚函数)有一个指向虚函数表的指针，也就是我们上面的0x00DF820，而在这个虚函数表中，分别存了3个虚函数的地址，我们通过函数指针fun可以访问到这些函数，因此就得到我们的输出结果了。通过sizeof(Base)=4也说明此时b对象仅仅存有一个指针，指向虚函数表。 所以就得到了我们的对象模型：

![这里写图片描述](http://www.2cto.com/uploadfile/Collfiles/20161101/20161101092309990.png)

注意：在上面这个图中，虚函数表的最后多加了一个结点，这是虚函数表的结束结点，就像字符串的结束符“\\0”一样，其标志了虚函数表的结束，也就是我们这里虚函数表的最后地址存的全是0。注意这个结束标志的值在不同的编译器下是不同的。 **一般继承(无虚函数覆盖)**

![这里写图片描述](http://www.2cto.com/uploadfile/Collfiles/20161101/20161101092310991.png)

注意到： 1\. 虚函数按照其声明顺序放于表中。 2. 父类的虚函数在子类的虚函数前面。 **一般继承(有虚函数覆盖)**

![这里写图片描述](http://www.2cto.com/uploadfile/Collfiles/20161101/20161101092310992.png)

注意到： 1\. 覆盖的f()函数被放到了虚表中原来父类虚函数的位置。 2. 没有被覆盖的函数依旧。 因此对于程序：

Base *b = new Derive();

b->f();

  由b所指的内存中的虚函数表的f()的位置已经被Derive::f()函数地址所取代，于是在实际调用发生时，是Derive::f()被调用了。这就实现了多态。 **多重继承(无虚函数覆盖)**

![这里写图片描述](http://www.2cto.com/uploadfile/Collfiles/20161101/20161101092310993.png)

注意到： 1\. 对于实例Derived d的对象，每个父类都有存有一个指针，指向对应的虚函数表。 2. 子类的成员函数被放到了第一个父类的表中。(第一个父类是按照声明顺序来判断的) **多重继承(有虚函数覆盖)**

![这里写图片描述](http://www.2cto.com/uploadfile/Collfiles/20161101/20161101092310994.png)

我们可以写一段代码对上图进行测试：

typedef void(*pFun)(void);
 
class Base1 {
 
public:
 
    virtual void f() { cout << "Base1::f" << endl; }
 
    virtual void g() { cout << "Base1::g" << endl; }
 
    virtual void h() { cout << "Base1::h" << endl; }
 
};
 
class Base2 {
 
public:
 
    virtual void f() { cout << "Base2::f" << endl; }
 
    virtual void g() { cout << "Base2::g" << endl; }
 
    virtual void h() { cout << "Base2::h" << endl; }
 
};
 
class Base3 {
 
public:
 
    virtual void f() { cout << "Base3::f" << endl; }
 
    virtual void g() { cout << "Base3::g" << endl; }
 
    virtual void h() { cout << "Base3::h" << endl; }
 
};
 
class Derived : public Base1, public Base2, public Base3 {
 
public:
 
    virtual void f() { cout << "Derived::f" << endl; }
 
    virtual void g1() { cout << "Derived::g1" << endl; }
 
};
 
void FunTest()
{
    Derived d;
 
    cout << sizeof(Derived) << endl;
 
    //访问Base1虚函数表
 
    pFun* fun = (pFun*)*((int*)&d + 0);
 
    while (*fun) {
 
        (*fun)();
 
        fun++;
    }
 
    cout << endl;
 
    //访问Base2虚函数表
 
    fun = (pFun*)*((int*)&d + 1);
 
    while (*fun) {
 
        (*fun)();
 
        fun++;
    }
 
    cout << endl;
 
    //访问Base3虚函数表
 
    fun = (pFun*)*((int*)&d + 2);
 
    while (*fun) {
 
        (*fun)();
 
        fun++;
    }
}
 
int main()
{
 
    FunTest();
 
    return 0;
}

最后显示结果为：

![这里写图片描述](http://www.2cto.com/uploadfile/Collfiles/20161101/20161101092310995.png)

虚函数表总结 Base虚表：Base类如果有虚函数的话，就按照虚函数出现的先后次序来填写续表 Derived虚表：对于继承Base类的对象，首先按照Base类的虚表格式复制，如果有重写(覆盖)基类的虚函数，则在对应的位置修改，不改变次序。如果派生类中新增虚函数，则将这虚函数填写到第一个父类虚函数后面即可。 **虚继承**

class B
{
public:
    int _b;
};
 
class C1 : public B
{
public:
    int _c1;
};
 
class C2 : public B
{
public:
    int _c2;
};

class D :public C1, public C2
{
public:
    int _d;
};
 
int main()
{
    D d;
    //d._b = 10;错误，访问不明确
    d.C1::_b = 10;//正确
    d.C2::_b = 10;//正确
 
    return 0; 
}

为什么会出现这样的问题? 我们知道它们的继承层次如下图所示：

![这里写图片描述](http://www.2cto.com/uploadfile/Collfiles/20161101/20161101092309980.png)

这种看似菱形的多继承会带来二义性：也就是说D中_b到底是从C1这条路继承而来的还是从C2这条路继承而来的?C++中为了避免这种访问不明确，从而引入了虚拟继承的机制。 虚拟继承是多重继承中特有的概念。虚拟基类是为解决多重继承而出现的。如上述类D继承自类C1、C2，而类C1、C2都继承自类B，因此在类D中两次出现类B中的变量。为了节省内存空间，可以将C1、C2对B的继承定义为虚拟继承，而B就成了虚拟基类。实现代码如下：

class B
{
public:
    int _b;
};
 
class C1 :virtual public B 
{
public:
    int _c1;
};
 
class C2 :virtual public B
{
public:
    int _c2;
};
 
class D :public C1, public C2
{
public:
    int _d;
};
 
int main()
{
    D d;
    d._d = 4;
 
    return 0;
}

这样就可以达到我们的要求了，直接使用d.\_d访问到\_d。然而虚继承到底是一种怎么样的实现机制?我们不妨在加不加virtual这两中情况下看下在内存中D d这个对象模型是怎么样的? 对于普通继承，我们通过VS2013的内存窗口可以看到：

![这里写图片描述](http://www.2cto.com/uploadfile/Collfiles/20161101/20161101092309981.png)

先是C1类中的成员，再是C2类中的成员，最后是D类自己的成员，此时sizeof(D) = 20。而一旦加了虚继承了，变化就比较明显了，如下图：

![这里写图片描述](http://www.2cto.com/uploadfile/Collfiles/20161101/20161101092309982.png)

**题目**

*   1.以下代码输出什么：

class C {
public:
    virtual string toString()
    {
        return "class C"; 
    }
};
 
class B : public C {
public:
    /\*virtual\*/ string toString()
    { 
        return "class B";
    }
};
 
class A : public B {
public:
    /\*virtual\*/ string toString()
    {
        return "class A";
    }
};

int main()
{

 C *pC = new A();
 cout << pC->toString().c_str() << endl;
 delete pC;

 pC = new B();
 cout << pC->toString().c_str() << endl;


 cout << endl;
 system("pause");

 return 0;
}

结果：输出class A;class B 分析：如果基类定义了虚同名函数，那么派生类中的同名函数自动变成了虚函数

*   2

![这里写图片描述](http://www.2cto.com/uploadfile/Collfiles/20161101/20161101092309984.png)

对这四种情况分别求sizeof(a), sizeof(b)。结果是什么样的呢?我在VS2013的win32平台测试结果为： 第一种：4，12 第二种：4，4 第三种：8，16 第四种：8，8 解释：首先我们看a类，我们知道每个存在虚函数的类都要有一个4字节的指针指向自己的虚函数表，再加上如果有数据，根据内存对齐机制，四种情况的类a所占的字节数应该是没有什么问题的。我们再看sizeof(B)，我们先看普通继承，对于普通继承仅仅是在原来的基础对虚表指针指向的虚函数表进行改写，类B依旧只有一个虚表指针，再加上如果有数据，根据内存对齐机制，所以第二种和第四种情况下，sizeof(B)分别为4和8。然而对于虚拟继承，会增加了一个偏移指针，而且由于类B中新增了虚函数，所以它的一般对象模型为这样(具体为什么是这样本文菱形虚拟继承会讲)：

![这里写图片描述](http://www.2cto.com/uploadfile/Collfiles/20161101/20161101092309986.png)

根据图示，在第一种的情况下，由于没有对应的数据成员，所以大小为12个字节。在第三种情况下，子类有自己的数据成员，而基类没有，所以删去最后一项，大小就是16个字节了

*   3.下面代码输出什么

class A
{
public:
	void show(int val) { cout << "A--" << val << "--A"; }
};

class B : public A
{
public:
	void show(int val) { cout << "B--" << val << "--B"; }
};

int main()
{
	A* p = new B;
	p->show(5);

	cin.get();
}

结果：A--5--A 分析：这就是隐藏 隐藏是指派生类的函数屏蔽了与其同名的基类函数，规则如下： （1）如果派生类的函数与基类的函数同名，但是参数不同。此时，不论有无virtual关键字，基类的函数将被隐藏（注意别与重载混淆）。 （2）如果派生类的函数与基类的函数同名，且参数也相同，但基类函数没有virtual 关键字。此时，基类的函数被隐藏（注意别与覆盖混淆）。 （3）但是，《Effective C++》条款36说：绝不重新定义继承而来的non-virtual函数，所以就不要这么做了。 转自：(部分重新整理) [解析虚函数表和虚继承](http://www.2cto.com/kf/201701/561029.html) [C++类成员函数的 重载、覆盖和隐藏区别](http://www.cnblogs.com/jiayith/p/3939683.html)
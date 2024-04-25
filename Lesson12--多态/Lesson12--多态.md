# Lesson12--多态

## 1. 多态的概念

多态通俗来说,就是多种形态.当类之间存在层次结构,并且类之间时通过继承关联时,就会用到多态.  
C++多态意味着调用成员函数时,会根据调用函数的对象的类型来执行不同的函数.

## 2. 多态定义及实现

### 2.1 多态的构成条件

多态是在不同继承关系的类对象,去调用同一函数,产生了不同的行为.比如 `Student` 继承了 `Person` .  
`Person` 对象买票全价, `Student` 对象买票半价.

那么在继承中要构成多态还有两个条件:
1. 必须通过 **基类** 的 **指针** 或者 **引用** 调用虚函数.
2. 被调用的函数必须是 **虚函数** ,且 **派生类必须对基类的虚函数进行重写** .

![](2.1-a.svg)

### 2.2 虚函数

虚函数:被 `virtual` 修饰的类成员函数称为虚函数.

```C++{.line-numbers}
class Person
{
public:
    virtual void BuyTicket()
    {
        std::cout << "买票全价" << std::endl;
    }
};
```

### 2.3 虚函数的重写

虚函数的重写(覆盖):派生类中有一个跟基类完全相同的虚函数(即派生类虚函数与基类虚函数的返回值类型,函数名字,参数列表完全相同),称子类的虚函数重写了基类的虚函数.

```C++{.line-numbers}
class Person
{
public:
    virtual void BuyTicket()
    {
        std::cout << "买票全价" << std::endl;
    }
};
class Student : public Person
{
public:
    virtual void BuyTicket()
    {
        std::cout << "买票半价" << std::endl;
    }
}; 
/*
注意:在重写基类虚函数时,派生类的虚函数在不加virtual关键字时,虽然也可以构成重写(因为继承后,基类的虚函数被继承下来了,在派生类依旧保持虚函数属性),但是这种写法不是很规范,不建议这样使用
*/
void Func(Person& p)
{
    p.BuyTicket();
}
int main()
{
    Person ps;
    Student st;
    Func(ps);
    Func(st);
    return 0;
}
```
**虚函数重写的两个例外** :
1. **协变(基类与派生类虚函数返回值类型不同)**
   派生类重写基类虚函数时,与基类虚函数返回值类型不同.即 **基类虚函数返回基类对象的指针或者引用** , **派生类虚函数返回派生类对象的指针或者引用** 时,称为协变.
   ```C++{.line-numbers}
   class A{};
   class B:public A{};
   class Person
   {
   public:
      virtual A* f(){return new A;}
   };
   class Student:public Person
   {
   public:
      virtual B* f(){return new B;}
   };
   ```

2. **析构函数的重写(基类与派生类析构函数的名字不同)**
   如果基类的析构函数为虚函数,此时派生类析构函数只要定义, **无论是否加 `virtual` 关键字,都与基类的析构函数构成重写** ,虽然基类与派生类析构函数名字不同.虽然函数名不相同,看起来违背了重写的规则,其实不然,这里可以理解为编译器对析构函数的名称做了特殊处理,编译后析构函数的名称统一处理成 `destructor` .
   ```C++{.line-numbers}
   class Person
   {
   public:
        virtual ~Person(){std::cout << "~Person()" << std::endl;}
        //析构函数的函数名会被处理成destructor
   };
   class Student : public Person
   {
   public:
        virtual ~Student(){std::cout << "~Student()" << std::endl;}
        //析构函数的函数名会被处理成destructor
    };
    //只要派生类Student的析构函数重写了Person的析构函数,下面的delete对象调用析构函数,才能构成多态,才能保证p1和p2指向的对象正确地调用析构函数
    int main()
    {
        Person* p1=new Person;
        Person* p2=new Student;
        delete p1;
        delete p2;
        return 0;
    }
    ```

    ```C++{.line-numbers}
    class Person
    {
    public:
        ~Person(){std::cout << "~Person()" << std::endl;}
    };
    class Student:public Person
    {
    public:
        ~Student(){std::cout << "~Student()" << std::endl;}
    };
    int main()
    {
        Person* p1=new Person;
        Person* p2=new Student;//若不构成多态,调用的指针类型是谁就调用谁的析构函数
        delete p1;
        delete p2;
        //若基类析构函数不是虚函数,则此处派生类对象会调用基类的析构函数
        //如果Student析构函数中有资源释放,而没有调用,却调用Person析构函数,则会发生内存泄漏
        return 0;
    }
    ```

### 2.4 C++11 override和final

从上面可以看出,C++对函数重写的要求比较严格,但是有些情况下由于疏忽,可能会导致函数名字母次序写反而无法构成重载,而这种错误在编译期间是不会报出的,只有在程序运行时没有得到预期结果才来debug会得不偿失,因此C++11提供了 `override` 和 `final` 两个关键字,可以帮助用户检测是否重写.

1. **final:修饰虚函数,表示该虚函数不能再被继承** 
   ```C++{.line-numbers}
   class A
   {
    public:
        virtual void func()final{}
   };
   class B : public A
   {
    public:
        virtual void func(){} //error:cannot override 'final' function "A::func"
   }
   ```
2. **override:检查派生类虚函数是否重写了基类某个虚函数,如果没有重写编译报错**
   ```C++{.line-numbers}
   class A
   {
    public:
        virtual void func1(){} 
   };
   class A : public B
   {
   public:
      virtual void func1()override{}
      virtual void func2()override{} //error:member function declared with 'override' dose not override a base class member
   };
   ```

### 2.5 重载,覆盖(重写),隐藏(重定义)的对比

![](2.5-a.svg)

## 3. 抽象类

### 3.1 概念

在虚函数的后面写上 `=0` ,则这个函数为 **纯虚函数** .包含纯虚函数的类叫作抽象类(也叫接口类), **抽象类不能实例化出对象** . **派生类继承后也不能实例化出对象** ,只有重写纯虚函数,派生类才能实例化出对象.纯虚函数规范了派生类必须重写,另外纯虚函数更体现出了接口继承.

```C++{.line-numbers}
class Car//抽象类
{
public:
    virtual void Drive()=0;//不需要实现,纯虚函数
};
class Benz : public Car
{
public:
    virtual void Drive(){}//必须对纯虚函数重写
};
class BMW : public Car
{
public:
    virtual void Drive(){}
};
```

### 3.2 接口继承和实现继承

普通函数的继承是一种实现继承,派生类继承了基类函数,可以使用函数,继承的是函数的实现.虚函数的继承是一种接口继承,派生类继承的是基类虚函数的接口,目的是为了重写,达成多态,继承的是接口.所以如果不实现多态,不要把函数定义成虚函数.

## 4. 多态的原理

### 4.1 虚函数表

```C++{.line-numbers}
//sizeof(Base)是多少？
class Base
{
public:
    virtual void Func1()
    {
        cout<<"Func1()"<<endl;
    }
private:
    int _b=1;
};
```

通过观察测试我们发现b对象是8bytes(x32),除了_b成员,还多了一个 `__vfptr` 放在对象的前面(注意有些平台可能会放到对象的后面),对象中的这个指针我们叫作虚函数表指针(v代表virtual,f代表function).一个含有虚函数的类中都至少有一个虚函数表指针,因为虚函数的地址要被放到虚函数表中,虚函数表也简称虚表.  
虚函数表指针是一个指针数组.  
多态是在运行时到指向的对象的虚表中查找要调用的虚函数的地址来进行调用.  
普通函数和虚函数都会被编译成指令以后存在代码段.

```C++{.line-numbers}
class Base
{
public:
    virtual void Func1()
    {
        cout << "Base::Func1()" << endl;
    }
    virtual void Func2()
    {
        cout << "Base::Func2()" << endl;
    }
    void Func3()
    {
        cout << "Base::Func3()" << endl;
    }
private:
    int _b = 1;
};
//Base中虚函数表__vfptr存放着Base::Func1与Base::Func2
class Derive : public Base
{
public:
    virtual void Func1()//只重写Func1
    {
        cout << "Derive::Func1()" << endl;
    }
private:
    int _d = 2;
};
//Derive中虚函数表__vfptr存放着Derive::Func1与Base::func2
//可见派生类继承了基类的虚函数表,并对重写虚函数的指针进行覆盖
int main()
{
    Base b;
    Derive d;
    return 0;
}
```

通过观察和测试,我们发现了以下几点问题:
1. 派生类对象d中也有一个虚表指针,d对象由两部分构成,一部分是父类继承下来的成员,虚表指针也就是存在部分的另一部分是自己的成员.
2. 基类b对象和派生类d对象虚表是不一样的,这里我们发现Func1完成了重写,所以d的虚表中存的是重写的Derive::Func1,所以虚函数的重写也叫作覆盖,覆盖就是指虚表中虚函数的覆盖.重写是语法的叫法,覆盖是原理层的叫法.
3. 另外Func2继承下来后是虚函数,所以放进了虚表,Func3也继承下来了,但是不是虚函数,所以不会放进虚表.
4. 虚函数表本质是一个存虚函数指针的指针数组,这个数组最后面放了一个nullptr.
5. 总结一下派生类的虚表生成:
   * 先将基类中的虚表内容拷贝一份到派生类虚表中
   * 如果派生类重写了基类中某个虚函数,用派生类自己的虚函数覆盖虚表中基类的虚函数
   * 派生类自己新增加的虚函数按其在派生类中的声明次序增加到派生类虚表的最后
6. 虚表存的是虚函数指针,不是虚函数,虚函数和普通函数一样,都存在代码段,只是它的指针又存到了虚表中.另外对象中存的不是虚表,存的是虚表指针.

### 4.2 多态的原理

`Func` 函数传 `Person` 调用的 `Person::BuyTicker` ,传 `Student` 调用的是 `Student::BuyTicket` .

```C++{.line-numbers}
class Person
{
public:
    virtual void BuyTicket(){}
};
class Student : public Person
{
public:
    virtual void BuyTicket(){}
};
void Func(Person& p)
{
    p.BuyTicket();
}
int main()
{
    Person Mike;
    Func(Mike);

    Student Johnson;
    Func(Johnson);

    return 0;
}
```

1. `p` 是指向 `Mike` 对象时, `p->BuyTicket` 在 `Mike` 的虚表中找到虚函数是 `Person::BuyTicket` .
2. `p` 是指向 `Johnson` 对象时, `p->BuyTicket` 在 `Johson` 的虚表中找到虚函数是 `Student::BuyTicket` .
3. 这样就实现出了不同对象去完成同一行为时,展现出不同的形态.
4. 反过来思考我们要达到多态,有两个条件,一个是虚函数覆盖,一个是对象的指针或引用调用虚函数.
5. 满足多态以后的函数调用,不是在编译时确定的,是运行起来以后到对象的中去找的.不满足多态的函数调用时编译时确认好的.

```C++{.line-numbers}
void Func(Person* p)
{
    p->BuyTicket();
}
int main()
{
    Person mike;
    Func(&mike);
    mike.BuyTicket();
    return 0;
}
```

### 4.3 动态绑定与静态绑定

1. 静态绑定又称为前期绑定(早绑定),在程序编译期间确定了程序员的行为,也称为静多态,比如:函数重载
2. 动态绑定又称为后期绑定(晚绑定),在程序运行期间,根据具体拿到的类型确定程序的具体行为,调用具体的函数,也称为动态多态.

## 5. 单继承和多继承关系的虚函数表

### 5.1 单继承中的虚函数表

```C++{.line-numbers}
//x86 debug
#include<iostream>
class Base
{
public:
    virtual void func1(){std::cout << "Base::func1" << std::endl;}
    virtual void func2(){std::cout << "Base::func2" << std::endl;}
private:
    int a;
};
class Derive:public Base
{
public:
    virtual void func1(){std::cout << "Derive::func1" << std::endl;}
    virtual void func3(){std::cout << "Derive::func3" << std::endl;}
    virtual void func4(){std::cout << "Derive::func4" << std::endl;}
private:
    int b;
};

typedef void (*VFPTR) ();
void PrintVTable(VFPTR vTable[])
{
    //依次取虚表中的虚函数指针打印并调用.调用可以看出存放的函数.
    std::cout << "虚表地址>" << vTable << std::endl;
    for(int i = 0;vTable[i]! = nullptr;++i)
    {
        printf("第%d个虚函数地址: 0x%x,->",i+1,vTable[i]);
        VFPTR f = vTable[i];
        f();
    }
    std::cout << std::endl;
}
int main()
{
    Base b;
    Derive d;
    //思路:取出b,d对象的头4bytes,就是虚表的指针,虚函数表本质是一个存虚函数指针的指针数组,这个数组最后面放了一个nullptr
    //1.先取b的地址,强转成一个int*的指针
    //2.再解引用取值,就得到b对象头4bytes的值,这个值就是指向虚表的指针
    //3.再强转成VFPTR*,因为虚表就是一个存VFPTR类型(虚函数指针类型)的数组
    //4.虚表指针传递给PrintVTable进行打印虚表
    //5.需要说明的是这个打印虚表的代码经常会崩溃,因为编译器有时对虚表的处理不干净,虚表最后面没有放nullptr,导致越界,这是编译器的问题,可以清理解决方案,再编译
    VFPTR* vTableb = (VFPTR*)(*(int*)&b);
    PrintVTable(vTableb);
    VFPTR* vTabled = (VFPTR*)(*(int*)&d);
    PrintVTable(vTabled);
    return 0;
}
```

> 虚表地址>009C9B34  
> 第1个虚函数地址: 0x9c12b7,->Base::func1  
> 第2个虚函数地址: 0x9c1113,->Base::func2  
>   
> 虚表地址>009C9B64  
> 第1个虚函数地址: 0x9c1244,->Derive::func1  
> 第2个虚函数地址: 0x9c1113,->Base::func2  
> 第3个虚函数地址: 0x9c1230,->Derive::func3  
> 第4个虚函数地址: 0x9c116d,->Derive::func4  

### 5.2 多继承中的虚函数表

```C++{.line-numbers}
class Base1
{
public:
    virtual void func1(){std::cout << "Base1::func1" << std::endl;}
    virtual void func2(){std::cout << "Base1::func2" << std::endl;}
private:
    int b1;
};
class Base2
{
public:
    virtual void func1(){std::cout << "Base2::func1" << std::endl;}
    virtual void func2(){std::cout << "Base2::func2" << std::endl;}
private:
    int b2;
};
class Derive : public Base1,public Base2
{
public:
    virtual void func1(){std::cout << "Derive::func1" << std::endl;}
    virtual void func3(){std::cout << "Derive::func3" << std::endl;}
private:
    int d1;
};
typedef void(*VFPTR)();
void PrintVTable(VFPTR vTable[])
{
    std::cout << "虚表地址>" << vTable << std::endl;
    for(int i = 0;vTable[i] != nullptr;++i)
    {
        printf("第%d个虚函数地址: 0x%x,->",i+1,vTable[i]);
        VFPTR f = vTable[i];
        f();
    }
    std::cout << std::endl;
}
int main()
{
    Derive d;
    VFPTR* vTableb1=(VFPTR*)(*(int*)&d);
    PrintVTable(vTableb1);
    VFPTR* vTableb2=(VFPTR*)(*(int*)((char*)&d+sizeof(Base1)));
    PrintVTable(vTableb2);
    return 0;
}
```

> 虚表地址>005B9B94  
> 第1个虚函数地址: 0x5b1249,->Derive::func1  
> 第2个虚函数地址: 0x5b12f3,->Base1::func2  
> 第3个虚函数地址: 0x5b1235,->Derive::func3  
>   
> 虚表地址>005B9BA8  
> 第1个虚函数地址: 0x5b136b,->Derive::func1  
> 第2个虚函数地址: 0x5b10b4,->Base2::func2  

多继承派生类的未重写的虚函数放在第一个继承基类部分的虚函数表中.

### 5.3 菱形继承,菱形虚拟继承

实际中不建议设计出菱形继承及菱形虚拟继承,一方面过于复杂容易出问题;另一方面,这样的模型访问基类成员有一定的性能损耗.
# Lesson02--类与对象(下)

## 1. 再谈构造函数

### 1.1 构造函数体赋值

在创建对象时,编译器通过调用构造函数,给对象中各个成员变量一个合适的初始值.

```C++{.line-numbers}
class Date
{
public:
    Date(int year,int month, int day)
    {
        _year = year;
        _month = month;
        _day = day;
    }
private:
    int _year;
    int _month;
    int _day;
};
```

虽然上述构造函数调用之后,对象中已经有了一个初始值,但是不能将其称作为类对象成员的初始化, **构造函数体中的语句只能将其称作为赋初始值** ,而不能称作初始化.因为 **初始化只能初始化一次,而构造函数体内可以多次赋值** .

### 1.2 初始化列表

初始化列表,以一个冒号开始,接着是一个以逗号分隔的数据成员列表,每个成员变量后面跟一个放在括号中的初始值或表达式.

```C++{.line-numbers}
class Date
{
public:
    Date(int year,int month,int day)
    :_year(year)
    ,_month(month)
    ,_day(day)
    {}
private:
    int _year;
    int _month;
    int _day;
};
```

**注意:**
1. 每个成员变量在初始化列表中只能出现一次(初始化只能初始化一次).
2. 类中包含以下成员,必须放在初始化列表位置进行初始化.
   * 引用成员变量
   * const成员变量
   * 自定义类型成员(该类没有默认构造函数)
3. 尽量使用初始化列表初始化,因为不管你是否使用初始化列表,对于自定义类型成员变量,一定会先使用初始化列表初始化.
4. **成员变量在类中声明次序就是其在初始化列表中的初始化次序,与其在初始化列表中的先后次序无关** .

```C++{.line-numbers}
class A
{
public:
    A(int a)
    :_a(a)
    {}
private:
    int _a;
};
class B
{
public:
    //成员变量的定义
    B(int a,int ref)
    :_aobj(a)
    ,_ref(ref)
    ,_n(10)
    {}
private:
    //成员变量的声明
    A _aobj;//没有默认构造函数(不用传参就可以调用的构造函数)
    int& _ref;//引用
    const int _n;//const
};
//对象定义
B b(1,2);
```

```C++{.line-numbers}
class A
{
public:
    A(int a)
    :_a1(a)
    ,_a2(_a1)
    {}
    void Print()
    {
        cout << _a1 << endl << _a2 << endl;
    }
private:
    //成员变量在类中声明次序就是其在初始化列表中的初始化次序,与其在初始化列表中的先后次序无关
    int _a2;
    int _a1;
};
int main()
{
    A a(1);
    a.Print();//输出结果:_a1为1,_a2为随机值
    return 0;
}
```

### 1.3 explicit关键字

构造函数不仅可以构造与初始化对象,对于单个参数的构造函数,还具有类型转换的作用.

```C++{.line-numbers}
class Date
{
public:
    explicit Date(int year)
    :_year(year)
    {}
    int _year;
    int _month;
    int _day;
};
void PrintDate(const Date& d)
{
    cout << d._year << " " << d._month << " " << d._day << endl;
}
void TestDate()
{
    Date d1(2018);
    PrintDate(d1);//正常调用不涉及隐式类型转换
    Date d2 = 2019;//error:构造函数被声明为explicit,不能进行隐式类型转换
    PrintDate(2020);//error:构造函数被声明为explicit,不能进行隐式类型转换
    Date d3 = Date(2021);//正常调用,显式调用构造函数进行对象初始化

    //若构造函数未用explicit修饰时
    //用一个整型变量给日期类型对象赋值,会进行隐式转换
    //实际编译器背后会构造一个临时对象最后用临时对象给d2对象进行赋值
    Date& d4 = 2020;//error
    //临时变量具有常性,应该为const Date& d4 = 2020;
}
```

用explicit修饰构造函数,将会禁止单参构造函数的隐式转换.

## 2. static成员

### 2.1 概念

声明为static的类成员称为类的静态成员,用static修饰的成员变量,称之为静态成员变量;用static修饰的成员函数,称之为静态成员函数.静态的成员变量一定要在类外进行初始化.

> 实现一个类,计算程序中创建了多少个对象
> ```C++{.line-numbers}
> class A
> {
>   public:
>   A(){++_scount;}
>   A(const A& t){++_scount;}
>   ~A(){--_scount;}
>   static int GetACount(){return _scount;}
>   private:
>   static int _scount;
> };
> int A::_scount = 0;//作用域限定符可以在类外访问类的静态成员变量
> void Test()
> {
>   cout << A::GetACout() << endl;
>   A a1,a2;
>   A a3(a1);
>   cout << A::GetACount() << endl;
> }
> ```

### 2.2 特性

1. 静态成员为所有类对象所共享,不属于某个具体的实例,存放在静态区.
2. 静态成员变量必须在类外定义,定义时不添加static关键字.
3. 类静态成员即可用 `类名::静态成员` 或者 `对象.静态成员` 来访问.
4. 静态成员函数没有隐藏 `this` 指针,不能访问任何非静态成员.
5. 静态成员和类的普通成员一样,也有 `public` , `protected` , `private` 3种访问级别,也可以具有返回值.

## 3. 友元

友元分为: **友元函数** 和 **友元类** .

友元提供了一种突破封装的方式,有时提供了便利.但是友元会增加耦合度,破坏了封装,所以友元不宜多用.

### 3.1 友元函数

问题:现在我们尝试去重载 `operator<<` ,然后发现我们没办法将 `operator<<` 重载成成员函数.因为 `cout` 的输出流对象和隐含的 `this` 指针在抢占第一个参数的位置. `this` 指针默认是第一个参数也就是左操作数.但是实际使用中 `cout` 需要的是第一个形参对象,才能正常使用.所以我们要将 `operator<<` 重载成全局函数.但是这样的话,又会导致类外没办法访问成员,那么这里就需要友元来解决.

```C++{.line-numbers}
#include<iostream>
using namespace std;
class Date
{
public:
    Date(int year,int month,int day)
    :_year(year)
    ,_month(month)
    ,_day(day)
    {}
    ostream& operator<<(ostream& _cout)
    {
        _cout << _year << "-" << _month << "-" << _day << endl;
        return _cout;
    }
private:
    int _year;
    int _month;
    int _day;
};
int main()
{
    Date d(2023,3,20);
    d << cout;//<<定义在类内,默认第一个参数为this指针
    //d << cout; -> d.operator<<(&d,cout);这不符合常规调用
    return 0;
}
```

**友元函数可以直接访问类的私有成员,它是定义在类外部的普通函数,不属于任何类,但需要在类的内部声明,声明时需要加 `friend` 关键字.**

```C++{.line-numbers}
#include<iostream>
using namespace std;
class Date
{
    friend ostream& operator<<(ostream& _cout,const Date& d);
    friend istream& operator>>(istream& _cin,Date& d);
public:
    Date(int year,int month,int day)
    :_year(year)
    ,_month(month)
    ,_day(day)
    {}
private:
    int _year;
    int _month;
    int _day;
};
ostream& operator<<(ostream& _cout,const Date& d)
{
    _cout << d._year << "-" << d._month << "-" << d._day;
    return _cout;
}
istream& operator>>(istream& _cin,Date& d)
{
    _cin >> d._year;
    _cin >> d._month;
    _cin >> d._day;
    return _cin;
}
int main()
{
    Date d(2023,3,20);
    cin >> d;
    cout << d << endl;
    return 0;
}
```

**说明**
* 友元函数可以访问类的私有和保护成员,但不是类的成员函数.
* 友元函数不能用 `const` 修饰.
* 友元函数可以在类定义的任何地方声明,不受类访问限定符限制.
* 一个函数可以是多个类的友元函数.
* 友元函数的调用与普通函数的调用和原理相同.

### 3.2 友元类

友元类的所有函数都可以是另一个类的友元函数,都可以访问另一个类中的非公有成员.

* 友元关系是单向的,不具有交换性.
  比如下述Time类和Date类,在Time类中声明Date类为其友元类,那么可以在Date类中直接访问Time类的私有成员变量,但想在Time类中访问Date类中私有的成员变量则不行.
* 友元关系不能传递.
  如果B是A的友元,C是B的友元,则不能说明C是A的友元.

```C++{.line-numbers}
class Time
{
    friend class Date;//声明日期类为时间类的友元类,则在日期类中就可以直接访问Time类中的私有成员变量
public:
    Time(int hour=0,int minute=0,int day=0)
    :_hour(hour)
    ,_minute(minute)
    ,_second(second)
    {}
private:
    int _hour;
    int _minute;
    int _second;
};
class Date
{
public:
    Date(int year = 0,int month = 0,int day = 0)
    :_year(year)
    ,_month(month)
    ,_day(day)
    {}
    void SetTimeOfDate(int hour,int minute,int second)
    {
        _t.hour = hour;
        _t.minute = minute;
        _t.second = second;
    }
private:
    int _year;
    int _month;
    int _day;
    Time _t;
};
```

## 4. 内部类

### 4.1 概念

如果一个类定义在另一个类的内部,这个内部的类就叫做内部类.注意此时这个内部类是一个独立的类,它不属于外部类,更不能通过外部类的对象去调用内部类.外部类对内部类没有任何优越的访问权限.
注意: **内部类就是外部类的友元类** .注意友元类的定义,内部类可以通过外部类的对象参数来访问外部类中的所有成员.但是外部类不是内部类的友元.

### 4.2 特性

1. 内部类可以定义在外部类的 `public` , `protected` , `private` 都是可以的.
2. 注意内部类可以直接访问外部类中的 `static` ,枚举成员,不需要外部类的对象/类名.
3. `sizeof(外部类)=外部类` ,和内部类没有任何关系.

```C++{.line-numbers}
class A
{
private:
    static int k;
    int h;
public:
    class B//B为A的友元
    {
    public:
        void foo(const A& a)
        {
            cout << k << endl;
            cout << a.h << endl;
        }
    };
};
int A::k = 1;
int main()
{
    A::B b;//定义B类对象b
    b.foo(A());
    return 0;
}
```

## 5. 匿名对象

```C++{.line-numbers}
class A
{
public:
    A(int a = 0)
    :_a(a)
    {
        cout << "A(int a)" << endl;
    }
    ~A()
    {
        cout << "~A()" << endl;
    }
private:
    int _a;
};
int main()
{
    //有名对象
    A a1;
    A a2(10);

    //匿名对象
    //生命周期只在当前一行
    A();
    A(10);
    return 0;
}
```

```C++{.line-numbers}
class Solution
{
public:
    int Sum_Solution(int n)
    {
        //...
        return n;
    }
};
int main()
{
    Solution s1;
    s1.Sum_Solution(10);
    Solution().Sum_Solution(1);
    return 0;
}
```

## 6. 拷贝对象时的一些编译器优化

在传参和返回值的过程中,一般编译器会做一些优化,减少对象的拷贝.

```C++{.line-numbers}
class A
{
public:
    A(int a = 0)
    :_a(a)
    {
        cout << "A(int a)" << endl;
    }
    A(const A& a)
    :_a(a._a)
    {
        cout << "A(const A& a)" << endl;
    }
    A& operator=(const A& a)
    {
        cout << "A& operator=(const A& a)" << endl;
        if(this != &a)//判断是否给自己赋值
            _a=a._a;
        return *this;//为实现连续赋值,必须要有返回值
    }
    ~A()
    {
        cout << "~A()" << endl;
    }
private:
    int _a;
};
void f1(A a)
{}
void f2(const A& a)
{}
A f3()
{
    A aa;//构造
    return aa;//拷贝构造
}
A f4()
{
    return A();//构造+拷贝构造
}
int main()
{
    A a1 = 2;//构造+拷贝构造->直接构造

    const A& a2 = 2;//调用构造函数创建临时变量,并被a2引用

    f1(a1);//调用拷贝构造

    f1(A(2));//构造+拷贝构造->直接构造
    //编译器可能会直接将A(2)对象的构造直接放在f1(A a)函数的参数中,而不进行拷贝构造

    f2(a1);//传引用,不调用拷贝构造

    A a3 = f3();//构造+拷贝构造+拷贝构造->构造+拷贝构造
    //编译器将两次拷贝构造优化为一次拷贝构造
    //f3函数结束前,用aa直接拷贝给a3,不创建临时变量

    A a4 = f4();//构造+拷贝构造+拷贝构造->一次构造
    return 0;
}
```
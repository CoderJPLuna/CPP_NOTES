# Lesson04--模板初阶

## 1. 泛型编程

如何实现一个通用的交换函数:

```C++{.line-numbers}
template<class T>
void Swap(T& x1,T& x2)
{
    T x=x1;
    x1=x2;
    x2=x;
}
```

使用函数重载虽然可以实现,但有以下几个缺点:
1. 重载的函数仅仅只是类型不同,代码的复用率比较低,只要有新类型出现时,就需要增加对应的函数.
2. 代码的可维护性比较低,一个出错可能所有的重载均出错.

**泛型编程:** 编写与类型无关的通用代码,是代码复用的一种手段.模板是泛型编程的基础.

## 2. 函数模板

### 2.1 概念

函数模板代表了一个函数家族,该函数模板与类型无关,在使用时被参数化,根据实参类型产生函数的特定类型版本.

### 2.2 函数模板格式

`template<typename T1,typename T2,……,typename Tn>`

```C++{.line-numbers}
template<typename T>
void Swap(T& left,T& right)
{
    T temp=left;
    left=right;
    right=temp;
}
```

**注意:**
`typename` 是用来定义模板参数关键字,也可以使用 `class` (切记不能用 `struct` 代替 `class` ).

### 2.3 函数模板的原理

函数模板是一个蓝图,它本身并不是函数,是编译器使用方式产生特定具体类型函数的模具.所以模板就是将本来应该重复做的事交给编译器.
在编译器的预处理阶段,对于模板函数的使用,编译器需要根据传入的实参类型来推演生成对应类型的函数以供调用.

### 2.4 函数模板的实例化

用不同类型的参数使用函数模板时,称为函数模板的实例化.模板参数实例化分为:隐式实例化和显式实例化.

1. 隐式实例化:让编译器根据实参推演模板参数的实际类型.

   ```C++{.line-numbers}
   template<class T>
   T Add(const T& left,const T& right)
   {
       return left+right;
   }
   int main()
   {
       int a1 = 10;
       int a2 = 20;
       double d1 = 10.0;
       double d2 = 20.0;
       Add(a1,a2);
       Add(d1,d2);
       Add(a1,d1);//错误
       //该语句不能通过编译,因为在编译期间,当编译器看到该实例化时,需要推演其实参类型,通过实参a1将T推演为int,通过实参d1将T推演为double类型,但模板参数列表中只有一个T,编译器无法确定此处到底该将T确定为int还是double类型而报错
       //注意:在模板中,编译器一般不会进行类型转换操作,因为一旦转化出现问题,编译器就要背黑锅
       //此时有两种处理方式:
       //1.用户自己来强制类型转化
       //2.使用显式实例化
       Add(a1,(int)d1);
       return 0;
   }
   ```

2. 显式实例化:在函数名后的 `<>` 中指定模板参数的实际类型.

   ```C++{.line-numbers}
   template<class T>
   T Add(const T& left,const T& right)
   {
    return left + right;
   }
   int main()
   {
    int a = 10;
    double b = 20.0;
    //显式实例化
    Add<int>(a,b);
    return 0;
   }
   ```

   如果类型不匹配,编译器会尝试进行隐式类型转换,如果无法转换成功,编译器将会报错.

### 2.5 模板参数的匹配原则

1. 一个非模板函数可以和一个同名的函数模板同时存在,而且该函数模板还可以被实例化为这个非模板函数.

   ```C++{.line-numbers}
   //专门处理int的加法函数
   int Add(int left,int right)
   {
        return left + right;
   }
   //通用加法函数
   template<class T>
   T Add(T left,T right)
   {
        return left + right; 
   }
   void Test()
   {
        Add(1,2);//与非模板函数匹配,编译器不需要特化
        Add<int>(1,2.0);//调用编译器特化的Add版本
   }
   ```

2. 对于非模板函数和同名函数模板,如果其它条件都相同,在调用时会优先调用非模板函数而不会从该模板函数产生一个实例.如果模板可以产生一个具有更好匹配的函数,那么选择模板.

   ```C++{.line-numbers}
   //专门处理int的加法函数
   int Add(int left,int right)
   {
        return left + right;
   }
   //通用加法函数
   template<class T1,class T2>
   T1 Add(T1 left,T2 right)
   {
        return left + right;
   }
   void Test()
   {
        Add(1,2);//与非函数模板类型完全匹配,不需要函数模板实例化
        Add(1,2.0);//模板函数可以生成更加匹配的版本,编译器根据实参生成更加匹配的Add函数
   }
   ```

3. 模板函数不允许自动类型转换,但普通函数可以进行自动类型转换.

## 3. 类模板

### 3.1 类模板的定义格式

```C++{.line-numbers}
template<class T1,class T2,……,class Tn>
class 类模板名
{
    //类内成员定义
};
```

```C++{.line-numbers}
//动态顺序表
//注意:Vector不是具体的类,是编译器根据被实例化的类型生成具体类的模具
template<class T>
class Vector
{
public:
    Vector(size_t capacity = 0)
    :_pDate(new T[capacity])
    ,_size(0)
    ,_capacity(capacity);
    {}
    //使用析构函数演示：在类中声明，在类外定义。
    ~Vector();
    void pushBack(const T& data);
    void PopBack();
    size_t Size()
    {
        return size;
    }
    T& operator[](size_t pos)
    {
        assert(pos < _size);
        return _pData[pos];
    }
private:
    T* _pData;
    size_t _size;
    size_t _capacity;
};
//注意:类模板中函数在类外进行定义时,需要加模板参数列表
vector<T>::~Vector()
{
    if(_pData)
        delete[] _pData;
    _size = _capacity = 0;
}
```

### 3.2 类模板的实例化

类模板实例化与函数模板实例化不同,类模板实例化需要在类模板名字后跟 `<>` ,然后将实例化的类型放在 `<>` 中即可,类模板名字不是真正的类,而实例化的结果才是真正的类.

```C++{.line-numbers}
//Vector类名,Vector<int>才是类型
Vector<int> s1;
Vector<double> s2;
```

> **C语言实现一个栈**
>
> ```C{.line-numbers}
> typedef int StackDataType;//若要更改栈中数据类型，只需更改此处的int
> struct Stack
> {
>     StackDataType* _a;
>     int _size;//_top
>     int _capacity;
> };
> typedef struct Stack Stack;
> void StackInit(Stack* ps){}
> void StackDestory(Stack* ps){}
> void StackPush(Stack* ps,StackDataType x){}
> void StackPop(Stack* ps){}
> //...
> int main()
> {
>     //缺陷：    
>     //1.忘记初始化和销毁->构造函数+析构函数(C++)
>     //2.没有封装，谁都可以修改结构体的数据->类+访问限定符(C++)
>     //3.如果想同时定义两个栈，一个栈存int，一个栈存double，无法做到->模板(C++)
>     Stack stc;
>     StackInit(&stc);
>     StackPush(&stc,1);
>     StackPush(&stc,2);
>     StackPush(&stc,3);
>     stc._capacity = 0;//非法修改
>     StackDestory(&stc);
>     return 0;
> }
> ```

```C++{.line-numbers}
template<class T>
class Stack
{
public:
    Stack()
    {}
    ~Stack()
    {}
    void Push(T x)
    {}
private:
    T* _a;
    int _size;
    int _capacity;
};
int main()
{
    Stack<int> stcpp_int;
    stcpp_int.Push(1);//实际也是两个参数，有一个是隐含的this指针
    stcpp_int.Push(2);
    stcpp_int.Push(3);
    Stack<double> stcpp_double;
    stcpp_double.Push(1.0);
    stcpp_double.Push(2.3);
    return 0;
}
```

```C++{.line-numbers}
template<class T>
class vector//动态增长的数组
{
public:
    vector()
    :_a(nullptr)
    ,_size(0)
    ,_capacity(0)
    {}
    ~vector()
    {
        delete[] _a;
        _a = nullptr;
        _size = _capacity=0;
    }
    //类里面声明，类外面定义
    void push_back(const T& x);
    void push_pop();
    size_t size()
    {
        return _size;
    }
    T operator[](size_t i)
    {
        assert(i < _size);
        return _a[i];
    }
private:
    T* _a;
    size_t _size;
    size_t _capacity;
};
//类外面定义
template<class T>
void vector<T>::push_back(const T& x)
{
    //如果空间不够，需要进行增容
    if(_size == _capacity)
    {
        size_t newcapacity = _capacity == 0?2:_cpapcity*2;
        T* tmp = new T[newcapacity];
        if(_a)
        {
            memcpy(tmp,_a,sizeof(T)*_size);
            delete[] _a;
        }
        _a = tmp;
        _capacity = newcapacity;
    }
}
```
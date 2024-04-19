# Lesson10--模板进阶

## 1. 非类型模板参数

模板参数分为 **类类型形参** 与 **非类型形参** .  
类型形参:出现在模板参数列表中,跟在 `class` 或者 `typename` 之类的参数类型名称.  
非类型形参:用常量作为类/函数模板的一个参数,在类(函数)模板中可将该参数当成常量来使用.

```C++{.line-nimbers}
template<class T,size_t N=10>
class array
{
public:
    T& operator[](size_t index)
    {
        return _array[index];
    }
    const T& operator[](size_t index)const
    {
        return _array[index];
    }
    size_t size()const
    {
        return _size;
    }
    bool empty()const
    {
        return 0 == _size;
    }
private:
    T _array[N];
    size_t _size;
};
```

**注意** :
1. 浮点数,类对象及字符串不允许作为非类型模板参数.
2. 非类型的模板参数必须在编译器就能确认结果.

## 2. 模板的特化

### 2.1 概念

通常情况下,使用模板可以实现一些与类型无关的代码,但对于一些特殊类型的可能会得到一些错误的结果,需要特殊处理.

```C++{.line-numbers}
template<class T>
bool Less(T left,T right)
{
    return left < right;
}
int main()
{
    std::cout << Less(1,2) < <std::endl;//可以比较,结果正确
    Date d1(2000,12,22);
    Date d2(2012,12,23);
    std::cout << Less(d1,d2) << std::endl;//可以比较,结果正确
    Date* p1 = &d1;
    Date* p2 = &d2;
    std::cout << Less(p1,p2) << std::endl;//可以比较,结果错误
    return 0;
}
```

可以看到, `Less` 绝大多数情况下都可以正常比较,但是在特殊场景下就得到错误的结果.上述示例, `p1` 指向的 `d1` 显然小于 `p2` 指向的 `d2` 对象,但是 `Less` 内部并没有比较 `p1` 和 `p2` 指向的对象内容,而比较的是 `p1` 和 `p2` 指针的地址,这就无法达到预期而错误.

此时,就需要对模板进行特化,即:在原模板类的基础上,针对特殊类型所进行所特殊化的实现方式.模板特化中分为 **函数模板特化** 与 **类模板特化** .

### 2.2 函数模板特化

函数模板特化步骤:
1. 必须要先有一个基础的函数模板.
2. 关键字 `template` 后面接一对空的 `<>` .
3. 函数名后跟一对 `<>` , `<>` 中指定需要特化的类型.
4. 函数形参表:必须要和模板函数的基础类型完全相同,如果不同编译器可能会报一些错误.

```C++{.line-numbers}
//函数模板--参数匹配
template<class T>
bool Less(T left,T right)
{
    return left < right;
}
//对Less函数模板进行特化
template<>
bool Less<Date*>(Date* left,Date* right)
{
    return *left < *right;
}
int main()
{
    Date(2022,7,7);
    Date(2022,7,8);
    Date* p1 = &d1;
    Date* p2 = &d2;
    std::cout << Less(p1,p2) << std::endl;
    return 0;
}
```

**注意** :一般情况下如果函数模板遇到不能处理或者处理有误的类型,为了实现简单通常都是将该函数直接给出.

```C++{.line-numbers}
bool Less(Date* left,Date* right)
{
    return *left < *right;
}
```

这种实现简单明了,代码的可读性高,容易书写,因为对于一些参数类型复杂的函数模板,特化时特别给出,因此函数模板不建议特化.

### 2.3 类模板特化

#### 2.3.1 全特化

全特化即是将模板参数列表中所有的参数都确定化.

```C++{.line-numbers}
template<class T1,class T2>
class Data
{
public:
    Data()
    {
        std::cout << "Data<T1,T2>" << std::endl;
    }
private:
    T1 _d1;
    T2 _d2;
};

template<>
class Data<int,char>
{
public:
    Data()
    {
        std::cout << "Data<int,char>" << std::endl;
    }
private:
    int _d1;
    char _d2;
};
```

#### 2.3.2 偏特化

偏特化:任何针对模板参数进一步进行条件限制设计的特化版本.

```C++{.line-numbers}
template<class T1,class T2>
class Data
{
public:
    Data()
    {
        std::cout << "Data<T1,T2>" << std::endl;
    }
private:
    T1 _d1;
    T2 _d2;
};
```

偏特化有以下两种表现方式:
* 部分特化
  将模板参数类表中的一部分参数特化.
  ```C++{.line-numbers}
  //将第二个参数特化为int
  template<class T1>
  class Data<T1,int>
  {
    public:
        Data()
        {
            std::cout << "Data<T1,int>" << std::endl;
        }
    private:
        T1 _d1;
        int _d2;
  };
  ```
* 参数更进一步的限制
  偏特化并不仅仅是指特化部分参数,而是针对模板参数更进一步的条件限制所设计出来的一个特化版本.
  ```C++{.line-numbers}
  //两个参数偏特化为指针类型
  template<typename T1,typename T2>
  class Data<T1*,T2*>
  {
    public:
        Data()
        {
            std::cout << "Data(T1*,T2*)>" << std::endl;
        }
    private:
        T1 _d1;
        T2 _d2;
  };
  //两个参数偏特化为引用类型
  template<typename T1,typename T2>
  class Data<T1&,T2&>
  {
    public:
        Data(const T1& d1,const T2& d2)
        :_d1(d1)
        ,_d2(d2)
        {
            std::cout << "Data<T1&,T2&>" << std::endl;
        }
    private:
        const T1& _d1;
        const T2& _d2;
  };
  ```

  ```C++{.line-numbers}
  void test()
  {
    Data<double,int> d1;//调用特化的int版本
    Data<int,double> d2;//调用基础的模板
    Data<int*,int*> d3;//调用特化的指针版本
    Data<int&,int&> d4(1,2);//调用特化的引用版本
  }
  ```
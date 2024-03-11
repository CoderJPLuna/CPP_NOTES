# Lesson06--string

## 1. 标准库中的string类

### 1.1 string类简介

1. 字符串是表示字符序列的类.
2. 标准的字符串类提供了对此类对象的支持,其接口类似与标准字符容器的接口,但添加了专门用于操作单字节字符字符串的设计特性.
3. `string` 类是使用 `char` (即作为它的字符类型,使用它的默认 `char_traits` 和分配器类型).
4. `string` 类是 `basic_string` 模板类的一个实例,它使用 `char` 来实例化 `basic_string` 模板类,并用 `char_traits` 和 `allocator` 作为 `basic_string` 的默认参数.
5. 注意,这个类独立于所使用的编码来处理字节,如果用来处理多字节或变长字符(如UTF-8)的序列,这个类的所有成员(如长度或大小)以及它的迭代器,将仍然按照字节(而不是实际编码的字符)来操作.

**总结:**
1. `string` 是表示字符串的字符串类.
2. 该类的接口与常规容器的接口基本相同,再添加了一些专门用来操作 `string` 的常规操作.
3. `string` 在底层实际是: `basic_string` 模板类的别名, `typedef basic_string<char,char_traits<char>,allocator<char>>string;` .
4. 不能操作多字节或者变长字符的序列.

在使用 `string` 类时,必须包含 `#include<string>` 头文件以及 `using namespace std;` .

### 1.2 string类的常用接口说明

1. string类对象的常见构造.
   |函数名称|功能说明|
   |-|-|
   |string()(重点)|构造空的string类对象，即空字符串|
   |string(const char* s)(重点)|用C-string来构造string类对象|
   |string(size_t n,char c)|string类对象中包含n个字符c|
   |string(const string& s)(重点)|拷贝构造函数|
   ```C++{.line-numbers}
    string s1;//构造空的string类对象s1
    string s2("Hello!");//用c格式字符串构造string类对象s2
    string s3(s2);//拷贝构造s3
    string s4(s2,2);//从第三个字符拷贝到字符串结束
    string s5(s2,1,4);//从第二个字符拷贝到第五个字符
    string s6(s2,2,9);//若超出,拷贝到字符串结束
    string s7(s2,2,string::npos)//static const size_t npos = -1
    ```
2. string类对象的容量操作
   |函数名称|功能说明|
   |-|-|
   |size(重点)|返回字符串有效字符长度|
   |length|返回字符串有效字符长度|
   |capacity|返回空间总大小|
   |empty(重点)|检测字符串是否为空串，是返回true，否则返回false|
   |clear(重点)|清空有效字符|
   |reserve(重点)|为字符串预留空间|
   |resize(重点)|将有效字符的个数改为n个，多出的空间用字符c填充|
   ```C++{.line-numbers}
   //size/clear/resize
   //注意:string类对象支持直接用cin和cout进行输入和输出
   string s("hello");
   cout << s.size() << endl;
   cout << s.length() << endl;
   cout << s.capacity() << endl;
   cout << s << endl;
   //将s中的字符串清空,注意清空时只是将size清0,不改变底层空间的大小
   s.clear();
   cout << s.size() << endl;
   cout << s.capacity() << endl;
   //将s中有效字符个数增加到10个,多出位置用'a'进行填充
   s.resize(10,'a');
   cout << s.size() << endl;
   cout << s.capacity() << endl;
   //将s中有效字符个数增加到15个,多出位置用缺省值'\0'进行填充
   s.resize(15);
   cout << s.size() << endl;
   cout << s.capacity() << endl;
   cout << s << endl;
   //将s中有效字符个数缩小到5个
   s.resize(5);
   cout << s.size() << endl;
   cout << s.capacity() << endl;
   cout << s << endl;
   ```
   `void push_back(const value_type& val);`  
   `void push_back(value_type&& val);` *(C++11)*
   `push_back()` 函数可以在 `string` 的末尾插入一个字符,当 `string` 容量不足时,会自动进行扩容,每次扩容约为之前的1.5倍.  
   扩容时会开辟新的更大的空间,并进行拷贝,再释放原先的空间.
   因此频繁的扩容会导致效率降低.
   ```C++{.line-numbers}
   string s;
   size_t = s.capacity();
   cout << "making s grow:" << endl;
   for(int i =0 ;i < 100;i++)
   {
      s.push_back('c');
      if(sz != s.capacity())
      {
         sz = s.capacity();
         cout << "capacity changed:" << sz << endl;
      }
   }
   ```
   > making s grow:  
   > capacity changed:31//实际为32，包含'\0'，且遵循内存对齐规则  
   > capacity changed:47  
   > capacity changed:70  
   > capacity changed:105  

   ```C++{.line-numbers}
   string s;
   s.reserve(100);//reserve为string开辟指定空间大小,实际空间大小会遵循内存对齐规则而调整
   size_t sz = s.capacity();
   cout << sz << endl;
   s.resize(100);//resize指定capacity为100同时,也指定size为100,默认指定字符为'\0'
   cout << sz << endl;
   s.resize(100,'c');//指定字符为'c'
   ```
   > 111//size为0 capacity为111  
   > 111//size为100 capacity为111  
   ```C++{.line-numbers}
   string s("hello world");//11个字符,size为11,capacity为15
   s.resize(5);//size为5,capacity为15,world消失
   s.resize(20,'x');//size为20,capacity为31,hello之后均为'x'
   ```
   **注意:**
   1. `size()` 与 `length()` 底层实现原理完全相同,引入 `size()` 的原因是为了与其它容器的接口保持一致,一般情况下基本都是用 `size()` .
   2. `clear()` 只是将 `string` 中有效字符 `size` 清空,不改变底层空间 `capacity` 大小.
   3. `resize(size_t n)` 与 `resize(size_t n,char c)` 都是将字符串中有效字符个数改为n,不同的是当字符个数增多时: `resize(n)` 用0来填充多出的元素空间, `resize(size_t n,char c)` 用字符c来填充多出的元素空间. **`resize()` 在改变元素个数时,如果是元素个数增多,可能会改变底层容量的大小,如果是将元素个数减少,底层空间总大小不变** .
   4. `reserve(size_t res_arg = 0)` :为 `string` 预留空间,不改变有效元素个数,当 `reserve` 的参数小于 `string` 的底层空间总大小时, `reserve` 不会改变容量大小.
3. string类对象的访问及遍历操作
   |函数名称|功能说明|
   |-|-|
   |operator[] (重点)|返回pos位置的字符，const string类对象调用|
   |begin+end|begin获取一个字符的迭代器+end获取最后一个字符下一个位置的迭代器|
   |rbegin+rend|反向迭代器|
   |范围for|C++支持更简洁的范围for的新遍历方式|
   **迭代器:**
   ```C++{.line-numbers}
   string s;
   string::iterator it = s.begin();//iterator为定义在string中的类
   while(it! = s.end())
   {
      cout << *it << " ";
      ++it;
   }
   cout << endl;```
4. string类对象的修改操作
   |函数名称|功能说明|
   |-|-|
   |push_back|在字符串末尾插入字符|
   |append|在字符串末尾追加一个字符串|
   |operator+=(重点)|在字符串后追加字符串str|
   |c_str(重点)|返回C格式字符串|
   |find+npos(重点)|从字符串pos位置开始往后找字符，返回该字符在字符串中的位置|
   |rfind|从字符串pos位置开始往前找字符，返回该字符在字符串中的位置|
   |substr|在str中从pos位置开始，截取n个字符，然后将其返回|
   ```C++{.line-numbers}
   string str;
   str.push_back(' ');//在str后插入空格
   str.append("hello");//在str后追加一个字符串"hello"
   str += 'b';//在str后追加一个字符'b'
   str += "it";//在str后追加一个字符串"it"
   cout << s << endl;//调用string重载的operator<<
   cout << s.c_str() << endl;//直接输出const char*,遇到'\0'就结束
   string s("string.cpp.zip");
   size_t pos1 = s.find('.');
   size_t pos2 = s.rfind('.');
   //substr()从指定位置复制子字符串
   cout << s.substr(pos1) << endl;//输出".cpp.zip"
   cout << s.substr(pos2) << endl;//输出".zip"
   ```
   **注意:**
   1. 在 `string` 尾部追加字符时, `s.push_back(c)` / `s.append(1,c)` / `s+='c'` 三种的实现方式差不多,一般情况下 `string` 类的 `+=` 操作用的比较多, `+=` 操作不仅可以连接单个字符,还可以连接字符串.
   2. 对 `string` 操作时,如果能够大概预估到放多少字符,可以先通过 `reserve` 把空间预留好.
5. string类非成员函数
   |函数|功能说明|
   |-|-|
   |operator+|尽量少用，因为传值返回，导致深拷贝效率低|
   |operator>>(重点)|输入运算符重载|
   |operator<<(重点)|输出运算符重载|
   |getline(重点)|获取一行字符串|
   |relational operators(重点)|大小比较|

## 2. string类的模拟实现

### 2.1 经典的string类问题

```C++{.line-numbers}
class string
{
public:
/*
   string()
   :_str(new char[1])
   {*_str = '\0';}
*/
   //string(const char* str = '\0');错误示范
   //string(const char* str = nullptr);错误示范
   string(const char* str = "")
   {
      //构造string类对象时，如果传递nullptr指针，认为程序非法，此处断言
      if(nullptr == str)
      {
         assert(false);
         return;
      }
      _str = new char[strlen(str+1)];
      strcpy(_str,str);
   }
   ~string()
   {
      if(_str)
      {
         delete[] _str;
         _str = nullptr;
      }
   }
private:
   char* _str;
};
void Teststring()
{
   string s1("Hello World!");//s1调用构造函数string(const char* str)
   steing s2(s1);//s2调用拷贝函数来创建,该类没有显示定义,所以调用默认拷贝函数
}
```

**说明:**
上述string类没有显式定义其拷贝构造函数与赋值运算符重载,此时编译器会生成默认的,当用s1构造s2时,编译器会调用默认的拷贝构造.最终导致的问题是,s1,s2共用同一块内存空间,在释放时同一块空间被释放两次而引起程序崩溃,这种拷贝方式称为浅拷贝.

### 2.2 浅拷贝

浅拷贝也称位拷贝,编译器只是将对象中的值拷贝过来.如果对象中管理资源,最后就会导致多个对象共享同一份资源,当一个对象销毁时就会将该资源释放掉,而此时另一些对象不知道该资源已经被释放,所以当继续对该空间进行释放操作时,就会发生违规访问.
浅拷贝是按位拷贝对象,它会创建一个新对象,这个对象有着原始对象属性值的一份精确拷贝.如果属性是基本类型,拷贝的就是基本类型的值;如果属性是内存地址(引用类型),拷贝的就是内存地址,因此如果其中一个对象改变了这个地址,就会影响到另一个对象,即默认拷贝构造函数只是对对象进行浅拷贝复制(逐个成员一次拷贝),即只复制对象空间而不复制资源.

### 2.3 深拷贝

一个引用对象一般来说由两个部分组成:一个具名的Handle,也就是我们所说的声明(如变量)和一个内部不具名的对象,也就是具名Handle的内部对象.它在Manged Heap(托管堆)中分配,一般由新增引用对象的New方法进行创建.深拷贝是指源对象与拷贝对象互相独立,其中任何一个对象的改动都不会对另外一个对象造成影响.

```C++{.line-numbers}
#include<iostream>
using namespace std;
class Pata
{
private:
   int a;
public:
   Pata(int a)
   {
      this->a = a;
   }
   Pata(Pata& ast)//系统默认另开辟一片空间，把ast复制到这片空间中去
   {
      this->a = ast.a;
   }
};
int main()
{
   Pata s1(2);
   Pata s2(s1);
   return 0;
}
```

this和ast地址不同.  
对象作为函数参数传递时,调用拷贝构造函数.

#### 2.3.1 传统string类的模拟实现

传统写法:开辟空间并复制内容。

1. 迭代器
   指针是良好的迭代器。

   ```C++{.line-numbers}
   /*
   typedef char* iterator;
   iterator begin()
   {
      return _str;
   }
   iterator end()
   {
      return _str + _size;
   }
   */
   typedef const char* iterator;
   iterator begin()const
   {
      return _str;
   }
   iterator end()const
   {
      return _str + _size;
   }
   //这里使用迭代器实现遍历打印
   void print(const st::string& s)
   {
      st::string::iterator it = s.begin();
      while(it < s.end())
      {
         cout << *it;
         ++it;
      }
   }
   ```

2. operator[]

   ```C++{.line-numbers}
   char& operator[](size_t i)
   {
      assert(i < _size);
      return _str[i];
   }
   ```

3. size()

   ```C++{.line-numbers}
   size_t size()
   {
      return _size;
   }
   ```

4. c_str()

   ```C++{.line-numbers}
   const char* c_str()
   {
      return _str;
   }
   ```

5. operator=()

   ```C++{.line-numbers}
   //现代写法
   //s1=s2
   string& operator = (string st)
   {
      this->swap(st);
      return *this;
   }
   /*
   string& operator=(const string& st)
   {
      if(this != &st)
      {
         delete[] _str;
         _str = new char[strlen(st._str) + 1];
         strcpy(_str,st._str);
      }
      return *this;
   }
   */
   ```

6. reserve()

   ```C++{.line-numbers}
   void reserve(size_t n)
   {
      if(n > _capacity)
      {
         char* tem = new char[n + 1];
         strcpy(tem,_str);
         delete[] _str;
         _str = tem;
         _capacity = n;
      }
   }
   ```

7. resize()

   ```C++{.line-numbers}
   void resize(size_t n,char ch = '\0')
   if(n < _size)
   {
      _str[n] = '\0';
      _size = n;
   }
   else
   {
      if(n > _capacity)
      {
         reserve(n);
      }
      for(size_t i = _size;i < n;i++)
      {
         _str[n] = '\0';
         _size = n;
      }
   }
   ```

8. push_back()

   ```C++{.line-numbers}
   void push_back(char ch)
   {
      if(_size == _capacity)
      {
         reserve(2*_capacity);//容量不足时开辟空间
      }
      _str[_size] = ch;
      ++_size;
      _str[_size] = '\0';
   }
   ```

9. append()

   ```C++{.line-numbers}
   void append(const char* ch)
   {
      size_t len = strlen(ch)+1;
      if(_size+len > _capacity)
      {
         reserve(len + _size);
      }
      strcpy(_str + _size,ch);
      _size += len;
   }
   ```

10. operator+=

    ```C++{.line-numbers}
    string& operator+=(char s)
    {
       push_back(s);
       return *this;
    }
    string& operator+=(const string &ss)
    {
       append(ss._str);
       return *this;
    }
    ```

11. insert()

    ```C++{.line-numbers}
    void insert(size_t n,char ch)
    {
       if(_size == _capacity)
       {
          size_t capacity = (_capacity == 0) ? 8 : _capacity*2;
          reserve(capacity);
       }
       int end = _size+1;
       while(n < end)
       {
          _str[end] = _str[end - 1];
          --end;
       }
       _str[n] = ch;
       _size++;
    }
    void insert(size_t n,const char* ch)
    {
       size_t len = strlen(ch);
       if(_size + len > _capacity)
       {
          reserve(_size + len);
       }
       while(end >= n+len)
       {
          _str[end] = _str[end - len];
          --end;
       }
       strncpy(_str + n,ch,len);
       _size += len;
    }
    ```

12. operator>

    ```C++{.line-numbers}
    bool operator>(string s1,string s2)
    {
       size_t i = 0,j = 0;
       while(i < s1.size() && s2.size())
       {
          if(s1[i] > s2[j])
          {
             return true;
          }
          else if(s1[i] < s2[i])
          {
             return false;
          }
          else
          {
             i++;
             j++;
          }
       }
       if(i < s1.size())
       {
          return true;
       }
       else if(j < s2.size())
       {
          return false;
       }
       else
       {
          return true;
       }
    }
    ```

### 2.3.2现代string类的模拟实现

现代写法:借助中间变量,交换指针.

```C++{.line-numbers}
#include<iostream>
#include<assert.h>
using namespce std;
namespace st
{
   class string
   {
   private:
      char* _str;
      size_t _size;
      size_t _capacity;
      static const size_t npos;
   public:
      void swap(string& st)
      {
         ::swap(_str,st._str);
         ::swap(_size,st._size);
         ::swap(_capacity,st._capacity);
      }
      string(const char* st='')
      {
         _size = strlen(st);
         _capacity = _size;
         _str = new char[_capacity + 1];
         strcpy(_str,st);
      }
      //构造函数采用深拷贝
      //拷贝构造的现代写法
      //s2(s1)
      string(const string& st)
      :_str(nullptr)//s是局部对象出了作用域这个空间就会释放,但不能对随机指向的空间释放
      ,_size(0)
      ,_capacity(0)
      {
         string s(st._str);//s这个临时变量指向st._str
         swap(s);
      }
      /*传统写法
      string(const string& st)
      :_str(new char[strlen(st._str) + 1])
      {
         strcpy(_str,st._str);
      }
      */
      //重载函数现代写法
      //s1=s2
      string& operator=(string st)
      {
         this->swap(st);
         return *this;
      }
      /*传统写法
      string& operator=(const string& st)
      {
         if(this != &st)
         {
            delete[] _str;
            _str = new char[strlen(st._str) + 1];
            strcpy(_str,st._str);
         }
         return *this;
      }
      */
   }
};
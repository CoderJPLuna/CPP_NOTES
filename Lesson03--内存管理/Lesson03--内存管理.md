# Lesson03--内存管理

## 1. C/C++内存分布

```C++{.line-numbers}
int globalVar = 1;//全局变量
static int staticGlobalVar = 1;//静态全局变量
//这两个变量的作用域是全局的
//区别在于链接属性不一样:globalVar在所有文件中可见,staticGlobalVar只在当前文件可见
void Test()
{
    static int staticVar = 1;//静态局部变量
    //生命周期是全局的,但作用域仅在Test函数中
    int localVar = 1;//局部变量
    int num1[10] = {1,2,3,4};//局部数组
    char char2[] = "abcd";//局部数组
    const char* pChar3 = "abcd";//局部指针变量,存储着字符常量
    int* ptr1 = (int*)malloc(sizeof(int)*4);//
    int* ptr2 = (int*)calloc(4,sizeof(int));//
    int* ptr3 = (int*)realloc(ptr2,sizeof(int)*4);//
    free(ptr1);
    free(ptr3);
}
//globalVar在数据段(也叫静态区)
//staticGlobalVar在数据段
//staticVar在数据段
//localVar在栈上
//num1在栈上
//char2在栈上
//*char2在栈上
//pChar3在栈上
//*pChar3在代码段(也叫常量区)
//ptr1在栈上
//*ptr1在堆上
//sizeof(num1) = 40
//sizeof(char2) = 5("\0")
//strlen(char2) = 4("strlen不计算'\0'")
//sizeof(pChar3) = 4/8(32位/64位)
//strlen(pChar3) = 4
//sizeof(ptr1) = 4/8
```

1. **栈** 又叫堆栈,非静态局部变量/函数参数/返回值等等,栈是向下增长的.
2. **内存映射段** 是高效的I/O映射方式,用于装载一个共享的动态内存库.用户可使用系统接口创建共享内存,做进程间通信.
3. **堆** 用于程序运行时动态内存分配,堆是可以向上增长的.
4. **数据段** 用于存储全局数据和静态数据.
5. **代码段** 用于存放可执行的代码/只读常量.

### 1.1 C/C++中程序内存区域划分

|内存空间|
|:-----:|
|内核空间<br>(用户代码不能读写)|
||
|栈<br>(向下增长)|
||
|共享映射段<br>(文件映射、动态库、匿名映射)|
||
|堆<br>(向上增长)|
||
|数据段<br>(全局数据、静态数据)|
|代码段<br>(可执行代码/只读常量)|

## 2. C语言中动态内存管理方式

### 2.1 malloc/calloc/realloc/free

```C++{.line-numbers}
int* p1 = (int*)malloc(sizeof(int));
int* p2 = (int*)calloc(4,sizeof(int));//第一个参数为要分配元素的个数,第二个参数为元素的大小
//calloc()函数将分配的内存全部初始化为零
int* p3 = (int*)realloc(p2,sizeof(int)*10);
free(p1);
free(p3);//p2被扩容成p3,此处不需要free(p2);
```

> **malloc/calloc/realloc的区别**
>
> 1. malloc用于动态分配一块内存.  
> 2. calloc用于动态分配一块内存,并初始化为0.  
> 3. realloc用于修饰一块原先已经分配好的内存块大.

## 3. C++内存管理方式

C语言内存管理方式在C++中可以继续使用,但有些地方就无能为力而且使用起来比较麻烦,因此C++又提出了自己的内存管理方式:通过 `new` 和 `delete` 操作符进行动态内存管理.

### 3.1 new/delete操作内置类型

```C++{.line-numbers}
//动态申请一个int类型的空间
int* ptr1 = new int;
//动态申请一个int类型的空间并初始化为10
int* ptr2 = new int(10);
//动态申请10个int类型的空间
int* ptr3 = new int[10];
delete ptr1;
delete ptr2;
delete[] ptr3;
```

**注意:**  
申请和释放单个元素的空间,使用 `new` 和 `delete` 操作符;申请和释放连续的空间,使用 `new[]` 和 `delete[]` .

### 3.2 new和delete操作自定义类型

```C++{.line-numbers}
class A
{
public:
    A(int a = 0)
    :_a(a)
    {
        cout << "A()" << endl;
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
    //new/delete和malloc/free最大的区别是new/delete对于自定义类型除了开辟空间还会调用构造函数和析构函数
    A* p1 = (A*)malloc(sizeof(A));//申请空间
    A* p2 = new A(1);//申请空间+构造函数
    free(p1);//释放空间
    delete p2;//析构函数+释放空间

    //对于内置类型几乎是一样的
    int* p3 = (int*)malloc(sizeof(int));
    int* p4 = new int;
    free(p3);
    delete p4;

    A* p5 = (A*)malloc(sizeof(A)*10);
    A* p6 = new A[10];
    free(p5);
    delete[] p6;
    return 0;
}
```

对于自定义类型的空间时, `new` 会调用构造函数, `delete` 会调用析构函数,而 `malloc` 与 `free` 不会.

## 4. operator new与operator delete函数

### 4.1 operator new与operator delete函数

`new` 和 `delete` 是用户进行动态内存申请和释放的操作符, `operator new` 和 `operator delete` 是系统提供的全局函数. `new` 在底层调用 `operator new` 全局函数来申请空间, `delete` 在底层通过 `operator delete` 全局函数来释放空间.

`operator new` :该函数实际通过 `malloc` 来申请空间,当 `malloc` 申请空间成功时直接返回;申请空间失败,尝试执行空间不足应对措施,如果用户设置了该应对措施,则继续申请,否则抛出异常.

```C++{.line-numbers}
void* __CRTDECL operator new(size_t size) _THROW1(_STD bad_alloc)
{
    //try to allocate size bytes
    void* p;
    while((p = malloc(size)) == 0)
        if(_callnewh(size) == 0)
        {
            //report no memory
            //如果申请内存失败,这里会抛出bad_alloc类型异常
            static const std::bad_alloc nomem;
            _RAISE(nomem);
        }
        return(p);
}
```

`operator delete` :该函数最终是通过free来释放空间.

```C++{.line-numbers}
void operator delete(void* pUserData)
{
    _CrtMemBlockHeader* pHead;
    RTCCALLBSCK(_RTC_Free_hook,(pUserData,0));
    if(pUserDate==NULL)
        return;
    _mlock(_HEAP_LOCK);//block other threads
    __TRY
        //get a pointer to memory block header
        pHead=pHdr(pUserData);
        //verify block type
        _ASSERTE(_BLOCK_TYPE_IS_VALID(pHead->nBlockUse));
    __FINALLY
        _munlock(_HEAP_LOCK);//release other threads
    _END_TRY_FINALLY
    return;
}
/*
free的实现
#define free(p) _free_dbg(p,_NORMAL_BLOCK)
*/
```

`operator free` 与 `free` 无区别。

通过上述两个全局函数的实现知道, `operator new` 实际也是通过 `malloc` 来申请空间,如果 `malloc` 申请空间成功就直接返回,否则执行用户提供的空间不足应对措施,如果用户提供该措施就继续申请,否则就抛异常. `operator delete` 最终是通过 `free` 来释放空间.

### 4.2 operator new与operator delete的类专属重载

下面代码演示了,针对链表的节点 `ListNode` 通过类专属重载 `operator new` / `operator delete` ,实现链表节点使用内存池申请和释放内存,提高效率.

```C++{.line-numbers}
struct ListNode
{
    ListNode* _next;
    ListNode* _prev;
    int _data;
    void* operator new(size_t n)
    {
        void* p = nullptr;
        p = allocator<ListNode>().allocate(1);
        cout << "memory pool allocate" << endl;
        return p;
    }
    void operator delete(void* p)
    {
        allocator<ListNode>().deallocate((ListNode*)p,1);
        cout << "memory pool deallocate" << endl;
    }
};
class List
{
public:
    List()
    {
        _head = new ListNode;//全局operator new+构造函数
        _head->_next = _head;
        _head->_prev = _head;
    }
    ~List()
    {
        ListNode* cur = _head->_next;
        while(cur! = _head)
        {
            ListNode* next = cur->_next;
            delete cur;
            cur = next;
        }
        delete _head;
        _head = nullptr;
    }
private:
    ListNode* _head;
};
int main()
{
    List l;
    return 0;
}
```

## 5. new和delete的实现原理

### 5.1 内置类型

如果申请的是内置类型的空间, `new` 和 `malloc` , `delete` 和 `free` 基本类似,不同的地方是: `new` / `delete` 申请和释放的是单个元素的空间, `new[]` 和 `delete[]` 申请和释放的是连续空间,而且 `new` 在申请空间失败时会抛异常, `malloc` 会返回 `NULL` .

### 5.2 自定义类型

* `new` 的原理
  1. 调用 `operator new` 函数申请空间
  2. 在申请的空间上执行构造函数,完成对象的构造
* `delete` 的原理
  1. 在空间上执行析构函数,完成对象中资源的清理工作
  2. 调用 `operator delete` 函数释放对象的空间
* `new []` 的原理
  1. 调用 `operator new[]` 函数,在 `operator new[]` 中实际调用 `operator new` 函数完成N个对象空间的申请
  2. 在申请的空间上执行N次构造函数
* `delete[]` 的原理
  1. 在释放的对象空间上执行N次析构函数,完成N个对象中资源的清理
  2. 调用 `operator delete[]` 释放空间,实际在 `operator delete[]` 中调用 `operator delete` 来释放空间

> 使用 `new` 对类类型开辟连续空间时,若类类型的成员函数中实现了析构函数,编译器会在开辟空间的开头额外开辟4个字节的空间,用以存放元素的个数,原因是为了记录元素个数,以便在 `delete` 时知道要调用析构函数的次数.
> ```C++{.line-numbers}
> class A
> {
> public:
>   A(){};
>   ~A(){};
> };
> int main()
> {
>   A* pa = new A[10];
>   delete pa;//程序会崩溃
>   return 0;
> }
> ```
> 程序崩溃的原因是:在开辟空间时,会在开辟空间的目标位置前额外开辟4个字节的空间,当 `delete` 时, `delete` 没有从空间的开头销毁,而是从目标位置开始销毁,程序崩溃.
> 若未显式实现析构函数时,则不会在目标位置前额外开辟4个字节的空间.

## 6. 定位new表达式(placement-new)

定位new表达式是在已分配的原始内存空间中调用构造函数初始化一个对象.

**使用格式:**
`new(place_address)type` 或者 `new(place_address)type(initializer-list)`  
`place_address` 必须是一个指针, `initializer-list` 是类型的初始化列表.

**使用场景:**
定位 `new` 表达式在实际中一般是配合内存池使用.因为内存池分配出的内存没有初始化,所以如果是自定义类型的对象,需要使用 `new` 的定义表达式进行显式调用构造函数进行初始化.

```C++{.line-numbers}
class A
{
public:
    A(int a = 0)
    :_a(a)
    {
        cout << "A()" << this << endl;
    }
    ~A()
    {
        cout << "~A()" << this << endl;
    }
private:
    int _a;
};
//replacement new
int main()
{
    //p1现在指向的是与A对象相同大小的一段空间,还不是一个对象,因为构造函数没有执行
    A* p1 = (A*)malloc(sizeof(A));
    new(p1)A;//注意:如果A类的构造函数有参数时,此处需要传参
    p1->~A();
    free(p1);
    A* p2=(A*)operator new(sizeof(A));
    new(p2)A(10);
    p2->~A();
    operator delete(p2);
}
```

> **malloc/free和new/delete的区别:**  
> `malloc` / `free` 和 `new` / `delete` 的共同点是:都是从堆上申请空间,并且需要用户手动释.。  
> 不同的地方是:  
> 1. `malloc` 和 `free` 是函数, `new` 和 `delete` 是操作符.  
> 2. `malloc` 申请的空间不会初始化,new可以初始化.  
> 3. `malloc` 申请空间时,需要手动计算空间大小并传递,new只需在其后跟上空间的类型即可.  
> 4. `malloc` 的返回值为 `void*` ,在使用时必须强制类型转换,new不需要,因为new后跟的是空间的类型.  
> 5. `malloc` 申请空间失败时,返回的是 `NULL` ,因此使用时必须判空, `new` 不需要,但是 `new` 需要捕获异常.  
> 6. 申请自定义类型对象时, `malloc` / `free` 只会开辟空间,不会调用构造函数与析构函数,而 `new` 在申请空间后会调用构造函数完成对象的初始化, `delete` 在释放空间前会调用析构函数完成空间中资源的清理.  

## 7. 内存泄漏

### 7.1 什么是内存泄漏

**内存泄漏** 指因为疏忽或错误造成程序未能释放已经不再使用的内存的情况.内存泄漏并不是指内存在物理上的消失,而是应用程序分配某段内存后,因为设计错误,失去了对该段内存的控制,因而造成了内存的浪费.

### 7.2 内存泄漏的危害

**内存泄露的危害** 长期运行的程序出现内存泄漏,影响很大,如操作系统,后台服务等等,出现内存泄漏会导致响应越来越慢,最终卡死.

```C++{.line-numbers}
void MemoryLeaks()
{
    //1.内存申请了忘记释放
    int* p1 = (int*)malloc(sizeof(int));
    int* p2 = new int;
    //2.异常安全问题
    int* p3 = new int[10];
    Func();//这里Func函数抛异常导致delete[]p3未执行，p3没被释放
    delete[] p3；
}
```

### 7.3 内存泄漏分类

C/C++程序中一般我们关心两种方面的内存泄漏

* 堆内存泄漏(Heap Leak)
  堆内存指的是程序中依据需要分配通过 `malloc` / `calloc` / `realloc` / `new` 等从堆中分配的一块内存,用完后必须通过调用相应的 `free` 或者 `delete` 删掉.假设程序的设计错误导致这部分内存没有被释放,那么以后这部分空间将无法再被使用,就会产生Heap Leak.
* 系统资源泄漏
  指程序使用系统分配的资源,比如套接字,文件描述符,管道等没有使用对应的函数释放掉,导致系统资源的浪费,严重可导致系统效能减少,系统执行不稳定.

> **如何一次在堆上申请4G的内存:**
> ```C++{.line-numbers}
> //将程序编译成x64的进程
> #include<iostream>
> using namespace std;
> int main()
> {
>     void* p=new char[0xfffffffful];
>     cout<<"new:"<<p<<endl;
>     return 0;
> }
> ```

### 7.4 如何检测内存泄漏

在VS下,可以使用Windows操作系统提供的 `_CrtDumpMemoryLeak()` 函数进行简单检测,该函数只报出大概泄漏了多少个字节,没有其它更准确的位置信息.

### 7.5 如何避免内存泄漏

1. 工程前期良好的实际规范,养成良好的编码规范,申请的内存空间记着匹配的去释放.如果碰上异常时,就算注意了释放,还是可能会出问题,需要智能指针来管理.
2. 采用RAII思想或智能指针来管理资源.
3. 有些公司内部规范使用内部实现的私有内存管理库,这套库自带内存泄漏检测的功能选项.
4. 使用内存泄漏工具检测.
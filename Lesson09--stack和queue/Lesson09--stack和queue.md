# Lesson09--stack和queue

## 1. stack的介绍和使用

### 1.1 stack的介绍

1. `stack` 是一种容器适配器,专门用在具有后进先出操作的上下文环境中,其删除只能从容器的一端进行元素的插入与提取操作.
2. `stack` 是作为容器适配器被实现的,容器适配器即是对特定类封装作为其底层的容器,并提供一组特定的成员函数来访问其元素,将特定类作为其底层的,元素特定容器的尾部(即栈顶)被压入和弹出.
3. `stack` 的底层容器可以是任何标准的容器类模板或者一些其它特定的容器类,这些容器类应该支持以下操作:
   * `empty` 判空操作
   * `back` 获取尾部元素操作
   * `push_back` 尾部插入元素操作
   * `pop_back` 尾部删除元素操作
4. 标准容器 `vector` , `deque` , `list` 均符合这些需求,默认情况下,如果没有为 `stack` 指定特定的底层容器,默认情况下使用 `deque` .

### 1.2 stack的使用

| 函数说明 |       接口说明        |
| :------: | :-------------------: |
| stack()  |      构造空的栈       |
| empty()  |   检测stack是否为空   |
|  size()  | 返回stack中元素的个数 |
|  top()   |  返回栈顶元素的引用   |
|  push()  | 将元素val压入stack中  |
|  pop()   | 将stack中尾部元素弹出 |

### 1.3 stack的模拟实现

```C++{.line-numbers}
#include<vector>
namespace my_stack
{
   template<class T>
   class stack
   {
   public:
      stack(){}
      void push(const T& x){_c.push_back(x);}
      void pop(){_c.pop_back();}
      T& top(){return _c.back();}
      const T& top()const {return _c.back();}
      size_t size()const {return _c.size();}
      bool empty()const {return _c.empty();}
   private:
      std::vector<T> _c;
   };
}
```

## 2. queue的使用和介绍

### 2.1 queue的介绍

1. 队列是一种容器适配器,专门用于在FIFO上下文(先进先出)中操作,其中从容器一端插入元素,另一端
提取元素.
2. 队列作为容器适配器实现,容器适配器即将特定容器类封装作为其底层容器类, `queue` 提供一组特定的
成员函数来访问其元素.元素从队尾入队列,从队头出队列.
3. 底层容器可以是标准容器类模板之一,也可以是其他专门设计的容器类.该底层容器应至少支持以下操
作:
   * `empty` 检测队列是否为空
   * `size` 返回队列中有效元素的个数
   * `front` 返回队头元素的引用
   * `back` 返回队尾元素的引用
   * `push_back` 在队列尾部入队列
   * `pop_front` 在队列头部出队列
4. 标准容器类 `deque` 和 `list` 满足了这些要求.默认情况下,如果没有为 `queue` 实例化指定容器类,则使用标
准容器 `deque` .

### 2.2 queue的使用

| 函数声明 |          口说明          |
| :------: | :----------------------: |
| queue()  |       构造空的队列       |
| empty()  |     检测队列是否为空     |
|  size()  | 返回队列中有效元素的个数 |
| front()  |    返回队头元素的引用    |
|  back()  |    返回队尾元素的引用    |
|  push()  |  在队尾将元素val入队列   |
|  pop()   |     将队头元素出队列     |

### 2.3 queue的模拟实现

因为 `queue` 的接口中存在头删和尾插,因此使用 `vector` 来封装效率太低,故可以借助 `list` 来模拟实现 `queue` .

```C++{.line-numbers}
#include <list>
namespace bite
{
 template<class T>
 class queue
 {
 public:
 queue() {}
 void push(const T& x) {_c.push_back(x);}
 void pop() {_c.pop_front();}
 T& back() {return _c.back();}
 const T& back()const {return _c.back();}
 T& front() {return _c.front();}
 const T& front()const {return _c.front();}
 size_t size()const {return _c.size();}
 bool empty()const {return _c.empty();}
 private:
 std::list<T> _c;
 };
}
```

## 3. priority_queue的介绍和使用

### 3.1 priority_queue的介绍

1. 优先队列是一种容器适配器,根据严格的弱排序标准,它的第一个元素总是它所包含的元素中最大的.
2. 此上下文类似于堆,在堆中可以随时插入元素,并且只能检索最大堆元素(优先队列中位于顶部的元
素).
3. 优先队列被实现为容器适配器,容器适配器即将特定容器类封装作为其底层容器类, `queue` 提供一组特
定的成员函数来访问其元素.元素从特定容器的"尾部"弹出,其称为优先队列的顶部.
4. 底层容器可以是任何标准容器类模板,也可以是其他特定设计的容器类.容器应该可以通过随机访问迭
代器访问,并支持以下操作:
   * `empty()` 检测容器是否为空
   * `size()` 返回容器中有效元素个数
   * `front()` 返回容器中第一个元素的引用
   * `push_back()` 在容器尾部插入元素
   * `pop()` 删除容器尾部元素
5. 标准容器类 `vector` 和 `deque` 满足这些需求.默认情况下,如果没有为特定的 `priority_queue` 类实例化指
定容器类,则使用 `vector` .
6. 需要支持随机访问迭代器,以便始终在内部保持堆结构.容器适配器通过在需要时自动调用算法函数 `make_heap` , `push_heap` 和 `pop_heap` 来自动完成此操作.

### 3.2 priority_queue的使用

优先级队列默认使用 `vector` 作为其底层存储数据的容器,在 `vector` 上又使用了堆算法将 `vector` 中元素构造成堆的结构,因此 `priority_queue` 就是堆,所有需要用到堆的位置,都可以考虑使用 `priority_queue` .

1. 注意:默认情况下 `priority_queue` 是大堆.
2. 如果在 `priority_queue` 中放自定义类型的数据,用户需要在自定义中提供 `>` 或 `<` 的重载.

|                   函数声明                    |                 接口说明                  |
| :-------------------------------------------: | :---------------------------------------: |
| priority_queue()/priority_queue()(first,last) |          构造一个空的优先级队列           |
|                    empty()                    |  检测优先级队列是否为空,返回true或false   |
|                     top()                     | 返回优先级队列中最大(最小元素),即堆顶元素 |
|                    push(x)                    |          在优先级队列中插入元素x          |
|                     pop()                     | 删除优先级队列中最大(最小)元素,即堆顶元素 |

## 4. 容器适配器

### 4.1 适配器

**适配器是一种设计模式** .设计模式是一套被反复使用的,多数人知晓的,经过分类编目的,代码设计经验的总结.该模式是将一个类的接口转换成客户希望的另一个接口.

### 4.2 STL标准库中stack和queue的底层结构

虽然 `stack` 和 `queue` 中也可以存放元素,但在STL中并没有将其划分在容器的行列,而是将其称为 **容器适配器** ,这是因为 `stack` 和 `queue` 只是堆其它容器的接口进行了包装,STL中 `stack` 和 `queue` 默认使用 `deque` .( `deque` 头/尾插入和删除效率较高,且支持下标的随机访问).

```C++{.line-numbers}
template<class T,class Container = deque<T>> class stack;
template<class T,class Container = deque<T>> class queue;
template<class T,class Container = vector<T>,class Compare = less<typename Container::value_type>> class priority_queue;
```

### 4.3 deque介绍

#### 4.3.1 deque的原理介绍

**deque(双端对列)** :是一种双开口的"连续"空间的数据结构,双开口的含义是:可以在头尾两端进行插入和删除操作,且时间复杂度为 $O(1)$ ,与 `vector` 比较 ,头插效率高,不需要搬移元素;与 `list` 比较,空间利用率比较高.

** `deque` 并不是真正连续的空间,而是由一段段连续的小空间拼接而成,实际 `deque` 类似一个动态二维数组.**

双端队列底层是一段假象的连续空间,实际是分段连续的,为了维护其"整体连续"以及随机访问的假象,就需要靠 `deque` 迭代器来实现.

#### 4.3.2 deque的缺陷

与 `vector` 比较, `deque` 的优势是:头部插入和删除时,不需要搬移元素,效率特别高,而且在扩容时,也不需要搬移大量的元素,因此其效率是比 `vector` 高的.

与 `list` 比较,其底层是连续空间,空间利用率比较高,不需要存储额外字段.

但是, `deque` 有一个致命缺陷:不适合遍历,因为在遍历时, `deque` 的迭代器要频繁的去检测其是否移动到某段小空间的边界,导致效率低下,而序列式场景中,可能需要经常遍历,因此在实际中,需要线性结构时,大多数情况下优先考虑 `vector` 和 `list` , `deque` 的应用并不多,而目前能看到的一个应用就是,STL用其作为 `stack` 和 `queue` 的底层数据结构.

### 4.4 为什么选择deque作为stack和queue的底层默认容器

`stack` 是一种后进先出的特殊线性数据结构,因此只要具有 `push_back()` 和 `pop_back()` 操作的线性结构,都可以作为 `stack` 的底层容器,比如 `vector` 和 `list` 都可以; `queue` 是先进先出的特殊线性数据结构,只要具有 `push_back` 和 `pop_front` 操作的线性结构,都可以作为 `queue` 的底层容器,比如 `list` .但是STL中对 `stack` 和 `queue` 默认选择 `deque` 作为其底层容器,主要是因为:
1. `stack` 和 `queue` 不需要遍历(因此 `stack` 和 `queue` 没有迭代器),只需要在固定的一端或者两端进行操作.
2. 在 `stack` 中元素增长时, `deque` 比 `vector` 的效率高(扩容时不需要搬移大量数据); `queue` 中的元素增长时, `deque` 不仅效率高,而且内存使用率高.

结合了 `deque` 的优点,而完美的避开了其缺陷.
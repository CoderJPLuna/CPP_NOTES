# Lesson08--list

## 1. list的介绍及使用

### 1.1 list的介绍

1. list是可以在常数范围内在任意位置进行插入和删除的序列式容器,并且该容器可以前后双向迭代.
2. list的底层是双向链表结构,双向链表中每个元素存储在互不相关的独立节点中,在节点中通过指针指向其前一个元素和后一个元素.
3. list与forward_list非常相似：最主要的不同在于forward_list是单链表,只能朝前迭代,以让其更简单高效.
4. 与其它的序列容器相比(array,vector,deque),list通常在任意位置进行插入、移除元素的执行效率更好.
5. 与其它序列容器相比,list和forward_list最大的缺陷是不支持任意位置的随机访问,比如：要访问list的第6个元素,必须从已知的位置(比如头部或者尾部)迭代到该位置.
6. list还需要一些额外的空间,以保存每个节点的相关联信息.

> **为什么会有list？**
> 
> list是为了补充vector的缺点存在的.
>
> vector优点：
> 支持下标的随机访问.间接地就很好地支持排序、二分查找、堆算法等.
> 
> vector缺点：
> 1. 头部和中部的插入效率低.O(N),因为需要挪动数据.
> 2. 插入数据空间不够需要扩容.增容需要开辟新空间、拷贝数据、释放旧空间,会付出很大的代价.
> 
> list出厂就是为了解决vector的缺陷.
> 
> list优点：
> 1. list头部、中间插入不再需要挪动数据,效率高,O(1).
> 2. list插入数据是新增节点,不需要增容.
> 
> list缺点：
> 不支持随机访问.
>
> 实际使用中vector和list是相辅相成的两个容器.

### 1.2 list的使用

#### 1.2.1 list的构造

|                       构造函数                       |              接口说明              |
| :--------------------------------------------------: | :--------------------------------: |
|                        list()                        |            构造空的list            |
| list(size_type n,const value_type& val=value_type()) |  构造的list中包含n个值为val的元素  |
|                 list(const list& x)                  |            拷贝构造函数            |
|     list(InputIterator first,InputIterator last)     | 用[first,last)区间中的元素构造list |

```C++{.line-numbers}
#include<list>
list<int> lt1;//构造int类型的空容器
list<int> lt2(3,2);//构造含有3个2的int类型容器
list<int> lt3(lt2);//拷贝构造lt2
string s("hello");
list<char> lt4(s.begin(),s.end());//利用迭代器构造
```

#### 1.2.2 list插入和删除数据

|  函数声明  |                接口说明                |
| :--------: | :------------------------------------: |
| push_front |    在list首元素前插入值为val的元素     |
| pop_front  |          删除list中第一个元素          |
| push_back  |      在list尾部插入值为val的元素       |
|  pop_back  |         删除list中最后一个元素         |
|   insert   | 在list position位置中插入值为val的元素 |
|   erase    |      删除list position位置的元素       |


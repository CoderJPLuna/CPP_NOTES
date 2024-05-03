# Lesson14--map和set

## 1. 关联式容器

如 `vector` , `list` , `deque` , `forward_list` 等,这些容器统称为序列式容器,因为其底层为线性序列的数据结构,里面存储的是元素本身.

**关联式容器** 也是用来存储数据的,与序列式容器不同,其里面存储的是 `<key,value>` 结构的键值对,在数据检索时比序列式容器效率更高.

## 2. 键值对

用来表示具有一一对应关系的一种结构,该结构中一般只包含两个成员变量 `key` 和 `value` , `key` 代表键值, `value` 表示与 `key` 对应的信息.

SGI-STL中关于键值对的定义:

```C++{.line-numbers}
template <class T1,class T2>
struct pair
{
typedef T1 first_type;
typedef T2 second_type;

T1 first;
T2 second;
pair():first(T1()),second(T2())
{}
pair(const T1& a,const T2& b):first(a),second(b)
{}
};
```

## 3. 树形结构的关联式容器

根据应用场景的不同,STL总共实现了两种不同结构的关联式容器: **树形结构** 与 **哈希结构** .  
树形结构的关联式容器主要有四种: `map` , `set` , `multimap` , `multiset` .这四种容器的共同点是:使用平衡搜索树(即红黑树)作为其底层结果,容器中的元素是一个有序的序列.

### 3.1 set

#### 3.1.1 set的介绍

1. `set` 是按照一定次序存储元素的容器.
2. 在 `set` 中,元素的 `value` 也标识它( `value` 就是 `key` ,类型为 `T` ),并且每个 `value` 必须是唯一的. `set` 中的元素不能在容器中修改(元素总是 `const` ),但是可以从容器中插入或删除它们.
3. 在内部, `set` 中的元素总是按照其内部比较对象(类型比较)所指示的特定严格弱排序准则进行排序.
4. `set` 容器通过 `key` 访问单个元素的速度通常比` `unordered_set` 容器慢,但它们允许根据顺序对子集进行直接迭代.
5. `set` 在底层是用二叉搜索树(红黑树)实现的.

注意:
1. 与 `map/multimap` 不同, `map/multimap` 中存储的是真正的键值对 `<key, value>` , `set` 中只放 `value` ,但在底层实际存放的是由 `<value, value>` 构成的键值对.
2. `set` 中插入元素时,只需要插入 `value` 即可,不需要构造键值对.
3. `set` 中的元素不可以重复(因此可以使用 `set` 进行去重).
4. 使用 `set` 的迭代器遍历 `set` 中的元素,可以得到有序序列.
5. `set` 中的元素默认按照小于来比较.
6. `set` 中查找某个元素,时间复杂度为: $log_2 N$ .
7. `set` 中的元素不允许修改.
8. `set` 中的底层使用二叉搜索树(红黑树)来实现.

#### 3.1.2 set的使用

1. `set` 的模板参数列表
   ```C++{.line-numbers}
   template <class T,class Compare = less<T>,class Alloc = allocator<T>> 
   class set;
   ```
   `T` : `set` 中存放元素的类型,实际在底层存储 `<value,value>` 的键值对.  
   `Compare` : `set` 中元素默认按照小于来比较.
   `Alloc` : `set` 中元素空间的管理方式,使用STL提供的空间配置器管理.
2. `set` 的构造
   |                                                   函数声明                                                    |                 功能介绍                 |
   | :-----------------------------------------------------------------------------------------------------------: | :--------------------------------------: |
   |                       `set(const Compare&comp=Compare(),const Allocator&=Allocator());`                       |              构造空的 `set`              |
   | `set(InputIterator first,InputIterator last,const Compare& comp = Compare(),const Allocator& = Allocator());` | 用 `(first,last)` 区间中的元素构造 `set` |
   |                                  `set(const set<Key,Compare,Allocator>& x);`                                  |            `set` 的`拷贝构造             |
3. `set` 的构造
   |                函数声明                 |                         功能介绍                         |
   | :-------------------------------------: | :------------------------------------------------------: |
   |           `iterator begin()`            |            返回 `set` 中起始元素位置的迭代器             |
   |            `iterator end()`             |        返回 `set` 中最后一个元素后面位置的迭代器         |
   |     `const_iterator cbegin()const`      |        返回 `set` 中起始元素位置的 `const` 迭代器        |
   |      `const_iterator cend()const`       |    返回 `set` 中最后一个元素后面位置的 `const` 迭代器    |
   |       `reverse_iterator rbegin()`       |        返回 `set` 中最后一个元素位置的反向迭代器         |
   |        `reverse_iterator rend()`        |  返回 `set` 中最第一个元素的前一个元素位置的反向迭代器   |
   | `const_reverse_iterator crbegin()const` |    返回 `set` 中最后一个元素位置的反向 `const` 迭代器    |
   |  `cosnt_reverse_iterator crend()const`  | 返回 `set` 中第一个元素的前一个位置的反向 `const` 迭代器 |
4. `set`的容量
   |       函数声明       |                      功能介绍                       |
   | :------------------: | :-------------------------------------------------: |
   | `bool empty()const`  | 检测 `set` 是否为空,空返回 `true` ,否则返回 `false` |
   | `size_t size()const` |              返回 `set` 中有效元素个数              |
5. `set` 修改操作
   |                     函数声明                     |                                                                                      功能介绍                                                                                      |
   | :----------------------------------------------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
   | `pair<iterator,bool>insert(const value_type& x)` | 在 `set` 中插入元素 `x` ,实际插入的是 `<x,x>` 构成的键值对,如果插入成功,返回 `<该元素在set中的位置,true>` ,如果插入失败,说明 `x` 在 `set` 中已经存在,返回 `<x在set中的位置,false>` |
   |         `void erase(iterator position)`          |                                                                       删除 `set` 中 `position` 位置上的元素                                                                        |
   |       `size_type erase(const key_type& x)`       |                                                                  删除 `set` 中值为 `x` 的元素,返回删除元素的个数                                                                   |
   |    `void erase(iterator first,iterator last)`    |                                                                     删除 `set` 中 `[first,last)` 区间中的元素                                                                      |
   |   `void swap(set<Key,Compare,Allocator>& st)`    |                                                                                交换 `set` 中的元素                                                                                 |
   |                  `void clear()`                  |                                                                               将 `set` 中的元素清空                                                                                |
   |     `iterator find(const key_type& x)const`      |                                                                         返回 `set` 中值为 `x` 的元素的位置                                                                         |
   |    `size_type count(const key_type& x)const`     |                                                                         返回 `set` 中值为 `x` 的元素的个数                                                                         |

```C++{.line-numbers}
#include<set>
void TestSet()
{
   int array[]={1,3,5,7,9,2,4,6,8,0,1,3,5,7,9,2,4,6,8,0};
   std::set<int>s(array,array + sizeof(array) / sizeof(int));
   std::cout << s.size() << std::endl;

   for(auto&e:s)
   {
      std::cout << e << " ";
   }
   std::cout << std::endl;

   for(auto it = s.rbegin();it != s.rend();++it)
   {
      std::cout << *it << " ";
   }
   std::cout << std::endl;

   std::cout << s.count(3) << std::endl;
}
```

### 3.2 map

#### 3.2.1 map的介绍

1. `map` 是关联容器,它按照特定的次序(按照 `key` 来比较)存储由键值 `key` 和值 `value` 组合而成的元素.
2. 在 `map` 中,键值 `key` 通常用于排序和唯一地标识元素,而值 `value` 中存储与此键值 `key` 关联的内容.键值 `key` 和值 `value` 的类型可能不同,并且在 `map` 的内部, `key` 与 `value` 通过成员类型 `value_type` 绑定在一起,为其取别名称为 `pair` : `typedef pair<const key, T> value_type;` .
3. 在内部, `map` 中的元素总是按照键值 `key` 进行比较排序的.
4. `map` 中通过键值访问单个元素的速度通常比 `unordered_map` 容器慢,但 `map` 允许根据顺序对元素进行直接迭代(即对 `map` 中的元素进行迭代时,可以得到一个有序的序列).
5. `map` 支持下标访问符,即在 `[]` 中放入 `key` ,就可以找到与 `key` 对应的 `value` .
6. `map` 通常被实现为平衡二叉搜索树(红黑树).

#### 3.2.2 map的使用

1. `map` 的模板参数说明
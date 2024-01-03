# Effective STL

## 1.慎重选择容器类型

需要在容器任意位置插⼊元素，就选择序列容器（vector string deque list ）

不关⼼容器中的元素是否是排序的，使用哈希容器

slist（单链表）等不属于C++标准的容器最好不要选择

当发⽣元素的插⼊和删除，避免移动原来容器的元素移动很重要，那么避免使⽤连续内存的容器

需要兼容c，就vector

基于节点的容器不会使迭代器、指针、引⽤变为⽆效



## 2.不要试图编写独立于容器类型的代码



## 3.确保容器中的对象拷贝正确和高效



## 4.调用empty而不是检查size()是否为0

前者是常数时间操作，后者可能消耗的是线性时间



## 5.区间成员函数优先于与之对应的单元素成员函数

区间创建、删除、赋值（assign）、插⼊可以⽤到区间成员函数



## 6.考虑到C++编译器将会把代码尽可能解释为函数声明

```c++
class Weight{...};
Weight w();
```

希望对w进行默认初始化的行为会被编译器认为是一个函数声明，这个函数不带任何一个参数，返回一个Weight

避免的方法就是使用括号进行初始化，或者使用匿名的对象。

```c++
Weight w{}; //使用括号初始化，确保解释为对象的定义
Weight(); // 创建一个匿名对象
```



## 7.容器中包含指针

如果在容器中包含了通过new操作创建的对象的指针，切记在容器对象调⽤析构函数之前将指针delete掉

最简单的⽅法⽤智能指针代替指针容器，这⾥的智能指针通常是被引⽤计数的指针



## 8.不要创建包含智能指针的容器

”直接创建包含智能指针的容器“和“使用智能指针管理容器的生命周期”本质上是两个概念



## 9.慎重选择删除元素的⽅法

1.要删除容器中有特定值的所有对象

​	vector，string或deque，可以使用erase嵌套remove函数的方式进行删除

```c++
string str;
str.erase(std::remove(str.begin(), str.end(), charToRemove), str.end());
```

​	list，则使用remove

​	如果是一个关联容器，则使用erase

2.要在循环内部进行操作时

​	如果容器是⼀个标准序列容器，则写⼀个循环来遍历容器中的元素，记住每次调⽤erase时，要⽤它的返回值更新迭代器

​	如果是关联容器，写⼀个循环来遍历容器中的元素，记住当把迭代器传给erase时，要对他进⾏后缀递增

3.要删除容器中满足判别式的所有对象时

​	如果容器是vector，string和deque，则使用erase嵌套remove_if

​	如果容器是list，则使用list::remove_if

​	如果是关联容器，则使⽤remove嵌套copyif和swap（把我们需要的值复制到新容器，然后交换容器），或者写⼀个循环 来编译容器中的元素，记住当把迭代器传给erase时，要对他进⾏后缀递增



## 10.当使用动态分配数组的时候，使用vector和string



## 11.使用reserve来避免不必要的重新分配

vector和string的扩容：分配⼀块⼤⼩为旧内存两倍的新内存，把容器中所有的元素复制到新内存中，析构旧内存的对象，释放旧内存。

四个“容量”相关函数（只有vector与string提供所有的这4个函数）：

​	1.size()：容器中的元素数量

​	2.capacity()：容器能够容纳的元素总数

​	3.resize(Container::size_type n)：强迫容器改变到包含n个元素的状态，当前元素数量小于n时，会默认构造新的元素添加到末尾；当大于n时，会析构相应数量的容器尾部元素。如果n比容器容量大，就会在添加元素之前重新分配内存

​	4.reserve(Container::size_type n)：强迫容器改变容量变为⾄少n，前提是不⽐当前的容量⼩，这会导致重新分配



## 12.string实现的多样性

1. string 的值可能会被引⽤计数 
2. string对象的⼤⼩可能是char*的⼤⼩的1~7倍 
3. 创建⼀个新的字符串可能会发⽣0，1，2次的动态分配内存 
4. string也可能共享其容量，⼤⼩信息 
5. string可能⽀持针对单个对象的分配⼦
6. 不同的实现对字符内存的最⼩分配单位有不同的策略



## 13.使用swap来进行删除

```c++
string s;
string().swap(s); // 将s与空容器交换实现s的删除
```

在swap的时候，不仅两个容器的元素被交换了，他们的迭代器，指针和引⽤依然有效（string除外），只是他们的元素已经在另⼀个容器⾥⾯



## 14.了解排序相关的选择

1.如果需要对vector string，deque，或者数组中的元素进⾏⼀次完全排序，那么可以使⽤sort和stable_sort

2.如果有⼀个vector，string，deque或者数组，并且只需要对最前⾯的n个元素进⾏排序，那就是可以使⽤partial_sort

3.如果有⼀个vector，string，deque或者数组，并且只需要找到第n个位置上的正确排序元素，那么nth_element就⾏

```c++
 std::vector<int> numbers = {5, 2, 8, 1, 9, 4, 6};
 // 找到第3个位置上的元素
 int n = 2;  // 注意：这里 n 是从 0 开始计数的，nth_element默认是升序排列，它保证它之前的都比他小，它之后的都比他大
 std::nth_element(numbers.begin(), numbers.begin() + n, numbers.end());
 // 打印结果，序号为2的元素为4
 std::cout << "The " << n + 1 << "th smallest element is: " << numbers[n] << std::endl;
```

4.如果需要将⼀个标准序列容器中的元素按照是否满⾜某个特定区间区分开，那么就选择partition、 stable_partition



## 15.remove与erase嵌套

std::remove不会直接删除元素，而是将需要删除的元素移到容器的末尾，然后返回一个指向新的逻辑结尾的迭代器。实际的删除操作需要结合其他函数，比如 erase。

```c++
v.erase(remove(v.begin(),v.end(),elemnet),v.end())
```



## 16.使用accumulate或for_each进行区间统计

```c++
std::accumulate(numbers.begin(), numbers.end(), 0)
std::for_each(numbers.begin(), numbers.end(), [](int num){ 。。。})
```


















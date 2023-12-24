# C++STL

## STL原理及其实现

STL提供的六大组件——容器、算法、迭代器、仿函数、适配器（配接器）、空间配置器

六大组件的相互关系：

​	1.容器通过空间配置器取得数据存储空间 

​	2.算法通过迭代器存储容器中的内容

​	3.仿函数可以协助算法完成不同的策略的变化 

​	4.适配器可以修饰仿函数

STL的分类：

​	1.序列式容器——array，list，vector，deque，stack，queue等（array的大小会在一开始就确定，与其他标准容器不同，它是一个固定大小的数组容器）

​	2.关系式容器——set，map，multiset，multimap，unordered_set，unordered_map等

### 1.容器

​	各种数据结构，如vector、list（双向链表）、deque、set、map等，⽤来存放数据，从实现⾓度来看，STL容器是⼀种class template

### 2.算法

​	常⽤的算法，如sort、find、copy、for_each。从实现的⾓度来看，STL算法是⼀种function tempalte

### 3.迭代器

​	容器与算法之间的胶合剂，共有五种类型，从实现⾓度来看，迭代器是⼀种将operator* , operator-> , operator++,operator–等指针相关操作予以重载的class template

​	所有STL容器都附带有⾃⼰专属的迭代器，只有容器的设计者才知道如何遍历⾃⼰的元素

​	原⽣指针(native pointer，即直接使用的C++指针)也是⼀种迭代器

### 4.仿函数

​	⾏为类似函数，可作为算法的某种策略。从实现⾓度来看，仿函数是⼀种重载了operator()的class 或者class template 

### 5.适配器

​	⽤来修饰容器或者仿函数或迭代器接口。

​	STL提供的queue 和 stack，虽然看似容器，但其实只能算是⼀种容器配接器，因为它们的底部完全借助deque，所有操作都由底层的deque供应

### 6.空间配置器

​	负责空间的配置与管理。从实现⾓度看，配置器是⼀个实现了动态空间配置、空间管理、空间释放的class tempalte

​	⼀般的分配器的std:alloctor都含有两个函数allocate与deallocte，这两个函数分别调⽤operator new()与delete()，这 两个函数的底层又分别是malloc()and free();但是每次malloc会带来格外开销（因为每次malloc⼀个元素都要带有附加信息）

### STL的优点

​	1.高可重用性——几乎所有的代码都是模版类和模版函数

​	2.高性能——红黑树与哈希表在关系式容器中的高效查询

​	3.高移植性——数据与操作分离，编写的STL模块可以直接移植



## vector

vector在堆中分配了一段连续的内存空间来存放元素

包含三个迭代器：

​	1.first：vector中对象的起始字节位置

​	2.last：当前最后一个元素的末尾字节位置

​	3.end：指向整个vector容器所占内存空间的末尾字节

### 扩容过程

需要新分配一块更大的内存，将原来的数据复制，并释放之前的内存，插入新增的元素。

size()方法返回当前已经存储的元素的个数，capacity()方法返回当前分配内存下可以保存的元素个数。

### 固定扩容（一般不用）

每次增加固定的容量，虽然利用率高，但很容易导致反复扩容的情况，时间复杂度高。

### 加倍扩容（STL扩容方法）

每次扩容的时候capacity翻倍，此时预留空间较多，时间复杂度低。

### 方法与重要成员

resize()——改变当前容器内含有元素的数量，而不是容器的数量

reverse()——改变当前容器的最大容量（capacity）

vector迭代器——由于vector维护的是⼀个线性区间，所以普通指针具备作为vector迭代器的所有条件，就不需要重载 operator+，operator*等方法

元素操作——pop_back，erase，clear，insert，push_back



## list

每个元素都是放在⼀块内存中，内存空间可以是不连续的，通过指针来进⾏数据的访问。常⽤来做随机插⼊和删除操作容器。

list属于双向链表，其结点与list本⾝是分开设计的。

list是⼀个环状的双向链表，同时它也满⾜STL对于“前闭后开”的原则，即在链表尾端可以加上空⽩节点

### （list）迭代器的设计

迭代器中的五种属性（属于迭代器的特征，iterator traits）：

​	1.value_type：迭代器所指对象的类型

​	2.difference_type：表示两个迭代器之间的距离的类型

​	3.reference：迭代器得到的引用类型，通常是value_type&

​	4.reference：迭代器支持的指针类型，通常是value_type*

​	5.iterator_catrgory：input_iterator，output_iterator，forward_iterator，bidrectional_iterator，random_access

std::iterator_traits可以在输入iterator的情况下调用这五种方法查看对应的属性，然后让泛型算法对他们进行一致的操作。

### vector和list的区别

1.vector底层实现是数组；list是双向链表

2.vector是顺序内存,⽀持随机访问，list不⾏

3.vector在中间节点进⾏插⼊删除会导致内存拷贝，list不会

4.vector⼀次性分配好内存，不够时才进⾏翻倍扩容；list每次插⼊新节点都会进⾏内存申请

5.vector随机访问性能好，插⼊删除性能差；list随机访问性能差，插⼊删除性能好



## deque（双端列表）

deque支持快速随机访问，由于deque需要处理内部跳转，因此速度上没有vector快。

deque是⼀个双端开口的连续线性空间，其内部为分段连续的空间组成，随时可以增加⼀段新的空间并链接。

```
由于deque的迭代器⽐vector要复杂，这影响了各个运算层⾯，所以除⾮必要尽量使⽤vector；为了提⾼效率，在对deque进⾏排序操作的时候，我们可以先把deque复制到vector中再进⾏排序最后在复制回deque
```

deque可以在头部或尾端加入新的空间，避免了vector的“重新配置，复制，释放”的轮回，维护连整体连续的假象，并提供随机访问的接口。不过这样也导致了其迭代器的复杂性。

![image-20231211124845313](C:\Users\Vica\AppData\Roaming\Typora\typora-user-images\image-20231211124845313.png)

deque采用一块map（指的是一段连续的内存空间，不是std::map）作为控制中心，其中的每个元素都是指针，指向另一片连续线性空间，称为缓冲区，这个区才是用来储存数据的。

deque除了维护⼀个map指针以外，还维护了start与finish迭代器分别指向第⼀缓冲区的第⼀个元素，和最后⼀个缓冲区的最后⼀个元素的下⼀个元素，同时它还必须记住当前map的⼤⼩。

deque的插入和删除操作相比vector效率要更高，但访问速度上相比vector要慢一些。



## stack/queue

栈与队列被称之为duque的配接器，其底层是以deque为底部架构。通过deque执⾏具体操作。

![image-20231211125639407](C:\Users\Vica\AppData\Roaming\Typora\typora-user-images\image-20231211125639407.png)

以queue为例，在以deque为底层实现时，它的push方法本质就是deque中的push_back，并且它不提供pop_back方法。



## heap/priority_queue

heap：建立在完全二叉树（仅允许最后一层不满，最后一层都是满的就是满二叉树）上，分为两种：大根堆（父节点的值一定大于子节点，最大值一定在根节点），小跟堆（父节点的值一定小于子节点，最小值一定在根节点）

priority_queue：也是配接器。其内的元素不是按照被推⼊的顺序排列，⽽是⾃动取元素的权值排列，确省情况下利⽤⼀个max-heap（大根堆）完成，后者是以vector表现的完全⼆叉树



## map/set

这两个容器的底层都是采用红黑树实现。

set：用来判断一个元素是不是在一个组里面

map：相当于字典，把一个值映射成另一个值

优势——查找一个数的时间为O(logn)，遍历时采用iterator，效果不错

缺点——每次插入值的时候都需要调整红黑树，存在一定的效率影响



## map/unordered_map

map：

​	1.底层基于红⿊树实现，因此map内部元素排列是有序的。  

​	2.有序性，这是map结构最⼤的优点，其元素的有序性在很多应⽤中都会简化很多的操作。map的查找、删除、增加等⼀系列操作时间复杂度稳定，都为O(logn)。

​	3.查找、删除、增加等操作平均时间复杂度较慢，与n相关。

unordered_map：

​	1.底层基于哈希表实现，因此其元素的排列顺序是杂乱⽆序的。

​	2.查找、删除、添加的速度快，时间复杂度为常数级O(1）。

​	3.unordered_map内部基于哈希表，以（key,value）对的形式存储，因此空间占⽤率⾼。

​	4.unordered_map的查找、删除、添加的时间复杂度不稳定，平均为O(1)，取决于哈希函数。极端情况下可能为 O(n)。

### 问题

​	1.为什么insert之后，map和set以前保存的iterator不会失效？

​	因为 map 和 set 存储的是结点，不需要内存拷⻉和内存移动。

​	2.为何map和set的插⼊删除效率⽐其他序列容器⾼？

​	map 和 set 底部使⽤红⿊树实现，插⼊和删除的时间复杂度是 O(logn)，⽽向 vector 这样的序列容器插⼊和删除的时间复杂度是 O(N)。














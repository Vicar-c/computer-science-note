# C++新特性

## 智能指针

### 1.shared_ptr

shared_ptr的实现机制是在拷贝构造时使⽤同⼀份引⽤计数

shared_ptr模版类包含内容：

​	1.一个模版指针T* ptr

​	2.一个引用计数：在底层实现中，这个引用计数器保存在某个内部类型里（这个类型中还包含了deleter，它控制了指针的释放策略，默认情况下就是普通的delete操作），而这个内部类型对象在shared_ptr第一次构造时以指针的形式保存在shared_ptr中。

​	3.重载operator*和operator->：使得能像指针⼀样使⽤shared_ptr

​	4.重载copy constructor（拷贝构造函数）：此时的引用计数加1

​	5.重载operator=（赋值运算符）：如果原来的shared_ptr已经有对象，则让其引⽤次数减⼀并判断引⽤是否为零(是否调⽤delete)，然后将新的对象引⽤次数加⼀

​	6.重载析构函数：使引⽤次数减⼀并判断引⽤是否为零; (是否调⽤delete)

shared_ptr使用时可能遇到的问题：

​	1.同时一个shared_ptr被多个线程”读“是安全的（引用计数管理）

​	2.同⼀个shared_ptr被多个线程“写”是不安全的（shared_ptr仅保证引用计数是原子的，不包括与引用计数相关对象的访问，对于写操作还是得用互斥量等进行额外处理）

​	3.shared_ptr失效的唯一情况在循环引用。循环引用指的是，一个引用通过一系列的引用链，最终引用回自身（常常出现在观察者模式中，观察者与目标类之保存着互相的shared_ptr）。因此它永远无法将内存释放(引用计数始终大于0)，导致内存泄漏。

​	4.shared_ptr虽然允许多个无关的shared_ptr通过赋值操作符实现共享管理，但不允许多个无关shared_ptr管理同一个裸指针。最好直接用make_shared接口以实现对象的管理。

​	5.如果希望一个由shared_ptr管理的类能够在方法内部得到this指针的shared_ptr，并且共享管理，则需要继承enable_shared_from_this。

### 2.weak_ptr

weak_ptr是shared_ptr的"观察者"，它与一个shared_ptr绑定，但却不参与引用计数的计算，有效避免了循环引用的问题。

同时，在需要的时候，可以随时调用lock方法变为一个shared_ptr。

在上面提到的观察者模式下，只需要将观察者下的目标对象指针改为weak_ptr即可（在逻辑上也符合，观察者不应该影响目标对象的资源释放）

### 3.unique_ptr

需要将其绑定在一个new返回的指针上，表示同一时刻只有一个unique_ptr指向给定对象，离开作用域时，如果指向对象，则会将所指对象销毁。

unique不支持拷贝和赋值，不过可以使用release和reset将指针所有权从一个unique_ptr转移到另一个unique_ptr。



## 类型推导

### 1.auto

auto可以让编译器在编译期就推导出变量的类型

​	1.auto的使⽤对象必须初始化，否则⽆法推导出类型

​	2.auto在⼀⾏定义多个变量时，各个变量的推导不能产⽣⼆义性，否则编译失败

​	3.auto不能⽤作函数参数

​	4.在类中auto不能⽤作⾮静态成员变量

​	5.auto不能定义数组，可以定义指针

​	6.auto⽆法推导出模板参数

​	7.在不声明为引⽤或指针时，auto会忽略等号右边的引⽤类型和cv限定；相反，在声明为引⽤或者指针时，auto会保留等号右边的引⽤和cv属性

### 2.decltype

decltype则⽤于推导表达式类型，这⾥只⽤于编译器分析表达式的类型，表达式实际不会进⾏运算 

decltype不会像auto⼀样忽略引⽤和cv属性，decltype会保留表达式的引⽤和cv属性

对于decltype(exp)：

​	1.exp是表达式，decltype(exp)和exp类型相同

​	2.exp是函数调⽤，decltype(exp)和函数返回值类型相同

​	3.其他情况，若exp为左值，decltype(exp)为exp类型的左值引用

```c++
template<typename T, typename U>
auto add(T a, U b) -> decltype(a+b){
    return a+b;
}
```



## 右值引用

左值：可以放等号左边，可以取地址并有名字

右值：不可以放等号左边，不能取地址，没有名字

```
++i,--i为左值;i++,i--为右值
```

将亡值：C++11新增的与右值引用相关的表达式

左值引用：

​	1.对左值进行引用的类型，是对象的一个别名，并不拥有所绑定的堆存，所以需要立即初始化。

​	2.对左值引用而言，等号右边的值必须可以取地址，否则会编译失败。

右值引用：

​	1.可以使⽤std::move函数强制把左值转换为右值。

移动语义：

​	1.将资源的所有权进行转移，被转移者会失去这块资源的所有权。

​	2.通过移动构造函数使⽤移动语义，也就是std::move；移动语义仅针对于那些实现了移动构造函数的类的对象，对于那种基本类型int、float等没有任何优化作⽤，还是会拷贝，因为它们实现没有对应的移动构造函数。

类提供的五种默认函数：

​	默认构造函数，析构函数，拷贝构造函数，拷贝赋值函数，移动构造函数

​	拷贝构造函数使用的场景：

​		1.对象作为构造函数的参数，以值传递的方式传递给函数

​		2.对象作为函数的返回值，以值的方式从函数返回

​		3.使用一个对象给另一个对象“初始化”

​	注意，拷贝构造函数和赋值构造函数的核心区别是，后者将对象的值赋给一个已经存在的实例，而前者是创建一个新的对象实例

```c++
Person p;
Person p1 = p; // p1并没有声明，这里是拷贝构造
Person p2;
p2 = p; // p2已经声明，这里是赋值
```

深拷贝/浅拷贝：

​	1.浅拷贝——a和b的指针指向了同⼀块内存，就是浅拷贝，只是数据的简单赋值（默认的拷贝构造，拷贝赋值）

​	2.深拷贝——再拷贝对象时，如果被拷贝对象内部还有指针引⽤指向其它资源，⾃⼰需要重新开辟⼀块新内存存储资源

​		深拷贝的使用场景：a.动态分配内存

​						b.自定义类的资源管理

​						c.容器类中的元素为指针类型

完美转发：

​	写⼀个接受任意实参的函数模板，并转发到其它函数，⽬标函数会收到与转发函数完全相同的实参，通过 std::forward()实现



## nullptr

nullptr是⽤来代替NULL，⼀般C++会把NULL、0视为同⼀种东西，这取决去编译器如何定义NULL，有的定义为 ((void*)0)，有的定义为0

C++不允许直接将void* 隐式转换到其他类型，在进⾏C++重载时会发⽣混乱，这也导致指针如果定义为NULL时出现需要隐式转换的场合编译器报错

C++11引⼊nullptr关键字来区分空指针和0。nullptr 的类型为 nullptr_t，能够转换为任何指针或成员指针的类型， 也可以进⾏相等或不等的⽐较



## 范围for循环

```c++
for(auto &i : arr){ // 如果是针对arr的修改，最好加上引号
    ...
}
```



## 列表初始化

C++采用花括号进行初始化，被称为列表初始化

在对于内置类型变量进行初始化时，如果使用列表初始化，且初始值存在信息丢失风险时，编译器会报错

```c++
long double d = 3.1415926536;
int a = {d}; // 信息有丢失风险，转换未执行
```



## lambda表达式

lambda表达式表⽰⼀个可调⽤的代码单元，没有命名的内联函数，不需要函数名因为我们直接（⼀次性的）⽤它， 不需要其他地⽅调⽤

[捕获列表] (参数列表) -> 返回类型 {函数体 }

其中，只有捕获列表和函数体是必要的。大多是情况下，lambda表达式的返回值可由编译器猜测得出

lambda表达式的捕获列表不同内容时的含义：

​	1.[]：不捕获任何变量,这种情况下lambda表达式内部不能访问外部的变量

​	2.[&]：以引⽤⽅式捕获所有变量（保证lambda执⾏时变量存在）

​	3.[=]：⽤值的⽅式捕获所有变量（创建时拷贝，修改对lambda内对象⽆影响)

​	4.[=,&foo]：以引⽤捕获变量foo, 但其余变量都靠值捕获

​	5.[bar]：以值⽅式捕获bar; 不捕获其它变量

​	6.[this]：捕获所在类的this指针



并发：

### 1.std::thread

​	std::thread禁止拷贝构造函数

​	默认构造函数会创建一个空的thread执行对象

​	初始化构造函数，会创建一个thread对象，该对象可以被joinable，并绑定一个函数执行

​	std::thread还存在一个detach的方法，通过这个方法可以让线程对象和线程函数脱离关系



### 2.lock_guard

​	创建即加锁，作⽤域结束⾃动析构并解锁，⽆需⼿⼯解锁。不能中途解锁，必须等作⽤域结束才解锁。不能复制。



### 3.unique_lock

​	uniquelock 是 lockguard 的升级加强版，它具有 lock_guard 的所有功能，同时又具有其他很多⽅法， 使⽤起来更强灵活⽅便，能够应对更复杂的锁定需要。

特点：

​	1.创建时可以不锁定（通过指定第⼆个参数为std::defer_lock），⽽在需要时再锁定 

​	2.可以随时加锁解锁 

​	3.作⽤域规则同 lock_grard，析构时⾃动释放锁

​	4.不可复制，可移动 

​	5.条件变量需要该类型的锁作为参数（此时必须使⽤unique_lock）














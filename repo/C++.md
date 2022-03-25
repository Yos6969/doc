# 类

## 封装与继承

### 几种继承方式

![1](.\img\屏幕截图 2021-11-17 144148.png)

`下面的表总结了公有、私有和保护继承的不同`

| 变化      | 公有继承        | 保护继承        | 私有继承        |
| ------- | ----------- | ----------- | ----------- |
| 基类的公有成员 | 派生类的公有成员    | 派生类的保护成员    | 派生类私有成员     |
| 基类的保护成员 | 派生类的保护成员    | 派生类的保护成员    | 派生类私有成员     |
| 基类的私有成员 | 只能通过基类的接口访问 | 只能通过基类的接口访问 | 只能通过基类的接口访问 |

## 类的构造函数

### 空类的默认函数



![2](.\img\未命名图片.png)

###构造或析构函数中可以调用虚函数吗？

```c++
class A{
public:
    A(){
        print();
    }
      ~A(){

        //print();
    }
    virtual void print(){
        cout<<"A->print()"<<endl;
    }


};

class B: public A{
public:
    B(){
        print();
    }
    ~B(){
        //print();
    }
     void print(){
        cout<<"B->print()"<<endl;
    }
};

int main(){
    cout<<"father pointer point to child obj"<<endl;
    A* ba=new B();
    cout<<"child pointer point to child obj"<<endl;
    B* bb=new B();

}
//在构造函数调用print，按照构造顺序调用print版本
//father pointer point to child obj
//A->print()
//B->print()
//child pointer point to child obj
//A->print()
//B->print()

//在析构函数调用print，按照析构顺序调用print版本
//father pointer point to child obj
//B->print()
//A->print()
//child pointer point to child obj
//B->print()
//A->print()
```



# 函数

## 虚函数

![3](.\img\未命名图3片.png)

![4](.\img\4.png)

virtual =0 关键字说明这是一个纯虚函数，需要在子类去实现它，但纯虚函数本身是可以有定义的。在父类中声明为加了virtual关键字的函数，在子类中重写时候不需要加virtual也是虚函数

每个对象的开头位置都有一个虚指针vptr，这个类的所有对象中的vptr指向共同的vtbl，vtbl专门存放有virtual关键字的函数

![5](.\img\5.png)



# 类型

## static关键字和extern关键字

```
static
```



- static修饰全局变量时，在静态存储器（bss、data段）开辟存储空间，其实全局变量本来就是被隐式static修饰的，被修饰后，这个变量只在本文件中使用（include进入文件也能用），如果要在其他源文件使用，需要加extern关键字

- static修饰局部变量（一般是函数中的变量，函数中的变量一般在栈中，修饰后放入data段内），约等于全局变量，在整个进程周期中，只定义和初始化一次，

- 关于初始化：可分为1.编译时初始化  2.加载时初始化  3.运行时初始化
  - 编译时初始化--静态变量是基本数据类型，且是个常值，是这样的
  - 加载时初始化--1.静态变量是一个基本数据类型，但是初始值非常量  2.静态变量是一个类对象
  - 运行时初始化--初始化发生在变量第一次被引用，对于局部静态对象是这样的 

- static修饰函数，修饰函数时，表示该函数的作用域时源文件，其他文件无法调用该函数，一般用作某文件的内部函数使用

- static修饰类成员变量：这个变量在所有这个类的对象间共享

- static修饰类成员函数:也叫静态成员函数
    　　> 它的形参列表之中没有隐含的this指针
  　　　　> 　　> 不能调用非静态的数据成员
    　　> 　　> 　　> 不能调用非静态的成员函数
    　　> 　　> 　　> 　　> 　 可以直接通过类名调用

  

![6](.\img\6.png)

```
extern
```



- extern声明变量：表明该变量已经在其他源文件中被定义
- extern修饰函数：表明该函数在其他源文件里已经被定义

## 类型转换

c++提供四种类型转换的关键字    static_cast   dynamic_cast   const_cast   reinterpret_cast

![8](.\img\8.png)

![9](.\img\9.png)

![10](.\img\10.png)

![11](.\img\11.png)

### 隐式转换

隐式类型转换是指c++自动将一种类型转换成另一种类型，是编译器的一种自主行为。

为什么c++需要隐式类型转换？

c++多态的特性，就是通过父类的对象实现对子类的封装，以父类的类型返回之类对象。

c++中使用父类的地方一定可以使用子类代替，这也得益于隐式类型转换。

c++是一种强类型的语言，有着非常严格的类型检查，采用隐式类型转换会使程序员更方便快捷一点。

```c++
class complex{
public:
    complex(int real,int image):imag(image),rea(real){}
    operator double(){
        return rea;
    }
private:
    int rea,imag;
};
int main(){
    complex *cp =new complex(100,101);
    cout<<cp->operator double()<<endl;// 显示转为 double 100
    cout<<*cp<<endl;//隐式转为 double 100
}
```

#Lambda

先来看一下lambda表达式的语法形式：

```
[ capture ] ( params ) opt -> ret { body; };
```

其中carpture是捕获列表，params是参数，opt是选项，ret则是返回值的类型，body则是函数的具体实现。

1.捕获列表描述了lambda表达式可以访问上下文中的哪些变量。
	[] :表示不捕获任何变量
	[=]：表示按值捕获变量
	[&]：表示按引用捕获变量
	[this]：值传递捕获当前的this
	但是捕获列表不允许变量的重复传递：例如

```
[=,x]
```

上面这种捕获是不允许的，=表示按值的方式捕获所有的变量，x相当于被重复捕获了。

2.params表示lambda的参数，用在{}中。
3.opt表示lambda的选项，例如mutable，后面会介绍一下mutable的用法。
4.ret表示lambda的返回值，也可以显示指明返回值，lambda会自动推断返回值，但是值得注意的是只有当lambda的表达式仅有一条return语句时，自动推断才是有效的。像下面这种的表达式就需要加上返回类型。

```c++
[](double x )->double{int y = x ;return x - y;};
```

虽然lambda表达式是匿名函数，但是实际上也可以给lambda表达式指定一个名称，如下表示：

```c++
auto f = [](int x ){return x % 3 ==0;};
```

# 智能指针

智能指针(smart pointer)是存储指向动态分配（堆）对象指针的类，用于生存期控制，能够确保自动正确的销毁动态分配的对象，防止内存泄露（利用自动调用类的析构函数来释放内存）。它的一种通用实现技术是使用引用计数（除此之外还有资源独占(unique_ptr),只引用不计数（weak_ptr））。智能指针类将一个计数器与类指向的对象相关联，引用计数跟踪该类有多少个对象共享同一指针。每次创建类的新对象时，初始化指针并将引用计数置为1；当对象作为另一对象的副本而创建时，拷贝构造函数拷贝指针并增加与之相应的引用计数；对一个对象进行赋值时，赋值操作符减少左操作数所指对象的引用计数（如果引用计数为减至0，则删除对象），并增加右操作数所指对象的引用计数；调用析构函数时，构造函数减少引用计数（如果引用计数减至0，则删除基础对象）。

## shared_ptr

- *智能指针将一个计数器与类指向的对象相关联，引用计数跟踪共有多少个类对象共享同一指针*
- 每次创建类的新对象时，初始化指针并将引用计数置为1
- *当对象作为另一对象的副本而创建时，拷贝构造函数拷贝指针并增加与之相应的引用计数*
- 对一个对象进行赋值时，赋值操作符减少左操作数所指对象的引用计数（如果引用计数为减至0，则删除对象），并增加右操作数所指对象的引用计数；这是因为左侧的指针指向了右侧指针所指向的对象，因此右指针所指向的对象的引用计数加1
- 调用析构函数时，减少引用计数，如果引用计数减至0，则删除基础对象和引用计数对象

```c++
//
// Created by 18181 on 2022/3/24.
//

#ifndef FORK_SHARED_PTR_H
#define FORK_SHARED_PTR_H
#include "iostream"

template<typename T>
class shared_ptr {
public:
    shared_ptr(T *_t) : p(_t), count_(new int(1)){}
    shared_ptr() : p(nullptr), count_(nullptr){}

    shared_ptr(shared_ptr<T>& r) {
        p = r.get();
        count_ = r.getcount();
        if (count_) {
            (*count_)++;
        }
    }

    shared_ptr& operator=(const shared_ptr<T>& rhs) {
        if(this == &rhs) {
            return *this;
        }

        reset();
        p = rhs.get();
        count_ = rhs.getcount();

        if(count_) {
            (*count_)++;
        }
        return *this;
    }

    T* get() const {
        return p;
    }

    int* getcount() const{
        return count_;
    }
    int count(){
        return count_?*count_:0;
    }

    T* operator->() {
        return p;
    }

    T& operator*() {
        return *p;
    }

    ~shared_ptr(){
        reset();
    }

    void reset(){
        if(count_){
            (*count_)--;
            if(*count_ == 0){
                delete p;
                delete count_;
            }
        }
    }

private:
    T *p;
    int * count_;
};

class A{
public:
	int s=1;
    friend ostream& operator<<(std::ostream& os, const A&a){
    os<<a.s;
    return os;
}
};

int main(){
    shared_ptr<A> emp;
    std::cout<< " empty shared_pointer emp count: " <<emp.count()<<std::endl;
    shared_ptr<A> sp(new A());
    std::cout<< " shared_pointer sp count: " <<sp.count()<<std::endl;
    shared_ptr<A> sp2(sp);//sp2 share with sp
    std::cout<< " shared_pointer sp count: " <<sp.count()<<" sp2 count: "<<sp2.count()<<std::endl;
    shared_ptr<A> sp1(new A());
    std::cout<< "after copy assignment sp1 count: "<< sp1.count() <<std::endl;
    sp = sp1;
    std::cout<< "after == assignment sp count: "<< sp.count() <<" sp1: " <<sp1.count()<<" sp2 count: "<<sp2.count()<<std::endl;

    shared_ptr<A>a(new A());
    std::cout<<*a<<std::endl;
    std::cout<<a->s<<std::endl;

}

#endif //FORK_SHARED_PTR_H
//输出
//shared_ptr<A>a(new A());
//std::cout<<*a<<std::endl;
//std::cout<<a->s<<std::endl;
```



## weak_ptr

- weak_ptr是一种持有被shared_ptr管理者的资源的弱引用的智能指针。它必须通过转化为shared_ptr来访问管理的资源。
- weak_ptr被用来跟踪资源，它通过转化为shared_ptr来获取临时所有权。如果这个时候原先拥有资源的shared_ptr销毁了，资源的生命周期将会被延长至这个转化得到的shared_ptr析构之前。
- weak_ptr另外一个作用是打破shared_ptr可能的循环引用（循环引用会导致应该释放的资源没有释放，此处不展开）。

## unique_ptr

和 shared_ptr 指针最大的不同之处在于，unique_ptr 指针指向的堆内存无法同其它 unique_ptr 共享，也就是说，每个 unique_ptr 指针都独自拥有对其所指堆内存空间的所有权。

- *每个 unique_ptr 指针指向的堆内存空间的引用计数，都只能为 1*
- 一旦该 unique_ptr 指针放弃对所指堆内存空间的所有权，则该空间会被立即释放回收。

```c++
//
// Created by 18181 on 2022/3/25.
//

#ifndef FORK_UNIQUE_PTR_H
#define FORK_UNIQUE_PTR_H
#include "iostream"
template<typename T>
class unique_ptr {
public:
    unique_ptr():p_(nullptr), count_(nullptr){}
    unique_ptr(T *_t):p_(_t),count_(new int(1)){}

    unique_ptr(const unique_ptr& p) = delete;
    unique_ptr<T>& operator=(const unique_ptr& p) = delete;
    unique_ptr<T>& operator=(T* p) = delete;

    T* get()
    {
        return p_;
    }


    int* getcount() const{
        return count_;
    }
    int count(){
        return count_?*count_:0;
    }

    T* operator->() {
        return p_;
    }

    T& operator*() {
        return *p_;
    }


    void reset(){
        if(count_){
            delete count_;
            delete p_;
        }
    }


private:
    T * p_;
    int *count_;
};

class A{
public:
    int s=1;


};
std::ostream& operator<<(std::ostream& os, const A&a){
    os<<a.s;
    return os;
}
int main(){
    unique_ptr<A>up;
    std::cout<< "empty unique_pointer emp count: " <<up.count()<<std::endl;
    unique_ptr<A>a(new A());
    std::cout<< "unique_pointer  count: " <<a.count()<<std::endl;
    std::cout<<*a<<std::endl;
    std::cout<<a->s<<std::endl;
}

#endif //FORK_UNIQUE_PTR_H
//输出;
//empty unique_pointer emp count: 0
//unique_pointer  count: 1
//1
//1
```



# std::thread()

- 默认构造函数，创建一个空的 `std::thread` 执行对象。
- 初始化构造函数，创建一个 `std::thread` 对象，该 `std::thread` 对象可被 `joinable`，新产生的线程会调用 `fn` 函数，该函数的参数由 `args` 给出。
- 拷贝构造函数(被禁用)，意味着 `std::thread` 对象不可拷贝构造。
- Move 构造函数，move 构造函数(move 语义是 C++11 新出现的概念，详见附录)，调用成功之后 `x` 不代表任何 `std::thread` 执行对象

成员函数

**join**：主线程等待子线程的终止。也就是说主线程的代码块中，如果碰到了t.join()方法，此时主线程需要等待（阻塞），等待子线程结束了(Waits for this thread to die.),才能继续执行t.join()之后的代码块。

**get_id**: 获取线程 ID，返回一个类型为 std::thread::id 的对象。

**joinable**: 检查线程是否可被 join。检查当前的线程对象是否表示了一个活动的执行线程，由默认构造函数创建的线程是不能被 join 的。另外，如果某个线程 已经执行完任务，但是没有被 join 的话，该线程依然会被认为是一个活动的执行线程，因此也是可以被 join 的。

**detach**: Detach 线程。 将当前线程对象所代表的执行实例与该线程对象分离，使得线程的执行可以单独进行。一旦线程执行完毕，它所分配的资源将会被释放。

调用 detach 函数之后：

- `*this` 不再代表任何的线程执行实例。
- joinable() == false
- get_id() == std::thread::id()

另外，如果出错或者 joinable() == false，则会抛出 std::system_error。

- ```c++
  #include <iostream>
  #include <utility>
  #include <thread>
  #include <chrono>
  #include <functional>
  #include <atomic>
  
  void f1(int n)
  {
      for (int i = 0; i < 5; ++i) {
          std::cout << "Thread " << n << " executing\n";
          std::this_thread::sleep_for(std::chrono::milliseconds(10));
      }
  }
  
  void f2(int& n)///这些参数会拷贝至新线程的内存空间中(同临时变量一样)。即使函数中的参数是引用的形式，拷贝操作也会执行。
  {
      for (int i = 0; i < 5; ++i) {
          std::cout << "Thread 2 executing\n";
          ++n;
          std::this_thread::sleep_for(std::chrono::milliseconds(10));
      }
  }
  
  int main()
  {
      int n = 0;
      std::thread t1; // t1 is not a thread
      std::thread t2(f1, n + 1); // pass by value
      std::thread t3(f2, std::ref(n)); // pass by reference 见函数声明注释
      std::thread t4(std::move(t3)); // t4 is now running f2(). t3 is no longer a thread
      t2.join();
      t4.join();
      std::cout << "Final value of n is " << n << '\n';
  }
  ```

## __thread关键字

  __thread是GCC内置的线程局部存储设施，存取效率可以和全局变量相比。__thread变量每一个线程有一份独立实体，各个线程的值互不干扰。可以用来修饰那些带有全局性且值可能变，但是又不值得用全局变量保护的变量。

       __thread使用规则：只能修饰POD类型(类似整型指针的标量，不带自定义的构造、拷贝、赋值、析构的类型，二进制内容可以任意复制memset,memcpy,且内容可以复原)，不能修饰class类型，因为无法自动调用构造函数和析构函数，可以用于修饰全局变量，函数内的静态变量，不能修饰函数的局部变量或者class的普通成员变量，且__thread变量值只能初始化为编译器常量(值在编译器就可以确定const int i=5,运行期常量是运行初始化后不再改变const int i=rand()).


## 互斥量和临界区

我们在操作系统、亦或是数据库的相关知识中已经了解过了有关并发技术的基本知识，mutex 就是其中的核心之一。 C++11 引入了 mutex 相关的类，其所有相关的函数都放在 <mutex> 头文件中。

std::mutex 是 C++11 中最基本的 mutex 类，通过实例化 std::mutex 可以创建互斥量， 而通过其成员函数 lock() 可以进行上锁，unlock() 可以进行解锁。 但是在实际编写代码的过程中，最好不去直接调用成员函数， 因为调用成员函数就需要在每个临界区的出口处调用 unlock()，当然，还包括异常。 这时候 C++11 还为互斥量提供了一个 RAII 语法的模板类 std::lock_guard。 RAII 在不失代码简洁性的同时，很好的保证了代码的异常安全性。

在 RAII 用法下，对于临界区的互斥量的创建只需要在作用域的开始部分，例如：


```c++
#include <thread>
#include<iostream>
int v = 1;

void critical_section(int change_v) {

static std::mutex mtx;
std::lock_guard<std::mutex> lock(mtx);

// 执行竞争操作
v = change_v;

// 离开此作用域后 mtx 会被释放
int main(){
  std::thread t1(critical_section, 2), t2(critical_section, 3);
t1.join();
t2.join();

std::cout << v << std::endl;
return 0;
}
```
由于 C++ 保证了所有栈对象在生命周期结束时会被销毁，所以这样的代码也是异常安全的。 无论 critical_section() 正常返回、还是在中途抛出异常，都会引发堆栈回退，也就自动调用了 unlock()。

而 std::unique_lock 则相对于 std::lock_guard 出现的，std::unique_lock 更加灵活， std::unique_lock 的对象会以独占所有权（没有其他的 unique_lock 对象同时拥有某个 mutex 对象的所有权） 的方式管理 mutex 对象上的上锁和解锁的操作。所以在并发编程中，推荐使用 std::unique_lock。

std::lock_guard 不能显式的调用 lock 和 unlock， 而 std::unique_lock 可以在声明后的任意位置调用， 可以缩小锁的作用范围，提供更高的并发度。

如果你用到了条件变量 std::condition_variable::wait 则必须使用 std::unique_lock 作为参数。

例如：


```c++
#include <iostream>

#include <thread>

int v = 1;

void critical_section(int change_v) {

static std::mutex mtx;
std::unique_lock<std::mutex> lock(mtx);
// 执行竞争操作
v = change_v;
std::cout << v << std::endl;
// 将锁进行释放
lock.unlock();

// 在此期间，任何人都可以抢夺 v 的持有权

// 开始另一组竞争操作，再次加锁
lock.lock();
v += 1;
std::cout << v << std::endl;
  
  int main(){
    std::thread t1(critical_section, 2), t2(critical_section, 3);
    t1.join();
    t2.join();
    return 0;
  }
```
# std::future

期物（Future）表现为 `std::future`，它提供了一个访问异步操作结果的途径，这句话很不好理解。 为了理解这个特性，我们需要先理解一下在 C++11 之前的多线程行为。

试想，如果我们的主线程 A 希望新开辟一个线程 B 去执行某个我们预期的任务，并返回我一个结果。 而这时候，线程 A 可能正在忙其他的事情，无暇顾及 B 的结果， 所以我们会很自然的希望能够在某个特定的时间获得线程 B 的结果。

在 C++11 的 `std::future` 被引入之前，通常的做法是： 创建一个线程 A，在线程 A 里启动任务 B，当准备完毕后发送一个事件，并将结果保存在全局变量中。 而主函数线程 A 里正在做其他的事情，当需要结果的时候，调用一个线程等待函数来获得执行的结果。

而 C++11 提供的 `std::future` 简化了这个流程，可以用来获取异步任务的结果。 自然地，我们很容易能够想象到把它作为一种简单的线程同步手段，即屏障（barrier）。

介绍std::prioise 和 std::pakaged_task:

promise 对象可以保存某一类型 T 的值，该值可被 future 对象读取（可能在另外一个线程中），因此 promise 也提供了一种线程同步的手段。在 promise 对象构造时可以和一个共享状态（通常是std::future）相关联，并可以在相关联的共享状态(std::future)上保存一个类型为 T 的值。

可以通过 get_future 来获取与该 promise 对象相关联的 future 对象，调用该函数之后，两个对象共享相同的共享状态(shared state)

- promise 对象是异步 Provider，它可以在某一时刻设置共享状态的值。
- future 对象可以异步返回共享状态的值，或者在必要的情况下阻塞调用者并等待共享状态标志变为 ready，然后才能获取共享状态的值。

```c++
#include <iostream>       // std::cout
#include <functional>     // std::ref
#include <thread>         // std::thread
#include <future>         // std::promise, std::future

void print_int(std::future<int>& fut) {
    int x = fut.get(); // 获取共享状态的值.
    std::cout << "value: " << x << '\n'; // 打印 value: 10.
}

int main ()
{
    std::promise<int> prom; // 生成一个 std::promise<int> 对象.
    std::future<int> fut = prom.get_future(); // 和 future 关联.
    std::thread t(print_int, std::ref(fut)); // 将 future 交给另外一个线程t.
    prom.set_value(10); // 设置共享状态的值, 此处和线程t保持同步.
    t.join();
    return 0;
}
```

std::packaged_task是std::promise的简化形式

```c++
    std::packaged_task<int()> task([]{ return 7; }); // wrap the function
    std::future<int> f1 = task.get_future();  // get a future
    std::thread t(std::move(task)); // launch on a thread
	t.join();
	std::cout<<f1.get()<<std::endl;
```



# 模板

## 可变参数模板

对于可变类模板，基本示例如下：

```c++
template<typename... Arguments>
class classname;
```

由上式可知，其特殊性在于 `...` 的使用，可变参数模板，通过使用 `...` 来帮助定义，其中，`...` 左侧为参数包（`parameter pack` ），右侧将参数包展开成多个单独的参数。

上面的类 `classname` 可以接收任意数量的参数来进行实例化，例如：

```c++
classname<> c1();
classname<float, int> c2();
classname<float, std::string, std::vector<int>> c3();
```

当然，还可以指定必须填充固定数量的参数，例如：

```c++
template<typename first, typename... Arguments>
class classname2;

// classname2<> c4(); 这是错误的用法！参数必须大于等于 1
classname2<float> c4();
```

```c++
#include <iostream>
using namespace std;
//递归终止函数
void print()
{
   cout << "empty" << endl;
}
//展开函数
template <class T, class ...Args>
void print(T head, Args... rest)
{
   cout << "parameter " << head << endl;
   print(rest...);
}


int main(void)
{
   print(1,2,3,4);
   return 0;
}
```

递归调用的过程是这样的:

```
print(1,2,3,4);
print(2,3,4);
print(3,4);
print(4);
print();
```

上面的递归终止函数还可以写成这样：

```
template <class T>
void print(T t)
{
   cout << t << endl;
}
```

修改递归终止函数后，上例中的调用过程是这样的：

```
print(1,2,3,4);
print(2,3,4);
print(3,4);
print(4);
```

# 左值和右值

1) 可位于赋值号（=）左侧的表达式就是左值；反之，只能位于赋值号右侧的表达式就是右值

2) 有名称的、可以获取到存储地址的表达式即为左值；反之则是右值。

```c++
int && a = 10;
a = 100;
cout << a << endl;//100
```



## 右值引用

一般来说，右值无法使用&取地址，c++11提供了&&来取得右值的引用(只能绑定右值)

看下面的函数，

```c++
class Copyable {
public:
    Copyable(){}
    Copyable(const Copyable &o) {
        cout << "Copied" << endl;
    }
};
Copyable ReturnRvalue() {
    return Copyable(); //返回一个临时对象
}
void AcceptVal(Copyable a) {

}
void AcceptRef(const Copyable& a) {

}

int main() {
    cout << "pass by value: " << endl;
    AcceptVal(ReturnRvalue()); // 应该调用两次拷贝构造函数
    cout << "pass by reference: " << endl;
    AcceptRef(ReturnRvalue()); //应该只调用一次拷贝构造函数
}
```



1. 左值引用， 使用 `T&`, 只能绑定**左值**-----------函数参数中，传入一个例如  void fuc(int & a)
2. 右值引用， 使用 `T&&`， 只能绑定**右值**
3. 常量左值， 使用 `const T&`, 既可以绑定**左值**又可以绑定**右值**<上面的代码示例了这一点>-----------函数参数中，传入一个例如  void fuc(const int & a)
4. 已命名的**右值引用**，编译器会认为是个**左值**
5. 编译器有返回值优化，但不要过于依赖
6. 注意 ，不是所有&&都是右值引用，有时也有“通用引用(universal reference)”的含义，出现在类型推导的场景，详见下面的完美转发

## 移动语义

实现移动语义就必须增加两个函数：移动构造函数和移动赋值构造函数

```c++
class myclass{
	public：
	~~~
	~~~
     //移动构造函数
MyString(MyString&& str) noexcept
       :m_data(str.m_data) {
       MCtor ++;
       str.m_data = nullptr; //不再指向之前的资源了
   }
      //移动赋值函数
MyString& operator=(MyString&& str) noexcept{
       MAsgn ++;
       if (this == &str) // 避免自我赋值!!
          return *this;

       delete[] m_data;
       m_data = str.m_data;
       str.m_data = nullptr; //不再指向之前的资源了
       return *this;
   }
   private:
   char* m_data;
   
   }
```

移动构造函数与拷贝构造函数的区别是，拷贝构造的参数是`const MyString& str`，是*常量左值引用*，而移动构造的参数是`MyString&& str`，是*右值引用*，而`MyString("hello")`是个临时对象，是个右值，优先进入**移动构造函数**而不是拷贝构造函数。而移动构造函数与拷贝构造不同，它并不是重新分配一块新的空间，将要拷贝的对象复制过来，而是"偷"了过来，将自己的指针指向别人的资源，然后将别人的指针修改为`nullptr`，这一步很重要，如果不将别人的指针修改为空，那么临时对象析构的时候就会释放掉这个资源，"偷"也白偷了。

### move

对于一个左值，编译器肯定是调用拷贝构造函数，能不能也移动构造而不是拷贝构造呢？`C++11`为了解决这个问题，提供了`std::move()`方法来将左值转换为右值，从而方便应用移动语义

```c++
for(int i=0;i<1000;i++){
        MyString tmp("hello");
        vecStr2.push_back(std::move(tmp)); //调用的是移动构造函数，需要MyString中实现了移动构造函数
    }
```

1. 如果我们*没有提供移动构造函数，只提供了拷贝构造函数*，`std::move()`会失效但是不会发生错误，因为编译器找不到移动构造函数就去寻找拷贝构造函数，也这是拷贝构造函数的参数是`const T&`常量左值引用的原因！

## 完美转发

通过一个函数将参数继续转交给另一个函数进行处理，原参数可能是右值，可能是左值，如果还能继续保持参数的原有特征，那么它就是完美的

下面是一个非完美转发的例子

```c++
void process(int& i){
    cout << "process(int&):" << i << endl;
}
void process(int&& i){
    cout << "process(int&&):" << i << endl;
}

void myforward(int&& i){
    cout << "myforward(int&&):" << i << endl;
    process(i);
}

int main()
{
    int a = 0;
    process(a); //a被视为左值 process(int&):0
    process(1); //1被视为右值 process(int&&):1
    process(move(a)); //强制将a由左值改为右值 process(int&&):0
    myforward(2);  //右值经过forward函数转交给process函数，却称为了一个左值，
    //原因是该右值有了名字  所以是 process(int&):2
    myforward(move(a));  // 同上，在转发的时候右值变成了左值  process(int&):0
    // forward(a) // 错误用法，右值引用不接受左值
}
```

而c++中提供了一个`std::forward()`模板函数解决这个问题。将上面的`myforward()`函数简单改写一下：

```c++
void myforward(int&& i){
    cout << "myforward(int&&):" << i << endl;
    process(std::forward<int>(i));
}

myforward(2); // process(int&&):2
```

上面修改过后还是不完美转发，`myforward()`函数能够将右值转发过去，但是并不能够转发左值，解决办法就是借助`universal references`通用引用类型和`std::forward()`模板函数共同实现完美转发。例子如下：

```c++
void RunCode(int &&m) {
    cout << "rvalue ref" << endl;
}
void RunCode(int &m) {
    cout << "lvalue ref" << endl;
}
void RunCode(const int &&m) {
    cout << "const rvalue ref" << endl;
}
void RunCode(const int &m) {
    cout << "const lvalue ref" << endl;
}

// 这里利用了universal references，如果写T&,就不支持传入右值，而写T&&，既能支持左值，又能支持右值
//int b=1;这是模板的特性 
//auto && a=b;  //int a&&=b会报错
template<typename T>
void perfectForward(T && t) {
    RunCode(forward<T> (t));
}

template<typename T>
void notPerfectForward(T && t) {
    RunCode(t);
}

int main()
{
    int a = 0;
    int b = 0;
    const int c = 0;
    const int d = 0;

    notPerfectForward(a); // lvalue ref
    notPerfectForward(move(b)); // lvalue ref
    notPerfectForward(c); // const lvalue ref
    notPerfectForward(move(d)); // const lvalue ref

    cout << endl;
    perfectForward(a); // lvalue ref
    perfectForward(move(b)); // rvalue ref
    perfectForward(c); // const lvalue ref
    perfectForward(move(d)); // const rvalue ref
}
```

# C++11新特性

C++11 最常用的新特性如下：

可变模板参数

auto关键字：编译器可以根据初始值自动推导出类型。但是不能用于函数传参以及数组类型的推导

nullptr关键字：nullptr是一种特殊类型的字面值，它可以被转换成任意其它的指针类型；而NULL一般被宏定义为0，在遇到重载时可能会出现问题。

智能指针：C++11新增了std::shared_ptr、std::weak_ptr等类型的智能指针，用于解决内存管理的问题。

初始化列表：使用初始化列表来对类进行初始化

右值引用：基于右值引用可以实现移动语义和完美转发，消除两个对象交互时不必要的对象拷贝，节省运算存储资源，提高效率

atomic原子操作用于多线程资源互斥操作

新增STL容器array-和vector相比，大小是固定的，以及tuple，类似于pair，不过可以有多个值

增加标准的线程库std::thread()

新增尾置返回类型：通过 **-> 符号**连接到函数后面，配合 auto 简化上述复杂函数的定义：

final和override：C++11 引⼊ **override** 显式的声明**要覆盖基类的虚函数**，如果不存在这样的虚函数，将不会通过编译。 **final** 则终⽌虚类被继承或虚函数被覆盖

```c++
//override
class Parent {
  virtual void watchTv(int);
};
class Child : Parent {
  virtual void watchTv(int) override;    // 合法
  virtual void watchTv(double) override; // 非法，父类没有此虚函数
};
//final
class Parent2 {
  virtual void eat() final;
};

class Child2 final : Parent2 {};  // 合法

class Grandson : Child2 {};       // 非法，Child2 已经 Final，不可被继承

class Child3 : Parent2 {
  void eat() override; // 非法，foo 已 final
};
```

# C++14新特性

decltype(auto)：decltype是C++11新增的关键字，主要用于提取变量和表达式的类型。

# ASIO

Boost.Asio is a cross-platform C++ library for network and low-level I/O programming that provides developers with a consistent asynchronous model using a modern C++ approach.

## 异步回调

```c++
#include<iostream>
#include<boost/asio.hpp>

#include<boost/date_time/posix_time/posix_time.hpp>


void print (const boost::system::error_code &){
    std::cout<<"end"<<std::endl;

}
int 
main(){
    boost::asio::io_service io;
    boost::asio::deadline_timer dead1(io,boost::posix_time::seconds(2));
    dead1.async_wait(&print);
    std::cout<<"hello,world1"<<std::endl;
    io.run();
    std::cout<<"hello,world"<<std::endl;
}   
```

## 参数绑定

```c++
#include<iostream>
#include<boost/asio.hpp>
#include<boost/bind/bind.hpp>

void print(const boost::system::error_code& /*e*/,
    boost::asio::steady_timer* t, int* count)
{
  if (*count < 5)
  {
    std::cout << *count << std::endl;
    ++(*count);

    t->expires_at(t->expiry() + boost::asio::chrono::seconds(1));
    t->async_wait(boost::bind(print,
          boost::asio::placeholders::error, t, count));
  }
}

int main()
{
  boost::asio::io_context io;

  int count = 0;
  boost::asio::steady_timer t(io, boost::asio::chrono::seconds(1));
  t.async_wait(boost::bind(print,
        boost::asio::placeholders::error, &t, &count));

  io.run();

  std::cout << "Final count is " << count << std::endl;

  return 0;
}
```

## 多线程同步

```c++
#include<iostream>
#include<boost/asio.hpp>
#include<boost/thread/thread.hpp>
#include<boost/bind/bind.hpp>

class printer
{
private:
    boost::asio::steady_timer timer1_;
    boost::asio::steady_timer timer2_;
    boost::asio::strand<boost::asio::io_context::executor_type>strand_;
    int count_;

public:
    printer(boost::asio::io_context& io)
    : strand_(boost::asio::make_strand(io)),
      timer1_(io, boost::asio::chrono::seconds(1)),
      timer2_(io, boost::asio::chrono::seconds(1)),
      count_(0)
  {
      
    timer2_.async_wait(boost::asio::bind_executor(strand_,
          boost::bind(&printer::print2, this)));
    timer1_.async_wait(boost::asio::bind_executor(strand_,
          boost::bind(&printer::print1, this)));

  }

 void print1()
  {
    if (count_ < 10)
    {
      std::cout << "Timer 1: " << count_ << std::endl;
      ++count_;

      timer1_.expires_at(timer1_.expiry() + boost::asio::chrono::seconds(1));

      timer1_.async_wait(boost::asio::bind_executor(strand_,
            boost::bind(&printer::print1, this)));
    }
  }

  void print2()
  {
    if (count_ < 10)
    {
      std::cout << "Timer 2: " << count_ << std::endl;
      ++count_;

      timer2_.expires_at(timer2_.expiry() + boost::asio::chrono::seconds(1));

      timer2_.async_wait(boost::asio::bind_executor(strand_,
            boost::bind(&printer::print2, this)));
    }
  }
  
    ~printer(){
        std::cout << "Final count is " << count_ << std::endl;
    }

};


int main(){
    boost::asio::io_context io;
    printer p(io);
    boost::thread t(boost::bind(&boost::asio::io_context::run, &io));
  io.run();
  t.join();

}
```



# 关键字

## constexpr

```c++
constexpr int multiply(int x,int y)
{
    return x* y;
}
//将在编译时期计算
const int var = multiply(10,10);
```

除了编译时计算性能的优化，congtexpr的另外一个优势是：允许函数被应用到以前调用宏的所有场合。例如：想要计算数组size的函数，size是10的倍数。如果不用constexpr，则需要创建一个宏或者模板，因为我们不能用函数的返回值去声明数组的大小。但是我们可以调用一个constexpr函数去声明一个数组。

```c++
constexpr int getDefaultArraySize(int value)
{
    return value*10;
}
int my_array[getDefaultArraySize(3)];
```



## new\delete   和  malloc\free

- malloc\free是C的**标准库函数**

```
void* malloc(size_t size)
void* free(void* pointer)
```



- new\delete是C++的**操作运算符**

```c++
int* p1=new int(1);
delete p1;

int*p2=new int[4];
delete []p2;
```

  （1）malloc开辟空间类型大小需手动计算，new是由编译器自己计算；

  （2）malloc返回类型为void*,**必须强制类型转换**对应类型指针，new则直接返回对应类型指针；

  （3）malloc开辟内存时返回内存地址要检查判空，因为若它可能开辟失败会返回NULL；new则不用判断，因为内存分配失败时，它会抛出异常bac_alloc,可以使用异常机制；

  （4）无论释放几个空间大小，free只传递指针，多个对象时delete需加[]



## inline

```
类内定义的函数自动内联
```



用在函数定义前，将函数指定为内联，在运行到内联函数时，不发生函数调用，也就没有了调用的开销（省去了参数压栈、生成汇编语言的CALL调用、返回参数、执行return等过程）

一般的，在类声明之中的成员函数自动称为内联函数

内联能提高函数的执行效率，为什么不把所有的函数都定义成内联函数？
如果所有的函数都是内联函数，还用得着“内联”这个关键字吗？
内联是以代码膨胀（复制）为代价，仅仅省去了函数调用的开销，从而提高函数的
执行效率。如果执行函数体内代码的时间，相比于函数调用的开销较大，那么效率的收
获会很少。另一方面，每一处内联函数的调用都要复制代码，将使程序的总代码量增大，
消耗更多的内存空间。以下情况不宜使用内联：
（1）如果函数体内的代码比较长，使用内联将导致内存消耗代价较高。
（2）如果函数体内出现循环，那么执行函数体内代码的时间要比函数调用的开销大。

## explicit

用于显示地指明构造函数禁止隐式转换

当构造函数只有一个实参时，可以只用一个实参完成初始化，所以可以隐式转换，用explicit关闭这种转换

```c++
class things{
public:
  thing(int a):num(a){}
private:
  int num;
}
thing sth = 1;//这里存在一个隐式转换
```

##restrict

在C语言中，restrict关键字用于修饰指针(C99标准)。通过加上restrict关键字，编程者可提示编译器：在该指针的生命周期内，其指向的对象不会被别的指针所引用。

需要注意的是，在C++中，并无明确统一的标准支持restrict关键字。但是很多编译器实现了功能相同的关键字，例如gcc和clang中的__restrict关键字。

那么restrict关键字能给程序的实际运行带来哪些好处呢？下面举例说明

```cpp
int add1(int* a, int* b)
{
    *a = 10;
    *b = 12;
    return *a + *b;
}
```

大家猜猜`add1`函数的返回值是多少？是10 + 12 = 22吗？

答案是不一定。在指针a和b的地址不同时，返回22没有问题。但是当指针a与b指向的是同一个int对象时，该对象先被赋值为10，后被赋值为12，因此*a和*b都返回12,因此`add1`函数最终返回24

使用`-O3`优化, `add1`对应的汇编代码如下。可以看到，在计算返回值时，为了得到`*a`的值访问了1次内存，而不管在何种条件下(`a == b` or `a != b`)，`*b`的值都是12。因此聪明的编译器将`*a`的值载入`eax`寄存器后，直接加上立即数12，而无需再访问内存获取`*b`的值。在无法确定指针a和b是否相同的情况下，编译器只能帮你到这里了.

```
0000000000400a10 <_Z4add1PiS_>:
  400a10:   c7 07 0a 00 00 00       movl   $0xa,(%rdi) ; *a = 10
  400a16:   c7 06 0c 00 00 00       movl   $0xc,(%rsi) ; *b = 10
  400a1c:   8b 07                   mov    (%rdi),%eax ; 结果 = *a
  400a1e:   83 c0 0c                add    $0xc,%eax   ; 结果 += 12 
  400a21:   c3                      retq  
```

但是如果加上了restrict关键字，情况便大不相同。C/C++和经过-O3优化的汇编代码如下。通过restrict关键字，编译器依然确认指针a和b不可能指向同一个内存地址，因此在求`*a + *b`时，无需访问内存，因为`*a`必然等于立即数10，`*b`必然等于立即数12。

```cpp
int add2(int* __restrict  a, int* __restrict b) 
{
    *a = 10;
    *b = 12;
    return *a + *b ;
}
0000000000400a30 <_Z4add2PiS_>:
  400a30:   c7 07 0a 00 00 00       movl   $0xa,(%rdi) ; *a = 10
  400a36:   b8 16 00 00 00          mov    $0x16,%eax  ; 结果 = 22
  400a3b:   c7 06 0c 00 00 00       movl   $0xc,(%rsi) ; *b = 12
  400a41:   c3                      retq   
```

通过无restrict和有restrict两种情况下的汇编指令可看到，后者比前者少访问一次内存，且少执行一条指令。因此我们预期有restrict的版本能够获得可观的性能提升：

## define和const

```
编译器处理不同
宏定义是一个“编译时”概念，在预处理阶段展开（在编译时把所有用到宏定义值的地方用宏定义常量替换），不能对宏定义进行调试，生命周期结束于编译时期；
const常量是一个“运行时”概念，在程序运行使用，类似于一个只读行数据

存储方式不同
宏定义是直接替换，不会分配内存，存储于程序的代码段中；
const常量需要进行内存分配

类型和安全检查不同
宏定义是字符替换，没有数据类型的区别，同时这种替换没有类型安全检查，可能产生边际效应等错误；
const常量是常量的声明，有类型区别，需要在编译阶段进行类型检查
```

## =default和=delete

在C++中，声明自定义的类型之后，编译器会默认生成一些成员函数，这些函数被称为默认函数。其中包括

（1）（默认）构造函数

（2）拷贝（复制）构造函数

（3）拷贝（复制）赋值运算符

（4）移动构造函数

（5）移动赋值运算符

（6）析构函数

另外，编译器还会默认生成一些操作符函数，包括

（7）operator ,

（8）operator &

（9）operator &&

（10）operator *

（11）operator ->

（12）operator ->*

（13）operator new

（14）operator delete







【1】显式缺省函数（=default）

如果类设计者又实现了这些函数的自定义版本后，编译器就不会去生成默认版本。

大多数时候，我们需要声明带参数的构造函数，此时就不会生成默认构造函数，这样会导致类不再是POD类型（可参见随笔《[C++11 POD类型](https://www.cnblogs.com/Braveliu/p/12237340.html)》），从而影响类的优化：

```c++
#include <iostream>
using namespace std;

class Test
{
public:
    Test(int i) : data(i) {}

private:
    int data;
};

int main()
{
    std::cout << std::is_pod<Test>::value << std::endl;  // 0
}
```

C++11中提供了新的机制来控制默认函数生成来避免这个问题：声明时在函数末尾加上”= default”来显式地指示编译器去生成该函数的默认版本，这样就保证了类还是POD类型：

```c++
#include <iostream>
using namespace std;

class Test
{
public:
    Test() = default;  // 显式指定缺省函数
    Test(int i) : data(i) {}

private:
    int data;
};

int main()
{
    std::cout << std::is_pod<Test>::value << std::endl;  // 1
}
```





【2】显式删除函数（=delete）

另一方面，有时候可能需要限制一些默认函数的生成。

例如：需要禁止拷贝构造函数的使用。以前通过把拷贝构造函数声明为private访问权限，这样一旦使用编译器就会报错。

而在C++11中，只要在函数的定义或者声明后面加上”= delete”就能实现这样的效果（相比较，这种方式不容易犯错，且更容易理解）：

```c++
#include <iostream>
using namespace std;

class Test
{
public:
    Test() = default;  // 显式指定缺省函数
    Test(int i) : data(i) {}
    Test(const Test& t) = delete; // 显式删除拷贝构造函数

private:
    int data;
};

int main()
{
    Test objT1;
//    Test objT2(objT1); // 无法通过编译
}
```



# 杂项

## 面向对象

```
面向对象三大特性

封装：数据和代码捆绑在一起，避免外界干扰和不确定性访问。封装可以使得代码模块化。确保用户代码不会无意间破坏封装对象的状态

继承：让某种类型对象获得另一个类型对象的属性和方法。继承可以扩展已存在的代码

多态：同一事物表现出不同事物的能力，即向不同对象发送同一消息，不同的对象在接收时会产生不同的行为（重载实现编译时多态，虚函数实现运行时多态）。多态的目的则是为了接口重用
```



## 指针和引用

```
(1)指针是一个变量，存储的是一个地址，指向内存的一个存储单元；引用跟原来的变量实质上是同一个东西，是原变量的一个别名
(2)可以有const指针，但是没有const引用
(3)指针可以有多级，但是引用只能是一级（int **p；合法 而 int &&a是不合法的）
(4)指针的值可以为空，但是引用的值不能为NULL，引用在定义的时候必须初始化；
(5)指针的值在初始化后可以改变，即指向其它的存储单元，而引用在进行初始化后就不会再改变了。
(6)"sizeof引用"得到的是所指向的变量(对象)的大小，而"sizeof指针"得到的是指针本身的大小；
(7)指针和引用的自增(++)运算意义不一样；
```

## 循环和递归

```
两者的相同点：
1.两者都可以完成循环遍历的功能。
2.两者都需要设置结束循环的条件。
3.两者每次循环或递归时，执行的程序体都是一样的。
4.两者在条件判断有误时，都可能会发生无法终止的情况（死循环和无限递归）。

两者的不同点：

递归时，每递归一层，就会在内存生成一个调用栈，来保存本次递归的信息。所以如果递归深度过深，有栈溢出的问题。循环则不会有这样的问题。
循环是一次正向的过程，递归需要回溯。
写法上的区别：递归是不断的调用自身的函数。循环则不需要。
一般来说递归的写法更简洁。
所有递归都可以改写为循环。递归实现的效率不如循环，递归可能引起严重的调用栈溢出等问题。
```

##程序是从main开始运行的吗

```
一个典型的程序运行步骤如下：
 1、操作系统在创建进程后，把控制权交到了程序的入口，这个入口往往是运行库的中的某个入口函数。
 2、入口函数对运行库和程序运行环境进行初始化，包括堆、I/O、线程、全局变量构造，等等。
 3、入口函数在完成初始化之后，调用main函数，正式开始执行程序主体部分。
 4、main函数执行完毕之后，返回到入口函数，入口函数进行清理工作，包括全局变量析构、堆销毁、关闭I/O等，然后进行系统调用结束进程。
```

## 调试时指针地址只有48位

64位系统，调试时显示的地址为48位
用是64位的系统，

调试时显示的地址为48位，

地址为48位是表象，出现这样结果的原因是x86_64处理器硬件限制。

x86_64处理器地址线只有48条，

硬件要求传入的地址48位到63位地址必须相同，
若表示为16进制，
则前4位为ffff或者是0000。
有两段合法的地址空间：
0x00000000 0000 0000-0x0000 7fff ffff ffff
0xffff8000 0000 0000-0xffff ffff ffff ffff

可表示的地址空间为2 48 2^{48}2 
48
 Byte=256TB,这就是当前处理器的寻址能力

操作系统一般使用地段地址，只用到第一段地址空间，

要用到第二段地址空间，则需要内存达到寻址空间的一般128TB。

可知：0x7fff ffff da20完整的地址其实是0x0000 7fff ffff da20,为64位

 

地址数值只有48位是表像，实际上它是64位地址，这是当前的x86_64处理器硬件限制所致。
目前面世的x86_64处理器的地址线只有48条，硬件要求传入的地址的48到63位必须与47位相同。因此有两段合法的地址空间，最直观的是0 - 0x00007fff ffffffff，另一段是0xffff8000 00000000 - 0xffffffff ffffffff。
两段加一起一共2^48 = 256TB，这就是当前处理器的寻址能力。
一般我们是见不到第二段地址的，
因为操作系统一般用低段地址，
高段这部分需要你的机器至少有128T以上内存 

## 全局对象构造顺序


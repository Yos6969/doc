

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

# 函数

## 虚函数

![3](.\img\未命名图3片.png)

![4](.\img\4.png)

virtual =0 关键字说明这是一个纯虚函数，需要在子类去实现它，但纯虚函数本身是可以有定义的。在父类中声明为加了virtual关键字的函数，在子类中重写时候不需要加virtual也是虚函数

每个对象的开头位置都有一个虚指针vptr，这个类的所有对象中的vptr指向共同的vtbl，vtbl专门存放有virtual关键字的函数

![5](.\img\5.png)



# 类型

## static关键字和extern关键字

~~~
static
~~~



- static修饰全局变量时，在静态存储器（bss、data段）开辟存储空间，其实全局变量本来就是被隐式static修饰的，被修饰后，这个变量只在本文件中使用（include进入文件也能用），如果要在其他源文件使用，需要加extern关键字
- static修饰局部变量（一般是函数中的变量，函数中的变量一般在栈中，修饰后放入data段内），约等于全局变量，在整个进程周期中，只定义和初始化一次，
- 关于初始化：可分为1.编译时初始化  2.加载时初始化  3.运行时初始化
  - 编译时初始化--静态变量是基本数据类型，且是个常值，是这样的
  - 加载时初始化--1.静态变量是一个基本数据类型，但是初始值非常量  2.静态变量是一个类对象
  - 运行时初始化--初始化发生在变量第一次被引用，对于局部静态对象是这样的 
- static修饰函数，修饰函数时，表示该函数的作用域时源文件，其他文件无法调用该函数，一般用作某文件的内部函数使用
- static修饰类成员变量：这个变量在所有这个类的对象间共享
- static修饰类成员函数：对这个类来说，这个函数只能访问类的static成员

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

对于一个左值，编译器肯定是调用拷贝构造函数，但是有些左值是**局部变量**，生命周期也很短，能不能也移动构造而不是拷贝构造呢？`C++11`为了解决这个问题，提供了`std::move()`方法来将左值转换为右值，从而方便应用移动语义

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

新增STL容器array以及tuple

# 关键字

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




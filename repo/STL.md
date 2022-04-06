# vector

## vector扩容机制

vs扩容每次增大1.5倍

# list

std::list的底层是双向链表结构，双向链表中每个元素存储在互不相关的独立节点中，在节点中通过指针指向其前一个元素和后一个元素。

# vector和list区别

| vector                                                       | list                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| vector任意位置插入和删除的效率低，因为它每插入一个元素（尾插除外），都需要搬移数据，时间复杂度是O(N)，而且插入还有可能要增容，这样一来还要开辟新空间，拷贝元素，是旧空间，效率会更低。 | list任意位置插入和删除的效率高，他不需要搬移元素，只需要改变插入或删除位置的前后两个节点的指向即可，时间复杂度为O(1)。 |
| vector由于底层是动态顺序表，在内存中是一段连续的空间，所以不容易造成内存碎片，空间利用率高，缓存利用率高。 | list的底层节点动态开辟空间，容易造成内存碎片，空间利用率低，缓存利用率低。 |
| vector适合需要高效率存储，需要随机访问，并且不管行插入和删除效率的场景。 | list适合有大量的插入和删除操作，并且不关心随机访问的场景。   |

## push_back() and emplace_back()

emplace_back:在容器尾部添加一个元素，这个元素原地构造，不需要触发拷贝构造和转移构造。而且调用形式更加简洁，直接根据参数初始化临时对象的成员

push_back()多了构造临时对象，拷贝临时对象，析构临时对象的过程

``` c++
elections.emplace_back("Nelson Mandela", "South Africa", 1994); //没有类的创建  
```

emplace_back() 和 push_back() 的区别，就在于底层实现的机制不同。push_back() 向容器尾部添加元素时，首先会创建这个元素，然后再将这个元素拷贝或者移动到容器中（如果是拷贝的话，事后会自行销毁先前创建的这个元素）；而 emplace_back() 在实现时，则是直接在容器尾部创建这个元素，省去了拷贝或移动元素的过程。

# priorty_queue<>

在优先队列中，元素被赋予优先级。当访问元素时，具有最高优先级的元素最先删除。优先队列具有最高级先出 （first in, largest out）的行为特征。

 ```c++
 //升序队列，小顶堆，小的在上面
 priority_queue <int,vector<int>,greater<int> > q;
 //降序队列，大顶堆，默认 大的在上面
 priority_queue <int,vector<int>,less<int> >q;
 
 //greater和less是std实现的两个仿函数（就是使一个类的使用看上去像一个函数。其实现就是类中实现一个operator()，这个类就有了类似函数的行为，就是一个仿函数类了）
 ```



**首先要包含头文件`#include<queue>`**, 他和`queue`不同的就在于我们可以自定义其中数据的优先级, 让优先级高的排在队列前面,优先出队。

优先队列具有队列的所有特性，包括队列的基本操作，只是在这基础上添加了内部的一个排序，它本质是一个堆实现的。

> 和队列基本操作相同:
>
> - top 访问队头元素
> - empty 队列是否为空
> - size 返回队列内元素个数
> - push 插入元素到队尾 (并排序)
> - emplace 原地构造一个元素并插入队列
> - pop 弹出队头元素
> - swap 交换内容

# 全排列-next_permuation()

```c++
#include<iostream>
#include<stdio.h>
#include<queue>
#include<string>
#include<string.h>
#include<algorithm>
using namespace std;

int main()
{
    int a[3]={1,2,3};
    do
    {
        cout<<a[0]<<" "<<a[1]<<" "<<a[2]<<endl;
    }while(next_permutation(a,a+3));
    return 0;
}
//输出
/*
 1 2 3
 1 3 2
 2 1 3
 2 3 1
 3 1 2
 3 2 1
 */
```



# std::function

不同类型可能具有相同的调用形式，如：

```cpp
// 普通函数
int add(int a, int b){return a+b;} 

// lambda表达式
auto mod = [](int a, int b){ return a % b;}

// 函数对象类
struct divide{
    int operator()(int denominator, int divisor){
        return denominator/divisor;
    }
};
```

上述三种可调用对象虽然类型不同，但是共享了一种调用形式：

```cpp
int(int ,int)
```

std::function就可以将上述类型保存起来，如下：

```cpp
std::function<int(int ,int)>  a = add; 
std::function<int(int ,int)>  b = mod ; 
std::function<int(int ,int)>  c = divide(); 
```

# 函数子和函数子类

无论是C还是C++,都不允许将一个函数作为参数传递给另一个函数，必须传递函数指针

而**函数子**（函数对象）是这种指针的一种抽象和建模形式。在STL中，函数对象在函数之间来回传递时按值传递。以foreach为例，它需要一个函数对象作为参数，同时其返回值也是一个函数对象

- 判别式类(predicate class)时一个**函数子类**，这个类的operator()函数是一个判别式且返回值为bool，常常用在STL算法中，在STL中，凡是能接收判别式的地方，就既可以接收一个真正的判别式，也可以接收一个判别式类对象

对于函数子，常常需要考虑配接的问题：

```c++
list<Widget*>widgetPtrs;
bool isInteresting(const Widget*pw);//这是一个未配接的函数

list<Widget*>::iterator i = find_if(widgetPtrs.begin(),widgetPtrs.end(),isInteresting);
list<Widget*>::iterator i1 =find_if(widgetPtrs.begin(),widgetPtrs.end(),not1(isInteresting));//错误用法
```

对一个普通的函数，不能按照函数子的方法进行操作，需要使用ptr_fun进行配接

```c++
list<Widget*>::iterator i1 =find_if(widgetPtrs.begin(),widgetPtrs.end(),not1(ptr_fun(isInteresting)));
```

为什么呢？其实 ptr_fun是将这样一个普通的函数，定义出了一个函数子的对象，isInteresting这个普通函数缺少not1所需要的类型定义。

STL标准配接器有四种	*not1\not2\bind1st\bind2st*

STL提供了这种类型定义的模板	*unary_function*和*binary_function* 

1.对于 unary_function,在继承模板后，指定operator()所带参数的类型，以及operator()的返回类型

2.对于 binary_function,  在继承模板后，指定operator()所带第一个和第二个参数的类型,以及operator() 的返回类型

```c++
template<typename T>
class MeetThreshold:public std::unary_function<Widget,bool>{
public:
  	MeetThreshold(const T& threshold);
	bool operator()(const Widget&) const;
private:
	const T threshold;
}
//有状态的函数子类一般定义为class，无状态定义为struct
struct WidgetNameCompare:public std::binary_function<Widget,Widget,bool>{
  	bool operator()(const Widget& lhs,const Widget& rhs) const;
}
```



## bind配接器

bind是STL中的一个适配器

```c++
class Solution {
public:
    bool AZ(const string &lhs,const string &rhs){
        if(lhs.size()<rhs.size())
            return true;
        int sum1=0,sum2=0;
        for(int i=0;i<lhs.size();i++){
            sum1+=lhs[i];
            sum2+=rhs[i];
        }
        if(sum1<sum2)
            return true;
    }

    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        vector<vector<string>>ans;
        sort(strs.begin(),strs.end(),std::bind(&Solution::AZ, this,placeholders::_1, placeholders::_2));

    }
};
```



### std::bind`和 std::placeholder

`std::bind` 是用来绑定函数调用的参数的， 它解决的需求是我们有时候可能并不一定能够一次性获得调用某个函数的全部参数，通过这个函数， 我们可以将部分调用参数提前绑定到函数身上成为一个新的对象，然后在参数齐全后，完成调用。 例如：

```c++
int foo(int a, int b, int c) {
    ;
}
int main() {
    // 将参数1,2绑定到函数 foo 上，但是使用 std::placeholders::_1 来对第一个参数进行占位
    auto bindFoo = std::bind(foo, std::placeholders::_1, 1,2);
    // 这时调用 bindFoo 时，只需要提供第一个参数即可
    bindFoo(1);
}

```

> **提示：**注意 `auto` 关键字的妙用。有时候我们可能不太熟悉一个函数的返回值类型， 但是我们却可以通过 `auto` 的使用来规避这一问题的出现。

```c++
int f(int a, int b)
{
    return a + b;
}

int g(int a, int b, int c)
{
    return a + b + c;
}
bind(f, 1, 2) will produce a "nullary" function object that takes no arguments and returns f(1, 2). Similarly, bind(g, 1, 2, 3)() is equivalent to g(1, 2, 3).
std::bind1st(std::ptr_fun(f), 5)(x);   // f(5, x)
bind(f, 5, _1)(x);         // f(5, x)
bind(f, _2, _1)(x, y);                 // f(y, x)
bind(g, _1, 9, _1)(x);                 // g(x, 9, x)
bind(g, _3, _3, _3)(x, y, z);          // g(z, z, z)
bind(g, _1, _1, _1)(x, y, z);          // g(x, x, x)
int x = 8;
bind(std::less<int>(), _1, 9)(x);	// x < 9
```

## mem_fun()&&mem_fun_ref()

在一些例如foreach的算法中，foreach默认你使用的函数是非成员函数，但如果你的接收函数是成员函数，那就得使用配接器,即“在需要将成员函数传递给STL组件”时使用

```c++
class Wd{
public:
    void test(){
        cout<<1<<endl;
    }
};

void test1(Wd &w){
    cout<<1<<endl;
}

int main()
{
  vector<Wd>vec(2,Wd());

  for_each(vec.begin(),vec.end(),test1);// without menfun 1 1
  for_each(vec.begin(),vec.end(),mem_fun_ref(&Wd::test));// 1 1

  vector<Wd*>vec1(2,new Wd());
  for_each(vec1.begin(),vec1.end(),mem_fun(&Wd::test));// 1 1


}
```



## ptr_func()

将一个普通函数，配接成函数子对象

```c++
void test1(Wd &w){
    cout<<1<<endl;
}

int main()
{
  vector<Wd>vec(2,Wd());

  for_each(vec.begin(),vec.end(),test1);// 这两个是一样的
  for_each(vec.begin(),vec.end(),ptr_fun(test1));// 但如果要使用函数子对象的完整功能，要用ptr_fun配接
```



# lambda（c++11）

格式   [capture] (params) opt ->ret { body}

```c++
[](double x )->double{int y = x ;return x - y;};
```

可以不写  ->double 让编译器自己推断

```c++
auto f = [](int x ){return x % 3 ==0;};
```

虽然lambda表达式是匿名函数，实际上也可以给lambda一个指定名称 

其中carpture是捕获列表，params是参数，opt是选项，ret则是返回值的类型，body则是函数的具体实现。
1.捕获列表描述了lambda表达式可以访问上下文中的哪些变量。
[] :表示不捕获任何变量
[=]：表示按值捕获变量
[&]：表示按引用捕获变量
[this]：值传递捕获当前的this

# transform

transform函数的作用是：将某操作应用于指定范围的每个元素。有两种形式，分别对应一元和二元操作

transform(first,last,result,op);//first是容器的首迭代器，last为容器的末迭代器，result为存放结果的容器，op为要进行操作的一元函数对象或sturct、class。

transform(first1,last1,first2,result,binary_op);//first1是第一个容器的首迭代 器，last1为第一个容器的末迭代器，first2为第二个容器的首迭代器，result为存放结果的容器，第一个容器的每个值作为第一个参数，第二个容器的每个值作为第二个参数，binary_op为要进行操作的二元函数 对象或sturct、class。

# foreach

和transform功能类似，但foreach可以有返回值，返回的是pred中的比较类，如下

``` c++

struct print02{
    print02(){
        mCount= 0;
    }
    void operator()(int val){
        cout<< val << " ";
        mCount++;
    }
    int mCount;
};
int main(){
  
    print02 p = for_each(v.begin(), v.end(), print02());

}

```



# 排序算法

| 函数名                                      | 用法                                       |
| ---------------------------------------- | ---------------------------------------- |
| sort (first, last)                       | 对容器或普通数组中 [first, last) 范围内的元素进行排序，默认进行升序排序。 |
| stable_sort (first, last)                | 和 sort() 函数功能相似，不同之处在于，对于 [first, last) 范围内值相同的元素，该函数不会改变它们的相对位置。 |
| partial_sort(first,middle,last)          | 从 [first,last) 范围内，筛选出 muddle-first 个最小的元素并排序存放在 [first，middle) 区间中。 |
| partial_sort_copy (first, last, result_first, result_last) | 从 [first, last) 范围内筛选出 result_last-result_first 个元素排序并存储到 [result_first, result_last) 指定的范围中。 |
| is_sorted (first, last)                  | 检测 [first, last) 范围内是否已经排好序，默认检测是否按升序排序。 |
| is_sorted_until (first, last)            | 和 is_sorted() 函数功能类似，唯一的区别在于，如果 [first, last) 范围的元素没有排好序，则该函数会返回一个指向首个不遵循排序规则的元素的迭代器。 |
| nth_element (first, nth, last)           | 找到 [first, last) 范围内按照排序规则（默认按照升序排序）应该位于第 nth 个位置处的元素，并将其放置到此位置。同时使该位置左侧的所有元素都比其存放的元素小，该位置右侧的所有元素都比其存放的元素大。 |


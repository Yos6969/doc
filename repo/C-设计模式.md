[toc]

# 单例

##有缺陷的懒汉式

```c++
template<typename T>
class Singleton{
private:
    Singleton(){
    }
    Singleton(Singleton&)=delete;
    Singleton& operator=(const Singleton&)=delete;
    static T* m_instance_ptr;
public:
    ~Singleton(){
    }
    static T* get_instance(){
        if(m_instance_ptr==nullptr){
              m_instance_ptr = new Singleton;
        }
        return m_instance_ptr;
    }
};

T* Singleton::m_instance_ptr = nullptr;
```

## 推荐的懒汉式单例

```c++
class singleton{
public:
    static  singleton& getinstance()
    {
//static singleton instance; 放这也行
        return instance;
    }

private:
    static singleton instance;
    singleton(){}
};

singleton singleton::instance; //需要在类外实例化
int main()
{
    singleton s1 = singleton::getinstance();
    singleton s2 = singleton::getinstance();
    cout<<&s1<<endl<<&s2<<endl;
}
```

- 在内存中只有一个对象，节省内存空间。
- 避免频繁的创建销毁对象，可以提高性能。
- 避免对共享资源的多重占用。
- 可以全局访问。

例子：web后台的 访问统计、流量统计manager

#工厂

解耦
通过工厂模式可以把对象的创建和使用过程分割开来。比如说 Class A 想调用 Class B的方法，那么我们无需关心B是如何创建的，直接去工厂获取就行。
减少代码量，易于维护
如果我们直接new一个对象时，如果需要的对象构造方法比较复杂，那么可能需要一连串的代码去创建对象，如果在别的类中又需要创建该对象，那么代码的重复度肯定不小。通过工厂模式的话，我们把对象创建的具体逻辑给隐藏起来了，交给工厂统一管理，这样不仅减少了代码量，以后如果想改代码的话，只需要改一处即可，也方便我们日常的维护。

## 简单工厂

```c++
#include <iostream>
using namespace std;

enum CarType{BENZ, BMW};

class Car//车类
{
public:
    virtual void createdCar(void) = 0;
};

class BenzCar : public Car //奔驰车
{
public:
    BenzCar()
    {
        cout<<"Benz::Benz()"<<endl;
    }
    virtual void createdCar(void)
    {
        cout<<"BenzCar::createdCar()"<<endl;
    }
    ~BenzCar()
    {

    }
};

class BmwCar : public Car //宝马车
{
public:
    BmwCar()
    {
        cout<<"Bmw::Bmw()"<<endl;
    }
    virtual void createdCar(void)
    {
        cout<<"BmwCar::createdCar()"<<endl;
    }
};


class CarFactory //车厂
{
public:
    Car* createSpecificCar(CarType type)
    {
        switch(type)
        {
        case BENZ://生产奔驰车
            return (new BenzCar());
            break;
        case BMW://生辰宝马车
            return (new BmwCar());
            break;
        default:
            return NULL;
            break;
        }
    }
};

int main(int argc, char** argv)
{
    CarFactory carfac;
    Car* specificCarA = carfac.createSpecificCar(BENZ);
    Car* specificCarB = carfac.createSpecificCar(BMW);   
    return 0;
}
```

## 工厂方法模式

不再只由一个工厂类决定那一个产品类应当被实例化,这个决定权被交给子类去做。当有新的产品（新型汽车）产生时，只要按照抽象产品角色、抽象工厂角色提供的方法来生成即可（新车型可以用一个新类继承创建产品即可），那么就可以被客户使用，而不必去修改任何已有的代 码。可以看出工厂角色的结构也是符合开闭原则。

```c++
#include <iostream>
using namespace std;

class Car//车类
{
public:
    virtual void createdCar(void) = 0;
};

class BenzCar : public Car //奔驰车
{
public:
    BenzCar()
    {
        cout<<"Benz::Benz()"<<endl;
    }
    virtual void createdCar(void)
    {
        cout<<"BenzCar::createdCar()"<<endl;
    }
    ~BenzCar()
    {

    }
};

class BmwCar : public Car //宝马车
{
public:
    BmwCar()
    {
        cout<<"Bmw::Bmw()"<<endl;
    }
    virtual void createdCar(void)
    {
        cout<<"BmwCar::createdCar()"<<endl;
    }
};


class Factory//车厂
{
public:
    virtual Car* createSpecificCar(void) = 0;
};

class BenzFactory : public Factory//奔驰车厂
{
public:
    virtual Car* createSpecificCar(void)
    {
        return (new BenzCar());
    }
};

class BmwFactory : public Factory//宝马车厂
{
public:
    virtual Car* createSpecificCar(void)
    {
        return (new BmwCar());
    }
};


int main(int argc, char** argv)
{
    Factory* factory = new BenzFactory();
    Car* specificCarA = factory->createSpecificCar();
    factory = new BmwFactory();
    Car* specificCarB = factory->createSpecificCar();    
    return 0;
}
```

## 抽象工厂

在上面的工厂方法模式基础上，有需要生产高配版的奔驰和宝马，那工厂方法模式就有点鞭长莫及了，这就又有抽象工厂模式

```c++
#include <iostream>
using namespace std;

class Car//车类
{
public:
    virtual void createdCar(void) = 0;
};

class BenzCar : public Car //奔驰车
{
public:
    BenzCar()
    {
        cout<<"Benz::Benz()"<<endl;
    }
    virtual void createdCar(void)
    {
        cout<<"BenzCar::createdCar()"<<endl;
    }
    ~BenzCar()
    {

    }
};

class BmwCar : public Car //宝马车
{
public:
    BmwCar()
    {
        cout<<"Bmw::Bmw()"<<endl;
    }
    virtual void createdCar(void)
    {
        cout<<"BmwCar::createdCar()"<<endl;
    }
};

class HighCar //高配版车型
{
public:
    virtual void createdCar(void) = 0;
};

class HighBenzCar : public HighCar //高配奔驰车
{
public:
    HighBenzCar()
    {
        cout<<"HighBenzCarBenz::Benz()"<<endl;
    }
    virtual void createdCar(void)
    {
        cout<<"HighBenzCar::createdCar()"<<endl;
    }
};

class HighBmwCar : public HighCar //高配宝马车
{
public:
    HighBmwCar()
    {
        cout<<"HighBmwCar::Bmw()"<<endl;
    }
    virtual void createdCar(void)
    {
        cout<<"HighBmwCar::createdCar()"<<endl;
    }
};

class Factory//车厂
{
public:
    virtual Car* createSpecificCar(void) = 0;
    virtual HighCar* createdSpecificHighCar(void) = 0;
};

class BenzFactory : public Factory//奔驰车厂
{
public:
    virtual Car* createSpecificCar(void)
    {
        return (new BenzCar());
    }

    virtual HighCar* createdSpecificHighCar(void)
    {
        return (new HighBenzCar());
    }
};

class BmwFactory : public Factory//宝马车厂
{
public:
    virtual Car* createSpecificCar(void)
    {
        return (new BmwCar());
    }
    virtual HighCar* createdSpecificHighCar(void)
    {
        return (new HighBmwCar());
    }
};


int main(int argc, char** argv)
{
    Factory* factory = new BenzFactory();
    Car* specificCar = factory->createSpecificCar();
    HighCar* spcificHighCar = factory->createdSpecificHighCar();
   
    
    return 0;
}
```

#reactor模式

##**模式一：基本的Reactor模式（单线程Reactor）**

单线程Reactor的程序结构如下：

![wpsC334.tmp](C:\Users\18181\Desktop\doc\repo\img\1485398-20181022232217633-124484857.jpg)

上图中的**Reactor**的基本结构是一个事件循环(event loop),以事件驱动(event-driven)和事件回调的方式实现业务逻辑，伪代码如下：

```text
while (!done)
{
  int retval = :poll(fds, nfds, timeout_ms); // select和epoll等多路复用器类似   if (retval < 0) {
      // 处理错误哦，回调用户的error handler   } else if (retval > 0) {
      // 处理IO事件(也可以处理通过timerfd的定时器事件)，回调用户的IO event handler   }
 
}
```

上图中的**acceptor**的作用是：当监听描述符(listen  fd)可读时。acceptor会接受客户端的连接请求，并将新建立的连接(描述符)注册到Reactor中，等待可读事件发生(本模式中只有一个Reactor，后面会降到多Reactor模式，在多Reactor模式中，acceptor会将新建立的连接根据一定规则注册到不同的Reactor中)

上面仅仅是一个粗略的模型，没有给出codec或者dispatcher，在实际的服务器实现中，codec和dispather是必不可少的。codec负责解析消息，处理消息可能出现的各种情况，比如，如果是收到了半条消息，那么不会触发消息事件回调，数据会停留在Buffer里面(数据已经读到Buffer中了)，等待收到一个完整的消息再通知处理函数。  ，dispather则根据消息分发到不同的处理方法中。

**此模式的特点：** 由于只有一个线程，因此事件是顺序处理的，一个线程同时只能做一件事情，事件的优先级得不到保证。因为”从poll返回后”   到”下一次调用poll进入等待之前”这段时间内，线程不会被其他连接上的数据或者事件抢占。所以在使用这种模式时，需要避免业务逻辑阻塞当前IO线程。   redis就是采用的这种线程模型，在redis中，鉴于大部分操作，如GET、SET等，处理速度都很快，所以redis的整体性能还算不错，但是如果进行某些比较耗时的操作，如在数据量很大的情况下执行KEYS，会阻塞住IO线程，导致其他请求无法快速响应。也是基于这个原因某些Redis中间件方案(如twemproxy)会将这些耗时的操作禁用。

1、 当其中某个 handler 阻塞时， 会导致其他所有的 client 的 handler 都得不到执行， 并且更严重的是， handler 的阻塞也会导致整个服务不能接收新的 client 请求(因为 acceptor 也被阻塞了)。 因为有这么多的缺陷， 因此单线程Reactor 模型用的比较少。这种单线程模型不能充分利用多核资源，所以实际使用的不多。

2、因此，单线程模型仅仅适用于handler 中业务处理组件能快速完成的场景。

##**模式二： Reactor + 线程池(Thread Pool)**

此模式的程序结构如下：

![image](C:\Users\18181\Desktop\doc\repo\img\1485398-20181022232219420-1734756772.png)

相比于模式一，此模式中收到请求后，不在Reactor线程计算，而是使用线程池来计算，这会充分的利用多核CPU。采用此模式时有可能存在多个线程同时计算同一个连接上的多个请求，算出的结果的次序是不确定的，  所以需要网络框架在设计协议时带一个id标示，以便以便让客户端区分response对应的是哪个request。

##**模式三: Multiple Reactors**

此模式的程序结构如下：

![img](C:\Users\18181\Desktop\doc\repo\img\reactor3.png)

此模式的特点是one loop per thread， 有一个main Reactor负责accept连接, 然后把该连接挂在某个sub  Reactor中(可以采用round-robin或者随机方法)，这样该连接的所有操作都在哪个sub  Reactor所处的线程中完成。多个连接可能被分配到多个线程中，充分利用CPU。在应用场景中，Reactor的个数可以采用  固定的个数，比如跟CPU数目一致。此模式与模式二相比，减少了进出thread  pool两次上线文切换，小规模的计算可以在当前IO线程完成并且返回结果，降低响应的延迟。并可以有效防止当IO压力过大时一个Reactor处理能力饱和问题。**纯转发的proxy服务适合使用这种模式**

##**模式四: Multiple Reactors + 线程池(Thread Pool)**

此模式的程序结构如下

![img](C:\Users\18181\Desktop\doc\repo\img\reactor4.png)

此模式是模式二和模式三的混合体，它既使用多个 reactors 来处理 IO，又使用线程池来处理计算。此模式适适合既有突发IO(利用Multiple Reactor分担)，又有突发计算的应用（利用线程池把一个连接上的计算任务分配给多个线程）。

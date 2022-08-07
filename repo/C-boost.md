[toc]

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

# 计时器

```c++
#include <chrono> 
auto t1 = chrono::steady_clock::now();
......
auto t2 = chrono::steady_clock::now();
auto t = chrono::duration_cast<chrono::milliseconds>(t2 - t1);
cout << "time cost:" << t.count() << " 毫秒" << endl;

```

# 线程休眠

```c++
std::chrono::nanoseconds
std::chrono::microseconds
std::chrono::milliseconds
std::chrono::seconds
std::chrono::minutes
std::chrono::hours
std::this_thread::sleep_for(std::chrono::milliseconds(100));
```



# 杂项

## enable_shared_from_this

一.使用场合

       当类A被share_ptr管理，且在类A的成员函数里需要把当前类对象作为参数传给其他函数时，就需要传递一个指向自身的share_ptr。
       只能public继承
1.为何不直接传递this指针

       使用智能指针的初衷就是为了方便资源管理，如果在某些地方使用智能指针，某些地方使用原始指针，很容易破坏智能指针的语义，从而产生各种错误。

2.可以直接传递share_ptr<this>么？

       答案是不能，因为这样会造成2个非共享的share_ptr指向同一个对象，未增加引用计数导对象被析构两次。例如：

```c++
#include <memory>
#include <iostream>
using namespace std;
class Bad : public std::enable_shared_from_this<Bad>
{
public:
    std::shared_ptr<Bad> getptr() {
        //return std::shared_ptr<Bad>(this);  // 错误的示例，每个shared_ptr都认为自己是对象仅有的所有者,引用计数都为1;
        return shared_from_this();;
    }
    ~Bad() { std::cout << "Bad::~Bad() called" << std::endl; }
};

int main()
{
    // 错误的示例，每个shared_ptr都认为自己是对象仅有的所有者;
    std::shared_ptr<Bad> bp1(new Bad());
    std::shared_ptr<Bad> bp2 = bp1->getptr();
    // 打印bp1和bp2的引用计数;
    std::cout << "bp1.use_count() = " << bp1.use_count() << std::endl;
    std::cout << "bp2.use_count() = " << bp2.use_count() << std::endl;
}  // Bad 对象将会被删除两次;
```


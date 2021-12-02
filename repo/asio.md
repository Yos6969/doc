# 基本原理

大多数程序以某种方式与外界交互，无论是通过文件、网络、串行电缆还是控制台。有时，就像网络一样，单个 I/O 操作可能需要很长时间才能完成。这对应用程序开发提出了特殊的挑战。

Boost.Asio 提供了管理这些长时间运行的操作的工具，而无需程序使用基于线程和显式锁定的并发模型。

Boost.Asio 库适用于使用 C++ 进行系统编程的程序员，这些程序员通常需要访问操作系统功能，例如网络。特别是，Boost.Asio 解决了以下目标：

- **可移植性。**该库应该支持一系列常用的操作系统，并在这些操作系统之间提供一致的行为。
- **可扩展性。**该库应该促进可扩展到数千个并发连接的网络应用程序的开发。每个操作系统的库实现应该使用最能实现这种可伸缩性的机制。
- **效率。**该库应支持分散-聚集 I/O 等技术，并允许程序最大限度地减少数据复制。
- **来自已建立的 API 的模型概念，例如 BSD 套接字。**BSD 套接字 API 被广泛实现和理解，并且在许多文献中都有涉及。其他编程语言通常使用类似的网络 API 接口。在合理的情况下，Boost.Asio 应该利用现有的实践。
- **便于使用。**图书馆应该通过采用工具包而不是框架的方法为新用户提供较低的进入门槛。也就是说，它应该尽量减少前期投资，只需要学习一些基本的规则和指导方针。之后，库用户只需了解正在使用的特定功能。
- **进一步抽象的基础。**该库应该允许开发其他提供更高抽象级别的库。例如，HTTP 等常用协议的实现。

尽管 Boost.Asio 一开始主要关注网络，但它的异步 I/O 概念已经扩展到包括其他操作系统资源，例如串行端口、文件描述符等。

Boost.Asio 可用于对 I/O 对象（如套接字）执行同步和异步操作。在使用 Boost.Asio 之前，了解一下 Boost.Asio 的各个部分、您的程序以及它们如何协同工作的概念图可能会很有用。

作为介绍性示例，让我们考虑在套接字上执行连接操作时会发生什么。我们将从检查同步操作开始。

![同步操作](.\img\22.png)

**您的程序**将至少有一个**I/O 执行上下文**，例如一个对象、 对象或. 此**I/O 执行上下文**表示**您的程序**与**操作系统**I/O 服务的链接。 `boost::asio::io_context``boost::asio::thread_pool``boost::asio::system_context`

```
boost :: asio :: io_context  io_context ;
```

要执行 I/O 操作，**您的程序** 将需要一个**I/O 对象，**例如 TCP 套接字：

```
boost :: asio :: ip :: tcp :: socket  socket ( io_context );
```

##多线程同步

对于一个 boost::aiso::io_context 对象，在多线程回调中，使用了strand（链）来保证回调之间的隔离性，对于通过它来分派的处理程序，正在执行的处理程序将允许进入下一个开始之前完成担保。无论调用[io_context::run()](https://www.boost.org/doc/libs/1_77_0/doc/html/boost_asio/reference/io_context/run.html)的线程数如何，都可以保证这一点。当然，处理程序仍可能与其他未通过[链](https://www.boost.org/doc/libs/1_77_0/doc/html/boost_asio/reference/strand.html)分派或通过不同[链](https://www.boost.org/doc/libs/1_77_0/doc/html/boost_asio/reference/strand.html) 对象分派的处理程序同时执行。

```c++
/*
 * @Author: your name
 * @Date: 2021-11-30 20:00:33
 * @LastEditTime: 2021-11-30 21:09:36
 * @LastEditors: Please set LastEditors
 * @Description: 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
 * @FilePath: /asio/multithread_syn.cpp
 */
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




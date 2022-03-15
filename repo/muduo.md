# 互斥器mutex

1. 用RAII手法封装mutex的创建、销毁、加锁、解锁四个操作
2. 只用非递归的mutex
3. 不适用跨进程的mutex，进程间通信只用TCP sockets

```
pthread_mutex_t 和 pthread_cond_t是互斥量和条件变量
互斥量防止多个线程同时访问某一共享变量
条件变量允许某一线程就某个共享变量的状态变化通知其他线程，并让其他线程等待这一通知
```





## 非递归mutex

mutex可分为递归和非递归两种，也可叫做可重入和不可重入，区别在于：同一个线程可以重复对recursive mutex加锁，但不能重复对non-recursive mutex加锁。同一线程里对非递归mutex多次加锁会立即导致死锁



## 不要用读写锁和信号量

```
读写锁特点

1 如果一个线程用读锁锁定了临界区，那么其他线程也可以用读锁来进入临界区，这样可以有多个线程并行操作。这个时候如果再用写锁加锁就会发生阻塞。写锁请求阻塞后，后面继续有读锁来请求时，这些后来的读锁都将会被阻塞。这样避免读锁长期占有资源，防止写锁饥饿。

2 如果一个线程用写锁锁住了临界区，那么其他线程无论是读锁还是写锁都会发生阻塞。

```



初学者常把频繁读而很少写的数据结构加上读写锁来保护共享状态，其实：

1. 从性能来说，读写锁不一定比普通mutex更高效，无论如何reader lock加锁的开销不会比mutex lock小，因为要更新当前reader的数目
2. reader lock可能允许提升为writer lock，也可能不允许提升
3. reader lock是可重入的，writer lock是不可重入的，但为了防止writer饥饿，writer lock通常会阻塞后来的reader lock，因此reader lock在重入的时候也可能死锁

## 封装MutexLock、LockGuard、Condition

```c++
#ifndef MUTEX_H
#define MUTEX_H
#include <assert.h>
#include <pthread.h>
namespace muduo
{
class  MutexLock :boost::noncopyable
{
 public:
  MutexLock()
    : holder_(0)
  {
   pthread_mutex_init(&mutex_, NULL);
  }

  ~MutexLock()
  {
    assert(holder_ == 0);
    pthread_mutex_destroy(&mutex_);
  }

  void lock()
  {
    pthread_mutex_lock(&mutex_);
   // assignHolder();
  }

  void unlock() 
  {
   // unassignHolder();
    pthread_mutex_unlock(&mutex_);
  }

  pthread_mutex_t* getPthreadMutex() /* non-const */
  {
    return &mutex_;
  }


private:
  pthread_mutex_t mutex_;
};

class  MutexLockGuard : boost::noncopyable
{
 public:
  explicit MutexLockGuard(MutexLock& mutex) 
    : mutex_(mutex)
  {
    mutex_.lock();
  }

  ~MutexLockGuard() 
  {
    mutex_.unlock();
  }

 private:

  MutexLock& mutex_;
};

}  // namespace muduo

#endif
```

```c++
#include "Mutex.h"
#include "pthread.h"
namespace muduo{

class Condition :boost::noncopyable{
public:
    explicit Condition(MutexLock & mutex):mutex_(mutex){
        pthread_cond_init(&pcond_,NULL);
    }

    ~Condition(){
        pthread_cond_destroy(&pcond_);
    }

    void wait(){
        pthread_cond_wait(&pcond_,mutex_.getPthreadMutex());
    }

    void notify(){
        pthread_cond_signal(&pcond_);
    }

    void notifyall(){
        pthread_cond_broadcast(&pcond_);
    }

private:
    MutexLock& mutex_;
    pthread_cond_t pcond_;
};



}
```

# 单例Singleton，线程安全

## pthread_once()

在多线程环境中，有些事仅需要执行一次。通常当初始化应用程序时，可以比较容易地将其放在main函数中。但当你写一个库时，就不能在main里面初始化了，你可以用静态初始化，但使用一次初始化（pthread_once）会比较容易些。

```
int pthread_once(pthread_once_t *once_control, void (*init_routine) (void))；

功能：本函数使用初值为PTHREAD_ONCE_INIT的once_control变量保证init_routine()函数在本进程执行序列中仅执行一次。
```

在多线程编程环境下，尽管pthread_once()调用会出现在多个线程中，init_routine()函数仅执行一次，究竟在哪个线程中执行是不定的，是由内核调度来决定。

Linux Threads使用互斥锁和条件变量保证由pthread_once()指定的函数执行且仅执行一次，而once_control表示是否执行过。

如果once_control的初值不是PTHREAD_ONCE_INIT（Linux Threads定义为0），pthread_once() 的行为就会不正常。

在LinuxThreads中，实际"一次性函数"的执行状态有三种：NEVER（0）、IN_PROGRESS（1）、DONE （2），如果once初值设为1，则由于所有pthread_once()都必须等待其中一个激发"已执行一次"信号，因此所有pthread_once ()都会陷入永久的等待中；如果设为2，则表示该函数已执行过一次，从而所有pthread_once()都会立即返回0。

# 多线程编程

##多线程服务器的适用场合与常用模型

###单线程常用模型

例如

Reactor模式 --非阻塞IO+多路复用

程序的基本结构是一个事件循环，以事件驱动和事件回调的方式实现业务逻辑

###多线程常用模型

1.非阻塞IO+one loop per thread

好处：

- 线程数目基本固定，不会频繁创建与销毁
- 可以方便地在线程间调配负载
- IO事件发生的线程是固定地，同一个TCP连接不必考虑事件并发



## 进程间通信只用TCP

进程间通信首选Sockets，可以跨主机，把进程分散到同一局域网地多台机器上，而且TCP是双向的

##多线程与IO


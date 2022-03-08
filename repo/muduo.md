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
class CAPABILITY("mutex") MutexLock : noncopyable
{
 public:
  MutexLock()
    : holder_(0)
  {
    MCHECK(pthread_mutex_init(&mutex_, NULL));//动态初始化互斥量
  }

  ~MutexLock()
  {
    assert(holder_ == 0);
    MCHECK(pthread_mutex_destroy(&mutex_));
  }

  // must be called when locked, i.e. for assertion
  bool isLockedByThisThread() const
  {
    return holder_ == CurrentThread::tid();
  }

  void assertLocked() const ASSERT_CAPABILITY(this)
  {
    assert(isLockedByThisThread());
  }

  // internal usage

  void lock() ACQUIRE()
  {
    MCHECK(pthread_mutex_lock(&mutex_));
    assignHolder();
  }

  void unlock() RELEASE()
  {
    unassignHolder();
    MCHECK(pthread_mutex_unlock(&mutex_));
  }

  pthread_mutex_t* getPthreadMutex() /* non-const */
  {
    return &mutex_;
  }

 private:
  friend class Condition;

  class UnassignGuard : noncopyable
  {
   public:
    explicit UnassignGuard(MutexLock& owner)
      : owner_(owner)
    {
      owner_.unassignHolder();
    }

    ~UnassignGuard()
    {
      owner_.assignHolder();
    }

   private:
    MutexLock& owner_;
  };

  void unassignHolder()
  {
    holder_ = 0;
  }

  void assignHolder()
  {
    holder_ = CurrentThread::tid();
  }

  pthread_mutex_t mutex_;
  pid_t holder_;
};

```


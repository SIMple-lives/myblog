---
photo: https://www.educative.io/v2api/editorpage/6705609052258304/image/6189857970847744
title: c++ thread notes
time: 2024-4-9
tag:
- notes
- thread
- c++
category: notes
---

# C++11线程特性

## std::thread

[标准库](https://en.cppreference.com/w/cpp/thread/thread)

C++11引入了`std::thread`来创建线程，支持对线程join或者detach.

```c++
#include <iostream>
#include <thread>

using namespace std;

int main() {
    auto func = []() {
        for (int i = 0; i < 10; ++i) {
            cout << i << " ";
        }
        cout << endl;
    };
    std::thread t(func);
    if (t.joinable()) {
        t.detach();
    }
    auto func1 = [](int k) {
        for (int i = 0; i < k; ++i) {
            cout << i << " ";
        }
        cout << endl;
    };
    std::thread tt(func1, 20);
    if (tt.joinable()) { // 检查线程可否被join
        tt.join();
    }
    return 0;
}
```

```c++
class ThreadGuard
{
public:
    enum class DeAction{join,detach};
    ThreadGuard(std::thread&& t,DeAction action): m_t(move(t)) , m_action(action){};
    ~ThreadGuard()
    {
        if(m_t.joinable())
        {
            if(m_action==DeAction::join)
            {
                m_t.join();
            }
            else
            {
                m_t.detach();
            }
        }
    }
    ThreadGuard(ThreadGuard&&)=default;
    ThreadGuard& operator=(ThreadGuard&&)=default;
private:
    std::thread m_t;
    DeAction m_action;
};
```

c++11还提供了获取线程id，或者系统cpu个数，获取thread native_handle，使得线程休眠等功能

```c++
std::thread t(func);
cout << "当前线程ID " << t.get_id() << endl;
cout << "当前cpu个数 " << std::thread::hardware_concurrency() << endl;
auto handle = t.native_handle();// handle可用于pthread相关操作
std::this_thread::sleep_for(std::chrono::seconds(1));
```

## std::mutex

std::mutex是一种线程同步的手段，用于保存多线程同时操作的共享数据。

[文库](https://en.cppreference.com/w/cpp/thread/mutex)

mutex分为四种：

- std::mutex：独占的互斥量，不能递归使用，不带超时功能

- std::recursive_mutex：递归互斥量，可重入，不带超时功能

- std::timed_mutex：带超时的互斥量，不能递归

- std::recursive_timed_mutex：带超时的互斥量，可以递归使用

  std::mutex

```c++
#include <iostream>
#include <mutex>
#include <thread>

using namespace std;
std::mutex mutex_;

int main() {
    auto func1 = [](int k) {
        mutex_.lock();
        for (int i = 0; i < k; ++i) {
            cout << i << " ";
        }
        cout << endl;
        mutex_.unlock();
    };
    std::thread threads[5];
    for (int i = 0; i < 5; ++i) {
        threads[i] = std::thread(func1, 200);
    }
    for (auto& th : threads) {
        th.join();
    }
    return 0;
}
```

**std::timed_mutex**

```c++
#include <iostream>
#include <mutex>
#include <thread>
#include <chrono>

using namespace std;
std::timed_mutex timed_mutex_;

int main() {
    auto func1 = [](int k) {
        timed_mutex_.try_lock_for(std::chrono::milliseconds(200));
        for (int i = 0; i < k; ++i) {
            cout << i << " ";
        }
        cout << endl;
        timed_mutex_.unlock();
    };
    std::thread threads[5];
    for (int i = 0; i < 5; ++i) {
        threads[i] = std::thread(func1, 200);
    }
    for (auto& th : threads) {
        th.join();
    }
    return 0;
}

```

## std::lock相关

这里主要介绍两种RAII方式的锁封装，可以动态的释放锁资源，防止线程由于编码失误导致一直持有锁。

c++11主要有std::lock_guard和std::unique_lock两种方式，使用方式都类似，如下：

```text
#include <iostream>
#include <mutex>
#include <thread>
#include <chrono>

using namespace std;
std::mutex mutex_;

int main() {
    auto func1 = [](int k) {
        // std::lock_guard<std::mutex> lock(mutex_);
        std::unique_lock<std::mutex> lock(mutex_);
        for (int i = 0; i < k; ++i) {
            cout << i << " ";
        }
        cout << endl;
    };
    std::thread threads[5];
    for (int i = 0; i < 5; ++i) {
        threads[i] = std::thread(func1, 200);
    }
    for (auto& th : threads) {
        th.join();
    }
    return 0;
}
```

**std::lock_guard**

```c++
#include <mutex>

template <class Mutex>
class lock_guard;
```

`Mutex` 是互斥量的类型。

使用 `std::lock_guard` 时，只需在需要保护的代码块中创建一个 `std::lock_guard` 对象，并将需要保护的互斥量传递给它的构造函数。当 `std::lock_guard` 对象创建时，会自动锁定互斥量，当对象销毁时，会自动解锁互斥量。

**std::unique_lock**

```c++
#include <mutex>

template <class Mutex>
class unique_lock;
```

`Mutex` 是互斥量的类型。

与 `std::lock_guard` 不同，`std::unique_lock` 对象可以在构造时不锁定互斥量，并且可以在后续的代码中手动锁定或解锁。此外，`std::unique_lock` 还提供了更多的功能，如可延迟锁定、条件变量的支持等。

## std::atomic

c++11提供了原子类型std::atomic，理论上这个T可以是任意类型，但是我平时只存放整形，别的还真的没用过，整形有这种原子变量已经足够方便，就不需要使用std::mutex来保护该变量啦。看一个计数器的代码：

```c++
struct OriginCounter { // 普通的计数器
    int count;
    std::mutex mutex_;
    void add() {
        std::lock_guard<std::mutex> lock(mutex_);
        ++count;
    }

    void sub() {
        std::lock_guard<std::mutex> lock(mutex_);
        --count;
    }

    int get() {
        std::lock_guard<std::mutex> lock(mutex_);
        return count;
    }
};

struct NewCounter { // 使用原子变量的计数器
    std::atomic<int> count;
    void add() {
        ++count;
        // count.store(++count);这种方式也可以
    }

    void sub() {
        --count;
        // count.store(--count);
    }

    int get() {
        return count.load();
    }
};
```

## std::call_once

c++11提供了std::call_once来保证某一函数在多线程环境中只调用一次，它需要配合std::once_flag使用，直接看使用代码吧：

```c++
std::once_flag onceflag;

void CallOnce() {
    std::call_once(onceflag, []() {
        cout << "call once" << endl;
    });
}

int main() {
    std::thread threads[5];
    for (int i = 0; i < 5; ++i) {
        threads[i] = std::thread(CallOnce);
    }
    for (auto& th : threads) {
        th.join();
    }
    return 0;
}
```

## volatile 

volatile通常用来建立内存屏障，volatile修饰的变量，编译器对访问该变量的代码通常不再进行优化，看下面代码：

```c++
int *p = xxx;
int a = *p;
int b = *p;
```

a和b都等于p指向的值，一般编译器会对此做优化，把*p的放入寄存器，之后a,b都等于寄存器的值，但是如果把中间p地址的值改变，内存值改变了，但a,b还是从寄存器中取的值(不一定，看编译器的优化结果),这不符合需求，所以对p加上**volatile**修饰可以避免此类优化.

注意：volatile不能解决多线程安全问题，针对特种内存才需要使用volatile，它和atomic的特点如下：

- std::atomic用于多线程访问的数据，且不用互斥量，用于并发编程中
- volatile用于读写操作不可以被优化掉的内存，用于特种内存中

## std::condition_variable

条件变量是c++11引入的一种同步机制，它可以阻塞一个线程或者个线程，直到有线程通知或者超时才会唤醒正在阻塞的线程，条件变量需要和锁配合使用，这里的锁就是上面介绍的std::unique_lock。

[成员函数](https://en.cppreference.com/w/cpp/thread/condition_variable)

```c++
class CountDownLatch {
   public:
    explicit CountDownLatch(uint32_t count) : count_(count);

    void CountDown() {
        std::unique_lock<std::mutex> lock(mutex_);
        --count_;
        if (count_ == 0) {
            cv_.notify_all();
        }
    }

    void Await(uint32_t time_ms = 0) {
        std::unique_lock<std::mutex> lock(mutex_);
        while (count_ > 0) {
            if (time_ms > 0) {
                cv_.wait_for(lock, std::chrono::milliseconds(time_ms));
            } else {
                cv_.wait(lock);
            }
        }
    }

    uint32_t GetCount() const {
        std::unique_lock<std::mutex> lock(mutex_);
          return count_; 
    }

   private:
    std::condition_variable cv_;
    mutable std::mutex mutex_;
    uint32_t count_ = 0;
};
```

* `notify_one`
  * 通知一个线程等待

* `notify_all`
  * 通知所有线程等待

* `wait`
  * 阻塞当前线程，直到条件变量被唤醒

* `wait_for`
  * 阻塞当前线程，知道条件变量被唤醒或指定的超时时间后

* `wait_until`
  * 阻塞当前线程，直到条件变量被唤醒或到达指定时间点

## std::future

c++11关于异步操作提供了future相关的类，主要有std::future、std::promise和std::packaged_task，std::future比std::thread高级些，std::future作为异步结果的传输通道，通过get()可以很方便的获取线程函数的返回值，std::promise用来包装一个值，将数据和future绑定起来，而std::packaged_task则用来包装一个调用对象，将函数和future绑定起来，方便异步调用。而std::future是不可以复制的，如果需要复制放到容器中可以使用std::shared_future。

**std::promise与std::future配合使用**



```c++
#include <functional>
#include <future>
#include <iostream>
#include <thread>

using namespace std;

void func(std::future<int>& fut) {
    int x = fut.get();
    cout << "value: " << x << endl;
}

int main() {
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();
    std::thread t(func, std::ref(fut));
    prom.set_value(144);
    t.join();
    return 0;
}
```

**std::packaged_task与std::future配合使用**

```c++
#include <functional>
#include <future>
#include <iostream>
#include <thread>

using namespace std;

int func(int in) {
    return in + 1;
}

int main() {
    std::packaged_task<int(int)> task(func);
    std::future<int> fut = task.get_future();
    std::thread(std::move(task), 5).detach();
    cout << "result " << fut.get() << endl;
    return 0;
}
```

std::future用于访问异步操作的结果，而std::promise和std::packaged_task在future高一层，它们内部都有一个future，promise包装的是一个值，packaged_task包装的是一个函数，当需要获取线程中的某个值，可以使用std::promise，当需要获取线程函数返回值，可以使用std::packaged_task。

## async

async是比future，packaged_task，promise更高级的东西，它是基于任务的异步操作，通过async可以直接创建异步的任务，返回的结果会保存在future中，不需要像packaged_task和promise那么麻烦，关于线程操作应该优先使用async，看一段使用代码：

```c++
#include <functional>
#include <future>
#include <iostream>
#include <thread>

using namespace std;

int func(int in) { return in + 1; }

int main() {
    auto res = std::async(func, 5);
    // res.wait();
    cout << res.get() << endl; // 阻塞直到函数返回
    return 0;
}
```

async具体语法如下：

```text
async(std::launch::async | std::launch::deferred, func, args...);
```

第一个参数是创建策略：

- std::launch::async表示任务执行在另一线程
- std::launch::deferred表示延迟执行任务，调用get或者wait时才会执行，不会创建线程，惰性执行在当前线程。

如果不明确指定创建策略，以上两个都不是async的默认策略，而是未定义，它是一个基于任务的程序设计，内部有一个调度器(线程池)，会根据实际情况决定采用哪种策略。

若从 std::async 获得的 std::future 未被移动或绑定到引用，则在完整表达式结尾， std::future的析构函数将阻塞直至异步计算完成，实际上相当于同步操作：

```c++
std::async(std::launch::async, []{ f(); }); // 临时量的析构函数等待 f()
std::async(std::launch::async, []{ g(); }); // f() 完成前不开始
```

`注意`：关于async启动策略这里网上和各种书籍介绍的五花八门，这里会以cppreference为主。

有时候我们如果想真正执行异步操作可以对async进行封装，强制使用std::launch::async策略来调用async。

```c++
template <typename F, typename... Args>
inline auto ReallyAsync(F&& f, Args&&... params) {
    return std::async(std::launch::async, std::forward<F>(f), std::forward<Args>(params)...);
}
```

[原文章](https://zhuanlan.zhihu.com/p/137914574)

[补充](https://zhuanlan.zhihu.com/p/638684473)


---
title: 线程等待
date: 2020-01-14 18:13:30
tags: [c++, thread, condition_variable]
description: <center>线程休眠的操作、condition_variable的使用</center>
---

## 休眠等待

```c++
std::mutex mutex;
std::atomic<bool> ready{false};

void worker(int i){
    while(!ready){
        //等待
        //第一种：释放占用的核给其他线程，如果线程数多于核心数，可能没有什么效果
        std::this_thread::yield();
        //第二种：
      	//std::this_thread::sleep_for(std::chrono::seconds(1));
    }
}

int main() {
    const auto threadCount = 4;
    std::vector<std::thread> pool;  //线程池
    for(int i=0; i< threadCount; i++){
        pool.emplace_book(worker, i);//不需要触发拷贝构造和转移构造
        /*等价pool.push_back(std::thread(worker,i));
        在引入右值引用，转移构造函数，转移复制运算符之前，通常使用push_back()向容器中加入一个右值元素（临时对象）的时候，首先会调用构造函数构造这个临时对象，然后需要调用拷贝构造函数将这个临时对象放入容器中。原来的临时变量释放。这样造成的问题是临时变量申请的资源就浪费。
        */
    }
    
    std::this_thread::sleep_for(std::chrono::seconds(100));
    ready = true;
    for(auto &v: pool){
        if(v.joinable())
            v.join();
    }
}
```

## 使用condition_variable

```c++
//来自c++文档
#include <iostream>
#include <string>
#include <thread>
#include <mutex>
#include <condition_variable>
 
std::mutex m;
std::condition_variable cv;
std::string data;
bool ready = false;
bool processed = false;
 
void worker_thread()
{
    // Wait until main() sends data
    // std::lock_guard<std::mutex> lk(m)获得锁，并在析构时释放
    std::unique_lock<std::mutex> lk(m);  //获得锁，要与condition_variable配合
    cv.wait(lk, []{return ready;});  //释放锁，并在这里阻塞，当condition_variable获得通知，判断{return ready;}是否满足，如果满足，获得锁并继续执行
 
    // after the wait, we own the lock.
    std::cout << "Worker thread is processing data\n";
    data += " after processing";
 
    // Send data back to main()
    processed = true;
    std::cout << "Worker thread signals data processing completed\n";
 
    // Manual unlocking is done before notifying, to avoid waking up
    // the waiting thread only to block again (see notify_one for details)
    lk.unlock();
    cv.notify_one();
}
 
int main()
{
    std::thread worker(worker_thread);
 
    data = "Example data";
    // send data to the worker thread
    {
        std::lock_guard<std::mutex> lk(m);
        ready = true;
        std::cout << "main() signals data ready for processing\n";
    }
    cv.notify_one();
 
    // wait for the worker
    {
        std::unique_lock<std::mutex> lk(m);
        cv.wait(lk, []{return processed;});
    }
    std::cout << "Back in main(), data = " << data << '\n';
 
    worker.join();
}
```

output:

```c++
main() signals data ready for processing
Worker thread is processing data
Worker thread signals data processing completed
Back in main(), data = Example data after processing
```


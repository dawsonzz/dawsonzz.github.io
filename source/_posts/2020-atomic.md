---
title: 多线程初探
date: 2020-01-13 22:53:44
tags: [c++, thread, atomic, mutex]
description: <center>线程的创建、数据操作、线程结束的操作</center>
---

## 线程创建

### 值拷贝类型

```c++
void printAll(int a, int b, int c){
	std::cout << a << " " << b << " " << c << std::endl;
}

void testThreadInit() {
    int a = 3;
    int b = 4;
    int c = 5;
    std::thread t1([=]{printAll(a, b, c);});
    t1.join();
    std::thread t2(printAll, a, b, c);
    t2.join();
}
```

这两种创建方式都是值拷贝，但是`t2`比`t1`效率高点，`t1`多了一步从`lambda`生成的过程

### 引用类型

```c++
void add(int a, int b, int& c){
    c = a + b;
}

void testThreadInit2() {
    int a = 3;
    int b = 4;
    int c = 5;
    std::thread t3([=, &c]{add(a, b, c);});
    t3.join();
    std::thread t4(add, a, b, std::ref(c));
    t4.join();
}
```

由于`t4`这中创建是值拷贝的，需要使用`std::ref`来转化为引用

```c++
std::ref //引用
std::cref //常量引用
```

```c++
void pirntString(const std::string& info, const std::string& info2){
    std::cout<< info << " " << info2 <<std::endl;
}
void testThreadInit3() {
    std::string hello("hello");
    std::string world("world");
    std::thread t5([&]{pirntString(hello, world);});  //引用
    t5.join();
    std::thread t6(pirntString, hello, world); //值拷贝
    t6.join();
    std::thread t7(pirntString, std::cref(hello), std::cref(world)); //引用
    t7.join();
}

```

## 线程间数据操作

### 独立变量

将共享变量拆分为独立变量，这种方法具有局限性，有些变量没办法拆开，最后再累加

### 原子操作

```c++
std::atomic<int> count=0;
std::atomic_int count2=0;
```

使得多个线程访问时不会打断`count++`等操作，但是原子操作之前还是可能发生线程切换的情况。

### 锁

```C++
class Counter {
public:
    void addCount() {m_count++;}
    void lockMutex() {m_mutex.lock();}  //加锁
    void unlockMutex() {m_mutex.unlock();}  //解锁
private:
    std::atomic<int> m_count;
    std::mutex m_mutex;  //锁
}
```

#### 死锁

调用了两次加锁而没有解锁,常见情况

1.  函数写得太独立，嵌套加锁
    ```c++
   void debug(Counter& c){
       c.lockMutex();
       std::cout<<"debug"<<std::endl;
       c.unlockMutex();
   }
   
   void work(Counter& c){
       c.lockMutex();
       std::cout<<"work"<<std::endl;
       debug(c);  //这里没有解锁导致死锁  
       c.unlockMutex();
   }
   ```

2. throw异常跳过解锁

   ```c++
   void bug(){
       throw "bad";
   }
   
   void work(Counter& c){
       try{
           c.lockMutex();
           std::cout<<"work"<<std::endl;
           bug();  
           c.unlockMutex(); //解锁被跳过
       } catch(...){}
   }
   ```

   

#### 使用优化

原始手动加解锁

```c++
int count() {
    m_mutex.lock();
    auto r = m_count;
    m_mutex.unlock();
    return r;
}//一次次加锁解锁操作使得代码冗杂
```

利用类的在作用域存活周期来实现自动加解锁

```c++
template<typename T>
class Lock{
public:
    Lock(T& mutex): m_mutex(mutex) {m_mutex.lock();}
    ~Lock() {m_mutex.unlock();}
private:
    T& m_mutex;
}

int count() {
    Lock<std::mutex> lock(m_mutex);  
    return m_count;
}
```

使用STL中的std::lock_guard，使用起来更加灵活

```c++
class Counter {
public:
    void addCount() {
        std::lock_guard<std::mutex> lock(m_mutex);
        m_count++;
    }
private:
    std::atomic<int> m_count;
    std::mutex m_mutex;  //锁
}
```

lock_guard灵活的地方

```c++
struct BankAccount {
    BankAccount(int b) : Balance(b) {}
    int Balance;
    std::mutex Mutex;
};

void transferMoney(BankAccount& a, BankAccount& b, int money){
    if(&a == &b) return;
    std::lock_guard<std::mutex> lockA(a.Mutex);
    std::lock_guard<std::mutex> lockB(b.Mutex);
    if(a.Balance <= money) return;
    a.Balance -= money;
    b.Balance += money;
}

/* 死锁情况
thread1
transferMoney(a, b, 10);
thread2
transferMoney(b, a, 10);
粗暴解决方法：判断a，b的地址，先加锁地址小的变量，缺点是代码冗杂
*/


void transferMoneySTL(BankAccount& a, BankAccount& b, int money){
    if(&a == &b) return;
    std::lock(a.Mutex, b.Mutex /*....*/);  //只加锁 还需要解锁

    std::lock_guard<std::mutex> lockA(a.Mutex, std::adpot_lock);
    std::lock_guard<std::mutex> lockB(b.Mutex, std::adpot_lock);
    //std::adpot_lock 让lock_guard 知道已经加锁了，只需要析构的时候解锁
    
    if(a.Balance <= money) return;
    a.Balance -= money;
    b.Balance += money;
}
```

## 线程结束

```c++
class Obj(){
public:
    Obj(){std::cout<<"hello "<<std::endl;}
    ~Obj(){std::cout<<"world "<<std::endl;}
};

void method(){}
void detachWorker() {
    Obj obj();
    std::this_thread::sleep_for(std::chrono::seconds(1));
}
int main(){
    std::thread a(method);
    std::thread b(detachWorker);
    a.join();  //阻塞线程，主线程等待线程结束
    b.detach();  //主线程结束就结束，obj的析构函数来不及调用
    return 0;
}
```


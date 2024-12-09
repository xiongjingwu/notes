# Advanced Features

# Copy on Write (COW)

c++ string 在创建的时候如果用其他的字符串直接赋值或者构造，会与该字符串共享内存，一旦字符串内容有变化时，则新开辟空间

```cpp
#include <string>
#include <iostream>

int main() {
  std::string a = "Hello World";
  std::string b = a;
  printf("pointer of a: %p\n", a.c_str());
  printf("pointer of b: %p\n", b.c_str());
  b[0] = 'h';
  printf("pointer of a: %p\n", a.c_str());
  printf("pointer of b: %p\n", b.c_str());
  std::cout << a << std::endl;
  std::cout << b << std::endl;
}
```

# C++11

auto decltype typeid（b).name()比较

## 分支预测

```cpp
// gcc 引入的指令：__builtin_expect(EXP, N) -- 表示EXP==N的概率很大。
// 可以将最有可能执行的分支告诉编译器。一般封装为likely和unlikely宏使用

#define likely(x) __builtin_expect(!!(x), 1) //x很可能为真       
#define unlikely(x) __builtin_expect(!!(x), 0) //x很可能为假

int x = 1, y;
if(unlikely(x > 0)) //x > 0 为真的概率比较低
	if(likely(x == 1)) //x == 1 为真的概率很高
	    y = 1; 
	else 
	    y = -2;
else 
    y = -1;
```

## CAS (Compare and Swap)

CAS是一个原子指令，用于在多线程环境中实现同步。

CAS原语有三个参数，内存地址，期望值，新值。如果内存地址的值==期望值，表示该值未修改，此时可以修改成新值。否则表示修改失败，返回false，由用户决定后续操作

将存储位置的内容与给定值进行比较，当它们逐位相等，才将该存储位置的内容修改为新的给定值。整个流程为一个原子操作

乐观锁的一种实现方式。也是一种自旋锁

缺点：

1. 一直循环，开销比较大
2. 只能保证一个变量的原子操作，多个变量仍然要加锁
3. 有ABA问题

[https://www.cnblogs.com/gnivor/p/15919850.html](https://www.cnblogs.com/gnivor/p/15919850.html)

## 线程安全队列

```cpp
#include<thread>
#include<condition_variable>
#include<mutex>
#include<queue>
#include<stdio.h>

template <class T>
class ThreadSafeQueue{
    public:
        void Insert(T value);
        void Popup(T &value);
        bool Empty() const;

    private:
        mutable std::mutex mut_;
        std::queue<T> que_;
        std::condition_variable cond_;
};

template <class T>
void ThreadSafeQueue<T>::Insert(T value){
    std::lock_guard<std::mutex> lk(mut_);
    que_.push(value);
    cond_.notify_one();
}

template <class T>
void ThreadSafeQueue<T>::Popup(T &value){
    std::unique_lock<std::mutex> lk(mut_);
    cond_.wait(lk, [this]{return !que_.empty();}); // 如果lamda表达式 [this]{return !que_.empty(); 返回 true, 也就是队列非空，则上锁，继续执行下面的语句；
    value = que_.front();                          // 如果lamda表达式返回False, 也就是队列为空，则解开锁，该线程进入wait，阻塞模式，等待被唤醒
    que_.pop();
}

template <class T>
bool ThreadSafeQueue<T>::Empty() const{
    std::lock_guard<std::mutex> lk(mut_);
    return que_.empty();
}

int main(){
    ThreadSafeQueue<int> q;
    int value=1;
    std::thread t2(&ThreadSafeQueue<int>::Popup, &q, std::ref(value)); // 传引用参数的时候需要使用引用包装器std::ref
    std::thread t1(&ThreadSafeQueue<int>::Insert, &q, 10);
    printf("%d\n", value);
    while(!q.Empty());
    t1.join();
    t2.join();
    printf("%d\n", value);
    return 0;
}
```

# RTTI

```cpp
typeid(type-id)
typeid(expression)

const std::type_info& ti1 = typeid(A);
const std::type_info& ti2 = typeid(A);
 
assert(&ti1 == &ti2); // not guaranteed
assert(ti1 == ti2); // guaranteed
assert(ti1.hash_code() == ti2.hash_code()); // guaranteed
assert(std::type_index(ti1) == std::type_index(ti2)); // guaranteed
```
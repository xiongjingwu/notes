# Singleton

singleton

C++11标准中，局部静态变量的**初始化是线程安全**的，注意不是变量是线程安全，是初始化（可通过`fno-threadsafe-statics`选项关闭）。所以标准的c++11写法如下

```cpp
class Singleton {
public:
	static Singleton& Instance() {
		static Singleton instance;
		return instance;
	}
public:
	Singleton(const Singleton&) = delete;
	Singleton(Singleton&&) = delete;
	Singleton& operator=(const Singleton&) = delete;
	Singleton& operator=(Singleton&&) = delete;
private:
	Singleton() = default;
	~Singleton() = default;
};
```

C++11之前的写法

饿汉式：在main之前就初始化好了instance，不存在线程安全的问题。缺点是没有使用就已经占用内存了，还会减慢程序的启动速度。另外在不同的编译单元中初始化顺序是未定义的，`static Singleton instance;`和`static Singleton& getInstance()`二者的初始化顺序不确定，如果在初始化完成之前调用`getInstance()`方法会返回一个未定义的实例。

```cpp
class Singleton {
public:
  static Singleton& getInstance() {
      return instance;
  }
  Singleton(const Singleton&) = delete;
  Singleton& operator=(const Singleton&) = delete;
private:
  Singleton() = default;
private:
  static Singleton instance;
}
Singleton Singleton::instance;
```

懒汉式：

[https://blog.csdn.net/u011726005/article/details/82356538](https://blog.csdn.net/u011726005/article/details/82356538)

[https://zhuanlan.zhihu.com/p/382345901](https://zhuanlan.zhihu.com/p/382345901)
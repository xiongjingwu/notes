## 拷贝构造

`copy constructor`有时会被编译器隐式的调用，以下四种情况会调用：

```cpp
// 1. 显式调用copy constructor
{
  Test a("xxx");
  Test b(a);
}

// 2. 编译器优化调用copy constructor
{
  Test a("xxx");
  Test b = a;
}

// 3. 传入参数时调用copy constructor
void func(Test a) { /*do something*/ }
{
  Test a("xxx");
  func(a);
}

// 4. 函数返回对象时调用copy constructor
Test func() { return Test(); }
Test a = func();
```

函数返回对象时，一般会有编译器进行RVO

```cpp
#include <iostream>
struct C {
  C() = default;
  C(const C&) { std::cout << "A copy was made.\n"; }
};
C f() {
  return C();    /*这里是作为函数的返回值调用copy constructor*/
}
int main() {
  std::cout << "Hello World!\n";
  C obj = f();    /*这里是第二点的情况调用copy constructor*/
}
```

可能的输出结果为以下3种情况：

```cpp
// 1. ==========
Hello World!
A copy was made.
A copy was made.

// 2. ==========
Hello World!
A copy was made.

// 3. ==========
Hello World!
```

## Rule of Five/Three

如果一个类中显示的定义了一下3/5个函数中任何的一个，则应该显示的定义所有这3/5个函数。

- Desturctor (析构函数)
- Copy Constructor (拷贝构造函数)
- Copy assignment operator (拷贝赋值运算符)
- Move Constructor (移动构造函数)
- Move assignment operator (移动赋值运算符)

因为编译器默认生成的`Copy Constructor`和`Copy assignment operator`执行的是浅拷贝。如果类中有指针这类的成员变量，可能存在内存泄漏及未定义的行为。

编译器默认生成的`Move Constructor`和`Move assignment operator`不像拷贝会导致错误，但是会错失优化的可能，所以最好程序员自己写。If move semantics are missing, the compiler would normally use the less efficient copy operations wherever possible

## Rule of Zero

1. 类不应定义任何特殊函数（复制/移动构造函数/赋值和析构函数），除非它们是专用于资源管理的类。
    
    此举为了满足设计上的单一责任原则，将数据模块与功能模块在代码层面分离，降低耦合度。
    
2. 基类作为管理资源的类在被继承时，析构函数可能必须要声明为public virtual，这样的行为会破坏移动复制构造，因此，如果基类在此时的默认函数应设置为default。
    
    此举为了满足多态类在C ++核心准则中禁止复制的编码原则
    

## 编译器默认生成函数规则

```cpp
// 编译器可能生成的默认函数
Class A {
 public:
  A() ;                    // 构造函数
  ~A() ;                   // 析构函数
  A(const A&) ;            // 拷贝构造函数
  A& operator=(const A&) ; // 拷贝赋值运算符
  A(A&&) ;                 // 移动构造函数
  A& operator=(A&&) ;      // 移动赋值运算符
};
```

1. 只要指定了一个要求传参的构造函数, 就会阻止编译器生成默认构造函数。
2. 只要定义了拷贝构造函数或移动构造函数，就会阻止默认构造函数。
3. 两种复制操作是彼此独立的, 即显式声明了其中一个, 不会阻止编译器默认生成另一个。
4. 两种移动操作并不彼此独立, 即显式声明了其中一个, 就会阻止编译器默认生成另一个。
5. 一旦显式声明了复制操作, 就会阻止编译器默认生成移动操作。
6. 一旦显式声明了移动操作, 就会阻止编译器默认生成复制操作

1. 默认构造：只在用户没有定义其它类型构造函数时才会生成
2. 拷贝构造：只有用户没有定义 **移动构造函数**和 **移动赋值运算符**时，才会生成
3. 拷贝赋值：只有用户没有定义 **移动构造函数**和 **移动赋值运算符**时，才会生成
4. 析构函数：无限制
5. 移动构造：只有用户没有定义 **拷贝构造函数**, **拷贝赋值运算**, **析构函数** 和 **移动赋值运算符**时，才会生成
6. 移动赋值：只有用户没有定义 **拷贝构造函数**, **拷贝赋值运算符**, **析构函数**和 **移动构造函数**时，才会生成

# Thread

```cpp
普通函数，labmbda表达式，仿函数，类静态成员函数，类普通成员函数
// thread不能直接赋值，可以通过move赋值
void func() {}
std::thread t1(func);
std::thread t2 = t1; // error: use of deleted function ‘std::thread::thread(std::thread&)’
std::thread t2 = std::move(t1);  // ok

// 线程函数传递引用参数
void func2(std::string &s) {}
std::string test;
std::thread t(fun2, std::ref(test)); // 通过std::ref才能达到引用效果，否则还是值传递方式。高版本编译器不加ref好像直接就报错了
```

POD

[https://www.cnblogs.com/shadow-lr/p/cplusplus_pod_trivial_standard_layout.html](https://www.cnblogs.com/shadow-lr/p/cplusplus_pod_trivial_standard_layout.html)

# C++ literal type

literal 和 literal-type 是不同概念。literal 指的是变量，literal-type指的是类型，所有的类型要么是literal-type要么是non literal-type

- literal 字面量，文字量。比如10，3.14， true ，"123" 等都是literal
- literal type(字面值类型)：是指**或许**能用于常量表达式（constant expression）的类型。可以在用于编译期运算的对象

constexpr 函数的参数类型、返回类型、函数体中定义的变量的类型必须都是 literal type

可以通过`std::is_literal_type<point>::value`判断point类型是否为literal-type

A type is a *literal type* if it is:

- `void`; or
- a scalar type; or
- a reference type; or
- an array of literal type; or
- a class type (Clause 9) that has all of the following properties:
    - it has a trivial destructor,
    - it is an aggregate type (8.5.1) or has at least one `constexpr` constructor or constructor template that is not a copy or move constructor, and
    - all of its non-static data members and base classes are of non-volatile literal types

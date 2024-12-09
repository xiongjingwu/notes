
# 可调用对象

- 函数指针
- 具有 operator() 成员函数的类的对象 (注意是对象)
- 可被转换为函数指针的类对象
- 类成员函数指针
- lambda 表达式

```cpp
// 普通函数
int add(int a, int b){return a+b;}
// lambda表达式
auto mod = [](int a, int b){ return a % b;}
// 函数对象类
struct divide{
    int operator()(int denominator, int divisor){
        return denominator/divisor;
    }
};
auto sub = [](int i, int j){ return i - j; };

// 上述各种可调用对象虽然类型不同，但是共享了一种调用形式：
// int(int ,int)

// std::function就可以将上述类型保存起来
std::function<int(int, int)> a = add; 
std::function<int(int, int)> b = mod ; 
std::function<int(int, int)> c = divide();
std::function<int(int, int)> d = sub;
```

由于可调用对象定义五花八门，c++11提供了`std::function`和`std::bind`来统一可调用对象的各种操作

## std::function

- 定义：std::function<T> .   是一个类模板，是一个可调用对象包装器。T是函数类型
- 可以使用统一的方式处理函数/函数对象/函数指针，并允许保存和延迟他它们的执行
- 可以取代函数指针的使用，因为它可以延迟函数的执行，特别适合作为回调函数使用。它比普通函数指针更加的灵活和便利

## std::bind

- 是一个通用的函数适配器。接受一个可调用对象，生成一个新的可调用对象来”适应”原对象的参数列表
- 将可调用对象和参数绑定，绑定后的结果可以使用std::function保存

```cpp
// 返回一个基于f的函数对象，其参数被绑定到args上
template< class F, class... Args >
bind( F&& f, Args&&... args );
```

用法示例：

```cpp
double my_divide (double x, double y) {return x/y;}
// _1表示占位符，位于<functional>中，std::placeholders::_1；
auto fn_half = std::bind (my_divide, _1, 2);
std::cout << fn_half(10) << '\n';                        // 5

// 绑定成员函数
struct Foo {
    void print_sum(int n1, int n2) {
        std::cout << n1+n2 << '\n';
    }
    int data = 10;
};
int main()  {
    Foo foo;
    auto f = std::bind(&Foo::print_sum, &foo, 95, std::placeholders::_1);
    f(5); // 100
}

// 绑定引用参数要用std::ref
ostream & print(ostream &os, const string& s, char c) {
    os << s << c;
    return os;
}
ostringstream os;
char c = ' ';
std::bind(print, std::ref(os1), _1, c)
```

绑定类成员函数时：

- 第一个参数表示对象的成员函数的指针，第二个参数表示对象的地址
- 必须显示指定&Foo::print_sum，编译器不会将对象成员函数隐式转换成函数指针，所以必须加&
- 使用对象成员函数的指针时，必须知道该指针属于哪个对象，因此第二个参数为对象地址&foo；如果在类内部，地址用this指针

成员函数的指针的定义和使用

```cpp
#include <iostream>
struct Foo {
    int value;
    void f() { std::cout << "f(" << this->value << ")\n"; }
    void g() { std::cout << "g(" << this->value << ")\n"; }
};
void apply(Foo* foo1, Foo* foo2, void (Foo::*fun)()) {
    (foo1->*fun)();  // call fun on the object foo1
    (foo2->*fun)();  // call fun on the object foo2
}
int main() {
    Foo foo1{1};
    Foo foo2{2};
    apply(&foo1, &foo2, &Foo::f);
    apply(&foo1, &foo2, &Foo::g);
}
```

- 成员函数指针的定义：void (Foo::*fun)()，调用是传递的实参: &Foo::f
- fun为类成员函数指针，所以调用是要通过解引用的方式获取成员函数*fun,即`(foo1->*fun)();`



## Lambda 表达式

- 语法格式

`[捕获列表] (参数列表) -> 返回值类型 { 函数体 }` ，其中`[]{}`是必须的，其它可以省略。如果没有()说明没有函数参数，如果没有返回值类型，说明函数返回空

- 捕获变量方式

通过前缀`&`指定按引用捕获，或`=`指定按值捕获。值捕获的变量不能修改且值保持不变，引用捕获的变量可以修改。

如果要修改**值捕获**的变量，加上关键字**mutable**

```cpp
auto area = [=](double radius) mutable -> double {
  pi *= 2;
  return pi * radius * radius;
};
```

几种捕获方式：

```cpp
// 按值捕获i，引用捕获j
auto lbd = [i, &j](int x) -> int {};
// 按值捕获所以变量
auto lbd = [=](int x) -> int {};
// 按引用捕获所有变量
auto lbd = [&](int x) -> int {};
// 按引用捕获 j，其它所有变量按值捕获
auto lbd = [=, &j](int x) -> int {};
// 按值捕获 i，其它所有变量按引用捕获
auto lbd = [&, i](int x) -> int {};
```

# pybind11

以下所有示例默认包含：

```cpp
#include <pybind11/pybind11.h>
namespace py = pybind11;
```

## 函数绑定

```cpp
// c++ code
int add(int i, int j) {
    return i + j;
}
// python interface
PYBIND11_MODULE(example, m) {
    m.doc() = "pybind11 example plugin"; // optional module docstring
    m.def("add", &add, "A function which adds two numbers");
}
// 给函数指定参数
m.def("add", &add, "A function which adds two numbers",
      py::arg("i"), py::arg("j"));
// 更简短的声明方式
m.def("add1", &add, py::arg("i"), py::arg("j"));
using namespace pybind11::literals;
m.def("add2", &add, "i"_a, "j"_a);

// 指定参数默认值
m.def("add", &add, "A function which adds two numbers",
      py::arg("i") = 1, py::arg("j") = 2);
// 更简短的声明方式
m.def("add1", &add, py::arg("i") = 1, py::arg("j") = 2);
m.def("add2", &add, "i"_a=1, "j"_a=2);
// 后缀_a会生成一个等价于arg方法的字面量。使用这个后缀时，需要调用using namespace pybind11::literals来声明后缀所在的命名空间

// 模板函数
template <typename T>
T square(T x) {
    return x * x;
}
PYBIND11_MODULE(example, m) {
    m.doc() = "pybind11 example plugin"; // optional module docstring
    m.def("square", &square<double>);
    m.def("square", &square<float>);
    m.def("square", &square<int>);

// 函数参数为指针
void swap(int* a, int* b) {
    int temp;
    temp = *a;
    *a = *b;
    *b = temp;
}
PYBIND11_MODULE(example, m) {
    m.doc() = "pybind11 example plugin"; // optional module docstring
 
    m.def("swap", [](py::buffer a, py::buffer b) {\
        py::buffer_info a_info = a.request();
        py::buffer_info b_info = b.request();
        swap(static_cast<int*>(a_info.ptr), static_cast<int*>(b_info.ptr));
    });
}
```

`g++ -O3 -Wall -shared -std=c++11 -fPIC $(python3 -m pybind11 --includes) example.cpp -o example$(python3-config --extension-suffix)`

Note：构建的so名称，python代码中import的模块，PYBIND11_MODULE宏的第一个参数三者需要保持一致

## 导出变量

使用`attr`函数来注册需要导出到Python模块中的C++变量。内建类型和常规对象会在指定attriutes时自动转换，也可使用`py::cast`显式转换

```cpp
PYBIND11_MODULE(example, m) {
    m.attr("the_answer") = 42;
    py::object world = py::cast("World");
    m.attr("what") = world;
}
```

## 类绑定

```cpp
// c++ code
struct Pet {
    Pet(const std::string &name) : name(name) { }
    void setName(const std::string &name_) { name = name_; }
    const std::string &getName() const { return name; }

    std::string name;
};

// python interface
PYBIND11_MODULE(example, m) {
    py::class_<Pet>(m, "Pet")
        .def(py::init<const std::string &>())  // 对应构造函数
				.def_readwrite("name", &Pet::name)     // 成员变量绑定
        .def("setName", &Pet::setName)
        .def("getName", &Pet::getName);
}

// 静态成员函数使用class_::def_static来绑定
```

## 成员变量

```cpp
// def_readwrite导出公有成员变量，def_readonly导出只读成员
py::class_<Pet>(m, "Pet")
    .def(py::init<const std::string &>())
    .def_readwrite("name", &Pet::name)
    // ... remainder ...

// 如果Pet::name是一个私有成员变量，向外提供setter和getters方法
class Pet {
public:
    Pet(const std::string &name) : name(name) { }
    void setName(const std::string &name_) { name = name_; }
    const std::string &getName() const { return name; }
private:
    std::string name;
};
// 使用def_property()(只读成员使用class_::def_property_readonly())来定义并私有成员，并生成相应的setter和geter方法
py::class_<Pet>(m, "Pet")
    .def(py::init<const std::string &>())
    .def_property("name", &Pet::getName, &Pet::setName)
    // ... remainder ...

// def_readwrite_static(),def_readonly_static()，def_property_static(), class_::def_property_readonly_static()用于绑定静态变量和属性
```

## 动态属性

python 可以动态的获取新属性

> class Pet:
...    name = "Molly"
...
p = Pet()
p.name = "Charly"  # overwrite existing
p.age = 2  # dynamically add a new attribute
> 

要支持动态属性需要在`py::class_`的构造函数添加`py::dynamic_attr`标识

```cpp
py::class_<Pet>(m, "Pet", py::dynamic_attr())
    .def(py::init<>())
    .def_readwrite("name", &Pet::name);
```

Note：动态属性会带来一些运行时的开销

## 类继承

```cpp
struct Pet {
    Pet(const std::string &name) : name(name) { }
    std::string name;
};
struct Dog : Pet {
    Dog(const std::string &name) : Pet(name) { }
    std::string bark() const { return "woof!"; }
};

// python interface
py::class_<Pet>(m, "Pet")
   .def(py::init<const std::string &>())
   .def_readwrite("name", &Pet::name);

// Method 1: template parameter:
py::class_<Dog, Pet>(m, "Dog")
    .def(py::init<const std::string &>())
    .def("bark", &Dog::bark);

// Method 2: pass parent class_ object:
py::class_<Dog>(m, "Dog", pet)
    .def(py::init<const std::string &>())
    .def("bark", &Dog::bark);

// 在C++中，一个类至少有一个虚函数才会被视为多态类型。pybind11会自动识别这种多态机制
struct PolymorphicPet {
    virtual ~PolymorphicPet() = default;
};
struct PolymorphicDog : PolymorphicPet {
    std::string bark() const { return "woof!"; }
};

// Same binding code
py::class_<PolymorphicPet>(m, "PolymorphicPet");
py::class_<PolymorphicDog, PolymorphicPet>(m, "PolymorphicDog")
    .def(py::init<>())
    .def("bark", &PolymorphicDog::bark);
```

## 函数重载

```cpp
struct Pet {
    Pet(const std::string &name, int age) : name(name), age(age) { }

    void set(int age_) { age = age_; }
    void set(const std::string &name_) { name = name_; }

    std::string name;
    int age;
};

// python interface
py::class_<Pet>(m, "Pet")
   .def(py::init<const std::string &, int>())
   .def("set", static_cast<void (Pet::*)(int)>(&Pet::set), "Set the pet's age")
   .def("set", static_cast<void (Pet::*)(const std::string &)>(&Pet::set), "Set the pet's name");

// 编译器支持c++14时。py::overload_cast仅需指定函数类型，不用给出返回值类型
py::class_<Pet>(m, "Pet")
    .def("set", py::overload_cast<int>(&Pet::set), "Set the pet's age")
    .def("set", py::overload_cast<const std::string &>(&Pet::set), "Set the pet's name");

// 基于const的重载
struct Widget {
    int foo(int x, float y);
    int foo(int x, float y) const;
};

py::class_<Widget>(m, "Widget")
   .def("foo_mutable", py::overload_cast<int, float>(&Widget::foo))
   .def("foo_const",   py::overload_cast<int, float>(&Widget::foo, py::const_));
```

## 枚举&内部类型

```cpp
struct Pet {
    enum Kind {
        Dog = 0,
        Cat
    };

    struct Attributes {
        float age = 0;
    };

    Pet(const std::string &name, Kind type) : name(name), type(type) { }

    std::string name;
    Kind type;
    Attributes attr;
};

// python interface
py::class_<Pet> pet(m, "Pet");  // Pet 类示例，供下面使用，保证Kind和Attributes在Pet的作用域中

pet.def(py::init<const std::string &, Pet::Kind>())
    .def_readwrite("name", &Pet::name)
    .def_readwrite("type", &Pet::type)
    .def_readwrite("attr", &Pet::attr);

py::enum_<Pet::Kind>(pet, "Kind")
    .value("Dog", Pet::Kind::Dog)
    .value("Cat", Pet::Kind::Cat)
    .export_values();

py::class_<Pet::Attributes> attributes(pet, "Attributes")
    .def(py::init<>())
    .def_readwrite("age", &Pet::Attributes::age);

// 给enum_的构造函数增加py::arithmetic()标识，pybind11将创建支持算术/位运算的枚举
py::enum_<Pet::Kind>(pet, "Kind", py::arithmetic())
   ...
```

为确保嵌套类型`Kind`和`Attributes`在`Pet`的作用域中创建，必须向`enum_`和`class_`的构造函数提供`Pet` `class_`实例。`enum_::export_values()`用来导出枚举项到父作用域

## 返回值策略

明确 Python 侧接管返回值并负责释放资源，还是由C++侧来处理。默认策略为 `return_value_policy::automatic`

| 返回值策略 | 描述 |
| --- | --- |
| `return_value_policy::take_ownership` | 引用现有对象（不创建一个新对象），并获取所有权。在引用计数为0时，Pyhton将调用析构函数和delete操作销毁对象。 |
| `return_value_policy::copy` | 拷贝返回值，这样Python将拥有拷贝的对象。该策略相对来说比较安全，因为两个实例的生命周期是分离的。 |
| `return_value_policy::move` | 使用`std::move`来移动返回值的内容到新实例，新实例的所有权在Python。该策略相对来说比较安全，因为两个实例的生命周期是分离的。 |
| `return_value_policy::reference` | 引用现有对象，但不拥有所有权。C++侧负责该对象的生命周期管理，并在对象不再被使用时负责析构它。注意：当Python侧还在使用引用的对象时，C++侧删除对象将导致未定义行为。 |
| `return_value_policy::reference_internal` | 返回值的生命周期与父对象的生命周期相绑定，即被调用函数或属性的`this`或`self`对象。这种策略与reference策略类似，但附加了`keep_alive<0, 1>`调用策略保证返回值还被Python引用时，其父对象就不会被垃圾回收掉。这是由`def_property`、`def_readwrite`创建的属性getter方法的默认返回值策略。 |
| `return_value_policy::automatic` | 当返回值是指针时，该策略使用`return_value_policy::take_ownership`。反之对左值和右值引用使用`return_value_policy::copy`。是`py::class_`封装类型的默认策略。 |
| `return_value_policy::automatic_reference` | 和上面一样，但是当返回值是指针时，使用`return_value_policy::reference`策略。这是在C++代码手动调用Python函数和使用`pybind11/stl.h`中的casters时的默认转换策略。你可能不需要显式地使用该策略。 |

返回值策略也可以用于属性

```cpp
class_<MyClass>(m, "MyClass")
    .def_property("data", &MyClass::getData, &MyClass::setData,
                  py::return_value_policy::copy);
```

## **以Python对象作为参数**

```cpp
void print_dict(const py::dict& dict) {
    /* Easily interact with Python types */
    for (auto item : dict)
        std::cout << "key=" << std::string(py::str(item.first)) << ", "
                  << "value=" << std::string(py::str(item.second)) << std::endl;
}

// it can be exported as follow:
m.def("print_dict", &print_dict);

void generic(py::args args, const py::kwargs& kwargs) {
    /// .. do something with args
    if (kwargs)
        /// .. do something with kwargs
}

/// Binding code
m.def("generic", &generic);
```

`py::args`继承自`py::tuple`，`py::kwargs`继承自`py::dict`。

Ref：[https://github.com/charlotteLive/pybind11-Chinese-docs/blob/main/04. 首次尝试.md](https://github.com/charlotteLive/pybind11-Chinese-docs/blob/main/04.%20%E9%A6%96%E6%AC%A1%E5%B0%9D%E8%AF%95.md)
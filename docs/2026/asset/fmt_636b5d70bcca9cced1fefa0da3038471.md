
C++20有很多新的特性，其中最重要的四个是概念、范围、协程和模块。概念可以让我们定义泛型函数或类的约束条件，范围可以让我们更方便地操作容器和迭代器，协程可以让我们编写异步代码，模块可以让我们更高效地组织代码。除此之外，C++20还有一些其他的新特性，比如三向比较运算符、指定初始化、日历和时区功能等。

### 概念

概念是一种用来约束模板类型的语法糖。我们可以用`concept`关键字来定义一个概念，然后用`requires`关键字来指定一个模板参数必须满足某个概念。例如，我们可以定义一个`Integral`概念，表示一个类型必须是整数类型

```
template<typename T>
concept Integral = std::is_integral_v<T>;
// 然后我们可以用这个概念来约束一个函数模板的参数类型
template<Integral T>
T add(T a, T b) {
    return a + b;
}
```

这样，如果我们传入非整数类型的参数，就会在编译时报错。

概念可以自定义，使用`requires`关键字

```
template<typename T>
concept Sortable = requires(T a) {
    { std::sort(a.begin(), a.end()) } -> std::same_as<void>;
};
// 这个概念要求T类型有begin()和end()方法，并且可以用std::sort函数进行排序
```

标准库中提供了上百种常用的概念，放在和等头文件中。比较常用的一些有：`std::same_as`, `std::derived_from`, `std::convertible_to`, `std::floating_point`等

```
#include <concepts>

template<std::integral T>
T add(T a, T b) {
    return a + b;
}
```

### 范围

范围是C++20加入的一个重要的库功能，它提供了描述范围和对范围的操作的统一接口。一个范围是可以循环访问的任何东西，比如一个容器或者一个数组。我们可以用`begin()`和`end()`函数来获取一个范围的起始和终止位置。我们也可以用基于范围的`for`语句来遍历一个范围中的所有元素。例如，我们可以这样打印一个`vector`中的所有元素：

```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    for (auto x : v) {
        std::cout << x << " ";
    }
}
```

自定义的类型，满足`range`概念，都可以使用范围的特性。即它可以用`begin()`和`end()`函数来获取其起始和终止位置。这两个函数返回的对象必须是迭代器或者哨兵。迭代器是可以用`++`和`*`操作符来遍历元素的对象，哨兵是可以用`==`操作符来判断是否到达范围的末尾的对象

```
#include <ranges>
#include <iostream>

class IntRange {
public:
    IntRange(int a, int b) : a_(a), b_(b) {}
    // 迭代器
    class Iterator {
    public:
        Iterator(int x) : x_(x) {}
        int operator*() const { return x_; }
        Iterator& operator++() { ++x_; return *this; }
        bool operator==(const Iterator& other) const { return x_ == other.x_; }
        bool operator!=(const Iterator& other) const { return !(*this == other); }
    private:
        int x_;
    };
    // 哨兵
    class Sentinel {
    public:
        Sentinel(int y) : y_(y) {}
        bool operator==(const Iterator& iter) const { return *iter == y_; }
        bool operator!=(const Iterator& iter) const { return !(*this == iter); }
    private:
        int y_;
    };

    // begin()和end()函数
    Iterator begin() const { return Iterator(a_); }
    Sentinel end() const { return Sentinel(b_); }

private:
    int a_, b_;
};

int main() {
    IntRange r(1,5);

    for (auto x : r) {
        std::cout << x << " ";
    }
}
```

### 协程

协程是一种可以在执行过程中被挂起和恢复的函数。它可以用来实现异步编程，提高性能和并发度。

C++20中引入了三个新的关键字，`co_await`，`co_yield`和`co_return`，用来标记一个函数是协程。这些关键字只是语法糖，编译器会将协程的上下文打包成一个对象，并让未执行完的协程先返回给调用者。要实现一个C++20协程，还需要提供两个鸭子类型，`promise type`和`awaiter type`，分别用来管理协程的生命周期和等待机制。

例如，我们可以实现一个简单的生成器协程，它每次产生一个整数：

```
#include <coroutine>
#include <iostream>

// promise type
struct Generator {
    struct promise_type {
        int current_value;
        std::suspend_always yield_value(int value) {
            this->current_value = value;
            return {};
        }
        std::suspend_always initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        Generator get_return_object() {
            return Generator{std::coroutine_handle<promise_type>::from_promise(*this)};
        }
        void unhandled_exception() {}
    };

    // awaiter type
    bool move_next() {
        p.resume();
        return !p.done();
    }
    int current_value() { return p.promise().current_value; }

private:
    std::coroutine_handle<promise_type> p;
};

// 协程函数
Generator generator(int start = 0) {
    int i = start;
    while (true) {
        co_yield i++;
    }
}

int main() {
    auto g = generator(1);

    for (int i = 0; i < 10; ++i) {
        g.move_next();
        std::cout << g.current_value() << " ";
    }
}
```

使用协程实现异步网络编程的主要优点是可以用同步的语法写出异步的代码，提高代码的可读性和可维护性1。要使用协程实现异步网络编程，需要以下几个步骤：

- 使用标准库中提供的`std::jthread`或`std::thread`创建一个或多个工作线程，用来执行协程任务。
- 使用标准库中提供的`std::coroutine_handle`或自定义的协程句柄类型，管理协程的生命周期和调度。
- 使用标准库中提供的`std::future`或自定义的`awaiter`类型，等待异步操作完成并获取结果。
- 使用标准库中提供的`std::sync_wait`或自定义的同步等待函数，等待所有协程任务完成后退出程序。

例如，我们可以使用一个简单的网络框架ZED3，它提供了一些基本的异步IO操作，并封装了协程句柄和`awaiter`类型。我们可以用以下代码实现一个简单的回显服务器：

```
#include <zed/net.hpp>
#include <iostream>

using namespace zed;

int main() {
    // 创建一个io_context对象
    io_context ctx;
    // 创建一个工作线程
    std::jthread th([&ctx]() { ctx.run(); });
    // 创建一个tcp服务器
    tcp_server server(ctx);
    // 绑定端口
    server.bind(8080);
    // 开始监听
    server.listen();

    while (true) {
        try {
            // 接受连接，并返回一个tcp_socket对象
            auto socket = co_await server.accept();
            std::cout << "New connection from " << socket.remote_endpoint() << "\n";
            while (true) {
                // 接收数据，并返回接收到的字节数
                auto n = co_await socket.recv();
                if (n == 0) break; // 连接断开
                std::cout << "Received " << n << " bytes\n";
                // 发送数据，并返回发送出去的字节数
                auto m = co_await socket.send(n);
                std::cout << "Sent " << m << " bytes\n";
            }
            std::cout << "Connection closed\n";
        } catch (const std::exception& e) {
            std::cerr << e.what() << "\n";
        }
    }
}
```

### 模块

C++20模块是一种新的代码组织和重用的方式，它可以替代传统的头文件和翻译单元。`#include`多个头文件时编译很慢，使用`module`相当于直接调用编译好的二进制文件，这个二进制文件中描述了这个`module`导出的函数、类、模板等。模块可以提高编译速度，避免宏污染，隐藏实现细节，简化依赖关系等优点。要使用模块，需要以下几个步骤：

- 在源文件中使用`module`关键字声明一个模块，并指定模块名。
- 在源文件中使用`export`关键字导出需要对外提供的符号。
- 在其他源文件中使用`import`关键字导入需要使用的模块。
- 使用支持模块的编译器编译源文件，并生成相应的模块接口文件和目标文件。

例如，我们可以用以下代码定义一个名为`hello`的模块：

```
// hello.cppm
module hello; // 声明一个名为hello的模块
export void say_hello(); // 导出一个名为say_hello的函数
void say_hello() {
    std::cout << "Hello, world!\n";
}
```

然后我们可以在另一个源文件中导入并使用这个模块：

```
// main.cpp
import hello; // 导入hello模块
int main() {
    say_hello(); // 调用say_hello函数
}
```

子模块是一种在逻辑上划分模块的方法，它可以让用户选择性地导入模块的一部分或全部内容。子模块的命名规则中允许点存在于模块名字当中，但点并不代表语法上的从属关系，而只是帮助程序员理解模块间的逻辑关系。

例如，我们可以用以下代码定义一个名为`hello.sub_a`的子模块：

```
// hello.sub_a.cppm
export module hello.sub_a; // 声明一个名为hello.sub_a的子模块
export void say_hello_sub_a(); // 导出一个名为say_hello_sub_a的函数
void say_hello_sub_a() {
    std::cout << "Hello, sub a!\n";
}
```

然后我们可以在另一个源文件中定义一个名为`hello.sub_b`的子模块：

```
// hello.sub_b.cppm
export module hello.sub_b; // 声明一个名为hello.sub_b的子模块
export void say_hello_sub_b(); // 导出一个名为say_hello_sub_b的函数
void say_hello_sub_b() {
    std::cout << "Hello, sub b!\n";
}
```

最后我们可以在另一个源文件中定义一个名为`hello`的父模块，它导出了两个子模块：

```
// hello.cppm
export module hello; // 声明一个名为hello的父模块
export import hello.sub_a; // 导出并导入hello.sub_a子模块
export import hello.sub_b; // 导出并导入hello.sub_b子模块
```

这样，用户就可以根据需要导入不同的子模块或父模块：

```
// main.cpp
import hello; // 导入hello父模块，相当于同时导入了两个子模块
int main() {
    say_hello_sub_a(); // 调用say_hello_sub_a函数
    say_hello_sub_b(); // 调用say_hello_sub_b函数
}
```

命名空间冲突是指不同的模块或源文件中定义了相同的名称，导致编译器无法区分它们的含义。C++20 模块提供了一些方法来避免或解决命名空间冲突：

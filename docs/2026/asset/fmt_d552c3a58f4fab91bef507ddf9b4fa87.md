
- 使用不同的模块名字来区分不同的模块，例如 `hello.sub_a` 和 `hello.sub_b` 就是两个不同的模块，即使它们都定义了 `say_hello` 函数，也不会发生冲突。

- 使用限定名字来指定模块中的名称，例如 `hello.sub_a::say_hello` 和 `hello.sub_b::say_hello` 就可以明确地区分两个模块中的函数。

- 使用 `using` 声明或 `using` 指令来引入需要的名称，但要注意避免引入重复或冲突的名称。例如：

```
// main.cpp
import hello;
// 导入hello父模块
using hello.sub_a::say_hello;
// 引入hello.sub_a中的say_hello函数
int main() {
 say_hello();
// 调用hello.sub_a中的say_hello函数
 hello.sub_b::say_hello();
// 调用hello.sub_b中的say_hello函数
}
```

- 使用 `export` 关键字来控制哪些名称被导出到其他模块或源文件，以减少暴露给外部的名称。例如：

```
// math.cppm
export module math;
// 声明一个名为math的模块
namespace detail {
// 定义一个未导出的命名空间detail
 int add(int x, int y) {
 return x + y; }
// 定义一个未导出的函数add
}
export int sum(int x, int y) {
 return detail::add(x, y); }
// 定义并导出一个函数sum，它调用了detail命名空间中的add函数
// main.cpp
import math;
// 导入math模块
int main() {
 int s = math::sum(1, 2);
// 调用math模块中导出的sum函数
 int a = math::detail::add(1, 2);
// 错误：math模块没有导出detail命名空间或add函数
}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)

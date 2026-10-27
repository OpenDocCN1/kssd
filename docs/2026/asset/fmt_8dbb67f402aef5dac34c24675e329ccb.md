
# [原创]【编译原理】文法解析算法以及左递归消除算法

## `CFG.h`

```
#pragma once
#include <string>
#include <set>
#include <vector>
#include <iostream>
#include <memory>
namespace CFG {
enum SYMBOLIC\_TYPE
{
SYMBOLIC\_TYPE\_TERMINAL = 0, // 终结符号
SYMBOLIC\_TYPE\_N\_TERMINAL, // 非终结符号
SYMBOLIC\_TYPE\_UNKNOW // 未知类型
};
// 文法符号
class GrammarSymbolic
{
public:
GrammarSymbolic(const std::string &name, SYMBOLIC\_TYPE type) : name\_(name), type\_(type) { }
public:
std::string Name()
{
return name\_;
}
SYMBOLIC\_TYPE Type()
{
return type\_;
}
private:
std::string name\_; // 符号名称
SYMBOLIC\_TYPE type\_; // 符号类型
};
// 产生式体
typedef std::vector<GrammarSymbolic \*> ProductionBody;
// 产生式类
class Production
{
public:
Production(std::string header) : header\_(header) {}
public:
std::string Header() const
{
return header\_;
}
std::vector<ProductionBody> &Body() {
return bodys\_;
}
public:
void AppendBody(ProductionBody body);
Production &operator +=(const Production &rhs)
{
// 产生式头部相同才可以合并
if (header\_ != rhs.header\_)
{
return \*this;
}
for (auto body : rhs.bodys\_)
{
this->bodys\_.push\_back(body);
}
return \*this;
}
private:
std::string header\_; // 产生式头
std::vector<ProductionBody> bodys\_; // 产生式体
};
// 文法类
class CFG
{
friend std::ostream &operator <<(std::ostream &out, CFG &cfg);
public:
CFG(const std::set<std::string> n\_terminal\_set, // 非终结符的集合
const std::set<std::string> terminal\_set, // 终结符的集合
const std::string &productions, // 产生式列表
const std::string start // 文法的开始符号
);
~CFG();
public:
/// 是否是终结符号
bool IsTerminal(const std::string &symbolic);
/// 是否是非终结符号
bool IsNTerminal(const std::string &symbolic);
/// 是否是文法符号
bool IsCFGSymbolic(const std::string &symbolic);
/// 获取文法符号类型
SYMBOLIC\_TYPE GetSymbolicType(const std::string &symbolic);
public:
/// 移除左递归
CFG &RemoveRecursive();
/// 提取左公因子
CFG &TakeLeft();
private:
void AppendProduct(std::shared\_ptr<Production> product);
private:
std::set<std::string> n\_terminal\_set\_; // 非中介符号集合
std::set<std::string> terminal\_set\_; // 终结符号集合
std::string start\_; // 文法开始符号
std::vector<std::shared\_ptr<Production>> productions\_;// 产生式列表
};
}
```

## `CFG.cpp`

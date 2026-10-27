
```cpp
#include "Parser.h"
Parser::Parser(CFG::CFG *cfg, Lex *lex) : cfg_(cfg), lex_(lex)
{
}
Parser::~Parser()
{
}
// 获取FIRST集合, 文法必须是消除左递归后的文法 可以处理绝大多数文法，但不是所有文法，有一些特例。
std::set<std::string> Parser::First(CFG::GrammarSymbolic * symbolic)
{
std::set<std::string> first_set;
// 规则一 如果符号是一个终结符号，那么他的FIRST集合就是它自身
if (symbolic->Type() == CFG::SYMBOLIC_TYPE_TERMINAL)
{
first_set.insert(symbolic->Name());
return first_set;
}
// 规则二 如果一个符号是一个非终结符号(A)
// (1) A -> XYZ 如果 X 可以推导出nil 那么就去查看Y是否可以推导出nil
// 如果 Y 推导不出nil，那么把Y的First集合加入到A的First集合
// 如果 Y 不能推导出nil，那么继续推导 Z 是否可以推导出nil,依次类推
// (2) A -> XYZ 如果XYZ 都可以推导出 nil, 那么说明A这个产生式有可能就是nil， 这个时候我们就把nil加入到FIRST(A)中
std::string current_symbolic_name = symbolic->Name();
auto productions = cfg_->Productions();
for (size_t i = 0; i < productions.size(); i++)
{
// 不匹配的产生式
if (productions[i]->Header() != current_symbolic_name)
{
continue;
}
// 一个非终结符号有一个或者多个产生式体 要一一求出他的FIRST集
for (size_t k = 0; k < productions[i]->Body().size(); k++)
{
auto body = productions[i]->Body()[k];
bool has_nil = false;
// 推导这个产生式体的所有FIRST集合
for (size_t t = 0; t < body.size(); t++)
{
has_nil = false;
auto fset = First(body[t]);
for (auto terminal_sym : fset)
{
if (terminal_sym == "ε") // 查看集合中是否包含空
{
has_nil = true;
continue;
}
// 插入到当前符号
first_set.insert(fset.begin(), fset.end());
}
// 如果当前符号不为空的话 则不需要继续推导产生式下面的符号
if (has_nil == false)
{
break;
}
}
// 如果说到最后一个符号都可以推导出ε, 那么这个产生式就 可以为空
if (has_nil == true)
{
first_set.insert("ε");
}
}
break;
}
return first_set;
}
std::set<std::string> Parser::follow(std::string A)
{
// 一个文法符号的FOLLOW集就是 可能出现在这个文法符号后面的终结符
// 比如 S->ABaD, 那么FOLLOW(B)的值就是a。
// FOLLOW(A)的值包含了FIRST(B)中除了ε以外的所有终结符,如果First(B)包含空的话。说明跟在B后面的终结符号就可以跟在A后面，这时我们要把FOLLOW(B)的值也添加到FOLLOW(A)中
// 因为D是文法符号S的最右符号，那么所有跟在S后面的符号必定跟在D后面。所以FOLLOW(S)所有的值都在FOLLOW(D)中
// 以下是书中的总结
// 不断应用下面这两个规则，直到没有新的FOLLOW集 被加入
// 规则一: FOLLOW(S)中加入$, S是文法开始符号
// 规则二: A->CBY FOLLOW(B) 就是FIRST(Y)
// 规则三: A->CB 或者 A->CBZ(Z可以推导出ε) 所有FOLLOW(A)的符号都在FOLLOW(B)
std::set<std::string> follow_set;
// 如果是文法的开始符号，那么加入$
if (cfg_->StartSymbolic() == A)
{
follow_set.insert("$");
}
// 搜索所有包含 A 符号的产生式
auto productions = cfg_->Productions();
for (size_t i = 0; i < productions.size(); i++)
{
// 获取所有的产生式
for (size_t t = 0; t < productions[i]->Body().size(); t++)
{
auto body = productions[i]->Body()[t];
for (size_t k = 0; k < body.size(); k++)
{
if (A != body[k]->Name())
{
continue;
}
// 第一种情况 符号在产生式的最末尾, 那么我们就要去获取产生式头部的FOLLOW集
if (k == body.size() - 1)
{
// 没有被FOLLOW过, 避免陷入死循环
if (follow_record_.end() == std::find(follow_record_.begin(), follow_record_.end(), productions[i]->Header()))
{
follow_record_.insert(productions[i]->Header());
auto fset = follow(productions[i]->Header());
follow_set.insert(fset.begin(), fset.end());
}
continue;
}
// 不是在最后一个符号 我们获取文法符号的下一个符号的First集合
auto fset = First(body[k + 1]);
bool has_nil = false;
for (auto terminal_symbolic : fset)
{
if (terminal_symbolic == "ε")
{
has_nil = true;
continue;
}
follow_set.insert(terminal_symbolic);
}
// 如果下一个First集合包含ε
if (has_nil)
{
follow_record_.insert(body[k + 1]->Name());
auto fset = follow(body[k + 1]->Name());
follow_set.insert(fset.begin(), fset.end());
}
}
}
}
return follow_set;
}
std::set<std::string> Parser::Follow(std::string A)
{
follow_record_.clear();
follow_record_.insert(A);
return follow(A);
}
```

`LL1.h`

```cpp
#pragma once
#include "Parser.h"
#include <map>
struct ASTNode;
// 抽象语法树
struct AST
{
ASTNode *root;
};
struct ASTNode
{
CFG::GrammarSymbolic *symbolic; // 文法符号
std::vector<ASTNode *> nodes; // 子节点
};
// LL(1) 语法分析
class LL1 : Parser
{
public:
LL1(CFG::CFG *cfg, Lex *lex);
~LL1();
private:
void InitParser();
public:
/// 语法分析
AST Parse();
private:
// LL(1) 预测分析表
std::map<std::string, std::map<std::string, CFG::ProductionBody>> ll1_table_;
};
```

`LL1.cpp`

```cpp
#include "LL1.h"
#include <stack>
LL1::LL1(CFG::CFG *cfg, Lex *lex) : Parser(cfg, lex)
{
// 文法要移除左递归和提取左公因子
cfg_->RemoveRecursive();
cfg_->TakeLeft();
InitParser();
}
LL1::~LL1()
{
}
void LL1::InitParser()
{
// 生成预测分析表的目的就是让 LL(1) 文法， 向前看一个符号就可以确定使用非终结符的哪个产生式。
// 比如: A->cBD | bFA
// 当我们开始推导非终结符 A 的时候, 我们向前看输入中的一个终结符号,
// 如果终结符号是 c 我们就选择 A->cBD产生式来推导非终结符A。 如果终结符号是b那么我们就选择 A -> bFA来推导非终结符A。
// 预测分析表有两个规则
// 规则一: 产生式 A->B, 对于First(B)中的所有终结符号a, 把A->B这个产生式加入到 M[A, a]中。
// 规则二: 产生式 A->B, 如果First(B)中包含nil串, 那么对Follow(A)的所有终结符号b, 把A->B这个产生式加入到 M[A, b]中。
// 比如产生式:
// A->BC
// F->GAc
// B->a | ε
// C->f | ε
// 首先我们获取 FIRST(BC)的所有终结符号集合 {a, f, ε}, 我们把A->B 添加到M[A, a], M[A, f]。
// 其次我们发现 FIRST(BC)中是包含ε的， 那么我们就获取FOLLOW(A)的值. FOLLOW(A) => {c, $}, 那么我们就把 A->B 也添加到 M[A, c], M[A, $]中。
auto products = cfg_->Productions();
for (size_t i = 0; i < products.size(); i++)
{
auto pro = products[i]->Body();
// 获取所有的产生式右部
for (size_t k = 0; k < pro.size(); k++)
{
// 对于某一条产生式右部的所有符号
// 比如 A -> BCD
// 从左往右的顺序是 B-C-D
// 首先推导 FIRST(B) 如果FIRST(B)不能推导出空串那么就把 FIRST(B)中的所有终结符a。 把当前处理的产生式加入到 M[A, a]
// FIRST(B) 如果可以推导出空串, 那么意味着 B 这个符号是可以不存在的，那么我们就要查看 FIRST(C) 集合。依次循环这个步骤。
// -------------------
// 如果说 FIRST(B) FIRST(C) FIRST(D) 都包含空串那么就说明了 整个产生式都是可以推导出空串的。
// 比如说 B -> b | ε
// C -> c | ε
// D -> d | ε
// BCD 都可以为空, 那么 A 就可以为空了。这个时候我们就要获取 FOLLOW(A)的值来确定应用当前的产生式
bool has_nil = false;
for (size_t t = 0; t < pro[k].size(); t++)
{
auto first_set = First(pro[k][t]);
has_nil = false;
for (auto terminal_symbolic : first_set)
{
if (terminal_symbolic == "ε")
{
has_nil = true;
continue;
}
ll1_table_[products[i]->Header()][terminal_symbolic] = pro[k];
}
// 当前符号不能推导出空串那么就结束
if (false == has_nil)
{
break;
}
}
// 产生式右部所有的符号都可以为空
if (has_nil)
{
auto follow_set = Follow(products[i]->Header());
for (auto terminal_symbolic : follow_set)
{
ll1_table_[products[i]->Header()][terminal_symbolic] = pro[k];
}
}
}
}
}
AST LL1::Parse()
{
AST ast;
ast.root = new ASTNode;
auto cur_node = ast.root;
cur_node->symbolic = new CFG::GrammarSymbolic(cfg_->StartSymbolic(), CFG::SYMBOLIC_TYPE_N_TERMINAL);
std::stack<std::string> ll1_stack;
std::vector<std::vector<void *>> ast_stack; // 语法分析栈(递归栈)
ll1_stack.push("$"); // 结束符号
ll1_stack.push(cfg_->StartSymbolic()); // 文法开始符号
// 进入下一层语法栈
ast_stack.push_back(std::vector<void *>());
// 栈中的两个局部变量
ast_stack.back().push_back(0);
ast_stack.back().push_back(cur_node);
size_t stack_deep = 0;
auto X = ll1_stack.top();
auto input_token = lex_->FetchNext();
if (NULL == input_token)
{
throw std::runtime_error("词法分析失败");
}
while (X != "$")
{
if (cfg_->IsTerminal(X))
{
if (X == input_token->Name())
{
ll1_stack.pop(); // 当前符号出栈
input_token = lex_->FetchNext(); // 获取下一个输入符号
if (NULL == input_token)
{
throw std::runtime_error("词法分析失败");
}
}
else if (X == "ε")
{
ll1_stack.pop(); // 当前符号出栈
}
else
{
throw std::runtime_error("语法分析错误");
}
do
{
if (ast_stack.size() <= 1)
{
break;
}
// 索引右移动
ast_stack.back()[0] = (void *)((int)ast_stack.back()[0] + 1);
int idx = (int)ast_stack.back()[0];
int parent_idx = (int)ast_stack[ast_stack.size() - 2][0];
ASTNode *node = (ASTNode *)ast_stack[ast_stack.size() - 2][parent_idx + 1];
if (node->nodes.size() == idx)
{
ast_stack.pop_back(); // 类似于返回
continue;
}
break;
} while (true);
}
else if (cfg_->IsNTerminal(X)) // 如果是非终结符号
{
auto iter = ll1_table_[X].find(input_token->Name());
if (iter == ll1_table_[X].end())
{
throw std::runtime_error("语法分析错误");
}
ll1_stack.pop(); // 弹出当前符号
// 对于当前非终结符 应用的产生式体
auto product_body = ll1_table_[X][input_token->Name()];
for (int i = product_body.size() - 1; i >= 0; i--)
{
ll1_stack.push(product_body[i]->Name());
}
int idx = (int)ast_stack.back()[0];
ASTNode *node = (ASTNode *)ast_stack.back()[idx + 1];
// 进入下一层语法栈
ast_stack.push_back(std::vector<void *>());
// 栈中的两个局部变量
ast_stack.back().push_back(0);
for (size_t i = 0; i < product_body.size(); i++)
{
auto new_node = new ASTNode;
new_node->symbolic = product_body[i];
node->nodes.push_back(new_node);
ast_stack.back().push_back(new_node);
}
stack_deep++; // 栈深度添加
}
X = ll1_stack.top(); // 重新设置X的值
}
return ast;
}
```

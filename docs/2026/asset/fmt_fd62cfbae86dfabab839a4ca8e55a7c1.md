
# [原创]基于 ida 的反汇编转换 Obj 的可行性 笔记(1)

以前有种工具可以叫 `exe2c`, 听过可以把一个可执行文件转换成 c 代码，没有用过。

但是其中显示出的令人振奋的功能，其中很多问题一直想去了解，但是没有机会。

我最想了解的就是 `exe2c` 中的是他的"智能"的"反汇编引擎", 这个反汇编引擎要解决的主要问题：

1、如果有效的区分数据和指令

在现在的编译器中，常常会把全局性常量（比如跳转表）放在代码节中，我在做 `obj2asm` 时已经遇到，

但是我的解决办法全靠编译器在符号类型中留下的信息得以解决，对于已经剥离符号信息的可执行文件，

这可第一个要解决的问题。

2、在代码中解析编译器构造的函数例程

其中包括两个部分:

1、库函数识别

2、未知函数预测, 和所在范围的确定

其中 1 比较好解决，而 2 对与现代的高级语言编译器大多数的函数，使用简单的方法都可以被找出。

其中的主要干扰因素是，编译器对待结构化异常处理和内联优化有的时候会相当的“恶心”。

## 附

简单说说常见的数据和指令区分技术，还有就是未知函数预测

我们首先要把起始点（一般都是 `oep`）放到 “嫌疑函数" 表中,

然后进入一个大循环中。下面用随手写伪码简单描述一下：

```
// suspect_fun_lst; 嫌疑函数 表
// suspect_dat_lst; 嫌疑数据 表
// suspect_lab_lst; 嫌疑标号 表

for ( int i = 0; i < suspect_fun_lst.size(); i++ )
{
  instr_t instr; // 指令信息
  jmp_list jmplst;
  long p, bad;
  funinfo_t& fun = suspect_fun_lst.front();
  fun.size = 0;
  p = fun.base;
  bad = 0;
  while ( bad < 10 ) // 进行函数长度估测，和有效性判断
  {
   if ( !disam(p, &instr) ) // 进行反汇编
    break;
   // 检查这条指令是否罕见，如果是 bad 加 1, 超过一定范围这个函数的反汇编将结束
   is_rarity_instr(instr) && bad++;
   if ( instr.is_call_imm32 ) // call _fun
   {
    if ( !in_image(instr.call_target) ) // 检查目标地址是否在映像中
    {
     bad = 0x1000;
     break;
    }
    funinfo_t* xfun = find_fun(instr.call_target); // 查找表中是否已经存在
    if ( xfun == NULL )
    {
     xfun = new funinfot_t;
     xfun.base = instr.call_target; // 函数地址
     xfun.size = -1; // 大小
     xfun.nref = 0; // 被引用次数
     suspect_fun_lst.push_back(xfun);
    }
    xfun.nref++;
   }
   else if ( instr.opnum_is_mem32 )
   {
    if ( !in_image(instr.op_mem32) )
    {
     bad = 0x1001;
     break;
    }
    datinfo_t* xdat = find_dat(instr.op_mem32);
    if ( xdat = NULL )
    {
     xdat = new datinfo_t;
     xdat.addr = instr.op_mem32; // 数据地址
     xdat.size = instr.op_size; // 操作数大小
     xdat.nref = 0;
     suspect_dat_lst.push_back(instr.op_addr);
    }
    xdat.nref++;
   }
   else if ( instr.opnum_is_imm32 && in_image(instr.op_imm32) ) // mov eax, offset _i or mov eax, offset _callback
   {
    labinfo_t* xlab = find_lab(instr.op_imm32);
    if ( xlab == NULL )
    {
     xlab = new labinfo_t;
     xlab.addr = instr.op_imm32;
     xlab.nref = 0;
     suspect_lab_lst.push_back(instr.op_imm32);
    }
    xlab.nref++;
   }
   if ( instr.is_jmp_imm )
   {
    if ( jmplist.is_exist(instr.jmp_target) )
    {
     jmplist.push_back(instr.jmp_target);
    }
   }
   for ( long a = jmplist.begin(); a != jmplist.end(); a++ ) // 把
   {
    if ( a < p )
    {
     instr_t* t = get_instr(fun, a); // 判断过去的 jmp 目标地址是否合法
     if ( t == NULL )
     {
      bad = 0x10002;
      break;
     }
     jmplist.remove(a);
    }
   }
   if ( bad >=  10 )
    break;
   if ( jmplist.empty() && instr.is_ret ) // 函数结束
    break;
   p += instr.len;
   fun.instr_lst.push_back(instr); // 保存指令信息
  }
}

// 把三张表放在一起进行，按照权值高低(即主要包括引用次数，函数特征)除重过滤
// 首先把 suspect_lab_lst 中的项和其他2张表进行对撞，如果其基址值在
// 别的表中存在，则消去此表元素，将引用次数加到目标表中.
// 过滤的主要规则包括：发生重叠时，保留"极"高权项，否则没有全抛弃。
// ... 还有很多可以去除和提高可疑数据和函数的方法，就不细数了。
// 思路有了，代码也就好写了，伪码我也不列了。
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

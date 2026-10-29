
```c
struct arm_inst_ctx {

    reg_t   ld;     // 目的寄存器

    reg_t   ld2;    // 目的寄存器2

    reg_t   lm;     // 参数寄存器1

    reg_t   ln;     // 参数寄存器2

    reg_t   lp;     // 参数寄存器3

    int     register_list;  // 寄存器列表，一般给push和pop用

    int     imm;    // 立即数

    int     m; 

    int     setflags;   // 是否需要跟新flag

    int     cond;       // 条件

    int     vd;         // simd中需要



    struct {

        unsigned p : 1;

        unsigned u : 1;     // 在一些立即数中，指出是加还是减

        unsigned w : 1;

        unsigned t : 2;

    };

};
```

我们基于这个规则，完整的定义出整个`arm thumb`的指令集(暂时只定义了需要的部分)

```c
struct arm_inst_desc desclist[] = {

    {"0000    o1    i5  lm3 ld3",           {t1_inst_lsl, t1_inst_lsr}, {"lsl", "lsr"}},

    {"0001    0     i5  lm3 ld3",           {t1_inst_asr}, {"asr"}},

    {"0001    10 o1 lm3 ln3 ld3",           {t1_inst_add, thumb_inst_sub_reg}, {"add", "sub"}},

    {"0001    11 o1 i3  ln3 ld3",           {t1_inst_add, thumb_inst_sub}, {"add", "sub"}},

    {"0010    0 ld3    i8",                 {thumb_inst_mov}, {"mov"}},

    {"0010    1 lm3    i8",                 {t1_inst_cmp_imm}, {"cmp"}},

    {"0011    0 ld3 i8",                    {t1_inst_add}, {"add"}},

    {"0011    1 ld3 i8",                    {thumb_inst_sub}, {"sub"}},



    {"0100    0000 o2 lm3 ld3",             {t1_inst_and, thumb_inst_eor, t1_inst_lsl, t1_inst_lsr}, {"and", "eor", "lsl2", "lsr2"}},

    {"0100    0001 o2 lm3 ld3",             {t1_inst_asr, t1_inst_adc, t1_inst_sbc, t1_inst_ror}, {"asr", "adc", "sbc", "ror"}},

    {"0100    0010 0o1 lm3 ld3",            {t1_inst_tst, t1_inst_neg}, {"tst", "neg"}},

    {"0100    0010 1o1 lm3 ln3",            {thumb_inst_cmp, t1_inst_cmn}, {"cmp", "cmn"}},

    {"0100    0011 o2 lm3 ld3",             {thumb_inst_orr, t1_inst_mul, t1_inst_bic, t1_inst_mvn}, {"orr", "mul", "bic", "mvn"}},

    {"0100    0110 d1 lm4 ld3",             {t1_inst_mov_0100}, {"mov"}},

    {"0100    0100 d1 lm4 ld3",             {t1_inst_add}, {"add"}},

    {"0100    0101 01 hm3 ln3 ",            {thumb_inst_cmp}, {"cmp"}},

    {"0100    0101 10 lm3 hn3 ",            {thumb_inst_cmp}, {"cmp"}},

    {"0100    0101 11 hm3 hn3 ",            {thumb_inst_cmp}, {"cmp"}},

    {"0100    1 ld3 i8",                    {thumb_inst_ldr}, {"ldr"}},

    {"0100    0111 o1 lm4 000",             {t1_inst_bx_0100, thumb_inst_b}, {"bx", "blx"}},

    {"0110    0i5 ln3 lm3",                 {thumb_inst_str},    {"str"}},

    {"0110    1i5 lm3 ld3",                 {thumb_inst_ldr},       {"ldr"}},

    {"0111    0 i5 ln3 lm3",                {thumb_inst_strb},      {"strb<c> <lp>,[<ln>,#<imm>]"}},

    {"0111    1 i5 ln3 ld3",                {thumb_inst_ldrb},      {"ldrb<c> <ld>,[<ln>,#<imm5>]"}},

    {"1001    0 lm3 i8",                    {thumb_inst_str}, {"str"}},

    {"1001    1 ld3 i8",                    {thumb_inst_ldr}, {"ldr"}},

    {"1010    o1 ld3 i8",                   {t1_inst_add1, t1_inst_add}, {"add", "add"}},

    {"1011    o1 10 m1 rl8",                {thumb_inst_push, thumb_inst_pop}, {"push", "pop"}},

    {"1011    0000 0 i7",                   {thumb_inst_add}, {"add"}},

    {"1011    0000 1 i7",                   {thumb_inst_sub}, {"sub"}},

    {"1011    1111 c4 i4",                  {thumb_inst_it}, {"it"}},

    {"1100    1 ln3 rl8",                   {thumb_inst_ldmia}, {"ldmia<c> <Rn>!?, <registers>"}},

    {"1101    c4 i8",                       {thumb_inst_b}, {"b<cond>"}},

    {"1110    0 i11",                       {t1_inst_b}, {"b"}},



    {"1110 100p1 u1 1 w1 0 lm4 ln4 lp4 i8", {thumb_inst_strd},  "strd<c>.w"},

    {"1110 100p1 u1 1 w1 1 ln4 ld4 le4 i8", {thumb_inst_ldr},   "ldrd<c>.w"},



    {"1110 1001 0010 1101 0 m1 0 rl13",     {thumb_inst_push}, "push.w"},

    {"1110 1000 1011 1101 i1 m1 0 rl13",    {thumb_inst_pop}, "pop.w"},



    {"1110 1010 010s1 ln4 0 i3 ld4 i2 t2 lm4",    {thumb_inst_orr}, "orr{s}<c>.W <ld>,<lm>{, <shift>}"},



    {"1110 1101 0d1 10 1101 vd4 1011 i8",   {thumb_inst_vpush}, "vpush<c> <list>"},

    {"1110 1100 1d1 11 1101 vd4 1011 i8",   {thumb_inst_vpop},  "vpop <list>"},



    {"1110 1010 1001 ln4 0i3 1111 i2 t2 lm4",   {thumb_inst_teq},  "teq<c> <ln>, <lm>{, <shift>}"},



    {"111i1 1111 1d1 00 0i3 vd4 i4 0i2 1 i4",   {thumb_inst_vmov},  "vmov"},



    {"1110 1011 000s1 ln4 0i3 ld4 i2 t2 lm4 ",  {thumb_inst_add}, "add{s}<c>.W <ld>,<ln>,<lm>{, <shift>}"},

    //{"1111 0i1 01 000s1 1101 0i3 ld4 i8",       {thumb_inst_add}, "add{S}<c>.W <Rd>,SP#<const>"},
    {"1111 0i1 01 000s1 ln4 0i3 ld4 i8",        {thumb_inst_add}, "add{S}<c>.W <Rd>,<Rn>,#<const>"},
  

    {"1111 0s1 c4 i6 10 u1 0 w1 i11",       {thumb_inst_b}, "b<c>.w <label>"},
    {"1111 0s1 c4 i6 10 u1 1 w1 i11",       {thumb_inst_b}, "b<c>.w <label>"},
    {"1111 0s1 i10 11 u1 1 w1 i11",         {thumb_inst_b}, "bl<c> <label>"},
    {"1111 0s1 i10 11 u1 0 w1 i10 0",       {thumb_inst_b}, "blx<c> <label>"},



    {"1111 0i1 01 101s1 ln4 0i3 ld4 i8",    {thumb_inst_sub}, "sub{s}<c>.w <ld>,<ln>,#<const>"},
  

    {"1111 0i1 00010 s1 11110 i3 ld4 i8",   {t1_inst_mov_w}, "mov.w"},
    {"1111 0i1 101100 i4 0 i3 ld4 i8",      {thumb_inst_mov}, "movt"},
    {"1111 0i1 100100 i4 0 i3 ld4 i8",      {thumb_inst_mov}, "movw"},

  

    {"1111 1000 0001 ln4 ld4 1 p1 u1 w1 i8",    {thumb_inst_ldrb}, {"ldrb<c>.w <ld>, [<ln>,#<imm8>]"}},
    {"1111 1000 1001 ln4 ld4 i12",              {thumb_inst_ldrb}, {"ldrb<c>.w <ld>, [<ln>,#<imm12>]"}},
    {"1111 1000 0101 ln4 ld4 0000 00 i2 lm4",   {thumb_inst_ldr}, {"ldr<c>.w <ld>, [<ln>,<lm>{,LSL #<shift>}]"}},
    {"1111 1000 1101 ln4 ld4 i12",              {thumb_inst_ldr}, {"ldr.w"}},
    //{"1111 1000 u1 101 1111 ld4 i12",         {thumb_inst_ldr}, {"ldr.w"}},
    {"1111 1000 1000 ln4 lm4 i12",              {thumb_inst_strb}, {"strb<c>.w <Rt>,[<Rn>,#<imm12>]"}},
    {"1111 1000 0000 ln4 lm4 1p1 u1 w1 i8",     {thumb_inst_strb}, {"strb<c> <Rt>,[<Rn>,#+/-<imm8>]"}},
    {"1111 1000 1100 ln4 lm4 i12",              {thumb_inst_str}, {"str<c>.w <Rt>,[<Rn>,#<imm12>]"}},
    {"1111 1000 0100 ln4 lp4 000000 i2 lm4",    {thumb_inst_str}, {"str<c>.w <lo>,[<ln>,<lm>{, LSL #<shift>}]"}},



    {"1111 1001 0d1 00 ln4 vd4 i8 lm4",    {thumb_inst_vst1}, {"str<c>.w <lo>,[<ln>,<lm>{, LSL #<shift>}]"}},

};
```

然后基于此，按 `nfs` -> `dfs` -> `2d-trans`的方式来生成整个状态机图

`nfa`的如下:

![thumb指令nfa状态机](https://bbs.kanxue.com/upload/attach/202007/630711_38C6DKGYMGRNSMB.png)

`nfa`在归纳同字符集可到达的边，生成`dfa`， ![thumb指令dfa状态机](https://bbs.kanxue.com/upload/attach/202007/630711_M3K6YUPNB2RXPTG.png)

PS. 不知道为什么，看雪把我图片压缩了，想看具体大图，可以看着 [状态机大图](https://bbs.kanxue.com/elink@f6dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1j5h3W2C8j5h3W2K6K9r3W2#2j5#2)9J5c8X3k6S2M7%4c8$3L8g2)9J5c8Y4N6A6K9$3W2Q4x3V1k6S2M7X3#2Q4x3X3c8@1K9s2g2E0j5W2)9J5k6r3W2F1M7%4c8J5N6h3y4@1K9h3!0F1i4K6u0V1k6r3W2K6j5i4y4E0)

详细分析可以参考任意一本形式语言与自动机理论，或者 龙书， 《parsing techniques》都可。

# 语法分析

## `it`指令处理

语法分析反而比较简单，因为汇编语言不存在特殊的语法，唯一特殊的指令是`it`指令，在`ida`中`arm`的`it`指令是不会做切分，和上下的代码完整的放到一个`block`中，比如

![it示例](https://bbs.kanxue.com/upload/attach/202007/630711_FSJAPDFTHBESDA5.png)

但是假如我们需要用编译优化去掉混淆，必须得把`it`指令做为一个单独的`bcond`指令来处理，比如上图中的代码，转换成`c`语言后如下:

```
if (!r0) {

   rt = 0x58586a70;

}

```

基于此，上图中的指令，被我们转换成如下格式:

![it转换过后的](https://bbs.kanxue.com/upload/attach/202007/630711_T22J7S4E8EA7F85.png)

## `bcond`指令处理

`b<cond>`指令没啥太多需要注意的，他末尾跟着的指令指向他的`false_label`分支，它的跳转指向它的`true_label`分支，这一点上和`it`是相反的

# 编译优化

## 生成`def`和`use`关系

因为要做后端的数据流分析，必须给出所有指令的`def`和`use`关系， 举几个例子说明需要注意的地方，因为是直接在`arm`的汇编上做的`def`和`use`，所以实际的代码写起来很多的`corner case`非常难处理，举一些典型的例子说明下

### `cmp`

```
cmp r0, r1

// def = APSR 

// use = ln, lm
```

### `movw`

```
MOVW       R1, #0X7571

// def = r1

// use = empty
```

### `movt`

```
MOVT       R1, #0X791

// def = r1

// use = r1, 这个地方是把0x791移到r1的高位，所以它的use也包含r1
```

### `add`

```
adds     r0, 1

// def = apsr, r0; add的某些指令会设置apsr寄存器，所以它的def会有多个，这也导致了普通的活跃性分析的代码，要做修改才能适应反混淆

// use = empty,
```

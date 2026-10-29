
# [原创]一个烂尾的Android内联hook框架

## 1.引言

之前因为有某些需求想找一个arm平台的inline hook框架，但是没能找到一个既精简、又支持修改多参数、修改返回值、支持用自定义函数替换掉原函数的框架。所以就决定花点时间自己实现一个，写了几天以后被告知不用做了，于是便有了这个烂尾项目。今天整理文件的时候看到了，与其留着发霉不如分享给坛友作为学习的例子。此代码为没用的草稿，不合规范的地方就不用对其提意见了。

## 2.需求分析与初步设计

### 2.1 精简

使用一个结构体和一段shellcode维护一个hook

### 2.2 支持多线程hook

每个hook维护一段只属于自己的shellcode

### 2.3 支持修改参数

参数数量小于4的修改`r0-r3`寄存器，大于4的再修改栈空间

### 2.4 支持修改返回值

覆盖`r0`寄存器的值

### 2.5 支持函数替换

`hook`函数执行完以后加载原函数下一条指令地址到`lr`寄存器

总体设计为，每个hook维持一个结构体与一段自己的shellcode，初始化hook时直接修改shellcode中的变量值，备份并修改原函数的起始汇编代码，使其直接跳到shellcode执行，完成其余的工作。所有的hook保存在一个链表中进行统一管理。

## 3.具体实现

以下为维护hook的结构体：

```
typedef struct inline\_item{
void\* shell\_code; // shelcode的内存起始地址
int32\_t shell\_code\_len; // shellcode的长度
address\_t old\_fun; // 原始函数的起始地址
address\_t new\_fun; // hook函数的起始地址
void\* back\_code; // 备份的代码
int32\_t back\_code\_len; // 备份代码的长度
void\* reg; // 保存寄存器空间
void\* stack; // 保存栈空间
list\_node\_t\* param\_list; // 参数链表
void\* ret\_value; // 返回值
} inline\_item\_t;
```

在初始化hook时，首先获取shellcode中变量的内存地址，分配一块内存空间给结构体中的shellcode_code字段，并记录长度，记录原始函数的地址与hook函数的地址，存储原函数的前几条指令作为备份代码，并记录备份代码的长度，`param_list`中存储有想要修改的参数值，`ret_value`中存储有想要修改成的返回值。

初始化hook的代码如下：

```
inline\_item\_t\* init\_item(address\_t\* target\_fun, address\_t\* hook\_fun, list\_node\_t\* param\_list,address\_t ret\_value)
{
inline\_item\_t\* p\_item = 0;
address\_t temp = 0;
p\_item = (inline\_item\_t\*)malloc( sizeof(inline\_item\_t) );
p\_item->shell\_code\_len = (int8\_t\*)&asm\_shellcode\_end - (int8\_t\*)&asm\_shellcode\_begin;
p\_item->shell\_code = (int8\_t\*)malloc( p\_item->shell\_code\_len); // 分配shellcode空间
p\_item->old\_fun = target\_fun; // 保存原始函数地址
p\_item->new\_fun = hook\_fun; // 存储hook函数地址
p\_item->back\_code\_len = 12; // 备份12字节指令
p\_item->back\_code = malloc(p\_item->back\_code\_len);
p\_item->reg = malloc(0x10 \* sizeof(address\_t)); // 分配备份寄存器的空间
p\_item->stack = malloc(0x10 \* sizeof(address\_t)); // 分配备份栈的空间
p\_item->param\_list = 0;
p\_item->ret\_value = 0;
if( param\_list != 0){
p\_item->param\_list = param\_list; // 初始化要修改的参数列表
}
if( ret\_value!=0 ){
p\_item->ret\_value = (address\_t\*)malloc( sizeof(address\_t) ); // 初始化要修改的返回值
ret\_value = 10;
memcpy( p\_item->ret\_value, &ret\_value, sizeof(address\_t));
}
return p\_item;
}
```

在备份原函数前几条指令时需要对指令类型进行判断，如果是thumb指令，则需要对地址减1再进行保存。

判断指令类型的代码如下：

```
bool get\_type(address\_t addr)
{
#if defined(\_\_arm64\_\_) || defined(\_\_aarch64\_\_)
return ((int64\_t)addr & 0x1);
#else
return ((int32\_t)addr & 0x1);
#endif
}
```

备份指令的函数如下：

```
void back\_target(inline\_item\_t\* p\_item){
p\_item->back\_code\_len = BACKLEN;
//保存old\_fun前几条指令
p\_item->back\_code = (int8\_t\*)malloc(p\_item->back\_code\_len);
memcpy(p\_item->back\_code, (address\_t\*)p\_item->old\_fun, p\_item->back\_code\_len);
memcpy(p\_item->shell\_code + asm\_pos.back\_code\_pos, p\_item->back\_code, p\_item->back\_code\_len);
//修改old\_fun前几条指令
if( !set\_mem\_permission(p\_item->old\_fun,RWX) ){ // 修改内存属性为可读可写可执行
return ;
}
memcpy((address\_t\*)p\_item->old\_fun, p\_item->shell\_code + asm\_pos.load\_pc\_pos,sizeof(address\_t));
memcpy((address\_t\*)p\_item->old\_fun + 1, &p\_item->shell\_code, sizeof(address\_t));
memcpy(p\_item->shell\_code+asm\_pos.load\_pc\_pos, p\_item->back\_code, sizeof(address\_t)\*3);
}
```

指令地址为奇数是thumb指令

在shellcode开始执行之前，需要先对其中的变量进行初始化，以下为需要初始化的值：

```
asm\_old\_fun\_continue: // 原函数的地址
.word 0x12345678
asm\_new\_fun: // hook函数的地址
.word 0x12345678
asm\_param\_num: //参数的个数
.word 0x00000010
asm\_reg\_sp: // 栈顶值
.word 0x12345678
asm\_reg\_lr: // 返回地址
.word 0x12345678
callback\_modify\_param: // callback\_modify\_param函数的地址
.word 0x00000000
callback\_modify\_return: // callback\_modify\_return函数的地址
.word 0x00000000
asm\_item\_addr: // 当前hook结构体地址
.word 0x12345678
asm\_reg\_addr:
.word 0x12345678
asm\_stack\_addr:
.word 0x12345678
asm\_back\_code: // 备份的原始函数代码块
.word 0x12345678
.word 0x12345678
.word 0x12345678
ldr r15,asm\_old\_fun\_continue // 跳转到备份代码的下一条指令继续执行
```

shellcode在执行时，需要先保存原始上下文，由于`r13,r14,r15`寄存器比较特殊，在跳转到保存上下文代码块时，寄存器的值已经发生变化，因此需要单独对其进行保存

```
str r13,asm\_reg\_sp
str r14,asm\_reg\_lr
// stmfd sp!,{r0} // 这个需要注意，保存的pc应当是old\_fun的下一条指令地址，
// sub r0,r15,0x // 所以执行到这里以后要保存当前pc减去已经执行的指令步数\*4
// ldmfd sp!,{r0}
bl fun\_save\_context
```

再进行上下文环境保存，代码如下：

```
fun\_save\_context:
stmfd sp!,{r12} // ------- 开始保存寄存器
ldr r12,asm\_reg\_addr
stmea r12!,{r0-r11} //保存r0--r11
mov r0,r12
ldmfd sp!,{r12} //取出r12
stmea r0!,{r12} //保存r12
mrs r1,cpsr //保存CPSR寄存器
stmea r0!,{r1}
// mov r1,sp //不能直接操作sp
// stmea r0!,{r1} // ------- 寄存器保存完成
ldr r0,asm\_stack\_addr //-- 开始保存栈空间
mov r2,0x10 //需要保存的栈空间大小
mov r1,sp
\_loop\_save\_stack:
cmp r2,0x00
ble \_loop\_save\_context\_end
ldr r3,[r1]
str r3,[r0],0x04
add r1,r1,0x04
sub r2,r2,0x01
b \_loop\_save\_stack
\_loop\_save\_context\_end: //-- 栈空间保存完成
bx r14
fun\_save\_context\_end:
```

与保存上下文相对应的，以下是恢复上下文的代码块：

```
fun\_back\_context:
ldr r0,asm\_stack\_addr // 开始恢复栈空间
mov r1,sp
mov r2,0x10
\_loop\_back\_stack:
cmp r2,0x00
ble \_loop\_back\_stack\_end
ldr r3,[r0]
str r3,[r1],0x04
add r0,r0,0x04
sub r2,r2,0x01
b \_loop\_back\_stack
\_loop\_back\_stack\_end: // 恢复栈空间完成
ldr r12,asm\_reg\_addr
ldmfd r12!,{r0-r11} // 先取出r0-r11
stmfd sp!,{r0-r1}
mov r0,r12
ldmfd r0!,{r12} // 取出r12
ldmfd r0!,{r1}
msr cpsr,r1
// ldmfd r0!,{r1}
// mov sp,r1
ldmfd sp!,{r0-r1}
bx r14
fun\_back\_context\_end:
```

执行hook的代码：

```
ldr r13,asm\_reg\_sp
bl fun\_back\_context // 保存上下文
ldr r0,asm\_new\_fun
blx r0 // 带返回跳转到hook函数
stmfd sp!,{r0,r1}
ldr r1,callback\_modify\_param
cmp r1,0x00
beq not\_modofy\_param // 判断是否需要修改参数
ldr r0,asm\_item\_addr
blx r1 // 如果需要修改参数，以hook结构体作为参数带返回跳转callback\_modify\_param
not\_modofy\_param:
ldmfd sp!,{r0,r1}
ldr r13,asm\_reg\_sp
bl fun\_back\_context
bl asm\_back\_code // 执行原函数
stmfd sp!,{r0,r1}
ldr r1,callback\_modify\_return
cmp r1,0x00
beq not\_modofy\_return // 执行完原函数后，判断是否需要修改返回值
ldr r0,asm\_item\_addr // 如果需要修改返回值，以hook结构体作为参数带返回跳转
blx r1
not\_modofy\_return:
ldmfd sp!,{r0,r1}
bl fun\_back\_context // 恢复上下文
ldr r14,asm\_reg\_lr // 将old\_fun的lr地址填充回去
bx r14 // 跳转回正常执行流程
```

以上代码中，`callback_modify_param`被填充为以下函数：

```
void modify\_param(inline\_item\_t\* p\_item ){
int n;
int index;
address\_t data;
list\_node\_t\* param\_list ;
if(!p\_item){
return ;
}
param\_list = p\_item->param\_list; // 参数存储在结构体的一个链表中
while(param\_list != 0){ // 遍历整个链表
index = ((param\_t\*)param\_list->data)->index;
data = ((param\_t\*)param\_list->data)->data;
if( index <=3 ){ // 如果参数小于3个，修改寄存器的值
memcpy(p\_item->reg + sizeof(address\_t)\*index, &data, sizeof(address\_t) );
}
else { // 参数大于3个，修改栈顶值
memcpy(p\_item->stack + sizeof(address\_t)\*(index-4), &data, sizeof(address\_t));
}
param\_list = param\_list->next;
}
}
```

`callback_modify_return`的值被填充为以下函数的地址：

```
void modify\_return(inline\_item\_t\*p\_item){
if(!p\_item) {
return ;
}
//覆盖r0的值修改返回值
memcpy(p\_item->reg, p\_item->ret\_value, sizeof(address\_t));
}
```

指令修复部分还没完成。

最后附上Makefile:

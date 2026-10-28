
``` 
00401C68 . EB 0E jmp short Anunnaki.00401C78

00401C6A . EB 0C jmp short Anunnaki.00401C78

00401C6C > E8 EF030000 call <Anunnaki.Garble_Create_Imm> ;产生imm 方式

00401C71 . EB 05 jmp short Anunnaki.00401C78

00401C73 > E8 18010000 call <Anunnaki.Garble_Sub_Call> ;产生call

00401C78 > 85C9 test ecx,ecx

00401C7A . 74 06 je short Anunnaki.00401C82

00401C7C . 49 dec ecx

00401C7D . E8 9AFFFFFF call <Anunnaki.Garble>

00401C82 > C3 retn
```

## V) 各种指令的构造原理

随机产生各种寄存器的情况下，根据每一种指令的特点，产生不同寄存器的不同寻址方式，下面做简要说明，`rx`（表示任意一个可使用的寄存器）

### 1 push : 可产生 `push xxxx` / `push rx` / `push [rx + xxxxx]` 方式

a) 立即数寻找方式:

``` 
Garble_Push_Imm: mov eax,2

call random

push ebx

mov ebx,eax

rol eax,1

add eax,68h

stosb
```

b) push随机寄存器方式

``` 
Garble_Push_Reg32: call get_reg_32 ; 返回0 ~ 8 的寄存器索引

add al,50h

stosb

retn
```

c) push内存寻址方式

``` 
Garble_Push_Modrm: mov al,0ffh

stosb

push edi

call Garble_Create_Modrm_Byte

pop eax

and byte ptr [eax],11000111b ; 2(11):3(000):3(111) -> 2(11) -> [reg + rm]

add byte ptr [eax],6 SHL 3

retn
```

### 2 imm 方式

``` 
Garble_Create_Imm:

push ebx

call get_free_reg_32_no_set ;获得一个不被使用的寄存器,从poly_vars->poly_reg_usage 获得

mov ebx,eax ;并重新设置相关位

cmp eax,-1

je Garble_Create_Imm_E

mov eax,5

call random

test eax,eax

jz Garble_Imm_Init_Ptr

cmp eax,1

jbe Garble_Imm_Inc_Dec

mov eax,0b8h ;产生一个mov rx， (ebx 存放具体不能被使用的寄存器索引数值)

add eax,ebx

stosb

mov eax,-1 ;设置一个32bit最大值

call random

stosd ;将4byteimm写入内存，形成mov rx，imm

pop ebx

retn
```

同理还可以产生 `sub rx` / `inc rx` / `dec rx` 等等方式操作。

### 3 mod/rm 方式

步骤: 1 产生一个随机数，判断是否要有`0x66`前缀
2 获得当前能用的寄存器标志，如果不等于0，则产生`add` / `or` / `and` / `sub` / `xor` / `mov` 指令，否则跳向步骤3
3 没有可选寄存器，调用`Garble_Create_Modrm_Byte`，生成`mov rx,rx` 等指令。

对于没有寄存器可用的情况下，如何生成不影响当前代码的指令，ope使用的简洁的一个方案，那就是一律产生`mov r1，r1`（r1指相同的寄存器）类指令，具体方式，

向缓冲区写入`0x89` (opcode -> mov)

opcode format +----------+---------+---------+--------+-------------+----------+
|prefixes | opcode |Mod/rm |sib |displacement |immediate |
+----------+---------+---------+--------+-------------+----------+
|0x89 |
+---------+

可以看出，要产生`mov r1，r1` 指令，关键是mod/rm域中要填入的合适的值。mod/rm域用于指出寻址方式，包括内存寻址及寄存器寻址，mod/rm占1字节，按2![](https://bbs.kanxue.com/view/img/face/1.gif)3 bit解析,当m1可表示4种寻址模式，当m1 == 11时表示寄存器到寄存器，（m2，m3）占3bit，表示8种寄存器,故只要保证m2，m3数值相同，逻辑或 `0xc0` 即可。

m1 m2 m3
mod/rm --> +--------+---------+--------+
| 11 | 00 | 00 | ----> 0xc0
+--------+---------+--------+

其他类型指令的产生与上述情况类似，涉及mod/rm sib displacement immediate 格式的解析，生成不同指令，不再赘述。

具体的变形代码如下:

``` 
Garble_Create_Modrm_Byte:

push ebx ; 保存poly_vars结构

call get_free_reg_32_no_set ;获得一个未使用的寄存器索引

cmp eax,-1

je Modrm_reg1_reg1 ; 没有可用寄存器

rol eax,3

mov ebx,eax

mov eax,5

call random

test eax,eax

jz Modrm_Reg_Reg

cmp eax,3

jb Modrm_Stack_Read

Modrm_Mem_Acc:

mov eax,3

call random

test eax,eax

jz Modrm_Mem_Direct

cmp dword ptr [ebp].poly_junk_mem_pos,0

je Modrm_Mem_Direct

Modrm_Mem_AccNoDisp:

mov eax,dword ptr [ebp].poly_junk_mem_reg

add eax,ebx

stosb

mov eax,dword ptr [ebp].poly_read_mem_size

sub eax,4

call random

add eax,dword ptr [ebp].poly_read_mem_base

mov ebx,eax

sub ebx,dword ptr [ebp].poly_junk_mem_pos

mov eax,6

call random

cmp eax,0

je Modrm_Mem_Disp32

cmp eax,4

jb Modrm_Mem_Disp8

pop ebx

retn

Modrm_Mem_Disp8:

add byte ptr [edi - 1],40h

mov byte ptr [edi],bl

inc edi

pop ebx

retn

Modrm_Mem_Disp32:

add byte ptr [edi - 1],80h

mov dword ptr [edi],ebx

add edi,4

pop ebx

retn

Modrm_Mem_Direct:

mov eax,dword ptr [ebp].poly_options

and eax,POLY_OPT_MEM_ACC_DIRECT

test eax,eax

jnz Modrm_Stack_Read

cmp dword ptr [ebp].poly_read_mem_base,0

je Modrm_Stack_Read

mov eax,5

add eax,ebx

stosb

mov eax,dword ptr [ebp].poly_read_mem_size

call random

add eax,dword ptr [ebp].poly_read_mem_base

stosd

jmp Modrm_End

Modrm_Stack_Read:

mov eax,ebx

add eax,45h

mov byte ptr [edi],al

mov eax,2

call random

test eax,eax

jz Modrm_Stack_Ebp

mov byte ptr [edi + 1],24h

sub byte ptr [edi],1

inc edi

Modrm_Stack_Ebp:

inc edi

mov eax,POLY_ESP_ACC_RNG_MAX

call random

imul eax,4

mov ebx,eax

mov eax,2 ; +/- disp

call random

test eax,eax

jz Modrm_Stack_DispPos

xor eax,eax

sub eax,ebx

Modrm_Stack_DispPos:

mov byte ptr [edi],al

inc edi

jmp Modrm_End

Modrm_Reg_Reg:

call get_reg_32_no_stack;产生mov r1，r2

add eax,ebx ; free reg

add eax,0c0h

stosb

jmp Modrm_End

Modrm_reg1_reg1:

call get_reg_32 ; 产生 mov r1/r1

mov ah,al

rol al,3

add al,ah

add al,0c0h

stosb

Modrm_End:

pop ebx

retn
```

### 4 sub_call 方式

步骤 1 检测配置中是`poly_subroutines_count == 0` ？是0则退出，否则步骤2
2 检测配置中是否设置subroutine标志，没设置退出
3 检测`poly_subroutines_table`中是否参数标志，因为产生的call 是按`__cdecl`方式压栈的
4 如果是有参数的情况，负责清栈。
5 调用`Garble_Create_Push`，产生不同类型的`push 32_bit` / `push 8_bit` / `push rx` / `push [rx + rx]`
6 生成call 指令

``` 
Garble_Sub_Call:

push ecx

push ebx

cmp dword ptr [ebp].poly_subroutines_count,0 ; 检测是否设置的sub_call方式

je Garble_Sub_Call_End

call Is_In_Subr ;

test eax,eax

jnz Garble_Sub_Call_End

mov eax,dword ptr [ebp].poly_subroutines_count ;读取计数

call random

lea eax,[eax * 8]

lea ebx,[ebp].poly_subroutines_table

add ebx,eax

cmp dword ptr [ebp].poly_junk_mem_pos,0 ; 比较是否初始化junk内存数据

je Garble_Sub_No_Save1

mov ecx,dword ptr [ebp].poly_junk_mem_reg

add ecx,50h

mov byte ptr [edi],cl ;

inc edi

Garble_Sub_No_Save1:

mov ecx,dword ptr [ebx + 4] ;

test ecx,ecx

jz Garble_Sub_No_Arg ;生成无参数call 指令

Garble_Sub_Arg: ;为有参数call 设置push 指令

call Garble_Create_Push

loop Garble_Sub_Arg

Garble_Sub_No_Arg:

mov eax,dword ptr [ebx]

sub eax,edi

sub eax,5

mov byte ptr [edi],0e8h

inc edi

stosd

mov eax,dword ptr [ebx + 4] ;

test eax,eax

jz Garble_Sub_No_Save2

imul eax,4

rol eax,16

add eax,9000c483h ； 产生 sub esp , xxxx,清空堆栈

stosd

dec edi

Garble_Sub_No_Save2:

cmp dword ptr [ebp].poly_junk_mem_pos,0

je Garble_Sub_Call_End

mov eax,dword ptr [ebp].poly_junk_mem_reg

add eax,58h

stosb ; pop rx

Garble_Sub_Call_End: pop ebx

pop ecx

retn
```

### 5 loop 方式

步骤 1 检测配置中是否设置loop方式，随机设置loop的循环次数，范围`10000h ~ 1000h`
2 初始化loop 所用的寄存器
3 产生一个对`rx` 赋值循环计数的指令，支持的格式包括 `mov rx,cnt` / `lea rx ,[cnt]` / `push cnt ,pop rx`
4 在loop插入仿真无效指令
5 递减计数

``` 
Garble_Loop:

call Is_In_Loop

test eax,eax

jnz Garble_Loop_End

cmp ecx,5

jbe Garble_Loop_End

call get_free_reg_32

cmp eax,-1

je Garble_Loop_End

call Set_In_Loop ; 保存设置

mov ebx,eax
```


# 字符串处理汇编分析

```
.text:00409178                      test    edx, edx        ; 判断 byte 标志域  
.text:0040917A                      jz      short loc_40918D ; 相等说明不是 delim 继续循环  
.text:0040917C                      mov     ecx, [ebp+var_4] ; 不相等说明找到 delim 字符  
.text:0040917F                      mov     byte ptr [ecx], 0 ; 将与 delim 字符相同的位置设置为空字符  
.text:00409182                      mov     edx, [ebp+var_4] ; 把地址保存到 edx 中  
.text:00409185                      add     edx, 1           ; edx+1  
.text:00409188                      mov     [ebp+var_4], edx ; var_4 += 1;  
.text:0040918B                      jmp     short loc_40918F ; 跳转到返回代码处  
.text:0040918D ; ---------------------------------------------------------------------------  
.text:0040918D  
.text:0040918D loc_40918D:                            ; CODE XREF: _strtok+10Aj  
.text:0040918D                      jmp     short loc_40913F  
.text:0040918F ; ---------------------------------------------------------------------------  
.text:0040918F  
.text:0040918F loc_40918F:                            ; CODE XREF: _strtok+E1j  
.text:0040918F                                      ; _strtok+11Bj  
.text:0040918F                      mov     eax, [ebp+var_4] ; eax= str 查询完后的地址  
.text:00409192                      mov     dword ptr ?$S1@?1??_Nomemory@std@@YAXXZ@4EA+20h, eax ; 保存 eax，变量位置（？）  
.text:00409197                      mov     ecx, [ebp+Str]  ; ecx = str  
.text:0040919A                      cmp     ecx, [ebp+var_4] ; 判断 ecx 和 str 的地址  
.text:0040919D                      jnz     short loc_4091A3 ; 不相等则跳走  
.text:0040919F                      xor     eax, eax  
.text:004091A1                      jmp     short loc_4091A6 ; 如果相等则 eax=NULL 作为返回值返回  
.text:004091A3 ; ---------------------------------------------------------------------------  
.text:004091A3  
.text:004091A3 loc_004091A3:                           ; CODE XREF: _strtok+12Dj  
.text:004091A3                      mov     eax, [ebp+Str]  ; str->eax 作为返回值  
.text:004091A6  
.text:004091A6 loc_4091A6:                            ; CODE XREF: _strtok+131j  
.text:004091A6                      mov     esp, ebp  
.text:004091A8                      pop     ebp  
.text:004091A9                      retn  
.text:004091A9 _strtok          endp  
```

这个函数可能是所有 C 库字符串函数中最复杂的一个，如下几点值得学习：

A 取位 `&7` 对于二进制位的设置  
`and` 用于置 0，`or` 用于置 1 比较方便

B 位域是用位作为条件标志，该函数是将字节作为条件标志

C `*str>>3` 缩小 8 倍的方法比较有意思。根据 `2⁸` `char` 类型的排列组合数结合 32 位计算机来使用顺序表。

## 17. setmem

原型：`extern void setmem(void *buf, unsigned int count, char ch);`

用法：`#include <string.h>`

功能：把 `buf` 所指内存区域前 `count` 个字节设置成字符 `ch`。

在 `vc` 环境中好像没有，`msdn` 中也没有找到该函数。

## 18. movmem

`char` 类型本身主要考虑到内存的基本单位字节而引入，因此对于字符串函数中有一些内存操作函数。

在 `vc` 中没有这个函数，`msdn` 中有一个：

```
VOID MoveMemory (
  PVOID Destination,   // move destination
  CONST VOID *Source,  // block to move
  SIZE_T Length        // size of block to move
);
```

## 19. memset

```
.text:00408160 ; 函数逆向要解决的问题：
.text:00408160 ; 1:返回值  进入函数后寻找 retn 指令及 eax 中的值
.text:00408160 ; 2:参数个数的判断 IDA 中参数以 arg_ 为前缘标出，如果是
.text:00408160 ; __cdecl 则可以根据 call 后 add 指令判断栈中参数的个数，
.text:00408160 ; __stdcall 可以参考 retn 上面的回收栈帧的代码来判断
.text:00408160 ; 3:参数的类型判断，先看所占内存大小，如果为 4 字节再根据
.text:00408160 ; 汇编代码中的[]寻址方式判断是指针还是数值。
.text:00408160 ; 4:参数的功能 分析其反汇编代码可知，系统层
.text:00408160 ; 程序 ＝ API + 数据结构，通过分析 api 可以了解函数功能，
.text:00408160 ; 对于系统级函数则只能分析代码
.text:00408160

.text:00408160 ; void __cdecl memset(void *, int, size_t)
.text:00408160 _memset        proc near          ; CODE XREF: _main+2Cp
.text:00408160                                    ; std::char_traits<char>::assign(char *,uint,char const &)+12p ...
.text:00408160

.text:00408160 arg_dst         = dword ptr  4
.text:00408160 arg_value       = byte ptr  8
.text:00408160 arg_size        = dword ptr  0Ch
.text:00408160

.text:00408160                  mov     edx, [esp+arg_size] ; arg_ 有三，即参数有 3 个，offset 4 8 c 所以其大小都为 4
.text:00408160                                    ; 然后再结合下面的代码分析是指针还是 int 类型
.text:00408164                  mov     ecx, [esp+arg_dst] ; 目标地址保存到 ecx
.text:00408168                  test    edx, edx        ; 判断要初始化的大小是否为 0
.text:0040816A                  jz      short toend     ; 如果要初始化的大小为 0 表示不用操作直接跳到返回代码 k
.text:0040816C                  xor     eax, eax        ; eax=0
.text:0040816E                  mov     al, [esp+arg_value] ; al = 要初始化的数据
.text:00408172                  push    edi             ; 保存 edi
.text:00408173                  mov     edi, ecx        ; ecx 地址保存到 edi(目标操作数寄存器)
.text:00408175                  cmp     edx, 4          ; 比较要初始化的大小是否小于 4
.text:00408178                  jb      short tail      ; 小于 4 时跳到 tail 中执行初始化
.text:0040817A                  neg     ecx             ; 地址求补
.text:0040817C                  and     ecx, 3          ; 判断字符串的存储地址是否对齐
.text:0040817F                  jz      short dwords    ; ecx=eax=value
.text:00408181                  sub     edx, ecx
.text:00408183

.text:00408183 adjust_loop:                          ; CODE XREF: _memset+27j
.text:00408183                  mov     [edi], al
.text:00408185                  inc     edi
.text:00408186                  dec     ecx
.text:00408187                  jnz     short adjust_loop
.text:00408189

.text:00408189 dwords:                               ; CODE XREF: _memset+1Fj
.text:00408189                  mov     ecx, eax        ; ecx=eax=value
.text:0040818B                  shl     eax, 8          ; ah = value
.text:0040818E                  add     eax, ecx        ; ax = value value
.text:00408190                  mov     ecx, eax        ; 保存 eax
.text:00408192                  shl     eax, 10h        ; eax 左移 16 位
.text:00408195                  add     eax, ecx        ; 扩充 value 值为 dword 类型
.text:00408195                                    ; value |= value<<8 | value<<16 |value<<24;
.text:00408197                  mov     ecx, edx        ; size->ecx
.text:00408199                  and     edx, 3          ; size & 3  3 = 0011B 如果是 0 则只能说明 size 的尾数位为
.text:00408199                                    ; 0000 0100  //0 4
.text:00408199                                    ; 1000 1100  //8 c
.text:00408199                                    ;  即判断初始的大小是否为 4 对齐
.text:00408199                                    ;
.text:0040819C                  shr     ecx, 2          ; 大小缩小 4 倍，因为它是以 dword 的形式进行初始化
.text:0040819F                  jz      short tail      ; 如果未对齐则去执行未对齐的操作
.text:004081A1                  rep stosd               ; ecx!=0 ecx=ecx-1 stod eax->edi ++edi;
.text:004081A3

.text:004081A3 main_loop_tail:                      ; 如果 edx=0
.text:004081A3                  test    edx, edx
.text:004081A5                  jz      short finish  ; 直接跳转到 finish
.text:004081A7

.text:004081A7 tail:                                 ; CODE XREF: _memset+18j
.text:004081A7                                    ; _memset+3Fj ...
.text:004081A7                  mov     [edi], al       ; 初始化
.text:004081A9                  inc     edi             ; 地址递增
.text:004081AA                  dec     edx             ; edx -= 1; inc dec 指令不影响 CF 标志位但是会影响 ZF 标志位
.text:004081AB                  jnz     short tail      ; 继续循环
.text:004081AD

.text:004081AD finish:                               ; CODE XREF: _memset+45j
.text:004081AD                  mov     eax, [esp+4+arg_dst]
.text:004081B1                  pop     edi
.text:004081B2                  retn                  ; 返回
.text:004081B3 ; ---------------------------------------------------------------------------
.text:004081B3
.text:004081B3 toend:                                 ; CODE XREF: _memset+Aj
.text:004081B3                  mov     eax, [esp+arg_dst]
.text:004081B7                  retn
```

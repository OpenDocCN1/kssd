
```c
.text:0000555555554BB6 public challenge2
.text:0000555555554BB6 challenge2 proc near ; CODE XREF: start+147↓p
.text:0000555555554BB6
.text:0000555555554BB6 var\_1D8 = qword ptr -1D8h
.text:0000555555554BB6 var\_1D0 = dword ptr -1D0h
.text:0000555555554BB6 var\_1CC = dword ptr -1CCh
.text:0000555555554BB6 var\_1C8 = dword ptr -1C8h
.text:0000555555554BB6 var\_1C4 = dword ptr -1C4h
.text:0000555555554BB6 name = utsname ptr -1C0h
.text:0000555555554BB6 s = byte ptr -3Ah
.text:0000555555554BB6 var\_32 = word ptr -32h
.text:0000555555554BB6 var\_30 = byte ptr -30h
.text:0000555555554BB6 var\_18 = qword ptr -18h
.text:0000555555554BB6
.text:0000555555554BB6 push rbp
.text:0000555555554BB7 mov rbp, rsp
.text:0000555555554BBA push rbx
.text:0000555555554BBB sub rsp, 1D8h
.text:0000555555554BC2 mov [rbp+var\_1D8], rdi
.text:0000555555554BC9 mov rax, fs:28h ; 从Segment fs(用于TLS)对应的内存段,偏移量40h处取一个值，把这个值加载到rax寄存器
.text:0000555555554BD2 mov [rbp+var\_18], rax
.text:0000555555554BD6 xor eax, eax ; 清零eax寄存器，更新零标志位ZF = 1(表示eax为0)
.text:0000555555554BD8 lea rax, [rbp+name] ; 将[rbp+name]的地址传给rax
.text:0000555555554BDF mov rdi, rax ; name
.text:0000555555554BE2 call \_uname
.text:0000555555554BE7 test eax, eax ; 不修改eax的内容,只更新标志位
.text:0000555555554BE9 jz short loc\_555555554BFC ; ZF=1，则转移到目的地址
.text:0000555555554BEB lea rdi, s ; "uname"
.text:0000555555554BF2 call \_perror ; \_perror() 用于打印错误信息
.text:0000555555554BF7 jmp loc\_555555554D13
.text:0000555555554BFC ; ---------------------------------------------------------------------------
.text:0000555555554BFC
.text:0000555555554BFC loc\_555555554BFC: ; CODE XREF: challenge2+33↑j
.text:0000555555554BFC mov rax, cs:qword\_55555555567E
.text:0000555555554C03 mov qword ptr [rbp+s], rax
.text:0000555555554C07 movzx eax, cs:word\_555555555686
.text:0000555555554C0E mov [rbp+var\_32], ax
.text:0000555555554C12 mov rax, cs:qword\_555555555688
.text:0000555555554C19 mov rdx, cs:qword\_555555555690
.text:0000555555554C20 mov qword ptr [rbp+var\_30], rax
.text:0000555555554C24 mov qword ptr [rbp+var\_30+8], rdx
.text:0000555555554C28 mov [rbp+var\_1D0], 0
.text:0000555555554C32 mov [rbp+var\_1CC], 0
.text:0000555555554C3C jmp short loc\_555555554C6D
.text:0000555555554C3E ; ---------------------------------------------------------------------------
.text:0000555555554C3E
.text:0000555555554C3E loc\_555555554C3E: ; CODE XREF: challenge2+CF↓j
.text:0000555555554C3E mov eax, [rbp+var\_1C8]
.text:0000555555554C44 cdqe
.text:0000555555554C46 movzx edx, [rbp+rax+name.sysname]
.text:0000555555554C4E mov eax, [rbp+var\_1C8]
.text:0000555555554C54 cdqe
.text:0000555555554C56 movzx eax, [rbp+rax+s]
.text:0000555555554C5B cmp dl, al
.text:0000555555554C5D jnz short loc\_555555554C66
.text:0000555555554C5F add [rbp+var\_1D0], 1
.text:0000555555554C66
.text:0000555555554C66 loc\_555555554C66: ; CODE XREF: challenge2+A7↑j
.text:0000555555554C66 add [rbp+var\_1C8], 1
.text:0000555555554C6D
.text:0000555555554C6D loc\_555555554C6D: ; CODE XREF: challenge2+86↑j
.text:0000555555554C6D mov eax, [rbp+var\_1C8]
.text:0000555555554C73 movsxd rbx, eax
.text:0000555555554C76 lea rax, [rbp+s]
.text:0000555555554C7A mov rdi, rax ; s
.text:0000555555554C7D call \_strlen
.text:0000555555554C82 cmp rbx, rax ; 相等，则ZF=1, CF=0, OF=0
; 小于，则ZF=0, CF=1, OF=0
; 大于，则ZF=0, CF=0, OF=0
.text:0000555555554C85 jb short loc\_555555554C3E ; 前一条指令执行的结果为小于则跳转
.text:0000555555554C87 jmp short loc\_555555554CB8 ; 此处跳出循环
.text:0000555555554C89 ; ---------------------------------------------------------------------------
.text:0000555555554C89
.text:0000555555554C89 loc\_555555554C89: ; CODE XREF: challenge2+11A↓j
.text:0000555555554C89 mov eax, [rbp+var\_1C4]
.text:0000555555554C8F cdqe
.text:0000555555554C91 movzx edx, [rbp+rax+name.version]
.text:0000555555554C99 mov eax, [rbp+var\_1C4]
.text:0000555555554C9F cdqe
.text:0000555555554CA1 movzx eax, [rbp+rax+var\_30]
.text:0000555555554CA6 cmp dl, al
.text:0000555555554CA8 jnz short loc\_555555554CB1
.text:0000555555554CAA add [rbp+var\_1CC], 1
.text:0000555555554CB1
.text:0000555555554CB1 loc\_555555554CB1: ; CODE XREF: challenge2+F2↑j
.text:0000555555554CB1 add [rbp+var\_1C4], 1
.text:0000555555554CB8
.text:0000555555554CB8 loc\_555555554CB8: ; CODE XREF: challenge2+D1↑j
.text:0000555555554CB8 mov eax, [rbp+var\_1C4]
.text:0000555555554CBE movsxd rbx, eax ; 将eax寄存器中的32位符号扩展为64位，并将结果存储到rbx寄存器中
.text:0000555555554CC1 lea rax, [rbp+var\_30]
.text:0000555555554CC5 mov rdi, rax ; s
.text:0000555555554CC8 call \_strlen
.text:0000555555554CCD cmp rbx, rax
.text:0000555555554CD0 jb short loc\_555555554C89
.text:0000555555554CD2 mov ebx, [rbp+var\_1D0]
.text:0000555555554CD8 lea rax, [rbp+s]
.text:0000555555554CDC mov rdi, rax ; s
.text:0000555555554CDF call \_strlen
.text:0000555555554CE4 cmp rbx, rax
.text:0000555555554CE7 jnz short loc\_555555554D13
.text:0000555555554CE9 mov ebx, [rbp+var\_1CC]
.text:0000555555554CEF lea rax, [rbp+var\_30]
.text:0000555555554CF3 mov rdi, rax ; s
.text:0000555555554CF6 call \_strlen
.text:0000555555554CFB cmp rbx, rax
.text:0000555555554CFE jnz short loc\_555555554D13
.text:0000555555554D00 cmp [rbp+var\_1D0], 5
.text:0000555555554D07 jbe short loc\_555555554D13
.text:0000555555554D09 mov rax, [rbp+var\_1D8]
.text:0000555555554D10 mov byte ptr [rax], 1
.text:0000555555554D13
.text:0000555555554D13 loc\_555555554D13: ; CODE XREF: challenge2+41↑j
.text:0000555555554D13 ; challenge2+131↑j ...
.text:0000555555554D13 mov rax, [rbp+var\_18]
.text:0000555555554D17 xor rax, fs:28h
.text:0000555555554D20 jz short loc\_555555554D27
.text:0000555555554D22 call \_\_\_stack\_chk\_fail
.text:0000555555554D27 ; ---------------------------------------------------------------------------
.text:0000555555554D27
.text:0000555555554D27 loc\_555555554D27: ; CODE XREF: challenge2+16A↑j
.text:0000555555554D27 add rsp, 1D8h
.text:0000555555554D2E pop rbx
.text:0000555555554D2F pop rbp
.text:0000555555554D30 retn
.text:0000555555554D30 challenge2 endp
```

正确的运行逻辑： `challenge2` -> `loc_555555554BFC` -> `loc_555555554C6D` -> `loc_555555554CB8` -> `loc_555555554D13` -> `loc_555555554D27`

对应伪代码：

```c
unsigned \_\_int64 \_\_fastcall challenge2(\_BYTE \*a1)
{
unsigned int v2; // [rsp+10h] [rbp-1D0h]
int v3; // [rsp+14h] [rbp-1CCh]
int v4; // [rsp+18h] [rbp-1C8h]
int v5; // [rsp+1Ch] [rbp-1C4h]
struct utsname name; // [rsp+20h] [rbp-1C0h] BYREF
char s[10]; // [rsp+1A6h] [rbp-3Ah] BYREF
char v8[24]; // [rsp+1B0h] [rbp-30h] BYREF
unsigned \_\_int64 v9; // [rsp+1C8h] [rbp-18h]
v9 = \_\_readfsqword(0x28u);
if ( uname(&name) )
{
perror("uname");
}
else
{
strcpy(s, "QilingOS");
strcpy(v8, "ChallengeStart");
v2 = 0;
v3 = 0;
while ( v4 < strlen(s) )
{
if ( name.sysname[v4] == s[v4] )
++v2;
++v4;
}
while ( v5 < strlen(v8) )
{
if ( name.version[v5] == v8[v5] )
++v3;
++v5;
}
if ( v2 == strlen(s) && v3 == strlen(v8) && v2 > 5 )
\*a1 = 1;
}
return \_\_readfsqword(0x28u) ^ v9;
}
```

第一个if判断：

```c
if ( uname(&name) )
{
perror("uname");
}
```

`uname(&name)` 是一个系统调用函数，它的作用是获取当前系统的名称和版本信息，并将这些信息存储到 `struct utsname` 类型的结构体变量 `name` 中，`name` 使用 `struct utsname name;` 声明，如果成功获取的话函数返回值为0，就可以进入else了。else中我们最后需要执行 `*a1 = 1`，这里面有两个循环进行字符串逐个判断，如果相同就能在最后的if判断中让 `*a1 = 1`。此处我们需要了解 `uname` 的结构体，`uname` 的结构为：

```c
struct utsname {
char sysname[65];
char nodename[65];
char release[65];
char version[65];
char machine[65];
char domainname[65];
};
```

我们需要让 `name.sysname` 等于 `"QilingOS"`，`name.version` 等于 `"ChallengeStart"`。

出题人提供了源码，可以看到源码的逻辑跟分析的一致：

```c
void challenge2(char \*check) {
unsigned int i, j, k, l;
struct utsname name;
char qiling\_OS[10];
char chall\_start[24];
if ( uname(&name) ) {
perror("uname");
}
else {
strcpy(qiling\_OS, "QilingOS");
strcpy(chall\_start, "ChallengeStart");
i = 0;
j = 0;
while ( k < strlen(qiling\_OS) ) {
if ( name.sysname[k] == qiling\_OS[k] )
++i;
++k;
}
while ( l < strlen(chall\_start) ) {
if ( name.version[l] == chall\_start[l] )
++j;
++l;
}
if ( i == strlen(qiling\_OS) && j == strlen(chall\_start) && i > 5 )
\*check = 1;
}
}
```

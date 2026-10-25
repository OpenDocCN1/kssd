
# 注册码逆向分析

## 反汇编代码与计算过程

```text
.text:100069D5                movsx
edi, byte ptr [ecx+offset aEqualizer] ; "equalizer"
.text:100069DC                add
  edi, edx
.text:100069DE                inc
  ecx
.text:100069DF                mov
  [ebp+arg\_0], edi
.text:100069E2                cmp
  ecx, eax
.text:100069E4                fild
  [ebp+arg\_0]
.text:100069E7                fsubr
[ebp+var\_8]
.text:100069EA                fstp
  [ebp+var\_8]
.text:100069ED                jl
    short loc\_100069D5

100069D5-100069ED的计算过程：

for i=1 to strlen("equalizer");Code2=Code2+Username(1)+"equalizer"(i);next

RegCode2=RegCode1-Code2

.text:100069EF                pop
  edi
.text:100069F0                fld
  [ebp+var\_8]
.text:100069F3                fsub
  ds:dbl\_1000A8F0
.text:100069F9                push
  esi
.text:100069FA                fst
  [ebp+var\_8]

100069D5-100069ED的计算过程：

RegCode2=RegCode2-3419821

.text:100069FD                fadd
  [ebp+var\_18]
.text:10006A00                fstp
  [ebp+var\_20]
.text:10006A03                call
  strlen
.text:10006A08                pop
  ecx
.text:10006A09                xor
  ecx, ecx
.text:10006A0B                test
  eax, eax
.text:10006A0D                jle
  short loc\_10006A37
.text:10006A0F                fld
  [ebp+var\_18]
.text:10006A12                fadd
  ds:dbl\_1000A8E8    ====> dbl\_1000A8E8=1
              ====> st0=(56+2\*strlen(Username))+1
.text:10006A18                movsx
edx, byte ptr [ecx+esi]
.text:10006A1C                mov
  [ebp+arg\_0], edx
.text:10006A1F                inc
  ecx
.text:10006A20                fild
  [ebp+arg\_0]
.text:10006A23                cmp
  ecx, eax
.text:10006A25                fmul
  st, st(1)
.text:10006A27                fadd
  [ebp+var\_20]
.text:10006A2A                fsub
  ds:dbl\_1000A8E0    ====> dbl\_1000A8E0=5
.text:10006A30                fstp
  [ebp+var\_20]
.text:10006A33                jl
    short loc\_10006A18

100069D5-100069ED的计算过程：

RegCode3=RegCode2+(56+2\*strlen(Username))

for i=1 to strlen(Username);RegCode3=RegCode3+Username(i)\*((56+2\*strlen(Username))+1)-5;next

.text:10006A35                fstp
  st
.text:10006A37                fld
  [ebp+var\_20]
.text:10006A3A                push
  ecx
.text:10006A3B                push
  ecx
.text:10006A3C                fstp
  [esp+158h+var\_158]
.text:10006A3F                fld
  [ebp+var\_8]
.text:10006A42                push
  ecx
.text:10006A43                push
  ecx
.text:10006A44                fstp
  [esp+160h+var\_160]
.text:10006A47                fld
  [ebp+var\_10]
.text:10006A4A                call
  \_ftol
.text:10006A4F                push
  eax
.text:10006A50                lea
  eax, [ebp+var\_14C]
.text:10006A56                push
  offset aX\_0f\_0f ; "%X-%.0f-%.0f"    ====> 注册码格式("0XRegCode1-RegCode2-RegCode3")
.text:10006A5B                push
  eax
.text:10006A5C                call
  ds:sprintf
.text:10006A62                push
  [ebp+arg\_4]
.text:10006A65                lea
  eax, [ebp+var\_14C]
.text:10006A6B                push
  eax
.text:10006A6C                call
  strcmp            ====> 注册码比较
.text:10006A71                add
  esp, 24h
.text:10006A74                neg
  eax
.text:10006A76                sbb
  eax, eax
.text:10006A78                pop
  esi
.text:10006A79                inc
  eax                ====> 结果放到eax
.text:10006A7A                leave
.text:10006A7B                retn
.text:10006A7B sub\_10006958    endp
```

## 参数值

```text
.rdata:1000A8E0 dbl\_1000A8E0    dq 5.0
          ; DATA XREF: sub\_10006958+D2
.rdata:1000A8E8 dbl\_1000A8E8    dq 1.0
          ; DATA XREF: sub\_10006958+BA
.rdata:1000A8F0 dbl\_1000A8F0    dq 3.419821e6
    ; DATA XREF: sub\_10006958+5C
.rdata:1000A8F8 dbl\_1000A8F8    dq 2.0
          ; DATA XREF: sub\_10006958+4C
.rdata:1000A900 dbl\_1000A900    dq 5.6e1
        ; DATA XREF: sub\_10006958+9
```

## 注册码总结

注册码由3部分组成：分别为`RegCode1`、`RegCode2`、`RegCode3`。

```text
RegCode1=3419821+求和((用户名的第1个字符+用户名各字符的ASCII值)*(56+2*用户名各字符的位置))

RegCode2=RegCode1-3419821-求和(用户名的第1个字符+"equalizer"各字符的ASCII值)+(56+2*用户名长度+1)

RegCode3=求和((RegCode2+56+2*用户名长度)+用户名各字符的ASCII值*(56+2*用户名长度+1)-5)
```

注册码的第1部分用16进制显示，第2、3部分用十进制显示，各部分之间用`"-"`连上。

注册器已做成

## 后记

后记：`IDA`和`W32Dasm`相比较，各有各的优点，`W32Dasm`比较简单，上手快，字符串参考窗非常实用，可以说就是专为`Cracker`定制，但对于`MFC`函数支持不足，并且有一些如本例子的参数(在`rdata`段)并不能显示出来，要想得出注册码的计算过程，需要结合`SoftICE`，要想修改程序代码进行暴破，还须结合`HIEW`、`HexWorkshop`、`UE`等；而`IDA`相对来说就难懂一些，但它支持`MFC`函数，这样程序读起来就要容易，另外，就向本程序那样，查看注册码的计算参数很容易，当然，它的功能不止这些。对于一个程序，要想破解它，应该结合具体情况来选择不同的反编译器，通常首先选择`W32Dasm`――容易、直观，而对于象由`VC++`等编译的程序以及注册部分有内部参数计算的，最好还是用`IDA`，以上是我的浅见，不当指出请予指出。

2001.05.26 15:00

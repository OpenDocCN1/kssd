
# Teleport Pro 1.29

> 原文：[`www.pediy.com/kssd/pediy01/forum069.htm`](https://www.pediy.com/kssd/pediy01/forum069.htm)

- **最新版本**：`Teleport Pro 1.29 Build 1422`
- **文件大小**：`844KB`
- **软件授权**：共享软件
- **使用平台**：`Win95/98/NT`
- **发布公司**：`Home Page`
- **下载地址**：`www.newhua.com.cn`

## 软件简介

`Teleport Pro` 不仅能离线浏览网页，还可以从 `Internet` 抓取任意文件，在指定时间自动登录网站下载内容，或创建网站完整镜像作为参考。`Teleport Pro` 未注册时仅有 30 天试用期，下面使用 `softice4.05` 和 `W32dasm` 进行破解。

### 1. 使用 W32dasm 分析

使用 `W32dasm` 打开 `c:\progra~1\teleport pro\pro.exe`，反汇编并查找字符串：

```
"Thank you!  Your copy of Teleport Pro is now registered."
```

找到如下代码：

```
* Possible Reference to
String Resource ID=07028: "Thank you!  Your copy of Teleport Pro is now registered.  Al"

:00425F78 68741B0000               push 00001B74
:00425F7D 8D4DF0                   lea ecx, dword ptr [ebp-10]
:00425F80 895DFC                   mov dword ptr [ebp-04], ebx
............
```

向上查看相关代码（注释在右侧）：

```
:00425F29 8B87DD000000             mov eax, dword ptr [edi+000000DD]
:00425F2F 33DB                     xor ebx, ebx
:00425F31 6A0A                     push 0000000A
:00425F33 53                       push ebx
:00425F34 50                       push eax
:00425F35 E85B690000              call 0042C895
:00425F3A 8B0D68F04700            mov ecx, dword ptr [0047F068]
:00425F40 83C40C                   add esp, 0000000C
:00425F43 8945E8                   mov dword ptr [ebp-18], eax
:00425F46 3899DB040000             cmp byte ptr [ecx+000004DB], bl
:00425F4C 0F8412020000             je 00426164
:00425F52 3BC3                     cmp eax, ebx

* Possible StringData Ref from Data Obj ->"User"
:00425F54 BEB8C94700              mov esi, 0047C9B8
:00425F59 0F8406010000             je 00426065
:00425F5F FFB7D5000000            push dword ptr [edi+000000D5]
:00425F65 E896090000              call 00426900          <==干什么？F8 追入。
:00425F6A 3945E8                   cmp dword ptr [ebp-18], eax  <==比较什么？
:00425F6D 59                       pop ecx
:00425F6E 753A                     jne 00425FAA         <--此处不跳，胜利在望，跳转则死！
:00425F70 A114D44700              mov eax, dword ptr [0047D414]
:00425F75 8945F0                   mov dword ptr [ebp-10], eax
```

### 2. 使用 softice 动态分析

使用 `ice` 载入 `pro.exe`，设置断点：

```
:bpx 00425f29
:g
```

在程序中填入注册信息：

```
name: yubing
company: [CCG]
registration: 78787878
```

按“确定”后，程序被拦截在：

```
:00425F29 8B87DD000000             mov eax, dword ptr [edi+000000DD]  <--78787878 送 eax
:00425F2F 33DB                     xor ebx, ebx
:00425F31 6A0A                     push 0000000A
:00425F33 53                       push ebx
:00425F34 50                       push eax  <--? eax 可知 eax 中存放 78787878 的十六进制 4b23526
:00425F35 E85B690000              call 0042C895
:00425F3A 8B0D68F04700            mov ecx, dword ptr [0047F068]
:00425F40 83C40C                   add esp, 0000000C
:00425F43 8945E8                   mov dword ptr [ebp-18], eax    <--将 4b23526 送[ebp-18]
:00425F46 3899DB040000             cmp byte ptr [ecx+000004DB], bl
:00425F4C 0F8412020000             je 00426164
:00425F52 3BC3                     cmp eax, ebx
:00425F54 BEB8C94700              mov esi, 0047C9B8
:00425F59 0F8406010000             je 00426065
:00425F5F FFB7D5000000            push dword ptr [edi+000000D5]
:00425F65 E896090000              call 00426900          <==干什么？F8 追入。
:00425F6A 3945E8                   cmp dword ptr [ebp-18], eax  <==什么与 4b23526 比较呀？
:00425F6D 59                       pop ecx
:00425F6E 753A                     jne 00425FAA         <--此处不跳，胜利在望，跳转则死！
:00425F70 A114D44700              mov eax, dword ptr [0047D414]
:00425F75 8945F0                   mov dword ptr [ebp-10], eax
```

### 3. 深入分析 call 00426900

在 `call 00426900` 处追入，按 `F10` 继续执行，直到：

```
:00426914 83F805                   cmp eax, 00000005   <==比较输入注册名的个数是否大于 5
:00426917 7304                     jnb 0042691D           不大于 5，则提示错误。
:00426919 33C0                     xor eax, eax
:0042691B 5F                       pop edi
:0042691C C3                       ret
:0042691D 53                       push ebx
:0042691E 56                       push esi
:0042691F BEA4E4FE5D              mov esi, 5DFEE4A4      <--------|将输入的
:00426924 33DB                     xor ebx, ebx               |name，变
:00426926 85FF                     test edi, edi              |换，得到
:00426928 7409                     je 00426933                |正确的注
:0042692A 57                       push edi                 |册码,有兴趣
:0042692B E8F0560000              call 0042C020              |的话，可
:00426930 59                       pop ecx                  |以制出注册
:00426931 EB02                     jmp 00426935               |机!
:00426933 33C0                     xor eax, eax
:00426935 83C0FC                   add eax, FFFFFFFC
:00426938 3BD8                     cmp ebx, eax
:0042693A 730C                     jnb 00426948
:0042693C 33343B                   xor esi, dword ptr [ebx+edi]
:0042693F F6C340                   test bl, 40
:00426942 7401                     je 00426945
:00426944 43                       inc ebx
:00426945 43                       inc ebx
:00426946 EBDE                     jmp 00426926           <---------|
:00426948 8BC6                     mov eax, esi     <--将得到的正确的注
                                                        册码送 eax 中
:0042694A 5E                       pop esi
:0042694B 5B                       pop ebx
:0042694C 5F                       pop edi
:0042694D C3                       ret
```

### 4. 获取正确注册码

在 `:00425F6A cmp dword ptr [ebp-18], eax` 处观察 `eax`，即可看到正确的注册码。记下该十进制数值，用于注册。

**破解者**：十三少 `[CCG]`  
**团体**：`China Cracking Group`  
**日期**：`2000.05.27`

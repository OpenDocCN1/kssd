
# WinRescure 98 3.07

> 原文：[`www.pediy.com/kssd/pediy03/forum1192.htm`](https://www.pediy.com/kssd/pediy03/forum1192.htm)

**破解名称**：`WinRescure 98 3.07`

**软件说明**：系统恢复，可以抢救当机前工作的记录。

**工具**：`TRW2000 1.23`

**软件限制**：未注册版本运行时有 10 个数字倒记时，等的心烦。注册形式是填入固定的代码，没有用户名。

**破解过程**：用 `trw` 载入程序，……（此处省去 N 字），跑到这里：

....

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:004681E5(C)`

`|`

`:00468254 8B00 mov eax, dword ptr [eax]`
`:00468256 33D2 xor edx, edx`
`:00468258 E85F47FCFF call 0042C9BC`
`:0046825D A120384800 mov eax, dword ptr [00483820]`
`:00468262 E8C160FCFF call 0042E328`
`:00468267 8D55F4 lea edx, dword ptr [ebp-0C]`
`:0046826A 8B83F4010000 mov eax, dword ptr [ebx+000001F4]`
`:00468270 E8BF82FBFF call 00420534`
`:00468275 837DF400 cmp dword ptr [ebp-0C], 00000000`
`:00468279 7462 je 004682DD`
`:0046827B 8D55F4 lea edx, dword ptr [ebp-0C]`
`:0046827E 8B83F4010000 mov eax, dword ptr [ebx+000001F4]`
`:00468284 E8AB82FBFF call 00420534`
`:00468289 8B55F4 mov edx, dword ptr [ebp-0C]`

`* Possible StringData Ref from Code Obj -> "SvetCHRISTA"` `<===注册码 ^\_^`

```
:0046828C B870834600 mov eax, 00468370
:00468291 E81EBDF9FF call 00403FB4 <===下 EDX 见假的注册码，下 EAX 见真的注册码
:00468296 85C0 test eax, eax
:00468298 742D je 004682C7 <===比较注册码，不等就跳到失败处
:0046829A 33D2 xor edx, edx
:0046829C 8B83F4010000 mov eax, dword ptr [ebx+000001F4]
:004682A2 E8BD82FBFF call 00420564
:004682A7 A1A42A4800 mov eax, dword ptr [00482AA4]
:004682AC 8B00 mov eax, dword ptr [eax]
:004682AE 8B80F0010000 mov eax, dword ptr [eax+000001F0]
:004682B4 C7400C64000000 mov [eax+0C], 00000064

* Possible StringData Ref from Code Obj -> "Registration Key Accepted"
"you for registering."
<===注册成功信息

:004682BB B884834600 mov eax, 00468384
:004682C0 E89751FDFF call 0043D45C
:004682C5 EB16 jmp 004682DD

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00468298(C)
|

* Possible StringData Ref from Code Obj -> "WARNING - Incorrect Key Entered"
"WinRescue "
"98 may be used without entering "
"a Key"
"WinRescue 98 Terminating."
<===注册失败信息
```

........

**整理**：注册码为 `SvetCHRISTA`（注意大小写），或者修改 `85C0742D` 为 `85C0752D`。 `4dd466d0e1e0cc123afbe504da2462ca`

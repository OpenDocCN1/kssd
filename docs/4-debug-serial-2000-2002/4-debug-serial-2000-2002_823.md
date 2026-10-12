
# Icontoy 2

> 原文：[`www.pediy.com/kssd/pediy04/kanxue464.htm`](https://www.pediy.com/kssd/pediy04/kanxue464.htm)

Crack 之亲历手迹 2--我的第一个注册机

```
//-------------------------------------------------
目标程序：icontoy2.exe：392 KB (401,920 字节)
            绿色软件，单独 exe 文件
            Delphi 编写无需 pakage 支持，未加壳
破解目标：找到注册码，最好做出注册机
//--------------------------------------------------
```

## 破解过程

1. 用 `IDA` 反汇编。
2. 用 `SoftICE`，下 `bpx getwindowtext`
   ```
   bpx getwindowtexta
   bpx getwindowtextw
   bpx getdlgitemtext
   bpx getdlgitemtexta
   bpx getdlgitemtextw
   ```
   结果只栏到 `TApplication::GetTitle` 中的程式，并未拦截到程序获取注册码的过程。
   ???或许 `Delphi` 的 `TControl::GetText` 另有函数？(详细见 2002-2-15 日记）
3. 用 `W32Dasm` 反汇编，找到 `Registration key error!`字符串，点击后直接进入了 `OnOK` 的处理函数，That's OK！
4. 在 `IDA` 中找到这个处理函数，分析发现：
   首先打开注册表，将用户名、`Key` 写入注册表
   ```
   mov     edx, offset _str__software_akato_0.Text
   mov     eax, ebx
   call    @Registry@TRegistry@OpenKey$qqrx17System@AnsiString
   lea     edx, [ebp+var_4]
   mov     eax, [esi+270h]
   call    @TControl@GetText ; TControl::GetText
   mov     ecx, [ebp+var_4]
   mov     edx, offset _str_reg_key_0.Text
   mov     eax, ebx
   call    @TRegistry@WriteString ; TRegistry::WriteString
   ```
   然后再从注册表中读出来
   ```
   lea     ecx, [ebp+var_8]
   mov     edx, offset _str_reg_key_0.Text
   mov     eax, ebx
   call    @TRegistry@ReadString ; TRegistry::ReadString
   mov     edx, [ebp+var_8]
   mov     eax, edi
   call    @System@@LStrAsg$qqrv ; System __linkproc__ LStrAsg(void)
   ```
   再与十个可能的注册码一一比较
   ```
   mov     eax, [edi]
   mov     edx, offset _str_R3212959_0.Text
   call    @System@@LStrCmp$qqrv ; System __linkproc__ LStrCmp(void)
   jz      short loc_439CEA
   mov     eax, [edi]
   mov     edx, offset _str_8M6M8DTA_0.Text
   call    @System@@LStrCmp$qqrv ; System __linkproc__ LStrCmp(void)
   jnz     short loc_439D36
   ```
   如是十个注册码中的一个则显示注册成功，或则显示注册失败
   ```
   mov     eax, offset _str_Registration_ke.Text
   call    @Dialogs@ShowMessage$qqrx17System@AnsiString ; Dialogs::ShowMessage(System::AnsiString)
   ```
   现在我们可以知道他的注册码了，只能为下面的十个字串中的一个

   ```
   8M6M8DTA
   R3212959
   12959TAT
   322223XL
   reggy322
   reggy105
   reg10515
   reg15810
   22322322
   32222332
   ```

5. 使用 `VC++6.0` 做出注册机

`Email:y97523@hotmail.com`  `4dd466d0e1e0cc123afbe504da2462ca`

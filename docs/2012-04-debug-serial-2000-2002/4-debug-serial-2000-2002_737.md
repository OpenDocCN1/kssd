
# 隐藏目录的加密光盘

> 原文：[`www.pediy.com/kssd/pediy03/forum743.htm`](https://www.pediy.com/kssd/pediy03/forum743.htm)

现在有很多光盘都加了密，从光盘属性得知共有 640M 的文件，但是直接查看光盘只有几十个小文件，合计不到 40M，剩余的 600M 文件隐藏到什么地方去了？

这些使用隐藏目录方式加密的光盘，必须使用“冷雨浏览器”，而且必须输入密码。如果密码正确，就会自动打开一个资源管理器窗口，显示光盘上隐藏的文件夹。

一直没有留意过这一类光盘，因为据我所知这类光盘的内容相当多都不太正经。而我对此不感兴趣。

刚刚买了张 Delphi 专辑光盘，就属于这类隐藏目录的加密光盘。不知是何原因，连 Delphi 都加密！

封皮上写明运行序列号为"658888"，打开光盘，输入序列号，提示"密码错误"。Kao!做 D 版的越来越没有敬业精神了，连这个小问题也出错。

用 `GetType` 检测 `Setup.exe` 得知程序是用 Delphi 编写的。那就用 Delphi 的专用反编译软件 `DeDe` 破解吧！

1. 启动 `DeDe`，打开 `setup.exe` 后，单击 `Process` 开始反编译
2. `DeDe` 弹出对话框，提示"如果目标全部加载完毕，请单击确定"
3. 单击确定，很快 `DeDe` 说"转储完毕"，再次单击"确定"
4. 单击子例程(`Procedures`)按钮，得知程序有两个窗体(单元, `Unit`)
5. 查了下，知道 `setup1` 是接受口令输入的窗体
6. 在左侧的列表窗口选择 `setup2` 窗体，右侧的列表窗口出现与该窗体相关的子例程。
7. 双击右侧列表窗口内的 `B_OKClick` 事件子例程。

这表示显示在程序中输入口令后，单击确定按钮后执行的代码

```
00442D98  55                 push   ebp   
00442D99  8BEC               mov   ebp, esp   
00442D9B  33C9               xor   ecx, ecx   
00442D9D  51                 push   ecx   
00442D9E  51                 push   ecx   
00442D9F  51                 push   ecx   
00442DA0  51                 push   ecx   
00442DA1  51                 push   ecx   
00442DA2  53                 push   ebx   
00442DA3  8BD8               mov   ebx, eax   
00442DA5  33C0               xor   eax, eax   
00442DA7  55                 push   ebp   
00442DA8  68B92F4400         push   $00442FB9   

***** TRY   

|   

00442DAD  64FF30             push   dword ptr fs:[eax]   
00442DB0  648920             mov   fs:[eax], esp   
00442DB3  8D55F8             lea   edx, [ebp-$08]   

* Reference to control TForm2.Edit_Password : TEdit   

|   

00442DB6  8B83D4020000       mov   eax, [ebx+$02D4]   

* Reference to: controls.TControl.GetText(TControl):System.String;   

|   

00442DBC  E89B06FEFF         call   0042345C   
00442DC1  8B45F8             mov   eax, [ebp-$08]   
00442DC4  8D55FC             lea   edx, [ebp-$04]   

* Reference to: sysutils.Trim(System.AnsiString):System.AnsiString;   

|   

00442DC7  E80C49FCFF         call   004076D8   
00442DCC  837DFC00           cmp   dword ptr [ebp-$04], +$00   
00442DD0  750F               jnz   00442DE1   

* Possible String Reference to: '请输入序列号'   

|   

00442DD2  B8D02F4400         mov   eax, $00442FD0   

* Reference to: dialogs.ShowMessage(System.AnsiString);   

|   

00442DD7  E800FAFFFF         call   004427DC   
00442DDC  E9AD010000         jmp   00442F8E   

00442DE1  8B45FC             mov   eax, [ebp-$04] <==== 输入的假密码   

* Possible String Reference to: '238888'  <=======密码? Yes!   

|   

00442DE4  BAE82F4400         mov   edx, $00442FE8   

* Reference to: system.@LStrCmp;   


|   

00442DE9  E81A0EFCFF         call   00403C08 <====调用 system 单元的字符串比较函数   
00442DEE  0F85B8000000       jnz   00442EAC <====不相等则跳转   

00442DF4  6A00               push   $00   
00442DF6  8D45F4             lea   eax, [ebp-$0C]   
00442DF9  8A1569584400       mov   dl, byte ptr [$445869]   

* Reference to: system.@LStrFromChar(String;Char);   

|          or: system.@LStrFromWChar(String;WideChar);   

|          or: system.@WStrFromChar(WideString;Char);   

|          or: system.@WStrFromWChar(WideString;WideChar);   

|   

00442DFF  E81C0CFCFF         call   00403A20   
00442E04  8D45F4             lea   eax, [ebp-$0C]   

* Possible String Reference to: ':\Program Files\Accessories\200001' <===生成 Key 文件   

|   

00442E07  BAF82F4400         mov   edx, $00442FF8   

* Reference to: system.@LStrCat;   

|   

00442E0C  E8EF0CFCFF         call   00403B00   
00442E11  8B45F4             mov   eax, [ebp-$0C]   

* Reference to: system.@LStrToPChar;   

|   

00442E14  E8A30EFCFF         call   00403CBC   
00442E19  50                 push   eax   
00442E1A  8D55EC             lea   edx, [ebp-$14]   

* Reference to TApplication instance   

|   

00442E1D  A1944C4400         mov   eax, dword ptr [$444C94]   
00442E22  8B00               mov   eax, [eax]   

* Reference to: forms.TApplication.GetExeName(TApplication):System.AnsiString;   

|   

00442E24  E8F3C6FFFF         call   0043F51C   
00442E29  8B45EC             mov   eax, [ebp-$14]   
00442E2C  8A10               mov   dl, byte ptr [eax]   
00442E2E  8D45F0             lea   eax, [ebp-$10]   

* Reference to: system.@LStrFromChar(String;Char);   

|          or: system.@LStrFromWChar(String;WideChar);   

|          or: system.@WStrFromChar(WideString;Char);   

|          or: system.@WStrFromWChar(WideString;WideChar);   

|   

00442E31  E8EA0BFCFF         call   00403A20   
00442E36  8D45F0             lea   eax, [ebp-$10]   

* Possible String Reference to: ':\browse\aaa\200001'  <=== 隐藏的 Key 文件   

|   

00442E39  BA24304400         mov   edx, $00443024   

* Reference to: system.@LStrCat;   

|   

00442E3E  E8BD0CFCFF         call   00403B00   
00442E43  8B45F0             mov   eax, [ebp-$10]   

* Reference to: system.@LStrToPChar;   

|   

00442E46  E8710EFCFF         call   00403CBC   
00442E4B  50                 push   eax   

* Reference to: kernel32.CopyFileA()   

|   

00442E4C  E8AB2EFCFF         call   00405CFC   
00442E51  6A01               push   $01   
00442E53  6A00               push   $00   
00442E55  6A00               push   $00   
00442E57  8D55F0             lea   edx, [ebp-$10]   

* Reference to TApplication instance   

|   

00442E5A  A1944C4400         mov   eax, dword ptr [$444C94]   
00442E5F  8B00               mov   eax, [eax]   

* Reference to: forms.TApplication.GetExeName(TApplication):System.AnsiString;   

|   

00442E61  E8B6C6FFFF         call   0043F51C   
00442E66  8B45F0             mov   eax, [ebp-$10]   
00442E69  8A10               mov   dl, byte ptr [eax]   
00442E6B  8D45F4             lea   eax, [ebp-$0C]   

* Reference to: system.@LStrFromChar(String;Char);   

|          or: system.@LStrFromWChar(String;WideChar);   

|          or: system.@WStrFromChar(WideString;Char);   

|          or: system.@WStrFromWChar(WideString;WideChar);   

|   

00442E6E  E8AD0BFCFF         call   00403A20   
00442E73  8D45F4             lea   eax, [ebp-$0C]   

* Possible String Reference to: ':\browse\aaa\hcssee.exe'  <=== 看隐藏文件夹的主程序   

|   

00442E76  BA40304400         mov   edx, $00443040   

* Reference to: system.@LStrCat;   

|   

00442E7B  E8800CFCFF         call   00403B00   
00442E80  8B45F4             mov   eax, [ebp-$0C]   

* Reference to: system.@LStrToPChar;   

|   

00442E83  E8340EFCFF         call   00403CBC   
00442E88  50                 push   eax   

* Possible String Reference to: 'Open'   

|   

00442E89  6858304400         push   $00443058   
00442E8E  8BC3               mov   eax, ebx   

* Reference to: controls.TWinControl.GetHandle(TWinControl):Windows.HWND;   

|   

00442E90  E82363FEFF         call   004291B8   
00442E95  50                 push   eax   

* Reference to: shell32.ShellExecuteA()   

|   

00442E96  E859D3FFFF         call   004401F4   

* Reference to TApplication instance   

|   

00442E9B  A1944C4400         mov   eax, dword ptr [$444C94]   
00442EA0  8B00               mov   eax, [eax]   

* Reference to: forms.TApplication.Terminate(TApplication);   

|   

00442EA2  E861C2FFFF         call   0043F108   
00442EA7  E9E2000000         jmp   00442F8E   

00442EAC  8B45FC             mov   eax, [ebp-$04]   

* Possible String Reference to: '3456'       


|
```



00442EAF  BA68304400        mov    edx, $00443068   

`Reference to: system.@LStrCmp;`   

00442EB4  E84F0DFCFF        call   00403C08   

00442EB9  7558              jnz    00442F13   

00442EBB  6A01              push   $01   

00442EBD  6A00              push   $00   

00442EBF  6A00              push   $00   

00442EC1  8D55F0            lea    edx, [ebp-$10]   

`Reference to TApplication instance`   

00442EC4  A1944C4400        mov    eax, dword ptr [$444C94]   

00442EC9  8B00              mov    eax, [eax]   

`Reference to: forms.TApplication.GetExeName(TApplication):System.AnsiString;`   

00442ECB  E84CC6FFFF        call   0043F51C   

00442ED0  8B45F0            mov    eax, [ebp-$10]   

00442ED3  8A10              mov    dl, byte ptr [eax]   

00442ED5  8D45F4            lea    eax, [ebp-$0C]   

`Reference to: system.@LStrFromChar(String;Char);`   
or: `system.@LStrFromWChar(String;WideChar);`   
or: `system.@WStrFromChar(WideString;Char);`   
or: `system.@WStrFromWChar(WideString;WideChar);`   

00442ED8  E8430BFCFF        call   00403A20   

00442EDD  8D45F4            lea    eax, [ebp-$0C]   

`Possible String Reference to: ':\\browse\\setup.exe'`   

00442EE0  BA78304400        mov    edx, $00443078   

`Reference to: system.@LStrCat;`   

00442EE5  E8160CFCFF        call   00403B00   

00442EEA  8B45F4            mov    eax, [ebp-$0C]   

`Reference to: system.@LStrToPChar;`   

00442EED  E8CA0DFCFF        call   00403CBC   

00442EF2  50                push   eax   

`Possible String Reference to: 'Open'`   

00442EF3  6858304400        push   $00443058   

00442EF8  8BC3              mov    eax, ebx   

`Reference to: controls.TWinControl.GetHandle(TWinControl):Windows.HWND;`   

00442EFA  E8B962FEFF        call   004291B8   

00442EFF  50                push   eax   

`Reference to: shell32.ShellExecuteA()`   

00442F00  E8EFD2FFFF        call   004401F4   

`Reference to TApplication instance`   

00442F05  A1944C4400        mov    eax, dword ptr [$444C94]   

00442F0A  8B00              mov    eax, [eax]   

`Reference to: forms.TApplication.Terminate(TApplication);`   

00442F0C  E8F7C1FFFF        call   0043F108   

00442F11  EB7B              jmp    00442F8E   

00442F13  FF0564584400      inc    dword ptr [$445864] <==单击 Ok 的次数   

00442F19  833D6458440001    cmp    dword ptr [$445864], +$01 <=== 是第 2 次输入密码吗?   

00442F20  750C              jnz    00442F2E  <========== 是则跳转   

`Possible String Reference to: '该序列号无效，请查看输入的是否与 CD 封套上的序列号一致！'`   

00442F22  B894304400        mov    eax, $00443094   

`Reference to: dialogs.ShowMessage(System.AnsiString);`   

00442F27  E8B0F8FFFF        call   004427DC  <==== 提示"密码错!"   

00442F2C  EB60              jmp    00442F8E   

`Possible String Reference to: '欢迎使用图片浏览器的测试版！'`   

00442F2E  B8D4304400        mov    eax, $004430D4   

`Reference to: dialogs.ShowMessage(System.AnsiString);`   

00442F33  E8A4F8FFFF        call   004427DC  <==== 执行冷雨浏览器   

00442F38  6A01              push   $01   

00442F3A  6A00              push   $00   

00442F3C  6A00              push   $00   

00442F3E  8D55F0            lea    edx, [ebp-$10]   

`Reference to TApplication instance`   

00442F41  A1944C4400        mov    eax, dword ptr [$444C94]   

00442F46  8B00              mov    eax, [eax]   

`Reference to: forms.TApplication.GetExeName(TApplication):System.AnsiString;`   

00442F48  E8CFC5FFFF        call   0043F51C   

00442F4D  8B45F0            mov    eax, [ebp-$10]   

00442F50  8A10              mov    dl, byte ptr [eax]   

00442F52  8D45F4            lea    eax, [ebp-$0C]   

`Reference to: system.@LStrFromChar(String;Char);`   
or: `system.@LStrFromWChar(String;WideChar);`   
or: `system.@WStrFromChar(WideString;Char);`   
or: `system.@WStrFromWChar(WideString;WideChar);`   

00442F55  E8C60AFCFF        call   00403A20   

00442F5A  8D45F4            lea    eax, [ebp-$0C]   

`Possible String Reference to: ':\\browse\\setup.exe'`   

00442F5D  BA78304400        mov    edx, $00443078   

`Reference to: system.@LStrCat;`   

00442F62  E8990BFCFF        call   00403B00   

00442F67  8B45F4            mov    eax, [ebp-$0C]   

`Reference to: system.@LStrToPChar;`   

00442F6A  E84D0DFCFF        call   00403CBC   

00442F6F  50                push   eax   

`Possible String Reference to: 'Open'`   

00442F70  6858304400        push   $00443058   

00442F75  8BC3              mov    eax, ebx   

`Reference to: controls.TWinControl.GetHandle(TWinControl):Windows.HWND;`   

00442F77  E83C62FEFF        call   004291B8   

00442F7C  50                push   eax   

`Reference to: shell32.ShellExecuteA()`   

00442F7D  E872D2FFFF        call   004401F4   

`Reference to TApplication instance`   

00442F82  A1944C4400        mov    eax, dword ptr [$444C94]   

00442F87  8B00              mov    eax, [eax]   

`Reference to: forms.TApplication.Terminate(TApplication);`   

00442F89  E87AC1FFFF        call   0043F108   

00442F8E  33C0              xor    eax, eax   

00442F90  5A                pop    edx   

00442F91  59                pop    ecx   

00442F92  59                pop    ecx   

00442F93  648910            mov    fs:[eax], edx   

FINALLY

`Possible String Reference to: '[嬪]?'`   

00442F96  68C02F4400        push   $00442FC0   

00442F9B  8D45EC            lea    eax, [ebp-$14]   

00442F9E  BA03000000        mov    edx, $00000003   

`Reference to: system.@LStrArrayClr;`   

00442FA3  E8F808FCFF        call   004038A0   

00442FA8  8D45F8            lea    eax, [ebp-$08]   

`Reference to: system.@LStrClr(String);`   

00442FAB  E8CC08FCFF        call   0040387C   

00442FB0  8D45FC            lea    eax, [ebp-$04]   

`Reference to: system.@LStrClr(String);`   

00442FB3  E8C408FCFF        call   0040387C   

00442FB8  C3                ret   

00442FB9  E97E03FCFF        jmp    0040333C   

00442FBE  EBDB              jmp    00442F9B   

END

00442FC0  5B                pop    ebx   

00442FC1  8BE5              mov    esp, ebp   

00442FC3  5D                pop    ebp   

00442FC4  C3                ret   

### 8. 看明白了吗?

(摇头)我-------还是--------------不懂!

我倒!!!!!!!!!!

上面的程序用 pascal 表示如下:

```pascal
write('输入序列号: ');
readln(str_inputed_sn);

if string_compare(str_inputed_sn, '238888') then
  display_hidden_files()
else
begin
  int_compare_counter := int_compare_counter + 1;
  if (int_compare_counter >= 2) or string_compare(str_inputed_sn, '3456') then
    display_CoolRain_browser();
end;
```

### 9. 小结

本例的真正序列号是: `238888`

### 10. 做这张光盘的 JS 真是 TMD, 封皮上写着 Delphi 5 正式版, 里面却是 trial 版本

`4dd466d0e1e0cc123afbe504da2462ca`

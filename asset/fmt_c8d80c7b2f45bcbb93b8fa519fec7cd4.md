
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

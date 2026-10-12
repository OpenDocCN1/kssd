
# Delphi 逆向工程笔记

> 原文：[`www.pediy.com/kssd/pediy06/pediy6594.htm`](https://www.pediy.com/kssd/pediy06/pediy6594.htm)

一直是用 `TP/Delphi` 的。平时也喜欢玩玩汇编。没事破破软件玩，不过水平只能对付明码比较……faint

最近看见一个 `Keygen` 挺好玩的，可是人家不给模板，只好自己动手了。反了几天，挺有心得。于是就写下来，大牛们不要扔板砖。

如需 `ZT`，请先联系我

====================================================================

发现自己有个特别的爱好，喜欢收集 `KeyGen` 的模板。怎么说呢？感觉那些玩意既小巧又精致，特别是一些破解组织的 `KeyGen`，还有一些特效在里面。不过很明显，他们是不大会把模板放出来的。这样，只有试着从 `KeyGen` 的反汇编结果入手，尝试恢复出源代码了。所幸的是，为了减少最终文件的大小，`KeyGen` 一般是用 `SDK` 写的，所用语言不外乎 `ASM`、`Delphi`、`VC`。做个逆向还不算太难。

`OK`，先选个牺牲品。首先运行，出来个协议框，点同意，出主界面。脱壳，修复资源，用 `peid` 可以看出是 `Delphi` 的。用 `IDA` 反一下。

```asm
0000:00405370 start:
0000:00405370             push    ebp
0000:00405371             mov     ebp, esp
0000:00405373             add     esp, 0FFFFFFF4h
0000:00405376             mov     eax, offset dword_405338
0000:0040537B             call    @Sysinit@@InitExe$qqrv ; Sysinit::__linkproc__ InitExe(void)
0000:00405380             push    0
0000:00405382             call    GetModuleHandleA_0
0000:00405387             mov     ds:hInstance, eax

//注意，这是一个全局变量
0000:0040538C             push    1F1F1Fh
0000:00405391             call    CreateSolidBrush
0000:00405396             mov     ds:hbr, eax
0000:0040539B             push    70h
0000:0040539D             mov     eax, ds:hInstance
0000:004053A2             push    eax
0000:004053A3             call    LoadCursorA
0000:004053A8             mov     ds:hCursor, eax
0000:004053AD             push    64h
0000:004053AF             mov     eax, ds:hInstance
0000:004053B4             push    eax
0000:004053B5             call    LoadIconA
0000:004053BA             mov     ds:dword_4076E8, eax

//这里应该是把得到的 Handle 存放起来，IDA 有时有点问题的
0000:004053BF             push    offset sub_404DD4

//非常明显，这里是一个 Callback。还有其他的地方需要一个 sub 的地址吗？
//当然，这是指下面没有跟 ret 的情况！
0000:004053C4             xor     ecx, ecx
0000:004053C6             mov     edx, 3E8h
0000:004053CB             mov     eax, ds:hInstance
0000:004053D0             call    sub_403CCC

//跟进去！
0000:004053D5             cmp     ds:dword_4060B0, 0
0000:004053DC             jz      short loc_4053F4
0000:004053DE             push    offset sub_404A44
0000:004053E3             xor     ecx, ecx
0000:004053E5             mov     edx, 7D0h
0000:004053EA             mov     eax, ds:hInstance
0000:004053EF             call    sub_403CCC
0000:004053F4
0000:004053F4 loc_4053F4:
0000:004053F4             mov     eax, ds:hbr
0000:004053F9             push    eax
0000:004053FA             call    DeleteObject
0000:004053FF             push    0
0000:00405401             call    ExitProcess_0
0000:00405406             call    @System@@Halt0$qqrv ; System::__linkproc__ Halt0(void)
```

可以看到，开头和结尾的两个 `call` 很有意思，其实是 `System` 里的东西。这里可以看到，主程序很短，而且退出时用了 `ExitProcess` 而不是自然结束，有 5 个全局变量（在 `ds` 段里的）。`sub_403CCC` 和 `sub_404A44` 分别进去看看，是一段调用 `DialogBoxParamA` 的子程序。`OK`，`sub_403CCC` 看来就是协议框，而 `sub_404A44` 应该就是主对话框了。这里把 `sub_403CCC` 列出来：

```asm
0000:00403CCC sub_403CCC      proc near
0000:00403CCC
0000:00403CCC
0000:00403CCC lpDialogFunc    = dword ptr  8
0000:00403CCC
0000:00403CCC             push    ebp
0000:00403CCD             mov     ebp, esp
0000:00403CCF             push    ebx
0000:00403CD0             push    0               ; dwInitParam
0000:00403CD2             mov     ebx, [ebp+lpDialogFunc]

//就是 Callback
0000:00403CD5             push    ebx             ; lpDialogFunc
0000:00403CD6             push    ecx             ; hWndParent
0000:00403CD7             push    edx             ; lpTemplateName
0000:00403CD8             push    eax             ; hInstance
0000:00403CD9             call    DialogBoxParamA
0000:00403CDE             pop     ebx
0000:00403CDF             pop     ebp
0000:00403CE0             retn    4
0000:00403CE0 sub_403CCC      endp
```

结合上面调用此程序的代码，可以得到主程序的 `src`：

```delphi
Var
  h_Icon:HICON;
  h_Inst:HMODULE;
  h_Cur:hCursor;
  h_Brush:HBRUSH;
  Flag:Boolean;
Begin
  h_Inst:=GetModuleHandle(nil);
  h_Brush:=CreateSolidBrush($1F1F1F);
  h_Cur:=LoadCursor(h_Inst,MAKEINTRESOURCE(LINKCURSOR));
  h_Icon:=LoadIcon(h_Inst,MAKEINTRESOURCE(MAINICON));

  //显示协议对话框
  DialogBox(h_Inst,LPCTSTR(IDD_LICENSEDLG),0,@LicenseProc);

  If Flag Then DialogBox(h_Inst,LPCTSTR(IDD_MAINDLG),0,@MainProc);

  DeleteObject(h_Brush);

  //退出程序
  ExitProcess(0);
End.
```

有趣的地方出来了：`Flag` 没有赋值？答案是：在 `LicenseProc` 里！

`4dd466d0e1e0cc123afbe504da2462ca`

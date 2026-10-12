
# IDA sp-analysis failed 不能 F5 的 解决方案之(一)

> 原文：[`www.pediy.com/kssd/pediy12/140002.html`](https://www.pediy.com/kssd/pediy12/140002.html)

有时候用 `IDA` `F5` 的时候遇到带 `sp-analysis failed` 的函数会失败。

这只是一个极其简单的一个例子。希望大家能依此类推。也更希望大家把类似的代码放出来一起解决。原本是一朋友问我的，我只是顺便发出来。以后遇到类似情况百度就可以找到了。

代码：

```
.text:004015ED sub_4015ED      proc near               ; CODE XREF: sub_40116A+1A6p
.text:004015ED                                             ; sub_40116A+1C7p
.text:004015ED
.text:004015ED var_C             = byte ptr -0Ch
.text:004015ED var_8             = dword ptr -8
.text:004015ED var_4             = dword ptr -4
.text:004015ED arg_0             = dword ptr  10h
.text:004015ED
.text:004015ED                    push    ecx
.text:004015EE                    push    esi
.text:004015EF                    push    ebp
.text:004015F0                    mov     ebp, esp
.text:004015F2                    add     esp, 0FFFFFFF8h
.text:004015F5                    mov     ebx, offset String
.text:004015FA                    cmp     dword ptr [ebx], 0FFFFFFFFh
.text:004015FD                    jz      short loc_401606
.text:004015FF                    mov     eax, offset loc_401607
.text:00401604                    jmp     eax
.text:00401606 ; ---------------------------------------------------------------------------
.text:00401606
.text:00401606 loc_401606:                   ; CODE XREF: sub_4015ED+10j
.text:00401606                    nop
.text:00401607
.text:00401607 loc_401607:                   ; DATA XREF: sub_4015ED+12o
.text:00401607                    xor     eax, eax
.text:00401609                    mov     esi, [ebp+arg_0]
.text:0040160C                    mov     al, [esi+ebx]
.text:0040160F                    xor     ebx, ebx
.text:00401611                    push    eax
.text:00401612                    call    sub_4014BF
.text:00401617                    mov     [ebp+var_4], ebx
.text:0040161A                    call    sub_4014D5
.text:0040161F                    mov     [ebp+var_8], ebx
.text:00401622                    push    0
.text:00401624                    push    offset dword_403425
.text:00401629                    push    [ebp+var_4]
.text:0040162C                    push    [ebp+var_8]
.text:0040162F                    call    sub_401561
.text:00401634                    mov     eax, dword_403425
.text:00401639                    ror     eax, 8
.text:0040163C                    cmp     eax, 5
.text:0040163F                    jz      short loc_401649
.text:00401641                    push    eax
.text:00401642                    mov     eax, offset loc_40164A
.text:00401647                    jmp     eax
.text:00401649 ; ---------------------------------------------------------------------------
.text:00401649
.text:00401649 loc_401649:                   ; CODE XREF: sub_4015ED+52j
.text:00401649                    nop
.text:0040164A
.text:0040164A loc_40164A:                   ; DATA XREF: sub_4015ED+55o
.text:0040164A                    pop     eax
.text:0040164B                    xor     ah, [ebp+var_C]
.text:0040164E                    xor     al, [ebp+var_C]
.text:00401651                    xor     edx, edx
.text:00401653                    mov     bx, ax
.text:00401656                    mov     ecx, 4
.text:0040165B
.text:0040165B loc_40165B:                   ; CODE XREF: sub_4015ED+83j
.text:0040165B                    and     al, 0Fh
.text:0040165D                    cmp     al, 9
.text:0040165F                    jle     short loc_401663
.text:00401661                    add     al, 7
.text:00401663
.text:00401663 loc_401663:                   ; CODE XREF: sub_4015ED+72j
.text:00401663                    add     al, 30h
.text:00401665                    mov     dl, al
.text:00401667                    ror     edx, 8
.text:0040166A                    shr     bx, 4
.text:0040166E                    mov     al, bl
.text:00401670                    loop    loc_40165B
.text:00401672                    add     esp, 1Ch
.text:00401675                    pop     ebp
.text:00401676                    pop     esi
.text:00401677                    pop     ecx
.text:00401678                    retn
.text:00401678 sub_4015ED      endp ; sp-analysis failed
```

废话我就不多说了，造成不能 `F5` 的原因在于：

`.text:00401642 mov eax, offset loc_40164A`

`.text:00401647 jmp eax`

用 `OD` 打开之后 改掉代码在保存，然后重新用 `IDA` 打开 `F5` 就可以了，`sp-analysis failed` 也就消失了。

在 `OD` 里直接改成 `JMP 40164A`

分析之后不难发现不能 `F5` 的原因，就在于跳转。

一个函数里面的一个跳转在未知的情况下会出现不能 `F5`。

结果：

代码：

```
char __cdecl sub_4015ED(int a1)
{
  unsigned __int16 v1; // ax@1
  int v2; // edx@1
  signed int v3; // ecx@1
  unsigned __int16 v4; // bx@1
  char v5; // ST10_1@1
  int v6; // eax@1
  char v7; // ST10_1@1
  char v8; // al@2

  sub_4014BF(String[a1]);
  sub_4014D5(v5);
  sub_401561(0, 0, &dword_403425, 0);
  v6 = __ROR__(dword_403425, 8);
  HIBYTE(v1) = v7 ^ BYTE1(v6);
  LOBYTE(v1) = v7 ^ v1;
  v2 = 0;
  v4 = v1;
  v3 = 4;
  do
  {
    v8 = v1 & 0xF;
    if ( v8 > 9 )
      v8 += 7;
    LOBYTE(v2) = v1 + 48;
    v2 = __ROR__(v2, 8);
    v4 >>= 4;
    LOBYTE(v1) = v4;
    --v3;
  }
  while ( v3 );
  return v1;
}
```

还有一个问题

看看 `OPENFILENAMEA` 的声明：

`typedef struct tagOFN {  DWORD lStructSize;  HWND hwndOwner;  HINSTANCE hInstance;  LPCTSTR lpstrFilter;  LPTSTR lpstrCustomFilter;  DWORD nMaxCustFilter;  DWORD nFilterIndex;  LPTSTR lpstrFile;  DWORD nMaxFile;  LPTSTR lpstrFileTitle;  DWORD nMaxFileTitle;  LPCTSTR lpstrInitialDir;  LPCTSTR lpstrTitle;  DWORD Flags;  WORD nFileOffset;  WORD nFileExtension;  LPCTSTR lpstrDefExt;  LPARAM lCustData;  LPOFNHOOKPROC lpfnHook;  LPCTSTR lpTemplateName;  #if (_WIN32_WINNT >= 0x0500)  void * pvReserved;  DWORD dwReserved;  DWORD FlagsEx;  #endif // (_WIN32_WINNT >= 0x0500)  } OPENFILENAME, *LPOPENFILENAME;`

有个条件就是 `#if (_WIN32_WINNT >= 0x0500)` 的时候，结构多出三个 `DWORD`，结果大小为 `0X58H`，在 `MASM32` 中，没有这三个 `DWORD`，结果为 `0X4CH`，看看反汇编代码：

![](img/744acc8017109d3d00fdd21d96d05374.jpg)

如果按下 `F5` 就会酱紫：

![](img/c1bd46a265447f6116a90f19d6a42d79.jpg)

这个时候的解决方法：

`SHIFT+F9` 打开结构 `subview`，双击 `tagOFNA`，在最后三个成员上按 `U` 键，取消掉它们，结果如下：

![](img/a6117a40d3cf9bd227c2b503d6ad3006.jpg)

然后返回反汇编窗口，在函数名上按 `U`，然后按 `P` 键，现在按 `F5` 就可以啦！！

继续。。

`0xXXXXXX: call analysis failed` 的解决方案

原因是：反编译插件无法确定其中函数的参数个数。

代码：

```
push    4D8h
call    xxxx
push    eax
call    xxxPrint
add     esp, 8
```

在 `IDA` 里双击这个函数，进去之后

![](img/c9c71b25cdfa33a3974b988bf872a4b5.jpg)

按 `Y`，修改函数约定和参数个数....（`Y` 是 `IDA` 中的一个快捷键）

![](img/675240f971a118b9c481eed204e21ff0.jpg)

把函数 修改成 如图

![](img/7bd6392ba76e1024f74171a4648bd9f7.jpg)

然后就可以 `F5` 继续分析了。

这个函数本身是别人写的，格式化字符串的函数，像 `format` 参数本就是无法确定的。所以 `IDA` 没能识别出来。

我知道 这个函数有两个参数是固定的 后面是不确定的，所以 三个点 `...` 就 `ok` 啦

希望对某些人能有所帮助。

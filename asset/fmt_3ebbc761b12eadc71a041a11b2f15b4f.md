
# 尝试着给 winhex 增加二进制文件比较功能

> 原文：[`www.pediy.com/kssd/pediy10/78617.html`](https://www.pediy.com/kssd/pediy10/78617.html)

平时经常要对二进制文件进行比较，我比较喜欢使用 `WINHEX` 处理二进制文件，很希望它能够提供 `UE` 一样的二进制文件比较功能。

所以今天尝试给 `winhex` 增加这个功能。

## 1. 首先给 WINHEX 增加一个按扭。使用资源黑客。

```
引用: 2 MENU
LANGUAGE LANG_CHINESE, 0x2
{
POPUP "文件(&F)"
{
  MENUITEM "新建(&N)...\\tCtrl+N",  10
  MENUITEM "打开(&O)...\\tCtrl+O",  11
  MENUITEM "保存(&S)\\tCtrl+S",  12
  MENUITEM "另存为(&A)...",  13
  MENUITEM "比较文件",  7878                  //增加这个
```

## 2. 更改 winhex 原先的消息处理

```
引用:
00474F27  |> \8BC6          mov     eax, esi                     ; ESi 等于 ID
00474F29  |.  E8 CE24F9FF   call    004073FC
```

改成：

```
引用:
00474F27  |> \8BC6          mov     eax, esi                     ; ESi 等于 ID
00474F29      E8 B66F0B00   call    0052BEE4
```

`bp GetOpenFileNameA,运行`。来到这里的代码处：

```
引用:
004271EC  |.  8B0D 50395300 mov     ecx, dword ptr [533950]        ;  123.00541A1E
004271F2  |.  66:8B09       mov     cx, word ptr [ecx]
004271F5  |.  8B17          mov     edx, dword ptr [edi]
004271F7  |.  A1 74375300   mov     eax, dword ptr [533774]
004271FC  |.  E8 CF3A1000   call    0052ACD0

  0052ACD0  /$  0FB7C9        movzx   ecx, cx
  0052ACD3  |.  41            inc     ecx
  0052ACD4  |.  51            push    ecx                    ; /n
  0052ACD5  |.  52            push    edx                    ; |String2
  0052ACD6  |.  50            push    eax                    ; |String1
  0052ACD7  |.  E8 90C0EDFF   call    <jmp.&kernel32.lstrcpynA>        ; \lstrcpynA
  0052ACDC  \.  C3            retn
```

`EDX` 指向最近打开的文件完整路径名

`EAX=[533774]=00548D6C`

把文件的路径复制到了 `00548D6C` 位置。

往下翻翻，发现这个地址 `00548B6C` 始终指向最新打开的完整文件路径。

增加以下代码：

```
引用:
0052BEDD   .  63 6D 70 00      ascii   "cmp",0
0052BEE1      00               db      00
0052BEE2      00               db      00
0052BEE3      00               db      00
0052BEE4   $  60               pushad
0052BEE5   .  3D C61E0000      cmp     eax, 1EC6
0052BEEA   .  75 2B           jnz     short 0052BF17
0052BEEC   .  68 DDBE5200      push    0052BEDD                     ; /FileName = "cmp"
0052BEF1   .  E8 CEACEDFF      call    <jmp.&kernel32.LoadLibraryA>     ; \LoadLibraryA
0052BEF6   .  85C0             test    eax, eax
0052BEF8   .  74 1D            je      short 0052BF17
0052BEFA   .  68 DDBE5200      push    0052BEDD                     ; /ProcNameOrOrdinal = "cmp"
0052BEFF   .  50               push    eax                      ; |hModule
0052BF00   .  E8 4753EDFF      call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress
0052BF05   .  85C0             test    eax, eax
0052BF07   .^ 0F84 0A60FFFF     je      00521F17
0052BF0D   .  6A 00            push    0
0052BF0F   .  68 6C8B5400      push    00548B6C
0052BF14   .  FFD0             call    eax                         //调用 dll 里的函数 cmp
0052BF16   .  61               popad
0052BF17   >  E8 E2B4EDFF      call    004073FE
0052BF1C   .  C3               retn
```

我自己写了一个 dll，用来实现文件比较。编程老是学不会，见笑了。

```
引用:
 Unit1 in 'Unit1.pas' {Form1};      //增加一个窗体
 procedure cmp(file1,file2:pchar);stdcall;
 begin
 form1:=tform1.Create(nil);
 form1.Edit1.Text:=file1;
 form1.Edit2.Text:=file2;
 form1.Show ;
 end;
 exports cmp;
```

窗体里的代码：

```
引用: procedure myprint(p1,p2:pbyte);                              //比较 and 输出
var
x1,y1:integer;
x2,y2:integer;
i:byte;
begin
x1:=10;
y1:=10;
x2:=410;
y2:=10;
i:=0;
try
while true do
begin
if p1^<>p2^ then form1.listbox1.Canvas.Font.Color:=clred
else     form1.listbox1.Canvas.Font.Color:=clblack;
  form1.listbox1.Canvas.TextOut(x1,y1,inttohex(p1^,2));
  form1.listbox1.Canvas.TextOut(x2,y2,inttohex(p2^,2));
  x1:=x1+20;
  x2:=x2+20;
  i:=i+1;
  inc(p1);
  inc(p2);
  if i>=16 then
  begin
  i:=0;
  x1:=10;
  y1:=y1+20;
  x2:=410;
  y2:=y2+20;
  end;
end;
except
showmessage('完毕！');
end;
end;

procedure TForm1.Button3Click(Sender: TObject);                //比较按妞的事件
var
file1,file2:file;
p1,p2:pbyte;
size1,size2:cardinal;
begin
if (fileexists(edit1.Text)) and   (fileexists(edit2.Text))  then
begin
  assignfile(file1,edit1.Text);
  assignfile(file2,edit2.Text );
  try
  reset(file1,1);
  reset(file2,1);
  size1:=filesize(file1);
  size2:=filesize(file2);
     try
      getmem(p1,size1);
      getmem(p2,size2);
     except
      showmessage('分配内存出错！');
      exit;
     end;
  blockread(file1,p1^,size1);
  blockread(file2,p2^,size2);
  //CreateThread(nil,0,@myprint,nil,0,threadid);
  myprint(p1,p2);
  freemem(p1);
  freemem(p2);
  finally
  closefile(file1);
  closefile(file2);
  end;
end
else
 showmessage('请先打开文件');
end;
```

## 说明&总结一下：

1. 我的 `winhex` 是 `12.75 SR-R6` 这个版本。
2. 程序里的 `listbox` 没有增加滚动条，当文件过长的时候，不能完整显示，我也不知道如何给 `listbox` 增加垂直滚动条。
3. 我的 `winhex` 从破解版变成未注册版，并且运行不稳定，经常出 BUG。
4. 希望高手整一个给我这个菜鸟用用。
5. 没能很好的选择已经打开的文件，象 `UE` 那样，只能选择一个

### 效果图：

![](img/7215584706631c2acee357bb36e3db6d.jpg)

### 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `cmpdll.rar` |
|---|---|

![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif") ![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif") ![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif") 原来 `winhex` 有文件比较的功能．

![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心") 这么老的帖子还有人顶啊．．．我忘记说了，我把 `Cmpfile.dll` 改成 `cmp.dll`

```
0052BEEC   .  68 DDBE5200      push    0052BEDD                     ; /FileName = "cmp"
0052BEF1   .  E8 CEACEDFF      call    <jmp.&kernel32.LoadLibraryA>     ; \LoadLibraryA
```

这样修改的时候就省了几个字节

`00548B6C` 始终指向最新打开的完整文件路径。

这个可以下 `GetOpenFileNameA` 跟踪出来，

好象会保存文件名到一个数组那样的结构里

![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸") 我现在主要搞数据库开发了，其实我很久不接触破解了

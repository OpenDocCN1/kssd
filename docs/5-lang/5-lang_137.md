
# Delphi 逆向工程笔记[4]

> 原文：[`www.pediy.com/kssd/pediy06/pediy6599.htm`](https://www.pediy.com/kssd/pediy06/pediy6599.htm)

```text
=================================================================================
```

下面处理 `WM_LBUTTONDOWN`。

```asm
0000:00404E79 loc_404E79:
0000:00404E79             movzx   eax, [ebp+lParam]
//取 lParam 的低 16 位，注意它用的是 movzx，和下面不同。
//开始静态分析没有看出来差别，还是 debug 了一下才明白。

0000:00404E7D             mov     [ebp+pt.x], eax
0000:00404E80             mov     eax, dword ptr [ebp+lParam]
0000:00404E83             call    sub_403C6C
//这个 call 实际是取 eax 的高 16 位

0000:00404E88             movzx   eax, ax
//截掉废的 bit

0000:00404E8B             mov     [ebp+pt.y], eax
0000:00404E8E             push    [ebp+pt.y]
0000:00404E91             push    [ebp+pt.x]      ; pt
0000:00404E94             push    offset sRect    ; lprc
0000:00404E99             call    PtInRect
//这个 API 有点意思，一般 API 传结构都是传地址，比如 sRect，但是它
//传 Point 就直接传值了。以前还真没有看到这样的 API。

0000:00404E9E             test    eax, eax
0000:00404EA0             jz      loc_404F8F
//返回 True 才继续

0000:00404EA6             push    0               ; lParam
0000:00404EA8             push    2               ; wParam
0000:00404EAA             push    0A1h ; '?'      ; Msg
//碰到这种 Msg，应该查出它的名称。当然直接用值也可以，但是不利于理解
//上面还有 2 个参数，虽然查到意思，还是不很明白。

0000:00404EAF             mov     eax, [ebp+hWnd]
0000:00404EB2             push    eax             ; hWnd
0000:00404EB3             call    PostMessageA
0000:00404EB8             jmp     loc_404F8F

0000:00403C6C sub_403C6C     proc near
0000:00403C6C
0000:00403C6C             shr     eax, 10h
0000:00403C6F             retn
0000:00403C6F sub_403C6C     endp
```

OK，下面就是 `WM_LBUTTONDOWN` 的代码了：

```delphi
WM_LBUTTONDOWN:
  Begin
    sPoint.x:=lParam AND $FFFF;
    sPoint.y:=lParam SHR 16;
    //这里就不能用 Hi 和 Lo 了
    If PtInRect(sRect,sPoint) Then
      Begin
        PostMessage(hDlg,WM_NCLBUTTONDOWN,2,0);
      End;
  End;
```

运行一下，效果出来了。虽然还是没有看到标题栏，但是那个地方点上去已经可以拖了。

好，还有 2 个 `WM`。大概看看，`WM_DRAWITEM` 量比较少，而 `WM_PAINT` 写了一堆。当然先逆 `WM_DRAWITEM` 了。

```asm
0000:00404F3F loc_404F3F:
0000:00404F3F             mov     eax, dword ptr [ebp+lParam]
0000:00404F42             call    sub_4042AC
//拜托，又是一个公用 call。参数就是 lParam 的值

0000:00404F47             xor     ebx, ebx
0000:00404F49             jmp     short loc_404F8F

0000:004042AC sub_4042AC     proc near
//怎么知道它是公用的 call？很简单，IDA 有 ref 信息的。不过我贴的时候去掉了

0000:004042AC
0000:004042AC String           = byte ptr -0Ch
0000:004042AC
0000:004042AC             push    ebx
0000:004042AD             add     esp, 0FFFFFFF4h
//sub esp,0CH

0000:004042B0             mov     ebx, eax
//入口参数

0000:004042B2             mov     eax, ds:hbr
0000:004042B7             push    eax             ; hbr
0000:004042B8             lea     eax, [ebx+1Ch]
//有麻烦了，ebx 到底是什么？还好 Delphi 带的 SDK 文档很不错，可以查到
//WM_DRAWITEM 的 lParam 是^DRAWITEMSTRUCT
//另外，注意这里用了 LEA 取地址，所以 eax=ebx+1ch，是一个地址
//对照 Windows.pas 里的定义可以看到它用的定义是 const lprc: TRect。
//所以把结构的成员直接写了没有问题的，Delphi 会自己转换。
//后面的调用也有这个问题，但是 Delphi 很不错。用汇编就要自己注意了。

0000:004042BB             push    eax             ; lprc
0000:004042BC             mov     eax, [ebx+18h]
0000:004042BF             push    eax             ; hDC
0000:004042C0             call    FillRect
0000:004042C5             push    0A0A0A0h        ; COLORREF
0000:004042CA             mov     eax, [ebx+18h]
0000:004042CD             push    eax             ; HDC
0000:004042CE             call    SetTextColor
0000:004042D3             push    1               ; int
0000:004042D5             mov     eax, [ebx+18h]
0000:004042D8             push    eax             ; HDC
0000:004042D9             call    SetBkMode
0000:004042DE             push    0Fh             ; grfFlags
//在 SDK 里可以查到是一些值的组合，下面还有。

0000:004042E0             push    1               ; edge
0000:004042E2             lea     eax, [ebx+1Ch]
0000:004042E5             push    eax             ; qrc
0000:004042E6             mov     eax, [ebx+18h]
0000:004042E9             push    eax             ; hdc
0000:004042EA             call    DrawEdge
0000:004042EF             push    0Ah             ; nMaxCount
0000:004042F1             lea     eax, [esp+10h+String]
0000:004042F5             push    eax             ; lpString
0000:004042F6             mov     eax, [ebx+14h]
0000:004042F9             push    eax             ; hWnd
0000:004042FA             call    GetWindowTextA
0000:004042FF             push    25h             ; uFormat
0000:00404301             lea     eax, [ebx+1Ch]
0000:00404304             push    eax             ; lpRect
0000:00404305             push    0FFFFFFFFh      ; nCount
0000:00404307             lea     eax, [esp+18h+String]
0000:0040430B             push    eax             ; lpString
0000:0040430C             mov     eax, [ebx+18h]
0000:0040430F             push    eax             ; hDC
0000:00404310             call    DrawTextA
0000:00404315             test    byte ptr [ebx+10h], 1
0000:00404319             jz      short loc_404350
0000:0040431B             push    0DDFFh          ; COLORREF
0000:00404320             mov     eax, [ebx+18h]
0000:00404323             push    eax             ; HDC
0000:00404324             call    SetTextColor
0000:00404329             push    25h             ; uFormat
0000:0040432B             lea     eax, [ebx+1Ch]
0000:0040432E             push    eax             ; lpRect
0000:0040432F             push    0FFFFFFFFh      ; nCount
0000:00404331             lea     eax, [esp+18h+String]
0000:00404335             push    eax             ; lpString
0000:00404336             mov     eax, [ebx+18h]
0000:00404339             push    eax             ; hDC
0000:0040433A             call    DrawTextA
0000:0040433F             push    0Fh             ; grfFlags
0000:00404341             push    2               ; edge
0000:00404343             lea     eax, [ebx+1Ch]
0000:00404346             push    eax             ; qrc
0000:00404347             mov     eax, [ebx+18h]
0000:0040434A             push    eax             ; hdc
0000:0040434B             call    DrawEdge

0000:00404350
0000:00404350 loc_404350:
0000:00404350             add     esp, 0Ch
0000:00404353             pop     ebx
0000:00404354             retn
0000:00404354 sub_4042AC     endp
```

好的，`WM_DRAWITEM` 和 `sub_4042AC` 出来了。

```delphi
WM_DRAWITEM:
  Begin
    ItemDraw(PDrawItemStruct(lParam));
    Result:=0;
  End;

Procedure ItemDraw(lpDIS:PDrawItemStruct);
Var
  Str:String;
Begin
  FillRect(lpDIS^.hDC,lpDIS^.rcItem,h_Brush);
  SetTextColor(lpDIS^.hDC,$A0A0A0);
  SetBkMode(lpDIS^.hDC,TRANSPARENT);
  DrawEdge(lpDIS^.hDC,lpDIS^.rcItem,BDR_RAISEDOUTER,BF_RECT);
  GetWindowText(lpDIS^.hwndItem,PChar(Str),10);
  DrawText(lpDIS^.hDC,PChar(Str),-1,lpDIS^.rcItem,DT_SINGLELINE OR DT_VCENTER OR DT_CENTER);
  If lpDIS^.itemState MOD 2=1 Then
    Begin
      SetTextColor(lpDIS^.hDC,$DDFF);
      DrawText(lpDIS^.hDC,PChar(Str),-1,lpDIS^.rcItem,DT_SINGLELINE OR DT_VCENTER OR DT_CENTER);
      DrawEdge(lpDIS^.hDC,lpDIS^.rcItem,BDR_SUNKENOUTER,BF_RECT);
    End;
End;
```

试试看，原来是画三维边框。



另外，不知道大家注意到没有，贴上来的 `PAS` 代码其实都是排好的。直接 `C&P` 就可以，

连缩进也不用做。

还有，上面的代码有个问题，至少我认为是个问题。就是在 `GetWindowText` 那里。

虽然已经限制了长度，但是 `Str` 没有初始化，这个在以前没有什么，因为都是 `Short` 的。

但是现在还有个 `PChar`，而且和 `Short` 都对应到 `String` 类型。以前老是有人抱怨说 `PChar` 出 `AccessViolation`，结果最后发现是没有初始化，`PChar` 都指到随机地址了。后来用了 `SetLength` 就 `OK` 了。不过这里因为 `Str` 是在堆栈分配的，所以个人认为问题不大。不知老钱怎么看这个问题，最好是请他指教一下。

> 引用:

>

> * * *

>

> *最初由 `laoqian` 发布*

> **我也遇到 `PChar` 问题,老是有警告,但是忽略了,一般也没问题 ,这次你一说,大家都明白了原因,又长知识了. **

>

> * * *

但是问题绝对没有这么简单。而且我讲的很模糊，自己都没有完全明白，只是知道大概是这么回事。

问题是，现在 `string` 是在 `stack` 里分配的，从代码看，是局部变量。画个堆栈就可以发现，这个 `String` 的位置恰好对应到 `004042AD` 的指令分配的空间，而且长度为 `0ch`。而从下面可以看见，`get` 到的长度最多是 `0ah`。差 2 个字节。但是 `short` 的存放方式是 1 个长度字节+255 个 `buffer`，这里却有 2 个字节的富余。然而 `AnsiString` 起码也要 4 个字节。实在想不通是怎么回事。试了用 `String\[10\]`，但是在类型转换的地方出了问题。`Array of Byte` 没有试过，从理论和实际上应该可以自圆其说，但因为 `GetWindowText` 的关系，没敢造次。

想请 `laoqian` 把那个地方的代码贴一下，就是变量说明和 `GetWindowText` 的调用。反正我已经逆出来了，你再贴应该也不算违规，何况只是一部分。谢谢。

另外，我注意到这个问题是因为最近在研究缓冲区溢出，特别是堆栈溢出。从 `GetWindowText` 看应该不会有危险，但是 2 个字节的差实在是太巧合了，要是 1 个或者 3 个 4 个我都不会这么注意这个地方。安焦的 `mail` 系统有问题，我收不到认证信，没法发言问。

我为了大家的技术提高，和学习兴趣，只好放出原程序了，`youknown` 不要骂我啊。

看到了吗，是个数组，。

```
//////////////////////////////////////////////////////////////
//绘制按钮函数
//pdis:       绘制内容结构指针
procedure DrawButton(pdis: PDRAWITEMSTRUCT);
var
  szText: array[0..9] of char; //按钮文字
begin
  FillRect(pdis.hDC, pdis.rcItem, BKC); //以背景色 BKC 填充按钮
  SetTextColor(pdis.hDC, clText);
  SetBkMode(pdis.hDC, TRANSPARENT);
  //尚未点击，绘制按钮边框-突起状态
  DrawEdge(pdis.hDC, pdis.rcItem, BDR_RAISEDOUTER, BF_RECT);
  GetWindowText(pdis.hwndItem, szText, sizeof(szText));
  DrawText(pdis.hDC, szText, -1, pdis.rcItem, DT_SINGLELINE or DT_CENTER or DT_VCENTER);
  //已被按下，绘制按钮边框-凹陷状态
  //if (pdis.itemState and ODS_SELECTED)=ODS_SELECTED then
  if (pdis.itemState and ODS_SELECTED) <> 0 then
  begin
    SetTextColor(pdis.hDC, $00ddff);
    DrawText(pdis.hDC, szText, - 1, pdis.rcItem, DT_SINGLELINE or DT_CENTER or DT_VCENTER);
    DrawEdge(pdis.hDC, pdis.rcItem, BDR_SUNKENOUTER, BF_RECT);
  end;
end;
//////////////
```

大家可以看出他逆向的还真差不多呢！佩服

> 引用:

>

> * * *

>

> *最初由 `laoqian` 发布*

> **我为了大家的技术提高，和学习兴趣，只好放出原程序了，`youknown` 不要骂我啊。

> 看到了吗，是个数组，。

>   

> //////////////////////////////////////////////////////////////

> //绘制按钮函数

> ........ **

>

> * * *

好哇，果然是个 `Array of Byte`！害我死了 N 个脑细胞，真个是 `JR`！哈哈！

顺便说一句，我做缓冲区都是用 `Array[1..1] of Byte` 指针的，然后 `GetMem` 分配内存。有点烦，但是从来没有出过问题。

谢谢！！！

哇哈哈，看来我做的分析还是基本正确的。不过那个 `ODS_SELECTED` 就没有分析出来。逆向终究还是不可能和原来的程序完全一样的。

不过你分配了 10 个字节，还是差 2 个。这个只好等我动态调试了。

`Delphi` 真是好啊，`laoqian`，看到那个^了没有？自动转换的！

说实在的，因为原来的程序基本是 `SDK` 写的，加上我以前是学 `TP` 出道，而且专门研究过传值方式，所以才可以逆到这个程度。如果你加了算法，我就没辙了。另外我正在逆 `WM_PAINT`，这部分难度比较大。估计花的时间要以前所有时间的起码 2 倍。目前只做了过程首部形式的分析，觉得有个地方很奇怪，应该是模板编程的原因。如果你想先看看，我可以把那部分的分析发给你。

`4dd466d0e1e0cc123afbe504da2462ca`


## 汇编代码分析

以下是 `sub_406F99` 函数中调用 `GradientFill` 的关键片段：

```
seg000:004070B1                 cmp     ebx, 2
seg000:004070B4                 jnz     short loc_4070D0
//如果 ebx=GRADIENT_FILL_TRIANGLE，那么继续
seg000:004070B6                 mov     [ebp+var_54], 3
//var_54.Vertex1=3
seg000:004070BD                 push    2                   ; ULONG
//GRADIENT_FILL_TRIANGLE
seg000:004070BF                 push    1                   ; ULONG
seg000:004070C1                 lea     eax, [ebp+var_54]
seg000:004070C4                 push    eax                 ; PVOID
seg000:004070C5                 push    4                   ; ULONG
seg000:004070C7                 push    esi                 ; PTRIVERTEX
seg000:004070C8                 push    [ebp+arg_0]         ; HDC
seg000:004070CB                 call    GradientFill
seg000:004070D0
seg000:004070D0 loc_4070D0:                         ; CODE XREF: sub_406F99+11B
seg000:004070D0                 pop     edi
seg000:004070D1                 pop     esi
seg000:004070D2                 pop     ebx
seg000:004070D3                 mov     esp, ebp
seg000:004070D5                 pop     ebp
seg000:004070D6                 retn    0Ch
seg000:004070D6 sub_406F99      endp
```

从调用代码看，这段代码没有返回值。下面的代码做了一点整理。

### 对应 Delphi 代码

```
Type
  COLOR16 = Word;

  PTriVertex = ^TTriVertex;
  _TRIVERTEX = Packed Record
    x: Longint;
    y: Longint;
    Red: COLOR16;
    Green: COLOR16;
    Blue: COLOR16;
    Alpha: COLOR16;
  End;
  TTriVertex = _TRIVERTEX;
  TRIVERTEX = _TRIVERTEX;

  AlphaRGB=Packed Record
    Case Byte Of
    1:(
    Alpha:Byte;
    Blue:Byte;
    Green:Byte;
    Red:Byte);
    2:(
    ARGB:LongWord);
  End;

Procedure DoGradientFill(WinDC:HDC;DCRect:PRect;AlphaRGBArray:Array[0..3] of AlphaRGB);
Var
  GT:GRADIENT_TRIANGLE;
  TVs:Array[0..3] of TRIVERTEX;
  GR:GRADIENT_RECT;
  i:LongWord;
Begin

  For i:=0 To 3 Do
    Begin
      TVs[i].Red:=AlphaRGBArray[i].Red;
      TVs[i].Green:=AlphaRGBArray[i].Green;
      TVs[i].Blue:=AlphaRGBArray[i].Blue;
      TVs[i].Alpha:=AlphaRGBArray[i].Alpha;
    End;

  TVs[0].x:=DCRect^.Left;
  TVs[0].y:=DCRect^.Top;
  TVs[1].x:=DCRect^.Right;
  TVs[1].y:=DCRect^.Top;
  TVs[2].x:=DCRect^.Left;
  TVs[2].y:=DCRect^.Bottom;
  TVs[3].x:=DCRect^.Right;
  TVs[3].y:=DCRect^.Bottom;

  If (DCRect^.Left=DCRect^.Top) AND (DCRect^.Right=DCRect^.Bottom) Then
    Begin
      GR.UpperLeft:=0;
      GR.LowerRight:=3;
      GradientFill(WinDC,@TVs,4,@GR,1,GRADIENT_FILL_RECT_V);
    End
  Else
    Begin
      GT.Vertex1:=0;
      GT.Vertex2:=1;
      GT.Vertex3:=2;
      GradientFill(WinDC,@TVs,4,@GT,1,GRADIENT_FILL_TRIANGLE);

      GT.Vertex1:=3;
      GradientFill(WinDC,@TVs,4,@GT,1,GRADIENT_FILL_TRIANGLE);
    End;

End;
```

### 小结

保持清醒的头脑是非常重要的。我曾经算错 `mov` 的地址，后来从头看才发现错误所在。

OK，休息。

## 对 [FCG]KeyGen2.0 按钮特效的分析 5

这个过程应该比较简单。

```
seg000:00406CA5 sub_406CA5     proc near                 ; CODE XREF: sub_4070D9+18D
seg000:00406CA5
seg000:00406CA5 Rect          = tagRECT ptr -10h
seg000:00406CA5 arg_0         = dword ptr  8
seg000:00406CA5
seg000:00406CA5                 push    ebp
seg000:00406CA6                 mov     ebp, esp
seg000:00406CA8                 add     esp, 0FFFFFFF0h
seg000:00406CAB                 push    ebx
seg000:00406CAC                 mov     ebx, [ebp+arg_0]
seg000:00406CAF                 cmp     dword ptr [ebx+0Ch], 1
//arg_0 又是指指针
seg000:00406CB3                 jnz     short loc_406CE6
seg000:00406CB5                 lea     eax, [ebp+Rect]
seg000:00406CB8                 push    eax                 ; lpRect
seg000:00406CB9                 push    dword ptr [ebx] ; hWnd
seg000:00406CBB                 call    GetWindowRect
seg000:00406CC0                 push    0CC0020h            ; DWORD
seg000:00406CC5                 push    0                   ; int
seg000:00406CC7                 push    0                   ; int
seg000:00406CC9                 push    dword ptr [ebx+8] ; HDC
seg000:00406CCC                 mov     edx, [ebp+Rect.bottom]
seg000:00406CCF                 sub     edx, [ebp+Rect.top]
seg000:00406CD2                 push    edx                 ; int
seg000:00406CD3                 mov     ecx, [ebp+Rect.right]
seg000:00406CD6                 sub     ecx, [ebp+Rect.left]
seg000:00406CD9                 push    ecx                 ; int
seg000:00406CDA                 push    0                   ; int
seg000:00406CDC                 push    0                   ; int
seg000:00406CDE                 push    dword ptr [ebx+4] ; HDC
seg000:00406CE1                 call    BitBlt
seg000:00406CE6
seg000:00406CE6 loc_406CE6:                         ; CODE XREF: sub_406CA5+E
seg000:00406CE6                 cmp     dword ptr [ebx+10h], 0
seg000:00406CEA                 jz      short loc_406CFD
seg000:00406CEC                 push    dword ptr [ebx+10h] ; HGDIOBJ
seg000:00406CEF                 push    dword ptr [ebx+8] ; HDC
seg000:00406CF2                 call    SelectObject
seg000:00406CF7                 push    eax                 ; HGDIOBJ
seg000:00406CF8                 call    DeleteObject
seg000:00406CFD
seg000:00406CFD loc_406CFD:                         ; CODE XREF: sub_406CA5+45
seg000:00406CFD                 push    dword ptr [ebx+8] ; HDC
seg000:00406D00                 call    DeleteDC
seg000:00406D05                 push    dword ptr [ebx+4] ; hDC
seg000:00406D08                 push    dword ptr [ebx] ; hWnd
seg000:00406D0A                 call    ReleaseDC
seg000:00406D0F                 pop     ebx
seg000:00406D10                 mov     esp, ebp
seg000:00406D12                 pop     ebp
seg000:00406D13                 retn    4
seg000:00406D13 sub_406CA5     endp
```

实在没有什么难度，注释都不必写。唯一值得注意的是，`arg_0` 是个什么玩意。对照一下 `sub_406C06` 的分析，可以看出，`arg_0` 就是 `WindowsDCInfo` 的指针。

### 对应 Delphi 代码

```
Procedure sub_406CA5(DCInfo:PWindowsDCInfo);
Var
  Rect:TRect;
Begin
  If DCInfo^.unknown=1 Then
    Begin
      GetWindowRect(DCInfo.hWindow,Rect);
      BitBlt(DCInfo.WinDC,0,0,Rect.right-Rect.left,Rect.bottom-Rect.top,DCInfo.CompDC,0,0,SRCCOPY);
    End;

  If DCInfo^.hBitmap<>0 Then
    Begin
      DeleteObject(SelectObject(DCInfo.CompDC,DCInfo.hBitmap));
    End;

  DeleteDC(DCInfo.CompDC);
  ReleaseDC(DCInfo.hWindow,DCInfo.WinDC);
End;
```

看来这段代码是释放 `sub_406C06` 生成的 DC 资源。

终于开始分析 `sub_4070D9` 了：

```
seg000:004070D9 sub_4070D9     proc near                 ; CODE XREF: sub_406D7D+5C
seg000:004070D9                              ; sub_406D7D+89 ...
seg000:004070D9
seg000:004070D9 String        = byte ptr -124h
seg000:004070D9 var_24       = dword ptr -24h
seg000:004070D9 hDC         = dword ptr -1Ch
seg000:004070D9 var_18       = dword ptr -18h
seg000:004070D9 var_14       = dword ptr -14h
seg000:004070D9 var_10       = dword ptr -10h
seg000:004070D9 arg_0        = dword ptr  8
seg000:004070D9
seg000:004070D9                 push    ebp
seg000:004070DA                 mov     ebp, esp
seg000:004070DC                 add     esp, 0FFFFFEDCh
seg000:004070E2                 push    ebx
seg000:004070E3                 push    esi
seg000:004070E4                 push    edi
seg000:004070E5                 mov     ebx, [ebp+arg_0]
```

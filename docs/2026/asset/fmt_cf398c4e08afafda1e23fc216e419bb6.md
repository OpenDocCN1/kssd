
```pascal
SetWindowPos(h_Scroll,HWND_TOP,0,xCount,ScrollWidth,LineCount*12,0);

If xCount<(0-LineCount*12) Then
  Begin
    xCount:=ScrollHeight;
  End;
End;

Function CountCRLF(Str:String):Word;
Var
  Count:Word;
  i:Word;
Begin
  Count:=1;
  For i:=1 to Length(Str) Do
    Begin
      If Str[i]=#$0A Then Inc(Count);
    End;
  CountCRLF:=Count;
End;

Function ScrollProc(hDlg:HWND;Msg,wParam,lParam:DWORD):LRESULT;stdcall;
Var
  sRect:TRect;
  sPaint:PAINTSTRUCT;
  DC:HDC;
  LFont:HFONT;
Begin
  If Msg=WM_PAINT Then
    Begin
      DC:=BeginPaint(hDlg,sPaint);
      GetClientRect(hDlg,sRect);
      SetTextColor(DC,$A0A0A0);
      SetBkMode(DC,TRANSPARENT);
      LFont:=CreateFont(-$C,0,0,0,0,0,0,0,1,0,0,0,0,'宋体');
      SelectObject(DC,LFont);
      DrawText(DC,szScroll,-1,sRect,DT_CENTER);
      EndPaint(hDlg,sPaint);
      DeleteObject(LFont);
    End
  Else
    Begin
      CallWindowProc(Pointer(GetWindowLong(hDlg,GWL_USERDATA)),hDlg,Msg,wParam,lParam);
    End;
  Result:=1;
End;
```

这样，整个程序就分析完了。下面作一下总结。

从编程方面看，该程序有几处地方值得借鉴：

1. 按钮的三维边框；
2. 渐变标题栏；
3. 滚动字幕的方法。

从逆向工程的角度看，难点主要在渐变标题栏和滚动字幕的处理上。主要是这两个地方有较多的运算。

另外，在分析过程中，有几点是值得注意的：

1. 注意区分全局变量和局部变量。局部变量一般用`ebx`引用。在分析滚动字幕时，由于把一个全局变量当作局部变量，造成字幕无法重复滚动。不过，如果将该局部变量用`Const`说明，还是可以达到目的的。
2. 注意变量的类型和地址。这样就不会弄错变量。
3. API中的常量。这些常量一般可以在SDK中找到，但有些是组合值，比如`CreateWindowEx`里使用的`$50000001`，这就需要仔细分析。
4. 有符号数的处理。在本例中直接体现在渐变标题栏上。
5. 不要迷信反汇编程序。

另外，你必须熟悉该程序使用的语言（尤其是内部结构，而不是仅仅会写几行程序就可以的），这样才能够得心应手。但伪编译语言（VB、Java、.NET）除外。当然，你还必须熟悉汇编语言。什么？不熟悉？临时学习？呃，算了吧。逆向工程只是手段而不是目的，而且这对阁下来说也不好玩。倒是交谊舞对于阁下更实际点儿……阁下35以上？那……当我什么都没说好了……

OK，这个笔记系列就基本结束了。但是有个缺憾：渐变标题栏没有作到原程序那样的效果。也许以后我会分析出来的。不过现在只能说声抱歉了。

谢谢各位观赏，也谢谢各位的耐心。

PS:不要轻易决定做逆向工程，更不要轻易说你要做逆向工程并要写下心得以供分享。但是如果你决定了，那么无论如何要完成它。特别是在第一篇心得贴出来以后……想想那些网络上的“太监书”，再想想曹老先生的半部《红楼梦》……

---

原以为About不会很复杂，没想到里面居然套了那么多东西。直接后果就是这篇文章比以前贴的任何一篇都长，打开的速度肯定要慢点。不过你既然都看到这里了，就别抱怨了，OK？

## 附：关于Delphi逆向工程笔记系列的说明

事情的起因是我看到laoqian的一篇文章，他提到了FCG的KG模板，但是碍于FCG的规矩不能放出来，只是给了编译过的程序。但是我又比较喜欢收集这个。本来嘛，加入FCG就可以了，但是本人不会脱壳，破解又只能到明码比较的水平，估计没有哪个组织会要。很幸运，那个模板是Delphi的，而且laoqian只用upx加了壳，这样我才可以很方便地把它脱掉。不过他还是给我找了点小麻烦，破坏了upx头部的校验字节，拿upx就脱不掉了。

更幸运的是KG是用SDK写的（这也是惯例）……大家肯定都知道SDK程序编译出来是什么样子。也正是如此，我才可以做这个RE。如果是VCL的话，我也只有去找非明码比较的麻烦了。

做了半天，发现有些很有趣的东西。于是想拿出来。原以为这些东西肯定不太入眼，特别是对于在pediy混的人。但发现反响比较好，这是我始料不及的。

---

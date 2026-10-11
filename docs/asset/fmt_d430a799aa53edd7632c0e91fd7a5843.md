
# 光盘伴侣 2.0

> 原文：[`www.pediy.com/kssd/pediy03/forum1095.htm`](https://www.pediy.com/kssd/pediy03/forum1095.htm)

## 光盘伴侣 2.0 不完全注册机

**目的：**

1. 介绍一个 debug 软件 `Ollydbg1.03`，因为它支持浮点寄存器显示，且不必开机就运行……；而 `trw2000` 不支持浮点寄存器显示，`softice` 需要开机就运行。
2. delphi 的一个控件 `KOL&MCK`，你可以从 `http://kol.cjb.net` 下载，用它编写的 delphi 程序只有 30K 左右，大小是不用它编译的程序的十分之一；而且不必再害怕 `DEDE` 等基于 Form 的反编译软件。

## 注册码计算

```
:00408FF9 0FBE0F       movsx ecx, byte ptr [edi] 

:00408FFC 8BC1        mov eax, ecx 

:00408FFE 99          cdq 

:00408FFF 33C2        xor eax, edx 

:00409001 2BC2        sub eax, edx 

:00409003 85DB        test ebx, ebx 

:00409005 8BF0        mov esi, eax 

:00409007 750E        jne 00409017                  //用户名大于 1 个字符就跳 

:00409009 8975A8      mov dword ptr [ebp-58], esi //取用户名[1]ascii 码    

:0040900C DB45A8      fild dword ptr [ebp-58]  //装入 st0 

:0040900F D845AC      fadd dword ptr [ebp-54]  //加 dword  ptr[ebp-54]指向的数 

:00409012 D95DAC      fstp dword ptr [ebp-54] //存入 dword ptr [ebp-54]    

:00409015 EB31        jmp 00409048 

* Referenced by a (U)nconditional or (C)onditional Jump at Address: 

|:00409007(C) 

| 

:00409017 8975A8      mov dword ptr [ebp-58], esi //依次取用户名的 2、3....字符 ascii 码，换算为十进制数    

:0040901A DB45A8      fild dword ptr [ebp-58]  //装入 st0 

:0040901D 83C4F4      add esp, FFFFFFF4 

:00409020 DB3C24      fstp tbyte ptr [esp] 

:00409023 895DA4      mov dword ptr [ebp-5C], ebx 

:00409026 DB45A4      fild dword ptr [ebp-5C] 

:00409029 83C4F4      add esp, FFFFFFF4 

:0040902C DB3C24      fstp tbyte ptr [esp] 

:0040902F E8FC820A00  call 004B1330    //计算用户名[i]的（i-1）次方 

:00409034 D845AC      fadd dword ptr [ebp-54]  //累加 

:00409037 895DA0      mov dword ptr [ebp-60], ebx 

:0040903A DB45A0      fild dword ptr [ebp-60] 

:0040903D DEE9        fsubp st(1), st(0)  //减（i-1）    

:0040903F D825CC914000 fsub dword ptr [004091CC]  //减 1 

:00409045 D95DAC      fstp dword ptr [ebp-54]  //放入 dword  ptr [ebp-54] 

:00409048 43          inc ebx 

:00409049 47          inc edi 

:0040904A 8B45B4      mov eax, dword ptr [ebp-4C] 

:0040904D E8BEB60A00  call 004B4710 

:00409052 3BC3        cmp eax, ebx 

:00409054 77A3        ja 00408FF9    //i<=length(name)则跳回    

..... 

..... 

   但是存在一个问题，根据动态跟踪所得，若用户名>4 位，则注册码有校正，我不知道是如何进行的？
```

## 注册机（用户名不超过 4 位）

（使用 `KOL&MCK` 控件，其中部分函数的写法依照此控件规定；delphi5.0 编译通过，仅 26K，哈哈）

`main form` 中有 `editbox1`，`editbox2`，`button1`

\----------------cut here--------------------------------------

```pascal
{ KOL MCK } // Do not remove this line! 

{$DEFINE KOL_MCK} 

unit Unit1; 

interface 

{$IFDEF KOL_MCK} 

uses Windows, Messages, ShellAPI, KOL {$IFNDEF KOL_MCK}, mirror, Classes, 
  mckCtrls, Controls {$ENDIF}; 

{$ELSE} 

{$I uses.inc} mirror, 
  Windows, Messages, SysUtils, Classes, Graphics, Controls, Forms, Dialogs, 
  mirror; 

{$ENDIF} 

type 
  {$IFDEF KOL_MCK} 

  {$I MCKfakeClasses.inc} 

  PForm1 = ^TForm1; 

  TForm1 = object(TObj) 
    Form: PControl; 

  {$ELSE not_KOL_MCK} 

  TForm1 = class(TForm) 

  {$ENDIF KOL_MCK} 

    KOLProject1: TKOLProject; 

    KOLForm1: TKOLForm; 

    EditBox1: TKOLEditBox; 

    EditBox2: TKOLEditBox; 

    Button1: TKOLButton; 

    procedure Button1Click(Sender: PObj); 

  private 
    { Private declarations } 

  public 
    { Public declarations } 

  end; 

var 
  Form1 {$IFDEF KOL_MCK} : PForm1 {$ELSE} : TForm1 {$ENDIF} ; 
  len,a,b,i,t,code,sum:integer; 
  name:string; 

{$IFDEF KOL_MCK} 

procedure NewForm1( var Result: PForm1; AParent: PControl ); 

{$ENDIF} 

implementation 

{$IFNDEF KOL_MCK} {$R *.DFM} {$ENDIF} 

{$IFDEF KOL_MCK} 

{$I Unit1_1.inc} 

{$ENDIF} 

procedure TForm1.Button1Click(Sender: PObj); 

begin 

  len:=length(editbox1.text); 

  name:=editbox1.text; 

  if len=0 then editbox2.text:='input your name'; 

  if len=1 then editbox2.text:=int2str(ord(name[1])) 
    else 
    if len>4 then editbox2.text:='this keygen only support name of 4 digits max！' 
    else 
        begin 
          i:=2;  code:=0; 

          while i<=len do 
              begin 
                  a:=ord(name[i]); b:=1;  t:=2; 

                  while t<= i do 
                  begin 
                      b:=b*a; t:=t+1; 
                  end; 

                  b:=b-i; code:=code+b; 

                  i:=i+1; 
              end; 

          editbox2.text:=int2str(code+ord(name[1])); 
        end; 

end; 

end. 
```

\--------------------------end--------------------------------  4dd466d0e1e0cc123afbe504da2462ca

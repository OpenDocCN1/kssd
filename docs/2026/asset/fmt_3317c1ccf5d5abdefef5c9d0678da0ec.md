
# FolderView注册部分的计算 (13千字)

控件名称：FolderView 3.0

软件授权：共享软件

下载地址：http://www.filelibrary.com:8080/cgi-bin/registered/download/New\_Files/n/150/fldrvw30.zip

该站点虽然免费下载，但需要密码，因此需要先注册一下

http://www.ssware.com/fldrview/fldrvw30.zip

到该站点下载时，需要代理才能下载

控件简介：FolderView is an ActiveX control that behaves like the left pane of Windows Explorer, showing all the files and folders on the system. Some of its features include context menus for items, the ability to execute shell commands, and automatic updating.

目标程序：`FldrView.ocx`，81408 bytes

破解工具：W32Dasm白金版，IDA Pro 4.15，Caspr v1.012

说明：本人想用VB编一个程序，需要用到只取得文件夹名称的控件，而VB中的`Dirlist`控件只能显示当前驱动器的文件夹，必须结合`DriveList`控件，但这样用起来又太不方便了，然人感觉太别扭，于是上网搜索，奋战了3个多小时，终于找到这两个满意的，确实比微软的那个强，但都需要注册，既然没有条件进行注册，那么就只好Crack了^\_^

这个控件同样适用于Delphi、VC，用FI查看，判别不出来(我的FI有点老)，估计是Aspack压缩，用Caspr v1.012成功去壳，再用FI查看，VC++6.0编的，这回可以用W32Dasm和IDA了，闲话少说，开始干，等等，忽然想到，W32Dasm对MFC函数支持不好，因此这回想用IDA试试，但用IDA反汇编后，还是出现了一个问题，输入注册码和显示注册是否成功部分并没有显示出来，而W32Dasm则很容易地显示出来了，以前没怎么用过IDA，不知道怎么把这段代码改成汇编格式，经过一段探索之后，终于解决，方法是跳到`10006E99`，从菜单中选择Edit-->Functions-->Create Function即可，下面为IDA反汇编的结果

```asm
.text:10006E99 sub\_10006E99    proc near
      ; DATA XREF: .rdata:1000AB74
.text:10006E99                push
  esi
.text:10006E9A                mov
  esi, ecx
.text:10006E9C                push
  1
.text:10006E9E                call
  ?UpdateData@CWnd@@QAEHH@Z ; CWnd::UpdateData(int)
.text:10006EA3                mov
  eax, [esi+68h]
.text:10006EA6                cmp
  dword ptr [eax-8], 0
.text:10006EAA                jz
    short loc\_10006F17
.text:10006EAC                mov
  ecx, [esi+64h]
.text:10006EAF                cmp
  dword ptr [ecx-8], 0
.text:10006EB3                jz
    short loc\_10006F17
.text:10006EB5                push
  eax
.text:10006EB6                push
  offset aUsername ; "Username"
.text:10006EBB                call
  sub\_10006917    ====> Username填入到注册表中
.text:10006EC0                push
  dword ptr [esi+64h]
.text:10006EC3                push
  offset aRegcode ; "Regcode"
.text:10006EC8                call
  sub\_10006917    ====> Regcode填入到注册表中
.text:10006ECD                mov
  ecx, [esi+60h]
.text:10006ED0                add
  esp, 10h
.text:10006ED3                push
  1
.text:10006ED5                call
  ?SetModifiedFlag@COleControl@@QAEXH@Z ; COleControl::SetModifiedFlag(int)
.text:10006EDA                mov
  eax, [esi+64h]  ====> "Regcode"
.text:10006EDD                mov
  ecx, [esi+68h]  ====> "Username"
.text:10006EE0                push
  eax
.text:10006EE1                push
  ecx
.text:10006EE2                call
  sub\_10006958        ====> 注册码的计算部分，见后面的分析
.text:10006EE7                pop
  ecx
.text:10006EE8                test
  eax, eax            ====> eax为旗标
.text:10006EEA                pop
  ecx
.text:10006EEB                jz
    short loc\_10006F09  ====> 等于0则注册码错误
.text:10006EED                push
  40h
.text:10006EEF                push
  offset aRegistrationSu ; "Registration successful"
.text:10006EF4                push
  offset aThankYouForReg ; "Thank You for registering FolderView Ac"...
.text:10006EF9                mov
  ecx, esi
.text:10006EFB                call
  ?MessageBoxA@CWnd@@QAEHPBD0I@Z ; CWnd::MessageBoxA(char const \*,char
const \*,uint)
.text:10006F00                mov
  ecx, esi
.text:10006F02                call
  ?OnOK@CDialog@@MAEXXZ ; CDialog::OnOK(void)
.text:10006F07                pop
  esi
.text:10006F08                retn
.text:10006F09
.text:10006F09 loc\_10006F09:
            ; CODE XREF: sub\_10006E99+52j
.text:10006F09                push
  10h
.text:10006F0B                push
  offset aInvalidRegistr ; "Invalid registration code"
.text:10006F10                push
  offset aInvalidRegcode ; "Invalid Regcode. Please enter the corre"...
.text:10006F15                jmp
  short loc\_10006F23
.text:10006F17
.text:10006F17 loc\_10006F17:
            ; CODE XREF: sub\_10006E99+11
.text:10006F17
; sub\_10006E99+1A
.text:10006F17                push
  10h
.text:10006F19                push
  offset aEnterInformati ; "Enter information"
.text:10006F1E                push
  offset aPleaseEnterThe ; "Please enter the username and registrat"...
.text:10006F23
.text:10006F23 loc\_10006F23:
            ; CODE XREF: sub\_10006E99+7C
.text:10006F23                mov
  ecx, esi
.text:10006F25                call
  ?MessageBoxA@CWnd@@QAEHPBD0I@Z ; CWnd::MessageBoxA(char const \*,char
const \*,uint)
.text:10006F2A                pop
  esi
.text:10006F2B                retn
int \_\_cdecl sub\_10006958(int,int);
.text:10006958 sub\_10006958    proc near    ; CODE XREF:
.text:10004249  .text:10004498
.text:10006958
            ; 因为IDA没有将10006E99反编译出来，故这里代码参考没有列出来
.text:10006958 var\_160        = qword ptr -160h
.text:10006958 var\_158        = qword ptr -158h
.text:10006958 var\_14C        = byte ptr -14Ch
.text:10006958 var\_20          = qword ptr -20h
.text:10006958 var\_18          = qword ptr -18h
.text:10006958 var\_10          = qword ptr -10h
.text:10006958 var\_8          = qword ptr -8
.text:10006958 arg\_0          = dword ptr  8
  ====> "Username"
.text:10006958 arg\_4          = dword ptr  0Ch
====> "Regcode"
.text:10006958
.text:10006958                push
  ebp
.text:10006959                mov
  ebp, esp
.text:1000695B                sub
  esp, 14Ch
.text:10006961                fld
  ds:dbl\_1000A900    ====> 双击此处可知dbl\_1000A900=56
.text:10006967                push
  esi
.text:10006968                mov
  esi, [ebp+arg\_0]
.text:1000696B                fstp
  [ebp+var\_18]
.text:1000696E                fldz
.text:10006970                push
  esi
.text:10006971                fstp
  [ebp+var\_10]
.text:10006974                call
  strlen            ====> Username的长度
.text:10006979                pop
  ecx
.text:1000697A                xor
  ecx, ecx
.text:1000697C                test
  eax, eax
.text:1000697E                jle
  short loc\_100069B1
.text:10006980                movsx
edx, byte ptr [esi]
.text:10006983                mov
  [ebp+arg\_0], edx
.text:10006986                fild
  [ebp+arg\_0]        ====> st0=第1个Username字符
.text:10006989                movsx
edx, byte ptr [ecx+esi]    ====> 取1个Username字符
.text:1000698D                mov
  [ebp+arg\_0], edx
.text:10006990                inc
  ecx
.text:10006991                fild
  [ebp+arg\_0]
.text:10006994                cmp
  ecx, eax
.text:10006996                fadd
  st, st(1)          ====> st0=[ebp+arg\_0]，st1=Username(1)
.text:10006998                fmul
  [ebp+var\_18]
.text:1000699B                fadd
  [ebp+var\_10]
.text:1000699E                fstp
  [ebp+var\_10]
.text:100069A1                fld
  [ebp+var\_18]
.text:100069A4                fadd
  ds:dbl\_1000A8F8    ====> dbl\_1000A8F8=2
.text:100069AA                fstp
  [ebp+var\_18]
.text:100069AD                jl
    short loc\_10006989

10006989-100069AD的计算过程：

for i=1 to strlen(Username);Code1=Code1+(Username(1)+Username(i))\*(56+(i-1)\*2);next

.text:100069AF                fstp
  st
.text:100069B1                fld
  [ebp+var\_10]
.text:100069B4                fadd
  ds:dbl\_1000A8F0    ====> dbl\_1000A8F0=3419821
.text:100069BA                push
  offset aEqualizer ; "equalizer"
.text:100069BF                fst
  [ebp+var\_10]      ====> 令RegCode1=Code1+3419821
.text:100069C2                fstp
  [ebp+var\_8]
.text:100069C5                call
  strlen            ====> "equalizer"的长度
.text:100069CA                pop
  ecx
.text:100069CB                xor
  ecx, ecx
.text:100069CD                test
  eax, eax
.text:100069CF                jle
  short loc\_100069F0
.text:100069D1                movsx
edx, byte ptr [esi]
.text:100069D4                push
  edi
```

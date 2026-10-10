

# RadASM 环境下写 ollyscript

> 原文：[`www.pediy.com/kssd/pediy10/60551.html`](https://www.pediy.com/kssd/pediy10/60551.html)

起因：学习了一下 loveboom 写的 OSEditor，由于没有语法着色，所以这么一个 IDE 还是不够理想，而新写程序实现语法着色的确十分麻烦。突然想到 RadASM 的功能，于是就想能不能在 RadASM 下配置文档使能写 ollyscript 脚本。  
由于对 RadASM 的配置文档不是很熟悉，而且我本来就是一个大菜鸟。所以暂时写出来的这个 for ollyscript 很简陋。至于色彩的问题自己可以调整喽。  
两个文件，一个是配置档，用作 RadASM 载入新的编程语言。一个是 api 提示。  
还存在问题等待高手来解决：  
1，函数输入完后 tab 键变成自动输出(，很郁闷，如何才能实现准确的变换？  
  
希望这些起到抛砖引玉的作用，能有大牛把更帅的配置文档写一下，嘿嘿。![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")   
  
  
ollyscript.ini 文件如下：  
\[CODE\]  
\[Description\]  
1=ollyscript 编译环境(triones 2008-3-2)  
  
\[Environment\]  
1=path,D:\\Program Files\\OllyICE;  
  
\[CharTab\]  
2=0251222522222236;20-2F  
3=1111111111222221;30-3F  
  
\[Open\]  
0="ollyscript Files (\*.txt;\*.osc),\*.txt;\*.osc"  
1="All files (\*.\*),\*.\*"  
txt="ollyscript Files (\*.txt),\*.txt,txt"  
osc="ollyscript Files (\*.osc),\*.osc,osc"  
  
\[CodeMacro\]  
;KeyWord,InsertBeforeActiveLine,InsertAfterActiveLine  
;{I}=Copy indent from keyword line  
;{C}=Cr  
;{S}=Space  
;{T}=Tab  
;{$}=Copy Proc/Struct Name  
;The macros are set up to assume AutoIndent on  
;第二个逗句是 Marcro 完成时光标停留的地方  
1=exec,{T},{C}{I}ende  
2=  
3=  
4=  
5=  
6=  
7=  
8=  
9=  
10=  
  

代码:

```
Skip={C},$:
;这个好像是自动缩进
Code=
Const={T},"$"
Data=
Macro=
Struct=
Label={C},$:
Exec=ende,exec

\[CodeBlock\]
;这里设定代码分块标志，最后数字指向 CodeMarcro，
1=exec,ende,,,1
2=
3=
4=
5=
6=
7=
8=
9=
10=
11=
12=
13=
14=
15=

\[Api\]
Trig=
Call=ollyscript\\ollyscript.api
Const=ollyscript\\Const.api

\[Edit\]
Font=宋体
FontHeight=-16
FontWeight=400
FontItalic=0
FontCharSet=134
TxtFont=宋体
TxtFontHeight=-16
TxtFontWeight=400
TxtFontCharSet=134
TabSize=4
Backup=0
AutoSave=1
BacupPath=$P\\Bak\\
AutoIndent=1
ApiList=1
ApiToolTip=1
Properties=1
MouseWheel=1
SaveSize=1
Maximize=1
ApiConst=1
CodeWrite=1
TabToSpc=0
ApiStruct=1
PrnFont=Courier New
PrnFontHeight=-12
PrnFontWeight=400
PrnFontCharSet=0
PrnPage=20990,29690,1000,1000,1000,1000,0
PrnOption=2,0,1,1,1
PrnColor=0,32768,8421504,255,65280,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760,10485760
ApiWordConv=1
ApiWordLocal=1
CodeFiles=.txt.osc.api.
ProcsToApi=1
ProcInSBar=1
CmntBlock=/\*,\*/
CmntChar=//
LnrFont=Terminal
LnrFontHeight=-8
TxtFontItalic=0
ApiShiftSpace=1
LnrWidth=4
LnrOnOpen=0
PageSize=63
OpenCollapsed=0
LnSpc=0
AutoBrackets=1
CodeTooltip=1
HexFont=Courier New
HexFontHeight=-12
HexFontWeight=400
HexFontItalic=0
HexFontCharSet=0
DlgFont=Tahoma
DlgFontHeight=-12
DlgFontWeight=400
DlgFontItalic=0
DlgFontCharSet=0
ToolFont=MS Sans Serif
ToolFontHeight=-12
ToolFontWeight=400
ToolFontItalic=0
ToolFontCharSet=134
ThreadBuild=1
ChangeNotify=1
HiliteLine=0
EnterOnTab=0
HiliteCmnt=0

\[Dialog\]
Grid=1
GridX=3
GridY=3
SnapToGrid=1
ShowSize=1
SaveRC=1
SimpleProperty=0
DlgID=1000
CtrlID=1001

\[Error\]
BookMark=2
Text=$R\\error.txt

\[Paths\]
$A=D:\\Program Files\\OllyICE
$B=$A
$D=$R
$H=$A
$I=$A
$L=$A
$P=E:\\work
$S=$R\\
$T=$R\\
$M=$R\\
$E=D:\\Program Files

\[MakeFiles\]
0=.rap
1=.txt
2=.osc

\[Project\]
Type=ollyscript(txt),ollyscript(osc)
Files=txt,osc
Folders=
MenuMake=
Group=1
GroupExpand=1

\[ollyscript(txt)\]
Files=1,0
Folders=1,1,1
MenuMake=0,0,0,0,0,0,0,0,0,0
1=
2=
3=
4=
5=
6=
7=
11=
12=
13=
14=
15=
16=
17=0,0,"$E\\OllyICE",5

\[ollyscript(osc)\]
Files=0,1
Folders=1,1,1
MenuMake=0,0,0,0,0,0,0,0,0,0
1=
2=
3=
4=
5=
6=
7=
11=
12=
13=
14=
15=
16=
17=0,0,"$E\\OllyICE",5

\[MenuMake\]
1=&Compile resource,372,M,1
2=Assemble,116,M,2
3=&Link,1652,M,3
4=&Build,1396,M,2,3
5=-,0,M,
6=&Go,628,M,2,3,4
7=-,0,M,
8=&Run,884,M,4
9=Run w/&Debug,580,M,7
10=-,0,M,
11=Go &All,1140,M,1,2,3,4
12=-,0,M,
13=Compile &Module,0,M,6
Ver=100

\[Color\]
UseColor=1
DivLine=1
NoFlicker=0
Back=2039583
Fore=12632256
SelBack=16711680
SelText=16777215
HiLine1=6316128
HiLine2=12644544
HiLine3=65280
Bar=12550399
BarPen=8421504
LnrFontColor=8388608
Cmnt=8454143
Str=16777215
Oper=2631935
C0=8454016
C1=13138999
C2=33521664
C3=12615935
C4=12615808
C5=11184640
C6=4227327
C7=33489151
C8=33488896
C9=16711808
C10=285147136
C11=16711680
C12=16711680
C13=16711680
C14=16711680
C15=16711680
Output=12058623
Project=12058623
Property=12058623
Dialog=8421376
CustColors=0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0
Outputtext=0
Projecttext=0
Propertytext=0
Num=8421504

\[KeyWords\]
;寄存器
C0=eax ebx ecx edx ebp esp esi edi eip
;语法
C1=#inc #LOG add ai
;保留变量
C2=RESULT RESULT\_1 RESULT\_2 VERSION triones
C3=
C4=
C5=
C6=
C7=
C8=
C9=
C10=
C11=
C12=
C13=
C14=
C15=
;On Hold
C16=

\[ReallyRad\]
AddDlg=1
AddMnu=1
AddRes=1
AddVer=1
AddStr=1
CtlClk=1
MnuSel=1
AddFile=1
AddFolder=1
CtlNme=1
\[RadHelp\]
KeywordCacheFile=D:\\RadASM\\AddIns\\hlp15.tmp
\[Group\]
Group=Added files,Resources
\[MenuHelp\]
Ver=100
1=&RadASM,0,H,$R\\Help\\RadASM.chm
\[MenuTools\]
Ver=100
1=&Notepad,0,T,NOTEPAD.EXE
2=&Calculator,0,T,CALC.EXE
3=Note&pad (.rap),0,T,NOTEPAD.EXE,0
4=-,0,T,
\[MenuMacro\]
Ver=100

```

ollyscript.api  
代码:

```
add,dest,src:两数相加
ai  自动步入
an
and,dest,src
ask,"msg"
asm,addr,instruct
ao
bc
bp
bpcnd,addr,"condition"
bpl,addr,"expression"
bplcnd,addr,"expression","condition"
bpmc
bphwc,addr
bphws,addr,"x/r/w"
bpwm,addr,nsize
cmp,dest,src
cmt,addr,"str"
COB
COE
dbh
dbs
dec,var
dm,addr,nsize,"filename"
dma,addr,nsize,"filename"
dpe,"filename",EOP
eob,label
eoe,label
esti
esto
eval,"expression"
exec
ende
find,addr,#hex#
findop,addr,#hex#
fill,addr,nvalue
gn,addr
gpa,"func\_name","dll\_name"
go,addr
GMI,addr,MODULEBASE/MODULESIZE/CODEBASE/CODESIZE
inc,dest
ja,lable
jae,lable
jb,lable
jbe,lable
je,lable
jmp,lable
jne,lable
lbl,addr,"lable\_name"
log,dest
mov,dest,src
msg,"msg"
MSGYN,"msg"
or,dest,src
pause
repl,addr,#find\_code#,#replace\_code#,nsize
ret
rtr
rtu
run
shl,dest,n
shr,dest,n
sti
sto
sub,dest,src
ti
ticnd,"condition"
to
tocnd,"condition"
var,var
xor,dest,src
#inc,"filename"
#log

```

晕了，由于 ini 里有关键字，所以排版不对，还是把打包好的文件传上来吧。。。  
看了 RadASM.ini 的帮助说明与 Assmber.ini 的帮助说明，还是没有很大的帮助，英文的说明看得我眼晕。  
于是用折衷的办法，把这些指令放到 struct 提示里，这样就损失了 API 提示了，但是至少不会乱填充字符。然后就是把那些指令说明给写了、对 ollyscript 的一些专有的语法提示给调整了一下。就这样吧。呵呵。  
附件里有效果图，如果你觉得还好就用，如果觉得不好就当玩具喽，更好的情况就是我这个砖头能引出一块和氏璧来，嘿嘿。

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || ollyscript\_for\_RadASM.rar  
 |
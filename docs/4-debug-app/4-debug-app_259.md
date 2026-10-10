

# 改几个游戏

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-32.htm`](https://www.pediy.com/kssd/pediy08/pediy8-32.htm)

首先来修改个简单的，打砖块的游戏，DXBALL2  
先运行游戏，用游戏修改器（我用的是 GameExpert）找到保存生命的地址 43B1BC  
接着用 W32DASM 反汇编主程序 DXBALL2.EXE，搜索 43B1BC，找到以下代码段  
  
:00405645 A1BCB14300              mov eax, dword ptr \[0043B1BC\]  
:0040564A 8B0D24B24300            mov ecx, dword ptr \[0043B224\]  
:00405650 48                      dec eax  
:00405651 6A00                    push 00000000  
:00405653 A3BCB14300              mov dword ptr \[0043B1BC\], eax  
:00405658 890D108E5200            mov dword ptr \[00528E10\], ecx  
:0040565E C705A484520001000000    mov dword ptr \[005284A4\], 00000001  
:00405668 B9A0CB4F00              mov ecx, 004FCBA0  
:0040566D E84E350100              call 00418BC0  
  
把 00405650 48    dec eax 这句 nop 掉，生命值就不减了。  
  
至于游戏里的加强能力，没有是 0，有是 1，也很容易找到。  
  
编写修改器如下，VB6，XP1 通过。程序写得很烂，还望高手指教。  
  
Public combot As Long  
Public sd As Long  
  
  
Private Sub start\_Click()  
Dim hwnd As Long ' 储存 FindWindow 函数返回的句柄  
Dim pid As Long ' 储存进程标识符( Process Id )  
  
  
hwnd = FindWindow(vbNullString, "DX-Ball 2  (Press P to pause, and Alt-Tab to switch apps)")  
If (hwnd = 0) Then  
MsgBox "找不到游戏进程！", vbOKOnly, "出错！"  
Exit Sub  
End If  
  
GetWindowThreadProcessId hwnd, pid  
  
  
pHandle = OpenProcess(PROCESS\_VM\_OPERATION Or PROCESS\_VM\_READ Or PROCESS\_VM\_WRITE, 0, pid)  
  
  
If (pHandle = 0) Then  
MsgBox "发生错误！"  
Exit Sub  
Else: btnPasteName.Caption = "修改开始！": Command1.Enabled = True: Command2.Enabled = True  
  
End If  
EnableDebugPriv  
  
End Sub  
  
Private Sub Command1\_Click()  
sd = (sd + 1) Mod 2 '开关自动锁定  
If sd = 1 Then  
    ReadTimer.Enabled = True: Command1.Caption = "已经自动锁定"  
Else  
    ReadTimer.Enabled = False: Command1.Caption = "已经取消锁定"  
End If  
End Sub  
  
Private Sub Command2\_Click()  
Call ReadTimer\_Timer  
End Sub  
  
Private Sub Form\_Load()  
combot = 1  
sd = 0  
End Sub  
  
Private Sub Form\_Unload(Cancel As Integer)  
CloseHandle pHandle  
End Sub  
  
Private Sub ReadTimer\_Timer() '修改代码  
If CD0.Value = True Then combot = 0  
If CD1.Value = True Then combot = 1  
If CD2.Value = True Then combot = 2  
If CD3.Value = True Then combot = 3  
If CD4.Value = True Then combot = 4  
WriteProcessMemory pHandle, &H4FCBD0, combot, 1, ByVal 0& '长度  
WriteProcessMemory pHandle, &H52C660, ZD.Value, 1, ByVal 0& '子弹  
WriteProcessMemory pHandle, &H5284B8, HQ.Value, 1, ByVal 0& '火球  
WriteProcessMemory pHandle, &H52AEE0, CT.Value, 1, ByVal 0& '穿透  
WriteProcessMemory pHandle, &H5284B4, XY.Value, 1, ByVal 0& '吸引  
End Sub  
  
  
\------------------------  
接来来是另一个打砖块的游戏，幻想游戏里的星际弹球之失落的世界  
先用上面的方法找到保存生命的地址 D1ED1C。  
但是你如果反汇编主程序的话，照上面那种搜索方法，会毫无结果。  
再进一次游戏，会发现保存生命的地址又变成 D383B4，每一次都会变，所以这次要请出 OD 了  
  
用游戏修改器找到地址后，不要关闭游戏，直接用 OD 的附加，附加后（假设这次的地址是 D383B4），下硬件断点 hw D383B4，死掉一条命后，中断  
  
0046510A  |.  8B48 18       MOV ECX,DWORD PTR DS:\[EAX+18\]  
0046510D  |.  83C0 18       ADD EAX,18  
00465110  |.  41            INC ECX  
00465111  |.  8908          MOV DWORD PTR DS:\[EAX\],ECX  
00465113  |.  FF4F 44       DEC DWORD PTR DS:\[EDI+44\]      很明显这就是生命数减 1 的语句  
00465116  |.  8B0D 5CB16400 MOV ECX,DWORD PTR DS:\[64B15C\]  中断在这里  
0046511C  |.  E8 4F3F0700   CALL Rebound\_.004D9070  
  
  
  
0046510A  |.  8B48 18       MOV ECX,DWORD PTR DS:\[EAX+18\]  
0046510D  |.  83C0 18       ADD EAX,18  
00465110  |.  41            INC ECX  
00465111  |.  8908          MOV DWORD PTR DS:\[EAX\],ECX  
00465113      90            NOP  
00465114      90            NOP  
00465115      90            NOP  
00465116  |.  8B0D 5CB16400 MOV ECX,DWORD PTR DS:\[64B15C\]  
0046511C  |.  E8 4F3F0700   CALL Rebound\_.004D9070  
  
  
把它 NOP 掉，如上，进游戏一看，生命已经不再减了  
  
顺带说一下，调试全屏游戏时，如果中断后 OD 显示不出来，请把 OD 的总在最前打开就行，或者用 D3DWINDOW 把游戏窗口化。大多数游戏地址都不是直接引用和固定的，所以还是得靠调试器来进行才是。  
  
\------------------------  
  
接上次的文章，监狱 1.0  
http://bbs.pediy.com/showthread.php?threadid=19655   
  
XF00000000X  
F0000000000  
00000000000  
00000000000  
………  
F0000000000  
X000000000X  
  
在棋盘上置三个防守棋子（F），如图示，这时中断，来到内存地址  
  
00971FA0  00 03 00 00 00 00 00 00 00 03 00 03 00 00 00 00  .............  
00971FB0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00971FC0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00971FD0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00971FE0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00971FF0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00972000  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00972010  00 00 00 00 00 00 00 00 00 00 00 00 14 00 00 00  ...............  
00972020  03 00 00 00 14 00 00 00 06 00 00 00 00 00 00 00  .............  
  
971FA0 和 971FA9 都有 03，证明棋子在内存中是以第一列开始从上到下的顺序储存的。971FAA 是 00，971FAB 是 03，说明左下角的 X，虽然不能放置棋子，但是内存空间中它照样占有位置，照理四个角落的棋子都应该如此。  
这里采用的是直接写入内存的方式来修改，初步设想在窗口上画它 121 个棋子（四个角落的隐藏掉就行），通过点击使棋子依次变为灰色（无棋子），红色（红方棋子），蓝色（蓝方棋子），白色（防守棋子），并把相应数据写入游戏内存。  
  
当我在 VB 的窗口上扔上 121 个的 Shape 控件数组并把它们一一拉到相应位置并写了一点代码后，点击下去却全点没有反应。认真的一试，原来 Shape 控件没有任何事件，连 click 都没有#-\_-。不想再重来，于是就决定以点击时鼠标的座标为依据判断点击的是哪个位置。  
  
写修改器如下，VB6，XP1 通过。程序写得很烂，还望高手指教。  
  
Dim hang, lie, index2 As Long  
Private Sub Command1\_Click()  
Dim hwnd As Long ' 储存 FindWindow 函数返回的句柄  
Dim pid As Long ' 储存进程标识符( Process Id )  
  
  
hwnd = FindWindow(vbNullString, "监狱(Quod)")  
If (hwnd = 0) Then  
MsgBox "找不到游戏进程！", vbOKOnly, "出错！"  
Exit Sub  
End If  
  
GetWindowThreadProcessId hwnd, pid  
  
  
pHandle = OpenProcess(PROCESS\_VM\_OPERATION Or PROCESS\_VM\_READ Or PROCESS\_VM\_WRITE, 0, pid)  
  
  
If (pHandle = 0) Then  
MsgBox "发生错误！"  
Exit Sub  
Else: Command1.Caption = "修改开始！"  
  
End If  
EnableDebugPriv  
  
End Sub  
  
Private Sub Form\_MouseDown(Button As Integer, Shift As Integer, X As Single, Y As Single)  
Dim i As Byte, j As Byte  
Dim readad As Long, readad2 As Long  
If (pHandle = 0) Then  
MsgBox "找不到游戏进程！", vbOKOnly, "出错！"  
Exit Sub  
End If  
For i = 0 To 120  
    readad = &H971FA0 + i  
    ReadProcessMemory pHandle, readad, j, 2, ByVal 0&  
    Select Case j  
        Case 0  
            readad = &H8000000F '根据内存数据改变棋子颜色  
        Case 1  
            readad = &HFF&  
        Case 2  
            readad = &HFF0000  
        Case 3  
            readad = &HFFFFFF  
    End Select  
    Shape1(i).FillColor = readad  
Next i  
Select Case X '这两个 case 判断鼠标点击的是哪一个棋子  
    Case 120 To 615  
        lie = 1  
    Case 720 To 1215  
        lie = 2  
    Case 1320 To 1815  
        lie = 3  
    Case 1920 To 2415  
        lie = 4  
    Case 2520 To 3015  
        lie = 5  
    Case 3120 To 3615  
        lie = 6  
    Case 3720 To 4215  
        lie = 7  
    Case 4320 To 4815  
        lie = 8  
    Case 4920 To 5415  
        lie = 9  
    Case 5520 To 6015  
        lie = 10  
    Case 6120 To 6615  
        lie = 11  
    Case Else  
        lie = 0  
End Select  
Select Case Y  
    Case 120 To 615  
        hang = 1  
    Case 720 To 1215  
        hang = 2  
    Case 1320 To 1815  
        hang = 3  
    Case 1920 To 2415  
        hang = 4  
    Case 2520 To 3015  
        hang = 5  
    Case 3120 To 3615  
        hang = 6  
    Case 3720 To 4215  
        hang = 7  
    Case 4320 To 4815  
        hang = 8  
    Case 4920 To 5415  
        hang = 9  
    Case 5520 To 6015  
        hang = 10  
    Case 6120 To 6615  
        hang = 11  
    Case Else  
        hang = 0  
End Select  
If (lie <> 0 And hang <> 0) Then '点击后改变棋子颜色并写入游戏内存  
    index2 = ((lie - 1) \* 11 + hang - 1)  
    readad2 = &H971FA0 + index2  
    If Shape1(index2).FillColor = &H8000000F Then  
        Shape1(index2).FillColor = &HFF&: WriteProcessMemory pHandle, readad2, 1, 1, ByVal 0&  
    ElseIf Shape1(index2).FillColor = &HFF& Then  
        Shape1(index2).FillColor = &HFF0000: WriteProcessMemory pHandle, readad2, 2, 1, ByVal 0&  
    ElseIf Shape1(index2).FillColor = &HFF0000 Then  
        Shape1(index2).FillColor = &HFFFFFF: WriteProcessMemory pHandle, readad2, 3, 1, ByVal 0&  
    ElseIf Shape1(index2).FillColor = &HFFFFFF Then  
        Shape1(index2).FillColor = &H8000000F: WriteProcessMemory pHandle, readad2, 0, 1, ByVal 0&  
    End If  
End If  
End Sub  
  
  
写完之后测试了一下，修改器显示的是正常，但是游戏界面却没有显示出棋子，在修改器里面改了之后虽然游戏里没有显示棋子，但照样显示“某方胜利”。  
由此可知游戏在点击后才依据点击的位置来画棋子，并不是每次刷新。而且判断胜利也是用的上面的内存数据来判断。  
根据上次分析的结果下断，来到此处。  
  
  
00462464  |> \\8D1416        LEA EDX,DWORD PTR DS:\[ESI+EDX\]  
00462467      889C02 480300>MOV BYTE PTR DS:\[EDX+EAX+348\],BL         ;  这行里 BL 存的是棋子类型，用 MOV 写进内存  
0046246E      8B4D F8       MOV ECX,DWORD PTR SS:\[EBP-8\]  
00462471  |.  8B55 FC       MOV EDX,DWORD PTR SS:\[EBP-4\]  
00462474  |.  8BC6          MOV EAX,ESI  
00462476      E8 5DF4FFFF   CALL QUOD.004618D8                       ;  NOP 掉这一行，结果发现棋子不见了，由此跟进这个 CALL 里  
0046247B  |.  8BC6          MOV EAX,ESI  
0046247D  |.  E8 1AF0FFFF   CALL QUOD.0046149C  
00462482  |.  8BC6          MOV EAX,ESI  
00462484  |.  E8 8FEBFFFF   CALL QUOD.00461018  
00462489  |.  80BE 9A420000>CMP BYTE PTR DS:\[ESI+429A\],0             ;  是不是没有棋子  
00462490  |.  75 0C         JNZ SHORT QUOD.0046249E  
00462492  |.  80FB 03       CMP BL,3                                 ;  或者是防守棋子，不是的话，下面的 CALL 改变下棋方  
00462495  |.  74 07         JE SHORT QUOD.0046249E  
00462497  |.  8BC6          MOV EAX,ESI  
00462499  |.  E8 FEEEFFFF   CALL QUOD.0046139C                       ;  改变下棋方  
0046249E  |>  5E            POP ESI  
0046249F  |.  5B            POP EBX  
004624A0  |.  59            POP ECX  
004624A1  |.  59            POP ECX  
004624A2  |.  5D            POP EBP  
004624A3  \\.  C2 0C00       RETN 0C  
  
跟进，前面省略一部分代码  
  
00461951  |.  2C 01         SUB AL,1                                 ;  Switch (cases 0..3)  
00461953  |.  72 0C         JB SHORT QUOD.00461961  
00461955  |.  74 19         JE SHORT QUOD.00461970  
00461957  |.  FEC8          DEC AL  
00461959  |.  74 24         JE SHORT QUOD.0046197F  
0046195B  |.  FEC8          DEC AL  
0046195D  |.  74 2F         JE SHORT QUOD.0046198E  
0046195F  |.  EB 3A         JMP SHORT QUOD.0046199B  
00461961  |>  BA C0C0C000   MOV EDX,0C0C0C0                          ;  Case 0 of switch 00461951  
00461966  |.  8B43 14       MOV EAX,DWORD PTR DS:\[EBX+14\]  
00461969  |.  E8 32F9FBFF   CALL QUOD.004212A0  
0046196E  |.  EB 2B         JMP SHORT QUOD.0046199B  
00461970  |>  BA FF000000   MOV EDX,0FF                              ;  Case 1 of switch 00461951  
00461975  |.  8B43 14       MOV EAX,DWORD PTR DS:\[EBX+14\]  
00461978  |.  E8 23F9FBFF   CALL QUOD.004212A0  
0046197D  |.  EB 1C         JMP SHORT QUOD.0046199B  
0046197F  |>  BA 0000FF00   MOV EDX,0FF0000                          ;  Case 2 of switch 00461951  
00461984  |.  8B43 14       MOV EAX,DWORD PTR DS:\[EBX+14\]  
00461987  |.  E8 14F9FBFF   CALL QUOD.004212A0  
0046198C  |.  EB 0D         JMP SHORT QUOD.0046199B  
0046198E  |>  BA FFFFFF00   MOV EDX,0FFFFFF                          ;  Case 3 of switch 00461951  
00461993  |.  8B43 14       MOV EAX,DWORD PTR DS:\[EBX+14\]  
00461996      E8 05F9FBFF   CALL QUOD.004212A0                         
0046199B  |>  33D2          XOR EDX,EDX                              ;  Default case of switch 00461951  
  
OD 注释得真是详细，很明显看出，上面的 Case 根据内存中的数据判断棋子类型，并把相关颜色信息赋值  
  
继续往下走，用 NOP 法判断之后来到下面的代码  
  
00421644  /$  55            PUSH EBP  
00421645  |.  8BEC          MOV EBP,ESP  
00421647  |.  53            PUSH EBX  
00421648  |.  56            PUSH ESI  
00421649  |.  57            PUSH EDI  
0042164A  |.  8BF9          MOV EDI,ECX  
0042164C  |.  8BF2          MOV ESI,EDX  
0042164E  |.  8BD8          MOV EBX,EAX  
00421650  |.  8BC3          MOV EAX,EBX  
00421652  |.  8B10          MOV EDX,DWORD PTR DS:\[EAX\]  
00421654  |.  FF52 10       CALL DWORD PTR DS:\[EDX+10\]  
00421657  |.  8A15 88164200 MOV DL,BYTE PTR DS:\[421688\]  
0042165D  |.  8BC3          MOV EAX,EBX  
0042165F  |.  E8 58050000   CALL QUOD.00421BBC  
00421664  |.  8B45 08       MOV EAX,DWORD PTR SS:\[EBP+8\]  
00421667  |.  50            PUSH EAX                                 ; /Bottom  
00421668  |.  8B45 0C       MOV EAX,DWORD PTR SS:\[EBP+C\]             ; |  
0042166B  |.  50            PUSH EAX                                 ; |Right  
0042166C  |.  57            PUSH EDI                                 ; |Top  
0042166D  |.  56            PUSH ESI                                 ; |Left  
0042166E  |.  8B43 04       MOV EAX,DWORD PTR DS:\[EBX+4\]             ; |  
00421671  |.  50            PUSH EAX                                 ; |hDC  
00421672  |.  E8 6D55FEFF   CALL <JMP.&GDI32.Ellipse>                ; \\Ellipse  
00421677  |.  8BC3          MOV EAX,EBX  
00421679  |.  8B10          MOV EDX,DWORD PTR DS:\[EAX\]  
0042167B  |.  FF52 0C       CALL DWORD PTR DS:\[EDX+C\]  
0042167E  |.  5F            POP EDI  
0042167F  |.  5E            POP ESI  
00421680  |.  5B            POP EBX  
00421681  |.  5D            POP EBP  
00421682  \\.  C2 0800       RETN 8  
  
  
查了一下 Ellipse 函数（网上找到的是 VB 的，不过 API 应该都一样）  
  
VB 声明   
Declare Function Ellipse Lib "gdi32" Alias "Ellipse" (ByVal hdc As Long, ByVal X1 As Long, ByVal Y1 As Long, ByVal X2 As Long, ByVal Y2 As Long) As Long   
说明   
描绘一个椭圆，由指定的矩形围绕。椭圆用当前选择的画笔描绘，并用当前选择的刷子填充   
返回值   
Long，非零表示成功，零表示失败。会设置 GetLastError   
参数表   
参数 类型及说明   
hdc Long，设备场景的句柄   
X1，Y1 Long，约束矩形采用逻辑坐标的左上角位置   
X2，Y2 Long，约束矩形采用逻辑坐标的右下角位置   
  
把上面 Right，Top，Left 等参数修改再运行  
  
  
  
可见点击后，游戏先判断棋子类型，并把它写入内存，再在界面上用 Ellipse 函数画一个圆并填充，和我修改器用的改变控件颜色是不同方法。至于用修改器改完，要让游戏界面显示出棋子，目前对我来说显得比较困难，暂时搁下了。 

 4dd466d0e1e0cc123afbe504da2462ca
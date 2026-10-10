

# 蜘蛛纸牌游戏代码的修改

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-80.htm`](https://www.pediy.com/kssd/pediy07/pediy7-80.htm)

【逆文作者】moon  
【所用工具】flyODBG1.10，eXeScope6.30  
【逆向对象】WinXP 附带的蜘蛛纸牌游戏 spider.exe  
【逆文目标】给蜘蛛纸牌增加撤消发牌功能，并对原撤消功能进行改进  
  
【逆文正文】  
  
    看到 xdkui 大侠的《蜘蛛纸牌分析与简单 DIY》想要照着做一下。先找到了一个“蜘蛛.exe”，反汇编的结果和他的文章中的不一样，于是自己改了一下。后来又找到一个“spider.exe”，反汇编结果和 xdkui 大侠的又一样了，可他没有提供修改程序的方法，我试着再改一次吧。  
  
    要想撤消发牌，理论上讲就是把发牌后程序所有操作全部撤消。但要想知道程序都做了哪些事情，实在是一件麻烦的事情，所以偷个懒，利用一下 xdkui 大侠的成果。根据 xdkui 的贴子《蜘蛛纸牌分析与简单 DIY》可知，实现撤消发牌，程序需要做的事情有：  
  
a. 把下一次要发的牌的序号减小 10  
b. 把发牌次数值减小一  
c. 去掉 10 个链表的最后一个节点  
d. 把 10 个链表的节点数各减一  
e. 刷新蜘蛛窗体  
  
    要让程序实现这些操作，我们需要做的事情有：  
  
1. 找到一块空白区域，用来存放我们的代码  
2. 找到要操作的各数据的存放位置  
3. 添加一个菜单项“撤消发牌”  
4. 在原窗口过程中增加一个分支，跳到我们的代码处  
5. 编写我们的代码，实现上面的 a~e  
  
【1. 找空白区】  
  
    用 flyODBG1.10 打开 spider.exe 后，上下翻看代码区，可知 0100FAE8-0100FFFF 是一块足够大的空白段，可以放自己的代码。  
  
【2. 找数据位置】  
  
    要找到数据的存放位置，没有什么太好的办法，跟踪程序吧。  
  
    用 eXeScope 打开 spider.exe，在“资源”-->“菜单”中点开菜单 101，可以看到各菜单项的相关信息，“新一轮发牌”的 ID 为 40007，“发牌”的 ID 为 40016。（eXeScope 的操作很简单，顺手增加一个菜单项，ID 用 40008，提示信息写“撤消发牌”。）  
  
    在 od 中下断：bpx RegisterClassW 或 bpx DefWindowProcW，都可以找到窗口过程。通过分析窗口过程，可知点击“新一轮发牌”或“发牌”后，程序的实质执行体为：  
  
0100692C  mov ecx,esi                         ;  Cases 9C47,9C50 of switch 01006884  
0100692E  call spider.01006604  
  
    进入这个 call，看到：  
  
0100660A  cmp dword ptr ds:\[esi+58\],5  5 是发牌次数的限定值，那么 ecx+58 就是发牌次数的地址啦  
0100660E  jge spider.0100680F  
  
01006687  /mov ecx,dword ptr ds:\[esi+4\]  
0100668A  |mov ebp,dword ptr ds:\[ecx+10\]  
0100668D  |call spider.010070A8  
01006692  |mov ecx,dword ptr ds:\[esi+4\]  
01006695  |push 1  
01006697  |push ebp  
01006698  |call spider.01006F91  
0100669D  |mov ecx,dword ptr ds:\[esi+8\]  
010066A0  |push 1                             ; /Arg3 = 00000001  
010066A2  |push ebp                           ; |Arg2  
010066A3  |push ebx                           ; |Arg1  
010066A4  |call spider.01007A75               ; \\spider.01007A75  
  
  
    esi 中为 01010FC8，是窗口句柄，也是数据总的起始点。到\[esi+4\]、\[esi+8\]去看看，跟踪一下发牌过程，可知：  
  
1. 01010FC8 是数据总的起点，  
2. \[\[01010FC8+4\]+10\]是将要发的牌的序号，  
3. \[01010FC8+8\]连续存放着十个链表的表头的地址的地址，  
4. \[\[01010FC8+8\]+列数\*4+28\]是十个链表的长度，即每列牌的张数。  
5. \[01010FC8+58\]是已发牌的次数  
  
【3. 改代码】  
  
    分析可知下面是根据菜单项 ID 值执行操作的起始点：  
  
01006884  add eax,FFFF63BE                    ;  Switch (cases 9C42..9C51)  
01006889  cmp eax,0F  
  
    把 01006884 一行改为：  
  
01006884  jmp 0100FAE8  
  
    以执行我们编写的代码。  
  
【4. 编写代码】    
  
    在 0100FAE8 处编写代码，打算稿如下：  
  
cmp eax,9c48  
jnz j1  首先看是否选了菜单项“撤消发牌”，没点这一项，去执行原来的代码  
pushad  
mov esi,01010FC8  
mov ecx,esi  
cmp dword ptr \[ecx+58\],0  
jz j2  看是否发过牌，如没发过牌，也不做任何处理  
dec dword ptr \[ecx+58\]  第 1 件事，发牌次数减 1  
  
mov ecx,\[ecx+4\]  
sub dword ptr ds:\[ecx+10\],0A  第 2 件事，将要发的牌的序号减 10  
  
mov ecx,esi  
mov ecx,\[ecx+8\]  
xor edi,edi  
  
j4:  
mov ebx,\[ecx+edi\*4+28\]  
dec ebx  
mov \[ecx+edi\*4+28\],ebx    第 3 件事，链表节点数减 1  
  
mov eax,\[ecx+edi\*4\]  
mov eax,\[eax\]  
dec ebx  
j3:  
mov eax,\[eax+8\]  
dec ebx  
jnz j3  
xor edx,edx  
mov \[eax+8\],edx      第 4 件事，链表尾去掉  
inc edi  
cmp edi,0a  
jnz j4  
  
(  
push \[01010FC8\]  
call GetMenu  
mov edi,eax  
push 0   
push 9C50   
push edi  
call EnableMenuItem  后来发现的第 6 件事，使“发牌”菜单项可用，注解见下文  
push 0   
push 9C4A  
push edi  
call EnableMenuItem  后来发现的第 7 件事，使“撤消”菜单项可用，注解见下文  
)  
  
push 1  
push 0  
push 0  
call InvalidateRect  第 5 件事，刷新蜘蛛窗体  
  
j2:  
popad  
j1:  
sub eax,9C42  
jmp 01006889  
  
   调试过程中发现，按“call GetMenu”这种格式写，存在到另一台机器上不能用的问题。观察了一下，原来要写输入表中的地址，如“call GetMenu”要写成：call \[0100126C\]。共用到如下四个函数：  
  
GetMenu    0100126C > .  BEEAD377      dd USER32.GetMenu  
InvalidateRect  01001184 > .  9DB4D177      dd USER32.InvalidateRect  
EnableMenuItem  01001180 > .  3CFCD177      dd USER32.EnableMenuItem  
GetSubMenu  010011DC > .  5A35D277      dd USER32.GetSubMenu  
  
【5. 初步完善】  
  
    粗略测试，经以上修改后的 BUG 有：  
  
1. 移动牌后仍可以撤消发牌  
2. 发过 5 次牌后，发牌菜单项不可用，撤消发牌后仍不可用  
3. 发牌后撤消功能不可用，撤消发牌后问题仍不能解决  
  
    根据这些 BUG，可以编程实现如下功能：  
  
a. 移动牌后使“撤消发牌”菜单项不可用  
b. 发牌后使“撤消发牌”菜单项可用  
c. 撤消发牌后使“发牌”菜单项可用  
d. 使撤消发牌后“撤消”功能可用  
  
1. 功能 a  
  
    因为移动牌后程序会使“撤消”菜单项可用，因此很容易找到实现 a 的代码所放的位置。下断：bpx EnableMenuItem，移动一张牌后断于：  
  
01003A10  push dword ptr ds:\[edi\]            ; /hWnd  
01003A12  call dword ptr ds:\[<&USER32.GetMen>; \\GetMenu  
01003A18  push esi                           ; /Flags  
01003A19  push 9C4A                          ; |ItemID = 9C4A (40010.)  
01003A1E  push eax                           ; |hMenu  
01003A1F  call dword ptr ds:\[<&USER32.Enable>; \\EnableMenuItem  断于此  
01003A25  cmp esi,96  
  
    把 01003A18 一行改为：  
  
01003A18  jmp 0100FB6F  
  
    在 0100FB6F 处汇编如下代码：  
  
push eax  
push 0    
push 9C4A  
push eax   
call EnableMenuItem  
pop eax  
push 1    
push 9C48  
push eax   
call EnableMenuItem  
push \[01010FC8\]  
call DrawMenuBar  
push 1  
push 0  
push 0  
call InvalidateRect  
jmp 01003A25  
  
2. 功能 b  
  
    发牌后程序会使“撤消”菜单项不可用，因此很容易找到实现 b 的代码所放的位置。  
  
0100311A  push dword ptr ds:\[ecx\]           ; /hWnd  
0100311C  and dword ptr ds:\[ecx+F0C\],0      ; |  
01003123  call dword ptr ds:\[<&USER32.GetMe>; \\GetMenu  
01003129  push 1                            ; /Flags = MF\_BYCOMMAND|MF\_GRAYED|MF\_STRING  
0100312B  push 9C4A                         ; |ItemID = 9C4A (40010.)  
01003130  push eax                          ; |hMenu  
01003131  call dword ptr ds:\[<&USER32.Enabl>; \\EnableMenuItem  
01003137  retn  
  
    把 01003129 一行改为：  
  
01003129  jmp 0100FB9E  
  
    在 0100FB9E 处汇编如下代码：  
  
push eax  
push 1    
push 9C4A  
push eax   
call EnableMenuItem  
pop eax  
push 0    
push 9C48  
push eax   
call EnableMenuItem  
push \[01010FC8\]  
call DrawMenuBar  
push 1  
push 0  
push 0  
call InvalidateRect  
jmp 01003137  
  
3. 功能 c  
  
    看一下程序中发牌 5 次后改变菜单项使能状态的代码，参考一下方法，然后在 0100FAE8 处的代码增补上第 6 件事的代码：  
  
010067AA  push dword ptr ds:\[esi\]   ; /hWnd  
010067AC  call dword ptr ds:\[<&USER>; \\GetMenu  
010067B2  push 1                    ; /Flags = MF\_BYCOMMAND|MF\_GRAYED|MF\_STRING  
010067B4  push 9C47                 ; |ItemID = 9C47 (40007.)  
010067B9  mov ebx,eax               ; |  
010067BB  push ebp                  ; |/Pos  
010067BC  push ebx                  ; ||hMenu  
010067BD  call dword ptr ds:\[<&USER>; |\\GetSubMenu  
010067C3  mov edi,dword ptr ds:\[<&U>; |USER32.EnableMenuItem  
010067C9  push eax                  ; |hMenu  
010067CA  call edi                  ; \\EnableMenuItem  
010067CC  push 1                    ; /Flags = MF\_BYCOMMAND|MF\_GRAYED|MF\_STRING  
010067CE  push 9C50                 ; |ItemID = 9C50 (40016.)  
010067D3  push ebx                  ; |hMenu  
010067D4  call edi                  ; \\EnableMenuItem  
010067D6  push dword ptr ds:\[esi\]   ; /hWnd  
010067D8  call dword ptr ds:\[<&USER>; \\DrawMenuBar  
  
4. 功能 d  
  
    粗略跟踪一下点“撤消”后程序都做了什么，从窗口过程看出点“撤消”后的程序分支为：  
  
01006923  mov ecx,esi                       ;  Case 9C4A of switch 01006884  
01006925  call spider.01004C15  
  
    跟进这个 call，试验几次就可以看出\[esi+F0C\]是移动牌的次数：  
  
01004C15  push esi  
01004C16  mov esi,ecx  
01004C18  dec dword ptr ds:\[esi+F0C\]  这里记录着移动牌的次数  
01004C1E  mov eax,dword ptr ds:\[esi+F0C\]  取次数  
01004C24  lea eax,dword ptr ds:\[eax+eax\*4\]  次数乘以 5  
01004C27  lea eax,dword ptr ds:\[esi+eax\*4+354\]  数据起点为\[esi+354\]，偏移量为次数\*20 字节  
01004C2E  test eax,eax  
01004C30  je short spider.01004C5B  
  
    再跟进点“发牌”后的代码，改变菜单状态前有个 call 0100311A：  
  
01006798  call spider.0100311A  
0100679D  mov ecx,esi  
0100679F  call spider.01002D90  
010067A4  cmp dword ptr ds:\[esi+58\],5  
010067A8  jnz short spider.010067DE  
010067AA  push dword ptr ds:\[esi\]               ; /hWnd  
010067AC  call dword ptr ds:\[<&USER32.GetMenu>\] ; \\GetMenu  改变菜单状态  
  
    跟进 call 0100311A：  
  
0100311A  push dword ptr ds:\[ecx\]                ; /hWnd  
0100311C  and dword ptr ds:\[ecx+F0C\],0            ; |  
01003123  call dword ptr ds:\[<&USER32.GetMenu>\]  ; \\GetMenu  
  
    0100311C 一行把移动牌次数清零，这里如果为零，撤消操作就无法继续了，因此把这一行 nop 掉，或把 and 改为 or：  
  
0100311C  or dword ptr ds:\[ecx+F0C\],0            ;   
  
    然后在撤消发牌后的代码中加入使“撤消”菜单项可用的代码。  
  
【6. 更改总结】      
  
    所有改变后的代码总结如下：  
  
1.  
01006884  jmp 0100FAE8  
  
2.  
01003A18  jmp 0100FB6F  
  
3.  
01003129  jmp 0100FB9E  
  
4.  
0100311C  or dword ptr ds:\[ecx+0F0C\],0   
  
5.  
0100FAE8  cmp eax,9C48  
0100FAED  jnz short spider1.0100FB65  
0100FAEF  pushad  
0100FAF0  mov esi,spider1.01010FC8  
0100FAF5  mov ecx,esi  
0100FAF7  cmp dword ptr ds:\[ecx+58\],0  
0100FAFB  je short spider1.0100FB64  
0100FAFD  dec dword ptr ds:\[ecx+58\]  
0100FB00  mov ecx,dword ptr ds:\[ecx+4\]  
0100FB03  sub dword ptr ds:\[ecx+10\],0A  
0100FB07  mov ecx,esi  
0100FB09  mov ecx,dword ptr ds:\[ecx+8\]  
0100FB0C  xor edi,edi  
0100FB0E  mov ebx,dword ptr ds:\[ecx+edi\*4+28\]  
0100FB12  dec ebx  
0100FB13  mov dword ptr ds:\[ecx+edi\*4+28\],ebx  
0100FB17  mov eax,dword ptr ds:\[ecx+edi\*4\]  
0100FB1A  mov eax,dword ptr ds:\[eax\]  
0100FB1C  dec ebx  
0100FB1D  mov eax,dword ptr ds:\[eax+8\]  
0100FB20  dec ebx  
0100FB21  jnz short spider1.0100FB1D  
0100FB23  xor edx,edx  
0100FB25  mov dword ptr ds:\[eax+8\],edx  
0100FB28  inc edi  
0100FB29  cmp edi,0A  
0100FB2C  jnz short spider1.0100FB0E  
0100FB2E  push dword ptr ds:\[1010FC8\]          ; /hWnd = NULL  
0100FB34  call dword ptr ds:\[<&USER32.GetMenu>>; \\GetMenu  
0100FB3A  mov edi,eax  
0100FB3C  push 0                               ; /Flags = MF\_BYCOMMAND|MF\_ENABLED|MF\_STRING  
0100FB3E  push 9C50                            ; |ItemID = 9C50 (40016.)  
0100FB43  push edi                             ; |hMenu  
0100FB44  call dword ptr ds:\[<&USER32.EnableMe>; \\EnableMenuItem  
0100FB4A  push 0                               ; /Flags = MF\_BYCOMMAND|MF\_ENABLED|MF\_STRING  
0100FB4C  push 9C50                            ; |ItemID = 9C50 (40016.)  
0100FB51  push edi                             ; |hMenu  
0100FB52  call dword ptr ds:\[<&USER32.EnableMe>; \\EnableMenuItem  
0100FB58  push 1                               ; /Erase = TRUE  
0100FB5A  push 0                               ; |pRect = NULL  
0100FB5C  push 0                               ; |hWnd = NULL  
0100FB5E  call dword ptr ds:\[<&USER32.Invalida>; \\InvalidateRect  
0100FB64  popad  
0100FB65  sub eax,9C42  
0100FB6A  jmp spider1.01006889  
0100FB6F  push eax  
0100FB70  push 0                               ; /Flags = MF\_BYCOMMAND|MF\_ENABLED|MF\_STRING  
0100FB72  push 9C4A                            ; |ItemID = 9C4A (40010.)  
0100FB77  push eax                             ; |hMenu  
0100FB78  call dword ptr ds:\[<&USER32.EnableMe>; \\EnableMenuItem  
0100FB7E  pop eax  
0100FB7F  push 1                               ; /Flags = MF\_BYCOMMAND|MF\_GRAYED|MF\_STRING  
0100FB81  push 9C48                            ; |ItemID = 9C48 (40008.)  
0100FB86  push eax                             ; |hMenu  
0100FB87  call dword ptr ds:\[<&USER32.EnableMe>; \\EnableMenuItem  
0100FB8D  push 1                               ; /Erase = TRUE  
0100FB8F  push 0                               ; |pRect = NULL  
0100FB91  push 0                               ; |hWnd = NULL  
0100FB93  call dword ptr ds:\[<&USER32.Invalida>; \\InvalidateRect  
0100FB99  jmp spider1.01003A25  
0100FB9E  push eax  
0100FB9F  push 1                               ; /Flags = MF\_BYCOMMAND|MF\_GRAYED|MF\_STRING  
0100FBA1  push 9C4A                            ; |ItemID = 9C4A (40010.)  
0100FBA6  push eax                             ; |hMenu  
0100FBA7  call dword ptr ds:\[<&USER32.EnableMe>; \\EnableMenuItem  
0100FBAD  pop eax  
0100FBAE  push 0                               ; /Flags = MF\_BYCOMMAND|MF\_ENABLED|MF\_STRING  
0100FBB0  push 9C48                            ; |ItemID = 9C48 (40008.)  
0100FBB5  push eax                             ; |hMenu  
0100FBB6  call dword ptr ds:\[<&USER32.EnableMe>; \\EnableMenuItem  
0100FBBC  push 1                               ; /Erase = TRUE  
0100FBBE  push 0                               ; |pRect = NULL  
0100FBC0  push 0                               ; |hWnd = NULL  
0100FBC2  call dword ptr ds:\[<&USER32.Invalida>; \\InvalidateRect  
0100FBC8  jmp spider1.01003137  
  
【6. 遗留问题】      
  
    经以上修改，已初步具备撤消发牌的功能，还有遗留问题，比如：在经过类似“发牌”--“移动牌”--“发牌”后，可以撤消发牌两次，结果就不对了；撤消发牌后“撤消”仍不可用，等等。目前我还没有完全修改好这些 BUG，先将阶段性成果在第一时间里呈报给大家，其中错误和不足之处希大家批评指正。

 4dd466d0e1e0cc123afbe504da2462ca
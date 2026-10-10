

# diy pe 教学 2 

> 原文：[`www.pediy.com/kssd/pediy05/pediy50235.htm`](https://www.pediy.com/kssd/pediy05/pediy50235.htm)

上篇我讲述了如何修改中游军棋的求和显示到右边的 riched 框，   
这次我教大家如何在上面增加一个按钮，然后如何捕捉这个按钮的事件，当点击这个增加的按钮干一点我们自己想干的事情，好废话少说，突入正题。   
首先用观察军棋的右边有四个按钮，分别是帮助、设置、大厅、退出，好现在我们增加一个按钮叫欢迎（在我自己做的补丁里面这个按钮是作弊，专门用来解散棋局用的，但是限于中游的公平，我不能教大家如何作弊只能教大家如何 diy pe 了）。首先用资源编辑器（我用的是资源黑客）打开   
junqi.exe 观察到这四个按钮如下:   
104 DIALOGEX 0, 0, 213, 364   
STYLE WS\_CHILD   
CAPTION ""   
LANGUAGE LANG\_CHINESE, 0x2   
FONT 9, "宋体"   
{   
  CONTROL "", 1008, EDIT, ES\_LEFT | ES\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 0, 338, 181, 13   
  CONTROL "帮助(&H)", 1014, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 13, 170, 35, 15   
  CONTROL "设置(&S)", 1013, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 67, 170, 35, 15   
  CONTROL "大厅(&P)", 1001, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 13, 190, 35, 15   
  CONTROL "退出(&X)", 1000, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 67, 190, 35, 15   
  CONTROL "", 1016, "RICHEDIT", ES\_LEFT | ES\_MULTILINE | ES\_AUTOVSCROLL | ES\_WANTRETURN | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_VSCROLL | WS\_TABSTOP, 0, 213, 213, 106 , 0x00000200   
  CONTROL "颜色", 1015, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 181, 322, 25, 13   
  CONTROL "", 1005, "{8856F961-340A-11D0-A96B-00C04FD705A2}", 0x50010000, 0, 0, 213, 60   
  CONTROL "List1", 1006, "SysListView32", LVS\_REPORT | LVS\_SINGLESEL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 0, 60, 213, 103   
  CONTROL "", 1002, COMBOBOX, CBS\_DROPDOWN | CBS\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_VSCROLL, 0, 322, 120, 166   
  CONTROL "", 1003, COMBOBOX, CBS\_DROPDOWNLIST | WS\_CHILD | WS\_VISIBLE | WS\_VSCROLL | WS\_TABSTOP, 123, 322, 53, 81   
  CONTROL "发送", 1004, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 182, 338, 25, 14   
}   
好我们现在就增加一个按钮，增加以后的资源如下：   
104 DIALOGEX 0, 0, 213, 364   
STYLE WS\_CHILD   
CAPTION ""   
LANGUAGE LANG\_CHINESE, 0x2   
FONT 9, "宋体", FW\_DONTCARE, FALSE, 0   
{   
  CONTROL "", 1008, EDIT, ES\_LEFT | ES\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 0, 338, 181, 13   
  CONTROL "帮助(&H)", 1014, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 13, 171, 35, 15   
  CONTROL "欢迎(&L)", 1011, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 115, 170, 35, 15   
  CONTROL "设置(&S)", 1013, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 67, 170, 35, 15   
  CONTROL "大厅(&P)", 1001, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 13, 190, 35, 15   
  CONTROL "退出(&X)", 1000, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 67, 190, 35, 15   
  CONTROL "", 1016, "RICHEDIT", ES\_LEFT | ES\_MULTILINE | ES\_AUTOVSCROLL | ES\_WANTRETURN | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_VSCROLL | WS\_TABSTOP, 0, 213, 213, 106 , 0x00000200   
  CONTROL "颜色", 1015, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 181, 322, 25, 13   
  CONTROL "", 1005, "{8856F961-340A-11D0-A96B-00C04FD705A2}", 0x50010000, 0, 0, 213, 60   
  CONTROL "List1", 1006, "SysListView32", LVS\_REPORT | LVS\_SINGLESEL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 0, 60, 213, 103   
  CONTROL "", 1002, COMBOBOX, CBS\_DROPDOWN | CBS\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_VSCROLL, 0, 322, 120, 166   
  CONTROL "", 1003, COMBOBOX, CBS\_DROPDOWNLIST | WS\_CHILD | WS\_VISIBLE | WS\_VSCROLL | WS\_TABSTOP, 123, 322, 53, 81   
  CONTROL "发送", 1004, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 182, 338, 25, 14   
}   
这里和上面不同的是多了下面这句，   
CONTROL "欢迎(&L)", 1011 BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 115, 170, 35, 15   
记住增加按钮要更改其 control id，否则有重复的 id 的话，搞得几个按钮的功能会一样，这是因为 windows 是靠 id 来判别点击的是那个按钮   
我这里把 id 改为 1015，换成 16 进制就是 3f3，用资源黑客编译一下（你用别的资源编辑器如 c++的，都行），运行一下，看看旁边多了一个   
名叫欢迎的按钮，点击这个按钮，怎么没有反应，不要急你还没有给这个按钮加入事件呢，怎么会有反应呢？下一步就是给这个按钮加入   
事件，首先我们要了解 windows 的机制，点击这个按钮后，windows 会 sendmessage 一个 command 消息给应用程序查 api 手册就知道。command   
消息带有 wParam 参数，这个参数就是你的按钮 id，那么我们现在的任务就是找到应用程序判断消息的地方。。我们下断点 bpx seedmessage   
点击，那个帮助的按钮，然后仔细观察程序的走向，你会发现消息判别在这个地方：   
:004391AB B8D8A24400              mov eax, 0044A2D8   
:004391B0 E853EDFEFF              call 00427F08   
:004391B5 83EC54                  sub esp, 00000054   
:004391B8 8365F000                and dword ptr \[ebp\-10\], 00000000   
:004391BC 53                      push ebx   
:004391BD 8B5D08                  mov ebx, dword ptr \[ebp+08\]   
:004391C0 56                      push esi   
:004391C1 57                      push edi   
:004391C2 81FB11010000            cmp ebx, 00000111＝＝＝》比较消息是否是 111（wm\_command)   
:004391C8 8BF9                    mov edi, ecx   
:004391CA 7518                    jne 004391E4   
:004391CC FF7510                  push \[ebp+10\]   
:004391CF 8B07                    mov eax, dword ptr \[edi\]   
:004391D1 FF750C                  push \[ebp+0C\]======>这里传递按钮的 id（也就是 wParam 参数）点击帮助按钮，这里我们可以看到 id 为 3f6（1014）正好是帮助的 control id   
:004391D4 FF5078                  call \[eax+78\]   
:004391D7 85C0                    test eax, eax   
:004391D9 0F8455010000            je 00439334   
:004391DF E91D040000              jmp 00439601   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004391CA(C)   
|   
:004391E4 83FB4E                  cmp ebx, 0000004E   
  
  
跟踪进 call \[eax+78\]到下面（现在你就要牢牢的抓住这个 3f6 看看程序是如何判断和传递 3f6 的 id 的）我这里 call 【eax＋78】是到达   
:00439747 55                      push ebp   
:00439748 8BEC                    mov ebp, esp   
:0043974A 83EC2C                  sub esp, 0000002C   
:0043974D 8B4508                  mov eax, dword ptr \[ebp+08\]   
:00439750 53                      push ebx   
:00439751 56                      push esi   
:00439752 57                      push edi   
:00439753 0FB7F8                  movzx edi, ax   
:00439756 33DB                    xor ebx, ebx   
:00439758 8BF1                    mov esi, ecx   
:0043975A C1E810                  shr eax, 10   
:0043975D 395D0C                  cmp dword ptr \[ebp+0C\], ebx   
:00439760 894508                  mov dword ptr \[ebp+08\], eax   
:00439763 753A                    jne 0043979F   
:00439765 3BFB                    cmp edi, ebx   
:00439767 7466                    je 004397CF   
:00439769 8D4DD4                  lea ecx, dword ptr \[ebp\-2C\]   
:0043976C E8ACFFFFFF              call 0043971D   
:00439771 8B06                    mov eax, dword ptr \[esi\]   
:00439773 8D4DD4                  lea ecx, dword ptr \[ebp\-2C\]   
:00439776 53                      push ebx   
:00439777 51                      push ecx   
:00439778 6AFF                    push FFFFFFFF   
:0043977A 57                      push edi   
:0043977B 8BCE                    mov ecx, esi   
:0043977D 897DD8                  mov dword ptr \[ebp\-28\], edi   
:00439780 FF500C                  call \[eax+0C\]   
:00439783 395DFC                  cmp dword ptr \[ebp\-04\], ebx   
:00439786 743E                    je 004397C6   
:00439788 895D08                  mov dword ptr \[ebp+08\], ebx   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004397CD(C)   
|   
:0043978B 8B06                    mov eax, dword ptr \[esi\]   
:0043978D 53                      push ebx   
:0043978E 53                      push ebx   
:0043978F 8BCE                    mov ecx, esi   
:00439791 FF7508                  push \[ebp+08\]   
:00439794 57                      push edi   
:00439795 FF500C                  call \[eax+0C\]＝＝》这里就会 call 那个帮助的 ie 哦好我们再这个 call 里面去一趟   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:004397C9(U), :004397D1(U)   
|   
:00439798 5F                      pop edi   
:00439799 5E                      pop esi   
:0043979A 5B                      pop ebx   
:0043979B C9                      leave   
:0043979C C20800                  ret 0008   
  
call \[eax+0C\]会到达下面   
:0043E4D6 B85CA34400              mov eax, 0044A35C   
:0043E4DB E8289AFEFF              call 00427F08   
:0043E4E0 51                      push ecx   
:0043E4E1 51                      push ecx   
:0043E4E2 57                      push edi   
:0043E4E3 8BF9                    mov edi, ecx   
:0043E4E5 FF7514                  push \[ebp+14\]   
:0043E4E8 FF7510                  push \[ebp+10\]   
:0043E4EB FF750C                  push \[ebp+0C\]   
:0043E4EE FF7508                  push \[ebp+08\]==>这里又看到老朋友 3f6 了   
:0043E4F1 E8D0CAFFFF              call 0043AFC6==》看来进这个 call 吧   
:0043E4F6 85C0                    test eax, eax   
:0043E4F8 7405                    je 0043E4FF   
  
call 0043AFC6 里面继续跟踪，会发现比较消息的 id 是在下面这段   
:0043B09B FF7508                  push \[ebp+08\]＝＝》这里能看到老朋友 3f6   
:0043B09E FF750C                  push \[ebp+0C\]   
:0043B0A1 53                      push ebx   
:0043B0A2 FF7604                  push \[esi+04\]   
:0043B0A5 E87DE0FFFF              call 00439127   
:0043B0AA 85C0                    test eax, eax   
:0043B0AC 7504                    jne 0043B0B2   
  
call 00439127 里面如下   
:00439127 55                      push ebp   
:00439128 8BEC                    mov ebp, esp   
:0043912A 53                      push ebx   
:0043912B 8B5D08                  mov ebx, dword ptr \[ebp+08\]   
:0043912E 8B450C                  mov eax, dword ptr \[ebp+0C\]   
:00439131 8B5510                  mov edx, dword ptr \[ebp+10\]   
:00439134 8B4D14                  mov ecx, dword ptr \[ebp+14\]   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00439144(U)   
|   
:00439137 837B1000                cmp dword ptr \[ebx+10\], 00000000   
:0043913B 741D                    je 0043915A   
:0043913D 3B03                    cmp eax, dword ptr \[ebx\]   
:0043913F 7405                    je 00439146   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00439149(C), :0043914E(C), :00439153(C)   
|   
:00439141 83C318                  add ebx, 00000018   
:00439144 EBF1                    jmp 00439137   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0043913F(C)   
|   
:00439146 3B5304                  cmp edx, dword ptr \[ebx+04\]   
:00439149 75F6                    jne 00439141   
:0043914B 3B4B08                  cmp ecx, dword ptr \[ebx+08\]＝》cx 是老朋友 3f6   
这里就是比较 id 的地方了，你能看到 ebx+08 是 3e9(退出）等按钮的比较   
:0043914E 72F1                    jb 00439141   
:00439150 3B4B0C                  cmp ecx, dword ptr \[ebx+0C\]   
:00439153 77EC                    ja 00439141   
:00439155 895D08                  mov dword ptr \[ebp+08\], ebx   
:00439158 EB05                    jmp 0043915F   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0043913B(C)   
|   
:0043915A 33C0                    xor eax, eax   
:0043915C 894508                  mov dword ptr \[ebp+08\], eax   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00439158(U)   
|   
:0043915F 8B4508                  mov eax, dword ptr \[ebp+08\]   
:00439162 5B                      pop ebx   
:00439163 5D                      pop ebp   
:00439164 C21000                  ret 0010   
  
好了知道比较的地方就可以更改了，首先找一段空白的地址，放自己的代码，然后修改上面的比较代码 jmp 到自己代码地址，加入我们自己的 id 比较 3f3   
  
cmp ecx,3f3   
jz 自己想干的事情的地方，你可以参照我上篇文章，做个在 riched 输出文本的代码段   
下面就是恢复原程序的动作   
然后跳回原程序   
  
需要注意的事项：不要把堆栈搞乱了，否则你看到的就是非法操作了。而不是你想看到的东西：）   
这样你可以加好多个按钮，每个按钮做不同的事情。把程序玩弄于鼓掌之间，达到了 diy pe 的目的   
如果感觉我的 diy pe 还可以的话，我就再写篇 diy pe 之三，写教学很累，比看程序，和写程序都累，劳动需要得到肯定,谢谢大家支持！   
  
                        你的朋友：pll621  4dd466d0e1e0cc123afbe504da2462ca
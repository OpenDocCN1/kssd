
# 菲菲 0502 版

> 原文：`www.pediy.com/kssd/pediy07/pediy7-223.htm`

突破两个实例限制  
-----------一次不完美的 PEDIY

【破解作者】 `winndy` `[FCG]` `[PYG]`

【作者邮箱】 `CNwinndy@hotmail.com`

【使用工具】 `PEID v0.93` `OllyDbg v1.10` `fly 修改版`

【破解平台】 `Winxp SP2`

【软件名称】 `天骄 2 外挂--菲菲 0502 版`

【官方网址】 `http://bbs.tj2wg.com/`

【编写语言】 `VC6`

【软件介绍】 不用多说了...

【破解声明】 `For study ,For Fun,`

【破解说明】 只能运行两个实例，无壳，`0510` 版的是 `Armadillo 3.78` -> `Silicon Realms Toolworks` 加的壳，不会脱，  
所以找这个 `0502` 来揉一揉...  
玩到最后，可以运行 3 个实例，但是第 3 个实例运行，`hook` 住天骄后，不能启动热键 `F12`,...  
最后实在找不出原因，不过想想.这次不完美的 `pediy`，也费了不少周折，自己写个总结吧...  
失误之处还请各位老大指出!

【破解过程】

`OD` 载入菲菲，反键，“搜索”--“所有参考文本串”，  
在下面设断点，然后观察和比较，实例一和实例二的代码的区别，以及寄存器的状态等值.

```
004047DD    .  8B83 580F0000 mov eax,dword ptr ds:[ebx+F58]  
004047E3    .  8B4B 20       mov ecx,dword ptr ds:[ebx+20]  
004047E6    .  33ED          xor ebp,ebp  
004047E8    .  50            push eax                             ; /lParam  
004047E9    .  55            push ebp                             ; |wParam => 0  
004047EA    .  68 80000000   push 80                              ; |Message = WM_SETICON  
004047EF    .  51            push ecx                             ; |hWnd  
004047F0    .  FFD6          call esi                             ; \SendMessageA  
004047F2    .  8B35 24604100 mov esi,dword ptr ds:[<&KERNEL32.Op>;  kernel32.OpenMutexA  
004047F8    .  68 ACA24100   push TJMan9.0041A2AC                ; /MutexName = "TJMANRunOnlyOneInstance1"  
004047FD    .  55            push ebp                             ; |Inheritable => FALSE  
004047FE    .  68 01001F00   push 1F0001                          ; |Access = 1F0001  
00404803    .  89AB 54020000 mov dword ptr ds:[ebx+254],ebp       ; |  
00404809    .  FFD6          call esi                             ; \OpenMutexA  
0040480B    .  51            push ecx  
0040480C    .  3BC5          cmp eax,ebp  
0040480E    .  8983 00030000 mov dword ptr ds:[ebx+300],eax  
00404814    .  8BCC          mov ecx,esp  
00404816       75 7E         jnz short TJMan9.00404896           ;实例 1 已运行，去运行实例 2  
00404818    .  896424 14           mov dword ptr ss:[esp+14],esp  
//d esp  
//00125C30  71 FB 92 7C             q 麙|.  
0040481C    .  68 A0A24100         push TJMan.0041A2A0                     ;  ASCII "外挂 1 开!"  
00404821    .  E8 5CF20000         call <jmp.&MFC42.#537>  
00404826    .  8BCB                mov ecx,ebx                              ; |  
00404828    .  E8 430B0000         call TJMan.00405370                   ; \TJMan.00405370  
0040482D    .  BF 8CA24100         mov edi,TJMan.0041A28C                    ;  ASCII "TJ2HANDLE1byParker"  
00404832    .  83C9 FF             or ecx,FFFFFFFF  
00404835    .  33C0                xor eax,eax  
00404837    .  8D5424 24           lea edx,dword ptr ss:[esp+24]  
0040483B    .  F2:AE               repne scas byte ptr es:[edi]  
0040483D    .  F7D1                not ecx  
0040483F    .  2BF9                sub edi,ecx  
00404841    .  68 ACA24100         push TJMan.0041A2AC                      ; /MutexName = "TJMANRunOnlyOneInstance1"  
00404846    .  8BC1                mov eax,ecx                              ; |  
00404848    .  8BF7                mov esi,edi                              ; |  
0040484A    .  8BFA                mov edi,edx                              ; |  
0040484C    .  55                  push ebp                             ; |InitialOwner  
0040484D    .  C1E9 02             shr ecx,2                              ; |  
00404850    .  F3:A5               rep movs dword ptr es:[edi],dword ptr ds:[esi]  ; |  
00404852    .  8BC8                mov ecx,eax                              ; |  
00404854    .  55                  push ebp                             ; |pSecurity  
00404855    .  83E1 03             and ecx,3                              ; |  
00404858    .  F3:A4               rep movs byte ptr es:[edi],byte ptr ds:[esi]    ; |  
0040485A    .  FF15 20604100       call dword ptr ds:[<&KERNEL32.CreateMutexA>]    ; \CreateMutexA  
00404860    .  3BC5                cmp eax,ebp  
00404862    .  8983 00030000        mov dword ptr ds:[ebx+300],eax  
00404868    .  74 11                je short TJMan.0040487B  
0040486A    .  FF15 1C604100        call dword ptr ds:[<&KERNEL32.GetLastError>]    ; [GetLastError  
00404870    .  3D B7000000          cmp eax,0B7  
00404875    .  0F85 C9000000        jnz TJMan.00404944  
0040487B    >  55                   push ebp  
0040487C    .  55                   push ebp  
0040487D    .  68 70A24100          push TJMan.0041A270                      ;  ASCII "系统错误 1，外挂不能运行!"  
00404882    .  E8 3FF00000          call <jmp.&MFC42.#1200>  
00404887    .  8B13                mov edx,dword ptr ds:[ebx]  
00404889    .  8BCB                mov ecx,ebx  
0040488B    .  FF92 CC000000        call dword ptr ds:[edx+CC]  
00404891    .  E9 AE000000          jmp TJMan.00404944  
//运行实例 2  
00404896    >  896424 14            mov dword ptr ss:[esp+14],esp  
//d esp  
//00125C30  C0 5B 12 00             繹..  
//这个 esp 好像很重要，在手动增加第 3 个实例的代码之后，esp 为 00125C34，运行出错了，  
//最后比较寄存器的状态，再往堆栈里压入一个参数，然后 esp 变为 00125C30，就可以正常启动第 3 个实例了.  
0040489A    .  68 68A24100          push TJMan.0041A268                      ;  ASCII "外挂 2 开"  
0040489F    .  E8 DEF10000          call <jmp.&MFC42.#537>  
004048A4    .  8BCB                mov ecx,ebx                              ; |  
004048A6    .  E8 C50A0000          call TJMan.00405370                   ; \TJMan.00405370  
//通过后来的第三个实例可以知道，上面四行代码是类似用来初始化实例(ini instance)的代码，  
//////////////////  
//这段代码在实例一种是放在上面五行代码前面，现在把它放后面了，注意比较区别  
//也就是说，实例 2 是先初始化，然后调用 OpenMutexA 来看实例是否已运行，  
//而实例一是先调用 OpenMutexA 来看实例是否已运行，然后再来初始化，  
//显然，应该先判断实例是否存在，然后再来初始化.  
//在手动构造了实例 3 的代码之后，会发现实例 2 的这种顺序所带来的影响，在后来，我又手动调换了这两段小代码的顺序.  
004048AB    .  68 4CA24100          push TJMan.0041A24C                      ;  ASCII "TJMANRunOnlyOneInstance2"  
004048B0    .  55                  push ebp  
004048B1    .  68 01001F00          push 1F0001  

{  对照比较两个实例的代码:

实例 1 中这里有一句  

00404803    .  89AB 54020000 mov dword ptr ds:[ebx+254],ebp       ; |  

}  

004048B6    .  FFD6                call esi                             ; ESI 7C80EC1B kernel32.OpenMutexA  

{实例 1 中这里有一句  

0040480B    .  51            push ecx     

ECX 7C92FB71 ntdll.7C92FB71  

在后来建立实例 3 后，堆栈为 00125C34，运行错误，为了修正堆栈，压入了此 7C92FB71.  

不过当调换实例 2 的初始化和 openmutex 后，由于同样的堆栈原因，把 push 7C92FB71 NOP 了，  

运行正常(这个参数好像没用)，只要保证 esp 和 ecx 为 00125C30 就行了.  

}  

004048B8    .  3BC5                cmp eax,ebp  
004048BA    .  8983 00030000        mov dword ptr ds:[ebx+300],eax  

{  

实例 1 中这里有一句  

00404814    .  8BCC          mov ecx,esp  

ecx=00125C30  

d ecx  

00125C30  71 FB 92 7C             q 麙|.  

注意，这正是 0040480B 处 push 的 ecx.  

}  

004048C0    .  75 6C             jnz short TJMan.0040492E                  
  
////第二个实例已运行，显示"只支持双开".  

//所需要作的就是在适当的位置增加一段创建实例 3 的代码，然后从这里跳去.基本思想就是这样.  

//创建实例 3 的代码，是从实例 2 的代码拷过去的，从 00404896 到 0040492C.  

///////////////////  

004048C2    .  BF 38A24100          mov edi,TJMan.0041A238                     ;  ASCII "TJ2HANDLE2byParker"  
004048C7    .  83C9 FF              or ecx,FFFFFFFF  
004048CA    .  33C0                 xor eax,eax  
004048CC    .  8D5424 24            lea edx,dword ptr ss:[esp+24]     ;edx=esp+24=00125C34+24=00125C58  
004048D0    .  F2:AE                repne scas byte ptr es:[edi]  
```



# Amem

> 原文：[`www.pediy.com/kssd/pediy03/forum413.htm`](https://www.pediy.com/kssd/pediy03/forum413.htm)

 http://go1.163.com:80/~sl0925/amem4b.exe  
  
一个记录并重放鼠标操作的软件: 机器人 .  
但启动时有一个窗口.必须按一下(继续试用)按钮,挺烦的.   
我尝试将它破解.但发现程序始终在转圈.难度挺大,请求高手指点.   
能不能写篇教程出来.  

  

 CALL 0040BE10 等价于函数 IsEvaluationVersion( )，返回 TRUE 则为试用版本。该函数如下：   
  
\* Referenced by a CALL at Addresses:   
|:0040BC5A  , :0040D183  , :00415486  , :004156A9  , :00415839    
|:00415BDB  , :004196A8  , :004202D7    
|   
:0040BE10 B801000000              mov eax, 00000001   
:0040BE15 C3                      ret   
  
这个函数共被 8 个地方调用，在调用该函数的各个地方设一下断点，跟一下就清楚该函数的作用了。   
  
第一处：启动时是否判断日期并显示 Nag   
  
:0040BC5A E8B1010000              call 0040BE10  //这里   
:0040BC5F 85C0                    test eax, eax   
:0040BC61 0F84C4000000            je 0040BD2B   
:0040BC67 6A00                    push 00000000   
:0040BC69 8D4C2418                lea ecx, dword ptr \[esp+18\]   
:0040BC6D E8DEF5FFFF              call 0040B250   
:0040BC72 8D4C2414                lea ecx, dword ptr \[esp+14\]   
:0040BC76 C78424F400000002000000  mov dword ptr \[esp+000000F4\], 00000002   
:0040BC81 E8D3770200              call 00433459  //显示 Nag   
:0040BC86 83F802                  cmp eax, 00000002   
:0040BC89 7521                    jne 0040BCAC   
  
第二处：是否显示“购买”菜单   
  
:0040D183 E888ECFFFF              call 0040BE10   
:0040D188 85C0                    test eax, eax   
:0040D18A 7521                    jne 0040D1AD   
:0040D18C 8B561C                  mov edx, dword ptr \[esi+1C\]   
:0040D18F 52                      push edx   
  
\* Reference To: USER32.GetMenu, Ord:011Ch   
                                  |   
:0040D190 FF15A0444400            Call dword ptr \[004444A0\]   
:0040D196 50                      push eax   
:0040D197 E8CDAA0200              call 00437C69   
:0040D19C 8B4004                  mov eax, dword ptr \[eax+04\]   
:0040D19F 6A00                    push 00000000   
  
\* Possible Ref to Menu: MenuID\_00A0, Item: "Ordering/Registering EZ Macros"   
                                  |   
:0040D1A1 6806800000              push 00008006   
:0040D1A6 50                      push eax   
  
\* Reference To: USER32.DeleteMenu, Ord:0087h   
                                  |   
:0040D1A7 FF1598444400            Call dword ptr \[00444498\]   
  
第三处：是否显示 Nag   
  
:00415486 E88569FFFF              call 0040BE10   
:0041548B 85C0                    test eax, eax   
:0041548D 0F843C010000            je 004155CF   
:00415532 E829810000              call 0041D660   
......................................   
\* Possible StringData Ref from Data Obj ->"EZ Macros"   
                                  |   
:00415537 6830424500              push 00454230   
:0041553C 8D8C24E8000000          lea ecx, dword ptr \[esp+000000E8\]   
:00415543 C684240802000034        mov byte ptr \[esp+00000208\], 34   
:0041554B C78424EC00000001000000  mov dword ptr \[esp+000000EC\], 00000001   
:00415556 E8291A0200              call 00436F84   
  
\* Possible StringData Ref from Data Obj ->"Thank you for trying EZ Macros. "   
                                        ->" This window will not appear in "   
                                        ->"the full verion of EZ Macros and "   
                                        ->"your macro can run uninterrupted "   
                                        ->"for as long as you like.  Please "   
                                        ->"consider ordering EZ Macros.  "   
                                        ->"You can buy EZ Macros and upgrade "   
                                        ->"right now by going to http://www.ezmacros.com."   
                                  |   
:0041555B 68B85E4500              push 00455EB8   
  
其余省略。   
  
可见，只要把 mov eax, 00000001 改为 mov eax, 0 就行了。try!   4dd466d0e1e0cc123afbe504da2462ca
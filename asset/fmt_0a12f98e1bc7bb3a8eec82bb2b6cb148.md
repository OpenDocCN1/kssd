
```
00476903    A1 68D94700     mov eax,dword ptr ds:\[47D968\]  
00476908    8B00           mov eax,dword ptr ds:\[eax\]  
0047690A    E8 CD25FEFF    call chelper.00458EDC  
0047690F    B8 405A4700    mov eax,chelper.00475A40  
00476914    E8 17F6FFFF    call chelper.00475F30  
00476919    84C0           test al,al  
0047691B    74 0C          je short chelper.00476929  
0047691D    A1 68D94700    mov eax,dword ptr ds:\[47D968\]  
00476922    8B00           mov eax,dword ptr ds:\[eax\]  
00476924    E8 B325FEFF    call chelper.00458EDC  
00476929    B8 C05B4700    mov eax,chelper.00475BC0  
0047692E    E8 FDF5FFFF    call chelper.00475F30  
00476933    84C0           test al,al  
00476935    74 0C          je short chelper.00476943  
00476937    A1 68D94700    mov eax,dword ptr ds:\[47D968\]  
0047693C    8B00           mov eax,dword ptr ds:\[eax\]  
0047693E    E8 9925FEFF    call chelper.00458EDC        ; 调用算法 call(2)-根据硬盘号取得机器码  
00476943    803D A8EC4700 0>cmp byte ptr ds:\[47ECA8\],0  
0047694A    0F85 BF000000  jnz chelper.00476A0F  
00476950    8D4D F4         lea ecx,dword ptr ss:\[ebp-C\]  
00476953    8BD7           mov edx,edi                  ; edi=注册名  
00476955    A1 A4EC4700    mov eax,dword ptr ds:\[47ECA4\]  
0047695A    E8 69FCFFFF    call chelper.004765C8        ; 调用算法 call(4)-注册算法  
0047695F    8B45 F4        mov eax,dword ptr ss:\[ebp-C\]  ; EDX 中存的就是注册真码  
00476962    8BD6           mov edx,esi                  ; !!!!!关键所在，请看后文分析!!!!!!!  
00476964    E8 43DEF8FF    call chelper.004047AC        ; 由算法 call(2)得出真正注册码  
00476969    0F85 A0000000  jnz chelper.00476A0F  
0047696F    B2 01          mov dl,1  
00476971    A1 D0D74500    mov eax,dword ptr ds:\[45D7D0\]  
00476976    E8 556FFEFF    call chelper.0045D8D0  
0047697B    8BD8           mov ebx,eax  
0047697D    BA 00000080    mov edx,80000000  
..........  
017E7419    C3             retn                          ; 算法 Call(1)结束  

=========================================================================
```

### 破解总结

晕倒，易通果然牛，所有软件全部使用一模一样的注册方法，显然不必细说了(前提是你看了我之前的那两篇文章)。

用 `W32Dasm` 打开脱壳后的 `chelper.exe` 定位到 `00476962` 将 `8BD6 mov edx,esi` 改为 `8BF2 mov esi,edx`，然后覆盖原文件，运行软件随意输入用户名与注册码，即可成功注册。

再回来验证一下，将原文件覆盖回去，运行软件发现仍然是注册版。只要保留修改后的 `chelper.exe` 文件就可以不用 `OD` 跟码，一劳永逸的让程序自己注册了。

真的不想再说什么了，这篇破文还是抄袭自己的，目的依然是为菜鸟们提供简单的破解方法，技术含量超级低，希望 `cracker` 们留我口气，下回一定发表新的破解!!

另附：本文注册码存放于注册表的(依机器码不同而不同)：

`HKEY_LOCAL_MACHINE\SOFTWARE\Classes\SVS6YPYHR1QCA\tgdy`: `"\[DPVSF.'^ %'\]^PNDT"`

`HKEY_LOCAL_MACHINE\SOFTWARE\Classes\SVS6YPYHR1QCA\bdre`: `"耐辽"`

`4dd466d0e1e0cc123afbe504da2462ca`

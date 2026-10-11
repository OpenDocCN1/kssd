
```
007901CF   |.  8BD3             mov edx,ebx                     ;  ecx=edx=dbx=1  

007901D1   |.  8B45 F8         mov eax,dword ptr ss:\[ebp-8\]    ;  eax='Yq...' string  

007901D4   |.  E8 9F50C7FF     call dumped(?00405278            ;  got the first ECX chars from eax string. saved into \[ss:ebp-10\]  

007901D9   |.  8D45 F8         lea eax,dword ptr ss:\[ebp-8\]    ;  EAX=0012FD88  

007901DC   |.  8B55 F8         mov edx,dword ptr ss:\[ebp-8\]    ;  EDX='Yq...' string  

007901DF   |.  E8 444EC7FF     call dumped(?00405028  

007901E4   |.  8D45 F8         lea eax,dword ptr ss:\[ebp-8\]    ;  eax=0012FD88  

007901E7   |.  50             push eax  

007901E8   |.  B9 41000000     mov ecx,41                      ;  ecx=41（the length of 'Yq...' string)  

007901ED   |.  8BD3            mov edx,ebx                       ;  ebx=dex=1  

007901EF   |.  8B45 F8         mov eax,dword ptr ss:\[ebp-8\]    ;  eax='Y...' string  

007901F2   |.  E8 8150C7FF     call dumped(?00405278            ;  F7 to enter this one, check 008335C8  

007901F7   |.  8D45 EC         lea eax,dword ptr ss:\[ebp-14\]  

007901FA   |.  50             push eax  

007901FB   |.  8B4D F4         mov ecx,dword ptr ss:\[ebp-C\]    ;  string '\_$...'  

007901FE   |.  8B55 F8         mov edx,dword ptr ss:\[ebp-8\]    ;  string 'Yq...'  

00790201   |.  8B45 FC         mov eax,dword ptr ss:\[ebp-4\]    ;  eax=first 2 chars in input password  

00790204   |.  E8 B3FEFFFF     call dumped(?007900BC  

00790209   |.  8B55 EC         mov edx,dword ptr ss:\[ebp-14\]   ;  edx = 'j0', this was encrypted chars  

0079020C   |.  8D45 F0         lea eax,dword ptr ss:\[ebp-10\]  

0079020F   |.  E8 144EC7FF     call dumped(?00405028  

00790214   |.  8BC6            mov eax,esi  

00790216   |.  8B55 F0         mov edx,dword ptr ss:\[ebp-10\]  

00790219   |.  E8 8E4BC7FF     call dumped(?00404DAC  

0079021E   |.  33C0            xor eax,eax  

00790220   |.  5A             pop edx  

00790221   |.  59             pop ecx  

00790222   |.  59             pop ecx  

00790223   |.  64:8910         mov dword ptr fs:\[eax\],edx  

00790226   |.  68 40027900     push dumped(?00790240  

0079022B   |>  8D45 EC         lea eax,dword ptr ss:\[ebp-14\]  

0079022E   |.  BA 05000000     mov edx,5  

00790233   |.  E8 444BC7FF     call dumped(?00404D7C  

00790238   |\.  C3             retn                              ;  返回到上面  

\============================================================================================================  
```

上面我们完成了对注册码的所有分析，本来想写注册机的，可是我的汇编已经有 8 年没有用过了。我会努力的!! 感兴趣的大侠可以根据我的算法分析自己写吧 ：（

另外提醒，如果是爆破，有两个地方要改的。不要只改一个地方 ：）

`4dd466d0e1e0cc123afbe504da2462ca`

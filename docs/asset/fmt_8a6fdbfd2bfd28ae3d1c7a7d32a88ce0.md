
```asm
004939B4   .  50             push    eax                          ; 结果，track 后发现是连接上面的那三个结果。  
004939B5   .  8D85 FCFDFFFF lea     eax, dword ptr [ebp-204]  
004939BB   .  50             push    eax                          ; 连接后就是真注册码了。  
004939BC   .  FFD3           call    ebx                          ; <&MSVBVM60.__vbaVarCat>  
004939BE   .  50             push    eax  
004939BF   .  8D8D 4CFDFFFF lea     ecx, dword ptr [ebp-2B4]  
004939C5   .  51             push    ecx  
004939C6   .  8D95 3CFDFFFF lea     edx, dword ptr [ebp-2C4]  
004939CC   .  52             push    edx  
004939CD   .  FFD7           call    edi  
004939CF   .  50             push    eax  
004939D0   .  8D85 2CFDFFFF lea     eax, dword ptr [ebp-2D4]  
004939D6   .  50             push    eax  
004939D7   .  FFD3           call    ebx  
004939D9   .  8B3D 18104000 mov     edi, dword ptr [&<MSVBVM60.__vba>;  MSVBVM60.__vbaVarMove  

...........省略一点代码.....  

00493B0C   .  6A 22           push    22  
00493B0E   .  FF15 38104000 call    dword ptr [&<MSVBVM60.__vbaFreeV>; MSVBVM60.__vbaFreeVarList  
00493B14   .  8B55 0C         mov     edx, dword ptr [ebp+C]  
00493B17   .  8B02           mov     eax, dword ptr [edx]                 ; 假码出现了  
00493B19   .  81C4 C0000000 add     esp, 0C0  
00493B1F   .  8D8D 1CFDFFFF lea     ecx, dword ptr [ebp-2E4]  
00493B25   .  51             push    ecx                          ; /Arg2  
00493B26   .  8D55 DC         lea     edx, dword ptr [ebp-24]                ; |  
00493B29   .  52             push    edx                          ; |Arg1  
00493B2A   .  8985 24FDFFFF mov     dword ptr [ebp-2DC], eax             ; |  
00493B30   .  C785 1CFDFFFF>mov     dword ptr [ebp-2E4], 8008            ; |和上面计算的结果比较。变量比较  
00493B3A   .  FF15 18114000 call    dword ptr [&<MSVBVM60.__vbaVarTs>; \__vbaVarTstEq  
00493B40   .  66:85C0        test    ax, ax                        ;  标志位，不为 0 就注册成功  
00493B43   .  C785 24FDFFFF>mov     dword ptr [ebp-2DC], -1  
00493B4D     75 06         jnz     short 00493B55                    爆破点这里。  
00493B4F   .  89B5 24FDFFFF mov     dword ptr [ebp-2DC], esi  
00493B55   >  8D95 1CFDFFFF lea     edx, dword ptr [ebp-2E4]  
00493B5B   .  8D4D CC         lea     ecx, dword ptr [ebp-34]  
00493B5E   .  C785 1CFDFFFF>mov     dword ptr [ebp-2E4], 0B  
00493B68   .  FFD7           call    edi  
00493B6A   .  9B             wait  
```

## 破解总结

比如我的机器码为：`577020080877`

1、整个算法就是对机器码的运算，而得到真注册码。

2、首先分别取出机器码的第 8 位、 10 位、 11 位 、 1 位 、7 位 、12 位、 1 位 、4 位 、10 位得到（有些取两位，具体看注释）。得到下面

第一组 `8` `87` `7`

第二组 `5` `08` `7`

第三组 `5` `02` `8`

3、对应第一组取第五位和他们运算。得到 `1385`

对应第二组取第三位和他们运算。得到 `273`

对应第三组取第一位和他们运算。得到 `42`

4、连接结果得到 `138527342` 就是最终注册码了。

`4dd466d0e1e0cc123afbe504da2462ca`

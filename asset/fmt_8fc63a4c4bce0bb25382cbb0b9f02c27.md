
# 14. 第十四处被抽取的代码

```
0041AA08     90                   nop   
0041AA09     90                   nop   
0041AA0A     90                   nop   
0041AA0B     90                   nop   
0041AA0C     90                   nop   
0041AA0D     90                   nop   
0041AA0E     90                   nop   
0041AA0F     90                   nop               
0041AA10     8BF7                 mov esi,edi   
0041AA12     AC                   lods byte ptr ds:[esi]   
0041AA13     D2C0                 rol al,cl   
0041AA15     32C1                 xor al,cl   
0041AA17     51                   push ecx   
0041AA18     8ACA                 mov cl,dl   
0041AA1A     D2C8                 ror al,cl   
0041AA1C     59                   pop ecx   
0041AA1D     D2C8                 ror al,cl   
0041AA1F     32C1                 xor al,cl   
0041AA21     F6D0                 not al   
0041AA23     AA                   stos byte ptr es:[edi]   
0041AA24     E2 EC                loopd short 0041AA12   
```

————————————————————————

14、第十四处被抽取的代码

```
0041AA26     E8 00000000          call 0041AA2B   
0041AA2B     FF25 1B5AD307       jmp dword ptr ds:[7D35A1B]   
```

这段代码还原成：

```
0041AA26     90                   nop   
0041AA27     90                   nop   
0041AA28     90                   nop   
0041AA29     90                   nop   
0041AA2A     90                   nop   
0041AA2B     90                   nop   
0041AA2C     90                   nop   
0041AA2D     90                   nop   
0041AA2E     90                   nop   
0041AA2F     90                   nop   
0041AA30     90                   nop   
0041AA31     90                   nop   
0041AA32     90                   nop   
0041AA33     90                   nop   
0041AA34     90                   nop   
0041AA35     90                   nop   
0041AA36     C3                   retn   
```

————————————————————————

# 15. 第十五处被抽取的代码

```
0041A931     FF25 D13AD307       jmp dword ptr ds:[7D33AD1]   
                       ====>入壳还原！

07D3578B     F3:A4              rep movs byte ptr es:[edi], byte ptr ds:[esi]   
                       ====>这里对 0041A941 至 0041A985 之间的代码进行还原！   
```

好了，代码还原后 部分脱壳：`0041AB06`   大小：`0041A985-0041A941=44`  存为：`0041A941.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `0041AA00` 后面全部 `NOP`

```
0041A931     90                   nop   
0041A932     90                   nop   
0041A933     90                   nop   
0041A934     90                   nop   
0041A935     90                   nop   
0041A936     90                   nop   
0041A937     90                   nop   
0041A938     90                   nop   
0041A939     90                   nop   
0041A93A     90                   nop   
0041A93B     90                   nop   
0041A93C     90                   nop   
0041A93D     90                   nop   
0041A93E     90                   nop   
0041A93F     90                   nop   
0041A940     90                   nop   
0041A941     A1 BD384200         mov eax, dword ptr ds:[4238BD]   
0041A946     05 48BC0400         add eax,4BC48   
0041A94B     8987 A0000000       mov dword ptr ds:[edi+A0],eax   
0041A951     C787 A4000000 0A0000>mov dword ptr ds:[edi+A4],0A   
0041A95B     803D 7C114B00 01    cmp byte ptr ds:[4B117C],1   
0041A962     75 21              jnz short 0041A985   
0041A964     A1 40A74400         mov eax,dword ptr ds:[44A740]   
0041A969     A3 48F24600         mov dword ptr ds:[46F248],eax   
0041A96E     C705 4CF24600 0A0000>mov dword ptr ds:[46F24C],0A   
0041A978     B8 0F000000         mov eax,0F   
0041A97D     B4 30              mov ah,30   
0041A97F     66:A3 50F24600      mov word ptr ds:[46F250],ax 
```

# 16. 第十六处被抽取的代码

```
0041A985     E8 00000000          call 0041A98A   
0041A98A     FF25 1B5AD307       jmp dword ptr ds:[7D35A1B]   
```

这段代码还原成：

```
0041A985     90                   nop   
0041A986     90                   nop   
0041A987     90                   nop   
0041A988     90                   nop   
0041A989     90                   nop   
0041A98A     90                   nop   
0041A98B     90                   nop   
0041A98C     90                   nop   
0041A98D     90                   nop   
0041A98E     90                   nop   
0041A98F     90                   nop   
0041A990     90                   nop   
0041A991     90                   nop   
0041A992     90                   nop   
0041A993     90                   nop   
0041A994     90                   nop   
0041A995     C3                   retn   
```

————————————————————————

# 17. 第十七处被抽取的代码

```
0041A8F1     FF25 D13AD307       jmp dword ptr ds:[7D33AD1]   
                       ====>入壳还原！

07D3578B     F3:A4              rep movs byte ptr es:[edi],byte ptr ds:[esi]   
                       ====>这里对 0041A901 至 0041A91B 之间的代码进行还原！   
```

好了，代码还原后 部分脱壳：`0041A901`   大小：`0041A91B-0041A901=1A`  存为：`0041A901.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `0041A8F1` 后面全部 `NOP`

```
0041A8F1     90                   nop   
0041A8F2     90                   nop   
0041A8F3     90                   nop   
0041A8F4     90                   nop   
0041A8F5     90                   nop   
0041A8F6     90                   nop   
0041A8F7     90                   nop   
0041A8F8     90                   nop   
0041A8F9     90                   nop   
0041A8FA     90                   nop   
0041A8FB     90                   nop   
0041A8FC     90                   nop   
0041A8FD     90                   nop   
0041A8FE     90                   nop   
0041A8FF     90                   nop   
0041A900     90                   nop   
0041A901     A1 BD384200         mov eax, dword ptr ds:[4238BD]   
0041A906     05 7C600000         add eax, 607C   
0041A90B     8987 80000000       mov dword ptr ds:[edi+80], eax   
0041A911     C787 84000000 E40000>mov dword ptr ds:[edi+84], 0E4   
0041A91B     E8 00000000          call 0041A920   
```

————————————————————————

# 18. 第十八处被抽取的代码

```
0041A920     FF25 1B5AD307       jmp dword ptr ds:[7D35A1B]   
```

这段代码还原成：

```
0041A920     90                   nop   
0041A921     90                   nop   
0041A922     90                   nop   
0041A923     90                   nop   
0041A924     90                   nop   
0041A925     90                   nop   
0041A926     90                   nop   
0041A927     90                   nop   
0041A928     90                   nop   
0041A929     90                   nop   
0041A92A     90                   nop   
0041A92B     C3                   retn   
```

————————————————————————

# 19. 第十九处被抽取的代码

```
0041C9EB     FF25 D13AD307       jmp dword ptr ds:[7D33AD1]   
                       ====>入壳还原！

07D3578B     F3:A4              rep movs byte ptr es:[edi],byte ptr ds:[esi]   
                       ====>这里对 0041C9FB 至 0041CA4B 之间的代码进行还原！   
```

好了，代码还原后 部分脱壳：`0041C9FB`   大小：`0041CA4B-0041C9FB=50`  存为：`0041C9FB.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `0041C9EB` 后面全部 `NOP`

```
0041C9EB     90                   nop   
0041C9EC     90                   nop   
0041C9ED     90                   nop   
0041C9EE     90                   nop   
0041C9EF     90                   nop   
0041C9F0     90                   nop   
0041C9F1     90                   nop   
0041C9F2     90                   nop   
0041C9F3     90                   nop   
0041C9F4     90                   nop   
0041C9F5     90                   nop   
0041C9F6     90                   nop   
0041C9F7     90                   nop   
0041C9F8     90                   nop   
0041C9F9     90                   nop   
0041C9FA     90                   nop   
0041C9FB     E8 60000000          call 0041CA60   
0041CA00     E8 6B010000          call 0041CB70   
```


```asm
003A4A0A    8907                mov dword ptr ds:[edi],eax
003A4A0C    83C6 08             add esi,8
003A4A0F    83C7 04             add edi,4
003A4A12    FF4D 08             dec dword ptr ss:[ebp+8]
003A4A15  ^ 75 A8               jnz short 003A49BF
003A4A17    33C0                xor eax,eax
003A4A19    40                  inc eax
003A4A1A    5F                  pop edi
003A4A1B    5E                  pop esi
003A4A1C    5B                  pop ebx
003A4A1D    C9                  leave
003A4A1E    C2 1400             retn 14
003A4A21    33C0                xor eax,eax
003A4A23    5F                  pop edi
003A4A24    5E                  pop esi
003A4A25    5B                  pop ebx
003A4A26    C9                  leave
003A4A27    C2 1400             retn 14
003A4A2A    C8 000000           enter 0,0
003A4A2E    53                  push ebx
003A4A2F    56                  push esi
003A4A30    57                  push edi
003A4A31    8B5D 14             mov ebx,dword ptr ss:[ebp+14]
003A4A34    8B75 10             mov esi,dword ptr ss:[ebp+10]
003A4A37    8B7D 0C             mov edi,dword ptr ss:[ebp+C]
003A4A3A    8B5B 04             mov ebx,dword ptr ds:[ebx+4]
003A4A3D    66:833E 08          cmp word ptr ds:[esi],8
003A4A41    0F85 97000000       jnz 003A4ADE
003A4A47    8B46 04             mov eax,dword ptr ds:[esi+4]
//根据EAX值选择不同的CALL解码特殊函数
003A4A4A    83F8 00             cmp eax,0
003A4A4D    74 44               je short 003A4A93
003A4A4F    83F8 01             cmp eax,1
003A4A52    74 4E               je short 003A4AA2
003A4A54    83F8 02             cmp eax,2
003A4A57    74 58               je short 003A4AB1
003A4A59    83F8 03             cmp eax,3
003A4A5C    74 11               je short 003A4A6F
003A4A5E    83F8 04             cmp eax,4
003A4A61    75 7B               jnz short 003A4ADE
003A4A63    8B45 18             mov eax,dword ptr ss:[ebp+18]
003A4A66    05 1F9E4100         add eax,419E1F
003A4A6B    8907                mov dword ptr ds:[edi],eax
003A4A6D    EB 6F               jmp short 003A4ADE
003A4A6F    8B45 14             mov eax,dword ptr ss:[ebp+14]
003A4A72    68 C5B1662D         push 2D66B1C5
003A4A77    6A 00               push 0
003A4A79    FF50 18             call dword ptr ds:[eax+18]   这个CALL是处理特殊函数的
003A4A7C    50                  push eax
003A4A7D    53                  push ebx
003A4A7E    E8 EC030000         call 003A4E6F
003A4A83    53                  push ebx
003A4A84    E8 5B030000         call 003A4DE4
003A4A89    8BCB                mov ecx,ebx
003A4A8B    8D5C03 01           lea ebx,dword ptr ds:[ebx+eax+1]
003A4A8F    8BC1                mov eax,ecx
003A4A91    EB 2B               jmp short 003A4ABE
003A4A93    8B45 14             mov eax,dword ptr ss:[ebp+14]
003A4A96    68 0F1ACF4C         push 4CCF1A0F
003A4A9B    6A 00               push 0
003A4A9D    FF50 18             call dword ptr ds:[eax+18]    这个CALL是处理特殊函数的
003A4AA0    EB 1C               jmp short 003A4ABE
003A4AA2    8B45 14             mov eax,dword ptr ss:[ebp+14]
003A4AA5    68 A41A86D0         push D0861AA4
003A4AAA    6A 00               push 0
003A4AAC    FF50 18             call dword ptr ds:[eax+18]   这个CALL是处理特殊函数的
003A4AAF    EB 0D               jmp short 003A4ABE
003A4AB1    8B45 14             mov eax,dword ptr ss:[ebp+14]
003A4AB4    68 E313B41D         push 1DB413E3
003A4AB9    6A 00               push 0
003A4ABB    FF50 18             call dword ptr ds:[eax+18]   这个CALL是处理特殊函数的
003A4ABE    C603 B8             mov byte ptr ds:[ebx],0B8
003A4AC1    8943 01             mov dword ptr ds:[ebx+1],eax
003A4AC4    8B55 18             mov edx,dword ptr ss:[ebp+18]
003A4AC7    81C2 7A9C4100       add edx,419C7A
003A4ACD    8D43 0A             lea eax,dword ptr ds:[ebx+A]
003A4AD0    2BD0                sub edx,eax
003A4AD2    C643 05 E9          mov byte ptr ds:[ebx+5],0E9
003A4AD6    8953 06             mov dword ptr ds:[ebx+6],edx
003A4AD9    891F                mov dword ptr ds:[edi],ebx
003A4ADB    83C3 0A             add ebx,0A
003A4ADE    83C6 08             add esi,8
003A4AE1    83C7 04             add edi,4
003A4AE4    8B45 14             mov eax,dword ptr ss:[ebp+14]
003A4AE7    8958 04             mov dword ptr ds:[eax+4],ebx
003A4AEA    FF4D 08             dec dword ptr ss:[ebp+8]
003A4AED  ^ 0F85 4AFFFFFF       jnz 003A4A3D
003A4AF3    5F                  pop edi
003A4AF4    5E                  pop esi
003A4AF5    5B                  pop ebx
003A4AF6    C9                  leave
003A4AF7    C2 1400             retn 14
```

特殊函数我没有跟踪，偷了点懒，如果有兴趣的可以在那个几个`CALL`上下硬件执行断点，我跟过了确实能断下来，进入`CALL`后就可以得到你的函数了，当改完上面需要改的地方后，F9 运行，到达第5次除零异常。

可以用`Import REC`收集 IAT 了，有三个特殊函数，对照了一下原记事本的 IAT，用插件也可以认出来，不过有一个认不出，有一个认错了。

| 序号 | 地址 | 模块 | Ordinal | 函数 | 插件识别结果 |
|---|---|---|---|---|---|
| 1 | `005163A0` | `kernel32.dll` | `00AC` | `ExitProcess` | 插件认错了 |
| 1 | `005163A4` | `kernel32.dll` | `039E` | `lstrlen` | 插件没认出 |

其他是填充的垃圾数据，CUT掉就行了。改`OEP＝005110CC`，`FixDump`！

删除`00007000`区段，重建 PE 优化一下`dumped_.exe`即可。

## 脱壳后记

一开始碰到这个壳不知道怎么脱，发了贴也没人愿意回答，哎，心寒~！不过这里真的很感谢 stephenteh 和 fly 两位大哥（文中有些话是用的 fly 大哥的以前的脱1.1版的话，还请 fly 大哥海涵）是他们指导了我怎么脱这个壳，还有二哥的文章带我入了门。感谢曾帮助过我的人，还有谢谢您耐心看完此文，有什么不对的地方，还请大家指正~！


00790AA1 `. . 8D45 C0 lea eax,dword ptr ss:\[ebp-40\]`  
00790AA4 `. . 8A55 EF mov dl,byte ptr ss:\[ebp-11\]`  
00790AA7 `. . E8 8C44C7FF call dumped(?00404F38`  
00790AAC `. . 8B45 C0 mov eax,dword ptr ss:\[ebp-40\]`  
00790AAF `. . 8D55 F0 lea edx,dword ptr ss:\[ebp-10\]`  
00790AB2 `. . B9 09000000 mov ecx,9`  
00790AB7 `. . E8 4448C7FF call dumped(?00405300 ; insert d`  
00790ABC `. . 8B45 F8 mov eax,dword ptr ss:\[ebp-8\]`  
00790ABF `. . 8B55 F0 mov edx,dword ptr ss:\[ebp-10\]`  
00790AC2 `. . E8 E542C7FF call dumped(?00404DAC`  
00790AC7 `. . 33C0 xor eax,eax`  
00790AC9 `. . 5A pop edx`  
00790ACA `. . 59 pop ecx`  
00790ACB `. . 59 pop ecx`  
00790ACC `. . 64:8910 mov dword ptr fs:\[eax\],edx`  
00790ACF `. . 68 FE0A7900 push dumped(?00790AFE`  
00790AD4 `> . 8D45 C0 lea eax,dword ptr ss:\[ebp-40\]`  
00790AD7 `. . BA 03000000 mov edx,3`  
00790ADC `. . E8 9B42C7FF call dumped(?00404D7C`  
00790AE1 `. . 8D45 F0 lea eax,dword ptr ss:\[ebp-10\]`  
00790AE4 `. . BA 02000000 mov edx,2`  
00790AE9 `. . E8 8E42C7FF call dumped(?00404D7C`  
00790AEE `. . 8D45 FC lea eax,dword ptr ss:\[ebp-4\]`  
00790AF1 `. . E8 6242C7FF call dumped(?00404D58`  
00790AF6 `\. C3 retn ; 生成真正的注册码后，返回到上面 007CF00B 执行。`

下面是对真假注册码进行比较的部分：

```
0040535C /$ . 85C0 test eax,eax
0040535E |. . 74 40 je short dumped(?004053A0
00405360 |. . 85D2 test edx,edx
00405362 |. . 74 31 je short dumped(?00405395
00405364 |. . 53 push ebx
00405365 |. . 56 push esi
00405366 |. . 57 push edi
00405367 |. . 89C6 mov esi,eax ; esi 中存放真正的注册码
00405369 |. . 89D7 mov edi,edx ; edi 中存放我们输入的注册码
0040536B |. . 8B4F FC mov ecx,dword ptr ds:\[edi-4\] ; ecx 中存放我们输入的注册码的长度
0040536E |. . 57 push edi
0040536F |. . 8B56 FC mov edx,dword ptr ds:\[esi-4\] ; edx 中存放真注册码的长度
00405372 |. . 4A dec edx
00405373 |. . 78 1B js short dumped(?00405390
00405375 |. . 8A06 mov al,byte ptr ds:\[esi\] ; al 中放真注册码的第一个字符
00405377 |. . 46 inc esi
00405378 |. . 29D1 sub ecx,edx
0040537A |. . 7E 14 jle short dumped(?00405390
0040537C |> . F2:AE /repne scas byte ptr es:\[edi\] ; 在我们输入的注册码中找到程序生成的注册码的子串
0040537E |. . 75 10 |jnz short dumped(?00405390
00405380 |. . 89CB |mov ebx,ecx
00405382 |. . 56 |push esi
00405383 |. . 57 |push edi
00405384 |. . 89D1 |mov ecx,edx
00405386 |. . F3:A6 |repe cmps byte ptr es:\[edi\],byte>
00405388 |. . 5F |pop edi
00405389 |. . 5E |pop esi
0040538A |. . 74 0C |je short dumped(?00405398
0040538C |. . 89D9 |mov ecx,ebx
0040538E |.^ EB EC \jmp short dumped(?0040537C
00405390 |> . 5A pop edx
00405391 |. . 31C0 xor eax,eax
00405393 |. . EB 08 jmp short dumped(?0040539D
00405395 |> . 31C0 xor eax,eax
00405397 |. . C3 retn
00405398 |> . 5A pop edx
00405399 |. . 89F8 mov eax,edi
0040539B |. . 29D0 sub eax,edx ; eax = 3, 41-3E = 3
0040539D |> . 5F pop edi
0040539E |. . 5E pop esi
0040539F |. . 5B pop ebx
004053A0 \\> . C3 retn
```

通过对上面代码的分析，我们知道，只要程序生成注册码是我们输入的注册码的一个子串即可。因为程序还有另外一部分对注册码的其它部分进行校验。

我们在这里还应该注意到，真正注册码的生成和一个数有关：`A01BA81E`，这个数的产生和硬盘序列号有关，下面就是有关的算法：

```
00790624 /$ . 83C4 F4 add esp,-0C
00790627 |. . 6A 00 push 0 ; /pFileSystemNameSize = NULL
00790629 |. . 6A 00 push 0 ; |pFileSystemNameBuffer = NULL
0079062B |. . 8D4424 10 lea eax,dword ptr ss:\[esp+10\] ; |
0079062F |. . 50 push eax ; |pFileSystemFlags
00790630 |. . 8D4424 10 lea eax,dword ptr ss:\[esp+10\] ; |
00790634 |. . 50 push eax ; |pMaxFilenameLength
00790635 |. . 8D4424 10 lea eax,dword ptr ss:\[esp+10\] ; |
00790639 |. . 50 push eax ; |pVolumeSerialNumber
0079063A |. . 6A 00 push 0 ; |MaxVolumeNameSize = 0
0079063C |. . 6A 00 push 0 ; |VolumeNameBuffer = NULL
0079063E |. . 68 68067900 push dumped(?00790668 ; |RootPathName = "c:\\"
00790643 |. . E8 DC74C7FF call <jmp.&kernel32.GetVolumeInfo>; \GetVolumeInformationA
00790648 |. . 8B0424 mov eax,dword ptr ss:\[esp\]
0079064B |. . 05 85000000 add eax,85
00790650 |. . B9 22000000 mov ecx,22
00790655 |. . 33D2 xor edx,edx
00790657 |. . F7F1 div ecx
00790659 |. . 6BC0 29 imul eax,eax,29 ; EAX = A01BA81E，下面的计算中要用到这个值
0079065C |. . 890424 mov dword ptr ss:\[esp\],eax
0079065F |. . 8B0424 mov eax,dword ptr ss:\[esp\]
00790662 |. . 83C4 0C add esp,0C
00790665 \. . C3 retn
```

### 三、破解二

通过对上面程序的分析，我们能得到部分注册码：`b417d593d4834`。输入这个注册码。然后重新运行该软件。在经过一段时间后，又会跳出注册的对话框。

在对软件进行反复跟踪分析后，发现程序会对在地址 `00836948` 和 `0083694C` 处的值进行检查，如果是 `00000003`，程序认为是经过注册的，然后再次对注册码进行检查，如果检查正确，保持 `00000003` 不变，否则给这个变量赋 `00000002`。

最后程序又对这个值进行检查，如果是 `00000002` 表示没有完全注册；如果是 `00000003` 表示程序是正确注册的。（复杂呀！）

另外，在对程序的分析中，注册码的长度不能小于 14。程序对注册码的检查分为两种：

1. 注册码的长度为 14，程序使用密钥 `"YiISIvp * kb2W A9iIpC5 M#IbI kxIScYoWI='v2S2b'"` 进行验密；
2. 注册码的长度 > 14，程序使用密钥 `"YqhNrd8K1JOsbfZVUatRgB3McDiExTSu5GjvPekL2w7AXWzlIyHmQn94Co6Fp0#S_"` 和 `"_$#abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"` 作为密码字典。算法如下：

   拿出待校验的注册码的第一字符，在第一个字符串中，查找出它的位置 `X`，然后在第二个字符串中，取出第 `X` 个字符作为新串的第一个字符。然后再取出待校验注册码的第二个字符，在第一个字符串中找到它的位置 `Y`，然后再从第二个字符串找到第 `Y` 个字符，作为新串的第二个字符。生成新的字符串后，用这个长为 2 的新串和待校验注册码的倒数第 3 位开始，长度为 2 进行比较（如果是正确的注册码，应该相等）。然后再用待校验注册码的倒数第四位和 `0` 比，倒数第一位和 `1` 比。如果都相等了，我们的注册码校验就顺利通过了 :)

我们索性输入注册码为：`b417d593d4834Yj1`。

在 `00836948` 和 `0083694C` 处下硬件读断点，重新运行，在等到一段时间后，终于拦下来了 :)

```
007DE1F2 . 8338 03 cmp dword ptr ds:\[eax\],3 ; eax=00836948
007DE1F5 . 0F85 57020000 jnz dumped(?007DE452 ; 因为在程序启动的时候，从.ini 文件中读取了注册码，已经完成了部分检查，所以 ds:\[eax\]=3，在这里不跳转
007DE1FB . A1 A81D8300 mov eax,dword ptr ds:\[831DA8\] ; ds:\[eax\]中存放我们输入的注册码
007DE200 . 8338 00 cmp dword ptr ds:\[eax\],0 ; 当然这里不是 0 了 :)
007DE203 . 0F84 49020000 je dumped(?007DE452
007DE209 . A1 A81D8300 mov eax,dword ptr ds:\[831DA8\]
007DE20E . 8B00 mov eax,dword ptr ds:\[eax\] ; ds:\[eax\]再次拿到我们输入的注册码
007DE210 . E8 0B6EC2FF call dumped(?00405020 ; 取得注册码的长度
007DE215 . 8BF0 mov esi,eax ; 存入到 esi 中
007DE217 . A1 18188300 mov eax,dword ptr ds:\[831818\] ; eax=00836858
007DE21C . 8B00 mov eax,dword ptr ds:\[eax\] ; ds:\[eax\]中存放由程序自动生成的注册码
007DE21E . E8 FD6DC2FF call dumped(?00405020 ; 取得这个注册码的长度
007DE223 . 2BF0 sub esi,eax ; 两个注册码的长度相减，减的值放在 esi 中
007DE225 . 4E dec esi ; esi 减 1
007DE226 . 0F85 2E010000 jnz dumped(?007DE35A ; 如果不等于 0，则跳转
007DE22C . 8D45 F8 lea eax,dword ptr ss:\[ebp-8\] ; 下面是对长度为 14 的密码进行校验的算法。这次我们先不对它进行分析
...
```


# tElock V0.98 加壳的 DLL

> 原文：[`www.pediy.com/kssd/pediy06/pediy6722.htm`](https://www.pediy.com/kssd/pediy06/pediy6722.htm)

【目标程序】：`tElock` 加壳的 `EdrLib.dll`。附件中含有 `UnPacked` 以供参考。

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`Ollydbg1.10`、`LordPE`

【实例下载】：点击此处下载

---

【脱壳过程】：

借用《解密与解密》第二版中的话：`tElock` 加壳的 DLL 的致命弱点就是输入表和重定位加密代码可以被跳过！也就是说我们只要搞定了这两个 `Magic Jump` 就行了。下面的方法和书上的操作流程有点不同，其实原理是一样的。

---

## 一、双重 Magic Jump：避开重定位和输入表加密

设置 `Ollydbg` 忽略除了“INT3 中断”之外的所有其它异常选项。老规矩：用 `IsDebug 1.4` 插件去掉 `Ollydbg` 的调试器标志。

```
003B8BD6     E9 25E4FFFF         jmp EdrLib.003B7000 //进入 OD 后停在这
003B8BD0     FF6424 04          jmp dword ptr ss:[esp+4]
```

`Shift+F9` 运行，程序中断在 `INT3` 异常处。

```
003B708D     90                nop //第 1 次异常
003B7B28     90                nop //第 2 次异常
```

提供一个简便寻找 `ASPack` 加壳 DLL 的重定位处理代码段的方法！

现在设置 `Ollydbg` 忽略所有异常选项。`Ctrl+S` 在“整个段块”搜索命令序列：

```
mov ebx,edx
shr ebx,10
mov eax,dword ptr ds:[esi]
```

找到在 `003B8195` 处！我们在其上的 `003B8177` 处下断，`Shift+F9` 运行，中断。

```
003B8177     8BB5 56D34000      mov esi,dword ptr ss:[ebp+40D356]
 // [ebp+40D356]=[003B8CB4]=00006000 ★ 这个 00006000 就是重定位表的 RVA！
003B817D     85F6              test esi,esi
003B817F     0F84 8B000000     je EdrLib.003B8210
003B8185     8B95 62D34000     mov edx,dword ptr ss:[ebp+40D362]
003B818B     03F2              add esi,edx
003B818D     2B95 66D34000     sub edx,dword ptr ss:[ebp+40D366]
 //这里其实就是检测与映像基址是否相符，不符则重定位处理！★
003B8193     74 7B             je short EdrLib.003B8210
 //Magic Jump，可以第 2 次 Load 后在这里改标志位 Z=1，使其跳转 ★
003B8195     8BDA              mov ebx,edx //找到这里！
003B8197     C1EB 10           shr ebx,10
003B819A     8B06              mov eax,dword ptr ds:[esi]
003B819C     85C0              test eax,eax
003B819E     74 70             je short EdrLib.003B8210
 //重定位处理完毕后这里就跳转       在 003B8210 处下断
003B81A0     8B4E 04           mov ecx,dword ptr ds:[esi+4]
003B81A3     83E9 08           sub ecx,8
003B81A6     D1E9              shr ecx,1
003B81A8     8BBD 62D34000     mov edi,dword ptr ss:[ebp+40D362]
003B81AE     03F8              add edi,eax
003B81B0     83C6 08           add esi,8
003B81B3     0FB706            movzx eax,word ptr ds:[esi]
003B81B6     C1C8 0C           ror eax,0C
003B81B9     FEC8              dec al
003B81BB     78 4C             js short EdrLib.003B8209
003B81BD     74 0E             je short EdrLib.003B81CD
003B81BF     FEC8              dec al
003B81C1     74 13             je short EdrLib.003B81D6
003B81C3     FEC8              dec al
003B81C5     74 3C             je short EdrLib.003B8203
003B81C7     FEC8              dec al
003B81C9     74 14             je short EdrLib.003B81DF
003B81CB     EB 3C             jmp short EdrLib.003B8209
003B81CD     C1E8 14           shr eax,14
003B81D0     66:011C07         add word ptr ds:[edi+eax],bx
003B81D4     EB 33             jmp short EdrLib.003B8209
003B81D6     C1E8 14           shr eax,14
003B81D9     66:011407         add word ptr ds:[edi+eax],dx
003B81DD     EB 2A             jmp short EdrLib.003B8209
003B81DF     52                push edx
003B81E0     C1E8 14           shr eax,14
003B81E3     8BD8              mov ebx,eax
003B81E5     C1E0 10           shl eax,10
003B81E8     66:8B16           mov dx,word ptr ds:[esi]
003B81EB     66:81E2 FF0F      and dx,0FFF
003B81F0     66:8BC2           mov ax,dx
003B81F3     5A                pop edx
003B81F4     8D8410 00800000   lea eax,dword ptr ds:[eax+edx+8000]
003B81FB     89041F            mov dword ptr ds:[edi+ebx],eax
003B81FE     46                inc esi
003B81FF     46                inc esi
003B8200     49                dec ecx
003B8201     EB 06             jmp short EdrLib.003B8209
003B8203     C1E8 14           shr eax,14
003B8206     011407            add dword ptr ds:[edi+eax],edx
003B8209     46                inc esi
003B820A     46                inc esi
003B820B     49                dec ecx
003B820C     7F A5             jg short EdrLib.003B81B3
003B820E     EB 8A             jmp short EdrLib.003B819A //循环处理
003B8210     8B95 62D34000     mov edx,dword ptr ss:[ebp+40D362]
 //当我们中断在 003B8210 处时，重定位处理完毕。此时 ESI=003B63B0，★ 就是重定位表的结束地址啦。
 //得到重定位表信息：RVA=00006000，大小＝000003B0
003B8216     8BB5 52D34000     mov esi,dword ptr ss:[ebp+40D352]
 // [ebp+40D352]=[003B8CB0]=0000442C ★ 这就是输入表的 RVA 啦。
003B821C     85F6              test esi,esi
003B821E     0F84 06040000     je EdrLib.003B862A
 //第 2 个 Magic Jump，改标志位 Z=1，使其跳转，则不加密 IAT
003B8224     03F2              add esi,edx
003B8226     83A5 52D44000 00  and dword ptr ss:[ebp+40D452],0
003B822D     8B46 0C           mov eax,dword ptr ds:[esi+C]
003B8230     8366 0C 00        and dword ptr ds:[esi+C],0
003B8234     85C0              test eax,eax
003B8236     0F84 EE030000     je EdrLib.003B862A
003B823C     03C2              add eax,edx
003B823E     8BD8              mov ebx,eax
003B8240     50                push eax
003B8241     FF95 D0D24000     call dword ptr ss:[ebp+40D2D0] //GetModuleHandleA
003B8247     85C0              test eax,eax
003B8249     0F85 BA000000     jnz EdrLib.003B8309
```

咱们已经得到重定位表信息了，所以可以再次载入这个 DLL，重复上面的步骤，在 `003B8193` 处改标志位 `Z=1`，使其跳转，避开重定位表加密！

---

## 二、第 2 区段内存断点法：快速直达 OEP

当我们修改了以上两个 `Magic Jump`，自 `003B821E` 处跳至 `003B862A`。

```
003B862A     8BBD 5AD34000     mov edi,dword ptr ss:[ebp+40D35A]
003B8630     85FF              test edi,edi
003B8632     EB 03             jmp short EdrLib.003B8637
```

现在我们 `Alt+M` 打开 内存查看 窗口：

```
003B0000   00001000   EdrLib   003B0000 (itself)  PE header //第 1 区段
003B1000   00003000   EdrLib   003B0000             code //第 2 区段 ★
003B4000   00001000   EdrLib   003B0000             data
```

在 `003B1000` 第 2 区段上 设置内存访问断点！`Shift+F9` 运行，直接中断在 `OEP` 处！

```
003B11C9     55                push ebp //OEP ★
003B11CA     8BEC              mov ebp,esp
003B11CC     53                push ebx
003B11CD     8B5D 08           mov ebx,dword ptr ss:[ebp+8]
003B11D0     56                push esi
003B11D1     8B75 0C           mov esi,dword ptr ss:[ebp+C]
003B11D4     57                push edi
003B11D5     8B7D 10           mov edi,dword ptr ss:[ebp+10]
003B11D8     85F6              test esi,esi
003B11DA     75 09             jnz short EdrLib.003B11E5
```

用 `LordPE` 选中 `Ollydbg` 的 `loaddll.exe` 的进程，在下面的列表里选择 `EdrLib.dll`，然后完整脱壳，得到 `dumped.dll`。

---

## 三、PE 修正：完成脱壳

因为我们已经跳过了双重 `Magic Jump`，所以使得修复工作变得很简单：

用 `LordPE` 修正 `dumped.dll` 的 `OEP=000011C9`，输入表 `RVA=0000442C`，保存之。

用 `LordPE` 修正 `dumped.dll` 的重定位表 `RVA=00006000`、大小＝`000003B0`，保存之。

OK，脱壳完成啦。

---

```
           ,     \_/ 
        /| \_.-~/    \\_   ,      青春都一晌
      ( /~   /        \\~\. |
      `\     /_/        \\   ~\    忍把浮名 
  \_-~~~-.)  \_/;;,.     \\_  //'
 /'\_,\   --~   \ ~~~-  ,;;\_\_(  (.-~~~-.   换了破解轻狂
 `~ _( ,_.--\ (   ,;'' /   ~--   /.\_`
  /~~//'   / `~\      ![](img/e45fac309594d4f9580d902b0ed40a73.jpg) /--..\_,  ![](img/e45fac309594d4f9580d902b0ed40a73.jpg)_  `~
   "  `~"  "     "`      /~'`\   `\~~\
           "     "   "~'  ""
```

Cracked By 巢水工作坊——fly [OCN][FCG][NUKE][DCM]

2004-06-02  01:00

`4dd466d0e1e0cc123afbe504da2462ca`

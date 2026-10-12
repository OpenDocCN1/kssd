
# eXPressor V1.0 脱壳+破解——eXPressor 主程序

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-486.htm`](https://www.pediy.com/kssd/pediy07/pediy7-486.htm)

eXPressor V1.0 脱壳+破解——eXPressor 主程序

下载页面：`http://itimer.home.ro/`

软件大小：87 KB

软件简介：`eXpressor` can compress `EXE` files to half their normal size. Once compressed, the files execute just like normal.

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`Ollydbg` V1.10、`PEiD`、`LordPE`

---

【脱壳过程】：

`PEiD` 的 Sign：

`[eXPressor V1.X -> CGSoftLabs]`

`signature = E9 35 14 00 00 E9 31 13 00 00 E9 98 12 00 00 E9 EF 0C 00 00 E9 42 13 00 00 E9 E9 02 00 00 E9 EF 0B 00 00 E9 1B 0D 00 00`

---

`eXPressor` 是新出的压缩壳，脱壳很简单。

```
00432019 E9 35140000 jmp eXPresso.00433453
```

// 进入 `Ollydbg` 后暂停在这

`Ctrl+F` 在当前位置下搜索命令：

```
add eax,dword ptr ds:[edx+0F0]
```

找到在 `00433739` 处

直接下命令：`G 00433739`

确定弹出的"Nfo"提示，中断下来

```
00433730 8B55 F4       mov edx,dword ptr ss:[ebp-C]
00433733 8B85 ECFEFFFF mov eax,dword ptr ss:[ebp-114]
00433739 0382 F0000000 add eax,dword ptr ds:[edx+F0]
```

// `[edx+F0]=[004001F0]=0001F858` 输入表的 RVA ★

```
0043373F 8985 38FEFFFF mov dword ptr ss:[ebp-1C8],eax
00433745 8B8D 38FEFFFF mov ecx,dword ptr ss:[ebp-1C8]
0043374B 8379 0C 00    cmp dword ptr ds:[ecx+C],0
0043374F 0F84 B4020000 je eXPresso.00433A09
```

此时程序已经解开，API 函数的系统地址还未填充到 `IAT` 中，正是 Dump 的好时机！

运行 `LordPE` 完全 Dump 这个进程。

下面来寻找 OEP 了。

`Ctrl+F` 搜索命令：

```
jmp dword ptr ss:[ebp-1B0]
```

找到在 `00433C24` 处，`F4` 过去

```
00433C06 8B95 ECFEFFFF mov edx,dword ptr ss:[ebp-114]
00433C0C 0315 80904300 add edx,dword ptr ds:[439080]
00433C12 8B45 F4       mov eax,dword ptr ss:[ebp-C]
00433C15 0350 34       add edx,dword ptr ds:[eax+34]
00433C18 2B95 ECFEFFFF sub edx,dword ptr ss:[ebp-114]
00433C1E 8995 50FEFFFF mov dword ptr ss:[ebp-1B0],edx
00433C24 FFA5 50FEFFFF jmp dword ptr ss:[ebp-1B0] ; eXPresso.00404C19
```

// 飞向光明之巅！ ^O^

用 `LordPE` 修正脱壳文件的 OEP `RVA=00004C19`，修正 Import Table `RVA=0001F858`

删除`.epld`、`.eplid`、`.epldat`、`.eplrl` 壳区段，然后重建 PE 优化一下。

这样马马虎虎也算是完美脱壳吧。OK，脱壳完成了！

破解也很简单。去除运行时的 NAG，把 `0042E42D` 处的 `00` 修改为 `01` 即可。

---

```
          ,      \_/   
        /| \_.-~/          \\_     ,      青春都一晌
       ( /~   /           \\~-.\ |\\
       \`\\  \_/             \   ~\ )      忍把浮名  
  _-~~~-.)  )\_;;,.           \\__ //’
  /'\_,\   --~   \ ~~~-  ,;;\__\(  (.-~~~-.      换了破解轻狂
 \~ _( ,\_..--\ (     ,;'' /   ~--   /.\_\  \  /~~//'  /' \`~\   ) /--..\_, )\_ \~
  " \`~" "      \`"   /~'\`\\   \\~~\\       
          "   "   "~'  ""
        UnPacked By :  fly
           2005-02-01 10:00
```

`4dd466d0e1e0cc123afbe504da2462ca`

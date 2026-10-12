
# eXcalibur V1.03

> 原文：[`www.pediy.com/kssd/pediy06/pediy6675.htm`](https://www.pediy.com/kssd/pediy06/pediy6675.htm)

**linson 的 eXcalibur V1.03 脱壳——eXcalibur.exe 主程序**

【软件简介】：linson 的新壳

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【实例下载】：点击此处下载。

【调试环境】：`WinXP`、`Ollydbg`、`PEiD`、`LordPE`、`ImportREC`

—————————————————————————————————

【脱壳过程】：

linson 喜欢吓人，eXcalibur 伪装成 Xtreme-Protector 1.00-1.05 -> Rafael Ahucha & Sergio Lara 壳。

eXcalibur 比较精悍。具有压缩效果，新增了几处反跟踪和 Stolen Call，没有自校验。熟悉之后脱壳就没什么难度了。

eXcalibur 对于汇编和 Delphi 的程序支持较好。`Yjjh.mid` 大小：13.9K，欣赏 linson 提供的音乐吧。

—————————————————————————————————

## 一、`EnableWindow` 反跟踪

设置 `Ollydbg` 忽略所有异常选项。老规矩：用 `IsDebug` 1.4 插件去掉 `Ollydbg` 的调试器标志。

```
代码:
0040B000     E9 00000000               jmp eXcalibu.0040B005//进入 OD 后停在这
0040B005     60                        pushad
0040B006     E8 14000000               call eXcalibu.0040B01F
```

BP `GetForegroundWindow` 断下后取消断点，`Alt+F9` 返回

```
代码:
003B0219     FFD0                      call eax ; USER32.GetForegroundWindow
003B021B     8985 A4394000             mov dword ptr ss:[ebp+4039A4],eax//返回这里
003B0221     E8 0D000000               call 003B0233
```

```
代码:
003B0233     56                        push esi
003B0234     FFD7                      call edi
003B0236     8985 94394000             mov dword ptr ss:[ebp+403994],eax; USER32.EnableWindow
003B023C     6A 00                     push 0//改为：push 1  ★
003B023E     FFB5 A4394000             push dword ptr ss:[ebp+4039A4]
003B0244     FFD0                      call eax ; USER32.EnableWindow
003B0246     E8 0A000000               call 003B0255
```

————————————————————————

## 二、避开 Stolen Call

linson 这次把程序中的不少 `Call` 都“偷走”了，呵呵，想办法让他住手！

BP `VirtualAlloc` 断下后取消断点，`Alt+F9` 返回

```
代码:
003B02EB     FF15 12B44000             call dword ptr ds:[<&KERNEL32.VirtualAlloc>]
003B02F1     FC                        cld
003B02F2     8BF8                      mov edi,eax
003B02F4     8BB5 3C374000             mov esi,dword ptr ss:[ebp+40373C]
003B02FA     46                        inc esi
003B02FB     B9 10000000               mov ecx,10
003B0300     E8 5E000000               call 003B0363
003B0305     85C0                      test eax,eax//目标 CALL
003B0307     74 07                     je short 003B0310//改变标志位 Z=1， 使这里 JMP ★
003B0309     E8 08000000               call 003B0316//linson 开始破坏 :-) Stolen Call
003B030E     E2 F0                     loopd short 003B0300
003B0310     61                        popad
003B0311     E9 BE000000               jmp 003B03D4
```

————————————————————————

## 三、`ZwSetInformationThread` 反跟踪

```
代码:
003B03D4     8BBD 98394000             mov edi,dword ptr ss:[ebp+403998]
003B03DA     85FF                      test edi,edi
003B03DC     74 0F                     je short 003B03ED
003B03DE     FF95 78394000             call dword ptr ss:[ebp+403978]; kernel32.GetCurrentThread
003B03E4     6A 00                     push 0
003B03E6     6A 00                     push 0
003B03E8     6A 11                     push 11
003B03EA     50                        push eax
003B03EB     FFD7                      call edi; ntdll.ZwSetInformationThread//不能客气，当然是 NOP 掉！★
003B03ED     BE B4120000               mov esi,12B4
003B03F2     85F6                      test esi,esi
003B03F4     0F84 CA050000             je 003B09C4//记住这个地址，在 003B09C4 处下断
```

————————————————————————

## 四、避开 IAT 加密

快点的方法：`Ctrl+B` 在当前位置下搜索 16 进制值：`61 80` 找到在 `003B0985` 处

```
代码:
003B0975     61                        popad//这里下断，F9 运行断下
003B0976     3385 E0394000             xor eax,dword ptr ss:[ebp+4039E0]
003B097C     8907                      mov dword ptr ds:[edi],eax//修改
003B097E     8385 E4394000 04          add dword ptr ss:[ebp+4039E4],4
003B0985     61                        popad//找到这里
003B0986     80BD EC394000 00          cmp byte ptr ss:[ebp+4039EC],0
003B098D     75 1C                     jnz short 003B09AB
```

`003B097C` 处改为：

```
代码:
003B097C     61                        popad//呵呵，先来个 POPAD ★
003B097D     8907                      mov dword ptr ds:[edi],eax//正确的函数写入地址
003B097F     8385 E4394000 04          add dword ptr ss:[ebp+4039E4],4
003B0986     80BD EC394000 00          cmp byte ptr ss:[ebp+4039EC],0
003B098D     75 1C                     jnz short 003B09AB
```

————————————————————————

## 五、走至 OEP、DUMP、完成脱壳

还记得在 `003B09C4` 处下断吧？避开 IAT 加密后 `F9` 运行断在 `003B09C4` 处

```
代码:
003B09C4     E8 23020000               call 003B0BEC//中断。断下后取消断点
```

下断：BP `VirtualFree` 断下后取消断点，`Alt+F9` 返回

```
代码:
0040B001     FF15 16B44000             call dword ptr ds:[<&KERNEL32.VirtualFree>]
0040B007     9C                        pushfd//返回这里
```

花指令晃眼。`Ctrl+B` 在当前位置下搜索 16 进制值：`9D EB` 找到在 `0040B030` 处

```
代码:
0040B02D     83C4 04                   add esp,4
0040B030     9D                        popfd//下断，F9 断下
0040B031     EB 01                     jmp short eXcalibu.0040B034

0040B034     33C0                      xor eax,eax
0040B036     64:8F00                   pop dword ptr fs:[eax]
0040B039     83C4 0C                   add esp,0C
0040B03C     E8 01000000               call eXcalibu.0040B042

0040B042     58                        pop eax
0040B043     9D                        popfd
0040B044     61                        popad
0040B045     E8 15000000               call eXcalibu.0040B05F

0040B057     68 203A4000               push eXcalibu.00403A20
0040B05C     EB 01                     jmp short eXcalibu.0040B05F
```

在 `0040B062` 处中断几次就走到 OEP 啦

```
代码:
0040B05F     58                        pop eax
0040B060     40                        inc eax
0040B061     50                        push eax ; eXcalibu.00403A21
0040B062     C3                        retn
//返回 00403A21 飞向光明之巅！:-)
```

```
代码:
00403A21     6A 00                     push 0//在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程
00403A23     E8 8A000000               call eXcalibu.00403AB2 ; GetModuleHandleA
00403A28     A3 9C504000               mov dword ptr ds:[40509C],eax
00403A2D     E8 3A010000               call eXcalibu.00403B6C ; InitCommonControls
```

```
代码:
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆
下面可以看见 linson 或者说 forgot 给大家的留言:

00403A4A  68 69 20 2C 20 69 20 61 6D 20 66 6F 72 67 6F 74  hi , i am forgot
00403A5A  2C 20 64 69 64 20 79 6F 75 20 75 6E 70 61 63 6B  , did you unpack
00403A6A  20 74 68 69 73 20 70 61 63 6B 65 72 3F 20 3A 44  this packer? :d
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆
```

运行 `ImportREC`，选择这个进程。把 OEP 改为 `00003A21`，改 `RVA=00001408`，大小＝`000000C8`，点“Get Import”。

所有的函数都是有效的，当然是 `FixDump`，正常运行啦！

—————————————————————————————————

```
代码:
        ,     \_/
       /| \_.-~/            \\\_     ,        青春都一晌
      ( /~   /               \\~-.\_ |\\
      \`\\\\  \_/                    \   ~\             忍把浮名
  \_-~~~-.)  \_\_/;;,.               \\\  //'
 /'\_,\\   --~   \ ~~~-  ,;;\\_\\_(  (.-~~~-.        换了破解轻狂
 \`~ \_( ,\_..--\ (     ,;'' /    ~--   /.\_\\`
  /~~//'   /' \`~\          ![](img/e45fac309594d4f9580d902b0ed40a73.jpg) /--..\_,  ![](img/e45fac309594d4f9580d902b0ed40a73.jpg)\_  \`~
  "  \`~"  "      \`"      /~'\`\\    \`\\\\~~\\
                    "     "   "~'  ""
```

Cracked By 巢水工作坊——fly [OCN][FCG][NUKE][DCM]

2004-04-29 21:35 4dd466d0e1e0cc123afbe504da2462ca

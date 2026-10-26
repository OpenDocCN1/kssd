
立刻出现 `debugger detection` 的讯息

的讯息

;

所以这个 `call` 一定有问题

```
014F:00C11337  33C0                XOR       EAX,EAX
014F:00C11339  5A                  POP       EDX
014F:00C1133A  59                  POP       ECX
```

9:

再 `load` 一次 , 一 `load`, 立刻下 `g c11332`, 到了之后

立刻按 `F8`, 进入这个 `call` 去 `trace`

到了这边 ,

```
014F:00C10EFE  8BD3                MOV       EDX,EBX
014F:00C10F00  8B45F8              MOV       EAX,[EBP-08]
014F:00C10F03  E858F9FFFF          CALL      00C10860
014F:00C10F08  8A154466C100        MOV       DL,[00C16644]
014F:00C10F0E  8B45F8              MOV       EAX,[EBP-08]
014F:00C10F11  E88EF9FFFF          CALL      00C108A4
014F:00C10F16  8BD8                MOV       EBX,EAX
014F:00C10F18  85DB                TEST      EBX,EBX
014F:00C10F1A  7445                JZ        00C10F61
```

;
只要这个能跳 , 就没有 `debugger detection` 的讯息了

;
所以到了这边 , 可以下 `r fl z`, 来跳过

```
014F:00C10F1C  6A00                PUSH      00
014F:00C10F1E  A17066C100          MOV       EAX,[00C16670]
014F:00C10F23  50                  PUSH      EAX
```

到了这边

```
014F:00C11046  7425                JZ        00C1106D
014F:00C11048  8D55E4              LEA       EDX,[EBP-1C]
014F:00C1104B  33C0                XOR       EAX,EAX
014F:00C1104D  E8D616FFFF          CALL      00C02728
014F:00C11052  8B45E4              MOV       EAX,[EBP-1C]
014F:00C11055  8D55E8              LEA       EDX,[EBP-18]
014F:00C11058  E8C33DFFFF          CALL      00C04E20
014F:00C1105D  8B45E8              MOV       EAX,[EBP-18]
014F:00C11060  E83323FFFF          CALL      00C03398
014F:00C11065  8B5303              MOV       EDX,[EBX+03]
014F:00C11068  E853F9FFFF          CALL      00C109C0
```

;
走过这边 , 立刻出现 `file corrupted` 的讯息

;
应该是 `checksum` 吧 , 因为刚刚改了 `section character`

```
014F:00C1106D  8B45F8              MOV       EAX,[EBP-08]
014F:00C11070  E8AF18FFFF          CALL      00C02924
```

10:
那也要跳过 `cs:00c11068` 才行 , 那乾脆到了

```
014F:00C10F1A  7445                JZ        00C10F61
```

这个本来是要跳过 `debugger dection` 的 , 乾脆一次跳个够吧

用 `symbol loader` 载入 , 下 `G C10F1A`, 到了这边 , 下 `A`, 下

`JMP C1106D` 吧 , 抄下机械码 , 是 `E94E010000`

11:
到了 `CS:C1106D`, 继续 `TRACE`, 到了这边

```
014F:00C1141D  8B4508              MOV       EAX,[EBP+08]
014F:00C11420  E87BFCFFFF          CALL      00C110A0
```

;
这个 `CALL` 一走过就出现过期的讯息了

```
014F:00C11425  33C0                XOR       EAX,EAX
014F:00C11427  5A                  POP       EDX
014F:00C11428  59                  POP       ECX
014F:00C11429  59                  POP       ECX
```

12:

再 `load` 一次 , 一 `load`, 立刻下 `g c11420`, 到了之后

立刻按 `F8`, 进入这个 `call` 去 `trace`

到了这边 ,

```
014F:00C111E2  E80DEEFFFF          CALL      00C0FFF4
014F:00C111E7  84C0                TEST      AL,AL
014F:00C111E9  7456                JZ        00C11241
```

;
可以从这边下手

```
014F:00C111EB  8B55F4              MOV       EDX,[EBP-0C]
014F:00C111EE  8B45E8              MOV       EAX,[EBP-18]
014F:00C111F1  E8A2EFFFFF          CALL      00C10198
014F:00C111F6  8D55C8              LEA       EDX,[EBP-38]
014F:00C111F9  8B45E8              MOV       EAX,[EBP-18]
014F:00C111FC  E81FF0FFFF          CALL      00C10220
014F:00C11201  8B55C8              MOV       EDX,[EBP-38]
014F:00C11204  B8F076C100          MOV       EAX,00C176F0
014F:00C11209  E8A21EFFFF          CALL      00C030B0
014F:00C1120E  8D55CE              LEA       EDX,[EBP-32]
014F:00C11211  8B45E8              MOV       EAX,[EBP-18]
014F:00C11214  E893F0FFFF          CALL      00C102AC
014F:00C11219  33C0                XOR       EAX,EAX
014F:00C1121B  8A45CF              MOV       AL,[EBP-31]
014F:00C1121E  50                  PUSH      EAX
014F:00C1121F  8D45D0              LEA       EAX,[EBP-30]
014F:00C11222  50                  PUSH      EAX
014F:00C11223  E8C0F0FFFF          CALL      00C102E8
014F:00C11228  84C0                TEST      AL,AL
014F:00C1122A  751A                JNZ       00C11246
014F:00C1122C  6A00                PUSH      00
014F:00C1122E  68DC12C100          PUSH      00C112DC
014F:00C11233  68E412C100          PUSH      00C112E4
014F:00C11238  6A00                PUSH      00
014F:00C1123A  E8CD32FFFF          CALL      USER32!MessageBoxA
014F:00C1123F  EB05                JMP       00C11246
014F:00C11241  E88AF4FFFF          CALL      00C106D0
```

;
这个会出现 过期的讯息

```
014F:00C11246  33C0                XOR       EAX,EAX
014F:00C11248  5A                  POP       EDX
014F:00C11249  59                  POP       ECX
014F:00C1124A  59                  POP       ECX
014F:00C1124B  648910              MOV       FS:[EAX],EDX
014F:00C1124E  EB0F                JMP       00C1125F
014F:00C11250  E99F19FFFF          JMP       00C02BF4
014F:00C11255  E876F4FFFF          CALL      00C106D0
014F:00C1125A  E8011BFFFF          CALL      00C02D60
014F:00C1125F  33C0                XOR       EAX,EAX
```

13:
所以我们知道要改哪里了

在

```
014F:00C10F1A  7445                JZ        00C10F61
```

改成 `JMP C1106D` 吧 ,

也就是把机械码 `74456A00A1`

改成 `E94E010000`

第二个要改的地方是

```
014F:00C111E9  7456                JZ        00C11241
```

改成 `JMP 00C1125F` 就行了

所以是机械码 `7456`

改成 `EB74`

问题是要改吗 , 此软体有加壳耶 , 所以无法直接用 `ULTRAEDIT` 改

那就用 `Process Patcher` 吧

14

将 `ppatcher.ppc`, `ppatcher.exe copy` 到 `aspack 2.000` 的目录底下

修改 `ppatcher.ppc`, 使他的内容如下

```
#Process Patcher Configuration File

Version=3.60

DisplayName=
破  aspack2000

Filename=aspack.exe

Filesize=231424

Arguments=/quiet

WaitInfinite=true

Address=0xc10f1a:0x74:0xe9

Address=0xc10f1b:0x45:0x4e

Address=0xc10f1c:0x6a:0x01

Address=0xc10f1d:0x00:0x00

Address=0xc10f1e:0xa1:0x00

Address=0xc111e9:0x74:0xeb

Address=0xc111ea:0x56:0x74

#End of Configuration File
```

15:
这样执行 `ppatcher`, 就可以了 , 但奇怪的是 `debugger detection` 有时会过不了 ,

只好执行 `FrogsICE`, 在执行 `ppatcher` 了

但这样子仍是不满意 , 我是想用 `SMC(self modifying code)` 的技巧来改

但是 , 使用 `SMC` 有一个问题 , 要找哪里的 code 来放我所加入的 `code` 呢 ?

我的确是有找 , 但是找不到一个够大的地方来存放我的 `code`,

只好用 `topo` 来多一些空间来存放我的 `code` 了

这个 `topo` 真是太棒了 , 可以多一些 `section` 出来 , 可以放入我所想要的 `code`

所以先 启动 `topo`, 开启 `aspack.exe`, 旁边有三个选项

`Backup file`, 等等 , 通通打勾

`Byte to be add`, 随意填一个 , 我是填 `50`,

在按 `Do it!` 好了之后 , 我的 `aspack.exe` 档案变大了

从 `231424 byte` 变成 `232960`, 好像大了一点喔 , 没关系 , 可以用就好了

下面还有一个框 , 写著

`memory address : 478000h`

`file offset 38800h`

16:
用 `symbol loader` 载入看看

一开始是 `invalid`, 没关系 , 按一下 `F8`, 如果画面乱掉了 , 就按两下 `F4`, 如果画面没乱掉

就不必了 . 一直 `trace`, 到了这边 ,

```
014F:0047802C  90                  NOP
014F:0047802D  E9CFDFFEFF          JMP       00466001
014F:00478032  0000                ADD       [EAX],AL
014F:00478034  0000                ADD       [EAX],AL
014F:00478036  0000                ADD       [EAX],AL
```

那就是说 `cs:478032` 以后可以任我使用了 , 其实把 `cs:478000` 改成

`JMP       00466001`

也是可以啦 , 反正只要执行没问题就 `ok` 了

17:
那要改哪里呢 ? 用 `symbol loader` 载入 ,

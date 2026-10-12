
## 逆向分析过程

用 `RegMon` 监控发现程序对注册表键值 `HKEY_LOCALMACHINE\Software\ynxx\EStudy\RegInfo="Unregistered Version"` 的可疑读操作。猜测程序会把读到的数据与正确的信息进行比对，以确定是否以注册版方式运行。

所以在 `SoftICE` 中下条件断点 `bpx regqueryvalueexa if *(esp->8)=='RegI' do "d esp->14"`。这个断点的作用是当有程序读取注册表中前四位为 `"RegI"` 的键值时，`SoftICE` 将中断并显示读出的数据。

1. 运行程序，`SoftICE` 中断，`F12` 返回程序领空。
2. 查看数据区是否为 `"Unregistered Version"`，若没有就继续 `F5` 运行程序。
3. 在第三次中断时发现读取的正是 `RegInfo`，这时 `"Unregistered Version"` 字串的地址是 `0030:00EB21F8`。
4. 下一步程序肯定要对这个地址进行读操作来比对，所以先 `bc *` 清除以前的断点，下地址断点 `bpm eb21f8`。
5. `F5` 后中断在如下位置：

```asm
017F:00409FCE  8BC0                MOV       EAX,EAX  
017F:00409FD0  89FA                MOV       EDX,EDI  
017F:00409FD2  89C7                MOV       EDI,EAX  
017F:00409FD4  B9FFFFFFFF          MOV       ECX,FFFFFFFF  
017F:00409FD9  32C0                XOR       AL,AL  
017F:00409FDB  F2AE                REPNZ SCASB<==停在这里,在计算字串的长度,不赶兴趣,继续 F5  
017F:00409FDD  B8FEFFFFFF          MOV       EAX,FFFFFFFE  
017F:00409FE2  29C8                SUB       EAX,ECX  
017F:00409FE4  89D7                MOV       EDI,EDX<==edi=14h（即十进制的 20，“Unregistered Version”不是 20 位吗！）  
017F:00409FE6  C3                  RET
```

再次中断停在下面：

```asm
017F:00404F57  7426                JZ        00404F7F  
017F:00404F59  8B0E                MOV       ECX,[ESI]  
017F:00404F5B  8B1F                MOV       EBX,[EDI]  
017F:00404F5D  39D9                CMP       ECX,EBX<==停在这里,往上看发现 ecx 来自[esi],"d esi"看到一长串字符  
017F:00404F5F  7558                JNZ       00404FB9    "yi8z2k0d2k4jg595g71jg551h91ehbtwhd18"  
017F:00404F61  4A                  DEC       EDX         而 ebx 来自[edi],"d edi"看到是"Unregistered Version"  
017F:00404F62  7415                JZ        00404F79  
017F:00404F64  8B4E04              MOV       ECX,[ESI+04]  
017F:00404F67  8B5F04              MOV       EBX,[EDI+04]  
017F:00404F6A  39D9                CMP       ECX,EBX  
017F:00404F6C  754B                JNZ       00404FB9
```

先记下这串可疑字串，再次 `F5` 运行程序又中断在 `017F:00404F5D` 处。这次 `"d esi"` 发现变为 `"yi8z2k0d2k4jg595g71jg551020y04ocw4oc"` 而 `"d edi"` 看到的仍然是 `"Unregistered Version"`。再次 `F5` 把控制权交还给程序，程序运行了，`SoftICE` 没有中断。

## 注册验证结论

下面打开注册表找到 `"RegInfo"` 将键值改为 `"yi8z2k0d2k4jg595g71jg551h91ehbtwhd18"`，然后运行程序，打开 `"关于"` 菜单，显示为 `"已注册版本"`!!!

而改为第二次发现的字串 `"yi8z2k0d2k4jg595g71jg551020y04ocw4oc"` 后运行程序，发现也是注册版，不过在关于里显示的是 `"友情注册版"`。

**补充说明：** 关于自由英语 XP1.00 Build 778 版的 `pj`

## 自由英语 XP1.0 Build 778 破解方法

关于自由英语 XP1.0 Build 778 版本，加密方式与 0.97 版一样，关键比对在 `0050e603`, `0050e610`, `0050e61d`，对比成功后，分别为 `"友情注册版"` `"已注册版"` `"60 天试用版"`。

破解方法：脱壳后用二进制编辑器查找 `"8138611E0000741AA198245300"` 改为 `"C700B8220000C700B822000090"`，或用内存修改法。

`4dd466d0e1e0cc123afbe504da2462ca`

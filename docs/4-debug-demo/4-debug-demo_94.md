

# 美萍网管大师

> 原文：[`www.pediy.com/kssd/pediy03/forum1151.htm`](https://www.pediy.com/kssd/pediy03/forum1151.htm)

用 W23DASM，查找字符串：regnum   
查到一个，不理它，再往下查。   
停住后，可以看到一个经典对比：   
\* Possible StringData Ref from Code Obj ->"RegNum"   
                                  |   
:00474455 B8FC444700              mov eax, 004744FC   
:0047445A E8DDF6FFFF              call 00473B3C   
:0047445F 3BD8                    cmp ebx, eax  <--呵呵。   
:00474461 750A                    jne 0047446D  <--只改这里为 9090 不行，后面还有暗桩。   
:00474463 B801000000              mov eax, 00000001   
:00474468 E983000000              jmp 004744F0   
  
用 CRACKCODE2000，写 CRACKCODE.INI 内容：   
\[Options\]   
CommandLine=smenu.exe   
Mode=1  //因为注册码是 16 进制放在内存，所以用 1   
First\_Break\_Address=47445f   
First\_Break\_Address\_Code=3B   
First\_Break\_Address\_Code\_Lenth=2   
Save\_Code\_Address=EBX   
  
搞定。   
  
不知道为什么美萍从 2.0 开始就一直用这种注册码对比方法？虽然美萍不断变换算法，对了，新版的网管大师学万象加入了“You are big pig”这种无聊的字符进入运算，你们猜是什么？是“TRW”“SICE”还有一个好象是“NTSICE”。但对比却一成不变，所以我一直用上面的方法破。不知道为什么美萍一直用 aspack1.07 压缩？可能他们不知道有 unaspack1091，呵。   
  
美萍检测 SICE 的方法是运行时检查 C:\\AUTOEXEC.BAT 里有没有 SICE 的命令行。我就备份了一个空白的 AUTOEXEC.BAT，装载了 SICE 后，备份原 AUTOEXEC.BAT，把空白的 AUTOEXEC.BAT 放在 C 盘下，再运行美萍啥事都没。检测 TRW 是检查 TRW 是否装载在内存。脱壳后用 ULTRAEDIT32 等软件查找字符串"TRW"，会找到两个，把前几个字符是 RAM 的那个改为 000 或其他，另一个是用来加入运算的，改不改无所谓。   
  
  
另：我是情狼。呵呵。  4dd466d0e1e0cc123afbe504da2462ca
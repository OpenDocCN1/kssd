
0048074F   .  8D95 A8FEFFFF lea edx,dword ptr ss:[ebp-158]

00480755   .  E8 C615F8FF   call <jmp.&rtl70.Sysutils::Trim>

0048075A   .  8B95 A8FEFFFF mov edx,dword ptr ss:[ebp-158]

00480760   .  8D85 ACFEFFFF lea eax,dword ptr ss:[ebp-154]

00480766   .  E8 CD0AF8FF   call <jmp.&rtl70.System::LStrFromWStr>

0048076B   .  8B85 ACFEFFFF mov eax,dword ptr ss:[ebp-154]

00480771   .  8D95 B0FEFFFF lea edx,dword ptr ss:[ebp-150]

00480777   .  E8 4415F8FF   call <jmp.&rtl70.Sysutils::UpperCase>

0048077C   .  8B85 B0FEFFFF mov eax,dword ptr ss:[ebp-150]

00480782   .  B9 09000000   mov ecx,9

00480787   .  BA 01000000   mov edx,1

0048078C   .  E8 EF0AF8FF   call <jmp.&rtl70.System::LStrCopy>  复制注册码前九位---"1234-5678"

00480791   .  8B85 B4FEFFFF mov eax,dword ptr ss:[ebp-14C]        把"1234-5678"放到EAX

00480797   .  8B55 F8       mov edx,dword ptr ss:[ebp-8]          把--堆栈 ss:[0012F3F8]=00C59EF4, (ASCII "2498-2498")--放到EDX

0048079A   .  E8 C10AF8FF   call <jmp.&rtl70.System::LStrCmp>     比较EAX，EDX中的字符串，相等则跳向成功！

0048079F   .  0F84 8A000000 je Registra.0048082F

我们这个时候就奇怪了，dword ptr ss:[ebp-8]中的数据是哪里来的呢？

我们想到开始时候对用户名处理的那一段代码中的可疑CALL，于是看到了：

```
00480562   .  8B85 E4FEFFFF mov eax,dword ptr ss:[ebp-11C]
00480568   .  8D95 E8FEFFFF lea edx,dword ptr ss:[ebp-118]
0048056E   .  E8 5517F8FF   call <jmp.&rtl70.Sysutils::LowerCase>
00480573   .  8B85 E8FEFFFF mov eax,dword ptr ss:[ebp-118]
00480579   .  8D55 F8       lea edx,dword ptr ss:[ebp-8]   <------哈哈，原来那里面的数据是从这里来的！
0048057C   .  E8 C7E7FFFF   call Registra.0047ED48             ; 对用户名进行一系列操作后，出现了这个CALL，很可疑啊
00480581   .  8D95 D4FEFFFF lea edx,dword ptr ss:[ebp-12C]
00480587   .  8B45 FC       mov eax,dword ptr ss:[ebp-4]
0048058A   .  8B80 A0030000 mov eax,dword ptr ds:[eax+3A0]
00480590   .  E8 EF1AF9FF   call Registra.00412084             ；这个也很可疑
00480595   .  8B85 D4FEFFFF mov eax,dword ptr ss:[ebp-12C]
```

进入第一个可疑CALL；来到

```
0047ED48  /$  55            push ebp
0047ED49  |.  8BEC          mov ebp,esp
0047ED4B  |.  B9 04000000   mov ecx,4
……
省略一段代码....
0047EDB6  |.  E8 2D24F8FF   call <jmp.&rtl70.System::LStrLAsg>
0047EDBB  |>  8D45 F8       lea eax,[local.2]
0047EDBE  |.  50            push eax
0047EDBF  |.  6A 00         push 0
0047EDC1  |.  8B45 FC       mov eax,[local.1]
0047EDC4  |.  50            push eax
0047EDC5  |.  E8 F6FEFFFF   call <jmp.&Routine.HandleDec>  此句后出现"2498A77F5BB5244F432173FBF065B7FF0"，这又是什么东东？注意到调用了Routine.HandleDec函数
……
```

在它的安装文件里看见了`Routine.dll`，PEID插件KANAL V2.9查到算法

`DES [pbox] [char] :: 0000296C :: 0040416C`

`MD5 :: 000017A9 :: 004023A9`

哈哈，那十有八九是计算了用户名的MD5,用自己的MD5计算器验证，果然是的！

跟着看`0047EDC5  |.  E8 F6FEFFFF   call <jmp.&Routine.HandleDec>`下面的代码：

```
0047EDCA  |.  8BD0          mov edx,eax
0047EDCC  |.  8D45 E4       lea eax,[local.7]
0047EDCF  |.  E8 3C24F8FF   call <jmp.&rtl70.System::LStrFromPChar>
0047EDD4  |.  8B45 E4       mov eax,[local.7]
0047EDD7  |.  B9 04000000   mov ecx,4
0047EDDC  |.  BA 01000000   mov edx,1
0047EDE1  |.  E8 9A24F8FF   call <jmp.&rtl70.System::LStrCopy>
0047EDE6  |.  8D45 F4       lea eax,[local.3]
0047EDE9  |.  50            push eax
0047EDEA  |.  6A 00         push 0
0047EDEC  |.  8B45 F0       mov eax,[local.4]
0047EDEF  |.  50            push eax
0047EDF0  |.  E8 CBFEFFFF   call <jmp.&Routine.HandleDec>
0047EDF5  |.  8BD0          mov edx,eax
0047EDF7  |.  8D45 E0       lea eax,[local.8]
0047EDFA  |.  E8 1124F8FF   call <jmp.&rtl70.System::LStrFromPChar>；所有字母转换为大写
0047EDFF  |.  8B45 E0       mov eax,[local.8]
0047EE02  |.  B9 04000000   mov ecx,4       ----System::LStrCopy的参数
0047EE07  |.  BA 01000000   mov edx,1       ----System::LStrCopy的参数
0047EE0C  |.  E8 6F24F8FF   call <jmp.&rtl70.System::LStrCopy> ；由参数易知，它复制了"2498A77F5BB5244F432173FBF065B7FF0"的前四位字符
0047EE11  |.  FF75 F8       push [local.2]
0047EE14  |.  68 88EE4700   push Registra.0047EE88                             ;  -  <------看到这个'-'了？
0047EE19  |.  FF75 F4       push [local.3]
0047EE1C  |.  8D45 DC       lea eax,[local.9]
0047EE1F  |.  BA 03000000   mov edx,3
0047EE24  |.  E8 2F24F8FF   call <jmp.&rtl70.System::LStrCatN>      将字符串前四位用'-'重复连接，即----> "2498-2498"
0047EE29  |.  8B45 DC       mov eax,[local.9]
0047EE2C  |.  8BD3          mov edx,ebx
0047EE2E  |.  E8 8D2EF8FF   call <jmp.&rtl70.Sysutils::UpperCase>  ;字母转换为大写
0047EE33  |.  33C0          xor eax,eax
```

到此，算法就分析完了。

## 算法总结

计算用户名的MD5，用'-'重复连接MD5前四位，即XXXX-XXXX的形式，后面10位注册码随便写。

注册机源码就不放上来了，除了MD5的代码难写之外，其他写出来很简单。

---

### 【经验总结】

对于初学者，我想说，不要退缩，分析软件时要细心，要有耐心，能持之以恒，那么你进步一定很快。

---

### 【版权声明】

转载请注明作者并保持文章的完整, 谢谢!

2012年08月03日 下午 11:15:41

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [Advanced SystemCare离线算法分析+注册机.rar](javascript:void(0);)

  （1.01MB，71次下载）

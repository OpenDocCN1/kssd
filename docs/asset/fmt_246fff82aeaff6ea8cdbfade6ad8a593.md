
## 汇编代码片段

```
0042D276    .   8945 BC                       mov dword ptr ss:[ebp-44],eax  
0042D279    .   C745 B4 08000000             mov dword ptr ss:[ebp-4C],8  

0042D280    .   8D45 B4                       lea eax,dword ptr ss:[ebp-4C]  

0042D283    .   50                            push eax  

0042D284    .   FF15 04114000                 call dword ptr ds:[401104]                   ;   MSVBVM60.rtcKillFiles  

0042D28A    .   8D4D B4                       lea ecx,dword ptr ss:[ebp-4C]  

0042D28D    .   FF15 1C104000                 call dword ptr ds:[40101C]                   ;   MSVBVM60.__vbaFreeVar  

0042D293    .   C745 FC 0B000000             mov dword ptr ss:[ebp-4],0B  

0042D29A    .   8B4D D8                       mov ecx,dword ptr ss:[ebp-28]  

0042D29D    .   51                            push ecx  

0042D29E    .   68 E08D4000                   push fsfd.00408DE0                            ;   UNICODE "\iwinada.dll"  

0042D2A3    .   FF15 70104000                 call dword ptr ds:[401070]                   ;   MSVBVM60.__vbaStrCat  

....................................................................................  
```

共两处，你可以用这个作品去讨好 MM 了，成了好事别忘记二哥我啊. ^\_^  

## 破解总结

### 启动校验

```
00437CCB      /74 09                          je short fsfd.00437CD6  
```

修改为

```
00437CCB       90                             nop  
00437CCC       90                             nop  
```

### 实时校验

```
0042D24E    . /74 76                          je short fsfd.0042D2C6  
```

修改为

```
0042D24E     /EB 76                          jmp short fsfd.0042D2C6  
```

2004 年 12 月 2 日 13 点 25 分完成文章，历时 7-8 小时。  

\--------------------------------------------------------------------------------   

## 版权声明 

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`

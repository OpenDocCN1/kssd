
```
00835FBD     33D2                xor edx,edx   

00835FBF     52                  push edx   

00835FC0     50                  push eax   

00835FC1     8D95 58FAFFFF       lea edx,dword ptr ss:[ebp-5A8]   

00835FC7     B8 08000000         mov eax,8   

00835FCC     E8 AF2FF6FF         call VP.00798F80   

00835FD1     8B8D 58FAFFFF       mov ecx,dword ptr ss:[ebp-5A8]   

00835FD7     8D85 5CFAFFFF       lea eax,dword ptr ss:[ebp-5A4]   

00835FDD     BA 946B8300         mov edx,VP.00836B94         ; ASCII "Finalizing 0x"   

00835FE2     E8 B5E8F5FF         call VP.0079489C   

00835FE7     8B85 5CFAFFFF       mov eax,dword ptr ss:[ebp-5A4]   

00835FED     E8 46BCFFFF         call VP.00831C38   

00835FF2     FF65 FC             jmp dword ptr ss:[ebp-4]   

                              ==== >飞向光明之巅！     :-)   

————————————————————————   

0066B508     55                  push ebp   

                              ==== >在这儿用 LordPE 完全 DUMP 这个进程   

0066B509     8BEC                mov ebp,esp   

0066B50B     83C4 F0             add esp,-10   

0066B50E     B8 D0AC6600         mov eax,VisualPr.0066ACD0   

0066B513     E8 E8BCD9FF         call VisualPr.00407200   

0066B518     A1 34D86700         mov eax,dword ptr ds:[67D834]   

0066B51D     8B00                mov eax,dword ptr ds:[eax]   

0066B51F     E8 301EE2FF         call VisualPr.0048D354   

0066B524     A1 34D86700         mov eax,dword ptr ds:[67D834]   

0066B529     8B00                mov eax,dword ptr ds:[eax]   

0066B52B     BA B0B56600         mov edx,VisualPr.0066B5B0     ; ASCII "Visual Protect"   

0066B530     E8 D318E2FF         call VisualPr.0048CE08   

————————————————————————
```

重新运行`VisualProtect.exe`，运行`ImportREC`，选择这个进程。 把`OEP`改为`0026B508`，点`IT AutoSearch`，

点“`Get Import`”，有许多函数无效，用“追踪层次 3”全部修复。`FixDump`，正常运行！ 1.95M -> 3.53M

脱壳后看是`Delphi`编写，可以跨系统平台运行。原程序是采用`Trial Limits`+`Keyfile`保护的，脱壳后即取消了 20 天的试用期限制。至于注册，`[TMG]`的`Judas Priest`大侠已经发布这个版本的注册机啦，`RC4`+`MD5`+`CRC`+`BlowFish`   呵呵，一大堆的密码学算法，牛！

其他采用`Visual Protect V3.54`加壳试用期保护的程序的脱壳方法和本文介绍的大体相同。

—————————————————————————————————     

```
      ,     \_/   

    /| \_.-~/          \_     ,        青春都一饷   

   ( /~   /             ~-. \ |   

   \`\\  \_/                ~ )          忍把浮名   

  _-~~~-.)  )\_\_/;;,.      \_  //'    

 /'\_,   --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂   

 \`~ _( ,\_..-- (     ,;'' /    ~--   /.\_\`   

  /~~//'   /' \`~           ) /--..\_, )\_  \`~    

  "  \`~"  "       \`"       /~'\`    \`\\~~    

                  "     "   "~'  ""   

          Cracked By 巢水工作坊——fly [OCN][FCG]   

                  2003-10-26  17:11
```

`4dd466d0e1e0cc123afbe504da2462ca`

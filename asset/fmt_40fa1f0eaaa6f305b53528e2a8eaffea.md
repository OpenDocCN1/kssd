
```
00BD6251   E8 3610FFFF      CALL 00BC728C   

00BD6256   50              PUSH EAX   

00BD6257   A1 BC7BBE00     MOV EAX,DWORD PTR DS:[BE7BBC]   

00BD625C   8B48 3C         MOV ECX,DWORD PTR DS:[EAX+3C]   

00BD625F   3348 24         XOR ECX,DWORD PTR DS:[EAX+24]   

00BD6262   3348 1C         XOR ECX,DWORD PTR DS:[EAX+1C]   

00BD6265   2BF9           SUB EDI,ECX   

00BD6267   FFD7           CALL EDI    //这里 F7，飞向光明之颠。    

00BD6269   8BD8           MOV EBX,EAX   

00BD626B   5F             POP EDI   

00BD626C   8BC3           MOV EAX,EBX   

00BD626E   5E             POP ESI   

00BD626F   5B             POP EBX   

00BD6270   C3             RETN
```

在那个 `CALL EDI` 处 F7 我们就到入口了，这时可以用 `LORDPE` 把主程序 `DUMP` 出来了。

四、修复 IAT

好了，运行 `Import1.6`，OEP 填 `145C`，所有的的函数都完好无缺，`FixDump` 它。好，运行一下，没有问题，然后再拿 `Lordpe` `rebuild` 一下，减小体积。

【后语】自退出破界那么久，再次回来时已经是今非昔比了，如今是长江后浪推前浪，技术不断的发展，我们这些老 `CRACKER` 也该歇歇了。

`4dd466d0e1e0cc123afbe504da2462ca`

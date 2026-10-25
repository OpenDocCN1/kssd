
```asm
004487C4   .  FF15 9CD45400 CALL DWORD PTR DS:[<&KERNEL32.SetFileAttributesA>]      ; SetFileAttributesA

004487CA   .  FF75 F0       PUSH DWORD PTR SS:[EBP-10]                              ; /FileName

004487CD   .  FF15 C0D45400 CALL DWORD PTR DS:[<&KERNEL32.DeleteFileA>]             ; 这里拦住了

004487D3   .  A1 48D15900   MOV EAX,DWORD PTR DS:[59D148]
```

找到了就好办了，从`NOP`掉`4487ca`和`4487cd`两行。

好了，现在再试试运行没什么问题了。

## 总结

注册信息：

Name：loveboom[DFCG]

SN：R0D3G-Y6A50-0106M-Y6R40

去暗桩：

```asm
004487CA  FF 75 F0 FF 15 C0 D4 54 00                       u? 涝T.HY
```

改为

```asm
004487CA  90 90 90 90 90 90 90 90 90                       HY.
```

献给我亲爱的DFCG，愿它能够蒸蒸日上!

Thanks:

Fly 辉仔yock、jingulong、二哥、所有曾经关心支持或帮助过我的朋友！谢谢您们!

by  loveboom[DFCG]

Email:bmd2chen@tom.com

2004-3-15

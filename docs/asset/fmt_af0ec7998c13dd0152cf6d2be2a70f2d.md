
# 代码分析记录

```
00BEAFAE   09C0             OR EAX,EAX   

00BEAFB0   74 07            JE SHORT 00BEAFB9   

00BEAFB2   8903             MOV DWORD PTR DS:\[EBX\],EAX   

00BEAFB4   83C3 04          ADD EBX,4   

00BEAFB7  ^EB E1             JMP SHORT 00BEAF9A   

00BEAFB9   FF96 5CE07E00    CALL DWORD PTR DS:\[ESI+7EE05C\]   

00BEAFBF   61               POPAD   

00BEAFC0  ^E9 3CE0D1FF       JMP 00909001  ==跟
```

去 `909001`，又是壳？口吐泡沫 ing...

```
00909001   60               PUSHAD   

00909002   E8 03000000      CALL 0090900A   

00909007  -E9 EB045D45      JMP 45ED94F7   

0090900C   55               PUSH EBP   

0090900D   C3               RETN
```

由于太烦了，直接了当，去后面找 `POPAD`，找到下面的地方，是不是和刚才第二层壳的尾部非常相似?!

对啦，连 `PUSHAD` 和 `POPAD` 的地址都一样：前面的是 `ecd001`\-`ecd3af`，这里是 `909001`-`9093af`，所以，和 `UPX` 的壳一样，非常好找啊:) 到了 `PUSHAD` 之后直接加上偏移量就行了。(UPX 是 `11c` 附近，ASPack 是 `3ae`)

```
009093AF   61               POPAD   

009093B0   75 08            JNZ SHORT 009093BA   

009093B2   B8 01000000      MOV EAX,1   

009093B7   C2 0C00          RETN 0C   

009093BA   68 C4B25F00      PUSH 5FB2C4   

009093BF   C3               RETN
```

在 `9093af` F4。F7 到 `RETN` 后来到 `5fb2c4`：

```
005FB2C4   55               PUSH EBP   

005FB2C5   8BEC             MOV EBP,ESP   

005FB2C7   83C4 F0          ADD ESP,-10   

005FB2CA   53               PUSH EBX   

005FB2CB   B8 B4AE5F00      MOV EAX,5FAEB4   

005FB2D0   E8 8FBCE0FF      CALL 00406F64   

005FB2D5   8B1D 3C696000     MOV EBX,DWORD PTR DS:\[60693C\]   

005FB2DB   8B0B             MOV ECX,DWORD PTR DS:\[EBX\]   

005FB2DD   B2 01            MOV DL,1   

005FB2DF   A1 F4A05500      MOV EAX,DWORD PTR DS:\[55A0F4\]   

005FB2E4   E8 CB4FE8FF      CALL 004802B4   

......
```

看样子应该是 OEP 吧...试下 dump 出来...

直接用 `OllyDump`(Dump 了好几分钟，以为死机了:(，看看网页先:) )，再来 `ImportREC`。oep->`1fb2c4`,get imports,fix dump. 10M 多.运行正常。

作者也算用心良苦了，`EXEStealth` `2.72`-`2.73` / `ASPack` / `UPX` / `ASPack` 加了 4 次壳。

=====================================================================

`4dd466d0e1e0cc123afbe504da2462ca`

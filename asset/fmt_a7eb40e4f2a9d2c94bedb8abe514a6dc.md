
```asm
:006D90B2 C78528FAFFFF03000000 mov dword ptr [ebp+FFFFFA28], 00000003
* Reference To: MSVBVM50.__vbaVarAdd, Ord:0000h
|
:006D90BC FF1570758800 Call dword ptr [00887570]
====>相加：03A1C214 + 0F0A6BED=12AC2E01
…… …… 省 略 …… ……
:006D9207 C78530FAFFFF0C000000 mov dword ptr [ebp+FFFFFA30], 0000000C
====>[ebp+FFFFFA30]=0000000C
:006D9211 C78528FAFFFF02000000 mov dword ptr [ebp+FFFFFA28], 00000002
* Reference To: MSVBVM50.__vbaVarXor, Ord:0000h
|
:006D921B FF15AC738800 Call dword ptr [008873AC]
====>异或：12AC2E01 XOR 0000000C=12AC2E0D
…… …… 省 略 …… ……
:006D9366 C78530FAFFFF07296601 mov dword ptr [ebp+FFFFFA30], 01662907
====>[ebp+FFFFFA30]=01662907
:006D9370 C78528FAFFFF03000000 mov dword ptr [ebp+FFFFFA28], 00000003
* Reference To: MSVBVM50.__vbaVarAdd, Ord:0000h
|
:006D937A FF1570758800 Call dword ptr [00887570]
====>相加：12AC2E0D + 01662907=14125714
…… …… 省 略 …… ……
:006D94C5 C78530FAFFFF08000000 mov dword ptr [ebp+FFFFFA30], 00000008
====>[ebp+FFFFFA30]=00000008
:006D94CF C78528FAFFFF02000000 mov dword ptr [ebp+FFFFFA28], 00000002
* Reference To: MSVBVM50.__vbaVarXor, Ord:0000h
|
:006D94D9 FF15AC738800 Call dword ptr [008873AC]
====>异或：14125714 XOR 00000008=1412571C
…… …… 省 略 …… ……
:006D9624 C78530FAFFFF92300000 mov dword ptr [ebp+FFFFFA30], 00003092
====>[ebp+FFFFFA30]=00003092
:006D962E C78528FAFFFF02000000 mov dword ptr [ebp+FFFFFA28], 00000002
* Reference To: MSVBVM50.__vbaVarSub, Ord:0000h
|
:006D9638 FF1514738800 Call dword ptr [00887314]
====>相减：1412571C - 00003092=1412268A
…… …… 省 略 …… ……
:006D9783 C78530FAFFFF9F050000 mov dword ptr [ebp+FFFFFA30], 0000059F
====>[ebp+FFFFFA30]=0000059F
:006D978D C78528FAFFFF02000000 mov dword ptr [ebp+FFFFFA28], 00000002
* Reference To: MSVBVM50.__vbaVarXor, Ord:0000h
|
:006D9797 FF15AC738800 Call dword ptr [008873AC]
====>异或：1412268A XOR 0000059F=14122315
…… …… 省 略 …… ……
:006D98E2 C78530FAFFFFD598EF01 mov dword ptr [ebp+FFFFFA30], 01EF98D5
====>[ebp+FFFFFA30]=01EF98D5
:006D98EC C78528FAFFFF03000000 mov dword ptr [ebp+FFFFFA28], 00000003
* Reference To: MSVBVM50.__vbaVarAdd, Ord:0000h
|
:006D98F6 FF1570758800 Call dword ptr [00887570]
====>相加：14122315 + 01EF98D5=1601BBEA
…… …… 省 略 …… ……
:006D9A41 C78530FAFFFFF8920300 mov dword ptr [ebp+FFFFFA30], 000392F8
====>[ebp+FFFFFA30]=000392F8
:006D9A4B C78528FAFFFF03000000 mov dword ptr [ebp+FFFFFA28], 00000003
* Reference To: MSVBVM50.__vbaVarXor, Ord:0000h
|
:006D9A55 FF15AC738800 Call dword ptr [008873AC]
====>异或：1601BBEA XOR 000392F8=16022912（H）=369240338（D）
…… …… 省 略 …… ……
* Reference To: MSVBVM50.__vbaStrVarVal, Ord:0000h
|
:006D9B1E FF15E4748800 Call dword ptr [008874E4]
:006D9B24 50 push eax
====>EAX=13572468 试炼码
…… …… 省 略 …… ……
* Reference To: MSVBVM50.rtcR8ValFromBstr, Ord:0245h
|
:006D9B4F FF15F0758800 Call dword ptr [008875F0]
:006D9B55 DD9D30FAFFFF fstp qword ptr [ebp+FFFFFA30]
====>ST=13572468.00000000
…… …… 省 略 …… ……
* Reference To: MSVBVM50.__vbaVarTstEq, Ord:0000h
|
:006D9BCB FF154C748800 Call dword ptr [0088744C]
====>比较 CALL！
:006D9BD1 6685C0 test ax, ax
:006D9BD4 0F850E4D0000 jne 006DE8E8
———————————————————

### 进入比较 CALL

`006D9BCB Call dword ptr [0088744C]`

```asm
78356F97 DB43 08 fild dword ptr ds:[ebx+8]
78356F9A 8B55 0C mov edx,dword ptr ss:[ebp+C]
78356F9D DC52 08 fcom qword ptr ds:[edx+8] //比较啦 :-)
```

- `ST=369240338.00000000000` 注册码
- `[edx+8]=13572468.00000000` 试炼码

```asm
78356FA0 DD5D F8 fstp qword ptr ss:[ebp-8]
78356FA3 DFE0 fstsw ax
78356FA5 9E sahf
78356FA6 72 1B jb short MSVBVM50.78356FC3
```

—————————————————————————————————

### 【算法 总 结】

哎，有必要放如此多而烦琐的 简单运算 吗？建议使用密码学算法吧！ :-)

—————————————————————————————————

### 【注册信息保存】

```reg
REGEDIT4
[HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\explorer\Advanced\BitAdv]
"NoDriveRun"="kat"
"NoVirtMemPage"="'\\' & $'',"
```

BTW: 程序自身的密码保存在：

```reg
[HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Check_Pss]
"pssword"=" 125 127 121 123"
```
忘记口令删了这个就行了。或者你自己求逆出来 ^O^

—————————————————————————————————

### 【整 理】

- 用户名：`fly`
- 注册码：`369240338`

—————————————————————————————————

```text
     ,     _/ 
    /| _. -~/           _      ,        青春都一饷
   ( /~   /             ~-. _ |
   \`\\   _/              ~ )          忍把浮名
  _-~~~-.)  )\_/;;,.       _  //'    
 /'\_,   --~    ~~~-  ,;;__\_(  (.-~~~-.        换了破解轻狂
 \`~ _( ,_..-- (    ,;''/    ~--  /.\_`  
  /~~//'  /' \`~       ) /--..\_, )_  \`~    
  "  \`~"  "       \`"      /~'\`    \`\\~~    

          "     "   "~'  ""
        Cracked By 巢水工作坊——fly [OCN][FCG]
              2003-09-24  16:55
4dd466d0e1e0cc123afbe504da2462ca
```

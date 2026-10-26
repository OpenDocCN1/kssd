
```
00536C30  |.  8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]
00536C33  |.  8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]
00536C36  |.  B9 05000000   MOV ECX,5
00536C3B  |.  E8 3CE1ECFF   CALL kxcd.00404D7C
00536C40  |.  8D45 E8       LEA EAX,DWORD PTR SS:[EBP-18]
00536C43  |.  8A55 FA       MOV DL,BYTE PTR SS:[EBP-6]
00536C46  |.  E8 79DDECFF   CALL kxcd.004049C4
00536C4B  |.  8B45 E8       MOV EAX,DWORD PTR SS:[EBP-18]
00536C4E  |.  8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]
00536C51  |.  B9 09000000   MOV ECX,9
00536C56  |.  E8 21E1ECFF   CALL kxcd.00404D7C
00536C5B  |.  8BC6          MOV EAX,ESI
00536C5D  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
00536C60  |.  E8 D3DBECFF   CALL kxcd.00404838
00536C65  |.  33C0          XOR EAX,EAX
00536C67  |.  5A            POP EDX
00536C68  |.  59            POP ECX
00536C69  |.  59            POP ECX
00536C6A  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
00536C6D  |.  68 8F6C5300   PUSH kxcd.00536C8F
00536C72  |>  8D45 E8       LEA EAX,DWORD PTR SS:[EBP-18]
00536C75  |.  BA 04000000   MOV EDX,4
00536C7A  |.  E8 89DBECFF   CALL kxcd.00404808
00536C7F  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
00536C82  |.  E8 5DDBECFF   CALL kxcd.004047E4
00536C87  \.  C3            RETN
00536C88   .^ E9 FBD3ECFF   JMP kxcd.00404088
00536C8D   .^ EB E3         JMP SHORT kxcd.00536C72
00536C8F   .  5E            POP ESI
--------------------------------------------------------------------------------
```

## 经验总结

我的假确定码 `123456` 注册码 `123`

1. 将确定码 `123456` 的16进制/BF 余数送 `DL=46` 在将 `46` 转化为10进制 `70` 记做 `X`
2. 将确定码 `123456` 的16进制 XOR `B25F1=` `AC7B1` 转化为10进制 `706481` 取它的第1和2位 16进制ASIIC相加/5 余数加34 转化为字符 `'7'`

再取它的第3和4位 16进制ASIIC相加/5 余数加33 转化为字符 `'4'` 然后 `706481` 与 `74` 相连得 `70648174` 记做 `Y`
3. `Y` 的16进制 xor `ddfb7687=` `D9CD77E9` 转化为10进制 `3654121449` 记做 `M`

取它的第1和2位 16进制ASIIC相加/5 余数加66 转化为字符 `'f'`
取它的第3和4位 16进制ASIIC相加/5 余数加75 转化为字符 `'u'`
取它的第5和6位 16进制ASIIC相加/5 余数加7a 转化为字符 `'~'`
取它的第7 8 9位 16进制ASIIC相加/5 余数加69 转化为字符 `'1'`

得到字符 `fu~1` 记做 `Z`
4. `Z` 与 `M` 适当组合后加 上 `X` 就是注册码 即 `fu~1` 与 `3654121449` 适当组合后加 上 `70` 就是注册码

注册码这样排列 `7036+'1'+54+'u'+412+'~f'+1449` 我的注册码就是 `7036l5u412~f1449`

## 版权声明

谢谢 PYG论坛的fonge大哥的热心指导 没有你 我也许离写出算法还很远 再次感谢 祝好人一生平安

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)


```asm
00408DD9  |.  B8 FBFFFFFF   MOV EAX,-5
00408DDE  |.  5B            POP EBX
00408DDF  |.  8B4C24 6C     MOV ECX,DWORD PTR SS:[ESP+6C]
00408DE3  |.  E8 C1DD0000   CALL xxxxx.00416BA9
00408DE8  |.  83C4 70       ADD ESP,70
00408DEB  |.  C2 1C00       RETN 1C
00408DEE  |>  8B4C24 78     MOV ECX,DWORD PTR SS:[ESP+78]
00408DF2  |.  5E            POP ESI
00408DF3  |.  5D            POP EBP
00408DF4  |.  B8 FDFFFFFF   MOV EAX,-3
00408DF9  |.  5B            POP EBX
00408DFA  |.  E8 AADD0000   CALL xxxxx.00416BA9
00408DFF  |.  83C4 70       ADD ESP,70
00408E02  \.  C2 1C00       RETN 1C
```

总结：此软件的加密与用户名字密码，和系统运行时间有关，解密的时候和用户名、密码无关但是与系统运行时间有关。

加密和解密算法，关键处我已经列出就不再描述了。

文章写得乱，希望各位别丢砖头，其中有很多漏的地方请大家指正。

---

【版权声明】本文纯属技术交流, 转载请注明作者信息并保持文章的完整, 谢谢!

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

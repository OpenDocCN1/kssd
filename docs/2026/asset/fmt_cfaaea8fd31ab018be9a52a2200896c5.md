
```
  00CE20DD  |.  6A 10         PUSH 10                                  ; /n = 10 (16.)
  00CE20DF  |.  8B45 BC       MOV EAX,DWORD PTR SS:[EBP-44]            ; |
  00CE20E2  |.  8B08          MOV ECX,DWORD PTR DS:[EAX]               ; |把后面的一句提到这里来，改成 LEA EDX,DWORD PTR SS:[EBP-54]
  00CE20E4  |.  51            PUSH ECX                                 ; |这里改成 MOV ECX,EDX
  00CE20E5  |.  8D55 AC       LEA EDX,DWORD PTR SS:[EBP-54]            ; |这里改成 PUSH ECX
  00CE20E8  |.  52            PUSH EDX                                 ; |s1
  00CE20E9  |.  E8 62120000   CALL <JMP.&MSVCRT.memcmp>                ; \memcmp
```

经过这样改后，可以随意输入注册码注册。

附件是脚本及 `MoleBox9x.v2.3682` 脱壳破解后的文件。

--------------------------------------------------------------------------------

【版权声明】: 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [`MoleBox 2.x Unpacker.rar`](javascript:void(0);)
  （3.06kb，884次下载）
- [`MoleBox9x.v2.3682.Cracked.rar`](javascript:void(0);)
  （450.33kb，539次下载）


```
00E9D428    56              PUSH    ESI

00E9D429    FF7424 20       PUSH    DWORD PTR SS:[ESP+0x20]

00E9D42D    5B              POP     EBX

00E9D42E    FF53 08         CALL    NEAR DWORD PTR DS:[EBX+0x8]     ; 合成64位地址

00E9D431    6A 00           PUSH    0x0

00E9D433    56              PUSH    ESI

00E9D434    FF7424 38       PUSH    DWORD PTR SS:[ESP+0x38]

00E9D438    FF13            CALL    NEAR DWORD PTR DS:[EBX]         ; 完成复制动作

00E9D43A    61              POPAD                                   ; 以下动作恢复堆栈弹掉没用的

00E9D43B    58              POP     EAX

00E9D43C    58              POP     EAX

00E9D43D    58              POP     EAX

00E9D43E    58              POP     EAX

00E9D43F    58              POP     EAX

00E9D440    58              POP     EAX

00E9D441    58              POP     EAX

00E9D442    61              POPAD                                   ; 还原寄存器

00E9D443    C645 FC 28      MOV     BYTE PTR SS:[EBP-0x4], 0x28     ; 原程序动作

00E9D447    C645 FC 26      MOV     BYTE PTR SS:[EBP-0x4], 0x26

00E9D44B  - E9 9059D7FF     JMP     Portable.00C12DE0               ; 回去执行

00E9D450    90              NOP
```

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

                                                       2014年03月09日 9:06:52

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

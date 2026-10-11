
00402522  |.  68>push    AUTHREG.0040CD80                  ;  ASCII "ILOVEYOU"

00402527  |.  89>mov     dword ptr ds:[40F840], ecx

0040252D  |.  E8>call    AUTHREG.00402560                  ;  这里将注册码保存到注册表中！

上面这个 `CALL` 将你的注册信息保存到注册表，各位请自己分析。

00402532  |.  8B>mov     ecx, dword ptr ss:[esp+4C]

00402536  |.  8B>mov     edx, dword ptr ds:[40CD64]        ;  AUTH936.00C90000

0040253C  |.  83>add     esp, 8

0040253F  |.  8D>lea     eax, dword ptr ss:[esp]

00402543  |.  50 push    eax                                ;  /lParam = NULL

00402544  |.  68>push    AUTHREG.004023E0                  ;  |DlgProc = AUTHREG.004023E0

00402549  |.  51 push    ecx                                ;  |hOwner = 800050B8

0040254A  |.  6A>push    69                                 ;  |pTemplate = 69

0040254C  |.  52 push    edx                                ;  |hInst = 80005140

0040254D  |.  FF>call    near dword ptr ds:[<&USER32.DialogBo>;  \DialogBoxParamA：这里将显示我们成功的信息！

00402553  |.  A1>mov     eax, dword ptr ds:[40F840]

00402558  |.  83>add     esp, 40

0040255B  \.  C3 retn                                ；成功返回

### 作战手册第五条：成功永远属于我们

你记住刚才的那段数字了吗？OK！按 `F9` 回到主对话框，点“继续试用”退出程序，然后再关闭 `olldbg`，

记住先后顺序不要搞反了，不然你很麻烦。

关闭你所有的程序，然后打开豪*超级解*V8 的程序，很自然又会弹出请你注册的信息，没关系我们已经

有了正确的注册码，输入吧！然后享受影音快乐吧！成功永远属于我们

### 总结：

关键要头脑冷静，认真分析和观察，了解常用 `API` 函数的使用途径、使用方法。

```\
******
*  注册名:ILOVEYOU          *
*  注册码：4321-lqh2-5ts1-535s   *
******
```

作者：`chenjiwl`

`chenjiwl@sina.com`

`4dd466d0e1e0cc123afbe504da2462ca`

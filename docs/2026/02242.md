
# post NOW! 破解过程！有意思。 (1千字)

破解软件：`Post NOW! 1.0 Beta 2 build 1219`

软件URL：`http://soft.km169.net/soft/wlgj/postnow1219.zip`

软件大小：643218 bytes

说明：该软件是个论坛发表软件，其刚开始未注册画面特烦人，那就注册啦！

呼出`TRW`，运用`'79'`大法，二拦后在此停下=> 

```
15F:0040A601    TEST EAX,EAX
...
15F:0040A61F    CMP EAX,BYTE +0C
15F:0040A622    JNZ NEAR 40A830    =>如果输入注册码不等于12位则注册不成功！
```

重新输入12位的注册码`'797979797979'`

```
15F:0040A6A9    MOV ESI,06
15F:0040A6AE    XOR EAX,EAX
15F:0040A6B0    MOV DL,[ESP+EAX+29]
15F:0040A6B4    MOV CL,[ESP+EAX+28]
15F:0040A6B8    MOV [ESP+EAX+28],DL
15F:0040A6BC    MOV [ESP+EAX+29],CL
15F:0040A6C0    INC EAX
15F:0040A6C1    CMP EAX,BYTE +06
15F:0040A6C4    JL 0040A6B0
15F:0040A6C6    DEC ESI
15F:0040A6C7    JNZ 0040A6AE
...
```

此段循环程序很有意思，如果你不耐烦，可以将光标移到下一栏，按`F9`，`F5`后跳过。

```
15F:0040A6E4    MOV BL,[ESP+EAX+228]
15F:0040A6EB    ADD DL,BL
15F:0040A6ED    INC EAX
15F:0040A6EE    CMP EAX,ECX
15F:0040A6F0    JL 0040A6E4    =>将coolfly的ASCII串累加到DL中
```

...（如果你要做注册机的话，这段程序是关键）我们这里略过！

```
15F:0040A76B    JL 0040A70A
```

在此下命令：`D ESP+328`，看到注册码是：`aecb8787948d`

还不赶快拿笔抄下来，OK，退出程序，输入刚刚看到的注册码，按确定后：

得到的还是注册失败框，嗯？？这时就要看看上面提到的那段有意思的程式了。

对，上面程式的作用是将注册码的第七位移到第一位，所以我们要做的是：

`aecb8787948d` => `ecb878a7948d` （反推而得）

这也是这个程式比较有意思的地方哦。

## [结论]

注册名：`coolfly`

注册码：`ecb878a7948d`

欢迎光临破解交流网: `http://coolfly.126.com`

破解花费10分钟，5分钟整理！

2000年12月30日 1:15

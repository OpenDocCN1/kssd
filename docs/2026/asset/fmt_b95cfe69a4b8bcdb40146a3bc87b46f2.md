
```
        0040392D    51              PUSH ECX

        0040392E    89D1            MOV ECX,EDX

        00403930    F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]

        00403932    FF50 1C         CALL DWORD PTR DS:[EAX+1C]  //WriteFile  每次写80h字节

        00403935    85C0            TEST EAX,EAX

        00403937    75 04           JNZ SHORT shed.0040393D

        00403939    59              POP ECX

        0040393A    58              POP EAX

        0040393B  ^ EB DA           JMP SHORT shed.00403917   //返回继续

        .........................

        00403947    0148 0C         ADD DWORD PTR DS:[EAX+C],ECX

        0040394A    F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]

        0040394C    5F              POP EDI

        0040394D    5E              POP ESI

       0040394E    C3              RETN
```

![](https://bbs.kanxue.com/upload/attach/200910/236638_7608q3qiq7nqbht.jpg)

![](https://bbs.kanxue.com/upload/attach/200910/236638_r5r3kraixotin40.jpg)

![](https://bbs.kanxue.com/upload/attach/200910/236638_so57z8dywyk29am.jpg)

脚本内容：

```
        var addr

        sto

        mov addr,esp       //ESP定律

        bphws addr,"r"

        run

        bphwc  addr        //清除硬件访问断点

        sto                //单步一下来到oep

        mov addr,eip

        and addr, 0f00000

        add addr,3900

        bp addr

        run                //来到00403900

        bc addr

        MSG "You get back it!"

        add addr,47

        bp addr

        run               //来到00403947

        bc addr

        gpa "ExitProcess","kernel32.dll"  //获取ExitProcess的地址

        mov eax,$RESULT

        asm eip, "push 0"

        asm eip+5, "call eax"

        run              //执行ExitProcess强行退出，这时得到的\*.exe.exe文件就是我们原来的程序。
```

### 四、总结

感染该病毒的机器必须将扩展分区和移动硬盘中的染毒文件全部进行修复处理，系统盘ghost恢复一下就安全了。这个病毒的可恶之处就是把我辛辛苦苦从看雪论坛下载的exe文件都打包了，还好没有对rar文件下手，不幸中的万幸。大家得小心啊！（附件：修复脚本和病毒样本）

天易love

2009-10-4

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [begin.JPG](javascript:void(0);)

  （66.73kb，620次下载）

- [end.JPG](javascript:void(0);)

  （56.98kb，616次下载）

- [jiaoben.txt](javascript:void(0);)

  （0.44kb，83次下载）

- [registry.JPG](javascript:void(0);)

  （106.78kb，618次下载）

- [script0.JPG](javascript:void(0);)

  （35.63kb，617次下载）

- [script1.JPG](javascript:void(0);)

  （52.16kb，614次下载）

- [script2.JPG](javascript:void(0);)

  （35.28kb，614次下载）

- [virusend.JPG](javascript:void(0);)

  （30.94kb，621次下载）

- [tougao.rar](javascript:void(0);)

  （84.58kb，73次下载）


# [原创]注册高速缓存管理器“Cacheman v5.1 ”

## 题目
注册高速缓存管理器“Cacheman v5.1 ”

## 软件功能
通过优化磁盘高速缓存，内存及其他设置来提高计算机性能。它能够阻止频繁的换页，从而提高系统性能，响应时间
和稳定性。适用于 Windows 95/98/ME/NT4/2000/XP。

## 工具
`Softice`,`PEid`

## 引子
今天晚上搞掉Dr.Salman's 系统工具后，接着乘胜追击，又搞掉这个小软件。因为比较简单，我也顺手把破文写了一下，算是工作日志吧。让大侠见笑了！拿PEID查看，是DELPHI开发的。不带壳，抓代码比较方便了。启动程序，点击help，单击Enter Custermer ID，在出现的小窗口内输入ID号，比如`111_22222222`，为什么这么个格式，下面告诉你答案！调出`SOFTICE`，下断点`bpx hmemcpy`，F5退出，然后点击OK，被拦截。7次F12来到程序空间，然后换F10跟踪到如下代码：

```
004BDABE  |. BA E0DB4B00    MOV EDX,CACHEMAN.004BDBE0                ; |ASCII "Please enter your
customer ID code:"
004BDAC3  |. B8 0CDC4B00    MOV EAX,CACHEMAN.004BDC0C                ; |ASCII "Cacheman"
004BDAC8  |. E8 33A0F9FF    CALL CACHEMAN.00457B00                   ; \CACHEMAN.00457B00
004BDACD  |. 837D FC 00     CMP DWORD PTR SS:[EBP-4],0      //我们返回到这里来。
004BDAD1  |. 0F84 BA000000  JE CACHEMAN.004BDB91            //此处未跳。
004BDAD7  |. 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]    //ID码地址送EAX。
004BDADA  |. E8 E10CFFFF    CALL CACHEMAN.004AE7C0          //这个CALL是关键，F8跟入。后面分析。（*）
004BDADF  |. 84C0           TEST AL,AL
004BDAE1  |. 0F84 88000000  JE CACHEMAN.004BDB6F
004BDAE7  |. A1 E45A4C00    MOV EAX,DWORD PTR DS:[4C5AE4]
004BDAEC  |. 83C0 14        ADD EAX,14
004BDAEF  |. 8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]
004BDAF2  |. E8 5D62F4FF    CALL CACHEMAN.00403D54
004BDAF7  |. 8D45 F8        LEA EAX,DWORD PTR SS:[EBP-8]
004BDAFA  |. 8B4D FC        MOV ECX,DWORD PTR SS:[EBP-4]
004BDB02  |. E8 C564F4FF    CALL CACHEMAN.00403FCC
004BDB07  |. 8B55 F8        MOV EDX,DWORD PTR SS:[EBP-8]             ;  KERNEL32.BFF79138
004BDB0A  |. 8B83 64060000  MOV EAX,DWORD PTR DS:[EBX+664]
004BDB10  |. E8 3BA3F8FF    CALL CACHEMAN.00447E50
004BDB15  |. B2 01          MOV DL,1
004BDB17  |. 8B83 68060000  MOV EAX,DWORD PTR DS:[EBX+668]
004BDB1D  |. E8 16A2F8FF    CALL CACHEMAN.00447D38
004BDB22  |. 8B83 FC050000  MOV EAX,DWORD PTR DS:[EBX+5FC]
004BDB28  |. 83C0 70        ADD EAX,70
004BDB2B  |. 8B4D FC        MOV ECX,DWORD PTR SS:[EBP-4]
004BDB2E  |. BA 38DC4B00    MOV EDX,CACHEMAN.004BDC38                ;  ASCII "mailto:cacheman-
support@outertech.com?subject=Cacheman Support Request, Customer ID: "
004BDB33  |. E8 9464F4FF    CALL CACHEMAN.00403FCC
004BDB38  |. A1 E45A4C00    MOV EAX,DWORD PTR DS:[4C5AE4]
004BDB3D  |. 8B40 14        MOV EAX,DWORD PTR DS:[EAX+14]            ;  CACHEMAN.004C1004
004BDB40  |. 50             PUSH EAX                                 ;  CACHEMAN.<ModuleEntryPoint>
004BDB41  |. A1 CC5A4C00    MOV EAX,DWORD PTR DS:[4C5ACC]
004BDB46  |. 8B00           MOV EAX,DWORD PTR DS:[EAX]
004BDB48  |. B9 98DC4B00    MOV ECX,CACHEMAN.004BDC98                ;  ASCII "CustomerID"
004BDB4D  |. BA ACDC4B00    MOV EDX,CACHEMAN.004BDCAC                ;  ASCII "Information"
004BDB52  |. 8B18           MOV EBX,DWORD PTR DS:[EAX]
004BDB54  |. FF53 04        CALL DWORD PTR DS:[EBX+4]
004BDB57  |. A1 CC5A4C00    MOV EAX,DWORD PTR DS:[4C5ACC]
004BDB5C  |. 8B00           MOV EAX,DWORD PTR DS:[EAX]
004BDB5E  |. 8B10           MOV EDX,DWORD PTR DS:[EAX]
004BDB60  |. FF52 4C        CALL DWORD PTR DS:[EDX+4C]
004BDB63  |. B8 C0DC4B00    MOV EAX,CACHEMAN.004BDCC0                ;  ASCII "Thank you for
registering. The customer ID will be shown in the support section.
When launching the "Send support request" function from the help menu it will be
automatically inserted to the subject of the mail!"
004BDB68  |. E8 779CF9FF    CALL CACHEMAN.004577E4
004BDB6D  |. EB 22          JMP SHORT CACHEMAN.004BDB91
004BDB6F  |> 68 A0DD4B00    PUSH CACHEMAN.004BDDA0
004BDB74  |. FF75 FC        PUSH DWORD PTR SS:[EBP-4]
004BDB77  |. 68 ACDD4B00    PUSH CACHEMAN.004BDDAC                   ;  ASCII "" is not a valid
Customer ID!"
004BDB7C  |. 8D45 F4        LEA EAX,DWORD PTR SS:[EBP-C]
004BDB7F  |. BA 03000000    MOV EDX,3
004BDB84  |. E8 B764F4FF    CALL CACHEMAN.00404040
004BDB89  |. 8B45 F4        MOV EAX,DWORD PTR SS:[EBP-C]             ;  KERNEL32.BFFC0D90
004BDB8C  |. E8 539CF9FF    CALL CACHEMAN.004577E4
004BDB91  |> 33C0           XOR EAX,EAX                              ;  CACHEMAN.<ModuleEntryPoint>
004BDB93  |. 5A             POP EDX                                  ;  KERNEL32.BFF8B86C
004BDB94  |. 59             POP ECX                                  ;  KERNEL32.BFF8B86C
004BDB95  |. 59             POP ECX                                  ;  KERNEL32.BFF8B86C
004BDB96  |. 64:8910        MOV DWORD PTR FS:[EAX],EDX
004BDB99  |. 68 B3DB4B00    PUSH CACHEMAN.004BDBB3
004BDB9E  |> 8D45 F4        LEA EAX,DWORD PTR SS:[EBP-C]
004BDBA1  |. BA 03000000    MOV EDX,3
004BDBA6  |. E8 7961F4FF    CALL CACHEMAN.00403D24
004BDBAB  \. C3             RETN
```

======================================================================

下面是`004BDADA  CALL CACHEMAN.004AE7C0`代码分析：      （*）

```
004AE7C0  /$ 55             PUSH EBP
004AE7C1  |. 8BEC           MOV EBP,ESP
004AE7C3  |. 6A 00          PUSH 0
004AE7C5  |. 6A 00          PUSH 0
004AE7C7  |. 6A 00          PUSH 0
004AE7C9  |. 53             PUSH EBX
004AE7CA  |. 8945 FC        MOV DWORD PTR SS:[EBP-4],EAX
004AE7CD  |. 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
004AE7D0  |. E8 5F59F5FF    CALL CACHEMAN.00404134
004AE7D5  |. 33C0           XOR EAX,EAX
004AE7D7  |. 55             PUSH EBP
004AE7D8  |. 68 6AE84A00    PUSH CACHEMAN.004AE86A
004AE7DD  |. 64:FF30        PUSH DWORD PTR FS:[EAX]
004AE7E0  |. 64:8920        MOV DWORD PTR FS:[EAX],ESP
004AE7E3  |. 33DB           XOR EBX,EBX
004AE7E5  |. 8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]
004AE7E8  |. B8 80E84A00    MOV EAX,CACHEMAN.004AE880
004AE7ED  |. E8 7A5AF5FF    CALL CACHEMAN.0040426C          //这个函数是判断前4位ID号最后一位是否是下划线
\_。后面分析。（**）
004AE7F2  |. 83F8 04        CMP EAX,4                       //如果ID码第四位是下划线，则返回4。
004AE7F5  |. 75 58          JNZ SHORT CACHEMAN.004AE84F     //此处就不跳。
004AE7F7  |. 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
004AE7FA  |. E8 8157F5FF    CALL CACHEMAN.00403F80          //判断ID总长是否为C，即12位。
004AE7FF  |. 83F8 0C        CMP EAX,0C
004AE802  |. 75 4B          JNZ SHORT CACHEMAN.004AE84F    //如果是12位则不跳。否则OVER。
004AE804  |. 8D4D F4        LEA ECX,DWORD PTR SS:[EBP-C]
004AE807  |. BA 03000000    MOV EDX,3
004AE80C  |. 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
004AE80F  |. E8 F8240100    CALL CACHEMAN.004C0D0C
004AE814  |. 8B45 F4        MOV EAX,DWORD PTR SS:[EBP-C]
004AE817  |. 33D2           XOR EDX,EDX
004AE819  |. E8 6EA4F5FF    CALL CACHEMAN.00408C8C           //这个函数处理前3位ID码。后面分析（***）。
004AE81E  |. 8BD8           MOV EBX,EAX                      //EBX保存了前3位注册码的16进制值。后面用于比
较。
004AE820  |. 8D4D F8        LEA ECX,DWORD PTR SS:[EBP-8]
004AE823  |. BA 08000000    MOV EDX,8
004AE828  |. 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
004AE82B  |. E8 34250100    CALL CACHEMAN.004C0D64           // 把下划线'\_'后面的8位字符复制到另外地方去
。
004AE830  |. 8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]     //把后8位ID码地址送EAX。
004AE833  |. E8 4857F5FF    CALL CACHEMAN.00403F80
004AE838  |. 8B55 F8        MOV EDX,DWORD PTR SS:[EBP-8]
004AE83B  |. 0FB64402 FF    MOVZX EAX,BYTE PTR DS:[EDX+EAX-1] //取8位ID码的最后一位送EAX。
004AE840  |. 03C0           ADD EAX,EAX                                   //EAX=EAX+EAX。
004AE842  |. 8D0440         LEA EAX,DWORD PTR DS:[EAX+EAX*2]  //EAX=EAX*3。
004AE845  |. 83F0 14        XOR EAX,14                        //EAX与14异或运算。
004AE848  |. 3BC3           CMP EAX,EBX                       //EBX内放的是前面计算得到的16进制数，与刚刚
计算得到的EAX比较。相等则OK。
004AE84A  |. 0F94C0         SETE AL                           //相等则置AL为1。
004AE84D  |. 8BD8           MOV EBX,EAX
004AE84F  |> 33C0           XOR EAX,EAX
004AE851  |. 5A             POP EDX
004AE852  |. 59             POP ECX
004AE853  |. 59             POP ECX
004AE854  |. 64:8910        MOV DWORD PTR FS:[EAX],EDX
004AE857  |. 68 71E84A00    PUSH CACHEMAN.004AE871
```

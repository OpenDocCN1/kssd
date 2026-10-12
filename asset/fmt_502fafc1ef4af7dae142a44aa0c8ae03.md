
`0043ADA8   |. 51               PUSH ECX`  
`0043ADA9   |. 8B4D 08          MOV ECX,DWORD PTR SS:[EBP+8]`        //后面几个函数无用。  
`0043ADAC   |. E8 5E200400      CALL QUIKSYNC.0047CE0F`  
`0043ADB1   |. 8B55 A0          MOV EDX,DWORD PTR SS:[EBP-60]`  
`0043ADB4   |. 83CA 01          OR EDX,1`  
`0043ADB7   |. 8955 A0          MOV DWORD PTR SS:[EBP-60],EDX`  
`0043ADBA   |. C645 FC 00       MOV BYTE PTR SS:[EBP-4],0`  
`0043ADBE   |. 8D4D AC          LEA ECX,DWORD PTR SS:[EBP-54]`  
`0043ADC1   |. E8 D4220400      CALL QUIKSYNC.0047D09A`  
`0043ADC6   |. 8B45 08          MOV EAX,DWORD PTR SS:[EBP+8]`  
`0043ADC9   |. 8B4D F4          MOV ECX,DWORD PTR SS:[EBP-C]`  
`0043ADCC   |. 64:890D 000000> MOV DWORD PTR FS:[0],ECX`  
`0043ADD3   |. 8BE5             MOV ESP,EBP`  
`0043ADD5   |. 5D               POP EBP`  
`0043ADD6   \. C2 0800          RETN 8`  

`============================================================================`  

下面分析（4）处的函数，这个函数通过计算得到一个数。  

`0043C270  /$ 55               PUSH EBP`  
`0043C271  |. 8BEC             MOV EBP,ESP`  
`0043C273  |. 51               PUSH ECX`  
`0043C274  |. 894D FC          MOV DWORD PTR SS:[EBP-4],ECX`  
`0043C277  |. 8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]`  
`0043C27A  |. 8B08             MOV ECX,DWORD PTR DS:[EAX]`        //上次计算结果送 ECX。  
`0043C27C  |. 69C9 4DBD2000    IMUL ECX,ECX,20BD4D`               //ECX=ECX*20BD4D。  
`0043C282  |. 81C1 C39E2600    ADD ECX,269EC3`                    //ECX=ECX+269EC3 。  
`0043C288  |. 8B55 FC          MOV EDX,DWORD PTR SS:[EBP-4]`  
`0043C28B  |. 890A             MOV DWORD PTR DS:[EDX],ECX`        //ECX 保存到 EDX 指定的地址。  
`0043C28D  |. 8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]`  
`0043C290  |. 8B00             MOV EAX,DWORD PTR DS:[EAX]`        //把刚才的结果送 EAX。  
`0043C292  |. C1F8 10          SAR EAX,10`                         //EAX 右移 10h 次，即 16 位。  
`0043C295  |. 25 FF7F0000      AND EAX,7FFF`                      //EAX 和 7FFF 进行与运算，取最低 15 位。  
`0043C29A  |. 8BE5             MOV ESP,EBP`  
`0043C29C  |. 5D               POP EBP`  
`0043C29D  \. C3               RETN`  

`============================================================================`  

下面分析（2）处的函数，这个函数对注册码进行比较。  

`00409650  /$ 55               PUSH EBP`  
`00409651  |. 8BEC             MOV EBP,ESP`  
`00409653  |. 8B4D 0C          MOV ECX,DWORD PTR SS:[EBP+C]`  
`00409656  |. E8 552E0000      CALL QUIKSYNC.0040C4B0`  
`0040965B  |. 50               PUSH EAX`  
`0040965C  |. 8B4D 08          MOV ECX,DWORD PTR SS:[EBP+8]`  
`0040965F  |. E8 7CD2FFFF      CALL QUIKSYNC.004068E0`            //这个函数通往真正比较的地方。（5）  
`00409664  |. F7D8             NEG EAX`  
`00409666  |. 1BC0             SBB EAX,EAX`  
`00409668  |. 40               INC EAX`  
`00409669  |. 5D               POP EBP`  
`0040966A  \. C2 0800          RETN 8`  

`============================================================================`  

下面分析（5）处的函数，这个函数对注册码进行比较。  

`004068E0  /$ 55               PUSH EBP`  
`004068E1  |. 8BEC             MOV EBP,ESP`  
`004068E3  |. 51               PUSH ECX`  
`004068E4  |. 894D FC          MOV DWORD PTR SS:[EBP-4],ECX`  
`004068E7  |. 8B45 08          MOV EAX,DWORD PTR SS:[EBP+8]`  
`004068EA  |. 50               PUSH EAX`  
`004068EB  |. 8B4D FC          MOV ECX,DWORD PTR SS:[EBP-4]`  
`004068EE  |. 8B11             MOV EDX,DWORD PTR DS:[ECX]`  
`004068F0  |. 52               PUSH EDX`  
`004068F1  |. E8 0A000000      CALL QUIKSYNC.00406900`            //在这个函数里进行。             （6）  
`004068F6  |. 83C4 08          ADD ESP,8`  
`004068F9  |. 8BE5             MOV ESP,EBP`  
`004068FB  |. 5D               POP EBP`  
`004068FC  \. C2 0400          RETN 4`  

（6）处函数调用下面代码。  

`00406900  /$ 55               PUSH EBP`  
`00406901  |. 8BEC             MOV EBP,ESP`  
`00406903  |. 8B45 0C          MOV EAX,DWORD PTR SS:[EBP+C]`  
`00406906  |. 50               PUSH EAX`  
`00406907  |. 8B4D 08          MOV ECX,DWORD PTR SS:[EBP+8]`  
`0040690A  |. 51               PUSH ECX`  
`0040690B  |. E8 308B0600      CALL QUIKSYNC.0046F440`            //我们找的"兔子"在这里藏着啦。   （7）  
`00406910  |. 83C4 08          ADD ESP,8`  
`00406913  |. 5D               POP EBP`  
`00406914  \. C3               RETN`  

`============================================================================`  

下面分析（7）处的函数，这个函数对注册码进行比较。  

`*省去多行*`  

`0046F46F  |> 66:0FB60F        /MOVZX CX,BYTE PTR DS:[EDI]`    //假码字符依次送 CX。  
`0046F473  |. 0FB6C1           |MOVZX EAX,CL`                  //CL 送 EAX。  
`0046F476  |. 47               |INC EDI`                       //EDI 地址增 1。  
`0046F477  |. 894D 0C          |MOV DWORD PTR SS:[EBP+C],ECX`  //把假码送内存单元保存。  
`0046F47A  |. F680 C1354B00 >| TEST BYTE PTR DS:[EAX+4B35C1],4`  
`0046F481  |. 74 16            |JE SHORT QUIKSYNC.0046F499`  
`0046F483  |. 8A07             |MOV AL,BYTE PTR DS:[EDI]`      //取 0 送 AL。  
`0046F485  |. 84C0             |TEST AL,AL`  
`0046F487  |. 75 06            |JNZ SHORT QUIKSYNC.0046F48F`   //如果为 0 则不跳。  
`0046F489  |. 8365 0C 00       |AND DWORD PTR SS:[EBP+C],0`  
`0046F48D  |. EB 0A            |JMP SHORT QUIKSYNC.0046F499`   //这里跳。  
`0046F48F  |> 33D2             |XOR EDX,EDX`  
`0046F491  |. 47               |INC EDI`  
`0046F492  |. 8AF1             |MOV DH,CL`  
`0046F494  |. 8AD0             |MOV DL,AL`  
`0046F496  |. 8955 0C          |MOV DWORD PTR SS:[EBP+C],EDX`  
`0046F499  |> 66:0FB61E        |MOVZX BX,BYTE PTR DS:[ESI]`    //真码送 BX。  
`0046F49D  |. 0FB6C3           |MOVZX EAX,BL`                  //然后送 EAX。  
`0046F4A0  |. 46               |INC ESI`                       //ESI 增 1。  
`0046F4A1  |. F680 C1354B00 >| TEST BYTE PTR DS:[EAX+4B35C1],4`  //EAX 作为偏移地址取一个数与 4“与”运算。  
`0046F4A8  |. 74 13            |JE SHORT QUIKSYNC.0046F4BD`     //这里自动跳。  
`0046F4AA  |. 8A06             |MOV AL,BYTE PTR DS:[ESI]`  
`0046F4AC  |. 84C0             |TEST AL,AL`  
`0046F4AE  |. 75 04            |JNZ SHORT QUIKSYNC.0046F4B4`  
`0046F4B0  |. 33DB             |XOR EBX,EBX`  
`0046F4B2  |. EB 09            |JMP SHORT QUIKSYNC.0046F4BD`  
`0046F4B4  |> 33C9             |XOR ECX,ECX`  
`0046F4B6  |. 46               |INC ESI`  
`0046F4B7  |. 8AEB             |MOV CH,BL`  
`0046F4B9  |. 8AC8             |MOV CL,AL`  
`0046F4BB  |. 8BD9             |MOV EBX,ECX`  
`0046F4BD  |> 66:395D 0C       |CMP WORD PTR SS:[EBP+C],BX`    //[EBP+C]里的假码跟 BX 内的真码比较。  
`0046F4C1  |. 75 09            |JNZ SHORT QUIKSYNC.0046F4CC`   //不相等就 OVER。  
`0046F4C3  |. 66:837D 0C 00    |CMP WORD PTR SS:[EBP+C],0`     //假码与 0 比较。  
`0046F4C8  |. 74 16            |JE SHORT QUIKSYNC.0046F4E0`    //相等时则结束。  
`0046F4CA  |.^EB A3            \JMP SHORT QUIKSYNC.0046F46F`   //否则继续循环。  
`0046F4CC  |> 6A 19            PUSH 19`  
`0046F4CE  |. E8 283C0000      CALL QUIKSYNC.004730FB`  
`0046F4D3  |. 66:3B5D 0C       CMP BX,WORD PTR SS:[EBP+C]`  
`0046F4D7  |. 59               POP ECX`  
`0046F4D8  |. 1BC0             SBB EAX,EAX`  
`0046F4DA  |. 83E0 02          AND EAX,2`  
`0046F4DD  |. 48               DEC EAX`  
`0046F4DE  |. EB 0A            JMP SHORT QUIKSYNC.0046F4EA`  
`0046F4E0  |> 6A 19            PUSH 19`  
`0046F4E2  |. E8 143C0000      CALL QUIKSYNC.004730FB`  
`0046F4E7  |. 59               POP ECX`  
`0046F4E8  |. 33C0             XOR EAX,EAX`          //EAX 清 0。  
`0046F4EA  |> 5F               POP EDI`  
`0046F4EB  |. 5E               POP ESI`  
`0046F4EC  |. 5B               POP EBX`  
`0046F4ED  |. 5D               POP EBP`  
`0046F4EE  \. C3               RETN`  

`============================================================================`  

后记：

通过 30 分钟的跟踪分析出注册码产生机制了。但是没有时间写出注册机，原理搞明白了，写注册机也是水到渠成的事情了。写这篇破文却花了俺 1 个小时的时间呢。为了给广大坛友提供一点帮助，助坛友一臂之力，再苦再累也值得了！因为身后有许多坛友给予我极大支持和关注！所以，我的每篇破文里面应该都包含了看雪论坛各位坛友的无形的帮助，也有你的一份功劳在里面的！如果没有各位的支持，相信我是坚持不下来的。非常高兴跟大家分享我的快乐！

结论：随便给出几组注册码，注册码的产生只与前 4 位有关：

```
1111t-lWRRF-eRKxH-546nm   （第 6 位是小写的 L）
7878A-4j8PF-TlSzl-8dZDe   （第 2 段内第 12、15 位是小写的 L，不是数字 1.）
5211z-tUYrx-p2xv7-1Ia5o   (最后一位不是数字 0，而是小写的字符 O)
```

注册成功后在 `H.C.U\Software\Iomega QuickSync 3\Sync` 键下面建立 `Init` 子键，里面是注册码的 16 进制明码形式。

鸣谢：感谢看雪给予本菜鸟的宽容帮助和支持！感谢各位关心我的破文的坛友们！祝愿各位在看雪论坛里迅速成长！

`qduwg`

`qduwg@163.com`

`2006 年 2 月 2 日晚完稿`

`4dd466d0e1e0cc123afbe504da2462ca`

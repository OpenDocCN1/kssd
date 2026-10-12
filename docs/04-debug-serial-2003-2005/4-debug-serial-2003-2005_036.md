
# 用易图书馆管理系统 2003 V3.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50648.htm`](https://www.pediy.com/kssd/pediy05/pediy50648.htm)

简单算法——用易图书馆管理系统 2003 V3.0

下载页面： [`www.skycn.com/soft/10881.html`](http://www.skycn.com/soft/10881.html)

软件大小： 5486 KB

软件语言： 简体中文

软件类别： 国产软件 / 共享版 / 租借管理

应用平台： `Win9x/NT/2000/XP`

加入时间： 2003-04-06 15:09:03

下载次数： 2798

推荐等级： \*\*\*\*

开发商： [`www.ueasysoft.com/`](http://www.ueasysoft.com/)

【软件简介】：用易图书馆管理系统 2003 V2.0 是一款适合中小型图书馆和单位、学校图书馆使用的管理软件。此软件功能强大，界面美观，容易上手。包括书库管理、读者管理两个基本模块；图书类别设置、借出天数设置、借出册数设置、超期罚款和复杂罚款设置；借书操作、续借操作和还书操作；功能强大的书库查询、读者查询、借阅信息查询、还书信息查询和还书罚款统计并提供模糊查询等功能。并自动显示今日到期读者和已经超期的读者，自动记费等功能。今日盘点能帮助您很快的统计出今日所进图书以及数量和价格、今日借出图书、今日归还图书、今日登记读者和今日罚款汇总。此版本又新增加了报表管理，及时报表显示全部图书、在馆图书和已经借出去的图书，并可直接打印并保存为 `HTML` 文件或 `TXT` 文件。同时，又增加了数据库发送功能，您可将数据表分别发到 `Word` 和 `Excel` 当中，以便您能更清楚的了解书库表和读者登记情况，并快速打印。为保证您的系统能安全使用，还增加了管理员功能，每个管理员可分别设置自己的管理权限和登录密码。本系统功能详尽，是中小图书馆和单位、学校图书馆的理想选择。

【软件限制】：功能限制

【作者声明】：初学 `Crack`，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`W32Dasm` 9.0 白金版

---

【过程】：

呵呵，又是 `VB` 的东东，好象这段日子和 `VB` 比较有缘呀。^O^^O^

哎，朋友见我太痴狂 `CRACK`，拿了《花样年华》来帮我“转移视线”，（我居然还没看过此片），统统把电影复制入硬盘后我就又开始了我的 `CRACK`，呵呵，不知那天会有空闲看了，有点对不住朋友的好心了。^O^^O^

`library.exe` 无壳。`Visual Basic 6.0` 编写。

信息码：`5KA089DJ`

试炼码：`13572468`

---

```
* Reference To: MSVBVM60.__vbaHResultCheckObj, Ord:0000h
                         |
:004FCB04 FF156C104000           Call dword ptr [0040106C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004FCAF9(C)
|
:004FCB0A 8B45D8                 mov eax, dword ptr [ebp-28]
                                  ==== >EAX=5KA089DJ          信息码
:004FCB0D 8D4DA8                 lea ecx, dword ptr [ebp-58]
:004FCB10 8945C0                 mov dword ptr [ebp-40], eax
:004FCB13 8D45B8                 lea eax, dword ptr [ebp-48]
:004FCB16 50                     push eax
:004FCB17 51                     push ecx
:004FCB18 8975D8                 mov dword ptr [ebp-28], esi
:004FCB1B C745B808000000         mov [ebp-48], 00000008

* Reference To: MSVBVM60.rtcTrimVar, Ord:0208h
                         |
:004FCB22 FF15BC104000           Call dword ptr [004010BC]
:004FCB28 8D55A8                 lea edx, dword ptr [ebp-58]
:004FCB2B 52                     push edx

* Reference To: MSVBVM60.__vbaStrVarMove, Ord:0000h
                         |
:004FCB2C FF1528104000           Call dword ptr [00401028]
:004FCB32 8BD0                   mov edx, eax
                                  ==== >EDX=EAX=5KA089DJ
:004FCB34 8D4DDC                 lea ecx, dword ptr [ebp-24]

* Reference To: MSVBVM60.__vbaStrMove, Ord:0000h
                         |
:004FCB37 FF1508124000           Call dword ptr [00401208]
:004FCB3D 8D4DD0                 lea ecx, dword ptr [ebp-30]

* Reference To: MSVBVM60.__vbaFreeObj, Ord:0000h
                         |
:004FCB40 FF1528124000           Call dword ptr [00401228]
:004FCB46 8D45A8                 lea eax, dword ptr [ebp-58]
:004FCB49 8D4DB8                 lea ecx, dword ptr [ebp-48]
:004FCB4C 50                     push eax
:004FCB4D 51                     push ecx
:004FCB4E 6A02                   push 00000002

* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h
                         |
:004FCB50 FF1534104000           Call dword ptr [00401034]
:004FCB56 8B55DC                 mov edx, dword ptr [ebp-24]
                                  ==== >EDX=5KA089DJ
:004FCB59 83C40C                 add esp, 0000000C
:004FCB5C 52                     push edx

* Reference To: MSVBVM60.__vbaLenBstr, Ord:0000h
                         |
:004FCB5D FF152C104000           Call dword ptr [0040102C]
                                  ==== >取 5KA089DJ 的长度
:004FCB63 8BC8                   mov ecx, eax
                                  ==== >ECX=EAX=8
:004FCB65 83E901                 sub ecx, 00000001
:004FCB68 0F80A00C0000           jo 004FD80E

* Reference To: MSVBVM60.__vbaI2I4, Ord:0000h
                         |
:004FCB6E FF1504114000           Call dword ptr [00401104]

* Reference To: MSVBVM60.rtcMidCharVar, Ord:0278h
                                  ==== >依次取信息码的字符，根据是否是数字而进行不同的处理！
:004FCB74 8B3DDC104000           mov edi, dword ptr [004010DC]
:004FCB7A 8985E8FDFFFF           mov dword ptr [ebp+FFFFFDE8], eax
:004FCB80 BB08400000             mov ebx, 00004008

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004FD099(U)
|
:004FCB85 663BB5E8FDFFFF         cmp si, word ptr [ebp+FFFFFDE8]
:004FCB8C 0F8F0C050000           jg 004FD09E
:004FCB92 668BD6                 mov dx, si
:004FCB95 8D45DC                 lea eax, dword ptr [ebp-24]
:004FCB98 6683C201               add dx, 0001
:004FCB9C 8985D0FEFFFF           mov dword ptr [ebp+FFFFFED0], eax
:004FCBA2 0F80660C0000           jo 004FD80E
:004FCBA8 8D4DB8                 lea ecx, dword ptr [ebp-48]
:004FCBAB C745C001000000         mov [ebp-40], 00000001
:004FCBB2 0FBFC2                 movsx eax, dx
:004FCBB5 51                     push ecx
:004FCBB6 8D8DC8FEFFFF           lea ecx, dword ptr [ebp+FFFFFEC8]
:004FCBBC 50                     push eax
:004FCBBD 8D55A8                 lea edx, dword ptr [ebp-58]
:004FCBC0 51                     push ecx
:004FCBC1 52                     push edx
:004FCBC2 C745B802000000         mov [ebp-48], 00000002
:004FCBC9 899DC8FEFFFF           mov dword ptr [ebp+FFFFFEC8], ebx
:004FCBCF FFD7                   call edi
:004FCBD1 668BD6                 mov dx, si
:004FCBD4 8D45DC                 lea eax, dword ptr [ebp-24]
:004FCBD7 6683C201               add dx, 0001
:004FCBDB 8985A0FEFFFF           mov dword ptr [ebp+FFFFFEA0], eax
:004FCBE1 0F80270C0000           jo 004FD80E
:004FCBE7 8D4D88                 lea ecx, dword ptr [ebp-78]
:004FCBEA C785B0FEFFFFC0134200   mov dword ptr [ebp+FFFFFEB0], 004213C0
:004FCBF4 0FBFC2                 movsx eax, dx
:004FCBF7 51                     push ecx
:004FCBF8 8D8D98FEFFFF           lea ecx, dword ptr [ebp+FFFFFE98]
:004FCBFE 50                     push eax
:004FCBFF 8D9578FFFFFF           lea edx, dword ptr [ebp+FFFFFF78]
:004FCC05 51                     push ecx
:004FCC06 52                     push edx
:004FCC07 C785A8FEFFFF08800000   mov dword ptr [ebp+FFFFFEA8], 00008008
:004FCC11 C7459001000000         mov [ebp-70], 00000001
:004FCC18 C7458802000000         mov [ebp-78], 00000002
:004FCC1F 899D98FEFFFF           mov dword ptr [ebp+FFFFFE98], ebx
:004FCC25 FFD7                   call edi
:004FCC27 668BD6                 mov dx, si
:004FCC2A 8D45DC                 lea eax, dword ptr [ebp-24]
:004FCC2D 6683C201               add dx, 0001
:004FCC31 898570FEFFFF           mov dword ptr [ebp+FFFFFE70], eax
:004FCC37 0F80D10B0000           jo 004FD80E
:004FCC3D 8D8D48FFFFFF           lea ecx, dword ptr [ebp+FFFFFF48]
:004FCC43 C78580FEFFFFC8134200   mov dword ptr [ebp+FFFFFE80], 004213C8
:004FCC4D 0FBFC2                 movsx eax, dx
:004FCC50 51                     push ecx
:004FCC51 8D8D68FEFFFF           lea ecx, dword ptr [ebp+FFFFFE68]
:004FCC57 50                     push eax
:004FCC58 8D9538FFFFFF           lea edx, dword ptr [ebp+FFFFFF38]
:004FCC5E 51                     push ecx
:004FCC5F 52                     push edx
:004FCC60 C78578FEFFFF08800000   mov dword ptr [ebp+FFFFFE78], 00008008
:004FCC6A C78550FFFFFF01000000   mov dword ptr [ebp+FFFFFF50], 00000001
```



```
:004FCC74 C78548FFFFFF02000000     mov dword ptr [ebp+FFFFFF48], 00000002  

:004FCC7E 899D68FEFFFF             mov dword ptr [ebp+FFFFFE68], ebx  

:004FCC84 FFD7                     call edi  

:004FCC86 668BD6                  mov dx, si  

:004FCC89 8D45DC                  lea eax, dword ptr [ebp-24]  

:004FCC8C 6683C201                add dx, 0001  

:004FCC90 898540FEFFFF            mov dword ptr [ebp+FFFFFE40], eax  

:004FCC96 0F80720B0000            jo 004FD80E  

:004FCC9C 8D8D18FFFFFF            lea ecx, dword ptr [ebp+FFFFFF18]  

:004FCCA2 C78550FEFFFFD0134200    mov dword ptr [ebp+FFFFFE50], 004213D0  

:004FCCAC 0FBFC2                  movsx eax, dx  

:004FCCAF 51                      push ecx  

:004FCCB0 8D8D38FEFFFF            lea ecx, dword ptr [ebp+FFFFFE38]  

:004FCCB6 50                      push eax  

:004FCCB7 8D9508FFFFFF            lea edx, dword ptr [ebp+FFFFFF08]  

:004FCCBD 51                      push ecx  

:004FCCBE 52                      push edx  

:004FCCBF C78548FEFFFF08800000    mov dword ptr [ebp+FFFFFE48], 00008008  

:004FCCC9 C78520FFFFFF01000000    mov dword ptr [ebp+FFFFFF20], 00000001  

:004FCCD3 C78518FFFFFF02000000    mov dword ptr [ebp+FFFFFF18], 00000002  

:004FCCDD 899D38FEFFFF            mov dword ptr [ebp+FFFFFE38], ebx  

:004FCCE3 FFD7                     call edi  

:004FCCE5 8D45A8                  lea eax, dword ptr [ebp-58]  

:004FCCE8 8D8DA8FEFFFF            lea ecx, dword ptr [ebp+FFFFFEA8]  

:004FCCEE 50                      push eax  

:004FCCEF C78520FEFFFFD8134200    mov dword ptr [ebp+FFFFFE20], 004213D8  

:004FCCF9 C78518FEFFFF08800000    mov dword ptr [ebp+FFFFFE18], 00008008  

:004FCD03 51                      push ecx  

:004FCD04 8D5598                  lea edx, dword ptr [ebp-68]  

:004FCD07 52                      push edx

* Reference To: MSVBVM60.__vbaVarCmpGe, Ord:0000h  
                          |  

:004FCD08 FF1584104000             Call dword ptr [00401084]  

:004FCD0E 50                      push eax  

:004FCD0F 8D8578FFFFFF             lea eax, dword ptr [ebp+FFFFFF78]  

:004FCD15 8D8D78FEFFFF             lea ecx, dword ptr [ebp+FFFFFE78]  

:004FCD1B 50                      push eax  

:004FCD1C 8D9568FFFFFF             lea edx, dword ptr [ebp+FFFFFF68]  

:004FCD22 51                      push ecx  

:004FCD23 52                      push edx

* Reference To: MSVBVM60.__vbaVarCmpLe, Ord:0000h  
                          |  

:004FCD24 FF1570114000             Call dword ptr [00401170]  

:004FCD2A 50                      push eax  

:004FCD2B 8D8558FFFFFF             lea eax, dword ptr [ebp+FFFFFF58]  

:004FCD31 50                      push eax

* Reference To: MSVBVM60.__vbaVarAnd, Ord:0000h  
                          |  

:004FCD32 FF1548114000             Call dword ptr [00401148]  

:004FCD38 8D8D38FFFFFF             lea ecx, dword ptr [ebp+FFFFFF38]  

:004FCD3E 50                      push eax  

:004FCD3F 8D9548FEFFFF             lea edx, dword ptr [ebp+FFFFFE48]  

:004FCD45 51                      push ecx  

:004FCD46 8D8528FFFFFF             lea eax, dword ptr [ebp+FFFFFF28]  

:004FCD4C 52                      push edx  

:004FCD4D 50                      push eax

* Reference To: MSVBVM60.__vbaVarCmpGe, Ord:0000h  
                          |  

:004FCD4E FF1584104000             Call dword ptr [00401084]  

:004FCD54 8D8D08FFFFFF             lea ecx, dword ptr [ebp+FFFFFF08]  

:004FCD5A 50                      push eax  

:004FCD5B 8D9518FEFFFF             lea edx, dword ptr [ebp+FFFFFE18]  

:004FCD61 51                      push ecx  

:004FCD62 8D85F8FEFFFF             lea eax, dword ptr [ebp+FFFFFEF8]  

:004FCD68 52                      push edx  

:004FCD69 50                      push eax

* Reference To: MSVBVM60.__vbaVarCmpLe, Ord:0000h  
                          |  

:004FCD6A FF1570114000             Call dword ptr [00401170]  

:004FCD70 8D8DE8FEFFFF             lea ecx, dword ptr [ebp+FFFFFEE8]  

:004FCD76 50                      push eax  

:004FCD77 51                      push ecx

* Reference To: MSVBVM60.__vbaVarAnd, Ord:0000h  
                          |  

:004FCD78 FF1548114000             Call dword ptr [00401148]  

:004FCD7E 8D95D8FEFFFF             lea edx, dword ptr [ebp+FFFFFED8]  

:004FCD84 50                      push eax  

:004FCD85 52                      push edx

* Reference To: MSVBVM60.__vbaVarOr, Ord:0000h  
                          |  

:004FCD86 FF1510114000             Call dword ptr [00401110]  

:004FCD8C 50                      push eax

* Reference To: MSVBVM60.__vbaBoolVarNull, Ord:0000h  
                          |  

:004FCD8D FF15CC104000             Call dword ptr [004010CC]  
                          ==== > __vbaBoolVarNull 判断所取字符是否为数字！

:004FCD93 66898508FEFFFF            mov word ptr [ebp+FFFFFE08], ax  

:004FCD9A 8D8508FFFFFF             lea eax, dword ptr [ebp+FFFFFF08]  

:004FCDA0 8D8D18FFFFFF             lea ecx, dword ptr [ebp+FFFFFF18]  

:004FCDA6 50                      push eax  

:004FCDA7 8D9538FFFFFF             lea edx, dword ptr [ebp+FFFFFF38]  

:004FCDAD 51                      push ecx  

:004FCDAE 8D8548FFFFFF             lea eax, dword ptr [ebp+FFFFFF48]  

:004FCDB4 52                      push edx  

:004FCDB5 8D8D78FFFFFF             lea ecx, dword ptr [ebp+FFFFFF78]  

:004FCDBB 50                      push eax  

:004FCDBC 8D5588                  lea edx, dword ptr [ebp-78]  

:004FCDBF 51                      push ecx  

:004FCDC0 8D45A8                  lea eax, dword ptr [ebp-58]  

:004FCDC3 52                      push edx  

:004FCDC4 8D4DB8                  lea ecx, dword ptr [ebp-48]  

:004FCDC7 50                      push eax  

:004FCDC8 51                      push ecx  

:004FCDC9 6A08                     push 00000008

* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h  
                          |  

:004FCDCB FF1534104000             Call dword ptr [00401034]  

:004FCDD1 83C424                  add esp, 00000024  

:004FCDD4 6683BD08FEFFFF00         cmp word ptr [ebp+FFFFFE08], 0000  

:004FCDDC 0F84F5000000             je 004FCED7  
                          ==== > 如果是数字，则此处直接跳下去保存此字符！

:004FCDE2 8B55E0                  mov edx, dword ptr [ebp-20]  

:004FCDE5 8D45DC                  lea eax, dword ptr [ebp-24]  

:004FCDE8 8995A0FEFFFF             mov dword ptr [ebp+FFFFFEA0], edx  

:004FCDEE 668BD6                  mov dx, si  

:004FCDF1 6683C201                add dx, 0001  

:004FCDF5 8985D0FEFFFF             mov dword ptr [ebp+FFFFFED0], eax  

:004FCDFB 0F800D0A0000             jo 004FD80E  

:004FCE01 8D4DB8                  lea ecx, dword ptr [ebp-48]  

:004FCE04 C78598FEFFFF08000000     mov dword ptr [ebp+FFFFFE98], 00000008  

:004FCE0E 0FBFC2                  movsx eax, dx  

:004FCE11 51                      push ecx  

:004FCE12 8D8DC8FEFFFF             lea ecx, dword ptr [ebp+FFFFFEC8]  

:004FCE18 50                      push eax  

:004FCE19 8D55A8                  lea edx, dword ptr [ebp-58]  

:004FCE1C 51                      push ecx  

:004FCE1D 52                      push edx  

:004FCE1E C745C001000000           mov [ebp-40], 00000001  

:004FCE25 C745B802000000           mov [ebp-48], 00000002  

:004FCE2C 899DC8FEFFFF             mov dword ptr [ebp+FFFFFEC8], ebx  

:004FCE32 FFD7                     call edi  
                          ==== > 取信息码中的字母：K、A、D、J

:004FCE34 8D45A8                  lea eax, dword ptr [ebp-58]  

:004FCE37 8D4DD8                  lea ecx, dword ptr [ebp-28]  

:004FCE3A 50                      push eax  

:004FCE3B 51                      push ecx

* Reference To: MSVBVM60.__vbaStrVarVal, Ord:0000h  
                          |  
```



```
:004FCE3C FF157C114000           Call dword ptr [0040117C]
:004FCE42 50                     push eax

* Reference To: MSVBVM60.rtcAnsivalueBstr, Ord:0204h

:004FCE43 FF1544104000           Call dword ptr [00401044]

====>取字母所对应的 HEX 值：4B、41、44、4A

:004FCE49 668945A0               mov word ptr [ebp-60], ax

====>AX=4B、41、44、4A

:004FCE4D 8D5598                 lea edx, dword ptr [ebp-68]
:004FCE50 8D4588                 lea eax, dword ptr [ebp-78]
:004FCE53 52                     push edx
:004FCE54 50                     push eax
:004FCE55 C7459802000000         mov [ebp-68], 00000002

* Reference To: MSVBVM60.rtcVarStrFromVar, Ord:0265h

:004FCE5C FF15EC114000           Call dword ptr [004011EC]

====>取其 10 进制值：75、65、68、74

:004FCE62 8D4D88                 lea ecx, dword ptr [ebp-78]
:004FCE65 8D9578FFFFFF           lea edx, dword ptr [ebp+FFFFFF78]
:004FCE6B 51                     push ecx
:004FCE6C 52                     push edx

* Reference To: MSVBVM60.rtcTrimVar, Ord:0208h

:004FCE6D FF15BC104000           Call dword ptr [004010BC]
:004FCE73 8D8598FEFFFF           lea eax, dword ptr [ebp+FFFFFE98]
:004FCE79 8D8D78FFFFFF           lea ecx, dword ptr [ebp+FFFFFF78]
:004FCE7F 50                     push eax
:004FCE80 8D9568FFFFFF           lea edx, dword ptr [ebp+FFFFFF68]
:004FCE86 51                     push ecx
:004FCE87 52                     push edx

* Reference To: MSVBVM60.__vbaVarAdd, Ord:0000h

:004FCE88 FF15D8114000           Call dword ptr [004011D8]

====>里面有__vbaVarCat 函数，将以上所得连接起来！

:004FCE8E 50                     push eax

* Reference To: MSVBVM60.__vbaStrVarMove, Ord:0000h

:004FCE8F FF1528104000           Call dword ptr [00401028]
:004FCE95 8BD0                   mov edx, eax

最后得到 ==== >EDX=EAX=575650896874

:004FCE97 8D4DE0                 lea ecx, dword ptr [ebp-20]

* Reference To: MSVBVM60.__vbaStrMove, Ord:0000h

:004FCE9A FF1508124000           Call dword ptr [00401208]
:004FCEA0 8D4DD8                 lea ecx, dword ptr [ebp-28]

* Reference To: MSVBVM60.__vbaFreeStr, Ord:0000h

:004FCEA3 FF1524124000           Call dword ptr [00401224]
:004FCEA9 8D8568FFFFFF           lea eax, dword ptr [ebp+FFFFFF68]
:004FCEAF 8D8D78FFFFFF           lea ecx, dword ptr [ebp+FFFFFF78]
:004FCEB5 50                     push eax
:004FCEB6 8D5588                 lea edx, dword ptr [ebp-78]
:004FCEB9 51                     push ecx
:004FCEBA 8D4598                 lea eax, dword ptr [ebp-68]
:004FCEBD 52                     push edx
:004FCEBE 8D4DA8                 lea ecx, dword ptr [ebp-58]
:004FCEC1 50                     push eax
:004FCEC2 8D55B8                 lea edx, dword ptr [ebp-48]
:004FCEC5 51                     push ecx
:004FCEC6 52                     push edx
:004FCEC7 6A06                   push 00000006

* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h

:004FCEC9 FF1534104000           Call dword ptr [00401034]
:004FCECF 83C41C                 add esp, 0000001C
:004FCED2 E9B2010000             jmp 004FD089

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004FCDDC(C)

:004FCED7 668BD6                 mov dx, si
:004FCEDA 8D45DC                 lea eax, dword ptr [ebp-24]
:004FCEDD 6683C201               add dx, 0001
:004FCEE1 8985D0FEFFFF           mov dword ptr [ebp+FFFFFED0], eax
:004FCEE7 0F8021090000           jo 004FD80E
:004FCEED 8D4DB8                 lea ecx, dword ptr [ebp-48]
:004FCEF0 C745C001000000         mov [ebp-40], 00000001
:004FCEF7 0FBFC2                 movsx eax, dx
:004FCEFA 51                     push ecx
:004FCEFB 8D8DC8FEFFFF           lea ecx, dword ptr [ebp+FFFFFEC8]
:004FCF01 50                     push eax
:004FCF02 8D55A8                 lea edx, dword ptr [ebp-58]
:004FCF05 51                     push ecx
:004FCF06 52                     push edx
:004FCF07 C745B802000000         mov [ebp-48], 00000002
:004FCF0E 899DC8FEFFFF           mov dword ptr [ebp+FFFFFEC8], ebx
:004FCF14 FFD7                   call edi
:004FCF16 668BD6                 mov dx, si
:004FCF19 8D45DC                 lea eax, dword ptr [ebp-24]
:004FCF1C 6683C201               add dx, 0001
:004FCF20 8985A0FEFFFF           mov dword ptr [ebp+FFFFFEA0], eax
:004FCF26 0F80E2080000           jo 004FD80E
:004FCF2C 8D4D88                 lea ecx, dword ptr [ebp-78]
:004FCF2F C785B0FEFFFFE0134200   mov dword ptr [ebp+FFFFFEB0], 004213E0
:004FCF39 0FBFC2                 movsx eax, dx
:004FCF3C 51                     push ecx
:004FCF3D 8D8D98FEFFFF           lea ecx, dword ptr [ebp+FFFFFE98]
:004FCF43 50                     push eax
:004FCF44 8D9578FFFFFF           lea edx, dword ptr [ebp+FFFFFF78]
:004FCF4A 51                     push ecx
:004FCF4B 52                     push edx
:004FCF4C C785A8FEFFFF08800000   mov dword ptr [ebp+FFFFFEA8], 00008008
:004FCF56 C7459001000000         mov [ebp-70], 00000001
:004FCF5D C7458802000000         mov [ebp-78], 00000002
:004FCF64 899D98FEFFFF           mov dword ptr [ebp+FFFFFE98], ebx
:004FCF6A FFD7                   call edi
:004FCF6C 8D45A8                 lea eax, dword ptr [ebp-58]
:004FCF6F 8D8DA8FEFFFF           lea ecx, dword ptr [ebp+FFFFFEA8]
:004FCF75 50                     push eax
:004FCF76 8D5598                 lea edx, dword ptr [ebp-68]
:004FCF79 51                     push ecx
:004FCF7A 52                     push edx
:004FCF7B C78580FEFFFFE8134200   mov dword ptr [ebp+FFFFFE80], 004213E8
:004FCF85 C78578FEFFFF08800000   mov dword ptr [ebp+FFFFFE78], 00008008

* Reference To: MSVBVM60.__vbaVarCmpGe, Ord:0000h

:004FCF8F FF1584104000           Call dword ptr [00401084]
:004FCF95 50                     push eax
:004FCF96 8D8578FFFFFF           lea eax, dword ptr [ebp+FFFFFF78]
:004FCF9C 8D8D78FEFFFF           lea ecx, dword ptr [ebp+FFFFFE78]
:004FCFA2 50                     push eax
:004FCFA3 8D9568FFFFFF           lea edx, dword ptr [ebp+FFFFFF68]
:004FCFA9 51                     push ecx
:004FCFAA 52                     push edx

* Reference To: MSVBVM60.__vbaVarCmpLe, Ord:0000h

:004FCFAB FF1570114000           Call dword ptr [00401170]
:004FCFB1 50                     push eax
:004FCFB2 8D8558FFFFFF           lea eax, dword ptr [ebp+FFFFFF58]
:004FCFB8 50                     push eax

* Reference To: MSVBVM60.__vbaVarAnd, Ord:0000h

:004FCFB9 FF1548114000           Call dword ptr [00401148]
:004FCFBF 50                     push eax

* Reference To: MSVBVM60.__vbaBoolVarNull, Ord:0000h

:004FCFC0 FF15CC104000           Call dword ptr [004010CC]

==== >__vbaBoolVarNull 再判断一次是否是数字

:004FCFC6 8D8D78FFFFFF           lea ecx, dword ptr [ebp+FFFFFF78]
:004FCFCC 66898508FEFFFF         mov word ptr [ebp+FFFFFE08], ax
:004FCFD3 8D5588                 lea edx, dword ptr [ebp-78]
:004FCFD6 51                     push ecx
```



:004FCFD7 8D45A8                `lea eax, dword ptr [ebp-58]`

:004FCFDA 52                    `push edx`

:004FCFDB 8D4DB8                `lea ecx, dword ptr [ebp-48]`

:004FCFDE 50                    `push eax`

:004FCFDF 51                    `push ecx`

:004FCFE0 6A04                  `push 00000004`

* Reference To: `MSVBVM60.__vbaFreeVarList`, Ord:0000h

:004FCFE2 FF1534104000          `Call dword ptr [00401034]`

:004FCFE8 83C414                `add esp, 00000014`

:004FCFEB 6683BD08FEFFFF00      `cmp word ptr [ebp+FFFFFE08], 0000`

:004FCFF3 0F8490000000          `je 004FD089`

:004FCFF9 8B55E0                `mov edx, dword ptr [ebp-20]`

:004FCFFC 8D45DC                `lea eax, dword ptr [ebp-24]`

:004FCFFF 8995B0FEFFFF         `mov dword ptr [ebp+FFFFFEB0], edx`

:004FD005 668BD6               `mov dx, si`

:004FD008 6683C201             `add dx, 0001`

:004FD00C 8985D0FEFFFF         `mov dword ptr [ebp+FFFFFED0], eax`

:004FD012 0F80F6070000         `jo 004FD80E`

:004FD018 8D4DB8               `lea ecx, dword ptr [ebp-48]`

:004FD01B C785A8FEFFFF08000000 `mov dword ptr [ebp+FFFFFEA8], 00000008`

:004FD025 0FBFC2               `movsx eax, dx`

:004FD028 51                   `push ecx`

:004FD029 8D8DC8FEFFFF        `lea ecx, dword ptr [ebp+FFFFFEC8]`

:004FD02F 50                   `push eax`

:004FD030 8D55A8               `lea edx, dword ptr [ebp-58]`

:004FD033 51                   `push ecx`

:004FD034 52                   `push edx`

:004FD035 C745C001000000       `mov [ebp-40], 00000001`

:004FD03C C745B802000000       `mov [ebp-48], 00000002`

:004FD043 899DC8FEFFFF        `mov dword ptr [ebp+FFFFFEC8], ebx`

:004FD049 FFD7                 `call edi`

:004FD04B 8D85A8FEFFFF        `lea eax, dword ptr [ebp+FFFFFEA8]`

:004FD051 8D4DA8               `lea ecx, dword ptr [ebp-58]`

:004FD054 50                   `push eax`

:004FD055 8D5598               `lea edx, dword ptr [ebp-68]`

:004FD058 51                   `push ecx`

:004FD059 52                   `push edx`

* Reference To: `MSVBVM60.__vbaVarAdd`, Ord:0000h

:004FD05A FF15D8114000          `Call dword ptr [004011D8]`

:004FD060 50                   `push eax`

* Reference To: `MSVBVM60.__vbaStrVarMove`, Ord:0000h

:004FD061 FF1528104000          `Call dword ptr [00401028]`

:004FD067 8BD0                 `mov edx, eax`

====>EDX=5、0、8、9      信息码中的数字！

:004FD069 8D4DE0               `lea ecx, dword ptr [ebp-20]`

* Reference To: `MSVBVM60.__vbaStrMove`, Ord:0000h

:004FD06C FF1508124000          `Call dword ptr [00401208]`

:004FD072 8D4598               `lea eax, dword ptr [ebp-68]`

:004FD075 8D4DA8               `lea ecx, dword ptr [ebp-58]`

:004FD078 50                   `push eax`

:004FD079 8D55B8               `lea edx, dword ptr [ebp-48]`

:004FD07C 51                   `push ecx`

:004FD07D 52                   `push edx`

:004FD07E 6A03                 `push 00000003`

* Reference To: `MSVBVM60.__vbaFreeVarList`, Ord:0000h

:004FD080 FF1534104000          `Call dword ptr [00401034]`

:004FD086 83C410               `add esp, 00000010`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004FCED2(U), :004FCFF3(C)

:004FD089 B801000000            `mov eax, 00000001`

:004FD08E 6603C6               `add ax, si`

:004FD091 0F8077070000         `jo 004FD80E`

:004FD097 8BF0                 `mov esi, eax`

:004FD099 E9E7FAFFFF           `jmp 004FCB85`

====>循环信息码位数次！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004FCB8C(C)

:004FD09E 8B45E0               `mov eax, dword ptr [ebp-20]`

====>EAX=575650896874

:004FD0A1 50                   `push eax`

* Reference To: `MSVBVM60.__vbaLenBstr`, Ord:0000h

:004FD0A2 FF152C104000          `Call dword ptr [0040102C]`

====>求 575650896874 的长度  EAX=C

:004FD0A8 83F808               `cmp eax, 00000008`

====>8 位？

:004FD0AB 7E40                 `jle 004FD0ED`

:004FD0AD 8D95C8FEFFFF        `lea edx, dword ptr [ebp+FFFFFEC8]`

:004FD0B3 6A08                 `push 00000008`

:004FD0B5 8D45B8               `lea eax, dword ptr [ebp-48]`

:004FD0B8 8D4DE0               `lea ecx, dword ptr [ebp-20]`

:004FD0BB 52                   `push edx`

:004FD0BC 50                   `push eax`

:004FD0BD 898DD0FEFFFF        `mov dword ptr [ebp+FFFFFED0], ecx`

:004FD0C3 899DC8FEFFFF        `mov dword ptr [ebp+FFFFFEC8], ebx`

* Reference To: `MSVBVM60.rtcRightCharVar`, Ord:026Bh

:004FD0C9 FF150C124000          `Call dword ptr [0040120C]`

====>从字符串右边取相应字符

用 Ollydbg 复制的代码 ：  733B47F7   03C1   ADD EAX,ECX

====>ECX=575650896874 从右边取 8 位字符！=50896874

:004FD0CF 8D4DB8               `lea ecx, dword ptr [ebp-48]`

:004FD0D2 51                   `push ecx`

* Reference To: `MSVBVM60.__vbaStrVarMove`, Ord:0000h

:004FD0D3 FF1528104000          `Call dword ptr [00401028]`

:004FD0D9 8BD0                 `mov edx, eax`

====>EDX=EAX=50896874        机器码最后变为这个！

:004FD0DB 8D4DE0               `lea ecx, dword ptr [ebp-20]`

* Reference To: `MSVBVM60.__vbaStrMove`, Ord:0000h

:004FD0DE FF1508124000          `Call dword ptr [00401208]`

:004FD0E4 8D4DB8               `lea ecx, dword ptr [ebp-48]`

* Reference To: `MSVBVM60.__vbaFreeVar`, Ord:0000h

:004FD0E7 FF1520104000          `Call dword ptr [00401020]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004FD0AB(C)

:004FD0ED 8B7D08               `mov edi, dword ptr [ebp+08]`

:004FD0F0 57                   `push edi`

:004FD0F1 8B17                 `mov edx, dword ptr [edi]`

:004FD0F3 FF9204030000          `call dword ptr [edx+00000304]`

* Reference To: `MSVBVM60.__vbaObjSet`, Ord:0000h

:004FD0F9 8B1D94104000          `mov ebx, dword ptr [00401094]`

:004FD0FF 50                   `push eax`

:004FD100 8D45D0               `lea eax, dword ptr [ebp-30]`

:004FD103 50                   `push eax`

:004FD104 FFD3                 `call ebx`

:004FD106 8BF0                 `mov esi, eax`

:004FD108 8D55D8               `lea edx, dword ptr [ebp-28]`

:004FD10B 52                   `push edx`

:004FD10C 56                   `push esi`

:004FD10D 8B0E                 `mov ecx, dword ptr [esi]`

:004FD10F FF91A0000000          `call dword ptr [ecx+000000A0]`

:004FD115 85C0                 `test eax, eax`

:004FD117 DBE2                 `fclex`

:004FD119 7D12                 `jge 004FD12D`

:004FD11B 68A0000000           `push 000000A0`

:004FD120 6828084200           `push 00420828`

:004FD125 56                   `push esi`

:004FD126 50                   `push eax`

* Reference To: `MSVBVM60.__vbaHresultCheckObj`, Ord:0000h

:004FD127 FF156C104000          `Call dword ptr [0040106C]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004FD119(C)

:004FD12D 8B45D8               `mov eax, dword ptr [ebp-28]`



以下是关于注册码验证逻辑的反汇编代码分析：

```
:004FD130 50                           push eax
:004FD131 6828004200                   push 00420028

* Reference To: MSVBVM60.__vbaStrCmp, Ord:0000h

:004FD136 FF15F8104000                 Call dword ptr [004010F8]
:004FD13C 8BF0                         mov esi, eax
:004FD13E 8D4DD8                      lea ecx, dword ptr [ebp-28]
:004FD141 F7DE                         neg esi
:004FD143 1BF6                         sbb esi, esi
:004FD145 F7DE                         neg esi
:004FD147 F7DE                         neg esi

* Reference To: MSVBVM60.__vbaFreeStr, Ord:0000h

:004FD149 FF1524124000                 Call dword ptr [00401224]
:004FD14F 8D4DD0                      lea ecx, dword ptr [ebp-30]

* Reference To: MSVBVM60.__vbaFreeObj, Ord:0000h

:004FD152 FF1528124000                 Call dword ptr [00401228]
:004FD158 6685F6                       test si, si
:004FD15B 0F84F3040000                 je 004FD654
:004FD161 8B0F                         mov ecx, dword ptr [edi]
:004FD163 57                           push edi
:004FD164 FF9104030000                 call dword ptr [ecx+00000304]
:004FD16A 8D55D0                      lea edx, dword ptr [ebp-30]
:004FD16D 50                           push eax
:004FD16E 52                           push edx
:004FD16F FFD3                         call ebx
:004FD171 8BF0                         mov esi, eax
:004FD173 8D4DD8                      lea ecx, dword ptr [ebp-28]
:004FD176 51                           push ecx
:004FD177 56                           push esi
:004FD178 8B06                         mov eax, dword ptr [esi]
:004FD17A FF90A0000000                 call dword ptr [eax+000000A0]
:004FD180 85C0                         test eax, eax
:004FD182 DBE2                         fclex
:004FD184 7D12                         jge 004FD198
:004FD186 68A0000000                   push 000000A0
:004FD18B 6828084200                   push 00420828
:004FD190 56                           push esi
:004FD191 50                           push eax

* Reference To: MSVBVM60.__vbaHresultCheckObj, Ord:0000h

:004FD192 FF156C104000                 Call dword ptr [0040106C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004FD184(C)

:004FD198 8B45D8                       mov eax, dword ptr [ebp-28]
                                     ==== >EAX=13572468          试炼码

:004FD19B 8D55B8                      lea edx, dword ptr [ebp-48]
:004FD19E 8945C0                       mov dword ptr [ebp-40], eax
:004FD1A1 8D45A8                      lea eax, dword ptr [ebp-58]
:004FD1A4 52                           push edx
:004FD1A5 50                           push eax
:004FD1A6 C745D800000000               mov [ebp-28], 00000000
:004FD1AD C745B808000000               mov [ebp-48], 00000008

* Reference To: MSVBVM60.rtcTrimVar, Ord:0208h

:004FD1B4 FF15BC104000                 Call dword ptr [004010BC]
:004FD1BA 8D4DA8                      lea ecx, dword ptr [ebp-58]
:004FD1BD 8D55D4                      lea edx, dword ptr [ebp-2C]
:004FD1C0 51                           push ecx
:004FD1C1 52                           push edx

* Reference To: MSVBVM60.__vbaStrVarVal, Ord:0000h

:004FD1C2 FF157C114000                 Call dword ptr [0040117C]
:004FD1C8 50                           push eax

* Reference To: MSVBVM60.rtcR8ValFromBstr, Ord:0245h

:004FD1C9 FF152C124000                 Call dword ptr [0040122C]
:004FD1CF 8B45E0                       mov eax, dword ptr [ebp-20]
                                     ==== >EAX=50896874          信息码处理的结果

:004FD1D2 DD9D0CFEFFFF                 fstp qword ptr [ebp+FFFFFE0C]
:004FD1D8 50                           push eax

* Reference To: MSVBVM60.rtcR8ValFromBstr, Ord:0245h

:004FD1D9 FF152C124000                 Call dword ptr [0040122C]

* Reference To: MSVBVM60.__vbaFpI4, Ord:0000h

:004FD1DF FF15F8114000                 Call dword ptr [004011F8]
                                     ==== >将 50896874 转化成 16 进制值 03089FEA（H）

:004FD1E5 99                           cdq
:004FD1E6 B9E9471700                   mov ecx, 001747E9
                                     ==== >ECX=001747E9

:004FD1EB F7F9                         idiv ecx
                                     ==== >EDX=03089FEA % 001747E9=00085AE1

:004FD1ED 6BD20B                       imul edx, 0000000B
                                     ==== >EDX=00085AE1 * B=005BE7AB（H）=6023083（D）

:004FD1F0 0F8018060000                 jo 004FD80E
:004FD1F6 81C20F786F00                 add edx, 006F780F
                                     ==== >EDX=005BE7AB + 006F780F=00CB5FBA（H）=13328314（D）

:004FD1FC 0F800C060000                 jo 004FD80E
:004FD202 8995D0FDFFFF                 mov dword ptr [ebp+FFFFFDD0], edx
:004FD208 DB85D0FDFFFF                 fild dword ptr [ebp+FFFFFDD0]
:004FD20E DD9DC8FDFFFF                 fstp qword ptr [ebp+FFFFFDC8]
:004FD214 DD850CFEFFFF                 fld qword ptr [ebp+FFFFFE0C]

* Reference To: MSVBVM60.__vbaFpR8, Ord:0000h

:004FD21A FF15D0104000                 Call dword ptr [004010D0]
:004FD220 DC9DC8FDFFFF                 fcomp qword ptr [ebp+FFFFFDC8]
                                     ==== >真假码比较！
                                     ==== >ST=13572468.000000000000
                                     ==== >SS[0074E81C]=13328314.00000000

:004FD226 DFE0                         fstsw ax
:004FD228 F6C440                       test ah, 40
:004FD22B 7407                         je 004FD234
                                     ==== >跳则 OVER！

:004FD22D B801000000                   mov eax, 00000001
                                     ==== >置 1 则 OK！

:004FD232 EB02                         jmp 004FD236

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004FD22B(C)

:004FD234 33C0                         xor eax, eax
                                     ==== >清 0 则 OVER！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004FD232(U)

:004FD236 F7D8                         neg eax
:004FD238 8D4DD4                      lea ecx, dword ptr [ebp-2C]
:004FD23B 668BF0                       mov si, ax

* Reference To: MSVBVM60.__vbaFreeStr, Ord:0000h

:004FD23E FF1524124000                 Call dword ptr [00401224]
:004FD244 8D4DD0                      lea ecx, dword ptr [ebp-30]

* Reference To: MSVBVM60.__vbaFreeObj, Ord:0000h

:004FD247 FF1528124000                 Call dword ptr [00401228]

* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h

:004FD24D 8B1D34104000                 mov ebx, dword ptr [00401034]
:004FD253 8D55A8                      lea edx, dword ptr [ebp-58]
:004FD256 8D45B8                      lea eax, dword ptr [ebp-48]
:004FD259 52                           push edx
:004FD25A 50                           push eax
:004FD25B 6A02                         push 00000002
:004FD25D FFD3                         call ebx
:004FD25F 83C40C                       add esp, 0000000C
:004FD262 6685F6                       test si, si
:004FD265 0F8455030000                 je 004FD5C0
                                     ==== >跳则 OVER！

:004FD26B 8D7734                      lea esi, dword ptr [edi+34]
                                     ==== >下面保存注册信息！

* Possible StringData Ref from Code Obj ->"sselect * from Regsn"

:004FD26E BAF0134200                   mov edx, 004213F0
:004FD273 8BCE                         mov ecx, esi

…… ……省 略…… ……

* Possible StringData Ref from Code Obj ->"SSn"

:004FD389 B91C144200                   mov ecx, 0042141C
```



```
:004FD38E 898DD0FEFFFF           mov dword ptr [ebp+FFFFFED0], ecx  
:004FD394 8B30                   mov esi, dword ptr [eax]  
:004FD396 8913                   mov dword ptr [ebx], edx  
:004FD398 8B95CCFEFFFF           mov edx, dword ptr [ebp+FFFFFECC]  
:004FD39E 50                     push eax  
:004FD39F 8985FCFDFFFF           mov dword ptr [ebp+FFFFFDFC], eax  
:004FD3A5 895304                 mov dword ptr [ebx+04], edx  
:004FD3A8 894B08                 mov dword ptr [ebx+08], ecx  
:004FD3AB 8B8DD4FEFFFF           mov ecx, dword ptr [ebp+FFFFFED4]  
:004FD3B1 894B0C                 mov dword ptr [ebx+0C], ecx  
:004FD3B4 FF5628                 call [esi+28]  
:004FD3B7 85C0                   test eax, eax  
:004FD3B9 DBE2                   fclex  
:004FD3BB 7D15                   jge 004FD3D2  
:004FD3BD 8B95FCFDFFFF           mov edx, dword ptr [ebp+FFFFFDFC]  
:004FD3C3 6A28                   push 00000028  
:004FD3C5 685C0B4200             push 00420B5C  
:004FD3CA 52                     push edx  
:004FD3CB 50                     push eax
…… ……省 略…… ……

* Reference To: `MSVBVM60.rtcMsgBox`, Ord:0253h  

                            |  
:004FD597 FF1598104000       Call dword ptr [00401098]  
                            ====》呵呵，胜利女神！

…… ……省 略…… ……

* Reference To: `MSVBVM60.rtcMsgBox`, Ord:0253h  

                            |  
:004FD635 FF1598104000       Call dword ptr [00401098]  
                            ====》BAD BOY！
—————————————————————————————————  

【算 法  总 结】：

1、依次取信息码字符，如果是数字则直接保留，是字母则转化为所对应的 `ASCII` 码的 10 进制值。  
 信息码 `5KA089DJ` 变为 `575650896874`

2、取 `575650896874` 的后 8 位：`50896874`，并转化成 16 进制值 `03089FEA`

3、（`03089FEA` % `001747E9`）\* `B` + `006F780F`=`00CB5FBA`（H）=`13328314`（D）就是注册码了！

—————————————————————————————————   

【KeyMake 之{70th}内存注册机】：

中断地址：`004FD202`  

中断次数：`1`  

第一字节：`89`  

指令长度：`6`

寄存器方式：`EDX`              

10 进制

—————————————————————————————————   

【注册信息保存】：

同目录下的 `AcctSet.mdb` 中：`RegSn`

—————————————————————————————————   

【整      理】：

信息码：`5KA089DJ`  

注册码：`13328314`

—————————————————————————————————  

             ,     \_/   
        /| \_.-~/          \\_     ,       青春都一饷 
        ( /~   /            \\~.\_ |\
        \`\\   \_/             \   ~\ )          忍把浮名  
    _-~~~-.)  )\_\_/;;,.          \_  //' 
/'\_,\   --~   \ ~~~-  ,;;\_\_\_(  (.-~~~-.       换了破解轻狂 
\~ \_( ,\_..-\ (     ,;'' /    ~--  /\_\`\\ 
/~~//'   /' \`\\          ) /--..\_, )\_  \`~  
 "  \`~"  "      \`"      /~'\`\\    \`\\~~\\   

                    "      "   "~'  ""

                    Cracked By 巢水工作坊——fly [OCN][FCG]

                2003-04-25  2:16  4dd466d0e1e0cc123afbe504da2462ca

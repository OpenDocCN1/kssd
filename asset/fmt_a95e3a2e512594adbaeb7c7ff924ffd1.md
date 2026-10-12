
```
:00410580 050000             ImpAdLdRf            ;Push ptr

:00410583 240100             NewIfNullPr          ;[Pop] [SR]

****Reference To:Global.UnLoad

                              |

:00410586 0D10000200         VCallHresult         ;Call ptr_00407CCC

:0041058B 1A74FF             FFree1Ad             ;Push [LOCAL_008C]; Call [[[LOCAL_008C]]+8]; [[LOCAL_008C]]=0

:0041058E 2750FF             LitVar               ;PushVar LOCAL_00B0

:00410591 25                 PopAdLdVar          ;

:00410592 2760FF             LitVar               ;PushVar LOCAL_00A0

:00410595 25                 PopAdLdVar          ;

:00410596 050300             ImpAdLdRf            ;Push ptr

:00410599 240400             NewIfNullPr          ;[Pop] [SR]

:0041059C 0DB0020500         VCallHresult         ;Call ptr_0040759C

:004105A1 13                 ExitProcHresult      ;

:004105A2 13                 ExitProcHresult      ;

----------------------------------------
```

### 整理总结

1. 注册码必须长度为 `8`；
2. 注册码中的第 `3`/`5`/`7` 位，必须满足：`p3 * p5 + p7 = 30`；（例：可用注册码 `== 3 * 8 + 6 =`）
3. 注册成功后，程序会在注册表中做标记：
   - 项目：`HKEY_CURRENT_USER\Software\VB and VBA Program Settings\pyfk\Options`
   - 名称：`zcf`
   - 数据：`y`
   - 因此，可以直接修改注册表进行注册；
4. 如果注册成功后想再注册，只需要改动注册表中的名称或数据即可。

### 注册机

本程序注册比较简单，略。

by `acacia`

`2003-10-17`

`4dd466d0e1e0cc123afbe504da2462ca`

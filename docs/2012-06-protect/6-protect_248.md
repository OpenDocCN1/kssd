
# Multigen vega prime 2.0 详细跟踪

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-109.htm`](https://www.pediy.com/kssd/pediy09/pediy09-109.htm)

> 【软件名称】Multigen vega prime 2.0  
> 【应用平台】Win2000 SP4  
> 【作者邮箱】chubing6143@sina.com  
> 【使用工具】OllyDbg1.10，FlexLM 7.2 SDK  
> 【软件限制】FLEXlm v9.2.0 需要 120 位长度的 SIGN  
> 【软件简介】大名鼎鼎的视景仿真软件,具体我就不介绍了.  
> 【关于本文】呵呵,由于这个软件是采用的 120 位长度的 SIGN 的 FLEXLM9.2 的加密,如果要想产生真正的 License 文件,我没有这个水平了.而如果要想爆破的话,很简单了.但由于我想仔细跟踪一下该软件的 FlexLM 加密的具体过程,呵呵,我是菜鸟,这下可费劲了,经过几个日夜不停的工作,加上资料的消化,我终于有了点感觉了.因此,写出来希望给我这样的菜鸟有一定的帮助.

前几天,师弟毕业,大家喝高了,教授(62 岁)临别赠言"扶摇直上九万里",多么豪迈啊,想想我自己,虽然也老大不小了.但是和他比起来,我还年轻,可心已经没劲了.激励自己一下,该来个"扶摇直上九厘米"了啊!呵呵,跑题了啊!

补充说明,最近一直和自己说要好好干点正事了(毕业压力好大),因此手头一直只有 FlexLM 7.2 SDK,没有 9.2 的 SDK,因此,自己分析全靠死功夫了,有错的地方,请大家指点一下啊!

【跟踪过程】利用 `LMTOOLS` 可以发现该软件是采用 `FLEXlm v9.2.0` (`lmgr.lib`) 加密的,而加密的文件是 `vsgu2_0.dll`,而软件中主要限制就是运行时检查 License 文件,我刚开始直接利用 `OD` 加载 `LynxPrime.exe` 分析,可是总断不下来,后来仔细一看,运行部分其实是 `vprun.exe`,利用 `OD` 加载它,逐步分析,很快就爆破掉了,然后才有动力仔细寻找加密全过程,为今后找其他软件的爆破点做点积累了.为了叙述方便,我将软件中主要函数调用过程都列出来了,当然有些函数不重要,我懒得整理了.函数嵌套调用层次以"-"代表.我想如果你根据我的下面笔记一定能看明白了.文字就叙述的少点了.

-00402E14     .   E8 8CE2FFFF       call vprun.004010A5
--00402736     .   FF15 98A44000     call dword ptr ds:[<&vp2_0.vp::initialize>]       ;  vp2_0.vp::initialize
---1003FAFF      FF15 14030C10       call dword ptr ds:[<&vsgu2_0.vsgu::initialize>]   ; vsgu2_0.vsgu::initialize
----005C5C5C      E8 FFAF0400         call vsgu2_0.CLicenseManager::CLicenseManager
-----00610CE0      E8 CBFCFFFF         call vsgu2_0.CLicenseManager::OpenJob
------00610A45      E8 26950000         call vsgu2_0.lc_new_job
-------00619FB8      E8 72950200         call vsgu2_0.lc_init
--------00643544      E8 05000000         call vsgu2_0.l_init
---------0064366F      E8 8CAF0100         call vsgu2_0.lu_getdtablesize
---------00643814      E8 EC00FFFF         call vsgu2_0.l_getenv
---------00643973      E8 B8B1FEFF         call vsgu2_0.l_more_featdata
---------00643C3D      E8 C3FCFEFF         call vsgu2_0.l_getenv
---------00643F90      E8 999B0100         call vsgu2_0.l_get_attr
---------00643FA8      E8 819B0100         call vsgu2_0.l_get_attr
---------006440E0      E8 0590FFFF         call vsgu2_0.l_getattr_init
---------00644162      E8 098FFFFF         call vsgu2_0.l_getattr
---------006441B1      E8 555CFEFF         call vsgu2_0.l_sg                       ;第一次调用 l_sg
----------00629E83      E8 AB490200         call vsgu2_0.l_key
-----------0064E851      E8 74FFFFFF         call vsgu2_0.l_zinit
---------00644363      E8 088DFFFF         call vsgu2_0.l_getattr
---------0064439C      E8 BD35FDFF         call vsgu2_0.l_set_attr 设置 FLEXLM 版本,此处为 9.0
---------006443F1      E8 0FF5FEFF         call vsgu2_0.l_getenv
---------00644496      E8 6AF4FEFF         call vsgu2_0.l_getenv
---------00644544      E8 BCF3FEFF               call vsgu2_0.l_getenv
---------00644851      E8 1A88FFFF               call vsgu2_0.l_getattr
---------00644A7E      E8 ADE9FFFF               call vsgu2_0.l_add_key_filter
------00610A8D      E8 2E6E0000               call vsgu2_0.lc_set_attr
------00610A98      E8 236E0000               call vsgu2_0.lc_set_attr
------00610AAD      E8 0E6E0000               call vsgu2_0.lc_set_attr
------00610AED      FF15 D4916900             call dword ptr ds:[<&MSVCR71.getenv>>; MSVCR71.getenv
       堆栈压入:0012DE80    00699CE0     \name = "MPI_INSTALL_DEFAULT"
------00610B2D      FF15 88936900             call dword ptr ds:[<&MSVCR71._access>; MSVCR71._access 判断 license 文件存在否
       堆栈压入:0012DE64    0012DE94     |path = "C:\Program Files\MultiGen-Paradigm/flexlmCRO/serverlist.dat"
------00610B84      E8 376D0000               call vsgu2_0.lc_set_attr
------00610B8F      E8 2C6D0000               call vsgu2_0.lc_set_attr
------00610BAB      E8 106D0000               call vsgu2_0.lc_set_attr
------00610BBA      E8 016D0000               call vsgu2_0.lc_set_attr
------00610BC9      E8 F26C0000               call vsgu2_0.lc_set_attr
省略部分函数 call vsgu2_0.lc_set_attr
------00610C44      E8 075F0000               call vsgu2_0.l_flush_config
-------00616B57      E8 D46D0200               call vsgu2_0.l_init_file
--------0063DD36      E8 B15CFDFF               call vsgu2_0.l_free_job_license
--------0063DF8C      E8 DFAB0100               call vsgu2_0.l_open_file
--------0063DFD7      E8 839F0100               call vsgu2_0.l_master_list_lfp
--------0063E1C0      E8 AB7CFFFF               call vsgu2_0.l_allfeat
---------00635EFC      E8 52A9FFFF               call vsgu2_0.l_lfseek
---------00635F1A      E8 F1A8FFFF               call vsgu2_0.l_lfgets
 堆栈数据: ASCII "INCREMENT bb3d_dev MPICRO 2.9 permanent uncounted VENDOR_STRING=D99999:MPI/d>.....
   00635F1F      83C4 14                     add esp,14
  00635F22      85C0                       test eax,eax
  00635F24      0F84 F5010000              je vsgu2_0.0063611F
  上面这段代码决定了程序将循环取 license 文件，并分析每一个 Feature
---------00635FBA      E8 6C010000               call vsgu2_0.l_parse_feature_line
----------006361A2      E8 C9970100               call vsgu2_0.l_parse_decimal
----------00636275      E8 96E4FFFF               call vsgu2_0.l_keyword_eq
----------0063629E      E8 6DE4FFFF               call vsgu2_0.l_keyword_eq 判断 license 文件中是 Feature 还是 Increment
----------006362B0      E8 BB6D0000               call vsgu2_0.l_getattr
----------00636733      FF15 C8926900             call dword ptr ds:[<&MSVCR71.s>; MSVCR71.sscanf
 堆栈数据:00123E68    001248AC     |s = "INCREMENT bb3d_dev MPICRO 2.9 permanent uncounted VENDOR_STRING=D99999:MPI/dk>
    00123E6C    006D5C9C     |format = "%s %s %s %s %s %s %s %[^
 ]
"
        00123E70    00123EE0     ASCII "INCREMENT"
    00123E74    09030FB0     ASCII "bb3d_dev"
    00123E78    090317B1     ASCII "MPICRO"
----------00636768      E8 03E1FFFF               call vsgu2_0.l_keyword_eq_nlicense 文件是否有 start 选项
----------006367BB      E8 E3250100               call vsgu2_0.l_isxdigit
----------00636843      FF15 C8926900             call dword ptr ds:[<&MSVCR71.s>; MSVCR71.sscanf
 堆栈数据:00123E6C    001248AC     |s = "INCREMENT bb3d_dev MPICRO 2.9 permanent uncounted VENDOR_STRING=D99999:MPI/dk>
    00123E70    006D5CC0     |format = "%s %s %s %s %s %s %[^
 ]
"
    00123E74    00123EE0     ASCII "INCREMENT"
    00123E78    09030FB0     ASCII "bb3d_dev"
    00123E7C    090317B1     ASCII "MPICRO"
    00123E80    09031FB2     ASCII "2.9"
    00123E84    090327B3     ASCII "permanent"
    00123E88    09032FB4     ASCII "uncounted"
    00123E8C    09033FB6     ASCII "HOSTID=DEMO SN=demo SIGN="013C 9A5D C920 7271 C073 4D35 52D8 42E8 2528 D201 >
----------006368A0      E8 DE070000               call vsgu2_0.l_valid_version 版本检查
----------006368C7      E8 B1BDFFFF               call vsgu2_0.l_zcp
----------006369FD      E8 0EDDFFFF               call vsgu2_0.l_keyword_eq 是否属于 permenant license 文件
----------00636AE6      E8 85DDFFFF               call vsgu2_0.l_keyword_eq_n
----------00636B98      E8 73DBFFFF               call vsgu2_0.l_keyword_eq 获取节点限制信息
----------00636CBF      E8 DCDE0000               call vsgu2_0.l_uppercase
----------00636EAC      E8 CFDA0100               call vsgu2_0.l_parse_attr
----------0063705F      E8 57030000               call vsgu2_0.006373BB



---

## 汇编代码分析：`l_extract_date` 与 `l_crypt_private`

---------- 006373CC E8 FBE9FEFF call `vsgu2_0.l_extract_date`

堆栈数据: 00123E28 00124770 ASCII "013C 9A5D C920 7271"

---------- 0063740C E8 45000000 call `vsgu2_0.l_crypt_private`

传说中的检查 license 文件是否正确的地方，此处略过不分析了，但是给出函数原型，`F7` 会来到下面的地方：

```
00637456 vs> 55             push ebp
00637457 8BEC               mov ebp,esp
00637459 83EC 08            sub esp,8
0063745C C745 FC 00000000   mov dword ptr ss:[ebp-4],0
00637463 C745 F8 00000000   mov dword ptr ss:[ebp-8],0
0063746A 8B45 14            mov eax,dword ptr ss:[ebp+14]
0063746D 50                 push eax
0063746E 8B4D 10            mov ecx,dword ptr ss:[ebp+10]
00637471 51                 push ecx
00637472 8B55 0C            mov edx,dword ptr ss:[ebp+C]
00637475 52                 push edx
00637476 8B45 08            mov eax,dword ptr ss:[ebp+8]
00637479 50                 push eax
0063747A E8 0D000000        call vsgu2_0.0063748C
0063747F 83C4 10            add esp,10
00637482 8945 FC            mov dword ptr ss:[ebp-4],eax
00637485 8B45 FC            mov eax,dword ptr ss:[ebp-4]
00637488 8BE5               mov esp,ebp
0063748A 5D                 pop ebp
0063748B C3                 retn
```

而 `call vsgu2_0.0063748C` 函数如下：

```
0063748C 55                 push ebp
0063748D 8BEC               mov ebp,esp
0063748F 81EC AC090000      sub esp,9AC
00637495 57                 push edi
00637496 C685 C4F6FFFF 00   mov byte ptr ss:[ebp-93C],0
0063749D B9 4D020000        mov ecx,24D
006374A2 33C0               xor eax,eax
006374A4 8DBD C5F6FFFF      lea edi,dword ptr ss:[ebp-93B]
006374AA F3:AB              rep stos dword ptr es:[edi]
006374AC 66:AB              stos word ptr es:[edi]
006374AE C785 ACF6FFFF 00000000 mov dword ptr ss:[ebp-954],0
006374B8 8D85 C4F6FFFF      lea eax,dword ptr ss:[ebp-93C]
```

---

## 关键调用流程

---------- 00637427 E8 51B2FFFF call `vsgu2_0.l_zcp`
---------- 0063743E E8 012AFEFF call `vsgu2_0.l_err_info_cp`
---------- 0063744A E8 9B29FEFF call `vsgu2_0.l_free_err_info`
-------- 0063E255 E8 E6960100 call `vsgu2_0.l_supersede`
-------- 0063E261 E8 5E880100 call `vsgu2_0.l_post_pkg`
-------- 0063E26F E8 CC960100 call `vsgu2_0.l_supersede`
-------- 0063E2E0 E8 2126FFFF call `vsgu2_0.l_lfclose`

关闭文件

---- 005C5E67 E8 24EDFFFF call `vsgu2_0.vsgu::checkOutRTL`
----- 005C4BC0 E8 1BFFFFFF call `vsgu2_0.vsgu::licenseIsAllowed`
----- 005C4C05 E8 D6FEFFFF call `vsgu2_0.vsgu::licenseIsAllowed`
----- 005C4C1F E8 CCFDFFFF call `vsgu2_0.vsgu::checkOutLicense`

堆栈数据：

- 0012EE3C 0012EE48 ASCII "vsg_rtl"
- 0012EE40 0069A030 ASCII "2.0"
- 0012EE44 0012EEF0 ASCII "C:\Program Files\MultiGen-Paradigm\bin\vprun.exe"

------ 005C4A26 E8 D5C20400 call `vsgu2_0.CLicense::CLicense`

构造相应的 `Feature` 和版本号的 `CLicense` 类

------ 005C4A35 E8 A6CD0400 call `vsgu2_0.CLicense::Checkout`
------- 00611808 E8 53EBFFFF call `vsgu2_0.CLicenseManager::AttachLicense`

下面这段代码我不能不先提一下了：

```
00611843 3BF8             cmp edi,eax
00611845 7F 18            jg short vsgu2_0.0061185F ;注意这里,关键跳转（1）
00611847 8B86 A4160000    mov eax,dword ptr ds:[esi+16A4]
0061184D 0FAFC7           imul eax,edi
00611850 5F               pop edi
00611851 5E               pop esi
00611852 C1E0 02          shl eax,2
00611855 5D               pop ebp
00611856 81C4 54040000    add esp,454
0061185C C2 0400          retn 4
0061185F 57               push edi
00611860 8BCE             mov ecx,esi
```

程序在这里关键跳转（1）要跳了，所以我们暂时不管了。让它跳吧。

------- 00611862 E8 F9FAFFFF call `vsgu2_0.CLicense::DoDisableLicensingCallback`
------- 0061196E E8 AD280000 call `vsgu2_0.MPITrim`
------- 00611993 E8 08F8FFFF call `vsgu2_0.CLicense::Checkout`

注意此处与 `005C4A35` 进入的地方不一样，这就是 `C++` 的特点吧，允许形参不同相同函数名称了。

-------- 006111CC E8 EF660000 call `vsgu2_0.lc_set_attr`

设置 `FEATURE`

-------- 006111ED E8 7E500100 call `vsgu2_0.lc_checkout`
--------- 006262AC E8 04880000 call `vsgu2_0.l_mt_lock`
--------- 006262C0 E8 B11B0700 call `vsgu2_0._setjmp3`
--------- 006262F3 E8 B3000000 call `vsgu2_0.l_checkout`

堆栈数据：

- 0012E944 09029AD8
- 0012E948 0903BF1C ASCII "vsg_rtl"
- 0012E94C 0903BEF4 ASCII "2.0"

---------- 006263E3 E8 95C20000 call `vsgu2_0.l_zcp`
---------- 006263F1 E8 7A6C0100 call `vsgu2_0.l_getattr`
---------- 00626520 E8 689D0100 call `vsgu2_0.l_ckout_borrow`
----------- 00640342 E8 BE35FFFF call `vsgu2_0.l_getenv`
----------- 00640353 E8 18C70000 call `vsgu2_0.l_baddate`
----------- 0064047B FF15 F4916900 call dword ptr ds:[<&MSVCR71.sprintf>]

函数调用后的结果堆栈如下：

- 0012E664 0012E6DC ASCII "borrow-d8f349a2-vsg_rtl"
- 0012E668 006D868C ASCII "borrow-%s-%s"
- 0012E66C 0012E71C ASCII "d8f349a2"
- 0012E670 0903BF1C ASCII "vsg_rtl"

---------- 00626548 FF15 24366E00 call dword ptr ds:[6E3624] ; `vsgu2_0.00626AD5`
----------- 00626B4B E8 33050100 call `vsgu2_0.l_valid_version`
----------- 00626BF9 E8 7FBA0000 call `vsgu2_0.l_zcp`
----------- 00626C58 E8 2700FFFF call `vsgu2_0.l_next_conf_or_marker`
----------- 00626CA7 E8 5C0A0000 call `vsgu2_0.l_local_verify_conf`
----------- 00626E61 E8 89110000 call `vsgu2_0.l_good_lic_key`

小心了，如果是正确的 license 将返回 1，错误的 License 返回 0

------------ 00628096 E8 6B570100 call `vsgu2_0.l_xorname`
------------ 006280B3 E8 531D0000 call `vsgu2_0.l_sg` ; 第二次调用 `L_SG`
------------- 00629E64 FF15 98396E00 call dword ptr ds:[l_n36_buff] ; `vsgu2_0.00624600`
------------- 00628267 E8 E52B0000 call `vsgu2_0.0062AE51` ; 小心了
-------------- 0062AE75 E8 38020000 call `vsgu2_0.0062B0B2` ; 小心了

`F7` 进去看一下，看到没有，发现是不是与前面介绍的 `0063740C call vsgu2_0.l_crypt_private` 代码基本一样啊！！！

```
0062B0B2 55                 push ebp
0062B0B3 8BEC               mov ebp,esp
0062B0B5 81EC AC090000      sub esp,9AC
0062B0BB 57                 push edi
0062B0BC C685 C4F6FFFF 00   mov byte ptr ss:[ebp-93C],0
0062B0C3 B9 4D020000        mov ecx,24D
0062B0C8 33C0               xor eax,eax
```



```
0062B0CA      8DBD C5F6FFFF               lea edi,dword ptr ss:[ebp-93B]
0062B0D0      F3:AB                       rep stos dword ptr es:[edi]
0062B0D2      66:AB                       stos word ptr es:[edi]
0062B0D4      C785 ACF6FFFF 00000000      mov dword ptr ss:[ebp-954],0
0062B0DE      8D85 C4F6FFFF               lea eax,dword ptr ss:[ebp-93C]
```

呵呵,发现了吧,其实这里也是 `l_crypt_private` 传说中的检查 license 文件是否正确的地方了,但是如果你用"搜索-当前模块中的名称"的方法,是不能发现这处参考的.

由于这个函数太重要了,因此我将其关键代码列在此了:

```
---------------0062B3D9      E8 45ADFFFF                 call vsgu2_0.l_good_bin_date
```

经过一系列对日期和序列号,`Vendor Info` 的判断之后

然后关键比较出现了

```
---------------0062BC24      E8 46110000                 call vsgu2_0.0062CD6F

0062BBFB      8B95 C0F6FFFF               mov edx,dword ptr ss:[ebp-940]
0062BC01      52                          push edx                                             ; SN
0062BC02      8B85 A4F6FFFF               mov eax,dword ptr ss:[ebp-95C]
0062BC08      50                          push eax                                             ; 66D8B337
0062BC09      8B4D 14                     mov ecx,dword ptr ss:[ebp+14]
0062BC0C      51                          push ecx
0062BC0D      8B55 FC                     mov edx,dword ptr ss:[ebp-4]
0062BC10      8D85 C4F6FFFF               lea eax,dword ptr ss:[ebp-93C]
0062BC16      2BD0                         sub edx,eax
0062BC18      52                          push edx                                             ; 30
0062BC19      8D8D C4F6FFFF               lea ecx,dword ptr ss:[ebp-93C]
0062BC1F      51                          push ecx
0062BC20      8B55 08                     mov edx,dword ptr ss:[ebp+8]
0062BC23      52                          push edx
0062BC24      E8 46110000                 call vsgu2_0.0062CD6F
```

----------------这里给出 `call vsgu2_0.0062CD6F` 函数的部分分析:

......

```
0062CFF2      8A0A                          mov cl,byte ptr ds:[edx]
0062CFF4      888C05 60FEFFFF                mov byte ptr ss:[ebp+eax-1A0],cl
0062CFFB      8B95 54FEFFFF                  mov edx,dword ptr ss:[ebp-1AC]
0062D001      83C2 01                       add edx,1
0062D004      8995 54FEFFFF                  mov dword ptr ss:[ebp-1AC],edx
0062D00A      83BD 54FEFFFF 02              cmp dword ptr ss:[ebp-1AC],2
0062D011      75 49                         jnz short vsgu2_0.0062D05C
0062D013      C785 54FEFFFF 00000000        mov dword ptr ss:[ebp-1AC],0
0062D01D      8D85 68FEFFFF                  lea eax,dword ptr ss:[ebp-198]
0062D023      50                            push eax
0062D024      68 3C526D00                   push vsgu2_0.006D523C                         ; ASCII "%02X"
0062D029      8D8D 60FEFFFF                 lea ecx,dword ptr ss:[ebp-1A0]
0062D02F      51                            push ecx
0062D030      FF15 C8926900                  call dword ptr ds:[<&MSVCR71.sscanf>]         ; MSVCR71.sscanf
0062D036      83C4 0C                       add esp,0C
0062D039      8B95 50FEFFFF                  mov edx,dword ptr ss:[ebp-1B0]
0062D03F      0395 48FEFFFF                  add edx,dword ptr ss:[ebp-1B8]
0062D045      8A85 68FEFFFF                  mov al,byte ptr ss:[ebp-198]
0062D04B      8802                           mov byte ptr ds:[edx],al
0062D04D      8B8D 48FEFFFF                  mov ecx,dword ptr ss:[ebp-1B8]
0062D053      83C1 01                       add ecx,1
0062D056      898D 48FEFFFF                  mov dword ptr ss:[ebp-1B8],ecx
0062D05C    ^ E9 4CFFFFFF                  jmp vsgu2_0.0062CFAD                           ;循环取 SIGN 的值并转化为 16 进制值
0062D061      8B95 7CFEFFFF                  mov edx,dword ptr ss:[ebp-184]
0062D067      8B82 94000000                  mov eax,dword ptr ds:[edx+94]
0062D06D      50                            push eax
0062D06E      8B8D 7CFEFFFF                  mov ecx,dword ptr ss:[ebp-184]
0062D074      8B91 8C000000                  mov edx,dword ptr ds:[ecx+8C]
0062D07A      52                            push edx
0062D07B      8B85 7CFEFFFF                  mov eax,dword ptr ss:[ebp-184]
0062D081      83C0 14                       add eax,14
0062D084      50                            push eax
0062D085      8B8D 7CFEFFFF                  mov ecx,dword ptr ss:[ebp-184]
0062D08B      83C1 08                       add ecx,8
0062D08E      51                            push ecx
0062D08F      8B95 48FEFFFF                  mov edx,dword ptr ss:[ebp-1B8]
0062D095      52                            push edx
0062D096      8B85 50FEFFFF                  mov eax,dword ptr ss:[ebp-1B0]
0062D09C      50                            push eax
0062D09D      8B4D 10                       mov ecx,dword ptr ss:[ebp+10]
0062D0A0      51                            push ecx
0062D0A1      8B55 0C                       mov edx,dword ptr ss:[ebp+C]
0062D0A4      52                            push edx
0062D0A5      8B45 08                       mov eax,dword ptr ss:[ebp+8]
0062D0A8      50                            push eax
0062D0A9      FF95 64FEFFFF                  call dword ptr ss:[ebp-19C]                  ;l_pubkey_verify 看到没有
0062D0AF      83C4 24                       add esp,24
0062D0B2      8985 4CFEFFFF                  mov dword ptr ss:[ebp-1B4],eax
0062D0B8      8B8D 50FEFFFF                  mov ecx,dword ptr ss:[ebp-1B0]
0062D0BE      51                            push ecx
0062D0BF      E8 D2190000                  call vsgu2_0.l_free
0062D0C4      83C4 04                       add esp,4
0062D0C7      83BD 4CFEFFFF 00              cmp dword ptr ss:[ebp-1B4],0
0062D0CE      74 0C                         je short vsgu2_0.0062D0DC                     ; 关键跳转(2),必须跳
0062D0D0      C785 84FEFFFF 00000000        mov dword ptr ss:[ebp-17C],0
0062D0DA      EB 0A                         jmp short vsgu2_0.0062D0E6
0062D0DC      C785 84FEFFFF 44526D00        mov dword ptr ss:[ebp-17C],vsgu2_0.006D5244  ; ASCII "ok"
0062D0E6      E9 D30C0000                  jmp vsgu2_0.0062DDBE
```

-----------------关键函数 `l_pubkey_verify` 如果是正确的 license 将返回 0,错误的 License 返回 `FFFFFFF8`,呵呵,很熟悉吧,分析如下:

```
------------------006464B3      E8 A8850100                 call vsgu2_0.sb_dataSize
------------------00646509      E8 B2840100                 call vsgu2_0.sb_heapSize
------------------006465A3      E8 A8810100                 call vsgu2_0.sb_initialize
------------------006465F1      E8 1A8B0100                 call vsgu2_0.sb_ecdsaVerifyBegin
------------------00646681      E8 2A870100                 call vsgu2_0.sb_ecdsaSign
------------------006466D9      E8 D2860100                 call vsgu2_0.sb_ecdsaSign
------------------00646728      E8 538A0100                 call vsgu2_0.sb_ecdsaVerifyEnd
0064672D      83C4 14                       add esp,14
00646730      8945 AC                       mov dword ptr ss:[ebp-54],eax
00646733      837D AC 00                    cmp dword ptr ss:[ebp-54],0
00646737      74 1E                         je short vsgu2_0.00646757
00646739      8B4D AC                       mov ecx,dword ptr ss:[ebp-54]
0064673C      51                            push ecx
0064673D      68 30290000                   push 2930
00646742      8B55 08                       mov edx,dword ptr ss:[ebp+8]
00646745      52                            push edx
00646746      E8 40F1FFFF                  call vsgu2_0.l_pubkey_err
0064674B      83C4 0C                       add esp,0C
0064674E      C745 B0 8DFFFFFF              mov dword ptr ss:[ebp-50],-73
00646755      EB 5E                         jmp short vsgu2_0.006467B5
00646757      837D 9C 00                    cmp dword ptr ss:[ebp-64],0
```



## 汇编代码段 1

```
0064675B     75 58                 jnz short vsgu2_0.006467B5        ; 关键跳转(3),必须跳

0064675D     8B45 08               mov eax,dword ptr ss:[ebp+8]

00646760     8378 14 00            cmp dword ptr ds:[eax+14],0

00646764     74 14                 je short vsgu2_0.0064677A

00646766     33C9                  xor ecx,ecx

00646768     85C9                  test ecx,ecx

0064676A     74 0E                 je short vsgu2_0.0064677A

0064676C     8B55 08               mov edx,dword ptr ss:[ebp+8]

0064676F     8B42 14               mov eax,dword ptr ds:[edx+14]

00646772     8985 10FFFFFF        mov dword ptr ss:[ebp-F0],eax

00646778     EB 0A                jmp short vsgu2_0.00646784

0064677A     C785 10FFFFFF F8FFFFFF mov dword ptr ss:[ebp-F0],-8

00646784     8B4D 08               mov ecx,dword ptr ss:[ebp+8]

00646787     8B95 10FFFFFF        mov edx,dword ptr ss:[ebp-F0]

0064678D     8951 14               mov dword ptr ds:[ecx+14],edx

00646790     6A 00                 push 0

00646792     68 FF000000          push 0FF

00646797     6A 00                 push 0

00646799     6A 00                 push 0

0064679B     68 14020000          push 214

006467A0     6A F8                 push -8

006467A2     8B45 08               mov eax,dword ptr ss:[ebp+8]

006467A5     50                    push eax

006467A6     E8 E533FDFF          call vsgu2_0.l_set_error

006467AB     83C4 1C             add esp,1C

006467AE     C745 B0 F8FFFFFF     mov dword ptr ss:[ebp-50],-8

006467B5     8B45 B0               mov eax,dword ptr ss:[ebp-50]

006467B8     8BE5                 mov esp,ebp

006467BA     5D                    pop ebp

006467BB     C3                    retn
```

## 说明

下面经过一堆代码直到 `call vsgu2_0.l_good_lic_key` 返回了，如果是正确的 license 将返回 1，错误的 License 返回 0。

如果前面爆破后的代码：

```
-----------00626E90     E8 B0DDFEFF          call vsgu2_0.l_featon
-----------00626EB6     E8 85D7FEFF          call vsgu2_0.l_check_conf
-----------0062735B     E8 1DB30000          call vsgu2_0.l_zcp
-----------006273A4     E8 67D30000          call vsgu2_0.l_keyword_eq
-----------0062740E     E8 87B70000          call vsgu2_0.l_update_license_file
---------00626308     E8 813AFFFF          call vsgu2_0.l_clear_error
---------0062639C     E8 0F870000          call vsgu2_0.l_mt_unlock
--------0061121E     E8 C3480000          call vsgu2_0.lc_auth_data
--------00611264     E8 73470100          call vsgu2_0.lc_expire_days
-------
```

## 汇编代码段 2

```
00611998     8BE8                 mov ebp,eax

0061199A     85ED                 test ebp,ebp

0061199C     7D 7F                jge short vsgu2_0.00611A1D ;关键跳转(4),必须跳转.

0061199E     85FF                 test edi,edi

006119A0     74 19                je short vsgu2_0.006119BB

006119A2     803F 00              cmp byte ptr ds:[edi],0

006119A5     74 14                je short vsgu2_0.006119BB

006119A7     6A 01                push 1

006119A9     6A 00                push 0

006119AB     8D4424 1C            lea eax,dword ptr ss:[esp+1C]

006119AF     50                   push eax

006119B0     57                   push edi

006119B1     E8 3A290000          call vsgu2_0.MPIAddToBuf

006119B6     83C4 10              add esp,10

006119B9     8BF8                 mov edi,eax

006119BB     53                   push ebx

006119BC     E8 80850000          call vsgu2_0.lc_err_info

006119C1     50                   push eax

006119C2     E8 A9250000          call vsgu2_0.BuildErrorString

006119C7     6A 01                push 1

006119C9     50                   push eax

006119CA     8D4C24 24            lea ecx,dword ptr ss:[esp+24]

006119CE     51                   push ecx

006119CF     57                   push edi

006119D0     E8 1B290000          call vsgu2_0.MPIAddToBuf

006119D5     83C4 18              add esp,18

006119D8     83FD FC              cmp ebp,-4

006119DB     8BF8                 mov edi,eax

006119DD     74 05                je short vsgu2_0.006119E4

006119DF     83FD E8              cmp ebp,-18

006119E2     75 3E                jnz short vsgu2_0.00611A22

006119E4     85FF                 test edi,edi

006119E6     74 19                je short vsgu2_0.00611A01

006119E8     803F 00              cmp byte ptr ds:[edi],0

006119EB     74 14                je short vsgu2_0.00611A01

006119ED     6A 01                push 1

006119EF     6A 00                push 0

006119F1     8D5424 1C            lea edx,dword ptr ss:[esp+1C]

006119F5     52                   push edx

006119F6     57                   push edi

006119F7     E8 F4280000          call vsgu2_0.MPIAddToBuf

006119FC     83C4 10              add esp,10

006119FF     8BF8                 mov edi,eax

00611A01     8B4424 20            mov eax,dword ptr ss:[esp+20]

00611A05     50                   push eax

00611A06     53                   push ebx

00611A07     8D5C24 1C            lea ebx,dword ptr ss:[esp+1C]

00611A0B     8BC7                 mov eax,edi

00611A0D     E8 4EF5FFFF          call vsgu2_0.00610F60

00611A12     8B5C24 24            mov ebx,dword ptr ss:[esp+24]

00611A16     83C4 08              add esp,8

00611A19     8BF8                 mov edi,eax

00611A1B     EB 05                jmp short vsgu2_0.00611A22

00611A1D     C64424 13 01        mov byte ptr ss:[esp+13],1

00611A22     68 589C6900          push vsgu2_0.00699C58

00611A27     6A 00                push 0

00611A29     FF15 04926900        call dword ptr ds:[<&MSVCR71.strtok>]    ; MSVCR71.strtok

00611A2F     8A4C24 1B            mov cl,byte ptr ss:[esp+1B]

00611A33     83C4 08              add esp,8

00611A36     84C9                 test cl,cl

00611A38     ^ 0F84 22FFFFFF      je vsgu2_0.00611960

00611A3E     EB 50                jmp short vsgu2_0.00611A90

00611A40     8A4424 13            mov al,byte ptr ss:[esp+13]

00611A44     84C0                 test al,al

00611A46     75 48                jnz short vsgu2_0.00611A90

00611A48     85FF                 test edi,edi

00611A4A     74 18                je short vsgu2_0.00611A64

00611A4C     8BC7                 mov eax,edi

00611A4E     8D50 01              lea edx,dword ptr ds:[eax+1]

00611A51     8A08                 mov cl,byte ptr ds:[eax]

00611A53     40                   inc eax

00611A54     84C9                 test cl,cl

00611A56     ^ 75 F9              jnz short vsgu2_0.00611A51

00611A58     2BC2                 sub eax,edx

00611A5A     894424 1C            mov dword ptr ss:[esp+1C],eax

00611A5E     74 04                je short vsgu2_0.00611A64

00611A60     8BC7                 mov eax,edi
```



```
00611A62      EB 0F                     jmp short vsgu2_0.00611A73
00611A64      53                        push ebx
00611A65      E8 D7840000               call vsgu2_0.lc_err_info
00611A6A      50                        push eax
00611A6B      E8 00250000               call vsgu2_0.BuildErrorString
00611A70      83C4 08                   add esp,8
00611A73      8B8E 38170000             mov ecx,dword ptr ds:[esi+1738]
00611A79      8B19                      mov ebx,dword ptr ds:[ecx]
00611A7B      8D96 84120000             lea edx,dword ptr ds:[esi+1284]
00611A81      52                        push edx
00611A82      68 2CD66A00               push vsgu2_0.006AD62C               ; ASCII "Cannot check out feature "%s" because:
00611A87      50                        push eax
00611A88      55                        push ebp
00611A89      51                        push ecx
00611A8A      FF53 04                   call dword ptr ds:[ebx+4]
00611A8D      83C4 14                   add esp,14
00611A90      8BCE                      mov ecx,esi
00611A92      E8 09FAFFFF               call vsgu2_0.CLicense::setHostIds
00611A97      8BCE                      mov ecx,esi
00611A99      E8 72FAFFFF               call vsgu2_0.CLicense::setHostName
00611A9E      8BCE                      mov ecx,esi
00611AA0      E8 CBFAFFFF               call vsgu2_0.CLicense::setHostUser
00611AA5      8BCE                      mov ecx,esi
00611AA7      E8 24FBFFFF               call vsgu2_0.CLicense::setHostDiskID
00611AAC      8BCE                      mov ecx,esi
00611AAE      E8 7DFBFFFF               call vsgu2_0.CLicense::setHostDisplay
00611AB3      8BCE                      mov ecx,esi
00611AB5      E8 D6FBFFFF               call vsgu2_0.CLicense::setHostCPU
00611ABA      8BCE                      mov ecx,esi
00611ABC      E8 2FFCFFFF               call vsgu2_0.CLicense::setHostEther
00611AC1      8BCE                      mov ecx,esi
00611AC3      E8 88FCFFFF               call vsgu2_0.CLicense::setHostIP
00611AC8      8BCE                      mov ecx,esi
00611ACA      E8 E1FCFFFF               call vsgu2_0.CLicense::setLicenseFile
......
RETN

----005C4C2F      E8 74B00400             call vsgu2_0.__security_check_cookie
----005C5EB2      E8 19FBFFFF             call vsgu2_0.vsgu::initializeClassSessio>
---1003FB31      FF15 ACF40B10           call dword ptr ds:[<&vsgc2_0.vsgc::initialize>]   ; vsgc2_0.vsgc::initialize
```

呵呵，`NAG` 在这里了。

下面开始对下一个 `FEATURE` 进行 `License` 检查了。流程一样的，我就不分析了。到此，基本上将该软件 `FLEXLM9.0` 加密的全流程跟踪一遍了。

写到这，好累好累。

回到前面留下的关键跳转（1）处，其实这里是软件判断到底需不需要对该 `Feature` 进行检查了。程序中肯定是部分软件需要部分 `Feature` 的支持了。但这里给爆破者留下来的想象空间太大了，而且太容易了。

爆破点上面都列出来了。如果你想用该软件可以参考一下了。但我负其他责任哦。

### 总结
爆破 `FlexLM` 加密主要应该是分析清楚其加密流程，发现重点函数，然后关键点爆破。
对于关键函数要敏感了，而且其特征要自己整理出来，这样方便下个软件的定位了。

### 问题
在 `sln` 翻译的 `" FLEXlm latest information by CrackZ"` 介绍到了 `"对于新长度的 SIGN，破解者会选择对_lm_pubkey_verify()打补丁然后用自己的 LM_SEED 制作 SIGN=许可证。然而，可以用另外一种方法，就是强制许可授权层(licensing layer)进行旧格式的 SIGN=12 字符的校验方式。该法仅涉及简单地对我们发现的存在于_l_n36_buf()(如上)里面的第 2 个坏标记处补丁。我们可以象以前那样回收到加密过的 SEED 并用 SDK 或本站可下载的 Lmgryptgui 制作 license。该补丁的工作原理是告诉许可授权层(licensing layer)别去取_lm_pubkey_verify()的地址，_lm_pubkey_verify()会在_l_sg()之后马上就行校验。"` 这个办法，我没有试验成功，高手能够详细解释一下吗!!!

### 感谢
学习过程中大量参考了 `laoqian`、`sln`、`newsearch` 的文章。
感谢看雪上所有人。

2007 年 4 月 12 日
`4dd466d0e1e0cc123afbe504da2462ca`

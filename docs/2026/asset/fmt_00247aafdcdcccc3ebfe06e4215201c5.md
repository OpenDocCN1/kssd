
# Multigen vega prime 2.0 详细跟踪

## 软件信息

【软件名称】`Multigen vega prime 2.0`

【应用平台】`Win2000 SP4`

【作者邮箱】`chubing6143@sina.com`

【使用工具】`OllyDbg1.10`，`FlexLM 7.2 SDK`

【软件限制】`FLEXlm v9.2.0` 需要120位长度的`SIGN`

【软件简介】大名鼎鼎的视景仿真软件,具体我就不介绍了.

【关于本文】呵呵,由于这个软件是采用的120位长度的`SIGN`的`FLEXLM9.2`的加密,如果要想产生真正的`License`文件,我没有这个水平了.而如果要想爆破的话,很简单了.但由于我想仔细跟踪一下该软件的`FlexLM`加密的具体过程,呵呵,我是菜鸟,这下可费劲了,经过几个日夜不停的工作,加上资料的消化,我终于有了点感觉了.因此,写出来希望给我这样的菜鸟有一定的帮助.

前几天,师弟毕业,大家喝高了,教授(62岁)临别赠言"扶摇直上九万里",多么豪迈啊,想想我自己,虽然也老大不小了.但是和他比起来,我还年轻,可心已经没劲了.激励自己一下,该来个"扶摇直上九厘米"了啊!呵呵,跑题了啊!

补充说明,最近一直和自己说要好好干点正事了(毕业压力好大),因此手头一直只有`FlexLM 7.2 SDK`,没有9.2的`SDK`,因此,自己分析全靠死功夫了,有错的地方,请大家指点一下啊!

## 跟踪过程

利用`LMTOOLS`可以发现该软件是采用`FLEXlm v9.2.0` (`lmgr.lib`)加密的,而加密的文件是`vsgu2_0.dll`,而软件中主要限制就是运行时检查`License`文件,我刚开始直接利用`OD`加载`LynxPrime.exe`分析,可是总断不下来,后来仔细一看,运行部分其实是`vprun.exe`,利用`OD`加载它,逐步分析,很快就爆破掉了,然后才有动力仔细寻找加密全过程,为今后找其他软件的爆破点做点积累了.为了叙述方便,我将软件中主要函数调用过程都列出来了,当然有些函数不重要,我懒得整理了.函数嵌套调用层次以"-"代表.我想如果你根据我的下面笔记一定能看明白了.文字就叙述的少点了.

```
-00402E14     .  E8 8CE2FFFF       call vprun.004010A5
--00402736     .  FF15 98A44000     call dword ptr ds:[<&vp2_0.vp::initialize>]       ;  vp2_0.vp::initialize
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
---------006441B1      E8 555CFEFF         call vsgu2_0.l_sg                          ;第一次调用l_sg
----------00629E83      E8 AB490200         call vsgu2_0.l_key
-----------0064E851      E8 74FFFFFF         call vsgu2_0.l_zinit
---------00644363      E8 088DFFFF         call vsgu2_0.l_getattr
---------0064439C      E8 BD35FDFF         call vsgu2_0.l_set_attr       设置FLEXLM版本,此处为9.0
---------006443F1      E8 0FF5FEFF         call vsgu2_0.l_getenv
---------00644496      E8 6AF4FEFF         call vsgu2_0.l_getenv
---------00644544      E8 BCF3FEFF                call vsgu2_0.l_getenv
---------00644851      E8 1A88FFFF                call vsgu2_0.l_getattr
---------00644A7E      E8 ADE9FFFF                call vsgu2_0.l_add_key_filter
------00610A8D      E8 2E6E0000                call vsgu2_0.lc_set_attr
------00610A98      E8 236E0000                call vsgu2_0.lc_set_attr
------00610AAD      E8 0E6E0000                call vsgu2_0.lc_set_attr
------00610AED      FF15 D4916900              call dword ptr ds:[<&MSVCR71.getenv>>; MSVCR71.getenv
      堆栈压入:0012DE80    00699CE0     \name = "MPI_INSTALL_DEFAULT"
------00610B2D      FF15 88936900              call dword ptr ds:[<&MSVCR71._access>; MSVCR71._access判断license文件存在否
      堆栈压入:0012DE64    0012DE94     |path = "C:\Program Files\MultiGen-Paradigm/flexlmCRO/serverlist.dat"
------00610B84      E8 376D0000                call vsgu2_0.lc_set_attr
------00610B8F      E8 2C6D0000                call vsgu2_0.lc_set_attr
------00610BAB      E8 106D0000                call vsgu2_0.lc_set_attr
------00610BBA      E8 016D0000                call vsgu2_0.lc_set_attr
------00610BC9      E8 F26C0000                call vsgu2_0.lc_set_attr
省略部分函数call vsgu2_0.lc_set_attr
------00610C44      E8 075F0000                call vsgu2_0.l_flush_config
-------00616B57      E8 D46D0200                call vsgu2_0.l_init_file
--------0063DD36      E8 B15CFDFF                call vsgu2_0.l_free_job_license
--------0063DF8C      E8 DFAB0100                call vsgu2_0.l_open_file
--------0063DFD7      E8 839F0100                call vsgu2_0.l_master_list_lfp
--------0063E1C0      E8 AB7CFFFF                call vsgu2_0.l_allfeat
---------00635EFC      E8 52A9FFFF                call vsgu2_0.l_lfseek
---------00635F1A      E8 F1A8FFFF                call vsgu2_0.l_lfgets
  堆栈数据: ASCII "INCREMENT bb3d_dev MPICRO 2.9 permanent uncounted VENDOR_STRING=D99999:MPI/d>.....
          00635F1F      83C4 14                    add esp,14
        00635F22      85C0                       test eax,eax
        00635F24      0F84 F5010000              je vsgu2_0.0063611F
        上面这段代码决定了程序将循环取license文件，并分析每一个Feature
---------00635FBA      E8 6C010000                call vsgu2_0.l_parse_feature_line
----------006361A2      E8 C9970100                call vsgu2_0.l_parse_decimal
----------00636275      E8 96E4FFFF                call vsgu2_0.l_keyword_eq
----------0063629E      E8 6DE4FFFF                call vsgu2_0.l_keyword_eq判断license文件中是Feature还是Increment
----------006362B0      E8 BB6D0000                call vsgu2_0.l_getattr
----------00636733      FF15 C8926900              call dword ptr ds:[<&MSVCR71.s>; MSVCR71.sscanf
  堆栈数据:00123E68    001248AC     |s = "INCREMENT bb3d_dev MPICRO 2.9 permanent uncounted VENDOR_STRING=D99999:MPI/dk>
           00123E6C    006D5C9C     |format = "%s %s %s %s %s %s %s %[^
]
"
           00123E70    00123EE0     ASCII "INCREMENT"
           00123E74    09030FB0     ASCII "bb3d_dev"
           00123E78    090317B1     ASCII "MPICRO"
----------00636768      E8 03E1FFFF                call vsgu2_0.l_keyword_eq_nlicense文件是否有start选项
----------006367BB      E8 E3250100                call vsgu2_0.l_isxdigit
----------00636843      FF15 C8926900              call dword ptr ds:[<&MSVCR71.s>; MSVCR71.sscanf
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
----------006368A0      E8 DE070000                call vsgu2_0.l_valid_version版本检查
----------006368C7      E8 B1BDFFFF                call vsgu2_0.l_zcp
----------006369FD      E8 0EDDFFFF                call vsgu2_0.l_keyword_eq是否属于permenant license文件
----------00636AE6      E8 85DDFFFF                call vsgu2_0.l_keyword_eq_n
----------00636B98      E8 73DBFFFF                call vsgu2_0.l_keyword_eq获取节点限制信息
----------00636CBF      E8 DCDE0000                call vsgu2_0.l_uppercase
----------00636EAC      E8 CFDA0100                call vsgu2_0.l_parse_attr
----------0063705F      E8 57030000                call vsgu2_0.006373BB
-----------006373CC      E8 FBE9FEFF                call vsgu2_0.l_extract_date
  堆栈数据: 00123E28    00124770     ASCII "013C 9A5D C920 7271 "
-----------0063740C      E8 45000000                call vsgu2_0.l_crypt_private传说中的检查license文件是否正确的地方,此处略过不分析了,但是给出函数原型,F7会来到下面的地方:
00637456 vs>  55                         push ebp
00637457      8BEC                       mov ebp,esp
00637459      83EC 08                    sub esp,8
0063745C      C745 FC 00000000           mov dword ptr ss:[ebp-4],0
00637463      C745 F8 00000000           mov dword ptr ss:[ebp-8],0
```

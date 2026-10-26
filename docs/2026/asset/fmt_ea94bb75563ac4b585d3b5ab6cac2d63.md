
# [原创]CoolCmd接入教程(某游戏协议破解及完整可编译插件源码)

20151012 支持插件的免费版已放出 需要的可进QQ交流群下载版本和插件

------------------------------------20150921新加Begin--------------------------------------------------------------------

新活动,20150921开始,任何人根据此示例接入一款协议加密的游戏,并提交插件给官方,可获得一个永久授权

须进交流QQ群`101564837`,联系官方确认游戏,防止重复.

------------------------------------20150921新加End----------------------------------------------------------------------

## 前言

本有个通杀多款几十万在线游戏的插件教程,效果也不错,奈何多种原因不便公开.
So决定用个国产精品游戏做示例,物色了剑网3,又奈何游戏是收费模式,为了让感兴趣人员可以实际练习,选用剑侠世界2(清爽版)做示例.
任何有逆向和开发基础人员，学习此文，均可自主接入任何网游或其他网络软件,且近期将有机会靠接入必得工具永久授权@千游斩活动(一个工具破解市面所有网游还会远吗?).

`CoolCmd`是什么?  -->  http://bbs.pediy.com/showthread.php?t=203487

`CoolCmd`挖掘游戏漏洞?  -->  http://bbs.pediy.com/showthread.php?t=203777

`CoolCmd`交流Q群  -->  `101564837`

## 一、协议破解

### 1.下载运行游戏,登录账号,进入创建角色界面,OD附加之

选用创建角色协议的优势:

a.此时封包少,便于无干扰分析

b.已知加密前明文(创建的角色名),便于对照分析

### 2.堆栈回溯法回溯明文发包函数

⑴`bp send`下断点

⑵输入角色名：加密解密,点进入游戏,OD断下

![](https://bbs.kanxue.com/upload/attach/201509/690259_dtfuwtjad9oy1ao.jpg)

⑶OD Ctrl + F9 4次即可定位到明文发包函数

汇编代码:

```
       00565405    8B45 FC         mov     eax, dword ptr [ebp-4]
       00565408    50              push    eax				                 //压入参数
       00565409    8B4D F4         mov     ecx, dword ptr [ebp-C]
       0056540C    8B51 28         mov     edx, dword ptr [ecx+28]
       0056540F    8B45 F4         mov     eax, dword ptr [ebp-C]
       00565412    8B48 28         mov     ecx, dword ptr [eax+28]	//寄存器参数
       00565415    8B12            mov     edx, dword ptr [edx]
       00565417    8B42 14         mov     eax, dword ptr [edx+14]
       0056541A    FFD0            call    eax   ; //eax = 0x008551D0	即明文发包函数
       0056541C    8945 F8         mov     dword ptr [ebp-8], eax
       0056541F    8B4D FC         mov     ecx, dword ptr [ebp-4]
       00565422    8B11            mov     edx, dword ptr [ecx]
       00565424    8B45 FC         mov     eax, dword ptr [ebp-4]
       00565427    50              push    eax
       00565428    8B4A 08         mov     ecx, dword ptr [edx+8]
       0056542B    FFD1            call    ecx
       0056542D    837D F8 00      cmp     dword ptr [ebp-8], 0
       00565431    75 23           jnz     short 00565456
       00565433    68 A8E7A800  push    00A8E7A8         //SendMsg2GateWay
```

(对`0x008551D0`函数头、尾以及`send`同时下断点,可确认此函数同时具备加密和发送功能)

![](https://bbs.kanxue.com/upload/attach/201509/690259_aswabnskhsukjy1.jpg)

分析下此发包函数,为多语言通用,设定函数原型为:

`typedef DWORD (__stdcall *TMMSendProc)(PSendParamSt pParam);`

参数1         压栈参数        包含明文信息的结构体指针

对应结构体如下:

```
      typedef struct _SendParamSt
      {
				DWORD dwMMSend_A;	//参数A,固定基址
				DWORD dwMMSend_B;	//参数B,固定基址
				DWORD dwUnknow;	//未知,默认1
				DWORD dwLen0;		//协议长度
				DWORD dwLen;		//协议长度
				DWORD dwBuff;		//协议数据内存地址
       }TSendParamSt,*PSendParamSt;
```

内存断点分析`dwMMSend_A`,`dwMMSend_B`来源

`dwMMSend_A = 0x00ABE510`

`dwMMSend_B = 0x00ABE4F8`

对应汇编代码:

```
       0085A88A    C700 10E5AB00      MOV DWORD PTR DS:[EAX],game.00ABE510
       0085A890  C740 04 F8E4AB00   MOV DWORD PTR DS:[EAX+4],game.00ABE4F8
       0085A897    C740 08 01000000   MOV DWORD PTR DS:[EAX+8],1
       0085A89E    8948 14            MOV DWORD PTR DS:[EAX+14],ECX
       0085A8A1    5B                 POP EBX
       0085A8A2    C3                 RETN
```

参数2  寄存器`ECX`        加密Key和加密函数相关的指针

内存断点分析`Ecx`对应基址和偏移

基址`g_Adr_LoginKey = 0xB65FE4`

对应汇编代码:

```
       00428657    83F8 0A         CMP EAX,0A
       0042865A    75 07           JNZ SHORT game.00428663
       0042865C    33C0            XOR EAX,EAX
       0042865E    E9 A5040000     JMP game.00428B08
       00428663    A1 E45FB600     MOV EAX,DWORD PTR DS:[B65FE4]//B65FE4即基址
       00428668    8B10            MOV EDX,DWORD PTR DS:[EAX]
       0042866A    8B0D E45FB600   MOV ECX,DWORD PTR DS:[B65FE4]
       00428670    8B42 10         MOV EAX,DWORD PTR DS:[EDX+10]
       00428673    FFD0            CALL NEAR EAX
```

偏移`g_Oft_LoginKey1= 0x28`

对应汇编代码:

```
       0056540C    8B51 28         mov     edx, dword ptr [ecx+28]  //0x28偏移
       0056540F    8B45 F4         mov     eax, dword ptr [ebp-C]
       00565412    8B48 28         mov     ecx, dword ptr [eax+28]
```

`ECX = *(PDWORD)(*(PDWORD)(g_Adr_LoginKey) + g_Oft_LoginKey1);`

(此游戏,游戏场景内的Ke使用另外的基址和偏移,分析过程类似,此处直接贴出)

基址`g_Adr_SceneKey = 0x1C285F0`

对应汇编代码:

```
       005977B4    8B0D F085C201   MOV ECX,DWORD PTR DS:[1C285F0]//1C285F0
       005977BA    8B11            MOV EDX,DWORD PTR DS:[ECX]
       005977BC    8B0D F085C201   MOV ECX,DWORD PTR DS:[1C285F0]
       005977C2    8B02            MOV EAX,DWORD PTR DS:[EDX]
       005977C4    FFD0            CALL NEAR EAX
```

偏移`g_Oft_SceneKey1 = 0x30;`

对应汇编代码:

```
       00529F20    8B51 30         MOV EDX,DWORD PTR DS:[ECX+30]
       00529F23    8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]
       00529F26    8B48 30         MOV ECX,DWORD PTR DS:[EAX+30]
```

`ECX = *(PDWORD)(*(PDWORD)(g_Adr_SceneKey) + g_Oft_SceneKey1);`

明文发包函数接口和参数分析完毕

### 3.内存断点追溯明文收包函数

`bp recv`,过程类似,此处略去

函数原型:

`typedef DWORD (__cdecl *TMMRecvProc)(DWORD dwLen,DWORD dwBuff,DWORD dwParam);`

参数1：        `dwParam`        指向解密Key

参数2:        `dwBuff`        存储接受的数据(解密前后都存于此)

参数3:         `dwLen`        接收的数据长度(解密前后等长)

对应汇编代码:

```
       0085B200    8B4424 0C          MOV EAX,DWORD PTR SS:[ESP+C] //明文收包函数
       0085B204    8B08               MOV ECX,DWORD PTR DS:[EAX]
       0085B206    8B4424 08          MOV EAX,DWORD PTR SS:[ESP+8]
       0085B20A    8D5424 0C          LEA EDX,DWORD PTR SS:[ESP+C]
       0085B20E    52                 PUSH EDX
       0085B20F    894C24 10          MOV DWORD PTR SS:[ESP+10],ECX
       0085B213    8B4C24 08          MOV ECX,DWORD PTR SS:[ESP+8]
       0085B217    50                 PUSH EAX
       0085B218    51                 PUSH ECX
       0085B219    E8 52FFFFFF        CALL game.0085B170
       0085B21E    83C4 0C            ADD ESP,0C
       0085B221    C3                 RETN
```

明文收包函数接口和参数分析完毕

## 二、编写CoolCmd插件

只需把分析的内容填充到插件代码中即可,核心代码如下:

VC版代码:


## SDPI:0047F04C 至 0047F058

```
SDPI:0047F04C                 jno     short loc_47F056            ; 清场处理:-),结束了跳去 N 个 Ret,然后返回到程序 oep

SDPI:0047F04E                 scasb

SDPI:0047F04F                 mov     al, 42h

SDPI:0047F051                 add     eax, [eax]

SDPI:0047F053                 adc     [eax+0], al

SDPI:0047F056

SDPI:0047F056 loc_47F056:                              ; CODE XREF: sub_47F043+7 j

SDPI:0047F056                                 ; sub_47F043+9 j

SDPI:0047F056             rep stosb                     ; 清场处理:-),结束了跳去 N 个 Ret,然后返回到程序 oep

SDPI:0047F058             retn
```

***以下是各模块***

## SDPI:0047F061 至 0047F070

```
SDPI:0047F061

SDPI:0047F061 ; ************** S U B R O U T I N E **********************

SDPI:0047F061

SDPI:0047F061 ; 获取实际地址

SDPI:0047F061

SDPI:0047F061 ; int __stdcall GetAbsAddress(int relativeAddress)

SDPI:0047F061 GetAbsAddress   proc near                     ; CODE XREF: GetFileMD5Val_47F542+3D p

SDPI:0047F061                                 ; sub_47F6AA+22 p ...

SDPI:0047F061

SDPI:0047F061 relativeAddress = dword ptr  4

SDPI:0047F061

SDPI:0047F061             call    $+5

SDPI:0047F066             pop     eax

SDPI:0047F067             sub     eax, 406066h

SDPI:0047F06C             add     eax, [esp+relativeAddress]

SDPI:0047F070             retn    4

SDPI:0047F070 GetAbsAddress   endp

SDPI:0047F070

SDPI:0047F070 ; ----------------------------------------------------------------------
```

……

## SDPI:0047F077 至 0047F083

```
SDPI:0047F077 ; ************** S U B R O U T I N E **********************

SDPI:0047F077

SDPI:0047F077

SDPI:0047F077 Get_BASE        proc near                     ; CODE XREF: CRC_480467+6 p

SDPI:0047F077                                 ; CRC_480467+1A p ...

SDPI:0047F077             call    loc_47F07D

SDPI:0047F07C             nop

SDPI:0047F07D

SDPI:0047F07D loc_47F07D:                              ; CODE XREF: Get_BASE p

SDPI:0047F07D             pop     eax

SDPI:0047F07E             add     eax, 0FFFFB084h

SDPI:0047F083             retn

SDPI:0047F083 Get_BASE        endp ; sp =  4

SDPI:0047F083

SDPI:0047F083 ; ----------------------------------------------------------------------
```

……

## SDPI:0047F088 至 0047F097

```
SDPI:0047F088 ; ************** S U B R O U T I N E **********************

SDPI:0047F088

SDPI:0047F088

SDPI:0047F088 Decode_47F088   proc near                     ; CODE XREF: CRC_480467+89 p

SDPI:0047F088                                 ; MD5CHK_4813E4+2A p ...

SDPI:0047F088             pop     eax

SDPI:0047F089             push    eax                       ; EDX

SDPI:0047F08A             add     eax, 64h

SDPI:0047F08D             mov     edx, eax

SDPI:0047F08F             add     edx, 32h

SDPI:0047F092             call    Crypt_Decrypt_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)

SDPI:0047F092                                 ; 用于加密或解密代码,加解密操作是互换的

SDPI:0047F092                                 ; 是只简单的 xor,因此可以用来加密也可以

SDPI:0047F092                                 ; 用来解密

SDPI:0047F097             retn

SDPI:0047F097 Decode_47F088   endp ; sp =  8

SDPI:0047F097

SDPI:0047F097 ; ----------------------------------------------------------------------------
```

连他的 `Information` 都抓 出来的说.

```
SDPI:0047F098             dd 62AEF34h

SDPI:0047F09C aDebuggerDetect db 'Debugger detected  - please close it down and restart!',0Dh,0Ah

SDPI:0047F09C             db 0Dh,0Ah

SDPI:0047F09C             db 'For some debuggers, such as SoftIce, you must restart ',0Dh,0Ah

SDPI:0047F09C             db 'this machine without it enabled to run this application!',0Dh

SDPI:0047F09C             db 0Ah,0

SDPI:0047F149 aExit___        db 'Exit...',0Dh,0Ah,0

SDPI:0047F153 aThisProgramIsProtec db 'This program is protected by unregistered version of SoftDef'

SDPI:0047F153             db 'ender',0Dh,0Ah

SDPI:0047F153             db 0Dh,0Ah

SDPI:0047F153             db 'This message will not appear on programs protected by a regi'

SDPI:0047F153             db 'stered ',0Dh,0Ah

SDPI:0047F153             db 'version of SoftDefender, please register it from:',0Dh,0Ah

SDPI:0047F153             db 0Dh,0Ah

SDPI:0047F153             db '                   http://www.softdefender.com/order.htm',0Dh,0Ah

SDPI:0047F153             db 0

SDPI:0047F24A aUnregisteredVersion db 'Unregistered Version',0Dh,0Ah,0

SDPI:0047F261

SDPI:0047F261 ; ************** S U B R O U T I N E **********************

SDPI:0047F261

SDPI:0047F261 ; 检测调试器是否存在

SDPI:0047F261

SDPI:0047F261 INT3_47F261     proc near                     ; CODE XREF: Disposal_IMP+A p

SDPI:0047F261                                 ; CreateThread2+3 p ...

SDPI:0047F261

SDPI:0047F261 var_4           = dword ptr -4

SDPI:0047F261 arg_0           = dword ptr  4

SDPI:0047F261

SDPI:0047F261             jz      short loc_47F26D

SDPI:0047F263             jnz     short loc_47F26D

SDPI:0047F263 ; ----------------------------------------------------------------------------

SDPI:0047F265             db 0

SDPI:0047F266             db  10h

SDPI:0047F267             db 40h

SDPI:0047F268             db 0

SDPI:0047F269             db  0Bh

SDPI:0047F26A             db 0ABh

SDPI:0047F26B             db 0F7h ; ?

SDPI:0047F26C             db    9

SDPI:0047F26D ; ----------------------------------------------------------------------------

SDPI:0047F26D

SDPI:0047F26D loc_47F26D:                              ; CODE XREF: INT3_47F261 j

SDPI:0047F26D                                 ; INT3_47F261+2 j

SDPI:0047F26D             call    loc_47F273

SDPI:0047F272             nop

SDPI:0047F273

SDPI:0047F273 loc_47F273:                              ; CODE XREF: INT3_47F261:loc_47F26D p

SDPI:0047F273             pop     eax

SDPI:0047F274             add     eax, 0D1h

SDPI:0047F279             push    eax                       ; EDX

SDPI:0047F27A             xor     eax, eax

SDPI:0047F27C             push    dword ptr fs:[eax]

SDPI:0047F27F             mov     fs:[eax], esp

SDPI:0047F282             mov     ebp, 300EF1D3h            ; 设置 Magic

SDPI:0047F287             add     ebp, 12345678h

SDPI:0047F28D             mov     ax, 17h

SDPI:0047F291             sub     ax, 13h                   ; 产生异常

SDPI:0047F295             int     3                         ; Trap to Debugger

SDPI:0047F296             nop

SDPI:0047F297             cmp     al, 4                     ; 如果用了 sice int3 之后返回的就不是 4 了

SDPI:0047F299             jz      short NOICE_47F30C

SDPI:0047F29B

SDPI:0047F29B OVER_47F29B:                               ; CODE XREF: INT3_47F261+C4 j

SDPI:0047F29B                                 ; INT3_47F261+CF j

SDPI:0047F29B             nop

SDPI:0047F29C             nop

SDPI:0047F29D             nop

SDPI:0047F29E             nop

SDPI:0047F29F             nop

SDPI:0047F2A0             call    loc_47F2A6
```

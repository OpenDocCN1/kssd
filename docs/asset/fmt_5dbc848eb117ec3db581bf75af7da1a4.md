
```
CODE:00556548                 mov     eax, [ebp-4]   

CODE:0055654B                 call    sub_468E08   

CODE:00556550                 mov     ds:byte_5A6960, 1   

CODE:00556557   

CODE:00556557 loc_556557:                           ; CODE XREF: CODE:00556402j   

CODE:00556557                                      ; CODE:005564ACj   

CODE:00556557                 xor     eax, eax   

CODE:00556559                 pop     edx   

CODE:0055655A                 pop     ecx   

CODE:0055655B                 pop     ecx   

CODE:0055655C                 mov     fs:[eax], edx   

CODE:0055655F                 push    offset loc_556591   

CODE:00556564   

CODE:00556564 loc_556564:                           ; CODE XREF: CODE:0055658Fj   

CODE:00556564                 lea     eax, [ebp-20h]   

CODE:00556567                 mov     edx, 2   

CODE:0055656C                 call    sub_404840   

CODE:00556571                 lea     eax, [ebp-18h]   

CODE:00556574                 call    sub_40481C   

CODE:00556579                 lea     eax, [ebp-14h]   

CODE:0055657C                 call    sub_40481C   

CODE:00556581                 lea     eax, [ebp-0Ch]   

CODE:00556584                 call    sub_40481C   

CODE:00556589                 retn   
```

分析此段程序,其判断流程在 `IDA` 中是一目了然的。由以下信息

```
CODE:00556467                 mov     ecx, (offset aUserid+4);注意我们记录的关键字"Userid"   

CODE:0055646C                 mov     edx, (offset aRegister+4);新出的关键字"Register"   
```

判断，程序是读取注册信息文件中的`USERID`和`REGISTER`信息进行校验的，而我们的注册信息文件中没有`REGISTER`域，故此出现“文件中没有可用的注册信息”的错误提示。现在我们可以在注册信息文件中添加`register`域如下：

```
[register]   

userid=25001255117182133267124000077036091107165139053071043144211054050264238181044294044   

0730970150860742690810590951550651481832671150661322630751350610851350   

register=250012551171821332671240000770360911071651390530710431442110540502642381810442940   

440730970150860742690810590951550651481832671150661322630751350610851350   
```

在一次运行程序则没有了“文件中没有可用的注册信息”，而是“注册失败，请与作者联系！”这就证明我们的判断是正确的。

进一步分析上面的程序，

```
CODE:0055650C                 mov     eax, ds:dword_5989DC   

CODE:00556511                 cmp     byte ptr [eax], 0 ; eax=59b504 关键的旗帜单元   

CODE:00556514                 jz      short bad_cracker   
```

发现`eax=59b504`是关键的旗帜单元,记录下来,并在`TRW2000`中下断点：`BPM 59b504`

然后重新启动`unpacked.exe`,遇到如下的断点序列：

```
51103A: MOV BYTE [0059B504],0   

...   

5111AC: SETZ [0059B504]   

...   

55611A: CMP BYTE [EAX],0   

...   

5564D5: MOV BYTE [EAX],1   

...   

511471: MOV BYTE [0059B504],0  <<<---在注册的流程当中,此处是最后的出点,我们可由此切入.   
```

在`IDA`中分析`511471`附近的程序:

```
CODE:00511471 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   

CODE:00511471   

CODE:00511471 GHOST_GONIG:                        ; CODE XREF: sub_510FF4+32Dj   

CODE:00511471                 mov     ds:byte_59B504, 0 ; ++置 0++   

CODE:00511478                 xor     eax, eax   

CODE:0051147A                 push    ebp   

CODE:0051147B                 push    offset unk_5114A0   

CODE:00511480                 push    dword ptr fs:[eax]   
```

由交叉引用信息再到:

```
CODE:00511276 loc_511276:                           ; CODE XREF: sub_510FF4+F4j   

CODE:00511276                 call    sub_513534   

CODE:0051127B                 lea     eax, [ebp+var_64]   

CODE:0051127E                 mov     ecx, offset unk_51168C   

CODE:00511283                 mov     edx, ds:dword_59B518   

CODE:00511289                 call    sub_404B20   

CODE:0051128E                 mov     eax, [ebp+var_64]   

CODE:00511291                 lea     ecx, [ebp+var_60]   

CODE:00511294                 mov     dx, 0E214h   

CODE:00511298                 call    sub_50D550   

CODE:0051129D                 mov     eax, [ebp+var_60]   

CODE:005112A0                 lea     ecx, [ebp+var_5C]   

CODE:005112A3                 mov     dx, 0FF97h   

CODE:005112A7                 call    sub_50D550   

CODE:005112AC                 mov     ecx, [ebp+var_5C]   

CODE:005112AF                 mov     edx, (offset aProductdremark+4)   

CODE:005112B4                 mov     eax, [ebp+var_18]   

CODE:005112B7                 call    sub_473C88   

CODE:005112BC   

CODE:005112BC loc_5112BC:                           ; CODE XREF: sub_510FF4+DFj   

CODE:005112BC                                      ; sub_510FF4+27Aj ...   

CODE:005112BC                 mov     eax, [ebp+var_18]   

CODE:005112BF                 call    sub_473928   

CODE:005112C4                 mov     eax, [ebp+var_18]   

CODE:005112C7                 call    sub_40394C   

CODE:005112CC   

CODE:005112CC loc_5112CC:                           ; CODE XREF: sub_510FF4+80j   

CODE:005112CC                 cmp     byte ptr [ebp+var_4+3], 0   

CODE:005112D0                 jz      loc_5114B6   

CODE:005112D6                 mov     [ebp+var_D], 0   

CODE:005112DA                 mov     edx, 80000002h   

CODE:005112DF                 mov     eax, [ebp+var_18]   

CODE:005112E2                 call    sub_473958   

CODE:005112E7                 mov     cl, 1   

CODE:005112E9                 mov     edx, (offset aSoftwareMicrosof+4)   

CODE:005112EE                 mov     eax, [ebp+var_18]   

CODE:005112F1                 call    sub_4739BC   

CODE:005112F6                 test    al, al   

CODE:005112F8                 jz      loc_5114B6   

CODE:005112FE                 lea     eax, [ebp+var_1C]   

CODE:00511301                 mov     edx, [ebp+var_C]   

CODE:00511304                 call    sub_4048B4   

CODE:00511309                 lea     edx, [ebp+var_68]   

CODE:0051130C                 mov     eax, [ebp+var_1C] ; 注册信息文件中的数据   

CODE:0051130F                 call    sub_4094C8   

CODE:00511314                 mov     edx, [ebp+var_68]   

CODE:00511317                 mov     eax, [ebp+var_8] ; 注册信息文件中的数据   

CODE:0051131A                 call    calculate_register ; +++this call+++   

CODE:0051131F                 test    al, al   

CODE:00511321                 jz      GHOST_GONIG     ; ++置 0++   

CODE:00511327                 xor     eax, eax   

CODE:00511329                 push    ebp   

CODE:0051132A                 push    offset loc_51139C   

CODE:0051132F                 push    dword ptr fs:[eax]   

CODE:00511332                 mov     fs:[eax], esp   

CODE:00511335                 call    sub_513534   

CODE:0051133A                 lea     eax, [ebp+var_74]   

CODE:0051133D                 mov     ecx, offset unk_511698   

CODE:00511342                 mov     edx, ds:dword_59B518   

CODE:00511348                 call    sub_404B20   

CODE:0051134D                 mov     eax, [ebp+var_74]   

CODE:00511350                 lea     ecx, [ebp+var_70]   

CODE:00511353                 mov     dx, 0E214h   

CODE:00511357                 call    sub_50D550   

CODE:0051135C                 mov     eax, [ebp+var_70]   

CODE:0051135F                 lea     ecx, [ebp+var_6C]   

CODE:00511362                 mov     dx, 0FF97h   

CODE:00511366                 call    sub_50D550   

CODE:0051136B                 mov     ecx, [ebp+var_6C]   

CODE:0051136E                 mov     edx, (offset aProductdremark+4)   

CODE:00511373                 mov     eax, [ebp+var_18]   

CODE:00511376                 call    sub_473C88   

CODE:0051137B                 mov     ds:byte_59B504, 1 ; ++置"1"++   

CODE:00511382                 mov     eax, [ebp+var_18]   

CODE:00511385                 call    sub_473928   

CODE:0051138A                 mov     eax, [ebp+var_18]   

CODE:0051138D                 call    sub_40394C   

CODE:00511392                 xor     eax, eax   
```

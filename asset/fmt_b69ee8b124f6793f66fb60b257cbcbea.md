
# Update NOW 2000

> 原文：[`www.pediy.com/kssd/pediy03/forum613.htm`](https://www.pediy.com/kssd/pediy03/forum613.htm)

## Update NOW 2000 暴力破解方法！

### 工具

- `trw2000` (搞破解人士必备物品，可以消痛止血、健脾、化痰.....)
- `IDA` (巨酷的软件，高手必备)
- `HIEW` (不用他改字节用什么？)

### 步骤

1. 先下载 `Update NOW 2000` （@#$@%$&^%）
2. 先用 `W32dsm` 反编译，结果出错！（@#$@%$&^%）
3. 知道他防 `W32dsm`，那么我们的 `IDA` 就要上场了！！用 `IDA` 反编译！（使用方法到看雪那里找）
4. 在注册处随便输入几个字符提示谢谢（这也要谢？）发现他写到 `windows/update.ini` 里面：

   ```
   NAME=xxxxx
   SERIAL=xxxxx
   ```

   那么根据福尔莫斯理论，我们已经找到关键的线索的在 `IDA` 下找着 `SERIAL` 字符串（最好加上那个找字符串的插件），会找到好几个，但是关键在这里：

   ```
   004356FD           push    offset aName    ; "NAME"
   00435702           lea     edx, [esp+278h+var_258]
   00435706           push    offset aOption  ; "OPTION"
   0043570B           push    edx
   0043570C           mov     ecx, eax
   0043570E           call    sub_48716A
   00435713           push    eax
   00435714           lea     ecx, [esp+274h+var_250]
   00435718           mov     byte ptr [esp+274h+var_4], 5
   00435720           call    sub_470D4D
   00435725           lea     ecx, [esp+270h+var_258]
   00435729           mov     byte ptr [esp+270h+var_4], 3
   00435731           call    sub_470C14
   00435736           mov     eax, [esp+270h+var_250]
   0043573A           cmp     [eax-8], esi
   0043573D           jz      loc_4358D5
   00435743           push    offset aRegister ; "register"     --------黑名单！
   00435748           push    eax
   00435749           call    __strcmpi
   0043574E           add     esp, 8
   00435751           cmp     eax, esi
   00435753           jz      loc_4358D5
   00435759           mov     eax, [esp+270h+var_250]
   0043575D           push    offset aHanliner ; "hanliner"     --------黑名单！
   00435762           push    eax
   00435763           call    __strcmpi
   00435768           add     esp, 8
   0043576B           cmp     eax, esi
   0043576D           jz      loc_4358D5
   00435773           mov     ecx, [esp+270h+var_250]
   00435777           push    offset aOpq98  ; "opq98"      --------黑名单！（这个知道是谁吧，哈哈！）
   0043577C           push    ecx
   0043577D           call    __strcmpi
   00435782           add     esp, 8
   00435785           cmp     eax, esi
   00435787           jz      loc_4358D5
   0043578D           mov     edx, [esp+270h+var_250]
   00435791           push    offset aWenling ; "wenling"
   00435796           push    edx
   00435797           call    __strcmpi
   0043579C           add     esp, 8
   0043579F           cmp     eax, esi
   004357A1           jz      loc_4358D5
   004357A7           push    6
   004357A9           push    offset aWww_pcchina_ne ; "www.pcchina.net"
   004357AE           lea     eax, [esp+278h+var_18C]
   004357B5           push    0FFFFFFC7h
   004357B7           push    eax
   004357B8           lea     ecx, [esp+280h+var_250]
   004357BC           call    sub_401800
   004357C1           push    ecx
   004357C2           mov     ecx, esp
   004357C4           mov     [esp+284h+var_260], esp
   004357C8           push    eax
   004357C9           call    sub_470C82
   004357CE           mov     ecx, ebp
   004357D0           call    sub_43B100
   004357D5           call    ?AfxGetModuleState@@YGPAVAFX_MODULE_STATE@@XZ ; AfxGetModuleState(void)
   004357DA           mov     eax, [eax+4]
   004357DD           push    esi
   004357DE           push    offset aSerial  ; "SERIAL"              -------开始启动的时候检验这里！
   004357E3           lea     ecx, [esp+278h+var_258]
   004357E7           push    offset aOption  ; "OPTION"
   004357EC           push    ecx
   004357ED           mov     ecx, eax
   004357EF           call    sub_48716A
   004357F4           push    eax
   004357F5           lea     ecx, [esp+274h+var_254]
   004357F9           mov     byte ptr [esp+274h+var_4], 6
   00435801           call    sub_470D4D
   00435806           lea     ecx, [esp+270h+var_258]
   0043580A           mov     byte ptr [esp+270h+var_4], 3
   00435812           call    sub_470C14
   00435817           lea     edi, [esp+270h+var_18C]
   0043581E           or      ecx, 0FFFFFFFFh
   00435821           xor     eax, eax
   00435823           repne scasb
   00435825           not     ecx
   00435827           dec     ecx
   00435828           lea     edi, [esp+270h+var_10C]
   0043582F           mov     edx, ecx
   00435831           or      ecx, 0FFFFFFFFh
   00435834           repne scasb
   00435836           not     ecx
   00435838           dec     ecx
   00435839           cmp     ecx, edx
   0043583B           jz      short loc_43586D      --------程序启动的时候这里条转往下可是一
   0043583D                                               会儿程序又会条到会到下面！
   0043583D loc_43583D:            ; CODE XREF: sub_435550+36Bj  -----又跳到这里！通过 trw 跟踪发现问题！
   0043583D           lea     edi, [ebp+2D50h]
   00435843           push    esi
   00435844           push    0B1h
   00435849           mov     ecx, edi
   0043584B           call    sub_46BE10
   00435850           test    eax, eax
   00435852           jz      loc_4367F1
   00435858           push    5
   0043585A           mov     ecx, edi
   0043585C           call    sub_4700EC          --------这里出现那个延时干扰框！怎么办不不用说了吧！
   00435861           mov     edx, [ebp+2D6Ch]
   00435867           push    edx
   00435868           jmp     loc_435900
   ```

   程序已经没有干扰了，但是是，关于里面还是未注册不爽，继续改！

   ============================================================

   找到字符串 `UNREGISTERED`

   ```
   00401322           lea     edi, [esp+138h+var_10C]
   00401326
   00401326 loc_401326:            ; CODE XREF: sub_4011C0+184j
   00401326           mov     dl, [eax]
   00401328           mov     cl, dl
   0040132A           cmp     dl, [edi]
   0040132C           jnz     short loc_40134A
   0040132E           test    cl, cl
   00401330           jz      short loc_401346
   00401332           mov     dl, [eax+1]
   00401335           mov     cl, dl
   00401337           cmp     dl, [edi+1]
   0040133A           jnz     short loc_40134A
   0040133C           add     eax, 2
   0040133F           add     edi, 2
   00401342           test    cl, cl
   00401344           jnz     short loc_401326
   00401346
   00401346 loc_401346:            ; CODE XREF: sub_4011C0+170j
   00401346           xor     eax, eax
   00401348           jmp     short loc_40134F
   0040134A ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
   0040134A
   0040134A loc_40134A:            ; CODE XREF: sub_4011C0+16Cj
   0040134A                        ; sub_4011C0+17Aj
   0040134A           sbb     eax, eax
   0040134C           sbb     eax, 0FFFFFFFFh
   0040134F
   0040134F loc_40134F:            ; CODE XREF: sub_4011C0+188j
   0040134F           test    eax, eax
   00401351           jnz     short loc_401372                  -------这里枪毙！
   00401353           mov     eax, [esp+138h+var_128]
   00401357           lea     ecx, [esp+138h+var_10C]
   0040135B           push    eax
   0040135C           push    offset aRegisteredToS_ ; "Registered to %s."
   00401361           push    ecx
   ```

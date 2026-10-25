
# 简单算法――Windows设置大师 2003 V2.0 Build 0420

- 下载页面：`http://www.skycn.net/soft/11596.html`
- 软件大小：850 KB
- 软件语言：简体中文
- 软件类别：国产软件 / 共享版 / 系统设置
- 应用平台：Win9x/NT/2000/XP
- 加入时间：2003-04-06 15:23:02
- 下载次数：1810
- 推荐等级：\*\*\*

## 软件简介

1、Windows高级设置:在这个设置里，您可以系统进行一系列设置，包括禁止注册表运行、禁止按取消键登录系统等等……2、开始菜单和控制面板设置:在这个设置里，您可以对开始菜单和控制面板进行一系列设置……3、IE高级设置:在这个设置里，您可以IE进行一系列设置，包括隐藏部分选项、修改IE标题等等……4、其它高级设置:在这个设置里，您可以清除系统垃圾、隐藏驱动器等等……5、其它功能正在扩充中……

## 软件限制

20次试用。

## 作者声明

初学Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

## 破解工具

TRW2000娃娃修改版、PEiD、W32Dasm 10修改版

---

## 过程

升级很快呀。新版的主文件加的壳挺怪，几个侦测工具均没能认出来，还好，没有反调试代码，方便点。

Ollydbg却不好用了。冲击波也找不到入口，冲击波老了？呵呵，索性用TRW直接调试了。

申请号：14215752  
姓名：fly  
试炼码：13572468

---

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AC7F1(C)

:004AC858 53                      push ebx
:004AC859 56                      push esi
:004AC85A 57                      push edi
:004AC85B 8945FC                  mov dword ptr [ebp-04], eax
:004AC85E 33C0                    xor eax, eax
:004AC860 55                      push ebp
:004AC861 68A3C94A00              push 004AC9A3
:004AC866 64FF30                  push dword ptr fs:[eax]
:004AC869 648920                  mov dword ptr fs:[eax], esp
:004AC86C 33DB                    xor ebx, ebx
:004AC86E 8D55F4                  lea edx, dword ptr [ebp-0C]
:004AC871 8B45FC                  mov eax, dword ptr [ebp-04]
:004AC874 8B80A4040000            mov eax, dword ptr [eax+000004A4]
:004AC87A E8D508FBFF              call 0045D154
:004AC87F 8B45F4                  mov eax, dword ptr [ebp-0C]
                 ====>EAX=[ebp-0C]=fly
:004AC882 E8D17BF5FF              call 00404458
       ====>取位数   EAX=3
:004AC887 8BF8                    mov edi, eax
:004AC889 6A00                    push 00000000
:004AC88B 6800040000              push 00000400
:004AC890 B003                    mov al, 03
:004AC892 E891C4F5FF              call 00408D28
:004AC897 E82087F5FF              call 00404FBC
:004AC89C 8BC8                    mov ecx, eax
:004AC89E BE01000000              mov esi, 00000001
:004AC8A3 8D0431                  lea eax, dword ptr [ecx+esi]
:004AC8A6 99                      cdq
:004AC8A7 33C2                    xor eax, edx
:004AC8A9 2BC2                    sub eax, edx
:004AC8AB 054747A000              add eax, 00A04747
:004AC8B0 8BF0                    mov esi, eax
:004AC8B2 8D45E0                  lea eax, dword ptr [ebp-20]
:004AC8B5 50                      push eax
:004AC8B6 8D55DC                  lea edx, dword ptr [ebp-24]
:004AC8B9 8BC6                    mov eax, esi
:004AC8BB E854BDF5FF              call 00408614
:004AC8C0 8B45DC                  mov eax, dword ptr [ebp-24]
     ====>EAX=14215752            申请号
:004AC8C3 B908000000              mov ecx, 00000008
:004AC8C8 BA01000000              mov edx, 00000001
:004AC8CD E8E67DF5FF              call 004046B8
:004AC8D2 8B45E0                  mov eax, dword ptr [ebp-20]
                 ====>EAX=00D8EA48（H）=14215752（D）
:004AC8D5 E876BEF5FF              call 00408750
:004AC8DA 8945E8                  mov dword ptr [ebp-18], eax
:004AC8DD 8D45F0                  lea eax, dword ptr [ebp-10]
*
Possible StringData Ref from Data Obj ->"-MK5609ZW"
     |
:004AC8E0 BABCC94A00              mov edx, 004AC9BC
                   ====>EDX=-MK5609ZW
:004AC8E5 E84679F5FF              call 00404230
:004AC8EA 8B45F0                  mov eax, dword ptr [ebp-10]
:004AC8ED E8667BF5FF              call 00404458
                   ====>取-MK5609ZW的位数
:004AC8F2 8945E4                  mov dword ptr [ebp-1C], eax
                 ====>EAX=9
:004AC8F5 8BF7                    mov esi, edi
             ====>ESI=EDI=3
:004AC8F7 85F6                    test esi, esi
:004AC8F9 7E3E                    jle 004AC939
:004AC8FB C745EC01000000          mov [ebp-14], 00000001
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AC937(C)

:004AC902 8D45D8                  lea eax, dword ptr [ebp-28]
:004AC905 50                      push eax
:004AC906 B901000000              mov ecx, 00000001
:004AC90B 8B55EC                  mov edx, dword ptr [ebp-14]
:004AC90E 8B45F4                  mov eax, dword ptr [ebp-0C]
                 ====>EAX=[ebp-0C]=fly
:004AC911 E8A27DF5FF              call 004046B8
:004AC916 8B45D8                  mov eax, dword ptr [ebp-28]
:004AC919 E83A7DF5FF              call 00404658
:004AC91E 8A00                    mov al, byte ptr [eax]
                    1、====>AL=66
          2、====>AL=6C
                      3、====>AL=79
:004AC920 25FF000000              and eax, 000000FF
:004AC925 03D8                    add ebx, eax
      1、====>EBX=00000000 + 66=00000066
2、====>EBX=01C43869 + 6C=01C438D5
3、====>EBX=038870D8 + 79=03887151
:004AC927 81C3B24DEB00            add ebx, 00EB4DB2
1、====>EBX=00000066 + 00EB4DB2=00EB4E18
2、====>EBX=01C438D5 + 00EB4DB2=02AF8687
3、====>EBX=03887151 + 00EB4DB2=0473BF03
:004AC92D 035DE8                  add ebx, dword ptr [ebp-18]
          1、====>EBX=00EB4E18 + 00D8EA48=01C43860
2、====>EBX=02AF8687 + 00D8EA48=038870CF
3、====>EBX=0473BF03 + 00D8EA48=054CA94B
:004AC930 035DE4                  add ebx, dword ptr [ebp-1C]
          1、====>EBX=01C43860 + 9=01C43869
2、====>EBX=038870CF + 9=038870D8
3、====>EBX=054CA94B + 9=054CA954
               20:14 03-4-12 呵呵，054CA954的10进制值就是我的注册码了！
:004AC933 FF45EC                  inc [ebp-14]
:004AC936 4E                      dec esi
             ====>ESI 依次减1
:004AC937 75C9                    jne 004AC902
             ====>循环用户名位数次
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AC8F9(C)

:004AC939 8D55D4                  lea edx, dword ptr [ebp-2C]
:004AC93C 8B45FC                  mov eax, dword ptr [ebp-04]
:004AC93F 8B80A8040000            mov eax, dword ptr [eax+000004A8]
       ====>EAX=00CD45A0
:004AC945 E80A08FBFF              call 0045D154
:004AC94A 8B45D4                  mov eax, dword ptr [ebp-2C]
                 ====>EAX=13572468
:004AC94D E8FEBDF5FF              call 00408750
       ====>把13572468（D）转换成16进制值EAX=00CF1974（H）
:004AC952 3BD8                    cmp ebx, eax
             ====>EBX=054CA954（H）=88910164（D） 注册码！
               ====>EAX=00CF1974（H）=13572468（D） 试炼码
                  呵呵，比较注册码了。如果相等就OK了！所以我的注册码就是EBX里的10进制值！
:004AC954 7519                    jne 004AC96F
             ====>跳则OVER！
:004AC956 C645FB01                mov [ebp-05], 01
           ====>置1则OK！
:004AC95A B880714B00              mov eax, 004B7180
:004AC95F 8B55F4                  mov edx, dword ptr [ebp-0C]
:004AC962 E88578F5FF              call 004041EC
:004AC967 891D84714B00            mov dword ptr [004B7184], ebx
:004AC96D EB04                    jmp 004AC973
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AC954(C)

:004AC96F C645FB00                mov [ebp-05], 00
           ====>清0则OVER！
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AC96D(U)

:004AC973 33C0                    xor eax, eax
:004AC975 5A                      pop edx
:004AC976 59                      pop ecx
:004AC977 59                      pop ecx
:004AC978 648910                  mov dword ptr fs:[eax], edx
:004AC97B 68AAC94A00              push 004AC9AA
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AC9A8(U)

:004AC980 8D45D4                  lea eax, dword ptr [ebp-2C]
:004AC983 E81078F5FF              call 00404198
:004AC988 8D45D8                  lea eax, dword ptr [ebp-28]
:004AC98B BA03000000              mov edx, 00000003
:004AC990 E82778F5FF              call 004041BC
:004AC995 8D45F0                  lea eax, dword ptr [ebp-10]
:004AC998 BA02000000              mov edx, 00000002
:004AC99D E81A78F5FF              call 004041BC
:004AC9A2 C3                      ret
:004AC9A3 E95071F5FF              jmp 00403AF8
:004AC9A8 EBD6                    jmp
```

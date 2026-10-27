
# 软件试用次数验证流程分析

``` 
  00489254    |.  8BFA     mov     edi, edx
  00489256    |.  8BF0     mov     esi, eax
  00489258    |.  33C0     xor     eax, eax
  0048925A    |.  55       push    ebp
  0048925B    |.  68 FA934>push    <HideHelp.loc_4893FA>
  00489260    |.  64:FF30  push    dword ptr fs:[eax]
  00489263    |.  64:8920  mov     dword ptr fs:[eax], esp
  00489266    |.  8D45 FC  lea     eax, [local.1]
  00489269    |.  BA 10944>mov     edx, <HideHelp.aXyxue4kfs0g4sj>                                    ;  xyxue+4kfs0g4sjnsewvyx1hkk5folj8mgnghlgawqnwolumwkrpvy
  0048926E    |.  E8 FDB4F>call    <HideHelp.System::__linkproc__ LStrLAsg(void *,void *)>            ;  键路径密文
  00489273    |.  8D45 F8  lea     eax, [local.2]
  00489276    |.  BA 50944>mov     edx, <HideHelp.aS20eelxpba>                                        ;  s20eelxpba
  0048927B    |.  E8 F0B4F>call    <HideHelp.System::__linkproc__ LStrLAsg(void *,void *)>            ;  键名密文
  00489280    |.  8D55 FC  lea     edx, [local.1]
  00489283    |.  8B45 FC  mov     eax, [local.1]
  00489286    |.  E8 C12AF>call    <HideHelp.ConvertBase64to256(AnsiString,AnsiString &)>             ;  Base64解码
  0048928B    |.  8D4D F0  lea     ecx, [local.4]
  0048928E    |.  BA F3F05>mov     edx, 59F0F3
  00489293    |.  8B45 FC  mov     eax, [local.1]
  00489296    |.  E8 5DA9F>call    <HideHelp.subN_111111_Decrypt>                                     ;  解密键路径
  0048929B    |.  8B55 F0  mov     edx, [local.4]
  0048929E    |.  8D45 FC  lea     eax, [local.1]
  004892A1    |.  E8 CAB4F>call    <HideHelp.System::__linkproc__ LStrLAsg(void *,void *)>
  004892A6    |.  8D55 F8  lea     edx, [local.2]
  004892A9    |.  8B45 F8  mov     eax, [local.2]
  004892AC    |.  E8 9B2AF>call    <HideHelp.ConvertBase64to256(AnsiString,AnsiString &)>
  004892B1    |.  8D4D EC  lea     ecx, [local.5]
  004892B4    |.  BA F3F05>mov     edx, 59F0F3
  004892B9    |.  8B45 F8  mov     eax, [local.2]
  004892BC    |.  E8 37A9F>call    <HideHelp.subN_111111_Decrypt>                                     ;  解密键名
  004892C1    |.  8B55 EC  mov     edx, [local.5]
  004892C4    |.  8D45 F8  lea     eax, [local.2]
  004892C7    |.  E8 A4B4F>call    <HideHelp.System::__linkproc__ LStrLAsg(void *,void *)>
  004892CC    |.  B2 01    mov     dl, 1
  004892CE    |.  A1 EC844>mov     eax, dword ptr ds:[<off_4384EC>]
  004892D3    |.  E8 14F3F>call    <HideHelp.Registry::TRegistry::TRegistry(void)>
  004892D8    |.  8BD8     mov     ebx, eax
  004892DA    |.  BA 00000>mov     edx, 80000000
  004892DF    |.  8BC3     mov     eax, ebx
  004892E1    |.  E8 A6F3F>call    <HideHelp.Registry::TRegistry::SetRootKey(uint)>
  004892E6    |.  8B55 FC  mov     edx, [local.1]
  004892E9    |.  8BC3     mov     eax, ebx
  004892EB    |.  E8 B4F7F>call    <HideHelp.Registry::TRegistry::KeyExists(System::AnsiString)>      ;  判断键是否存在
  004892F0    |.  84C0     test    al, al
  004892F2    |.  74 1B    je      short <HideHelp.loc_48930F>
  004892F4    |.  33C9     xor     ecx, ecx
  004892F6    |.  8B55 FC  mov     edx, [local.1]
  004892F9    |.  8BC3     mov     eax, ebx
  004892FB    |.  E8 F0F3F>call    <HideHelp.Registry::TRegistry::OpenKey(System::AnsiString,bool)>
  00489300    |.  8D4D F4  lea     ecx, [local.3]
  00489303    |.  8B55 F8  mov     edx, [local.2]
  00489306    |.  8BC3     mov     eax, ebx
  00489308    |.  E8 ABF5F>call    <HideHelp.Registry::TRegistry::ReadString(System::AnsiString)>     ;  如果存在，则读出键值
  0048930D    |.  EB 19    jmp     short <HideHelp.loc_489328>
  0048930F <H>|>  B1 01    mov     cl, 1                                                              ;  如果不存在，则创建并设置值为"y4o"
  00489311    |.  8B55 FC  mov     edx, [local.1]
  00489314    |.  8BC3     mov     eax, ebx
  00489316    |.  E8 D5F3F>call    <HideHelp.Registry::TRegistry::OpenKey(System::AnsiString,bool)>
  0048931B    |.  8D45 F4  lea     eax, [local.3]
  0048931E    |.  BA 64944>mov     edx, <HideHelp.dword_489464>                                       ;  y4o
  00489323    |.  E8 48B4F>call    <HideHelp.System::__linkproc__ LStrLAsg(void *,void *)>
  00489328 <H>|>  8D55 F4  lea     edx, [local.3]                                                     ;  loc_489328
  0048932B    |.  8B45 F4  mov     eax, [local.3]
  0048932E    |.  E8 192AF>call    <HideHelp.ConvertBase64to256(AnsiString,AnsiString &)>             ;  读出的值解码
  00489333    |.  8D4D E8  lea     ecx, [local.6]
  00489336    |.  BA F3F05>mov     edx, 59F0F3
  0048933B    |.  8B45 F4  mov     eax, [local.3]
  0048933E    |.  E8 B5A8F>call    <HideHelp.subN_111111_Decrypt>                                     ;  解密读出的值
  00489343    |.  8B55 E8  mov     edx, [local.6]
  00489346    |.  8D45 F4  lea     eax, [local.3]
  00489349    |.  E8 22B4F>call    <HideHelp.System::__linkproc__ LStrLAsg(void *,void *)>
  0048934E    |.  8B45 F4  mov     eax, [local.3]
  00489351    |.  E8 C2FAF>call    <HideHelp.Sysutils::StrToInt(System::AnsiString)>
  00489356    |.  48       dec     eax                                                                ;  值转换为int后减1，如果结果为0则 退出程序
  00489357    |.  79 24    jns     short <HideHelp.loc_48937D>
  00489359    |.  E8 DEFCF>call    <HideHelp.DisableHook>                                             ;  jmp 到 form.DisableHook
  0048935E    |.  68 C4AE4>push    offset <HideHelp.Data>                                             ;  x
  00489363    |.  6A 02    push    2
  00489365    |.  E8 E65BF>call    <HideHelp.Shell_NotifyIconA>                                       ;  jmp 到 shell32.Shell_NotifyIconA
  0048936A    |.  8BC6     mov     eax, esi
  0048936C    |.  E8 AFFDF>call    <HideHelp.sub_489120>
  00489371    |.  A1 B89B4>mov     eax, dword ptr ds:[<off_499BB8>]
  00489376    |.  8B00     mov     eax, dword ptr ds:[eax]
  00489378    |.  E8 4F0AF>call    <HideHelp.Forms::TApplication::Terminate(void)>
  0048937D <H>|>  8B45 F4  mov     eax, [local.3]                                                     ;  loc_48937D
  00489380    |.  E8 93FAF>call    <HideHelp.Sysutils::StrToInt(System::AnsiString)>
  00489385    |.  48       dec     eax
  00489386    |.  8BD7     mov     edx, edi
  00489388    |.  E8 4FF9F>call    <HideHelp.Sysutils::IntToStr(int)>
  0048938D    |.  8B45 F4  mov     eax, [local.3]
  00489390    |.  E8 83FAF>call    <HideHelp.Sysutils::StrToInt(System::AnsiString)>
  00489395    |.  48       dec     eax
  00489396    |.  8D55 E0  lea     edx, [local.8]
  00489399    |.  E8 3EF9F>call    <HideHelp.Sysutils::IntToStr(int)>
  0048939E    |.  8B45 E0  mov     eax, [local.8]
  004893A1    |.  8D4D E4  lea     ecx, [local.7]
  004893A4    |.  BA F3F05>mov     edx, 59F0F3
  004893A9    |.  E8 AAA7F>call    <HideHelp.subN_222222_Encrypt>                                     ;  结果减1后，转为string，加密
  004893AE    |.  8B55 E4  mov     edx, [local.7]
  004893B1    |.  8D45 F4  lea     eax, [local.3]
  004893B4    |.  E8 B7B3F>call    <HideHelp.System::__linkproc__ LStrLAsg(void *,void *)>
  004893B9    |.  8D55 F4  lea     edx, [local.3]
  004893BC    |.  8B45 F4  mov     eax, [local.3]
  004893BF    |.  E8 0828F>call    <HideHelp.ConvertBase256to64(AnsiString,AnsiString &)>             ;  编码，保存到注册表
  004893C4    |.  8B4D F4  mov     ecx, [local.3]
  004893C7    |.  8B55 F8  mov     edx, [local.2]
  004893CA    |.  8BC3     mov     eax, ebx
  004893CC    |.  E8 BBF4F>call    <HideHelp.Registry::TRegistry::WriteString(System::AnsiString,Syst>
  004893D1    |.  8BC3     mov     eax, ebx
  004893D3    |.  E8 84F2F>call    <HideHelp.subN_RegClose_Flush>
  004893D8    |.  8BC3     mov     eax, ebx
  004893DA    |.  E8 69A5F>call    <HideHelp.System::TObject::Free(void)>
  004893DF    |.  33C0     xor     eax, eax
  004893E1    |.  5A       pop     edx
```

## 整理说明

软件作者保密意识很好，在软件中没有出现任何明码信息。软件的试用次数是保存在注册表中的，然后注册表路径，作者对其进行了加密存放，这样分析者在静态分析软件时，看到的只是一串串乱七八糟的密文。

### 试用次数验证的关键

试用次数验证的核心流程为解密注册表路径与键名、Base64 解码、解密后读取并判断次数、次数减一后重新加密编码写回注册表。

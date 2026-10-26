
# [原创]power video converter/v1.66破解文档

个人学习备用档案，高手勿看。

这是一款国外视频转换软件，可以在AVI/MPEG/VCD/DVD/DAT/VOB之间转换视频文件、分割文件、提取音频或图像。

如果未注册只能转换文件的一半，开始我被那个大大的注册码框吓倒了，以为注册码很长算法很复杂，就去网上找码子，找了半天也没找着，只好自己干，谁知算法十分的简单，注册机让我轻易地写出来了，这是不是老外和我们开的一个玩笑。

用字符串参考很快就来到关键点：

```
00421810  />push    -1
00421812  |>push    0042C5B0                                  ;  SE 处理程序安装
00421817  |>mov     eax, dword ptr fs:[0]
0042181D  |>push    eax
0042181E  |>mov     dword ptr fs:[0], esp
00421825  |>sub     esp, 14
00421828  |>mov     eax, dword ptr [esp+24]                   ;  eax=注册名
0042182C  |>push    ebx
0042182D  |>push    ebp                                       ;  kernel32.GetPrivateProfileStringA
0042182E  |>push    esi
0042182F  |>push    edi
00421830  |>push    eax
00421831  |>lea     ecx, dword ptr [esp+18]
00421835  |>call    <jmp.&MFC42.#537\_CString::CString>        ;  CString strName=CString(注册名);
0042183A  |>lea     ecx, dword ptr [esp+14]
0042183E  |>mov     dword ptr [esp+2C], 0
00421846  |>call    <jmp.&MFC42.#6282\_CString::TrimLeft>      ;  去左空格
0042184B  |>lea     ecx, dword ptr [esp+14]
0042184F  |>call    <jmp.&MFC42.#6283\_CString::TrimRight>     ;  去右空格
00421854  |>push    20
00421856  |>lea     ecx, dword ptr [esp+18]
0042185A  |>call    <jmp.&MFC42.#2915\_CString::GetBuffer>
0042185F  |>mov     ecx, dword ptr [esp+38]                   ;  ecx=注册码
00421863  |>mov     ebx, eax                                  ;  ebx=strName.GetBuffer(0x20);
00421865  |>push    ecx
00421866  |>lea     ecx, dword ptr [esp+14]
0042186A  |>call    <jmp.&MFC42.#537\_CString::CString>        ;  CString strCode=CString(注册码);
0042186F  |>lea     ecx, dword ptr [esp+10]
00421873  |>mov     byte ptr [esp+2C], 1
00421878  |>call    <jmp.&MFC42.#6282\_CString::TrimLeft>      ;  去左空格
0042187D  |>lea     ecx, dword ptr [esp+10]
00421881  |>call    <jmp.&MFC42.#6283\_CString::TrimRight>     ;  去右空格
00421886  |>push    20
00421888  |>lea     ecx, dword ptr [esp+14]
0042188C  |>call    <jmp.&MFC42.#2915\_CString::GetBuffer>
00421891  |>mov     edx, eax                                  ;  edx=strCode.GetBuffer(0x20);
00421893  |>or      esi, FFFFFFFF
00421896  |>mov     edi, edx
00421898  |>mov     ecx, esi
0042189A  |>xor     eax, eax
0042189C  |>mov     dword ptr [esp+20], edx
004218A0  |>repne   scas byte ptr es:[edi]
004218A2  |>not     ecx
004218A4  |>dec     ecx                                       ;  串长
004218A5  |>mov     edi, ebx
004218A7  |>mov     ebp, ecx                                  ;  ebp=注册码长度
004218A9  |>mov     ecx, esi
004218AB  |>repne   scas byte ptr es:[edi]
004218AD  |>not     ecx                                       ;  串长
004218AF  |>dec     ecx                                       ;  ecx为注册名长度
004218B0  |>cmp     ecx, ebp                                  ;  比较两串长度
004218B2  |>ja      00421A0C                                  ;  名度不能超过码长
004218B8  |>mov     edi, ebx
004218BA  |>mov     ecx, esi
004218BC  |>repne   scas byte ptr es:[edi]
004218BE  |>not     ecx
004218C0  |>dec     ecx                                       ;  串长
004218C1  |>je      00421A0C                                  ;  名长不能为０
004218C7  |>mov     edi, edx
004218C9  |>mov     ecx, esi
004218CB  |>repne   scas byte ptr es:[edi]
004218CD  |>not     ecx
004218CF  |>dec     ecx                                       ;  串长
004218D0  |>je      00421A0C                                  ;  码长不能为０
004218D6  |>mov     dword ptr [esp+38], eax                   ;  初始化为NULL
004218DA  |>/mov     edx, dword ptr [esp+38]                  ;  计算次数
004218DE  |>|lea     ecx, dword ptr [esp+34]
004218E2  |>|mov     al, byte ptr [edx+4388A8]                ;  未查到字符时用它
004218E8  |>|mov     byte ptr [esp+18], al
004218EC  |>|call    <jmp.&MFC42.#540\_CString::CString>
004218F1  |>|mov     edi, ebx
004218F3  |>|or      ecx, FFFFFFFF
004218F6  |>|xor     eax, eax
004218F8  |>|xor     ebp, ebp                                 ;  ebp=0
004218FA  |>|repne   scas byte ptr es:[edi]
004218FC  |>|not     ecx
004218FE  |>|dec     ecx                                      ;  串长
004218FF  |>|mov     byte ptr [esp+2C], 2
00421904  |>|je      short 00421951                           ;  名长是否为0?
00421906  |>|/mov     al, byte ptr [ebx+ebp]                  ;  取名的第 EBP 个字符
00421909  |>||xor     esi, esi                                ;  esi=0
0042190B  |>||/cmp     al, byte ptr [esi\*2+438840]
00421912  |>|||je      short 0042191C                         ;  在特定串中找与名的第 EBP 个字符相同的位置
00421914  |>|||inc     esi
00421915  |>|||cmp     esi, 34
00421918  |>||\jl      short 0042190B
0042191A  |>||jmp     short 0042192D
0042191C  |>||mov     cl, byte ptr [esi\*2+438841]             ;  取注册码字符
00421923  |>||push    ecx
00421924  |>||lea     ecx, dword ptr [esp+38]
00421928  |>||call    <jmp.&MFC42.#940\_CString::operator+=>   ;  Code+=cl;//连接生成注册码串
0042192D  |>||cmp     esi, 34
00421930  |>||jnz     short 00421940
00421932  |>||mov     edx, dword ptr [esp+18]
00421936  |>||lea     ecx, dword ptr [esp+34]
0042193A  |>||push    edx
0042193B  |>||call    <jmp.&MFC42.#940\_CString::operator+=>   ;  未查到字符的算法
00421940  |>||mov     edi, ebx
00421942  |>||or      ecx, FFFFFFFF
00421945  |>||xor     eax, eax
00421947  |>||inc     ebp
00421948  |>||repne   scas byte ptr es:[edi]
0042194A  |>||not     ecx
0042194C  |>||dec     ecx                                     ;  串长
0042194D  |>||cmp     ebp, ecx                                ;  处理到串尾了吗？
0042194F  |>|\jb      short 00421906                          ;  未完继续
00421951  |>|mov     eax, dword ptr [esp+34]                  ;  生成了一个与名长等长的串，为注册码的一部分,记为str1
00421955  |>|mov     ecx, dword ptr [eax-8]
00421958  |>|cmp     ecx, 10
0042195B  |>|jge     short 00421997                           ;  串长大于等于１６就去比较
0042195D  |>|mov     eax, ecx
0042195F  |>|mov     ecx, 10
00421964  |>|sub     ecx, eax
00421966  |>|lea     edx, dword ptr [esp+1C]
0042196A  |>|push    ecx
0042196B  |>|push    edx
0042196C  |>|mov     ecx, 00438EC4
00421971  |>|call    <jmp.&MFC42.#4129\_CString::Left>         ;  取特定字串左边的 １６－名长 个字符，记为str2
00421976  |>|push    eax
00421977  |>|lea     ecx, dword ptr [esp+38]
0042197B  |>|mov     byte ptr [esp+30], 3
00421980  |>|call    <jmp.&MFC42.#939\_CString::operator+=>    ;  注册码＝str1+str2
00421985  |>|lea     ecx, dword ptr [esp+1C]
00421989  |>|mov     byte ptr [esp+2C], 2
0042198E  |>|call    <jmp.&MFC42.#800\_CString::~CString>
00421993  |>|mov     eax, dword ptr [esp+34]
00421997  |>|mov     ecx, dword ptr [esp+20]
0042199B  |>|push    ecx                                      ; /你输入的码串
0042199C  |>|push    eax                                      ; |真正的注册码，可做内存注册机
0042199D  |>|call    dword ptr [<&MSVCRT.\_mbscmp>]            ; \比较注册码是否正确
004219A3  |>|add     esp, 8
004219A6  |>|lea     ecx, dword ptr [esp+34]
004219AA  |>|test    eax, eax
004219AC  |>|mov     byte ptr [esp+2C], 1
004219B1  |>|je      short 004219CE                           ;  两串相等就好了，爆破点，jmp就OK了
004219B3  |>|xor     esi, esi
004219B5  |>|call    <jmp.&MFC42.#800\_CString::~CString>
004219BA  |>|mov     eax, dword ptr [esp+38]
004219BE  |>|inc     eax
004219BF  |>|cmp     eax, 3                                   ;  是否计算了３次
004219C2  |>|mov     dword ptr [esp+38], eax                  ;  计算次数
004219C6  |>\jl      004218DA                                 ;  失败次数小于３，再算一遍，编程者很有耐心啊:)
004219CC  |>jmp     short 004219D8
004219CE  |>mov     esi, 1                                    ;  成功的标志
004219D3  |>call    <jmp.&MFC42.#800\_CString::~CString>
004219D8  |>lea     ecx, dword ptr [esp+10]
004219DC  |>mov     byte ptr [esp+2C], 0
004219E1  |>call    <jmp.&MFC42.#800\_CString::~CString>
004219E6  |>lea     ecx, dword ptr [esp+14]
004219EA  |>mov     dword ptr [esp+2C], -1
004219F2  |>call    <jmp.&MFC42.#800\_CString::~CString>
```

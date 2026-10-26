
```
0040FBB1    E8 9A302C00     call <jmp.&MFC42.#537\_CString::CString>
0040FBB6    8D4424 28       lea eax,dword ptr ss:[esp+28]
0040FBBA    50              push eax                                      ;将试炼码压入堆栈
0040FBBB    E8 F01AFFFF     call G2F.004016B0                             ;又一call,跟进
0040FBC0    83C4 08         add esp,8
0040FBC3    8B4C24 24       mov ecx,dword ptr ss:[esp+24]                 ;取得call后的结果
0040FBC7    8A140E          mov dl,byte ptr ds:[esi+ecx]
0040FBCA    8D4C24 04       lea ecx,dword ptr ss:[esp+4]
0040FBCE    885424 08       mov byte ptr ss:[esp+8],dl
0040FBD2    8B4424 08       mov eax,dword ptr ss:[esp+8]
0040FBD6    50              push eax
0040FBD7    E8 8A2F2C00     call <jmp.&MFC42.#940\_CString::operator+=>
0040FBDC    83C6 02         add esi,2
0040FBDF    83FE 0C         cmp esi,0C
0040FBE2  ^ 7C DF           jl short G2F.0040FBC3            ;依次取第0、2、4、6、8、10、12共6位，并将结果送给eax
0040FBE4    8B7424 20       mov esi,dword ptr ss:[esp+20]
0040FBE8    8D4C24 04       lea ecx,dword ptr ss:[esp+4]
0040FBEC    51              push ecx
0040FBED    8BCE            mov ecx,esi
0040FBEF    E8 602F2C00     call <jmp.&MFC42.#535\_CString::CString>
0040FBF4    C74424 0C 01000>mov dword ptr ss:[esp+C],1
0040FBFC    8D4C24 04       lea ecx,dword ptr ss:[esp+4]
0040FC00    C64424 18 01    mov byte ptr ss:[esp+18],1
0040FC05    E8 C62E2C00     call <jmp.&MFC42.#800\_CString::~CString>
0040FC0A    8D4C24 24       lea ecx,dword ptr ss:[esp+24]
0040FC0E    C64424 18 00    mov byte ptr ss:[esp+18],0
0040FC13    E8 B82E2C00     call <jmp.&MFC42.#800\_CString::~CString>
0040FC18    8B4C24 10       mov ecx,dword ptr ss:[esp+10]
0040FC1C    8BC6            mov eax,esi
0040FC1E    64:890D 0000000>mov dword ptr fs:[0],ecx
0040FC25    5E              pop esi
0040FC26    83C4 18         add esp,18
0040FC29    C3              retn
```

就拿试炼码来说，经过地址`0040FBBB`处的`call`计算得到的结果是"BC 72 D9 EC AB 02 AA 97 6E EC 91 BC",依次取第0、2、4、6、8、10、12位即"BC D9 AB AA 6E 91",然后于ASCII "&G#2f#"比较，即"26 47 23 32 66 23"比较，如果相等则注册成功。那么地址`0040FBBB`处的`call`是如何运行的呢？跟进`call G2F.004016B0`

```
004016B0    6A FF           push -1
004016B2    68 F84B7900     push G2F.00794BF8
004016B7    64:A1 00000000  mov eax,dword ptr fs:[0]
004016BD    50              push eax
004016BE    64:8925 0000000>mov dword ptr fs:[0],esp
004016C5    83EC 0C         sub esp,0C
004016C8    53              push ebx
004016C9    55              push ebp
004016CA    56              push esi
004016CB    57              push edi
004016CC    33FF            xor edi,edi
004016CE    8D4C24 10       lea ecx,dword ptr ss:[esp+10]
004016D2    897C24 24       mov dword ptr ss:[esp+24],edi
004016D6    E8 07142D00     call <jmp.&MFC42.#540\_CString::CString>
004016DB    8D4C24 14       lea ecx,dword ptr ss:[esp+14]
004016DF    C64424 24 01    mov byte ptr ss:[esp+24],1
004016E4    E8 F9132D00     call <jmp.&MFC42.#540\_CString::CString>
004016E9    8B7424 2C       mov esi,dword ptr ss:[esp+2C]
004016ED    BB 02000000     mov ebx,2
004016F2    8D4424 2C       lea eax,dword ptr ss:[esp+2C]
004016F6    53              push ebx
004016F7    50              push eax
004016F8    8BCE            mov ecx,esi                              ;取得试炼码
004016FA    885C24 2C       mov byte ptr ss:[esp+2C],bl
004016FE    E8 7B142D00     call <jmp.&MFC42.#5710\_CString::Right>   ;取试炼码的右边2位，即最后2位，即"56"
00401703    50              push eax
00401704    8D4C24 14       lea ecx,dword ptr ss:[esp+14]
00401708    C64424 28 03    mov byte ptr ss:[esp+28],3
0040170D    E8 36142D00     call <jmp.&MFC42.#858\_CString::operator=>
00401712    8D4C24 2C       lea ecx,dword ptr ss:[esp+2C]
00401716    885C24 24       mov byte ptr ss:[esp+24],bl
0040171A    E8 B1132D00     call <jmp.&MFC42.#800\_CString::~CString>
0040171F    8B0E            mov ecx,dword ptr ds:[esi]
00401721    8D5424 2C       lea edx,dword ptr ss:[esp+2C]
00401725    8B41 F8         mov eax,dword ptr ds:[ecx-8]                 ;取得试炼码位数，即1A
00401728    8BCE            mov ecx,esi                                  ;取得试炼码
0040172A    83C0 FE         add eax,-2                                   ;eax=18(十进制24）
0040172D    50              push eax
0040172E    52              push edx
0040172F    E8 44142D00     call <jmp.&MFC42.#4129\_CString::Left>        ;取得试炼码左边24位，即试炼码去掉最后2位
00401734    50              push eax
00401735    8BCE            mov ecx,esi
00401737    C64424 28 04    mov byte ptr ss:[esp+28],4
0040173C    E8 07142D00     call <jmp.&MFC42.#858\_CString::operator=>
00401741    8D4C24 2C       lea ecx,dword ptr ss:[esp+2C]
00401745    885C24 24       mov byte ptr ss:[esp+24],bl
00401749    E8 82132D00     call <jmp.&MFC42.#800\_CString::~CString>
0040174E    8B4424 10       mov eax,dword ptr ss:[esp+10]
00401752    50              push eax
00401753    FF15 68457D00   call dword ptr ds:[<&MSVCRT.atoi>]            ; MSVCRT.atoi  将试炼码最后2位转化为数字，即56，即eax=38H=56D
00401759    8D5424 34       lea edx,dword ptr ss:[esp+34]
0040175D    8BCC            mov ecx,esp
0040175F    896424 30       mov dword ptr ss:[esp+30],esp
00401763    52              push edx
00401764    8BE8            mov ebp,eax                                  ;ebp=eax=38H
00401766    E8 E9132D00     call <jmp.&MFC42.#535\_CString::CString>
0040176B    56              push esi                                     ;将试炼码的前24位压入堆栈
0040176C    E8 2F030000     call G2F.00401AA0                            ;又一个call,此call将试炼码的前24位转化为"bc72d9ecab02aa976eec91bc"
00401771    83C4 08         add esp,8
00401774    8D442D 00       lea eax,dword ptr ss:[ebp+ebp]               ;eax=2\*ebp=70H
00401778    8D4C24 2C       lea ecx,dword ptr ss:[esp+2C]
0040177C    50              push eax
0040177D    51              push ecx
0040177E    8BCE            mov ecx,esi                                  ;ecx指向"bc72d9ecab02aa976eec91bc"
00401780    E8 F3132D00     call <jmp.&MFC42.#4129\_CString::Left>        ;取"bc72d9ecab02aa976eec91bc"左边的70H位，由此可见，试炼码最后2位必须是数字，且大于12（十进制）
00401785    50              push eax
00401786    8D4C24 18       lea ecx,dword ptr ss:[esp+18]
0040178A    C64424 28 05    mov byte ptr ss:[esp+28],5
0040178F    E8 B4132D00     call <jmp.&MFC42.#858\_CString::operator=>
00401794    8D4C24 2C       lea ecx,dword ptr ss:[esp+2C]
00401798    885C24 24       mov byte ptr ss:[esp+24],bl
0040179C    E8 2F132D00     call <jmp.&MFC42.#800\_CString::~CString>
004017A1    8BCE            mov ecx,esi
004017A3    E8 CA132D00     call <jmp.&MFC42.#2614\_CString::Empty>       ;此处判断试炼码最后2位是否包含数字，如有则不跳，没有则跳，此处应不跳
004017A8    3BEF            cmp ebp,edi
004017AA    7E 52           jle short G2F.004017FE
004017AC    53              push ebx
004017AD    8D5424 30       lea edx,dword ptr ss:[esp+30]
004017B1    57              push edi
004017B2    52              push edx
004017B3    8D4C24 20       lea ecx,dword ptr ss:[esp+20]
004017B7    E8 B0132D00     call <jmp.&MFC42.#4278\_CString::Mid>
004017BC    50              push eax
004017BD    8D4C24 14       lea ecx,dword ptr ss:[esp+14]
004017C1    C64424 28 06    mov byte ptr ss:[esp+28],6
004017C6    E8 7D132D00     call <jmp.&MFC42.#858\_CString::operator=>
004017CB    8D4C24 2C       lea ecx,dword ptr ss:[esp+2C]
004017CF    885C24 24       mov byte ptr ss:[esp+24],bl
004017D3    E8 F8122D00     call <jmp.&MFC42.#800\_CString::~CString>
004017D8    51              push ecx
004017D9    8D4424 14       lea eax,dword ptr ss:[esp+14]
004017DD    8BCC            mov ecx,esp
004017DF    896424 1C       mov dword ptr ss:[esp+1C],esp
004017E3    50              push eax
004017E4    E8 6B132D00     call <jmp.&MFC42.#535\_CString::CString>
004017E9    E8 920B0000     call G2F.00402380
004017EE    83C4 04         add esp,4
004017F1    8BCE            mov ecx,esi
004017F3    50              push eax
004017F4    E8 6D132D00     call <jmp.&MFC42.#940\_CString::operator+=>
```


```asm
00434426             lea     ecx, [ebp-48h]  ; 连接上第四行字符串

00434429             call    ds:std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>::operator+=(wchar\_t const \*)

0043442F             push    offset a224c4e11 ; "224c4e11"

00434434             lea     ecx, [ebp-48h]  ; 连接上字符串"224c4e11"

00434437             call    ds:std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>::operator+=(wchar\_t const \*)

0043443D             push    edi

0043443E             lea     ecx, [ebp-2Ch]

00434441             call    ds:std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>(wchar\_t const \*)

00434447             mov     byte ptr [ebp-4], 0Ah

0043444B             lea     eax, [ebp-2Ch]

0043444E             push    eax

0043444F             lea     eax, [ebp-74h]  ; 第四行

00434452             push    eax

00434453             call    sub\_434B8E      ; 取第一个空格后面的字符串

00434458             pop     ecx

00434459             pop     ecx

0043445A             mov     byte ptr [ebp-4], 0Bh

0043445E             push    eax

0043445F             mov     ecx, offset lic\_73D02C ; 这里保存协议类型

00434464             call    ds:std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>::operator=(std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>> const &)

0043446A             mov     byte ptr [ebp-4], 0Ah

0043446E             lea     ecx, [ebp-74h]

00434471             call    ds:std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>::~basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>(void)

00434477             mov     byte ptr [ebp-4], 9

0043447B             lea     ecx, [ebp-2Ch]

0043447E             call    ds:std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>::~basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>(void)

00434484             mov     esi, offset lic\_73D02C ; 取协议类型

00434489             call    sub\_433965      ; 把协议类型字符串转换成数字

0043448E             push    ebx

0043448F             push    edi

00434490             lea     ecx, [ebp-278h]

00434496             mov     licnum\_73CF2C, eax ; 保存在这里

0043449B             call    ds:std::basic\_istream<wchar\_t,std::char\_traits<wchar\_t>>::getline(wchar\_t \*,int)

004344A1             push    edi             ; 取第五行，即版本

004344A2             lea     ecx, [ebp-48h]  ; 连接上第五行字符串

004344A5             call    ds:std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>::operator+=(wchar\_t const \*)

004344AB             push    ebx

004344AC             push    edi

004344AD             lea     ecx, [ebp-278h]

004344B3             call    ds:std::basic\_istream<wchar\_t,std::char\_traits<wchar\_t>>::getline(wchar\_t \*,int)

004344B9             push    edi             ; 取第六行，即注册日期

004344BA             lea     ecx, [ebp-48h]  ; 连接上第六行字符串

004344BD             call    ds:std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>::operator+=(wchar\_t const \*)

004344C3             push    edi

004344C4             lea     ecx, [ebp-2Ch]

004344C7             call    ds:std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>(wchar\_t const \*)

004344CD             mov     byte ptr [ebp-4], 0Ch

004344D1             lea     eax, [ebp-2Ch]

004344D4             push    eax

004344D5             lea     eax, [ebp-74h]  ; 第六行

004344D8             push    eax

004344D9             call    sub\_434B8E      ; 取第一个空格后面的字符串

004344DE             pop     ecx

004344DF             pop     ecx

004344E0             mov     byte ptr [ebp-4], 0Dh

004344E4             push    eax

004344E5             mov     ecx, offset date\_73D010 ; 这里保存注册日期

004344EA             call    ds:std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>::operator=(std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>> const &)

004344F0             mov     byte ptr [ebp-4], 0Ch

004344F4             lea     ecx, [ebp-74h]

004344F7             call    ds:std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>::~basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>(void)

004344FD             mov     byte ptr [ebp-4], 9

00434501             lea     ecx, [ebp-2Ch]

00434504             call    ds:std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>::~basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>(void)

0043450A             push    ebx

0043450B             push    edi

0043450C             lea     ecx, [ebp-278h]

00434512             call    ds:std::basic\_istream<wchar\_t,std::char\_traits<wchar\_t>>::getline(wchar\_t \*,int)

00434518             push    edi             ; 取第七行，即服务时间

00434519             lea     ecx, [ebp-2Ch]

0043451C             call    ds:std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>(wchar\_t const \*)

00434522             mov     byte ptr [ebp-4], 0Eh

00434526             lea     eax, [ebp-2Ch]

00434529             push    eax

0043452A             lea     eax, [ebp-154h]

00434530             push    eax

00434531             call    sub\_434BD6      ; 取第一个空格前面的字符串

00434536             pop     ecx

00434537             pop     ecx

00434538             mov     byte ptr [ebp-4], 10h

0043453C             lea     ecx, [ebp-2Ch]

0043453F             call    ds:std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>::~basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>(void)

00434545             mov     esi, ds:std::operator==<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>(std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>> const &,std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>> const &)

0043454B             lea     eax, [ebp-154h]

00434551             push    offset unk\_73D0F8 ; "MaintenancePlan:"

00434556             push    eax

00434557             call    esi ; std::operator==<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>(std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>> const &,std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>> const &)

00434559             test    al, al

0043455B             pop     ecx

0043455C             pop     ecx

0043455D             jz      short loc\_4345CB ; 不等则跳

0043455F             push    edi

00434560             lea     ecx, [ebp-48h]  ; 连接上第七行

00434563             call    ds:std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>::operator+=(wchar\_t const \*)

00434569             push    edi

0043456A             lea     ecx, [ebp-2Ch]

0043456D             call    ds:std::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>::basic\_string<wchar\_t,std::char\_traits<wchar\_t>,std::allocator<wchar\_t>>(wchar\_t const \*)

00434573             mov     byte ptr [ebp-4], 11h

00434577             lea     eax, [ebp-2Ch]

0043457A             push    eax

0043457B             lea     eax, [ebp-74h]

0043457E             push    eax

0043457F             call    sub\_434B8E      ; 取第一个空格后面的字符串

00434584             mov     byte ptr [ebp-4], 12h

00434588             push    offset MPL\_73CF20

0043458D             push    offset MPH\_73CF24

00434592             mov     edi, eax

00434594             call    sub\_43BE77      ; 服务时间转换成数字保存
```

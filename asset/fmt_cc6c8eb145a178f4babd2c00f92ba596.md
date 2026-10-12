
```
00434577            lea    eax, [ebp-2Ch]

0043457A            push   eax

0043457B            lea    eax, [ebp-74h]

0043457E            push   eax

0043457F            call   `sub_434B8E`      ; 取第一个空格后面的字符串

00434584            mov    byte ptr [ebp-4], 12h

00434588            push   offset `MPL_73CF20`

0043458D            push   offset `MPH_73CF24`

00434592            mov    edi, eax

00434594            call   `sub_43BE77`      ; 服务时间转换成数字保存

00434599            add    esp, 10h

0043459C            mov    byte ptr [ebp-4], 11h

004345A0            lea    ecx, [ebp-74h]

004345A3            call   ds:`std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::~basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>(void)`

004345A9            mov    byte ptr [ebp-4], 10h

004345AD            lea    ecx, [ebp-2Ch]

004345B0            call   ds:`std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::~basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>(void)`

004345B6            push   ebx

004345B7            push   dword ptr [ebp-54h]

004345BA            lea    ecx, [ebp-278h]

004345C0            call   ds:`std::basic_istream<wchar_t,std::char_traits<wchar_t>>::getline(wchar_t *,int)`

004345C6            mov    edi, [ebp-54h]  ; 取第八行，即校验码

004345C9            jmp    short `loc_4345DC`

004345CB

`004345CB loc_4345CB:`

004345CB            and    `MPL_73CF20`, 0

004345D2            mov    `MPH_73CF24`, 1   ; 如果没有服务时间则默认是 1 年

004345DC

`004345DC loc_4345DC:`

004345DC            push   edi

004345DD            lea    ecx, [ebp-2Ch]

004345E0            call   ds:`std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>(wchar_t const *)`

004345E6            mov    byte ptr [ebp-4], 13h

004345EA            lea    eax, [ebp-2Ch]

004345ED            push   eax

004345EE            lea    eax, [ebp-0ACh] ; 第八行

004345F4            push   eax

004345F5            call   `sub_434B8E`      ; 取第一个空格后面的字符串

004345FA            pop    ecx

004345FB            pop    ecx

004345FC            mov    byte ptr [ebp-4], 15h

00434600            lea    ecx, [ebp-2Ch]

00434603            call   ds:`std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::~basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>(void)`

00434609            push   3

0043460B            push   0

0043460D            lea    eax, [ebp-1C4h] ; 这里保存校验码前 3 个字符

00434613            push   eax

00434614            lea    ecx, [ebp-0ACh]

0043461A            call   ds:`std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::substr(uint,uint)`

00434620            mov    byte ptr [ebp-4], 16h

00434624            push   20h

00434626            push   3

00434628            lea    eax, [ebp-1A8h] ; 这里保存校验码中间 32 个字符

0043462E            push   eax

0043462F            lea    ecx, [ebp-0ACh]

00434635            call   ds:`std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::substr(uint,uint)`

0043463B            mov    byte ptr [ebp-4], 17h

0043463F            push   4

00434641            push   23h

00434643            lea    eax, [ebp-1E0h] ; 这里保存校验码后 4 个字符

00434649            push   eax

0043464A            lea    ecx, [ebp-0ACh]

00434650            call   ds:`std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>::substr(uint,uint)`

00434656            mov    byte ptr [ebp-4], 18h

0043465A            lea    eax, [ebp-0C8h]

00434660            push   eax

00434661            lea    ecx, [ebp-48h]

00434664            call   `sub_43B9A3`

00434669            pop    ecx

0043466A            mov    byte ptr [ebp-4], 19h

0043466E            cmp    dword ptr [ebp-0B0h], 10h

00434675            mov    edx, [ebp-0C4h]

0043467B            jnb    short `loc_434683`

0043467D            lea    edx, [ebp-0C4h]

00434683

`00434683 loc_434683:`

00434683            lea    eax, [ebp-300h] ; 这里是前面连接在一起的那个字符串

00434689            call   `MD5_47757C`      ; 求 md5

0043468E            lea    eax, [ebp-300h]

00434694            call   `sub_4775A4`      ; 转换成字符串

00434699            mov    edi, eax

0043469B            mov    edx, edi

0043469D            lea    eax, [ebp-300h]

004346A3            call   `MD5_47757C`      ; 求 md5

004346A8            lea    eax, [ebp-300h]

004346AE            call   `sub_4775A4`      ; 转换成字符串

004346B3            push   edi

004346B4            lea    ecx, [ebp-170h] ; 第一次求 md5 后的字符串

004346BA            mov    ebx, eax

004346BC            call   ds:`std::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string<char,std::char_traits<char>,std::allocator<char>>(char const *)`

004346C2            mov    byte ptr [ebp-4], 1Ah

004346C6            push   ebx

004346C7            lea    ecx, [ebp-11Ch] ; 第二次求 md5 后的字符串

004346CD            call   ds:`std::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string<char,std::char_traits<char>,std::allocator<char>>(char const *)`

004346D3            mov    byte ptr [ebp-4], 1Bh

004346D7            push   3

004346D9            push   5

004346DB            lea    eax, [ebp-18Ch]

004346E1            push   eax

004346E2            lea    ecx, [ebp-11Ch] ; 从第 6 个字符起取 3 个字符

004346E8            call   ds:`std::basic_string<char,std::char_traits<char>,std::allocator<char>>::substr(uint,uint)`

004346EE            mov    byte ptr [ebp-4], 1Ch

004346F2            push   4

004346F4            push   10

004346F6            lea    eax, [ebp-138h]

004346FC            push   eax

004346FD            lea    ecx, [ebp-11Ch] ; 从第 11 个字符起取 4 个字符

00434703            call   ds:`std::basic_string<char,std::char_traits<char>,std::allocator<char>>::substr(uint,uint)`

00434709            mov    byte ptr [ebp-4], 1Dh

0043470D            push   edi

0043470E            call   `operator delete(void *)`

00434713            push   ebx

00434714            call   `operator delete(void *)`

00434719            lea    eax, [ebp-170h]

0043471F            push   eax

00434720            lea    eax, [ebp-294h]

00434726            push   eax

00434727            call   `sub_43A9F1`

0043472C            mov    byte ptr [ebp-4], 1Eh

00434730            lea    ecx, [ebp-1A8h]

00434736            push   ecx

00434737            push   eax

00434738            mov    dword ptr [ebp-10h], 1

0043473F            call   esi ; `std::operator==<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>>(std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>> const &,std::basic_string<wchar_t,std::char_traits<wchar_t>,std::allocator<wchar_t>> const &)`

00434741            add    esp, 18h

00434744            test   al, al           ; 两个 32 字符长的字符串比较

00434746            push   31

00434748            pop    edi

00434749            jz     short `loc_4347A9`

0043474B            lea    eax, [ebp-18Ch]

00434751            push   eax

00434752            lea    eax, [ebp-2Ch]

00434755            push   eax

00434756            call   `sub_43A9F1`
```


``` 
0048A74A        .  E8 319EF>call RamSmash.00404580
0048A74F        .  8B55 E4  mov edx,dword ptr ss:[ebp-1C]
0048A752        .  B8 BCAA4>mov eax,RamSmash.0048AABC
0048A757        .  E8 089FF>call RamSmash.00404664             ; --->以"+"为分格，取Serial第2段
0048A75C        .  48       dec eax
0048A75D        .  50       push eax
0048A75E        .  8D45 E0  lea eax,dword ptr ss:[ebp-20]
0048A761        .  50       push eax
0048A762        .  8B55 F8  mov edx,dword ptr ss:[ebp-8]
0048A765        .  B8 BCAA4>mov eax,RamSmash.0048AABC
0048A76A        .  E8 F59EF>call RamSmash.00404664             ; --->以"+"为分格，取Serial第1段
0048A76F        .  40       inc eax
0048A770        .  50       push eax
0048A771        .  8B45 F8  mov eax,dword ptr ss:[ebp-8]
0048A774        .  E8 A79BF>call RamSmash.00404320
0048A779        .  8BC8     mov ecx,eax
0048A77B        .  8B45 F8  mov eax,dword ptr ss:[ebp-8]
0048A77E        .  5A       pop edx
0048A77F        .  E8 FC9DF>call RamSmash.00404580
0048A784        .  8B45 E0  mov eax,dword ptr ss:[ebp-20]
0048A787        .  BA 01000>mov edx,1
0048A78C        .  59       pop ecx
0048A78D        .  E8 EE9DF>call RamSmash.00404580
0048A792        .  BB 01000>mov ebx,1
0048A797        >  8D45 D8  lea eax,dword ptr ss:[ebp-28]      ; --->开始计算Name
0048A79A        .  50       push eax                           ; --->这是一个循环，对Name的每一位进行运算
0048A79B        .  B9 01000>mov ecx,1
0048A7A0        .  8BD3     mov edx,ebx
0048A7A2        .  8B45 FC  mov eax,dword ptr ss:[ebp-4]
0048A7A5        .  E8 D69DF>call RamSmash.00404580
0048A7AA        .  8B45 D8  mov eax,dword ptr ss:[ebp-28]
0048A7AD        .  0FB600   movzx eax,byte ptr ds:[eax]        ; --->EAX=第i位Name字符(ASCII码)Name[i]
0048A7B0        .  F7EB     imul ebx                           ; --->于i相乘
0048A7B2        .  8945 D4  mov dword ptr ss:[ebp-2C],eax
0048A7B5        .  DB45 D4  fild dword ptr ss:[ebp-2C]
0048A7B8        .  E8 D782F>call RamSmash.00402A94
0048A7BD        .  8945 CC  mov dword ptr ss:[ebp-34],eax
0048A7C0        .  8955 D0  mov dword ptr ss:[ebp-30],edx
0048A7C3        .  DF6D CC  fild qword ptr ss:[ebp-34]
0048A7C6        .  83C4 F4  add esp,-0C
0048A7C9        .  DB3C24   fstp tbyte ptr ss:[esp]            ; |
0048A7CC        .  9B       wait                               ; |
0048A7CD        .  8D55 DC  lea edx,dword ptr ss:[ebp-24]      ; |
0048A7D0        .  B8 C8AA4>mov eax,RamSmash.0048AAC8          ; |
0048A7D5        .  E8 8AF5F>call RamSmash.00409D64             ; --->将Name[i]的10进制ASCII码转换位字符串
0048A7DA        .  FF75 DC  push dword ptr ss:[ebp-24]
0048A7DD        .  8D55 C8  lea edx,dword ptr ss:[ebp-38]
0048A7E0        .  8BC3     mov eax,ebx
0048A7E2        .  E8 05DDF>call RamSmash.004084EC
0048A7E7        .  FF75 C8  push dword ptr ss:[ebp-38]
0048A7EA        .  FF35 841>push dword ptr ds:[491F84]
0048A7F0        .  8D45 EC  lea eax,dword ptr ss:[ebp-14]
0048A7F3        .  BA 03000>mov edx,3
0048A7F8        .  E8 E39BF>call RamSmash.004043E0             ; --->将Name[i]按10进制转换的字符串与位数按10进制转换的字符串、Serial第2段连接
0048A7FD        .  8B45 EC  mov eax,dword ptr ss:[ebp-14]
0048A800        .  E8 23DEF>call RamSmash.00408628
0048A805        .  8BF0     mov esi,eax
0048A807        .  8B45 EC  mov eax,dword ptr ss:[ebp-14]
0048A80A        .  E8 19DEF>call RamSmash.00408628             ; --->将该字符串按10进制转换为数字
0048A80F        .  03F0     add esi,eax                        ; --->该数乘2
0048A811        .  8BC6     mov eax,esi
0048A813        .  8D55 C4  lea edx,dword ptr ss:[ebp-3C]
0048A816        .  E8 D1DCF>call RamSmash.004084EC             ; --->将该数转换为字符串
0048A81B        .  8B55 C4  mov edx,dword ptr ss:[ebp-3C]
0048A81E        .  8D45 EC  lea eax,dword ptr ss:[ebp-14]
0048A821        .  E8 D298F>call RamSmash.004040F8
0048A826        .  43       inc ebx
0048A827        .  8B45 FC  mov eax,dword ptr ss:[ebp-4]
0048A82A        .  E8 F19AF>call RamSmash.00404320
0048A82F        .  40       inc eax
0048A830        .  3BD8     cmp ebx,eax
0048A832        .^ 0F85 5FF>jnz RamSmash.0048A797
0048A838        .  6A 16    push 16                            ; --->结束计算Name
0048A83A        .  68 7C7DC>push 56C77D7C
0048A83F        .  8B45 EC  mov eax,dword ptr ss:[ebp-14]
0048A842        .  E8 E1DDF>call RamSmash.00408628             ; --->将该字符串按10进制转换为数字
0048A847        .  99       cdq
0048A848        .  E8 0FA7F>call RamSmash.00404F5C             ; --->该数\*0x1656C77D7C
0048A84D        .  8945 CC  mov dword ptr ss:[ebp-34],eax
0048A850        .  8955 D0  mov dword ptr ss:[ebp-30],edx      ; --->结果按双精度数压入堆栈
0048A853        .  DF6D CC  fild qword ptr ss:[ebp-34]
0048A856        .  83C4 F4  add esp,-0C
0048A859        .  DB3C24   fstp tbyte ptr ss:[esp]            ; --->双精度转换为单精度
0048A85C        .  9B       wait                               ; |
0048A85D        .  8D55 C0  lea edx,dword ptr ss:[ebp-40]      ; |
0048A860        .  B8 C8AA4>mov eax,RamSmash.0048AAC8          ; |
0048A865        .  E8 FAF4F>call RamSmash.00409D64             ; --->将该单精度数按科学计数法转换为字符串M
0048A86A        .  8B55 C0  mov edx,dword ptr ss:[ebp-40]
0048A86D        .  8D45 EC  lea eax,dword ptr ss:[ebp-14]
0048A870        .  E8 8398F>call RamSmash.004040F8
0048A875        .  8D45 F0  lea eax,dword ptr ss:[ebp-10]
0048A878        .  50       push eax
0048A879        .  8B55 F8  mov edx,dword ptr ss:[ebp-8]
0048A87C        .  B8 BCAA4>mov eax,RamSmash.0048AABC
0048A881        .  E8 DE9DF>call RamSmash.00404664
0048A886        .  8BC8     mov ecx,eax                        ; --->ECX=Serial第1段长度
0048A888        .  49       dec ecx
0048A889        .  BA 01000>mov edx,1
0048A88E        .  8B45 F8  mov eax,dword ptr ss:[ebp-8]
0048A891        .  E8 EA9CF>call RamSmash.00404580             ; --->取得Serial第1段
0048A896        .  8B45 F0  mov eax,dword ptr ss:[ebp-10]
0048A899        .  E8 1EF5F>call RamSmash.00409DBC             ; --->按科学计数法将Serial第1段转换为单精度数
0048A89E        .  DB7D B4  fstp tbyte ptr ss:[ebp-4C]
0048A8A1        .  9B       wait
0048A8A2        .  8B45 EC  mov eax,dword ptr ss:[ebp-14]
0048A8A5        .  E8 12F5F>call RamSmash.00409DBC             ; --->按科学计数法将Name计算结果字符串M转换为单精度数
0048A8AA        .  DB6D B4  fld tbyte ptr ss:[ebp-4C]
0048A8AD        .  DEE1     fsubrp st(1),st
0048A8AF        .  D81D CCA>fcomp dword ptr ds:[48AACC]
0048A8B5        .  DFE0     fstsw ax
0048A8B7        .  9E       sahf
0048A8B8        .  0F87 2E0>ja RamSmash.0048A9EC
0048A8BE        .  8B45 EC  mov eax,dword ptr ss:[ebp-14]
0048A8C1        .  E8 F6F4F>call RamSmash.00409DBC
0048A8C6        .  DB7D A8  fstp tbyte ptr ss:[ebp-58]
0048A8C9        .  9B       wait
0048A8CA        .  8B45 F0  mov eax,dword ptr ss:[ebp-10]
0048A8CD        .  E8 EAF4F>call RamSmash.00409DBC
0048A8D2        .  DB6D A8  fld tbyte ptr ss:[ebp-58]
0048A8D5        .  DEE1     fsubrp st(1),st
0048A8D7        .  D81D CCA>fcomp dword ptr ds:[48AACC]        ; --->两数比较，相差小于0x4000则认证成功，不跳！！！
............................................................
省略代码若干行
中间好像还有和名单检测
............................................................
0048AA65        .  8A45 F7  mov al,byte ptr ss:[ebp-9]
0048AA68        .  5F       pop edi
0048AA69        .  5E       pop esi
0048AA6A        .  5B       pop ebx
0048AA6B        .  8BE5     mov esp,ebp
0048AA6D        .  5D       pop ebp
0048AA6E        .  C3       retn                               ; --->函数结束
```

=============================

=============================

KeyGen

注册算法：

`Name`最后一位的ASCII码与`Name`长度的乘积，按10进制转换为字符串，

然后与`Name`长度按10进制转换的字符串、`License`个数按10进制转换的字符串依次连接，再按10进制转换为数字。

该数乘2后，与95945194876的乘积，如果乘积大于0xFFFF FFFF FFFF FFFF,则忽略高位，只取低8个字节。

结果按科学计数法精确到小数点后13位，转换为字符串，就是`Serial`的第1段。

例如：

`Name="MrBeer[CCG]"`

最后一位是']',ASCII=93,`Name`长度是11,乘积为`93*11`=1023，转换为字符串"1023"

`Name`长度转换为字符串"11"，`License`个数假设为100,转换为字符串是"100"，连接后为"102311100"，转换为数字102311100。

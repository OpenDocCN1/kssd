
```
:0052FFDA 8945F0                mov dword ptr [ebp-10], eax    ;TRegistry 对象指针
:0052FFDD 33C0                  xor eax, eax
:0052FFDF 55                    push ebp
:0052FFE0 681C015300            push 0053011C
:0052FFE5 64FF30                push dword ptr fs:[eax]
:0052FFE8 648920                mov dword ptr fs:[eax], esp

TRegistry 对象的 RootKey := HKCR

:0052FFEB BA00000080            mov edx, 80000000    ;HKCR
:0052FFF0 8B45F0                mov eax, dword ptr [ebp-10]
:0052FFF3 E8888EF3FF            call 00468E80
:0052FFF8 C645EB00              mov [ebp-15], 00

HKCR\RichOleLink.TOleLink.1 这个 Key 存不存在？

* Possible StringData Ref from Code Obj ->"\RichOleLink.TOleLink.1"
                              |
:0052FFFC BA60015300            mov edx, 00530160
:00530001 8B45F0                mov eax, dword ptr [ebp-10]
:00530004 E87797F3FF            call 00469780
:00530009 84C0                  test al, al
:0053000B 0F84C4000000           je 005300D5    ;不存在，跳

HKCR\RichOleLink.TOleLink.1\CLSID 这个 Key 存不存在？

* Possible StringData Ref from Code Obj ->"\RichOleLink.TOleLink.1\CLSID"
                              |
:00530011 BA80015300            mov edx, 00530180
:00530016 8B45F0                mov eax, dword ptr [ebp-10]
:00530019 E86297F3FF            call 00469780
:0053001E 84C0                  test al, al
:00530020 0F84AF000000           je 005300D5    ;不存在，跳

打开(OpenKey)Key:HKCR\RichOleLink.TOleLink.1\CLSID

:00530026 33C9                  xor ecx, ecx

* Possible StringData Ref from Code Obj ->"\RichOleLink.TOleLink.1\CLSID"
                              |
:00530028 BA80015300            mov edx, 00530180
:0053002D 8B45F0                mov eax, dword ptr [ebp-10]
:00530030 E88F8FF3FF            call 00468FC4
读入 key 的缺省值，其实这是一个假 CLSID
:00530035 8D4DEC                lea ecx, dword ptr [ebp-14]
:00530038 33D2                  xor edx, edx    ;nil,default value
:0053003A 8B45F0                mov eax, dword ptr [ebp-10]
:0053003D E8DA94F3FF            call 0046951C    ;ReadString
:00530042 33C0                  xor eax, eax
:00530044 55                    push ebp
:00530045 68CB005300            push 005300CB
:0053004A 64FF30                push dword ptr fs:[eax]
:0053004D 648920                mov dword ptr fs:[eax], esp
取缺省值的第 26 个字符开始的 12 个字符放入另一个字符串
缺省值的形式如同一般的 CLSID,在我的机器上是：
{42BFA701-EC57-0000-C130-0000000092DB}，这是一个假 CLSID，在 HKCR\CLSID 中是没有的
                    ^^^^^^^^^^^^
                    这里正好是第 26 个字符开始的 12 个字符
提取后的字符串是：'0000000092DB'

:00530050 8D45E0                lea eax, dword ptr [ebp-20]
:00530053 50                    push eax
:00530054 B90C000000            mov ecx, 0000000C
:00530059 BA1A000000            mov edx, 0000001A
:0053005E 8B45EC                mov eax, dword ptr [ebp-14]
:00530061 E81642EDFF            call 0040427C
:00530066 8B4DE0                mov ecx, dword ptr [ebp-20] ;->提取的 substring

:00530069 8D45E4                lea eax, dword ptr [ebp-1C]
:0053006C BAA8015300            mov edx, 005301A8    ;'$'
:00530071 E84A40EDFF            call 004040C0
刚才的字符串串首加个字符'$'得到一个新字符串，表示是
十六进制数字字符串，
然后转换成数字
:00530076 8B45E4                mov eax, dword ptr [ebp-1C] ;->新字符串
:00530079 E83E94EDFF            call 004094BC    ;StrToInt
:0053007E 83E846                sub eax, 00000046
:00530081 2B45F8                sub eax, dword ptr [ebp-08]
:00530084 8945F4                mov dword ptr [ebp-0C], eax ;软件已使用天数，用负数表示

取缺省值的第 21 个字符开始的 4 个字符放入另一个字符串
{42BFA701-EC57-0000-C130-0000000092DB}
                   ^^^^这是这 4 个位置上的字符

:00530087 8D45D8                lea eax, dword ptr [ebp-28]
:0053008A 50                    push eax
:0053008B B904000000            mov ecx, 00000004
:00530090 BA15000000            mov edx, 00000015
:00530095 8B45EC                mov eax, dword ptr [ebp-14] ;刚才读入的 default value
:00530098 E8DF41EDFF            call 0040427C
同样前面加上十六进制标识符'$'后，转换成数字
:0053009D 8B4DD8                mov ecx, dword ptr [ebp-28]
:005300A0 8D45DC                lea eax, dword ptr [ebp-24]
:005300A3 BAA8015300            mov edx, 005301A8    ;'$'
:005300A8 E81340EDFF            call 004040C0
:005300AD 8B45DC                mov eax, dword ptr [ebp-24]
:005300B0 E80794EDFF            call 004094BC
:005300B5 2D00C00000            sub eax, 0000C000
:005300BA 3B45FC                cmp eax, dword ptr [ebp-04]
:005300BD 0F9D45EB              setnl byte ptr [ebp-15] ;似乎是比较年份？
:005300C1 33C0                  xor eax, eax
:005300C3 5A                    pop edx
:005300C4 59                    pop ecx
:005300C5 59                    pop ecx
:005300C6 648910                mov dword ptr fs:[eax], edx
:005300C9 EB0A                  jmp 005300D5
:005300CB E94C34EDFF            jmp 0040351C
:005300D0 E8A337EDFF            call 00403878

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:0053000B(C), :00530020(C), :005300C9(U)
|
:005300D5 807DEB00              cmp byte ptr [ebp-15], 00
:005300D9 752B                  jne 00530106
:005300DB B101                  mov cl, 01

* Possible StringData Ref from Code Obj ->"\RichOleLink.TOleLink.1\CLSID"
                              |
:005300DD BA80015300            mov edx, 00530180
:005300E2 8B45F0                mov eax, dword ptr [ebp-10]
:005300E5 E8DA8EF3FF            call 00468FC4
:005300EA 55                    push ebp
:005300EB 8D45D4                lea eax, dword ptr [ebp-2C]
:005300EE E81DFEFFFF             call 0052FF10
:005300F3 59                    pop ecx
:005300F4 8B4DD4                mov ecx, dword ptr [ebp-2C]
:005300F7 33D2                  xor edx, edx
:005300F9 8B45F0                mov eax, dword ptr [ebp-10]
:005300FC E8EF93F3FF            call 004694F0
:00530101 33C0                  xor eax, eax
:00530103 8945F4                mov dword ptr [ebp-0C], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:005300D9(C)
|
:00530106 33C0                  xor eax, eax
:00530108 5A                    pop edx
:00530109 59                    pop ecx
:0053010A 59                    pop ecx
:0053010B 648910                mov dword ptr fs:[eax], edx
:0053010E 6823015300            push 00530123

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00530121(U)
|
:00530113 8B45F0                mov eax, dword ptr [ebp-10]
:00530116 E8552FEDFF            call 00403070
:0053011B C3                    ret
:0053011C E9AF36EDFF            jmp 004037D0
:00530121 EBF0                  jmp 00530113
:00530123 33C0                  xor eax, eax
```

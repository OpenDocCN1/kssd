
```assembly
.text:00419EB2                 push    offset loc_438A9F

.text:00419EB7                 mov     eax, large fs:0

.text:00419EBD                 push    eax

.text:00419EBE                 mov     large fs:0, esp

.text:00419EC5                 sub     esp, 8

.text:00419EC8                 push    esi

.text:00419EC9                 lea     ecx, [esp+18h+var_14]

.text:00419ECD                 mov     [esp+18h+var_10], 0

.text:00419ED5                 call    CString::CString(void)

.text:00419EDA                 mov     ecx, [esp+18h+arg_4] ; 传进来的整数，设为i

.text:00419EDE                 mov     [esp+18h+var_4], 1

.text:00419EE6                 push    ecx

.text:00419EE7                 lea     eax, [ecx+ecx*8]

.text:00419EEA                 shl     eax, 5

.text:00419EED                 add     eax, ecx

.text:00419EEF                 lea     eax, [eax+eax*2]

.text:00419EF2                 lea     eax, [ecx+eax*4]

.text:00419EF5                 lea     ecx, [esp+1Ch+var_14]

.text:00419EF9                 shl     eax, 3

.text:00419EFC                 xor     eax, 6666h

.text:00419F01                 sub     eax, 1B08h

.text:00419F06                 xor     eax, 0FFFF805Bh

.text:00419F0B                 and     eax, 0FFA5h

.text:00419F10                 add     eax, 53h

.text:00419F13                 push    eax       ; (((i * 9 * 32 + i) * 3 * 4 + i) * 8 ^ 0x6666 - 0x1b08) ^ 0xffff805b & 0xffa5 + 53

.text:00419F14                 push    682

.text:00419F19                 push    0E5h

.text:00419F1E                 push    'S'

.text:00419F20                 push    offset aC0_2x0_3d0_4x0 ; "%c%0.2X%0.3d%0.4X%0.6d"

.text:00419F25                 push    ecx

.text:00419F26                 call    CString::Format(char const *,...)

.text:00419F2B                 mov     esi, [esp+34h+arg_0]

.text:00419F2F                 add     esp, 1Ch

.text:00419F32                 lea     edx, [esp+18h+var_14]

.text:00419F36                 mov     ecx, esi

.text:00419F38                 push    edx

.text:00419F39                 call    CString::CString(CString const &)

.text:00419F3E                 mov     [esp+18h+var_10], 1

.text:00419F46                 lea     ecx, [esp+18h+var_14]

.text:00419F4A                 mov     byte ptr [esp+18h+var_4], 0

.text:00419F4F                 call    CString::~CString(void)

.text:00419F54                 mov     ecx, [esp+18h+var_C]

.text:00419F58                 mov     eax, esi

.text:00419F5A                 pop     esi

.text:00419F5B                 mov     large fs:0, ecx

.text:00419F62                 add     esp, 14h

.text:00419F65                 retn    8

.text:00419F65 sub_419EB0      endp
```

呵呵。是不是很简单哦。

### 总结

注册码去掉空格和`-`后长度为16位。取后6位转换成整数，设为`i`（无符号型），则由下面的公式生成一个整数：

`(((i * 9 * 32 + i) * 3 * 4 + i) * 8 ^ 0x6666 - 0x1b08) ^ 0xffff805b & 0xffa5 + 53`

这个整数的16进制形式就是注册码的7-10位。

注册码1-6位固定是 `SE5682`

注册码后6位为一个整数的10进制形式。

【全文完】

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

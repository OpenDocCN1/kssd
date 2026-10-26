
`00401505`  |.  89A5 78FFFFFF mov     dword ptr [ebp-88], esp

`0040150B`  |.  8D55 E8       lea     edx, dword ptr [ebp-18]

`0040150E`  |.  52            push    edx

`0040150F`  |.  E8 B2590400   call    00446EC6

`00401514`  |.  8985 58FFFFFF mov     dword ptr [ebp-A8], eax

`0040151A`  |.  8B85 58FFFFFF mov     eax, dword ptr [ebp-A8]

`00401520`  |.  8985 54FFFFFF mov     dword ptr [ebp-AC], eax

`00401526`  |.  C645 FC 04    mov     byte ptr [ebp-4], 4

`0040152A`  |.  E8 F1060000   call    00401C20

`0040152F`  |.  8BC8          mov     ecx, eax

`00401531`  |.  C645 FC 02    mov     byte ptr [ebp-4], 2

`00401535`  |.  E8 F25F0100   call    0041752C                         ;  算法CALL

`0040153A`  |.  8985 50FFFFFF mov     dword ptr [ebp-B0], eax

`00401540`  |.  83BD 50FFFFFF>cmp     dword ptr [ebp-B0], 0

`00401547`  |.  0F84 DB010000 je      00401728                         ;  关键跳转

`0040154D`  |.  51            push    ecx

`0040154E`  |.  8BCC          mov     ecx, esp

`00401550`  |.  89A5 74FFFFFF mov     dword ptr [ebp-8C], esp

`00401556`  |.  8D55 EC       lea     edx, dword ptr [ebp-14]

`00401559`  |.  52            push    edx

`0040155A`  |.  E8 67590400   call    00446EC6

`0040155F`  |.  8985 4CFFFFFF mov     dword ptr [ebp-B4], eax

`00401565`  |.  8B85 4CFFFFFF mov     eax, dword ptr [ebp-B4]

`0040156B`  |.  8985 48FFFFFF mov     dword ptr [ebp-B8], eax

`00401571`  |.  C645 FC 05    mov     byte ptr [ebp-4], 5

`00401575`  |.  51            push    ecx

`00401576`  |.  8BCC          mov     ecx, esp

`00401578`  |.  89A5 70FFFFFF mov     dword ptr [ebp-90], esp

`0040157E`  |.  8D55 E8       lea     edx, dword ptr [ebp-18]

`00401581`  |.  52            push    edx

`00401582`  |.  E8 3F590400   call    00446EC6

`00401587`  |.  8985 44FFFFFF mov     dword ptr [ebp-BC], eax

`0040158D`  |.  8B85 44FFFFFF mov     eax, dword ptr [ebp-BC]

`00401593`  |.  8985 40FFFFFF mov     dword ptr [ebp-C0], eax

`00401599`  |.  C645 FC 06    mov     byte ptr [ebp-4], 6

`0040159D`  |.  E8 7E060000   call    00401C20

`004015A2`  |.  8BC8          mov     ecx, eax

`004015A4`  |.  C645 FC 02    mov     byte ptr [ebp-4], 2

`004015A8`  |.  E8 D1620100   call    0041787E

`004015AD`  |.  8D4D 80       lea     ecx, dword ptr [ebp-80]

`004015B0`  |.  E8 FB050000   call    00401BB0

`004015B5`  |.  C645 FC 07    mov     byte ptr [ebp-4], 7

`004015B9`  |.  68 D4E14500   push    0045E1D4                         ;  .

`004015BE`  |.  E8 5D060000   call    00401C20

`004015C3`  |.  05 04020000   add     eax, 204

`004015C8`  |.  50            push    eax

`004015C9`  |.  68 D8E14500   push    0045E1D8                         ;  this program is registered to

`004015CE`  |.  8D8D 6CFFFFFF lea     ecx, dword ptr [ebp-94]

`004015D4`  |.  51            push    ecx

----------------------------————————————————————————————————-

略去若干代码

———————————————————————————————————————————————

`00401703`  |.  6A 40         push    40

`00401705`  |.  68 F8E14500   push    0045E1F8                         ;  registration successful!

`0040170A`  |.  68 14E24500   push    0045E214                         ;  thank you for registering the program.

`0040170F`  |.  8B8D 64FFFFFF mov     ecx, dword ptr [ebp-9C]

`00401715`  |.  E8 B13E0400   call    004455CB

`0040171A`  |.  C645 FC 02    mov     byte ptr [ebp-4], 2

`0040171E`  |.  8D4D 80       lea     ecx, dword ptr [ebp-80]

`00401721`  |.  E8 2B5A0400   call    00447151

`00401726`  |.  EB 17         jmp     short 0040173F

`00401728`  |>  6A 10         push    10

`0040172A`  |.  68 3CE24500   push    0045E23C                         ;  registration unsuccessful

`0040172F`  |.  68 58E24500   push    0045E258                         ;  this user name does not match the registration code entered.

`00401734`  |.  8B8D 64FFFFFF mov     ecx, dword ptr [ebp-9C]

`0040173A`  |.  E8 8C3E0400   call    004455CB

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

段首下断，重载。点ABOUT，再点Regster now 断下,单步，中间输入试炼码，进入算法CALL `call 0041752C`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
0041752C  /$  55            push    ebp
0041752D  |.  8BEC          mov     ebp, esp
0041752F  |.  6A FF         push    -1
00417531  |.  68 51F24400   push    0044F251                         ;  SE 处理程序安装
00417536  |.  64:A1 0000000>mov     eax, dword ptr fs:[0]
0041753C  |.  50            push    eax
0041753D  |.  64:8925 00000>mov     dword ptr fs:[0], esp
00417544  |.  83EC 44       sub     esp, 44
00417547  |.  894D CC       mov     dword ptr [ebp-34], ecx
0041754A  |.  C745 FC 01000>mov     dword ptr [ebp-4], 1
00417551  |.  68 000F4600   push    00460F00                         ;  141040
00417556  |.  8D45 0C       lea     eax, dword ptr [ebp+C]
00417559  |.  50            push    eax
0041755A  |.  E8 91BDFEFF   call    004032F0                         ;  判断是否是固定注册码码“141040”
0041755F  |.  25 FF000000   and     eax, 0FF
00417564  |.  85C0          test    eax, eax
00417566  |.  75 17         jnz     short 0041757F                   ;  验证失败则进入第2组注册码判断
00417568  |.  68 080F4600   push    00460F08                         ;  117445
0041756D  |.  8D4D 0C       lea     ecx, dword ptr [ebp+C]
00417570  |.  51            push    ecx
00417571  |.  E8 7ABDFEFF   call    004032F0                         ;  判断是否是固定注册码码“117445”
00417576  |.  25 FF000000   and     eax, 0FF
0041757B  |.  85C0          test    eax, eax
0041757D  |.  74 64         je      short 004175E3                   ;  验证失败则进入另一组注册码判断
0041757F  |>  51            push    ecx
00417580  |.  8BCC          mov     ecx, esp
00417582  |.  8965 E8       mov     dword ptr [ebp-18], esp
00417585  |.  8D55 0C       lea     edx, dword ptr [ebp+C]
00417588  |.  52            push    edx
00417589  |.  E8 38F90200   call    00446EC6
0041758E  |.  8945 C8       mov     dword ptr [ebp-38], eax
00417591  |.  8B45 C8       mov     eax, dword ptr [ebp-38]
00417594  |.  8945 C4       mov     dword ptr [ebp-3C], eax
00417597  |.  C645 FC 02    mov     byte ptr [ebp-4], 2
0041759B  |.  51            push    ecx
0041759C  |.  8BCC          mov     ecx, esp
0041759E  |.  8965 E4       mov     dword ptr [ebp-1C], esp
004175A1  |.  8D55 08       lea     edx, dword ptr [ebp+8]
004175A4  |.  52            push    edx
004175A5  |.  E8 1CF90200   call    00446EC6
004175AA  |.  8945 C0       mov     dword ptr [ebp-40], eax
004175AD  |.  8B4D CC       mov     ecx, dword ptr [ebp-34]
004175B0  |.  C645 FC 01    mov     byte ptr [ebp-4], 1
004175B4  |.  E8 C5020000   call    0041787E
004175B9  |.  C745 E0 FFFFF>mov     dword ptr [ebp-20], -1
004175C0  |.  C645 FC 00    mov     byte ptr [ebp-4], 0
004175C4  |.  8D4D 08       lea     ecx, dword ptr [ebp+8]
004175C7  |.  E8 85FB0200   call    00447151
004175CC  |.  C745 FC FFFFF>mov     dword ptr [ebp-4], -1
004175D3  |.  8D4D 0C       lea     ecx, dword ptr [ebp+C]
004175D6  |.  E8 76FB0200   call    00447151
004175DB  |.  8B45 E0       mov     eax, dword ptr [ebp-20]
004175DE  |.  E9 0A010000   jmp     004176ED
004175E3  |>  8D4D EC       lea     ecx, dword ptr [ebp-14]          ;  第2组注册码失败后跳到这里
004175E6  |.  E8 C5A5FEFF   call    00401BB0
004175EB  |.  C645 FC 03    mov     byte ptr [ebp-4], 3
004175EF  |.  6A 0A         push    0A
004175F1  |.  68 00010000   push    100
004175F6  |.  8D4D EC       lea     ecx, dword ptr [ebp-14]
```

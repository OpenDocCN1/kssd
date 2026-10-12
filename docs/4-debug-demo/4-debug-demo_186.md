
# CoffeeCup Animator7 破解记录

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-786.htm`](https://www.pediy.com/kssd/pediy07/pediy7-786.htm)

CoffeeCup Animator7 破解记录  
`kongfoo`/2004.10.9  

　　Delphi 程序，启动时 NAG 21 天试用期，有几个月没调试东西了，今天因为做网站  
要用这个软件，于是跟踪了一下。  

　　先对一些有用的 API 下断，比如 `CreateFileA`，`ShowWindow` 等。  

```
004093E5    E8 D6D8FFFF      CALL <JMP.&kernel32.CreateFileA>  

0012FD6C   00C10860  |FileName = "D:\WINXP\System32\WinSys16.crc"  

0012FD70   80000000  |Access = GENERIC_READ  

0012FD74   00000000  |ShareMode = 0  

0012FD78   00000000  |pSecurity = NULL  

0012FD7C   00000003  |Mode = OPEN_EXISTING  

0012FD80   00000080  |Attributes = NORMAL  

0012FD84   00000000  \\hTemplateFile = NULL  
```

　　`CreateFileA` 之后跟一下：  

```
005ABE94    55                   PUSH EBP  

005ABE95    8BEC                 MOV EBP,ESP  

005ABE97    83C4 EC              ADD ESP,-14  

005ABE9A    53                   PUSH EBX  

005ABE9B    56                   PUSH ESI  

005ABE9C    57                   PUSH EDI  

005ABE9D    33C0                 XOR EAX,EAX  

005ABE9F    8945 EC              MOV DWORD PTR SS:[EBP-14],EAX  

005ABEA2    33C0                 XOR EAX,EAX  

005ABEA4    55                   PUSH EBP  

005ABEA5    68 99C05A00          PUSH Animator.005AC099  

005ABEAA    64:FF30              PUSH DWORD PTR FS:[EAX]  

005ABEAD    64:8920              MOV DWORD PTR FS:[EAX],ESP  

005ABEB0    E8 4BEEE5FF         CALL Animator.0040AD00  

005ABEB5    DD1D E8845D00        FSTP QWORD PTR DS:[5D84E8]  ==当前时间值->5d84e8  

005ABEF5    8B4D EC               MOV ECX,DWORD PTR SS:[EBP-14]  

005ABEF8    B2 01                 MOV DL,1  

005ABEFA    A1 40AC4100           MOV EAX,DWORD PTR DS:[41AC40]  

005ABEFF    E8 243EE7FF          CALL Animator.0041FD28   ==读文件（%system32%\\winsys16.crc）  

005ABFF3    E8 08EDE5FF      CALL Animator.0040AD00  

005ABFF8    DC25 E8845D00   FSUB QWORD PTR DS:[5D84E8]  ==当前时间值减去%system32%\\winsys16.crc 的时间值  

005ABFFE    DD5D F0         FSTP QWORD PTR SS:[EBP-10]  

005AC01E    6A 00         PUSH 0  

005AC020    6A 18         PUSH 18  ==24  

005AC022    6A 00         PUSH 0  

005AC024    6A 3C         PUSH 3C  ==60  

005AC026    6A 00         PUSH 0  

005AC028    6A 3C         PUSH 3C  ==将时间值转换成天数  

005AC02A    D905 A8C05A00   FLD DWORD PTR DS:[5AC0A8]  

005AC030    DC4D F0         FMUL QWORD PTR SS:[EBP-10]  

005AC033    D835 ACC05A00   FDIV DWORD PTR DS:[5AC0AC]  

005AC039    E8 226BE5FF     CALL Animator.00402B60  

005AC03E    E8 5996E5FF     CALL Animator.0040569C  

005AC043    E8 5496E5FF     CALL Animator.0040569C  

005AC048    E8 4F96E5FF     CALL Animator.0040569C  

005AC04D    A3 E0845D00     MOV DWORD PTR DS:[5D84E0],EAX  ==eax 就是使用过的天数了  

005AC052    833D E0845D00 1>CMP DWORD PTR DS:[5D84E0],16  ==和 22 比较  

005AC059    7C 09          JL SHORT Animator.005AC064  

005AC05B    C605 E4845D00 0>MOV BYTE PTR DS:[5D84E4],1   ==大于 21 就设置过期标志了  

005AC062    EB 07          JMP SHORT Animator.005AC06B  
```

　　启动时过期限制的爆破点很简单，把 `5ac059` 的 `jl` 改成 `jmp` 就可以了。  

顺便把 NAG 去除掉，拿出利器 `dede`，输出 `dpr`：  

```
begin  

{  

005BC20C   55                   push    ebp  

005BC20D   8BEC                 mov     ebp, esp  

005BC20F   83C4F0               add     esp, -$10  

005BC212   B8FCBC5B00           mov     eax, $005BBCFC  

* Reference to: SysInit.@InitExe(Pointer);  

|  

005BC217   E804A9E4FF           call    00406B20  

* Reference to TApplication instance  

|  

005BC21C   A1E8BE5C00           mov     eax, dword ptr [$005CBEE8]  

005BC221   8B00                 mov     eax, [eax]  

* Reference to: Forms.TApplication.Initialize(TApplication);  

|  

005BC223   E82821EBFF           call    0046E350  

* Reference to TApplication instance  

|  

005BC228   A1E8BE5C00           mov     eax, dword ptr [$005CBEE8]  

005BC22D   8B00                 mov     eax, [eax]  

* Possible String Reference to: 'CoffeeCup GIF Animator'  

|  

005BC22F   BA7CC25B00           mov     edx, $005BC27C  

* Reference to: Forms.TApplication.SetTitle(TApplication;AnsiString);  

|  

005BC234   E8231DEBFF           call    0046DF5C  

* Reference to: SplashForm.Proc_005AC90C           ==把这个 nop 掉就可以去掉 NAG 了，顺便做一下优化，在上面的 5bc22f 改成 jmp 5BC248 就 OK 了  

|  

005BC239   E8CE06FFFF           call    005AC90C  

* Reference to pointer to GlobalVar_005D8508  

|  

005BC23E   A1D4BE5C00           mov     eax, dword ptr [$005CBED4]  

005BC243   803800               cmp     byte ptr [eax], $00  

005BC246   7524                jnz     005BC26C  

* Reference to TfrmGifMain instance  

|  

005BC248   8B0DCCBF5C00         mov     ecx, [$005CBFCC]  

* Reference to TApplication instance  

|  

005BC24E   A1E8BE5C00           mov     eax, dword ptr [$005CBEE8]  

005BC253   8B00                 mov     eax, [eax]  

* Reference to class TfrmGifMain  

|  

005BC255   8B15184A5B00         mov     edx, [$005B4A18]  

* Reference to: Forms.TApplication.CreateForm(TApplication;TComponentClass;void;void);  

|  

005BC25B   E80821EBFF           call    0046E368  

* Reference to TApplication instance  

|  

005BC260   A1E8BE5C00           mov     eax, dword ptr [$005CBEE8]  

005BC265   8B00                 mov     eax, [eax]  

* Reference to: Forms.TApplication.Run(TApplication);  

|  

005BC267   E87C21EBFF           call    0046E3E8  

* Reference to: System.@Halt0;  

|  

005BC26C   E89F81E4FF           call    00404410  

5bc22f eb 17 90 90 90　(物理地址：1bb62f)  
```

　　去掉 NAG 窗体之后上面的日期检查也省了（没调用检查代码），呵呵。  

但还有 2 个地方要改，一是入主界面时有个对话框，退出前也有个对话框。  

再用 `dede` 和 `od` 分析一下，很快发现关键。  

`FormActivate` 地址：`5b5468`  

```
005B5468    80B8 11040000 00    CMP BYTE PTR DS:[EAX+411],0  

005B546F    75 0C              JNZ SHORT Animator.005B547D  ==改成 jmp 就去掉启动时的对话框了（物理地址 1b486f）  

005B5471    C680 11040000 01    MOV BYTE PTR DS:[EAX+411],1  

005B5478    E8 7BE1FFFF         CALL Animator.005B35F8  

005B547D    C3                 RETN  
```

看看 `dede` 里面的说明，一目了然嘛：  

```
* Reference to : TfrmStartNotice.Proc_005B35F8()  

|                      ^^^^^^^  

005B5478   E87BE1FFFF           call    005B35F8  
```

退出的 NAG：  

```
005B54C0   B201                  mov     dl, $01  ==改成 mov dl,0 就不生成 NAG 啦  

* Reference to class TfrmBackSplash  ==很明显，对不对:)  

|  

005B54C2   A1A82D5B00           mov     eax, dword ptr [$005B2DA8]  

* Reference to: Forms.TCustomForm.Create(TCustomForm;boolean;TComponent);  ==有 dede，delphi 程序很好玩:)  

|  

005B54C7   E8C017EBFF           call    00466C8C  
```

　　这个 `FormClose` 整个函数就是生成一个 NAG，把入口处直接改成 `retn` 就成了:)  

```
005B5480    53                  PUSH EBX      ==改成 retn（物理地址 1b4880）  

005B5481    8BD8                MOV EBX,EAX  

005B5483    B2 01               MOV DL,1  

005B5485    8BC3                MOV EAX,EBX  

005B5487    E8 50310000         CALL Animator.005B85DC  

005B548C    A1 30BD5C00         MOV EAX,DWORD PTR DS:[5CBD30]  

005B5491    8338 00             CMP DWORD PTR DS:[EAX],0  

005B5494    74 15              JE SHORT Animator.005B54AB  

005B5496    A1 30BD5C00         MOV EAX,DWORD PTR DS:[5CBD30]  

005B549B    8B00                MOV EAX,DWORD PTR DS:[EAX]  

005B549D    E8 BE5AEBFF         CALL Animator.0046AF60  

005B54A2    A1 30BD5C00         MOV EAX,DWORD PTR DS:[5CBD30]  
```

　　收工，顺便做个补丁:)

`4dd466d0e1e0cc123afbe504da2462ca`

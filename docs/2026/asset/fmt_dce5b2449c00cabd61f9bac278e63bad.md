
# 高手请进！帮手指出这个算法里面的错误 (12千字)

> PauYyoung大哥你还记得这句话吗??

> ↓↓↓↓↓

> 【有信心，凡事有可能】

> 【请各位大哥帮小弟找一下这个注册算法有什么错误】

- 作者：随风而去[CNCG]
- 所用时间：一周
- 软件名：Dance Dance Revlution for winamp
- 软件介绍：此乃winamp的一个插件，可以让winamp实现跳舞毯的功能，没有注册前，不能使用跳舞毯，还有一个特殊效果也用不上。注册之后就没有这些限制了。

---

先运行winamp，招到此插件的配置窗口，然后点击注册，标准的注册窗口，用户名加注册码。

先随便试了一下，有错误提示，，用trw追。

下中断`bpx hmemcpy`，不要下`pmodule`，直接按F12，按了11下就回到程序了。重新来一次，按10下F12后。按一下F10，来到如下的程序段：

```
  \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:10003740(C)
  |
  \* Reference To: USER32.GetDlgItemTextA, Ord:0104h
              |
  :10003769 8B35D0B10010            mov esi,
dword ptr [1000B1D0]//F10来到这里
  :1000376F 53
    push ebx
  :10003770 8B5C2430
mov ebx, dword ptr [esp+30]
  :10003774 6A64
    push 00000064
  :10003776 6830F40010
push 1000F430
  :1000377B 68FB030000
push 000003FB
  :10003780 53
    push ebx
  :10003781 FFD6
    call esi  <－－取注册码长度放入EAX中
  :10003783 8D4C240C
lea ecx, dword ptr [esp+0C] <－－D EDX可看假码
  :10003787 6A20
    push 00000020
  :10003789 51
    push ecx
  :1000378A 68FA030000
push 000003FA
  :1000378F 53
    push ebx
  :10003790 FFD6
    call esi
  :10003792 8D54240C
lea edx, dword ptr [esp+0C]
  :10003796 52
    push edx
  :10003797 6830F40010
push 1000F430  <－－此处D 0287F430可看用户名
  :1000379C E8EFF7FFFF
call 10002F90 //这里有一个经典的对比，
        所以这个CALL嫌疑非常大，
        F8跟进到看看。
  :100037A1 83C408
  add esp, 00000008
  :100037A4 85C0
    test eax, eax
  :100037A6 0F84A3000000            je 1000384F
  :100037AC 8D442430
lea eax, dword ptr [esp+30]
  :100037B0 50
    push eax
  F8来到这里

  \* Referenced by a CALL at Addresses:
  |:100010D8  , :1000379C
  |
  :10002F90 83EC28
  sub esp, 00000028
  :10002F93 55
    push ebp
  :10002F94 8B6C2430
mov ebp, dword ptr [esp+30]
  :10002F98 85ED
    test ebp, ebp  <--看用户名是否为空
  :10002F9A 57
    push edi
  :10002F9B 0F8429010000            je 100030CA
  :10002FA1 8B542438
mov edx, dword ptr [esp+38]
  :10002FA5 85D2
    test edx, edx  <--看注册码是否为空
  :10002FA7 0F841D010000            je 100030CA
  :10002FAD 8BFD
    mov edi, ebp
  :10002FAF 83C9FF
  or ecx, FFFFFFFF
  :10002FB2 33C0
    xor eax, eax
  :10002FB4 F2
    repnz
  :10002FB5 AE
    scasb
  :10002FB6 F7D1
    not ecx
  :10002FB8 49
    dec ecx
  :10002FB9 0F840B010000            je 100030CA
  :10002FBF 8BFA
    mov edi, edx
  :10002FC1 83C9FF
  or ecx, FFFFFFFF
  :10002FC4 F2
    repnz
  :10002FC5 AE
    scasb
  :10002FC6 F7D1
    not ecx
  :10002FC8 49
    dec ecx
  :10002FC9 0F84FB000000            je 100030CA
  :10002FCF 53
    push ebx
  :10002FD0 32DB
    xor bl, bl
  :10002FD2 B908000000
mov ecx, 00000008
  :10002FD7 8D7C2411
lea edi, dword ptr [esp+11]
  :10002FDB 885C2410
mov byte ptr [esp+10], bl
  :10002FDF 33D2
    xor edx, edx
  :10002FE1 F3
    repz
  :10002FE2 AB
    stosd
  :10002FE3 8BFD
    mov edi, ebp
  :10002FE5 83C9FF
  or ecx, FFFFFFFF
  :10002FE8 F2
    repnz
  :10002FE9 AE
    scasb
  :10002FEA F7D1
    not ecx
  :10002FEC 49
    dec ecx
  :10002FED 56
    push esi
  :10002FEE 85C9
    test ecx, ecx
  :10002FF0 894C243C
mov dword ptr [esp+3C], ecx
  :10002FF4 7E0A
    jle 10003000

  \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:10002FFE(C)
  |
  :10002FF6 8A042A
  mov al, byte ptr [edx+ebp] --| |注册名用十六进制放入al
  :10002FF9 32D8
    xor bl, al
  |    |    和BL异或后，
  :10002FFB 42
    inc edx
      |-----|BL再和第二位异或……
  :10002FFC 3BD1
    cmp edx, ecx
|    |到最后一位，得出BL
  :10002FFE 7CF6
    jl 10002FF6 -----------------|    |

  \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:10002FF4(C)
  |
  :10003000 33F6
    xor esi, esi

  \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:10003073(U)
  |
  :10003002 83FE08
  cmp esi, 00000008 //注册码不小于8个<1>
  :10003005 7C07
    jl 1000300E//小于就跳
  :10003007 8D0409
  lea eax, dword ptr [ecx+ecx]//注册码是注册名\*2
  :1000300A 3BF0
    cmp esi, eax //比较
  :1000300C 7D67
    jge 10003075 //大于或等于跳，跳到注册码比较

  \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:10003005(C)
  |
  :1000300E 8BFD
    mov edi, ebp
  :10003010 83C9FF
  or ecx, FFFFFFFF
  :10003013 33C0
    xor eax, eax
  :10003015 F2
    repnz
  :10003016 AE
    scasb
  :10003017 8BC6
    mov eax, esi
  :10003019 99
    cdq
  :1000301A F7D1
    not ecx
  :1000301C 2BC2
    sub eax, edx
  :1000301E 49
    dec ecx
  :1000301F D1F8
    sar eax, 1
  :10003021 33D2
    xor edx, edx
  :10003023 F7F1
    div ecx
  :10003025 83FE01
  cmp esi, 00000001
  :10003028 8BFA
    mov edi, edx
  :1000302A 7E05
    jle 10003031
  :1000302C 8D46FF
  lea eax, dword ptr [esi-01]
  :1000302F EB02
    jmp 10003033

  \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:1000302A(C)
  |
  :10003031 33C0
    xor eax, eax

  \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:1000302F(U)
  |
  :10003033 8A4C0414            |---mov cl,
byte ptr [esp+eax+14]//正确的注册码从0位开始放入cl
  :10003037 32CB          <B>==|  xor
cl, bl <---------------- //与上面得出的bl异或
  :10003039 80F18D              |
xor cl, 8D <---------------- //与8d异或
  :1000303C 8AD9
|---mov bl, cl <---------------- //放入bl, 保留着下次再计算
  :1000303E 885C2410
mov byte ptr [esp+10], bl
  :10003042 8B542410
mov edx, dword ptr [esp+10]  //得出的结果放入<1>计算
  :10003046 52
    push edx
  :10003047 E8F4FEFFFF
call 10002F40 <------------- //-----------<1>
  :1000304C 8B4C2440
mov ecx, dword ptr [esp+40]
  :10003050 88443418
mov byte ptr [esp+esi+18], al //放入<1>计算的注册码
  :10003054 2BCF
    sub ecx, edi
  :10003056 8A5429FF            |---mov dl,
byte ptr [ecx+ebp-01]----|  |注册名的最后一位和注册
  :1000305A 32D0          <C>==|  xor
dl, al
  |===|码第二位异或，再与D8异
  :1000305C 80F2D8              |---xor
dl, D8-----------------------|  |或。得出dl放入<1>计算
  :1000305F 52
    push edx
  :10003060 E8DBFEFFFF
call 10002F40 <------------- //-------------<1>
  :10003065 8B4C2444
mov ecx, dword ptr [esp+44]
  :10003069 83C408
  add esp, 00000008
  :1000306C 88443415
mov byte ptr [esp+esi+15], al //放入<1>计算的注册码
  :10003070 83C602
  add esi, 00000002
  :10003073 EB8D
    jmp 10003002  <------------- //向上跳,跳到<1>

  \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:1000300C(C)
  |
  :10003075 8B742440
mov esi, dword ptr [esp+40] //这里是假码,下d esp+40
  :10003079 8D442414
lea eax, dword ptr [esp+14] //这里是真码,下d esp+14

  \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:1000309F(C)
```

跟进<1> \*

---

```
  \* Referenced by a CALL at Addresses:
  |:10003047  , :10003060
  |
  :10002F40 8B442404
mov eax, dword ptr [esp+04] //将结果放入EAX
  :10002F44 B93E000000
mov ecx, 0000003E <---------//ECX=3e
  :10002F49 25FF000000
and eax, 000000FF <---------//取EAX最后两位
  :10002F4E 99
    cdq
  :10002F4F F7F9
    idiv ecx <------------------//eax/ecx  al=商 dl=余
  :10002F51 B03D
    mov al, 3D <----------------//al=3d
  :10002F53 2AC2
    sub al, dl <----------------//al=al-dl
  :10002F55 3C09
    cmp al, 09 <----------------//比较
  :10002F57 88442404
mov byte ptr [esp+04], al  //al放入[esp+04]
  :10002F5B 761A
    jbe 10002F77 <--------------//不低于或等于
  :10002F5D 3C23
    cmp al, 23  <---------------//比较
  :10002F5F 8B442404
mov eax, dword ptr [esp+04] //[esp+04]放入eax
  :10002F63 7609
    jbe 10002F6E <--------------//不低于或等于
  :10002F65 25FF000000
and eax, 000000FF <---------//取EAX最后两位
  :10002F6A 83C03D
```

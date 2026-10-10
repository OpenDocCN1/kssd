

# 序列号方式

> 原文：[`www.pediy.com/kssd/tutorial/chap6-1-11.htm`](https://www.pediy.com/kssd/tutorial/chap6-1-11.htm)

目录＞＞第六章

**序列号方式 **　**【习题】**

**1、习题一** chap6-1-1-01 2K Serial 中

**2、习题二** chap6-1-1-02 2K code 稍难

**3、习题三** chap6-1-1-03 16K Serial ( 需要 cw3220.dll ) 中

**4、习题四 **chap6-1-1-04 5K Name/Serial 中

**5、习题五 **chap6-1-1-05 103K Name/Serial 中

**6、习题六 **chap6-1-1-06 91K code 中

**7、习题七 **chap6-1-1-07 26K Name/Group/Serial

**8、习题八 **chap6-1-1-08 5K Name/Serial 中

**9、习题九 **chap6-1-1-09 119K Name/Serial 中

**10、习题十 **chap6-1-1-10 34K Name/Group/Serial 最难

 

  1、习题一 答案        

| 破解 chap6-1-1-01  
  
\* Referenced by a CALL at Address:   
|:004010B0    
|   
:004010C9 56                      push esi   
:004010CA 57                      push edi   
:004010CB 51                      push ecx   
:004010CC 33F6                    xor esi, esi   
:004010CE 33FF                    xor edi, edi   
:004010D0 B908000000              mov ecx, 00000008          ---定义循环次数   
:004010D5 BE44304000              mov esi, 00403044          ---调出输入的密码   
:004010DA 803632                  xor byte ptr \[esi\], 32      ---把密码个字节与 32 取异或   
:004010DD 46                      inc esi   
:004010DE E2FA                    loop 004010DA              ---向上循环   
:004010E0 BE44304000              mov esi, 00403044            
:004010E5 B904000000              mov ecx, 00000004          ---8 变 4   
:004010EA 8A06                    mov al, byte ptr \[esi\]   
:004010EC 8A5E01                  mov bl, byte ptr \[esi+01\]   
:004010EF 32C3                    xor al, bl                ---把变化后的密码前后取异或   
:004010F1 88874C304000            mov byte ptr \[edi+0040304C\], al   
:004010F7 83C602                  add esi, 00000002            
:004010FA 47                      inc edi   
:004010FB E2ED                    loop 004010EA              ---向上循环   
:004010FD BE4C304000              mov esi, 0040304C          ---将生成的 4 个数变成 2 个   
:00401102 8A06                    mov al, byte ptr \[esi\]   
:00401104 8A5E01                  mov bl, byte ptr \[esi+01\]   
:00401107 32C3                    xor al, bl   
:00401109 8A5E02                  mov bl, byte ptr \[esi+02\]   
:0040110C 8A4E03                  mov cl, byte ptr \[esi+03\]   
:0040110F 32D9                    xor bl, cl   
:00401111 32C3                    xor al, bl                  ---再由 2 个变 1 个放入 AL   
:00401113 B908000000              mov ecx, 00000008   
:00401118 BE44304000              mov esi, 00403044   
:0040111D 3006                    xor byte ptr \[esi\], al  ---将生成的 1 个与原来的取异或   
:0040111F 46                      inc esi   
:00401120 E2FB                    loop 0040111D   
:00401122 B908000000              mov ecx, 00000008          ---从这往下开始比较   
:00401127 BE44304000              mov esi, 00403044          ---放入算出的结果   
  
\* Possible StringData Ref from Data Obj ->"q"   
                                  |   
:0040112C BF08304000              mov edi, 00403008          ---放入正确的结果   
:00401131 8A06                    mov al, byte ptr \[esi\]   
:00401133 3A07                    cmp al, byte ptr \[edi\]   
:00401135 751D                    jne 00401154                ---跳向出错   
:00401137 46                      inc esi   
:00401138 47                      inc edi   
:00401139 E2F6                    loop 00401131              ---向上循环   
:0040113B 6A40                    push 00000040   
  
模拟运算:   
如果输入 12345678   
机器码                31  32  33  34  35  36  37  38    
与 32 异或              03  00  01  06  07  04  05  0A  ----(1)   
8 变 4 为                  03      07      03      0F   
4 变 2 为                      04              0C   
2 变 1 为                              08                ----(2)   
(1)与 08 取异或          0B  08  09  0E  0F  0C  0D  02   
  
00403008 内正确的为    71  18  59  1B  79  42  45  4C   
  
根据正确反推注册码:  (关键是如何计算(2))   
                    由算法可知(2)是由机器码反复取异或得到，其实由它的正确的密码重复这                    一算法也可求的(2),实验得出。缺少证明。   
  
机器码                71  18  59  1B  79  42  45  4C   
8 变 4 为                  69      42      3B      09   
4 变 2 为                      2B              32   
2 变 1 为                              19                ----正确的密码的(2)值应为 19   
  
接着反推正确的注册码:   
机器码                  71  18  59  1B  79  42  45  4C   
与 19 取异或              68  01  40  02  60  5B  5C  55   
与 32 取异或              5A  33  72  30  52  69  6E  67   
查表得正确的注册码为:    Z  3  r  0  R  i  n  g  (Z3r0Ring)   
ZXEM 2000.3.20  |
| --- |

 2、习题二 答案        

| 破解 chap6-1-1-02  
  
\* Referenced by a CALL at Address:   
|:004011A9    
|   
:0040120B C8000000                enter 0000, 00   
:0040120F 53                      push ebx   
:00401210 52                      push edx   
:00401211 33C0                    xor eax, eax   
:00401213 B8A6204000              mov eax, 004020A6        ----把输入的密码放入 EAX   
:00401218 803800                  cmp byte ptr \[eax\], 00   
:0040121B 7460                    je 0040127D   
:0040121D 33DB                    xor ebx, ebx   
:0040121F 33D2                    xor edx, edx   
:00401221 8A18                    mov bl, byte ptr \[eax\]    ----开始处理密码 过程见下（1）   
:00401223 C1C308                  rol ebx, 08   
:00401226 03D3                    add edx, ebx              ----处理后的都放入 EDX   
:00401228 40                      inc eax   
:00401229 803800                  cmp byte ptr \[eax\], 00   
:0040122C 75F3                    jne 00401221   
:0040122E 52                      push edx   
:0040122F 6854204000              push 00402054   
:00401234 68BF204000              push 004020BF   
:00401239 E88F000000              Call 004012CD   
:0040123E BBBF204000              mov ebx, 004020BF        ----与正确的开始比较   
:00401243 803B38                  cmp byte ptr \[ebx\], 38   
:00401246 7535                    jne 0040127D   
:00401248 807B0144                cmp byte ptr \[ebx+01\], 44   
:0040124C 752F                    jne 0040127D   
:0040124E 807B0243                cmp byte ptr \[ebx+02\], 43   
:00401252 7529                    jne 0040127D   
:00401254 807B0341                cmp byte ptr \[ebx+03\], 41   
:00401258 7523                    jne 0040127D   
:0040125A 807B0446                cmp byte ptr \[ebx+04\], 46   
:0040125E 751D                    jne 0040127D   
:00401260 807B0533                cmp byte ptr \[ebx+05\], 33   
:00401264 7517                    jne 0040127D   
:00401266 807B0636                cmp byte ptr \[ebx+06\], 36   
:0040126A 7511                    jne 0040127D   
:0040126C 807B0738                cmp byte ptr \[ebx+07\], 38   
:00401270 750B                    jne 0040127D   
:00401272 B801000000              mov eax, 00000001   
:00401277 5A                      pop edx   
:00401278 5B                      pop ebx   
:00401279 C9                      leave   
:0040127A C20400                  ret 0004   
  
（1）过程分析：   
              如果输入 123456   
              过程：          00 00 31 00   
                              00 31 32 00   
                              31 32 33 00   
                              32 33 34 31   
                              33 34 35 32                          
                          +）  34 35 36 33   
                      结果：  CB 00 35 96   
  
            而正确的应为：8D CA F3 68   
  我们通过上面的模拟分析可以大致的了解运算过程，由于相加结果是如上的有很多，我们就可以大胆的假设，来推出一组数。   
  我就推出一组 6 位的：""$%=)   
ZXEM 2000.3.23  |
| --- |

 3、习题三 答案         

| 这题我们用函数 getdlgitemtexta 设断，它作用是得指定输入框输入字符串。也是一个常用的函数。   
在注册框中输入：12345678   
然后设断：bpx getdlgitemtexta   
点击 OK,你将中断在 SOFTICE，具体如下：   
\* Reference To: USER32.GetDlgItemTextA, Ord:0000h   
                                  |   
:0040115D E8E4030000              Call 00401546   
:00401162 8D4DF4                  lea ecx, dword ptr \[ebp-0C\]   
:00401165 51                      push ecx   
:00401166 E811FFFFFF              call 0040107C<----此 CALL 计算密码   
:0040116B 59                      pop ecx   
  
按 F8 进入 00401166 的 CALL   
:0040108D B9E7030000              mov ecx, 000003E7   
:00401092 81C2495F0E00            add edx, 000E5F49<----用 000E5F49 加 12345678（十六进制 BC614E）   
:00401098 81C1A93E0F00            add ecx, 000F3EA9<----用 000F3EA9 加上固定数字 999（十六进制 03e7)   
:0040109E 90                      nop   
:0040109F 90                      nop   
......................................   
:004010A7 90                      nop   
:004010A8 83C258                  add edx, 00000058<----加上 58（十六进制 ）  
:004010AB 83C1A9                  add ecx, -57<----减 57 （十六进制 ）  
:004010AE 3BD1                    cmp edx, ecx<----比较这两个数字   
:004010B0 7518                    jne 004010CA<----如不正确就跳到错误信息   
:004010B2 6800100000              push 00001000   
  
在 004010AE (cmp EDX, ECX)键入：   
? EDX <---- 13287663 (我们输入经过计算的密码）   
? ECX <---- 999993 (正确的数字)   
下面全部以十六进制表示计算：   
ECX + 999 - 87 = 999993   
EDX + 941897 + 88 = 13287663   
  
因此我们反推密码：   
999993 - 88 - 941897 = 58008  |
| --- |

 4、习题四 答案         

| 破解 chap6-1-1-04   
  
\* Reference To: USER32.DialogBoxParamA, Ord:0000h   
                                  |   
:0040121E E87D020000              Call 004014A0   
:00401223 83F800                  cmp eax, 00000000   
:00401226 74BE                    je 004011E6   
:00401228 688E214000              push 0040218E   
:0040122D E84C010000              call 0040137E      ----通过 NAME 算出一个数字   
:00401232 50                      push eax   
:00401233 687E214000              push 0040217E   
:00401238 E89B010000              call 004013D8      ----通过输入的 SERIAL 算出一个数字   
:0040123D 83C404                  add esp, 00000004   
:00401240 58                      pop eax   
:00401241 3BC3                    cmp eax, ebx        ----比较两个数字是否相同   
:00401243 7407                    je 0040124C   
:00401245 E818010000              call 00401362   
:0040124A EB9A                    jmp 004011E6   
  
\* Referenced by a CALL at Address:   
|:0040122D    
|   
:0040137E 8B742404                mov esi, dword ptr \[esp+04\]   
:00401382 56                      push esi   
:00401383 8A06                    mov al, byte ptr \[esi\]  ----ESI 中放的是输入的姓名   
:00401385 84C0                    test al, al   
:00401387 7413                    je 0040139C   
:00401389 3C41                    cmp al, 41   
:0040138B 721F                    jb 004013AC   
:0040138D 3C5A                    cmp al, 5A   
:0040138F 7303                    jnb 00401394   
:00401391 46                      inc esi   
:00401392 EBEF                    jmp 00401383   
:00401394 E839000000              call 004013D2        ----把输入的名字变成大写   
:00401399 46                      inc esi   
:0040139A EBE7                    jmp 00401383   
:0040139C 5E                      pop esi   
:0040139D E820000000              call 004013C2        ----变后的姓名算出值放入 EDI (1)   
:004013A2 81F778560000            xor edi, 00005678    ----再变化    (2)   
:004013A8 8BC7                    mov eax, edi   
:004013AA EB15                    jmp 004013C1        
:004013AC 5E                      pop esi   
:004013AD 6A30                    push 00000030   
:004013AF 6860214000              push 00402160   
:004013B4 6869214000              push 00402169   
:004013B9 FF7508                  push \[ebp+08\]   
:004013BC E879000000              Call 0040143A   
:004013C1 C3                      ret   
  
\* Referenced by a CALL at Address:   
|:00401238    
|   
:004013D8 33C0                    xor eax, eax   
:004013DA 33FF                    xor edi, edi   
:004013DC 33DB                    xor ebx, ebx   
:004013DE 8B742404                mov esi, dword ptr \[esp+04\] ----把输入的的密码放入 ESI   
:004013E2 B00A                    mov al, 0A   
:004013E4 8A1E                    mov bl, byte ptr \[esi\]        
:004013E6 84DB                    test bl, bl   
:004013E8 740B                    je 004013F5   
:004013EA 80EB30                  sub bl, 30            ----BL-30   
:004013ED 0FAFF8                  imul edi, eax        ----EDI\*EAX(此处 EAX=0A=10!!!!!)   
:004013F0 03FB                    add edi, ebx          ----EDI+EBX   
:004013F2 46                      inc esi   
:004013F3 EBED                    jmp 004013E2   
:004013F5 81F734120000            xor edi, 00001234            
:004013FB 8BDF                    mov ebx, edi                ----把处理后的放入 EBX   
:004013FD C3                      ret   
  
模拟运算:   
输入姓名:zxem   
输入密码:123456   
我们可以看到(1)处算出的 EDI=144, (2)处为 EDI=573C.   
我们知道密码的处理中为乘 10 再类加.所以从 EDI=573C 反推真密码,如下:   
XOR  573C, 1234 得到 4508,   
而 4508 换成 10 进制为 17672   
所以真的密码为:17672   
  
                                              ZXEM 2000.3.20  |
| --- |

 5、习题五 答案         

| bpx hmemcpy 设断来到：   
    :00427B7E E80DE2FEFF              call 00415D90   
    :00427B83 8B45FC                  mov eax, dword ptr \[ebp-04\]   
    :00427B86 50                      push eax   
    :00427B87 8D55F8                  lea edx, dword ptr \[ebp-08\]   
    :00427B8A 8B83DC010000            mov eax, dword ptr \[ebx+000001DC\]   
    :00427B90 E8FBE1FEFF              call 00415D90   
    :00427B95 8B45F8                  mov eax, dword ptr \[ebp-08\]   
    :00427B98 5A                      pop edx;;在这 D EDX 你将看到姓名、你输入号码、正确号码   
    :00427B99 E882FEFFFF              call 00427A20      ;; 比较序列号设置旗标   
    :00427B9E 3D4E61BC00              cmp eax, 00BC614E  ;; eax 与 BC614E 比较   
    :00427BA3 7D1E                    jge 00427BC3        ;; 如大于或等于则"Correct"   
你也可进入 :00427B99 的 CALL，来到：   
    :00427AF6 8B45F4                  mov eax, dword ptr \[ebp-0C\]  ;; 正确 serial   
    :00427AF9 8B55F8                  mov edx, dword ptr \[ebp-08\]  ;; 输入 serial   
    :00427AFC E8BFBDFDFF              call 004038C0                ;; 比较它们  |
| --- |

 6、习题六 答案          

| 用 W32Dasm 打开程序，利用串式参考（String Data References）分析，看到"Well done"，双击来到：   
  
\* Possible StringData Ref from Code Obj ->"Delphi" <---这可能是第一个序列号   
                                  |   
:00421DC1 BA201F4200              mov edx, 00421F20   
:00421DC6 E8A916FEFF              call 00403474<---比较我们第一个序列号，就是"Delphi"   
:00421DCB 0F8522010000            jne 00421EF3<---不相等跳走   
:00421DD1 8D55FC                  lea edx, dword ptr \[ebp-04\]   
:00421DD4 8B83B4010000            mov eax, dword ptr \[ebx+000001B4\]   
:00421DDA E879FAFEFF              call 00411858   
:00421DDF 8B45FC                  mov eax, dword ptr \[ebp-04\]   
:00421DE2 E84117FEFF              call 00403528   
:00421DE7 E89437FEFF              call 00405580   
:00421DEC 83F809                  cmp eax, 00000009<---比较第个序列号是否是 9 位数？   
:00421DEF 0F85FE000000            jne 00421EF3<---不相等则跳走   
:00421DF5 8D55FC                  lea edx, dword ptr \[ebp-04\]   
:00421DF8 8B83B4010000            mov eax, dword ptr \[ebx+000001B4\]   
:00421DFE E855FAFEFF              call 00411858   
:00421E03 8B45FC                  mov eax, dword ptr \[ebp-04\]   
:00421E06 803848                  cmp byte ptr \[eax\], 48<---比较第一字符是否是 048h   
:00421E09 0F85E4000000            jne 00421EF3   
:00421E0F 8D55FC                  lea edx, dword ptr \[ebp-04\]   
:00421E12 8B83B4010000            mov eax, dword ptr \[ebx+000001B4\]   
:00421E18 E83BFAFEFF              call 00411858   
:00421E1D 8B45FC                  mov eax, dword ptr \[ebp-04\]   
:00421E20 80780165                cmp byte ptr \[eax+01\], 65<---比较第二字符是否是 065h   
:00421E24 0F85C9000000            jne 00421EF3   
:00421E2A 8D55FC                  lea edx, dword ptr \[ebp-04\]   
:00421E2D 8B83B4010000            mov eax, dword ptr \[ebx+000001B4\]   
:00421E33 E820FAFEFF              call 00411858   
:00421E38 8B45FC                  mov eax, dword ptr \[ebp-04\]   
:00421E3B 8078026C                cmp byte ptr \[eax+02\], 6C<---比较第三字符是否是 06Ch   
:00421E3F 0F85AE000000            jne 00421EF3   
:00421E45 8D55FC                  lea edx, dword ptr \[ebp-04\]   
:00421E48 8B83B4010000            mov eax, dword ptr \[ebx+000001B4\]   
:00421E4E E805FAFEFF              call 00411858   
:00421E53 8B45FC                  mov eax, dword ptr \[ebp-04\]   
:00421E56 8078036C                cmp byte ptr \[eax+03\], 6C<---比较第四字符是否是 06Ch   
:00421E5A 0F8593000000            jne 00421EF3   
:00421E60 8D55FC                  lea edx, dword ptr \[ebp-04\]   
:00421E63 8B83B4010000            mov eax, dword ptr \[ebx+000001B4\]   
:00421E69 E8EAF9FEFF              call 00411858   
:00421E6E 8B45FC                  mov eax, dword ptr \[ebp-04\]   
:00421E71 80780466                cmp byte ptr \[eax+04\], 66<---比较第五字符是否是 066h   
:00421E75 757C                    jne 00421EF3   
:00421E77 8D55FC                  lea edx, dword ptr \[ebp-04\]   
:00421E7A 8B83B4010000            mov eax, dword ptr \[ebx+000001B4\]   
:00421E80 E8D3F9FEFF              call 00411858   
:00421E85 8B45FC                  mov eax, dword ptr \[ebp-04\]   
:00421E88 8078056F                cmp byte ptr \[eax+05\], 6F<---比较第六字符是否是 06Fh   
:00421E8C 7565                    jne 00421EF3   
:00421E8E 8D55FC                  lea edx, dword ptr \[ebp-04\]   
:00421E91 8B83B4010000            mov eax, dword ptr \[ebx+000001B4\]   
:00421E97 E8BCF9FEFF              call 00411858   
:00421E9C 8B45FC                  mov eax, dword ptr \[ebp-04\]   
:00421E9F 80780672                cmp byte ptr \[eax+06\], 72<---比较第七字符是否是 072h   
:00421EA3 754E                    jne 00421EF3   
:00421EA5 8D55FC                  lea edx, dword ptr \[ebp-04\]   
:00421EA8 8B83B4010000            mov eax, dword ptr \[ebx+000001B4\]   
:00421EAE E8A5F9FEFF              call 00411858   
:00421EB3 8B45FC                  mov eax, dword ptr \[ebp-04\]   
:00421EB6 80780767                cmp byte ptr \[eax+07\], 67<---比较第八字符是否是 067h   
:00421EBA 7537                    jne 00421EF3   
:00421EBC 8D55FC                  lea edx, dword ptr \[ebp-04\]   
:00421EBF 8B83B4010000            mov eax, dword ptr \[ebx+000001B4\]   
:00421EC5 E88EF9FEFF              call 00411858   
:00421ECA 8B45FC                  mov eax, dword ptr \[ebp-04\]   
:00421ECD 80780865                cmp byte ptr \[eax+08\], 65<---比较第九字符是否是 065h   
:00421ED1 7520                    jne 00421EF3   
将这些 ASCII 转换成为：   
Hellforge   
小结：第一序列号是 Delphi   
      第一序列号是 Hellforge  |
| --- |

 7、习题七 答案         

| 用 W32Dasm 打开程序，利用串式参考（String Data References）分析，看到"REGISTERED!"，双击来到：   
\* Referenced by a CALL at Address:   
|:004012BA      
|   
:00401520 83EC10                  sub esp, 00000010   
:00401523 8B0D70974000            mov ecx, dword ptr \[00409770\]   
:00401529 030DAC974000            add ecx, dword ptr \[004097AC\]   
:0040152F 53                      push ebx   
:00401530 56                      push esi   
:00401531 81F9FFFFFF7F            cmp ecx, 7FFFFFFF   
:00401537 57                      push edi   
:00401538 7606                    jbe 00401540   
:0040153A 81E9FFFFFF7F            sub ecx, 7FFFFFFF   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00401538(C)   
|   
:00401540 890D70974000            mov dword ptr \[00409770\], ecx   
:00401546 390D88974000            cmp dword ptr \[00409788\], ecx  ;注意这里   
:0040154C 7563                    jne 004015B1   
:0040154E 8D44240C                lea eax, dword ptr \[esp+0C\]   
  
\* Possible StringData Ref from Data Obj ->"REGISTERED!"   
                                  |   
:00401552 686C844000              push 0040846C   
:00401557 50                      push eax   
  
通过用 W32DASM 分析后，我们用 SOFTICE 来调试，输入：   
姓名：toye 公司：toye 序列号：12345678   
bpx hmemcpy   
来到：:00401546 下命令：？ ECX 看到：:00401546   
这就是序列号。  |
| --- |

 8、习题八 答案        

| 破解 chap6-1-1-08  
如果输入的姓名少于 6 个字，就无法被拦截。(我也是试出来的)   
  
:0040156B E828030000              Call 00401898   
:00401570 33C0                    xor eax, eax   
:00401572 33DB                    xor ebx, ebx   
:00401574 33C9                    xor ecx, ecx   
:00401576 B901000000              mov ecx, 00000001   
:0040157B 33D2                    xor edx, edx   
:0040157D 8B45E4                  mov eax, dword ptr \[ebp-1C\]    ----把姓名放入 EAX   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040158B(C)   
|   
:00401580 8A18                    mov bl, byte ptr \[eax\]        ----对姓名进行处理（1）   
:00401582 32D9                    xor bl, cl   
:00401584 8818                    mov byte ptr \[eax\], bl   
:00401586 41                      inc ecx   
:00401587 40                      inc eax   
:00401588 803800                  cmp byte ptr \[eax\], 00   
:0040158B 75F3                    jne 00401580   
:0040158D 33C0                    xor eax, eax   
:0040158F 33DB                    xor ebx, ebx   
:00401591 33C9                    xor ecx, ecx   
:00401593 B90A000000              mov ecx, 0000000A   
:00401598 33D2                    xor edx, edx   
:0040159A 8B45F0                  mov eax, dword ptr \[ebp-10\]  ----把输入的密码放入 EAX   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004015A8(C)   
|   
:0040159D 8A18                    mov bl, byte ptr \[eax\]        ----对密码进行处理（2）   
:0040159F 32D9                    xor bl, cl   
:004015A1 8818                    mov byte ptr \[eax\], bl   
:004015A3 41                      inc ecx   
:004015A4 40                      inc eax   
:004015A5 803800                  cmp byte ptr \[eax\], 00   
:004015A8 75F3                    jne 0040159D   
:004015AA 8B45E4                  mov eax, dword ptr \[ebp-1C\]  ----处理后的姓名放入 EAX   
:004015AD 8B55F0                  mov edx, dword ptr \[ebp-10\]  ----处理后的密码放入 EBX   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004015BF(C)   
|   
:004015B0 33C9                    xor ecx, ecx   
:004015B2 8A18                    mov bl, byte ptr \[eax\]   
:004015B4 8A0A                    mov cl, byte ptr \[edx\]   
:004015B6 3AD9                    cmp bl, cl                  ----进行比较   
:004015B8 7509                    jne 004015C3   
:004015BA 40                      inc eax   
:004015BB 42                      inc edx   
:004015BC 803800                  cmp byte ptr \[eax\], 00   
:004015BF 75EF                    jne 004015B0   
:004015C1 EB16                    jmp 004015D9   
  
模拟运行：   
姓名：zxemzx   
密码: 123456   
  
处理姓名:    
                7A  78  65  6D  7A  78   
          XOR  1  2  3  4  5  6   
                7B  7A  66  69  7F  7E   
处理密码:   
                31  32  33  34  35  36   
          XOR  A  B  C  D  E  F   
                3B  39  3F  39  3B  39   
由姓名反推正确的密码:   
                7B  7A  66  69  7F  7E   
          XOR  A  B  C  D  E  F   
                71  71  6A  64  71  71   
查表的:          q  q  j  d  q  q   
  
所以正确的密码为:qqjdqq   
ZXEM 2000.3.23  |
| --- |

 9、习题九 答案         

| 破解 chap6-1-1-09   
我并没看懂他是如何处理我输入的名字的。但也破了。   
我输入的名字是：ZXEMZX   
        密码是：123456   
  
:0042DCB2 A360F74200              mov dword ptr \[0042F760\], eax   
:0042DCB7 A158F74200              mov eax, dword ptr \[0042F758\]   
:0042DCBC 3B0560F74200            cmp eax, dword ptr \[0042F760\]    ----比较（1）   
:0042DCC2 7517                    jne 0042DCDB   
:0042DCC4 6A00                    push 00000000   
:0042DCC6 668B0D1CDD4200          mov cx, word ptr \[0042DD1C\]   
:0042DCCD B202                    mov dl, 02   
  
\* Possible StringData Ref from Code Obj ->"Good Serial, Thanks For trying "   
                                        ->"this Crackme   
  
我在（1）中看见了 0042F760 中放着 1E240，这个就是 123456 的 16 进制。   
而 EAX 中为 31D4B070，所以真确的密码为 836022384。   
                      
                                    ZXEM 2000.3.23  |
| --- |

 10、习题十 答案 （英文）        

| Immortal Descendants CrackMe 4.0   
        Tutorial version 1.0 (4 avril 99) by Lucifer48   
  
\===============   
1\. INTRODUCTION   
\===============   
  
We go into the code by a BPX HMEMCPY, few F12/F10 and we are here:   
  
XXXX:00402260  PUSH EBX                    ;handle of the window's crackme   
XXXX:00402261  PUSH 0040BE28                ;serial   
XXXX:00402266  PUSH 0040BDE0                ;group   
XXXX:0040226B  PUSH 0040BF30                ;name   
XXXX:00402270  CALL 00401CF0                ;Everything is there!   
XXXX:00402275  ADD  ESP,10                  ;\*pop pop pop pop\*   
XXXX:00402278  CMP  WORD PTR \[0040BE20\],28  ;If FFFF, our serial is a bad one   
XXXX:00402280  JNZ  004022A0   
  
Well, let's go into call 00401CF0...   
  
\============================   
2\. HOLD ON! JUST A MINUTE...   
\============================   
  
To bypass the check of the number of try. 3 chances for solving this crackme; it's not enougth.   
In the code:   
  
XXXX:004022A0  MOV  AX,\[0040A074\]            ;numbers of tests already done   
XXXX:004022A6  INC  AX   
XXXX:004022A8  MOV  \[0040A074\],AX            ;save   
XXXX:004022AE  CMP  AX,0003   
XXXX:004022B2  JNZ  004022E6                ;3 tests = Button "Validate" disabled   
  
We change INC AX into NOP NOP and it's allright.   
For those who wants to put in AX a value >3, TORN@ADO thought about this.   
A MessageBox appears:   
  
"You're using a window editing tool like THE CUSTOMISER   
You have had 3 chances for entering the right code!   
Do you think that's fair?"   
  
\==================   
3\. LET'S CMP \[1/5\]   
\==================   
  
Going into the call 00401CF0   
  
...   
XXXX:00401CF9  CALL 004019F0                ;small and unseful   
XXXX:00401CFE  ADD  ESP,04                  ;pop-pop   
XXXX:00401D01  PUSH EBX   
XXXX:00401D02  CALL 00401720                ;HERE!   
XXXX:00401D07  ADD  ESP,04   
XXXX:00401D0A  CMP  EAX,01   
XXXX:00401D0D  JZ  00401D1D                ;to go further we must have EAX=1   
XXXX:00401D0F  MOV  WORD PTR \[0040BE20\],FFFF ;bad serial!   
...   
XXXX:00401D1C  RET                          ;ret   
  
We must exit the CALL 00401720 with EAX=1:   
  
XXXX:00401722  MOV  ESI,\[ESP+0C\]            ;d ESI : our serial   
XXXX:00401726  PUSH  EDI   
XXXX:00401727  MOV  AL,\[ESI+01\]            ;read the 2nd character   
XXXX:0040172A  MOV  CL,\[ESI+02\]            ;read the 3rd character   
XXXX:0040172D  XOR  AL,49   
XXXX:0040172F  XOR  CL,44   
XXXX:00401732  MOVSX EAX,AL   
XXXX:00401735  MOV  DL,\[ESI+03\]            ;read the 4th character   
XXXX:00401738  MOV  BL,\[ESI\]                :read the 1st character   
XXXX:0040173A  XOR  DL,39   
XXXX:0040173D  XOR  BL,54   
XXXX:00401740  MOVSX ECX,CL   
XXXX:00401743  MOVSX EDI,DL   
XXXX:00401746  MOV  DL,\[ESI+04\]            ;read the 5th character   
XXXX;00401749  XOR  DL,39   
  
Then, 5 cmp, if everything goes right then EAX=1 otherwise EAX=0   
CMP BL,1D  ( 1D XOR 54 = 49 \[I\] )   
CMP EAX,0D ( 0D XOR 49 = 44 \[D\] )   
CMP ECX,07 ( 07 XOR 44 = 43 \[C\] )   
CMP EDI,0D ( 0D XOR 39 = 34 \[4\] )   
CMP EDX,09 ( 09 XOR 39 = 30 \[0\] )   
  
end of CALL 00401720, the beginning of the serial is IDC40.   
Continue in 00401D1D.   
  
\==================================   
4\. LET'S CMP \[2/5\] : XOR MY FRIEND   
\==================================   
  
XXXX:00401D2D  CALL  00401A20                ;100% unuseful   
XXXX:00401D32  ADD  ESP,10   
XXXX:00401D35  PUSH  EBX   
XXXX:00401D36  CALL  00401790   
XXXX:00401D3B  ADD  ESP,10   
XXXX:00401D3E  CMP  EAX,01   
XXXX:00401D41  JZ    00401D51                ;the same as above...   
XXXX:00401D43  MOV  WORD PTR \[0040BE20\];FFFF ;bad serial...   
...   
XXXX:00401D50  RET                            ;ret   
  
Our goal: exit the call 00401790 with EAX=01.   
  
XXXX:00401790  MOV  EAX,\[ESP+04\]            ;d EAX : our serial   
XXXX:00401794  PUSH  EBX   
XXXX:00401795  PUSH  ESI   
XXXX:00401796  MOV  CL,\[EAX+07\]              ;read the 8th character   
XXXX:00401799  XOR  CL,52   
XXXX:0040179C  MOV  DL,\[EAX+08\]              :read the 9th character   
XXXX:0040179F  XOR  DL,45   
XXXX:004017A2  MOV  BL,\[EAX+09\]              ;read the 10th character   
XXXX:004017A5  XOR  BL,53   
XXXX:004017A8  MOV  AL,\[EAX+06\]              :read the 7th character   
XXXX:004017AB  MOVSX ECX,CL   
XXXX:004017AE  MOVSX EDX,DL   
XXXX:004017B1  MOVSX ESI,BL   
XXXX:004017B4  XOR  AL,54   
  
Then (again), few cmp:   
CMP AL,11  ( 11 XOR 54 = 45 \[E\] )   
CMP ECX,06 ( 06 XOR 52 = 54 \[T\] )   
CMP EDX,07 ( 07 XOR 45 = 42 \[B\] )   
CMP ESI,1F ( 1F XOR 53 = 4C \[L\] )   
  
End of call 00401790, with EAX=1   
  
For the moment, our serial is like IDC40\*ETBL\*\*\*...   
"\*" is any character.   
Go on XXXX:00401D51.   
  
\=============================   
5\. LET'S CMP \[3/5\] : THE DATE   
\=============================   
  
XXXX:00401D51  PUSH EBX   
XXXX:00401D52  CALL 004017E0                  ;HERE   
XXXX:00401D57  ADD  ESP,04   
XXXX:00401D5A  CMP  EAX,01   
XXXX:00401D5D  JZ  00401D6D   
XXXX:00401D5F  MOV  DWORD PTR \[0040BE20\],FFFF   
...   
XXXX:00401D6C  RET                            :ret   
  
We must (again) exit the CALL 004017E0 with EAX=1   
  
...   
XXXX:00401851  PUSH  EAX                      ;d EAX : the 4 last characters of my serial.   
XXXX:00401852  CALL  00403480                ;DEC->HEX. Result in EAX.   
XXXX:00401857  ADD  ESP,04   
XXXX:0040185A  MOV  ESI,EAX                  ;save the result.   
XXXX:0040185C  CALL  004016A0                ;HERE! (GetLocalTime...)   
XXXX:00401861  SUB  EAX,ESI   
XXXX:00401863  POP  EDI   
XXXX:00401864  POP  ESI   
XXXX:00401865  CMP  EAX,01   
XXXX:00401868  SBB  EAX,EAX                  ;we must have CF=1   
XXXX:0040186A  ADD  ESP,04   
XXXX:0040186D  NEG  EAX                      ;NEG FFFFFFFF = 00000001   
XXXX:0040186F  RET   
  
In the call 004016A0:   
  
XXXX:004016A8  PUSH  EAX   
XXXX:004016A9  CALL  \[KERNEL32!GetLocalTime\]  ;TORN@DO like this call...   
XXXX:004016AF  XOR  EDX,EDX   
XXXX:004016B1  XOR  EAX,EAX   
XXXX:004016B3  MOV  DX,\[ESP+06\]              ;month   
XXXX:004016B8  XOR  ECX,ECX   
XXXX:004016BA  MOV  AX,\[ESP+0A\]              ;day   
XXXX:004016BF  IMUL  EDX,EAX   
XXXX:004016C2  MOV  CX,\[ESP+0E\]              ;minute   
XXXX:004016C7  MOV  EAX,\[ESP+0C\]            ;hour   
XXXX:004016CB  AND  EAX,0000FFFF   
XXXX:004016D0  IMUL  EAX,ECX   
XXXX:004016D3  MOVSX ECX,BYTE PTR \[0040BF60\]  ;1st letter of (\*)   
XXXX:004016DA  ADD  EDX,EAX   
XXXX:004016DC  MOV  EAX,\[ESP+04\]            ;year   
XXXX:004016E0  SUB  EDX,ECX   
XXXX:004016E2  AND  EAX,0000FFFF   
XXXX:004016E7  LEA  ESI,\[EAX+EDX\]   
XXXX:004016EA  PUSH  ESI                      ;save the result   
XXXX:004016EB  CALL  00401290                ;Soft Ice (for win95/98) installed ?   
XXXX:004016F0  ADD  ESP,04   
XXXX:004016F3  TEST  EAX,EAX   
XXXX:004016F5  JZ    004016FD   
XXXX:004016F7  ADD  ESI,00000143            ;if "si95" is installed ESI=ESI+143   
XXXX:004016FD  PUSH  ESI   
XXXX:004016FE  CALL  004012D0                ;Soft Ice (for winNT) installed ?   
XXXX:00401703  ADD  ESP,04   
XXXX:00401706  TEST  EAX,EAX   
XXXX:00401708  JZ    00401710   
XXXX:0040170A  ADD  ESI,00000271            ;if "siNT" is installed ESI=ESI+143   
XXXX:00401710  MOV  EAX,ESI   
XXXX:00401712  POP  ESI   
XXXX:00401713  ADD  ESP,10   
XXXX:00401716  RET                            ;ret   
  
(\*)   
\[HKEY\_LOCAL\_MACHINE\\Software\\Microsoft\\Windows\\CurrentVersion\]   
"RegisteredOwner"="LuCiFeR48"   
  
So, our 4 last characters of our serial is the encryption of the date and knowing if Soft-Ice is   
installed.   
There is a new serial every minute!   
  
Continue in XXXX:00401D6D   
  
\====================================================   
6\. LET'S CMP \[4/5\] : FirstInstallDateTime OF WINDOWS   
\====================================================   
  
XXXX:00401D71  CALL  00401B70                ;unuseful   
XXXX:00401D76  ADD  ESP,10   
XXXX:00401D79  MOV  EAX,\[0040C380\]   
XXXX:00401D7E  PUSH  EAX   
XXXX:00401D7F  PUSH  EBX   
XXXX:00401D80  CALL  00401870                ;HERE   
XXXX:00401D85  ADD  ESP,08   
XXXX:00401D88  CMP  EAX,01   
XXXX:00401D8B  JZ    00401D9B                ;must have EAX=1   
XXXX:00401D8D  MOV  WORD PTR \[0040BE20\],FFFF   
...   
XXXX:00401D9A  RET                            ;retour (bad cracker)   
  
Explorons le call 00401870:   
  
(\*\*) copy a part of the serial from 6th character to 12th character (included) so 7 numbers.   
...   
XXXX:004018BB  LEA  EAX,\[ESP+10\]            ;a part of our serial (\*\*)   
...                                          ;then DEC=>HEX   
XXXX:004018EE  CALL  00403559                ;give a hexa number   
XXXX:004018F3  SUB  EAX,ESI                ;we must have EAX=ESI to win!   
XXXX:004018F5  POP  EBP   
XXXX:004018F6  POP  EDI   
XXXX:004018F7  CMP  EAX,01   
XXXX:004018FA  SBB  EAX,EAX   
XXXX:004018FC  POP  ESI   
XXXX:004018FD  NEG  EAX   
XXXX:004018FF  POP  EBX   
XXXX:00401900  ADD  ESP,64   
XXXX:00401803  RET                          ;end of call 00401870   
  
A few words on the call 00403559   
...   
XXXX:004018CA  MOV  EAX,\[0040BEFC\]                  ;(#)   
XXXX:004018CF  MOV  \[ESP+10\],EAX   
XXXX:004018D3  MOV  DWORD PTR \[ESP+14\],00000000   
XXXX:004018DB  FILD  QWORD PTR \[ESP+10\]              ; What are those instructions?   
XXXX:004018E1  CALL  00403559                        ;   
XXXX:004018E6  FMUL  REAL8 PTR \[00409020\]            ; I don't understand those lines   
XXXX:004018EC  FSUBP ST(1),ST                        ; help me!   
XXXX:004018EE  CALL  0040264C                        ;   
XXXX:004018F3  SUB  EAX,ESI                        ;subtract the result with our hexa number   
...   
              RET   
  
(#) This number is from the 4 bytes read in   
\[HKEY\_LOCAL\_MACHINE\\Software\\Microsoft\\Windows\\CurrentVersion\]   
FirstInstallDateTime   
And a small modification...   
XXXX:00401E70  MOV  EBP,\[USER32!wsprintfA\]   
XXXX:00401E76  PUSH  EAX                      ;the 4 bytes of "FirstInstallDateTime"   
XXXX:00401E77  PUSH  0040A6C8   
XXXX:00401E7C  PUSH  0040BF90                ;area for the result   
XXXX:00401E81  CALL  EBP   
  
The result (in EAX) of the call 00403559, must be converted in hexa and must belong in the serial.   
  
Example, for me:  IDC40-ETBL22226617393-xxxx   
                  ----- ----    ------- ----   
LET'S CMP          \[1/5\] ----    ------- ----   
part:                    \[2/5\]  ------- ----   
                                  \[4/5\]  ----   
                                        \[3/5\]   
  
\-: any character   
xxxx: to replace by the result of the encryption of the data (changes every minute)   
      (put a BPX 00401865, and R EAX=0, et go on...)   
  
Next, in XXXX:00401D9B   
  
\=========================================   
7\. LET'S CMP \[5/5\] : Encrytion Name+Group   
\=========================================   
  
XXXX:00401D9F  CALL 00401C30                ;unuseful   
XXXX:00401DA4  ADD  ESP,10   
XXXX:00401DA7  PUSH EBX   
XXXX:00401DA8  CALL 00401910                ;HERE   
XXXX:00401DAD  ADD  ESP,04   
XXXX:00401DB0  CMP  EAX,01   
XXXX:00401DB3  JZ  00401DC3                ;the end of the crackme!   
XXXX:00401DB5  MOV  WORD PTR \[0040BE20\],FFFF   
...   
XXXX:00401DC2  RET   
  
  
In XXXX:00401DC3, it's the end of the crackme   
But before...let's visit the call 00401910.   
  
...   
XXXX:00401972  LEA  EDX,\[ESP+08\]            ;D EDX : a part of our serial (\*\*\*)   
XXXX:00401976  PUSH EAX   
XXXX:00401977  CALL 00403480                ;DEC=>HEX (result in EAX)   
XXXX:0040197C  ADD  ESP,04                  ;pop   
XXXX:0040197F  MOV  EDI,EAX                ;save the result in EDI   
XXXX:00401981  CALL 00401590                ;In EAX, a new hexa number (few +)   
XXXX:00401986  MOV  ESI,EAX   
XXXX:00401988  CALL 00401540                ;In EAX, a new hexa number ("RegisteredOwner")   
XXXX:0040198D  XOR  ESI,EAX   
XXXX:0040198F  PUSH 0040BDE0                ;group   
XXXX:00401994  PUSH 0040BF30                ;nom   
XXXX:00401999  CALL 004015E0   
XXXX:0040199E  ADD  ESP,08   
XXXX:004019A1  ADD  EAX,ESI   
then:   
EAX=EAX\*DB (EAX times 219)   
ECX=4\*ECX   
ECX=ECX-EAX   
ECX=64\*ECX   
ECX=ECX-EAX   
ECX=2\*ECX   
XXXX:004019B8  MOV \[ESP+08\],ECX   
  
(\*\*\*) The serial is (definitively) like:   
IDC40-ETBL-24654564564-6617393-xxxx   
          -----------   
LET'S CMP    \[5/5\]   
          In 00401972, d EDX : you see 24654564564.   
  
| About the call 00401540, it adds every characters of the "RegisteredOwner" (except the last)   
| Example for me:   
| ------IDCRKME40!.data+1F60-----------------------------------------------------   
| XXXX:0040BF60  4C 75 43 69 46 65 52 34 38 00 00 00 00 00 00 00  LuCiFeR48......   
|   
| 4C+75+43+69+46+65+52+34 = 29E = result of EAX   
  
| About the call 00401590, each number of character of the "RegisteredOwner" (except the last)   
| is multiplicated par 2 and make an add of all of this.   
| Example: 1\*2 + 2\*2 + 3\*2 + 4\*2 + 5\*2 + 6\*2 + 7\*2 + 8\*2 = (1+2+3+4+5+6+7+8)\*2 = 24\*2 = 48   
| With letters: if 'n' is the length of the string of the "RegisteredOwner"   
| We've got the formula: (n-1)\*n   
| Length of "LuCiFeR48" = 9 and so 9\*8=48   
| Remarque: everything is calculated in hexa   
  
Few words on the call 004015E0,   
Each character (except the last) is multiplicated by A29 and added.   
We add the ascii value of character of the group (except the last).   
The result is multiplicated by B (11)   
  
\[...\]   
  
...   
XXXX:004019C4  FILD  QWORD PTR \[ESP+08\]      ;   
XXXX:004019C8  FLD    ST(0)                    ;   
XXXX:004019CA  FSIN                            ; I don't understand this!   
XXXX:004019CC  FMUL  REAL8 PTR \[00409028\]    ;   
XXXX:004019D2  FSUBP  ST(1),ST                ;   
XXXX:004019D4  CALL  0040264C                ;   
XXXX:004019D9  SUB    EAX,EDI                  ;subtract the result with our hexa number   
...   
XXXX:004019DD  CMP    EAX,01                  ;EAX=00 = good serial   
XXXX:004019E0  SBB    EAX,EAX   
XXXX:004019E2  ADD    ESP,64   
XXXX:004019E5  NEG    EAX   
XXXX:004019E7  RET   
  
  
\========================   
8\. FINALY: MY OWN SERIAL   
\========================   
  
Name: Lucifer48   
Group: None   
Serial: IDC40-ETBL-605573733-6617393-xxxx   
  
xxxx: to replace by the encrytped date (see part 5. LET'S CMP \[3/5\] )   
  
/\* FirstInstallDateTime: E1 B1 39 25   
  RegisteredOwner: LuCiFeR48   
\*/                                                                                |
| --- |

  4dd466d0e1e0cc123afbe504da2462ca
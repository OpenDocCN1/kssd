

# 周公解梦 2.11

> 原文：[`www.pediy.com/kssd/pediy02/forum263.htm`](https://www.pediy.com/kssd/pediy02/forum263.htm)

 周公解梦 2.11 试用 18 天，作者也太吝了，呵呵，破了它吧!   
  
软件下载：   
http://202.102.231.158/software/download/system/other/zhou211.zip   
前言：和上次的那个有声有色 3.10 一样，此软件也是用 aspack 加壳，用   
毕卫国先生的 unaspack1.0.9.1 或手动脱壳都很容易!。   
步骤：   
一、安装后将时间向后调一个月，出现了过期提示框，这就是切入点，   
我们可以从这里入手，此时在 trw 中下   
:pmodule   
:g  (返回)   
按一下“确定”，程序就被拦截下来：   
015F:00459046 50              PUSH    EAX   
015F:00459047 E838DAFAFF      CALL    \`USER32!MessageBoxA\`   
015F:0045904C 8BF0            MOV      ESI,EAX  //程序停于此   
由此可见，注册提示框是由上面的 CALL USER32!MessageBoxA 出来的。   
  
二、用 W32dasm 反汇编脱壳后的主程序，找到程序 00459047 处，看下面代码：   
  
\* Referenced by a CALL at Addresses:   
|:004A3F38  , :004A4382  , :004A43F5  , :004A4426  , :004A4461    
|  //此处有五个地址值得怀疑!破解关键所在。   
:00458FFC 55                      push ebp   
:00458FFD 8BEC                    mov ebp, esp   
:00458FFF 6A00                    push 00000000   
:00459001 53                      push ebx   
:00459002 56                      push esi   
:00459003 8BD8                    mov ebx, eax   
:00459005 33C0                    xor eax, eax   
:00459007 55                      push ebp   
:00459008 6873904500              push 00459073   
:0045900D 64FF30                  push dword ptr fs:\[eax\]   
:00459010 648920                  mov dword ptr fs:\[eax\], esp   
:00459013 8D55FC                  lea edx, dword ptr \[ebp\-04\]   
:00459016 8B4338                  mov eax, dword ptr \[ebx+38\]   
:00459019 8B08                    mov ecx, dword ptr \[eax\]   
:0045901B FF511C                  call \[ecx+1C\]   
:0045901E 66837B4200              cmp word ptr \[ebx+42\], 0000   
:00459023 7408                    je 0045902D   
:00459025 8BD3                    mov edx, ebx   
:00459027 8B4344                  mov eax, dword ptr \[ebx+44\]   
:0045902A FF5340                  call \[ebx+40\]   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00459023(C)   
|   
:0045902D 8B4328                  mov eax, dword ptr \[ebx+28\]   
:00459030 50                      push eax   
:00459031 8B4330                  mov eax, dword ptr \[ebx+30\]   
:00459034 E8C7ADFAFF              call 00403E00   
:00459039 50                      push eax   
:0045903A 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
:0045903D E8BEADFAFF              call 00403E00   
:00459042 50                      push eax   
:00459043 8B432C                  mov eax, dword ptr \[ebx+2C\]   
:00459046 50                      push eax   
:00459047 E838DAFAFF              call 00406A84  //提示框的出处!向上看   
:0045904C 8BF0                    mov esi, eax   
:0045904E 66837B4A00              cmp word ptr \[ebx+4A\], 0000   
:00459053 7408                    je 0045905D   
  
三、可以下分别对我们怀疑的地址下中断   
:bpx 004A3F38    
:bpx 004A4382      
:bpx 004A43F5      
:bpx 004A4426    
:bpx 004A4461   
重新运行程序，程序被拦中断于 004A4382，呵呵，看来有戏，接着看：    
  
015F:004A436A 8B8328030000    MOV      EAX,\[EBX+0328\]   
015F:004A4370 8B10            MOV      EDX,\[EAX\]   
015F:004A4372 FF92B4000000    CALL    NEAR \[EDX+B4\]   
015F:004A4378 3C01            CMP      AL,01   
015F:004A437A 750D            JNZ      004A4389  //此处跳走，则不会出错:)   
015F:004A437C 8B830C030000    MOV      EAX,\[EBX+030C\]   
015F:004A4382 E8754CFBFF      CALL    00458FFC  //程序拦于此处!向上看 2 行   
015F:004A4387 EB71            JMP      SHORT 004A43FA   
015F:004A4389 8B8318030000    MOV      EAX,\[EBX+0318\]   
015F:004A438F E8F014FBFF      CALL    00455884   
015F:004A4394 DD1C24          FSTP    QWORD \[ESP\]   
015F:004A4397 9B              WAIT      
此程序只要在 004A437A 处，改为 JMP 004A4389 就不会出现过期提示了。如何   
修改程序，我就不想多说了。朱江(作者)兄，我可没改你的程序呀~~O~~   
  好久没写这么多东西了，真是累呀:)   
  
                十三少   
        China Cracking Group   
              2000.08.21   4dd466d0e1e0cc123afbe504da2462ca
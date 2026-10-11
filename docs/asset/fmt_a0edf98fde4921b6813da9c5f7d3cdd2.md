
:004195E8 8A18  mov bl, byte ptr [eax]  
:004195EA FF4F1C  dec [edi+1C]  
:004195ED 8D45F4  lea eax, dword ptr [ebp-0C]  
:004195F0 BA02000000  mov edx, 00000002  
:004195F5 E8B6300A00  call 004BC6B0  
:004195FA 0FBEC3  movsx eax, bl  
:004195FD 83F830  cmp eax, 00000030  ----很典型的一小段代码(功能同上)  
:00419600 7C05  jl 00419607  ----  
:00419602 83F839  cmp eax, 00000039  ----  
:00419605 7E32  jle 00419639  ----这里小跳

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00419600(C)  
|

:00419607 33C0  xor eax, eax  
:00419609 BA02000000  mov edx, 00000002  
:0041960E 50  push eax  
:0041960F 8D4508  lea eax, dword ptr [ebp+08]  
:00419612 FF4F1C  dec [edi+1C]  
:00419615 E896300A00  call 004BC6B0  
:0041961A FF4F1C  dec [edi+1C]  
:0041961D 8D450C  lea eax, dword ptr [ebp+0C]  
:00419620 BA02000000  mov edx, 00000002  
:00419625 E886300A00  call 004BC6B0  
:0041962A 58  pop eax  
:0041962B 8B17  mov edx, dword ptr [edi]  
:0041962D 64891500000000  mov dword ptr fs:[00000000], edx  
:00419634 E944010000  jmp 0041977D

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00419605(C)  
|

:00419639 66C747103800  mov [edi+10], 0038  <===小跳来到这里  
:0041963F 33C9  xor ecx, ecx  
:00419641 8D45F0  lea eax, dword ptr [ebp-10]  
:00419644 894DF0  mov dword ptr [ebp-10], ecx  
:00419647 50  push eax  
:00419648 FF471C  inc [edi+1C]  
:0041964B 8D450C  lea eax, dword ptr [ebp+0C]  
:0041964E B901000000  mov ecx, 00000001  
:00419653 8BD6  mov edx, esi  
:00419655 E8CE310A00  call 004BC828  
:0041965A 8D45F0  lea eax, dword ptr [ebp-10]  
:0041965D E8AA320A00  call 004BC90C  
:00419662 8B55C4  mov edx, dword ptr [ebp-3C]  
:00419665 8902  mov dword ptr [edx], eax  
:00419667 FF4F1C  dec [edi+1C]  
:0041966A 8D45F0  lea eax, dword ptr [ebp-10]  
:0041966D BA02000000  mov edx, 00000002  
:00419672 E839300A00  call 004BC6B0  
:00419677 46  inc esi  
:00419678 8345C404  add dword ptr [ebp-3C], 00000004  
:0041967C 83FE09  cmp esi, 00000009  
:0041967F 0F8E32FFFFFF  jle 004195B7  <===循环 9 次后，这里就不再跳转了!  
:00419685 33F6  xor esi, esi  <===ESI 被清 0 了。又要循环了。  
:00419687 8D95BCFEFFFF  lea edx, dword ptr [ebp+FFFFFEBC]  
:0041968D 8D8548FFFFFF  lea eax, dword ptr [ebp+FFFFFF48]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004196A8(C)  
|  <===这一行到 004196A5，又构成一个小循环

:00419693 8B1A  mov ebx, dword ptr [edx]  
   <===这里出现 11 次的数字是固定的(4，A，6，8，9，2，1，3，5，7，B )，估计是机器码的变形(后来发现没有用)  
:00419695 8B08  mov ecx, dword ptr [eax]  
   <===这里依次就是我们输入的注册码的了(7，8，7，8，7，8，7，8，7，8，7)  
:00419697 83C004  add eax, 00000004  
:0041969A 83C204  add edx, 00000004  
:0041969D 46  inc esi  
:0041969E 898C9D14FFFFFF  mov dword ptr [ebp+4*ebx-000000EC], ecx  
:004196A5 83FE0A  cmp esi, 0000000A  <===由于初始值是 0，所以这里也要循环 11 次，  
:004196A8 7EE9  jle 00419693  <===循环 11 次后，不再跳转了。  
:004196AA BE01000000  mov esi, 00000001  <===又初始化了，呵呵，又要循环了  
:004196AF 8D55A0  lea edx, dword ptr [ebp-60]  
:004196B2 8D8518FFFFFF  lea eax, dword ptr [ebp+FFFFFF18]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004196C6(C)  <===这一行到 004196C6，又构成一个小循环  
|

:004196B8 8B08  mov ecx, dword ptr [eax]

*******注：这里的 ECX 的取值如下 1、取输入的注册码第 7 位(7)  
                       2、取输入的注册码第 6 位(8)  
                       3、取输入的注册码第 8 位(8)  
                       4、取输入的注册码第 1 位(7)  
                       5、取输入的注册码第 9 位(7)  
                       6、取输入的注册码第 3 位(7)  
                       7、取输入的注册码第 10 位(8)  
                       8、取输入的注册码第 4 位(8)  
                       9、取输入的注册码第 5 位(7)  
           *****注：这里可以推断出输入的注册码第 2 位和第 11 位，可以为任意数字!  
             由于是定位取值，所在用假码 78787878787 将很定位，所以将假码改为 12345678901，重新来。  
             对应的取值变码是 768193045，记住这个对应关系后面用得到!

:004196BA 890A  mov dword ptr [edx], ecx  
   <===取得这些值依次放在 EDX 步进为 4 的位置上，ECX 的初始地址 8AF250 正好与最后比较的地址相同，故要仔细观察其变化  
:004196BC 46  inc esi  
:004196BD 83C204  add edx, 00000004  
:004196C0 83C004  add eax, 00000004  
:004196C3 83FE09  cmp esi, 00000009  <===初始值是 1，所以要循环 9 次。  
:004196C6 7EF0  jle 004196B8  <===循环 9 次后，不再跳转了。  
:004196C8 BB04000000  mov ebx, 00000004  
:004196CD BE01000000  mov esi, 00000001  <===又初始化了，呵呵，又要循环了  
:004196D2 3BDE  cmp ebx, esi  <===不相等  
:004196D4 7C2B  jl 00419701  <===不会跳

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004196FF(C)  
            <===这一行到 004196FF，又构成一个小循环，

:004196D6 8D8574FFFFFF  lea eax, dword ptr [ebp+FFFFFF74]  
:004196DC 50  push eax  
:004196DD 8D559C  lea edx, dword ptr [ebp-64]  
:004196E0 52  push edx  
:004196E1 E8FAFCFFFF  call 004193E0  
            <===第一个 CALL 会使我们所关心的值(以 8AF250 为起始地址的步进为 4 的位置)有变化，F8 进去看得究竟吧  
:004196E6 83C408  add esp, 00000008  
:004196E9 8D4D9C  lea ecx, dword ptr [ebp-64]  
:004196EC 51  push ecx  
:004196ED 8D8574FFFFFF  lea eax, dword ptr [ebp+FFFFFF74]  
:004196F3 50  push eax  
:004196F4 E8E7FCFFFF  call 004193E0  
            <===第二个同样的 CALL 使我们所关心的值有变化  
:004196F9 83C408  add esp, 00000008  
:004196FC 46  inc esi  <===esi 为计数器，每次加 1  
:004196FD 3BDE  cmp ebx, esi  <===EBX=4  
:004196FF 7DD5  jge 004196D6  
            <===这里循环 4 次，但由于有了两个同样的 CALL，所以实际上 768193045(假码的变形)已经被处理了 8 次，如何处理的呢？我们再研究一下 call 004193E0(功能在下面有说)，  
             *****  将 768193045 处理后为：524472195  
             将 524472195 处理后为：502225745  
             将 502225745 处理后为：055757895  
             将 055757895 处理后为：550523445  
             将 550523445 处理后为：969148595  
             将 969148595 处理后为：815477145  
             将 815477145 处理后为：083225295  
             将 083225295 处理后为：826757845  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004196D4(C)  
|

:00419701 BE01000000  mov esi, 00000001  <===又初始化了，呵呵，又要循环了  
:00419706 8D95F0FEFFFF  lea edx, dword ptr [ebp+FFFFFEF0]  
:0041970C 8D45A0  lea eax, dword ptr [ebp-60]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041974E(C)  
|  <===这一行到 0041974E，又构成一个小循环，将注册码处理成 826757845 后，与机器码依次比较，只要一次错了，就 OVER

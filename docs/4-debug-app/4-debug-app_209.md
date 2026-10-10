

# 过掉 佣兵天下 游戏的驱动保护，支持下看雪

> 原文：[`www.pediy.com/kssd/pediy12/126084.html`](https://www.pediy.com/kssd/pediy12/126084.html)

b1b465b4 8bff            mov     edi,edi  
b1b465b6 55              push    ebp  //这两行是内核的特征，凑足 5 个字  
b1b465b7 8bec            mov     ebp,esp  
b1b465b9 83ec10          sub     esp,10h  
b1b465bc 53              push    ebx  
b1b465bd 8b5d14          mov     ebx,dword ptr \[ebp+14h\] //  3 参  ObjectAttributes  
b1b465c0 56              push    esi  
b1b465c1 8b750c          mov     esi,dword ptr \[ebp+0Ch\] //  1 参  ProcessHandle  
b1b465c4 57              push    edi   
b1b465c5 33c0            xor     eax,eax  
b1b465c7 8d7df4          lea     edi,\[ebp-0Ch\]  
b1b465ca ab              stos    dword ptr es:\[edi\]  
b1b465cb ab              stos    dword ptr es:\[edi\]  
b1b465cc ab              stos    dword ptr es:\[edi\]  
b1b465cd 8b03            mov     eax,dword ptr \[ebx\]  //  3 参给 Eax  
b1b465cf 8365f400        and     dword ptr \[ebp-0Ch\],0  
b1b465d3 8945f8          mov     dword ptr \[ebp-8\],eax //  3 参给局部变量 ebp-8  
b1b465d6 c645f001        mov     byte ptr \[ebp-10h\],1  //给 ebp-10h 赋值为 1  
b1b465da 8975fc          mov     dword ptr \[ebp-4\],esi  //1 参给局部变量 ebp-4  
b1b465dd ff151410b5b1    call    dword ptr ds:\[0B1B51014h\] //调用了一个 CALL，估计是判断是否是自己的 PID  
b1b465e3 8bf8            mov     edi,eax  
b1b465e5 8d45f0          lea     eax,\[ebp-10h\]  //取出地址  
b1b465e8 50              push    eax    //这个地址里的值是 1，压入 1  
b1b465e9 b9503ab5b1      mov     ecx,0B1B53A50h  //这里是个全局变量  
b1b465ee e8873a0000      call    b1b4a07a  
b1b465f3 57              push    edi  
b1b465f4 e885dcffff      call    b1b4427e  
b1b465f9 85c0            test    eax,eax //判断 EAX 是否为空  
b1b465fb 7406            je      b1b46603 //如果 EAX 为空就跳  
  
b1b465fd c645f001        mov     byte ptr \[ebp-10h\],1//如果 EAX 不为空，再次给\[ebp-10h\]赋值为 1  
b1b46601 eb03            jmp     b1b46606  
  
b1b46603 8b75fc          mov     esi,dword ptr \[ebp-4\]  //1 参赋值给 ESI  
  
b1b46606 8b4d08          mov     ecx,dword ptr \[ebp+8\]  //EBP+8 是 CALL 的返回地址  
b1b46609 832100          and     dword ptr \[ecx\],0  
b1b4660c 807df000        cmp     byte ptr \[ebp-10h\],0 //比较下是否为 0  
b1b46610 b8220000c0      mov     eax,0C0000022h  
b1b46615 8bf8            mov     edi,eax  
b1b46617 7413            je      b1b4662c  //突破口就在这里，搞掉这句，让它执行原始的 NTopenProcess，这个保护也就过了  
b1b46619 837df400        cmp     dword ptr \[ebp-0Ch\],0  比较执行的那个 CALL 的返回值，如果为 0 就跳  
b1b4661d 740b            je      b1b4662a  
  
b1b4661f 53              push    ebx  //如果  \[ebp-0Ch\]不为 0 就执行下面的  
b1b46620 ff7510          push    dword ptr \[ebp+10h\] //2 参  
b1b46623 56              push    esi  
b1b46624 51              push    ecx  
b1b46625 ff55f4          call    dword ptr \[ebp-0Ch\]   //这里执行的原始的 NTopenprocess  
b1b46628 8bf8            mov     edi,eax  
  
b1b4662a 8bc7            mov     eax,edi  
  
b1b4662c 5f              pop     edi  
b1b4662d 5e              pop     esi  
b1b4662e 5b              pop     ebx  
b1b4662f c9              leave  
b1b46630 c21000          ret     10h  
  
  
警告，此贴，只为学习交流，切勿用于商业目的
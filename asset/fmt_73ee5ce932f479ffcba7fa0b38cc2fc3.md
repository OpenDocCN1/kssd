
### 第 18 处：`add esp,0c`

```
0046EE26     FF15 C4124000        call dword ptr ds:[<&msvbvm60.__vbaFreeStrList>]     
0046EE2C     0F33                rdpmc  
0046EE2E     D28D 4D8CFF15        ror byte ptr ss:[ebp+15FF8C4D],cl  
0046EE34     2C 10               sub al,10  
0046EE36     40                  inc eax  
0046EE37     00C7                add bh,al  
0046EE39     45                  inc ebp  
0046EE3A     FC                  cld  
0046EE3B     04 00               add al,0  
0046EE3D     0000                add byte ptr ds:[eax],al  
0046EE3F     837D D0 00          cmp dword ptr ss:[ebp-30],0  
0046EE43     75 1A               jnz short dumped-x.0046EE5F  
0046EE04     FF15 AC104000        call dword ptr ds:[<&msvbvm60.__vbaHresultCheckObj>] 
0046EE0A     8985 F4FEFFFF        mov dword ptr ss:[ebp-10C],eax  
0046EE10     EB 0A               jmp short dumped-x.0046EE1C  
0046EE12     C785 F4FEFFFF 000000>mov dword ptr ss:[ebp-10C],0  
0046EE1C     8D45 A4             lea eax,dword ptr ss:[ebp-5C]  
0046EE1F     50                  push eax  
0046EE20     8D4D A8             lea ecx,dword ptr ss:[ebp-58]  
0046EE23     51                  push ecx  
0046EE24     6A 02               push 2  
0046EE26     FF15 C4124000        call dword ptr ds:[<&msvbvm60.__vbaFreeStrList>]   
0046EE2C     83C4 0C             add esp,0C            // 上面 3 个 push  
0046EE2F     8D4D 8C             lea ecx,dword ptr ss:[ebp-74]  
0046EE32     FF15 2C104000        call dword ptr ds:[<&msvbvm60.__vbaFreeVar>]    
0046EE38     C745 FC 04000000     mov dword ptr ss:[ebp-4],4  
0046EE3F     837D D0 00          cmp dword ptr ss:[ebp-30],0  
0046EE43     75 1A               jnz short dumped-x.0046EE5F  
```

这里根据堆栈平衡原理，`0046EE26` 处的 `call` 上面有 3 个 `push`，因此 `add esp,0c`。  

### 第 19 处：`mov dword ptr ss:[ebp-14C],eax`

```
0046F3D9     FF15 AC104000        call dword ptr ds:[<&msvbvm60.__vbaHresultCheckObj>]   
0046F3DF     82E7 B7             and bh,FFFFFFB7  
0046F3E2     48                  dec eax  
0046F3E3     AF                  scas dword ptr es:[edi]  
0046F3E4     0BEB                or ebp,ebx  
0046F3E6     0AC7                or al,bh  
0046F3E8     85B4FE FFFF0000     test dword ptr ds:[esi+edi*8+FFFF],esi  
0046F3EF     0000                add byte ptr ds:[eax],al  
0046F3F1     8B55 9C             mov edx,dword ptr ss:[ebp-64]  
0046F3F4     8995 30FFFFFF        mov dword ptr ss:[ebp-D0],edx  
0046F3FA     8D45 8C             lea eax,dword ptr ss:[ebp-74]  
0046F3D9     FF15 AC104000        call dword ptr ds:[<&msvbvm60.__vbaHresultCheckObj>]    
0046F3DF     8985 B4FEFFFF        mov dword ptr ss:[ebp-14C],eax  
0046F3E5     EB 0A               jmp short dumped-x.0046F3F1  
0046F3E7     C785 B4FEFFFF 000000>mov dword ptr ss:[ebp-14C],0  
0046F3F1     8B55 9C             mov edx,dword ptr ss:[ebp-64]  
0046F3F4     8995 30FFFFFF        mov dword ptr ss:[ebp-D0],edx  
0046F3FA     8D45 8C             lea eax,dword ptr ss:[ebp-74]  
```

这个是参考程序其他地方的值，都是类似的：  

```
call dword ptr ds:[<&msvbvm60.__vbaHresultCheckObj>]   
mov dword ptr ss:[ebp-xxx],eax  
jmp short xxxxxxxx  
mov dword ptr ss:[ebp-xxx],0      // 根据这里的地址推测的  
```

### 第 20 处：`mov dword ptr ss:[ebp-154],eax`

```
0046F51E     FF15 AC104000        call dword ptr ds:[<&msvbvm60.__vbaHresultCheckObj>]   
0046F524     4F                  dec edi  
0046F525     AB                  stos dword ptr es:[edi]  
0046F526     C5D0                lds edx,eax                
0046F528     8EE9                mov gs,cx  
0046F52A     EB 0A               jmp short dumped-x.0046F536  
0046F52C     C785 ACFEFFFF 000000>mov dword ptr ss:[ebp-154],0  
0046F51E     FF15 AC104000        call dword ptr ds:[<&msvbvm60.__vbaHresultCheckObj>]    
0046F524     8985 ACFEFFFF        mov dword ptr ss:[ebp-154],eax  
0046F52A     EB 0A               jmp short dumped-x.0046F536  
0046F52C     C785 ACFEFFFF 000000>mov dword ptr ss:[ebp-154],0  
```

### 第 21 处：`add esp,0C`

```
0046F556     FF15 4C104000        call dword ptr ds:[<&msvbvm60.__vbaFreeVarList>]    
0046F55C   ^ E3 E8               jecxz short dumped-x.0046F546  
0046F55E     C3                  retn  
0046F55F     C745 FC 0E000000     mov dword ptr ss:[ebp-4],0E  
0046F566     8B55 D4             mov edx,dword ptr ss:[ebp-2C]  
0046F569     83C2 01             add edx,1  
0046F56C     0F80 78150000        jo dumped-x.00470AEA  
0046F556     FF15 4C104000        call dword ptr ds:[<&msvbvm60.__vbaFreeVarList>] 
0046F55C     83C4 0C              add esp,0C                 // 上面 3 个 push  
0046F55F     C745 FC 0E000000     mov dword ptr ss:[ebp-4],0E  
0046F566     8B55 D4             mov edx,dword ptr ss:[ebp-2C]  
0046F569     83C2 01             add edx,1  
0046F56C     0F80 78150000        jo dumped-x.00470AEA  
```

### 第 22 处：`mov ecx,eax`

```
0046FD0B     FF15 04134000        call dword ptr ds:[<&msvbvm60.__vbaVarAdd>]         
0046FD11     B8 82B81000          mov eax,10B882  
0046FD16     0000                add byte ptr ds:[eax],al  
0046FD18     E8 6335F9FF          call <jmp.&msvbvm60.__vbaChkstk>  
0046FD1D     8BD4                mov edx,esp  
0046FD1F     8B01                mov eax,dword ptr ds:[ecx]  
0046FD21     8902                mov dword ptr ds:[edx],eax  
0046FD23     8B41 04             mov eax,dword ptr ds:[ecx+4]  
0046FD26     8942 04             mov dword ptr ds:[edx+4],eax  
0046FD29     8B41 08             mov eax,dword ptr ds:[ecx+8]  
0046FD0B     FF15 04134000        call dword ptr ds:[<&msvbvm60.__vbaVarAdd>]         
0046FD11     8BC8                mov ecx,eax     
0046FD13     B8 10000000          mov eax,10  
0046FD18     E8 6335F9FF          call <jmp.&msvbvm60.__vbaChkstk>  
```

这个参考程序另一处：

```
0046F487     FF15 04134000        call dword ptr ds:[<&msvbvm60.__vbaVarAdd>]    
0046F48D     8BC8                mov ecx,eax  
0046F48F     B8 10000000          mov eax,10  
0046F494     E8 E73DF9FF          call <jmp.&msvbvm60.__vbaChkstk>  
```

### 第 23 处：`mov dword ptr ss:[ebp-198],eax`

```
0046FDC4     FF15 68104000        call dword ptr ds:[<&msvbvm60.__vbaFreeObjList>]   
0046FDCA     43                  inc ebx  
0046FDCB     36:B1 8D            mov cl,8D  
0046FDCE     857CFF FF           test dword ptr ds:[edi+edi*8-1],edi  
0046FDD2     FF50 8D             call dword ptr ds:[eax-73]  
0046FDD5     4D                  dec ebp  
0046FDD6     8C51 6A             mov word ptr ds:[ecx+6A],ss  
0046FDD9     02FF                add bh,bh  
0046FDDB     15 4C104000          adc eax,<&msvbvm60.__vbaFreeVarList>  
0046FDE0     83C4 0C              add esp,0C  
0046FDE3     C745 FC 1A000000     mov dword ptr ss:[ebp-4],1A  
0046FDEA     837D D0 00          cmp dword ptr ss:[ebp-30],0  
0046FDEE     75 1A               jnz short dumped-x.0046FE0A  
0046FDC4     FF15 68104000        call dword ptr ds:[<&msvbvm60.__vbaFreeObjList>]    
0046FDCA     83C4 0C              add esp,0C          // 上面 3 个 push  
0046FDCD     8D85 7CFFFFFF        lea eax,dword ptr ss:[ebp-84]  
0046FDD3     50                  push eax  
0046FDD4     8D4D 8C             lea ecx,dword ptr ss:[ebp-74]  
0046FDD7     51                  push ecx  
0046FDD8     6A 02               push 2  
0046FDDA     FF15 4C104000        call dword ptr ds:[<&msvbvm60.__vbaFreeVarList>]    
0046FDE0     83C4 0C              add esp,0C  
0046FDE3     C745 FC 1A000000     mov dword ptr ss:[ebp-4],1A  
```


```
00013799: 7446                 jz 000137E1  
0001379B: E810010000           call 000138B0  
000137A0: 6689857DB44000       mov word ptr [ebp+0040B47D],ax  
000137A7: 50                   push eax  
000137A8: 663D0100             cmp ax,0001  
000137AC: 7507                 jnz 000137B5  
000137AE: BBF5B84000           mov ebx,0040B8F5  
000137B3: EB05                 jmp 000137BA  
000137B5: BB59B94000           mov ebx,0040B959  
000137BA: 03DD                 add ebx,ebp  
000137BC: 66813B7C7C           cmp word ptr [ebx],7C7C  
000137C1: 7413                 jz 000137D6  
000137C3: B8C0B94000           mov eax,0040B9C0  
000137C8: 03C5                 add eax,ebp  
000137CA: 6A30                 push 00000030  
000137CC: 50                   push eax  
000137CD: 53                   push ebx  
000137CE: 6A00                 push 00000000  
000137D0: FF9500BC4000         call dword ptr [ebp+0040BC00]   <--MessageBoxA  
000137D6: 58                   pop eax  
000137D7: 663D0100             cmp ax,0001  
000137DB: 0F84E3FEFFFF         jz 000136C4  
000137E1: C6855DBD400000       mov byte ptr [ebp+0040BD5D],00  
000137E8: 8BC5                 mov eax,ebp  
000137EA: 5B                   pop ebx  
000137EB: 59                   pop ecx  
000137EC: 5A                   pop edx  
000137ED: 5E                   pop esi  
000137EE: 5F                   pop edi  
000137EF: 5D                   pop ebp  
000137F0: 9D                   popfd  
000137F1: E991110000           jmp 00014987  

-------子程序开始-------  

******获取系统时间，不知道要干什么，不会是比较时间差来判断是否被 SOFTICE 跟踪吧******  

000137F6: BEE1B94000           mov esi,0040B9E1  
000137FB: 03F5                 add esi,ebp  
000137FD: 56                   push esi  
000137FE: FF959FBB4000         call dword ptr [ebp+0040BB9F]   <---GetSystemTime  
00013804: 668B8585B44000       mov ax,word ptr [ebp+0040B485]  
0001380B: 668B9D81B44000       mov bx,word ptr [ebp+0040B481]  
00013812: 663D0000             cmp ax,0000  
00013816: 7430                 jz 00013848  
00013818: 663985E3B94000       cmp word ptr [ebp+0040B9E3],ax  
0001381F: 7209                 jb 0001382A  
00013821: 66399DE1B94000       cmp word ptr [ebp+0040B9E1],bx  
00013828: 7309                 jnb 00013833  
0001382A: 66399DE1B94000       cmp word ptr [ebp+0040B9E1],bx  
00013831: 7606                 jbe 00013839  
00013833: 66B80000             mov ax,0000  
00013837: EB0D                 jmp 00013846  
00013839: 66C7857DB440000100   mov word ptr [ebp+0040B47D],0001  
00013842: 66B80100             mov ax,0001  
00013846: EB04                 jmp 0001384C  
00013848: 66B80100             mov ax,0001  
0001384C: C3                   ret  

--------子程序结束----------  

---------子程序开始-------------  

0001384D: 60                   pushad  
0001384E: 6683BD89B4400000     cmp word ptr [ebp+0040B489],0000  
00013856: 750B                 jnz 00013863  
00013858: 66C785FBAB40000100   mov word ptr [ebp+0040ABFB],0001  
00013861: EB42                 jmp 000138A5  
00013863: E8CD030000           call 00013C35   <--从注册表中获取信息  
00013868: BE60B14000           mov esi,0040B160  
0001386D: 03F5                 add esi,ebp  
0001386F: 663D0100             cmp ax,0001  
00013873: 7405                 jz 0001387A  
00013875: 66C7060000           mov word ptr [esi],0000  
0001387A: 66833E00             cmp word ptr [esi],0000  
0001387E: 7417                 jz 00013897  
00013880: 66FF0E               dec word ptr [esi]  
00013883: 66C785FBAB40000100   mov word ptr [ebp+0040ABFB],0001  
0001388C: 66C7857DB440000100   mov word ptr [ebp+0040B47D],0001  
00013895: EB09                 jmp 000138A0  
00013897: 66C785FBAB40000000   mov word ptr [ebp+0040ABFB],0000  
000138A0: E84C040000           call 00013CF1  <--获取磁盘和卷的信息  
000138A5: 61                   popad  
000138A6: 668B85FBAB4000       mov ax,word ptr [ebp+0040ABFB]  
000138AD: C3                   ret  
000138AE: 0000                 add byte ptr [eax],al  
000138B0: E8BB000000           call 00013970  
000138B5: 8B85A8AD4000         mov eax,dword ptr [ebp+0040ADA8]  
000138BB: BBB0AD4000           mov ebx,0040ADB0  
000138C0: 03DD                 add ebx,ebp  
000138C2: B983B14000           mov ecx,0040B183  
000138C7: 03CD                 add ecx,ebp  
000138C9: 50                   push eax  
000138CA: 53                   push ebx  
000138CB: 51                   push ecx  
000138CC: FF9504BC4000         call dword ptr [ebp+0040BC04]  
000138D2: 83C40C               add esp,0000000C  
000138D5: B865B14000           mov eax,0040B165  
000138DA: 03C5                 add eax,ebp  
000138DC: BB79B14000           mov ebx,0040B179  
000138E1: 03DD                 add ebx,ebp  
000138E3: B983B14000           mov ecx,0040B183  
000138E8: 03CD                 add ecx,ebp  
000138EA: 51                   push ecx  
000138EB: 53                   push ebx  
000138EC: 50                   push eax  
000138ED: FF95BCBC4000         call dword ptr [ebp+0040BCBC]  
000138F3: E8F9030000           call 00013CF1  
000138F8: E873000000           call 00013970  
000138FD: C3                   ret  
------子程序------  
000138FE: 6683BD49B740007C     cmp word ptr [ebp+0040B749],007C  
00013906: 7505                 jnz 0001390D  
00013908: 66B80100             mov ax,0001  
0001390C: C3                   ret  
0001390D: B89FAC4000           mov eax,0040AC9F  
00013912: 03C5                 add eax,ebp  
00013914: 50                   push eax  
00013915: FF95B8BC4000         call dword ptr [ebp+0040BCB8]  
0001391B: BE49B74000           mov esi,0040B749  
00013920: 03F5                 add esi,ebp  
00013922: BF9FAC4000           mov edi,0040AC9F  
00013927: 03FD                 add edi,ebp  
00013929: 668B8567B44000       mov ax,word ptr [ebp+0040B467]  
00013930: 2020                 and byte ptr [eax],ah  
00013932: 2020                 and byte ptr [eax],ah  
00013934: 2038                 and byte ptr [eax],bh  
00013936: 3530343400           xor eax,00343430  
0001393B: 0000                 add byte ptr [eax],al  
0001393D: 0000                 add byte ptr [eax],al  
0001393F: 0000                 add byte ptr [eax],al  
00013920: 03F5                 add esi,ebp  
00013922: BF9FAC4000           mov edi,0040AC9F  
00013927: 03FD                 add edi,ebp  
00013929: 668B8567B44000       mov ax,word ptr [ebp+0040B467]  
00013930: 8A26                 mov ah,byte ptr [esi]  
00013932: 3A27                 cmp ah,byte ptr [edi]  
00013934: 7509                 jnz 0001393F  
00013936: 80FC00               cmp ah,00  
00013939: 7409                 jz 00013944  
0001393B: 46                   inc esi  
0001393C: 47                   inc edi  
0001393D: EBF1                 jmp 00013930  
0001393F: 66B80000             mov ax,0000  
00013943: C3                   ret  
```

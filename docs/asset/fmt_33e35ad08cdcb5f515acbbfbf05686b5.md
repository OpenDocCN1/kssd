
```
0043A78C   |.  E8 5C31FEFF       call 3\_.0041D8ED  
0043A791   |.  40              inc eax  
0043A792   |.  50              push eax  
0043A793   |.  E8 75470000      call 3\_.0043EF0D  
0043A798   |.  59              pop ecx  
0043A799   |.  5E              pop esi  
0043A79A   |.  8BF8            mov edi,eax  
0043A79C   |.  F3:A4           rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]  
0043A79E   |>  E9 00000000      jmp 3\_.0043A7A3  
0043A7A3   |>  50              push eax  
0043A7A4   |.  8B5D FC         mov ebx,dword ptr ss:\[ebp-4\]  
0043A7A7   |.  53              push ebx  
0043A7A8   |.  68 0A000600     push 6000A  
0043A7AD   |.  6A 00           push 0  
0043A7AF   |.  53              push ebx  
0043A7B0   |.  6A 01           push 1  
0043A7B2   |.  BB FC000000      mov ebx,0FC  
0043A7B7   |.  B8 05000000      mov eax,5  
0043A7BC   |.  E8 6A470000      call 3\_.0043EF2B  
0043A7C1   |.  83C4 10         add esp,10  
0043A7C4   |.  E8 3E470000      call 3\_.0043EF07  
0043A7C9   |.  83C4 04         add esp,4  
0043A7CC   |.  8B5D F8         mov ebx,dword ptr ss:\[ebp-8\]  
0043A7CF   |.  53              push ebx  
0043A7D0   |.  68 0A000600     push 6000A  
0043A7D5   |.  6A 00           push 0  
0043A7D7   |.  53              push ebx  
0043A7D8   |.  6A 01           push 1  
0043A7DA   |.  BB FC000000      mov ebx,0FC  
0043A7DF   |.  B8 05000000      mov eax,5  
0043A7E4   |.  E8 42470000      call 3\_.0043EF2B  
0043A7E9   |.  83C4 10         add esp,10  
0043A7EC   |.  E8 16470000      call 3\_.0043EF07  
0043A7F1   |.  83C4 04         add esp,4  
0043A7F4   |.  8B5D F4         mov ebx,dword ptr ss:\[ebp-C\]  
0043A7F7   |.  53              push ebx  
0043A7F8   |.  68 0A000600     push 6000A  
0043A7FD   |.  6A 00           push 0  
0043A7FF   |.  53              push ebx  
0043A800   |.  6A 01           push 1  
0043A802   |.  BB FC000000      mov ebx,0FC  
0043A807   |.  B8 05000000      mov eax,5  
0043A80C   |.  E8 1A470000      call 3\_.0043EF2B  
0043A811   |.  83C4 10         add esp,10  
0043A814   |.  E8 EE460000      call 3\_.0043EF07  
0043A819   |.  83C4 04         add esp,4  
0043A81C   |.  8B5D F0         mov ebx,dword ptr ss:\[ebp-10\]  
0043A81F   |.  85DB            test ebx,ebx  
0043A821   |.  74 09           je short 3\_.0043A82C  
0043A823   |.  53              push ebx  
0043A824   |.  E8 DE460000      call 3\_.0043EF07  
0043A829   |.  83C4 04         add esp,4  
0043A82C   |>  58              pop eax  
0043A82D   |.  8BE5            mov esp,ebp  

************************************************************************************************
```

总结：算法看了很讨厌,不过是明码的

具体算法没看以后有空我会慢慢跟踪一下

`4dd466d0e1e0cc123afbe504da2462ca`

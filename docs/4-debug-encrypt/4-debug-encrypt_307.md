

# XXDownload1.14

> 原文：[`www.pediy.com/kssd/pediy05/pediy50373.htm`](https://www.pediy.com/kssd/pediy05/pediy50373.htm)

from DEDE we got the info below:   
\--------------------------------   
005A1F1D  E84224E6FF            call    00404364                  ; cat MC behind NAME and a '-', and form a long STRING   
005A1F22  8B45EC                mov    eax, \[ebp\-$14\]   
005A1F25  5A                    pop    edx   
  
005A1F26  E859180300            call    005D3784                  ; here is the main call for CODE   
005A1F2B  84C0                  test    al, al   
  
let's deep into CALL 5D3784, and see what is in it:   
\---------------------------------------------------   
005D37C8  8B45FC                mov    eax, \[ebp\-$04\]            ; here is the long STRING   
005D37CB  E848000000            call    005D3818                  ; some kind calculation   
005D37D0  8B45F0                mov    eax, \[ebp\-$10\]            ; the result CODE   
005D37D3  8B55F8                mov    edx, \[ebp\-$08\]            ; the input CODE   
  
\* Reference to: system.@LStrCmp;   
005D37D6  E8D90BE3FF            call    004043B4   
005D37DB  7506                  jnz    005D37E3                  ; FAILED!   
  
  
see what is in CALL 005D3818:   
\-----------------------------   
005D3851  |. 8D4D D8        LEA ECX,DWORD PTR SS:\[EBP\-28\]   
005D3854  |. BA B8385D00    MOV EDX,unpacked.005D38B8                ; ASCII "hidownload1.14"   
005D3859  |. 8B45 FC        MOV EAX,DWORD PTR SS:\[EBP\-4\]            ; Long STRING   
005D385C  |. E8 8FDF0000    CALL unpacked.005E17F0                  ; step 1()   
          result1 is: 'ylUQQbbOCBkVHn7X/POg+V/BefqmnRucVd3yORd/xh=='   
  
  
005D3861  |. 8B45 D8        MOV EAX,DWORD PTR SS:\[EBP\-28\]            ; result1   
005D3864  |. 8D55 DC        LEA EDX,DWORD PTR SS:\[EBP\-24\]   
005D3867  |. E8 4037FAFF    CALL unpacked.00576FAC                  ; step 2()   
          result2 is: 92 B6 9C FE 3A 66 FE 95 7C 11 C0 AD 28 2B 6C F1 128bits   
  
005D386C  |. 8D45 DC        LEA EAX,DWORD PTR SS:\[EBP\-24\]            ; result2   
005D386F  |. 8B55 F8        MOV EDX,DWORD PTR SS:\[EBP\-8\]   
005D3872  |. E8 A937FAFF    CALL unpacked.00577020                  ; step 3(change result2 to a HEX string)   
                                                                    ; the HEX string is the right code   
\----------------------------------   
see step 1 in CALL 005E17F0 first:   
\----------------------------------   
005E182A  |. A1 F8C85400    MOV EAX,DWORD PTR DS:\[54C8F8\]   
005E182F  |. E8 9CB1F6FF    CALL unpacked.0054C9D0                  ; BlowFish.Create   
005E1834  |. 8945 F0        MOV DWORD PTR SS:\[EBP\-10\],EAX            ; store BlowFish   
005E1837  |. 8B55 F8        MOV EDX,DWORD PTR SS:\[EBP\-8\]   
005E183A  |. 8B45 F0        MOV EAX,DWORD PTR SS:\[EBP\-10\]   
005E183D  |. E8 1EAFF6FF    CALL unpacked.0054C760   
  
          CALL unpacked.0054C760:   
          -----------------------   
          0054C76C  |. A1 C0BD5400    MOV EAX,DWORD PTR DS:\[54BDC0\]   
          0054C771  |. E8 06F7FFFF    CALL unpacked.0054BE7C        ; SHA1.Create   
          0054C776  |. 8BD8          MOV EBX,EAX   
          0054C778  |. 8BC3          MOV EAX,EBX   
          0054C77A  |. 8B10          MOV EDX,DWORD PTR DS:\[EAX\]   
          0054C77C  |. FF52 34        CALL NEAR DWORD PTR DS:\[EDX+34\]; SHA1.Initial values(0x67452301...)   
  
          0054C7B0  |. 8B08          MOV ECX,DWORD PTR DS:\[EAX\]    ; 'hidownload1.14'   
          0054C7B2  |. FF51 40        CALL NEAR DWORD PTR DS:\[ECX+40\]; SHA1.Encrypt   
  
          SHA1('hidownload1.14') = FD BD AD D9 20 79 52 03 2A 24 0B AE 48 E7 ED 7E F0 28 6A 8B   
  
          0054C7D0  |. 8BD6          MOV EDX,ESI   
          0054C7D2  |. 8BCD          MOV ECX,EBP   
          0054C7D4  |. 8BC7          MOV EAX,EDI   
          0054C7D6  |. 8B38          MOV EDI,DWORD PTR DS:\[EAX\]   
          0054C7D8  |. FF57 30        CALL NEAR DWORD PTR DS:\[EDI+30\]; BlowFish\_Init(SHA1.result)   
                                                                    ; BlowFish\_EN(-1)   
  
005E1867  |. 8BD0          MOV EDX,EAX   
005E1869  |. 8B45 F0        MOV EAX,DWORD PTR SS:\[EBP\-10\]   
005E186C  |. 59            POP ECX   
005E186D  |. 8B18          MOV EBX,DWORD PTR DS:\[EAX\]   
005E186F  |. FF53 4C        CALL NEAR DWORD PTR DS:\[EBX+4C\]          ; Loops of BlowFish\_EN xor long STRING   
                                                                    ; if U want to know more, just track in   
  
005E1875  |. 8B45 EC        MOV EAX,DWORD PTR SS:\[EBP\-14\]            ; result of last op   
005E1878  |. E8 FBA2F6FF    CALL unpacked.0054BB78                  ; something like base64   
          'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'   
  
005E187D  |. 8B55 E8        MOV EDX,DWORD PTR SS:\[EBP\-18\]            ; result of last op   
  
\--------------------------------------   
then see step 2 in CALL 00576FAC next:   
\--------------------------------------   
00576FCE  |. 8D45 A0        LEA EAX,DWORD PTR SS:\[EBP\-60\]   
00576FD1  |. E8 1AFEFFFF    CALL unpacked.00576DF0                  ; MD5.Initial   
  
00576FED  |. E8 52FEFFFF    CALL unpacked.00576E44                  ; grouped result1   
00576FF2  |. 8B55 F8        MOV EDX,DWORD PTR SS:\[EBP\-8\]   
00576FF5  |. 8D45 A0        LEA EAX,DWORD PTR SS:\[EBP\-60\]   
00576FF8  |. E8 1FFFFFFF    CALL unpacked.00576F1C                  ; MD5.Encrypt   
                                                                    ; it is the result2   
  
  
  In HiDownLoad1.15 it still use visible code compare:), but how to get the code changed:   
  
  Name + ':' + EMail + 'chs-1.15'   
  
  MD5   
  
  change MD5's to string  4dd466d0e1e0cc123afbe504da2462ca
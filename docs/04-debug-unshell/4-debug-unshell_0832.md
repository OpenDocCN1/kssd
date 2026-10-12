
# Hying 壳的 OEP 脚本 bugfix

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-790.htm`](https://www.pediy.com/kssd/pediy07/pediy7-790.htm)

修复个 bug,似乎还是 2 对付后门安全些。

```
///////////////////////////////////////////////////////////////////////////

// Hying's PE-Armor v0.7x Unpacking Script

//

// Author:       forgot

// OS:           Windows XP SP2

// Date:         2005-07-24

// Config:       Ignore all exceptions

// Feature:      Anti Anti Debugging

//               OEP Detection

//               Imports Decryption(unfinished)

//

var                 i

var                 j

var                 k

var                 ldrbase

var                 ldrsize

var                 importflag

var                 apioutputstr

var                 apivalloc

var                 apicfilew

var                 apigmodhw

var                 apigetheap

var                 patch1

var                 setfakeoep

start:

                    // get API address

                    gpa     "VirtualAlloc", "Kernel32.Dll"

                    mov     apivalloc, $RESULT

                    gpa     "CreateFileW", "Kernel32.Dll"

                    mov     apicfilew, $RESULT

                    gpa     "GetModuleHandleW", "Kernel32.Dll"

                    mov     apigmodhw, $RESULT

                    gpa     "OutputDebugStringA", "Kernel32.Dll"

                    mov     apioutputstr, $RESULT

                    gpa     "GetProcessHeap", "Kernel32.Dll"

                    mov     apigetheap, $RESULT

                    // hide debugger

                    dbh

                    mov     i, apioutputstr         // ignore OutputDebugStringA trap

                    asm     i, "pop eax"            // "retn xx" will be detected

                    add     i, $RESULT

                    asm     i, "add esp, 4"

                    add     i, $RESULT

                    asm     i, "jmp eax"

                    // bypass GetProcessHeap

                    sub     esp, 4                  // call stack frame

                    mov     [esp], eip

                    mov     eip, apigetheap

      rtu

                    mov     i, eax

                    add     i, 0C

                    and     [i], 2          // clear 27-31 bit

                    // get some info

                    bp      apivalloc

                    eob     __get_depack_info

                    run

__get_depack_info:

                    bc      apivalloc               // hijack VirtualAlloc

                    rtu                 // get address of depacked loader

                    mov     ldrbase, eax

                    // game start

                    bp      apicfilew

                    eob     __cfw

                    run

__cfw:

                    bc      apicfilew

                    rtu

                    bp      apigmodhw

                    eob     __gmhw

                    run

__gmhw:

                    bc      apigmodhw

                    rtu

                    // bypass ZwSetInfromationThread

                    find    eip, #6AFE#             // push -2

                    find    $RESULT, #8D85#         // lea  eax, [ebp+xxxxxxxx]

                    mov     i, $RESULT

                    add     i, 2

                    mov     j, [i]

                    add     j, ebp

                    mov     eip, j

                    // pre-get FAKE entrypoint

                    find    eip, #8B4424CC#         // mov  eax, [esp-34]

                    mov     setfakeoep, $RESULT

                    // skip decrypt procedure

                    find    eip, #87E6#             // xchg esi, esp

                    find    $RESULT, #E2??#         // loop xxxxxxxxx

                    find    $RESULT, #87E6#         // xchg esi, esp

                                                        // anti junk instructions

                    mov     i, $RESULT

                    bp      i

                    eob     __decrypt_0

                    run

__decrypt_0:

                    bc      i

                    // process imports

                    find    eip, #8985#             // mov  [ebp+xxxxxxxx], eax

                    find    $RESULT, #83A5#         // and  dword ptr [ebp+xxxxxxxx], 0

                    find    $RESULT, #8B85#         // mov  eax, [ebp+xxxxxxxx]

                    mov     i, $RESULT

                    bp      i

                    eob     __test_it_enc

                    run

__test_it_enc:

                    bc      i

                    sto

                    mov     importflag, eax

                    // I don't have a target with no imports protections, sorry.

                    //cmp   eax, 0

                    //jz    __normal_it

                    //deihohoho

                    // bypass ZwQueryInformationProcess detection

                    find    eip, #5A775175#         // "ZwQueryInformationProcess"

                    find    $RESULT, #0BC0#         // or   eax, eax

                    mov     i, $RESULT

                    bp      i

                    eob     __zw_q

                    run

__zw_q:

                    bc      i

                    mov     eax, 0

                    // anti anti dumpping

                    find    eip, #64FF3530000000#   // push dword ptr fs:[0]

                    find    $RESULT, #C74020#       // mov  dword ptr [eax+20], xxxxxxxx

                                                        // length = 7

                    mov     i, $RESULT

                    bp      i

                    eob     __antidump

                    run

__antidump:

                    bc      i

                    add     eip, 7

                    // Go Go Go

                    find    eip, #334104#           // xor  eax, [ecx+4]

                    find    $RESULT, #034108#       // add  eax, [ecx+8]

                    find    $RESULT, #33410C#       // add  eax, [ecx+C]

                    find    $RESULT, #648F01#       // pop  dword ptr fs:[ecx]

                    mov     i, $RESULT

                    bp      i

                    eob     __seh_3

                    run

__seh_3:

                    bc      i

                    // go to OEP

                    eoe     __find_bound

                    run

__find_bound:

                    mov     i, [eip]

                    and     i, 0FFFF

                    cmp     i, 8562                 // bound  eax, [ebp+4148E8]

                    je      __bound

                    esto

                    jmp     __find_bound

__bound:

                    mov     i, setfakeoep

                    bp      i

                    eob     __final

                    esto

__final:

                    sti

                    mov     i, eax

                    bp      i

                    eob     __oep

                    run

__oep:

                    bc      i
```



```
// game over

ret

4dd466d0e1e0cc123afbe504da2462ca
```

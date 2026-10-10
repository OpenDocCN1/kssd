

# VMProtect 逆向分析

> 原文：[`www.pediy.com/kssd/pediy10/70192.html`](https://www.pediy.com/kssd/pediy10/70192.html)

VMProtect, Part 0: Basics  
  
Author: RolfRolles  
  
VMProtect is a virtualization protector. Like other protections in the genre, among others   
ReWolf's x86 Virtualizer and CodeVirtualizer, it works by disassembling the x86 bytecode of   
the target executable and compiling it into a proprietary, polymorphic bytecode which is   
executed in a custom interpreter at run-time. This is unlike the traditional notions of   
packing, in which the x86 bytecode is simply encrypted and/or compressed: with virtualization,   
the original x86 bytecode in the protected areas is gone, never to be seen again. Or so the   
idea goes.  
  
If you've never looked at VMProtect before, I encourage you to take a five-minute look in   
IDA (here's a sample packed binary). As far as VMs go, it is particularly skeletal and   
easily comprehended. The difficulty lies in recreating working x86 bytecode from the VM   
bytecode. Here's a two-minute analysis of its dispatcher.  
  
push edi ; push all registers  
push ecx  
push edx  
push esi  
push ebp  
push ebx  
push eax  
push edx  
pushf  
push 0 ; imagebase fixup  
mov esi, \[esp+8+arg\_0\] ; esi = pointer to VM bytecode  
mov ebp, esp ; ebp = VM's "stack" pointer  
sub esp, 0C0h  
mov edi, esp ; edi = "scratch" data area  
  
VM\_\_FOLLOW\_\_Update:  
add esi, \[ebp+0\]  
  
VM\_\_FOLLOW\_\_Regular:  
mov al, \[esi\] ; read a byte from EIP  
movzx eax, al  
sub esi, -1 ; increment EIP  
jmp ds:VM\_\_HandlerTable\[eax\*4\] ; execute instruction handler  
  
A feature worth discussing is the "scratch space", referenced by the register edi throughout   
the dispatch loop. This is a 16-dword-sized area on the stack where VMProtect saves the   
registers upon entering the VM, modifies them throughout the course of a basic block,   
and from whence it restores the registers upon exit. For each basic block protected by the VM,   
the layout of the registers in the scratch space can potentially be different.  
  
Here's a disassembly of some instruction handlers. Notice that A) VMProtect is a stack machine   
and that B) each handler -- though consisting of scant few instructions -- performs several   
tasks, e.g. popping several values, performing multiple operations, pushing one or more values.  
  
#00: x = \[EIP-1\] & 0x3C; y = popd; \[edi+x\] = y  
  
.text:00427251 and al, 3Ch ; al = instruction number  
.text:00427254 mov edx, \[ebp+0\] ; grab a dword off the stack  
.text:00427257 add ebp, 4 ; pop the stack  
.text:0042725A mov \[edi+eax\], edx ; store the dword in the scratch space  
  
#01: x = \[EIP-1\] & 0x3C; y = \[edi+x\]; pushd y  
  
.vmp0:0046B0EB and al, 3Ch ; al = instruction number  
.vmp0:0046B0EE mov edx, \[edi+eax\] ; grab a dword out of the scratch space  
.vmp0:0046B0F1 sub ebp, 4 ; subtract 4 from the stack pointer  
.vmp0:0046B0F4 mov \[ebp+0\], edx ; push the dword onto the stack  
  
#02: x = popw, y = popw, z = x + y, pushw z, pushf  
  
.text:004271FB mov ax, \[ebp+0\] ; pop a word off the stack  
.text:004271FF sub ebp, 2  
.text:00427202 add \[ebp+4\], ax ; add it to another word on the stack  
.text:00427206 pushf  
.text:00427207 pop dword ptr \[ebp+0\] ; push the flags  
  
#03: x = \[EIP++\]; w = popw; \[edi+x\] = Byte(w)  
  
.vmp0:0046B02A movzx eax, byte ptr \[esi\] ; read a byte from EIP  
.vmp0:0046B02D mov dx, \[ebp+0\] ; pop a word off the stack  
.vmp0:0046B031 inc esi ; EIP++  
.vmp0:0046B032 add ebp, 2 ; adjust stack pointer  
.vmp0:0046B035 mov \[edi+eax\], dl ; write a byte into the scratch area  
  
#04: x = popd, y = popw, z = x << y, pushd z, pushf  
  
.vmp0:0046B095 mov eax, \[ebp+0\] ; pop a dword off the stack  
.vmp0:0046B098 mov cl, \[ebp+4\] ; pop a word off the stack  
.vmp0:0046B09B sub ebp, 2  
.vmp0:0046B09E shr eax, cl ; shr the dword by the word  
.vmp0:0046B0A0 mov \[ebp+4\], eax ; push the result  
.vmp0:0046B0A3 pushf  
.vmp0:0046B0A4 pop dword ptr \[ebp+0\] ; push the flags  
  
#05: x = popd, pushd ss:\[x\]  
  
.vmp0:0046B5F7 mov eax, \[ebp+0\] ; pop a dword off the stack  
.vmp0:0046B5FA mov eax, ss:\[eax\] ; read a dword from ss  
.vmp0:0046B5FD mov \[ebp+0\], eax ; push that dword 

Part 2: Introduction to Optimization  
Author: RolfRolles  
  
Basically, VMProtect bytecode and the IR differ from x86 assembly language in four ways:  
  
1)  It's a stack machine;  
2)  The IR contains "temporary variables";  
3)  It contains what I've called a "scratch" area, upon which computations are performed   
rather than in the registers;  
4)  In the case of VMProtect Ultra, it's obfuscated (or rather, "de-optimized") in certain   
ways.  
  
It turns out that removing these four aspects from the IR is sufficient preparation for   
compilation into sensible x86 code.  We accomplish this via standard compiler optimizations   
applied locally to each basic block.  In general, there are a few main compiler optimizations   
used in this process.  The first one is "constant propagation".  Consider the following C code.  
  
int x = 1;  
function(x);  
  
Clearly x will always be 1 when function is invoked:  it is defined on the first line,   
and is not re-defined before it is used in the following line (the definition in line one   
"reaches" the use in line two; alternatively, the path between the two is "definition-clear"   
with respect to x).  Thus, the code can be safely transformed into "function(1)".    
If the first line is the only definition of the variable x, then we can replace all uses of   
x with the integer 1.  If the second line is the only use of the variable x, then we can   
eliminate the variable.  
  
The next is "constant folding".  Consider the following C code.  
  
int x = 1024;  
function(x\*1024);  
  
By the above remarks, we know we can transform the second line into "function(1024\*1024);".    
It would be silly to generate code that actually performed this multiplication at   
run-time:  the value is known at compile-time, and should be computed then.    
We can replace the second line with "function(1048576);", and in general we can replace any   
binary operation performed upon constant values with the computed result of that operation.  
  
Similar to constant propagation is "copy propagation", as in the below.  
  
void f(int i)  
{  
  int j = i;  
  g(j);  
}  
  
The variable j is merely a copy of the variable i, and so the variable i can be substituted   
in for j until the point where either is redefined.  Lacking redefinitions,   
j can be eliminated entirely.  
  
The next optimization is "dead-store elimination".  Consider the following C code.  
  
int y = 2;  
y = 1;  
  
The definition of the variable y on line one is immediately un-done by the one on line two.    
Therefore, there is no reason to actually assign the value 2 to the variable;   
the first store to y is a "dead store", and can be eliminated by the standard liveness-based  
compiler optimization known as "dead-store elimination", or more generally   
"dead-code elimination".  Here's an example from the VMProtect IR.  
  
ecx = DWORD Scratch:\[Dword(44)\]  
ecx = DWORD Scratch:\[Dword(20)\]  
  
After dead-store-eliminating the first instruction, it turns out no other instructions use   
Scratch:\[Dword(44)\], and so its previous definition can be eliminated as well. 

Part 3: Optimizing and Compiling  
Author: RolfRolles  
  
The reader should inspect this unoptimized IR listing before continuing.  In an attempt to   
keep this entry from becoming unnecessarily long, the example snippets will be small, but   
for completeness a more thorough running example is linked throughout the text.  
  
We begin by removing the stack machine features of the IR.  Since VMProtect operates on   
disassembled x86 code, and x86 itself is not a stack machine, this aspect of the protection   
is unnatural and easily removed.  Here is a 15-line fragment of VMProtect IR.  
  
push Dword(-88)  
push esp  
push Dword(4)  
pop t3  
pop t4  
t5 = t3 + t4  
push t5  
push flags t5  
pop DWORD Scratch:\[Dword(52)\]  
pop t6  
pop t7  
t8 = t6 + t7  
push t8  
push flags t8  
pop DWORD Scratch:\[Dword(12)\]  
pop esp  
  
All but two instructions are pushes or pops, and the pushes can be easily matched up with   
the pops.  Tracking the stack pointer, we see that, for example, t3 = Dword(4).    
A simple analysis allows us to "optimize away" the push/pop pairs into assignment statements.    
Simply iterate through each instruction in a basic block and keep a stack describing the   
source of each push.  For every pop, ensure that the sizes match and record the location   
of the corresponding push.  We wish to replace the pop with an assignment to the popped   
expression from the pushed expression, as in  
  
t3 = Dword(4)  
t4 = esp  
t7 = Dword(-88)  
  
With the stack aspects removed, we are left with a more conventional listing containing many   
assignment statements.  This optimization substantially reduces the number of instructions   
in a given basic block (~40% for the linked example) and opens the door for other optimizations.    
The newly optimized code is eight lines, roughly half of the original:  
  
t3 = Dword(4)  
t4 = esp  
t5 = t3 + t4  
DWORD Scratch:\[Dword(52)\] = flags t5  
t6 = t5  
t7 = Dword(-88)  
t8 = t6 + t7  
DWORD Scratch:\[Dword(12)\] = flags t8  
esp = t8  
  
A complete listing of the unoptimized IR versus the one with the stack machine features removed   
is here, which should be perused before proceeding.  
  
Now we turn our attention to the temporary variables and the scratch area.  Recall that the   
former were not part of the pre-protected x86 code, nor the VMProtect bytecode -- they were   
introduced in order to ease the IR translation.  The latter is part of the VMProtect bytecode,   
but was not part of the original pre-protected x86 code.  Since these are not part of the   
languages we are modelling, we shall eliminate them wholesale.  On a high level, we treat   
each temporary variable, each byte of the scratch space, and each register as being a variable   
defined within a basic block, and then eliminate the former two via the compiler optimizations   
previously discussed.  
  
Looking again at the last snippet of IR, we can see several areas for improvement.  First,   
consider the variable t6.  It is clearly just a copy of t5, neither of which are redefined   
before the next use in the assignment to t8.  Copy propagation will replace variable t6   
with t5 and eliminate the former.  More generally, t3, t4, and t7 contain either constants   
or values that are not modified between their uses.  Constant and copy propagation will   
substitute the assignments to these variables in for their uses and eliminate them.  
  
The newly optimized code is a slender three lines compared to the original 15; we have removed   
80% of the IR for the running example.  
  
DWORD Scratch:\[Dword(52)\] = flags Dword(4) + esp  
esp = Dword(4) + esp + Dword(-88)  
DWORD Scratch:\[Dword(12)\] = flags Dword(4) + esp + Dword(-88)  
  
The side-by-side comparison can be found here.  
  
The IR now looks closer to x86, with the exception that the results of computations are being   
stored in the scratch area, not into registers.  As before, we apply dead-store elimination,   
copy and constant propagation to the scratch area, removing dependence upon it entirely in the   
process.  See here for a comparison with the last phase.  
  
Here is a comparison of the final, optimized code against the original x86:  
  
push ebp                               push    ebp                      
ebp = esp                              mov     ebp, esp                
push Dword(-1)                         push    0FFFFFFFFh              
push Dword(4525664)                    push    450E60h                  
push Dword(4362952)                    push    offset sub\_4292C8        
eax = DWORD FS:\[Dword(0)\]              mov     eax, large fs:0  
push eax                               push    eax                      
DWORD FS:\[Dword(0)\] = esp              mov     large fs:0, esp          
eflags = flags esp + Dword(-88)                              
esp = esp + Dword(-88)                 add     esp, 0FFFFFFA8h          
push ebx                               push    ebx                      
push esi                               push    esi                      
push edi                               push    edi                      
DWORD SS:\[Dword(-24) + ebp\] = esp      mov     \[ebp-18h\], esp          
call DWORD \[Dword(4590300)\]            call    dword ptr ds:unk\_460ADC  
vmreturn Dword(0) + Dword(4638392)  
  
Code generation is an afterthought.
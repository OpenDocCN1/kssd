

# 软件注册保护与“多态变形混淆”技术浅述

> 原文：[`www.pediy.com/kssd/pediy06/pediy6311.htm`](https://www.pediy.com/kssd/pediy06/pediy6311.htm)

软件注册保护与“多态变形混淆”技术浅述   
  
  
导引   
  
多态（Polymorphic），变形（Metamorphic）和混淆（Obfuscation, Jumble,Chaos）出现在病毒应用技术中已经不是什么新鲜的话题了，变形技术在病毒中的不断改进和发展，已经引起反毒专家的重视，这种变化趋势的确对目前的反毒引擎，自动探测技术提出了更高的要求和挑战。“变形病毒非常难于去分析，他们的随机传染和散布机制带给自动反毒系统大的挑战”\[Peter szor Virus Bulletin 2000\]。   
  
既然这些技术可以用于病毒，当然，放到软件保护中来完全适合。的确，这样的工作一直在进行和发展中。“变形技术是非常有前途的虽然它目前很少被应用于与软件保护。变形技术在每次使用时，通过变换自身代码去阻止被破解，当前还没有哪个保护软件采用了全部的变形技术。\[Pavol Cerven Crackproof Your Software 2002\]。（此书作者已经在他本人发售的软件保护系统：The Slovak Protector －SVKP 中尝试采用了变形技术，不知效果如何）   
  
  
简介   
  
多态：既是完成同一功能采用了不同的方法。多态的特点是随机性和变化性。例如一个检查过程，调用了 3 个子过程完成全部安全检查机制。应用多态技术，这个检查过程在每次程序运行时，会随机调用不同的子程序并以不同的调用次序来完成全部的安全检查计划。如此，破解者很难跟踪，并做出通用破解补丁。这个随机的初始化过程一般由一个初始化程序完成，这个程序通常称为“多态引擎”。   
  
变形：基于多态技术，它发展和提升了多态技术。它可以动态生成更复杂的检查过程，包括对重要程序片断的动态加密、解密，动态变换全部检查过程的代码和数据，基于内置的各种不同加密、解密算法。对目标代码段进行动态控制和编/解码。这其中可能还包括代码混淆过程。使得跟踪者迷惑于复杂的变换和还原过程，无法找到真正的检查过程和解码程序。如果其中包含复杂的加密算法（比如多态加密算法，参见后文中的连接地址）会使得整个安全检查机制十分难于分析和最终的破解。“多态加密算法属于当今的强加密算法之一，很可能是最强的”\[CyProtect AG\]   
  
混淆：将一段代码变得非常难以理解，但功能完全相同。例如：   
  
原始代码：   
Mov ax, 01   
Mov bx,34   
Push ax   
Call 123456   
  
混淆后代码：   
Mov ax,09   
Mov bx,4   
Shr bx,1   
Sub ax,bx   
Mov si, 123456   
Nop   
Xchg ax,bx   
Push bx   
Pop ax   
Push 34   
Pop bx   
Call \[si\]   
  
显然，混淆后的代码更需要花费更多时间去分析。通常，混淆代码是由一个混淆发生器（Obfuscation generator）来自动生成的，它按照事先约定的特征标识去使用哪些未被混淆过程中应用的寄存器去搞乱目标程序片断，并且它可以控制生成代码片断的长度和复杂性，以便于随后的其他加、解密程序修改相应的特定程序片断。当然，这个混淆发生器需要全配合整个程序中其他处理过程的，并非独立的非关联性子程序。   
  
通过上述简单的描述，大家可以看出，一套好的加密机制，需要非常严密的构造。高强度，随机性，动态变换的基于多态，变形和混淆技术的加密系统，是完全可以实现的。在这个较新的领域，已经有了比较好的理论基础，和一些实践应用。   
  
  
简单应用分析   
  
PE 加密   
  
通用的 PE 加密商用软件产品的发展趋势是逐渐融合外壳与被加密程序，使得普通脱壳后的程序根本无法正常运行。由于一部分原始代码已经被转移到外壳中，所以整个程序执行期间，外壳与被加密程序已经是不可分割的一个整体，所以破解变得必须搞清更多的加密程序以及原始程序的细节。对于较大的软件，这是一个非常困难的工作。   
  
如果 PE 加密采用注册方式（这几乎也是最常见的方式），它可以在 PE 加密软件中本身定义一个 key 空间，并依据个别计算机硬件特征，动态生成一个范围更小的有效 key 空间。当被加密软件尚未被真正注册时，每次运行它时动态产生一组有效 key 用于校验过程，当然这组 key 不会出现在判断过程，而是被运行时用于控制程序执行序列和调用程序入口产生。如此，跟踪会变得很困难。   
  
Pe 加密程序可以定期更新自己的加、解密算法和 key 空间数据库从自家的主机上，来提高自身的加密强度和随机性。当一个真正的用户注册了程序后，pe 加密程序应该设置足够的功能，使得产生的有效 key 仅仅依赖用户的计算机，被注册后的程序不能脱离本机执行，并且在一定的时间许可范围内有效。这些技术相信已经广泛被采用。这样，避免了合法用户非法散布有效 key 提供给其他人注册一个未注册的程序版本。外壳程序好像是被加密软件的一部分，一旦加装，无法剥离。当然，这种有效融合技术，需要 PE 加密软件的生产者具有很强的功力，并且加密机制是经过周全的研究而建立起来的。   
  
一个简单的加密检查过程，可以采用动态程序地址生成技术，例如   
Call 取得解密字   
Mov ax,解密字 1   
Mov bx,解密字 2   
Mov cx,解密字 3   
Mov dx,解密字 4   
Mov si,地址表基地址指针   
Call \[si+ax\]   
Call \[si+bx\]   
Call \[si+cx\]   
Call \[si+dx\]   
  
解密字可以是取自注册号，注册号可以采用单向 hash 算法（例如 md5）通过用户输入的用户名和其他信息（比如硬件信息）算出。如此，如果程序中提供 20 解密子程序，调用次序的不同，在解密字不可知，解码过程不确定的情况下，穷举测试跟踪和暴力攻击变得几乎难以进行。   
  
有效的注册码包含有效的解码字和解码次序，这些解码字以及执行次序被 PE 保护程序事先已经初始化，相关的检查过程同样已经被初始化，本地化，用于配合有效注册码。这一步，应该是被在用户计算机上安装被加密程序时进行的。如果破解者，这是跟踪程序，由于算法过程的复杂性，跟踪者很难明白，程序安装过程中的行为的意义。如此，一旦程序被安装在用户计算机中，有效的解码字和解码次序已经被包含在用户计算机上生成的数据库中。所以一个用户拥有一个独立的注册生成器，它不能用于生成通用的注册码，它的有效性基于用户的计算机，它可以包含本地的 pe 加密软件中。   
  
破解者无法找到一个通用的办法去破解被保护软件即使他已经是合法使用者。有关的简化过程演示，小弟已经在一个简单的 crackme 中完成，虽然，它仅仅表现了一个静态的检查过程，但可以把它看成是应用多态变形技术在一次执行过程中的静态内存影像。真正应用多态变形技术，应该每次执行都有不同的检查过程，这个过程还有可能动态改变。（crackme 源代码附后，请参考）。   
  
有关该主题更多的信息可以在 INTERNET 上查找，这里只给出几个有关关联，参看多态加密算法“Introduction to the polymorphic encryption method http://english.cyprotect.com/main0110.php”，以及“Polymorphic Cipher Theory C.B. Roellgen, 2002) http://www.ciphers.de/downloads/roellgen02generalizedPMCmodel.pdf”   
  
  
随笔   
  
关于花指令   
  
一般的花指令仅仅是混淆技术的简单化应用，它部分的实现混淆技术，但缺少随机性，复杂性，动态性。因此，它容易被破解者编制一个小程序自动移除。当然，一个好的有一定强度的混淆器绝对是需要很多时间去调试的，估计对于一个有经验的程序员，这个工作需要几个月的时间，这其中，可能包含一些基本的汇编指令编译原理，如果加上针对不同类型 CPU 指令流的预取功能分析，对不同平台的兼容性，适应性研究，这个工作看起来就不大适合普通的编程人员了。因为需要很多基础知识的运用。在程序保护技术中，混淆是一种很好的对付反向工程的措施。   
  
关于保护强度   
  
一个保护软件的强度并不仅仅基于它采用的加密算法的强度。往往数学和理论上的强加密算法在应用过程中由于各种条件限制或者编程人员的疏忽，使得信赖它的采用者最终失去利益。这个世界上从来就没有不可能，只有想不到。所以，有效的机制必须被建立在一个软件保护产品规划的初期，并被不断的完善和发展。这其中综合了好的加密算法，反跟踪技术，混淆技术，程序构造复杂性研究等等。   
  
只有不断的创新和发展，才是出路。因为技术是不断发展的，软件保护者如果闭起他的眼睛，不再关注破解者的努力时，他的产品就已经过时了。   
  
法律规定任何人在未经允许的情况下，不得进入你的房子，但你更愿意给自己的房子加把锁。1650 位的 RSA 真的需要 25 年才可以攻破吗？   
  
\====crackme source codes====   
  
  
;========================================================================   
;                                                                       =   
; Title: Poly-metamorphism algorithm Testing System                     =   
;                                                                       =   
; Version: 1.0.2                                                        =   
;                                                                       =   
; Author: Virtualspace                                                  =   
;                                                                       =   
; Description:                                                          =   
;                                                                       =   
;      This Testing System is designed for demonstrating                =   
;      a simple implement based on Poly-metamorphism algorithm.         =   
;      the algorithm is proposed by Virtualspace to improve             =   
;      registration number protection technique with software           =   
;      protection industry.                                             =   
;                                                                       =   
;                                                                       =   
; Created Date  : 04,Aug,2003                                           =   
; Modified Date : 19,Aug,2003                                           =   
;                                                                       =   
;                                                                       =   
; Note:The Source code is designed in Assembly Language(Intel) for      =   
;      suiting running qualifications under Windows/98/2000/XP/2003     =   
;                                                                       =   
;      default   user name : take me home                               =   
;                       MD5: 4FA4E6AD4FE671273B49F08838ACC9D3           =   
;                          : 4FA4-E6AD-4FE6-7127-3B49-F088-38AC-C9D3  =   
;       Registration Number: 4FA4-E6AD-4FE6-7127-1884-1F20-1ABE-1AEC    =   
;                       xor:                          efa8 2212 d33f    =   
; 3 decryption is efa8 2212 d33f                                        =   
;                  =   
; magic number 1 :d33f xor 2212 xor efa8 = 1e85        =   
; magic number 2 :d33f sub 2212 xor efa8 = 5e85        =   
; magic number 2-1 (ah xchg al)= 40          =   
; magic number 3 :efa8 sub 2212 xor d33f = 1ea9        =   
;                  =   
;                  =   
; Comments:                =   
;                  =   
; This version of crackme is encrypted by        =    
; different encryption algorithms          =   
;                  =   
;                  =   
;========================================================================   
  
  
.386   
.model flat,stdcall   
option casemap:none   
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Include files                                                         \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
include    masm32includegdi32.inc   
include    masm32includewindows.inc   
include    masm32includeuser32.inc   
include    masm32includekernel32.inc   
includelib masm32libuser32.lib   
includelib masm32libkernel32.lib   
includelib masm32libgdi32.lib   
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Function prototypes                                                   \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
Dialog\_Main          PROTO :DWORD,:DWORD,:DWORD,:DWORD   
Dialog\_Process       PROTO :HWND,:UINT,:WPARAM,:LPARAM   
Dialog\_About\_Process   PROTO :HWND,:UINT,:WPARAM,:LPARAM   
Dlalog\_Success       PROTO :DWORD,:DWORD,:DWORD,:DWORD   
MessageBox              PROTO :DWORD,:DWORD,:DWORD,:DWORD    
  
MD5HASH\_Process         PROTO :DWORD,:DWORD,:DWORD   
HEX2ASCII               PROTO :DWORD,:DWORD,:DWORD   
  
Decryption\_Word\_Process PROTO   
Decryption\_Process      PROTO :DWORD,:DWORD,:DWORD,:DWORD   
Encryption\_Process      PROTO :DWORD,:DWORD,:DWORD,:DWORD   
  
Judge\_Process           PROTO :HWND,:DWORD,:DWORD,:DWORD,:DWORD   
Check\_Process           PROTO :DWORD,:DWORD,:DWORD   
  
Block\_Encrypt\_Process   PROTO    
Block\_Decrypt\_Process   PROTO    
  
Tea\_Encrypt\_Process    PROTO   
Tea\_Decrypt\_Process    PROTO   
  
  
Function\_Jmp\_Next\_Process  proto   
  
  
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Macros                                                                \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
RGB macro red,green,blue   
    xor eax,eax   
    mov ah,blue   
    shl eax,8   
    mov ah,green   
    mov al,red   
endm   
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Initialize constant                                                   \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
INITIALIZE\_ENCRYPTION\_INFORMATION equ 0   
  
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Defined constants                                                      \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
.const   
  
IDD\_DIALOG\_MAIN          equ            1000   
  
IDC\_EDIT\_USER\_NAME            equ                     1004   
IDC\_EDIT\_EMAIL                equ                     1005   
IDC\_EDIT\_REGISTRATION\_NUMBER    equ                     1006   
IDC\_BUTTON\_SUBMIT             equ                     1007   
IDC\_BUTTON\_CLEAR              equ                     1008   
IDC\_BUTTON\_EXIT               equ                     1009   
IDC\_BUTTON\_ABOUT              equ                     1010   
  
IDD\_DIALOG\_ABOUT        equ            2000   
IDC\_BUTTON\_ABOUT\_OK        equ            2001   
  
  
dtBufferLength equ 64   
  
;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   
; ENCRYPT TABLE   
;   
; Ensure all values are different   
;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   
  
  
FUNCTION\_1\_ENCRYPT\_VALUE\_1 equ  04f9ah   
FUNCTION\_1\_ENCRYPT\_VALUE\_2 equ  023cdh   
FUNCTION\_1\_ENCRYPT\_VALUE\_3 equ  06685h   
FUNCTION\_1\_ENCRYPT\_VALUE\_4 equ  02a73h   
  
FUNCTION\_2\_ENCRYPT\_VALUE\_1 equ  0efa8h   
FUNCTION\_2\_ENCRYPT\_VALUE\_2 equ  045c3h   
FUNCTION\_2\_ENCRYPT\_VALUE\_3 equ  03cdeh   
FUNCTION\_2\_ENCRYPT\_VALUE\_4 equ  06679h   
  
FUNCTION\_3\_ENCRYPT\_VALUE\_1 equ  02212h   
FUNCTION\_3\_ENCRYPT\_VALUE\_2 equ  011a1h   
FUNCTION\_3\_ENCRYPT\_VALUE\_3 equ  02133h   
FUNCTION\_3\_ENCRYPT\_VALUE\_4 equ  03344h   
  
FUNCTION\_4\_ENCRYPT\_VALUE\_1 equ  0d33fh   
FUNCTION\_4\_ENCRYPT\_VALUE\_2 equ  0a0e6h   
FUNCTION\_4\_ENCRYPT\_VALUE\_3 equ  0f89ah   
FUNCTION\_4\_ENCRYPT\_VALUE\_4 equ  00d44h   
  
  
;ENCRYPT TABLE END   
;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   
  
  
  
;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   
; VALID ADDRESS MARK   
;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   
  
FUNCTION\_1\_VALID\_ADDRESS\_MARK\_1    equ 044558765h   
FUNCTION\_2\_VALID\_ADDRESS\_MARK\_2    equ 041243434h   
FUNCTION\_3\_VALID\_ADDRESS\_MARK\_3    equ 04df52342h   
FUNCTION\_4\_VALID\_ADDRESS\_MARK\_4    equ 04e7f12eah   
  
  
; VALID ADDRESS MARK END   
;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   
  
FUNCTION\_MAX\_VALID\_ADDRESS\_AMOUNT equ 5   
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Initialized data                                                      \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
.data   
Dialog\_Main\_Name    db "IDD\_DIALOG\_MAIN",0   
Dialog\_Success\_Name db "Dialog\_Success",0   
  
Error\_Name          db "  Error",0   
Error\_Name\_Empty    db "       ",0   
ErrorString         db "You have entered an invalid password.",0   
SuccessString       db "Congratulation! You did it! You are successful!",0   
Error\_String        db "You have entered an invalid password.",0   
  
Success\_String      db "Congratulation! You did it! You are successful!",0   
  
Success\_Info\_Address dd Success\_Info1   
  
                     db 056h,21h,0f5h,0a3h,0beh,08dh    ;invalid chaos codes   
  
Success\_Decryption\_Word dw 0dfbh                        ;0dfbh is an invalid full code    
  
                     db 0,1,0   
Success\_Info\_Status  db 0,1,1,1   
                     db "sldfm12!4jhghdTYKJ NB(\*("      ;invalid chaos codes   
;Success\_Info        equ $   
;                    db "You are the most welcome!"   
;                    db "This program is a Demo, "   
;                    db "If you have any suggestion,"   
;                    db "Please send email to me ",0   
  
  
    
;encrypted Success\_Info    
;   
;Success\_Info1-4 encryption word are  FUNCTION\_1\_ENCRYPT\_VALUE\_1 equ  0173fh   
;                                     FUNCTION\_1\_ENCRYPT\_VALUE\_2 equ  025e6h   
;                                     FUNCTION\_1\_ENCRYPT\_VALUE\_3 equ  0369ah   
;                                     FUNCTION\_1\_ENCRYPT\_VALUE\_4 equ  0477fh   
;   
  
Success\_Info1   db 020h,0C3h,06Fh,0EFh,03Dh,0FBh,06Fh,0FFh,027h,0EEh,06Fh,0FFh,020h,0F7h,03Bh,0E9h     
        db 038h,0BAh,023h,0FFh,020h,0F9h,02Ah,0F7h,01Bh,0BBh,026h,0F2h,06Fh,0E9h,03Dh,0EAh     
        db 028h,0F5h,02Eh,0E8h,06Fh,0F7h,03Ch,0F3h,02Eh,0BAh,00Bh,0BAh,022h,0FFh,063h,0F5h     
        db 006h,0BAh,06Fh,0FCh,020h,0E3h,06Fh,0EFh,02Eh,0F2h,02Ah,0ECh,02Eh,0BAh,036h,0F4h     
        db 03Ch,0BAh,028h,0EFh,02Ah,0FDh,03Bh,0E9h,020h,0F3h,063h,0F4h,023h,0CAh,02Eh,0FFh     
        db 02Ah,0E9h,03Ch,0BAh,021h,0FFh,06Fh,0FEh,022h,0FFh,026h,0FBh,06Fh,0F6h,020h,0EEh     
        db 020h,06Dh,065h,020h,00h                                                             
                                                                                       
Success\_Info2  db 04Ch,094h,003h,0B8h,051h,0ACh,003h,0A8h,04Bh,0B9h,003h,0A8h,04Ch,0A0h,057h,0BEh     
        db 054h,0EDh,04Fh,0A8h,04Ch,0AEh,046h,0A0h,077h,0ECh,04Ah,0A5h,003h,0BEh,051h,0BDh     
        db 044h,0A2h,042h,0BFh,003h,0A0h,050h,0A4h,042h,0EDh,067h,0EDh,04Eh,0A8h,00Fh,0A2h     
        db 06Ah,0EDh,003h,0ABh,04Ch,0B4h,003h,0B8h,042h,0A5h,046h,0BBh,042h,0EDh,05Ah,0A3h     
        db 050h,0EDh,044h,0B8h,046h,0AAh,057h,0BEh,04Ch,0A4h,00Fh,0A3h,04Fh,09Dh,042h,0A8h     
        db 046h,0BEh,050h,0EDh,04Dh,0A8h,003h,0A9h,04Eh,0A8h,04Ah,0ACh,003h,0A1h,04Ch,0B9h     
        db 020h,06Dh,065h,020h,00h                                                             
                                                                                               
Success\_Info3  db 009h,0DCh,046h,0F0h,014h,0E4h,046h,0E0h,00Eh,0F1h,046h,0E0h,009h,0E8h,012h,0F6h     
        db 011h,0A5h,00Ah,0E0h,009h,0E6h,003h,0E8h,032h,0A4h,00Fh,0EDh,046h,0F6h,014h,0F5h     
        db 001h,0EAh,007h,0F7h,046h,0E8h,015h,0ECh,007h,0A5h,022h,0A5h,00Bh,0E0h,04Ah,0EAh     
        db 02Fh,0A5h,046h,0E3h,009h,0FCh,046h,0F0h,007h,0EDh,003h,0F3h,007h,0A5h,01Fh,0EBh     
        db 015h,0A5h,001h,0F0h,003h,0E2h,012h,0F6h,009h,0ECh,04Ah,0EBh,00Ah,0D5h,007h,0E0h     
        db 003h,0F6h,015h,0A5h,008h,0E0h,046h,0E1h,00Bh,0E0h,00Fh,0E4h,046h,0E9h,009h,0F1h     
        db 020h,06Dh,065h,020h,00h                                                             
                                                                                               
                                                                                               
Success\_Info4   db 045h,02Ah,00Ah,006h,058h,012h,00Ah,016h,042h,007h,00Ah,016h,045h,01Eh,05Eh,000h     
        db 05Dh,053h,046h,016h,045h,010h,04Fh,01Eh,07Eh,052h,043h,01Bh,00Ah,000h,058h,003h     
        db 04Dh,01Ch,04Bh,001h,00Ah,01Eh,059h,01Ah,04Bh,053h,06Eh,053h,047h,016h,006h,01Ch     
        db 063h,053h,00Ah,015h,045h,00Ah,00Ah,006h,04Bh,01Bh,04Fh,005h,04Bh,053h,053h,01Dh     
        db 059h,053h,04Dh,006h,04Fh,014h,05Eh,000h,045h,01Ah,006h,01Dh,046h,023h,04Bh,016h     
        db 04Fh,000h,059h,053h,044h,016h,00Ah,017h,047h,016h,043h,012h,00Ah,01Fh,045h,007h     
        db 020h,06Dh,065h,020h,00h                                                             
  
Success\_Info\_Length equ ($-Success\_Info4)/8   
                    db 0,0,0   
                    db "fd&\*314e5fgfghkl986734w34zxzvcnlvc fdg^",0   
                   
Test\_Success\_Info   db "You are the most welcome!"   
  
Chaos\_Char          db "435$%3sdfwq$%7f",0,"4gH98sSDG^$%",0,05h,076h,24h,04h   
                    db "fd&tyutyutyutyghnbm bvkjhkN34345gnvG!&^",0   
                    db "fd&\*314e56^$&4567\*\*^(&45;'klkl;k;'k;lJQK",0   
  
IF INITIALIZE\_ENCRYPTION\_INFORMATION   
  
Finial\_Success\_Info db "Congratulation!",0ah,0dh,0ah,0dh   
              db "You did, You are an cracking master !",0ah,0dh,0ah,0dh   
          db "This crackme is designed for "   
          db "validating the Poly-metamorphism algorithm, "   
                    db "which is proposed by author, "   
                    db "Any more useful information,Please contact with me by email,"   
                    db "Thanks a lot",0   
  
ELSE   
  
;encrypted success info   
  
Finial\_Success\_Info db 0A3h,0D7h,07Ah,08Fh,04Eh,019h,052h,016h,0E8h,05Dh,0FFh,089h,05Ah,0E5h,029h,0ABh,009h,01Dh,009h,028h,0BAh,014h,063h,0A2h,0E5h,058h,048h,0C1h,059h,0F3h,013h,01Fh   
          db 0A0h,0F1h,0C1h,080h,041h,06Ah,064h,0A2h,0E8h,047h,0FDh,09Ah,05Ah,0E3h,013h,06Dh,0B1h,0FDh,06Dh,0B2h,0BDh,06Eh,026h,0A2h,0BCh,0FFh,05Ch,023h,060h,0C8h,040h,013h   
          db 0B0h,032h,000h,0B3h,044h,068h,062h,06Dh,0E0h,091h,048h,08Ah,063h,0F8h,04Ah,019h,0A4h,0FBh,07Dh,080h,049h,0ABh,055h,013h,0D3h,091h,0FCh,094h,064h,0E3h,0BFh,06Bh   
          db 0A4h,0F9h,0C1h,086h,047h,01Fh,027h,065h,0DAh,0A1h,0EFh,081h,05Ch,027h,0BFh,06Fh,0A5h,0F1h,06Fh,08Eh,047h,01Bh,054h,069h,0A9h,05Ch,0FCh,094h,059h,0FDh,04Ah,01Ah   
          db 0AAh,0E6h,035h,088h,0B8h,0ABh,05Eh,06Ah,0E1h,056h,0FFh,0D5h,028h,0E9h,041h,014h,0A2h,0E3h,06Eh,08Eh,04Bh,06Eh,065h,0A2h,0A9h,048h,0F3h,094h,060h,0E8h,041h,015h   
          db 073h,03Eh,073h,0FCh,00Fh,002h,058h,06Dh,0E4h,047h,0F3h,0D5h,063h,0E9h,0BEh,06Eh,072h,0FEh,073h,084h,040h,06Dh,05Ah,010h,0D5h,050h,0F9h,09Ch,01Ch,0E6h,047h,074h   
          db 0B1h,0F5h,07Ch,0B2h,04Ch,0ABh,059h,013h,0E8h,045h,0F4h,09Ah,052h,034h,0BFh,013h,072h,0FAh,07Ch,088h,04Dh,0ABh,027h,019h,0DCh,054h,0FFh,094h,01Ch,0E0h,04Bh,078h   
          db 0A4h,0F1h,06Eh,08Ah,04Eh,0ABh,05Bh,0A2h,0D5h,042h,000   
  
ENDIF   
  
  
Decryption\_Block\_Length    equ $-Finial\_Success\_Info   
          db 05h,0edh,013h,04fh,0deh,038h,01ah,099h,0b7h,08ah,09bh,043h   
          db 12h,02eh,02bh,04fh,01eh,02ah,01ah,004h,0b7h,04bh,09dh,0aah   
          db 05h,0dfh,04ch,05eh,044h,032h,02fh,043h,087h,02ah,023h,0aeh   
          db 32h,022h,023h,04fh,0deh,037h,05ah,099h,007h,081h,0fdh,067h   
            db "4Rdf5\*&etretN34try3ty45gnvG!&^",0   
                    db "fd&\*314e5klyuo55678^(&34432LJfgEgnvG!77",0   
  
 dw 08945h   
                       
Decryption\_Word1    dw 03462h       ; invalid value   
Decryption\_Word2    dw 075fdh       ; invalid value   
Decryption\_Word3    dw 05c54h       ; invalid value   
Decryption\_Word4    dw 0ea39h       ; invalid value   
  
Current\_Decrypt\_Word dw 056f2h      ; invalid full value   
  
                    db "fd&56j4l5 ^$&45rt rdt 4yt NLJd65   G!&^",0   
  
IF INITIALIZE\_ENCRYPTION\_INFORMATION   
  
FUNCTION\_1\_ENCRYPT\_VALUE\_1\_ADDRESS dw FUNCTION\_1\_ENCRYPT\_VALUE\_1   
FUNCTION\_1\_ENCRYPT\_VALUE\_2\_ADDRESS dw FUNCTION\_1\_ENCRYPT\_VALUE\_2   
FUNCTION\_1\_ENCRYPT\_VALUE\_3\_ADDRESS dw FUNCTION\_1\_ENCRYPT\_VALUE\_3   
FUNCTION\_1\_ENCRYPT\_VALUE\_4\_ADDRESS dw FUNCTION\_1\_ENCRYPT\_VALUE\_4   
  
  
Show\_Info1          db "You are the most welcome!"   
                    db "This program is a Demo, "   
                    db "If you have any suggestion,"   
                    db "Please send email to me ",0   
                    db "1-end"   
  
Show\_Info2          db "You are the most welcome!"   
                    db "This program is a Demo, "   
                    db "If you have any suggestion,"   
                    db "Please send email to me ",0   
                    db "2-end"   
  
Show\_Info3          db "You are the most welcome!"   
                    db "This program is a Demo, "   
                    db "If you have any suggestion,"   
                    db "Please send email to me ",0   
                    db "3-end"   
  
Show\_Info4          db "You are the most welcome!"   
                    db "This program is a Demo, "   
                    db "If you have any suggestion,"   
                    db "Please send email to me ",0   
                    db "4-end"   
  
  
  
ENDIF   
                                           
TitleText           db "Poly-metamorphism algorithm: CrackMe 1.02"   
                    db 0   
  
FontName            db "NewRoman",0   
FontNameSmall       db "Script",0   
  
  
Mark                db 00   
Hello\_String        db "Hello"    
Error\_Input\_String  db "Invalid length of String inputted"   
                    db "(at least 5 characters)",0   
  
Password\_Name       db "Password",0   
Password\_Value      db 64 dup(0)   
                    db "&&&"   
Password\_String     db 64 dup(0)   
  
EMail               db "Newcastlecity@hotmail.com",0   
  
                 db "                === Welcome ===         ",0dh,0ah,0dh,0ah   
                 db "Please attempt to crack this program version 1.0.2",0dh,0ah   
               db "with your favourite debugger/disassembler.",0dh,0ah   
                   db "There's a tutorial available for this CrackMe.",0dh,0ah   
                   db "If you have any question or suggestion",0dh,0ah   
  
Dialog\_Main\_Welcome\_String db 0ah,0dh   
  
                db "                === Welcome ===         ",0dh,0ah,0dh,0ah   
                db "This program is a testing system        ",0dh,0ah,0   
                db "Version 1.02 Author: Michael Z    2003  ",0dh,0ah,0   
                db "University of Northumbria  2003         ",0dh,0ah,0   
                db "Newcastle City upon Tyne in United Kinkdom ",0dh,0ah,0   
                db "Please send email to me for help. thanks",0dh,0ah,0dh,0ah   
                db "Email Address: Newcastlecity@hotmail.com",0dh,0ah,0   
  
StringBufferSize equ $-Dialog\_Main\_Welcome\_String   
  
StringBuffer    db 0ah,0dh   
                db "                === Welcome ===         ",0dh,0ah,0dh,0ah   
                db "This program is a testing system        ",0dh,0ah,0   
                db "Version 1.02 Author: Michael Z    2003  ",0dh,0ah,0   
                db "University of Northumbria  2003         ",0dh,0ah,0   
                db "Newcastle City upon Tyne in United Kinkdom ",0dh,0ah,0   
                db "Please send email to me for help. thanks",0dh,0ah,0dh,0ah   
                db "Email Address: Newcastlecity@hotmail.com",0dh,0ah,0   
  
db 00h,0EAh,0A6h,05Eh,080h,0e2h,040h,000h,021h,055h,02Dh,06Ah,0F6h,000h,000h,000h,000   
  
Jump\_Address\_Matrix dd offset Dialog\_Process   
            dw 034feh,0409ah   
          dd offset Block\_Encrypt\_Process+10h   
          dd offset Block\_Encrypt\_Process+5   
          dd offset Block\_Decrypt\_Process   
          db 01,02,03,04   
  
Exit\_Address equ $-Jump\_Address\_Matrix  ; Jump\_Exit\_Process   
             
          dd offset Function\_Jump\_Exit\_Address ; point to exit process   
          dd offset xTea\_Encrypt\_Process   
          dd offset Block\_Encrypt\_Process+10dh   
          dd offset Block\_Decrypt\_Process   
            dd offset xBlock\_Decrypt\_Process             
          dd offset Tea\_Decrypt\_Process   
          dd offset Tea\_Encrypt\_Process+15h   
          db 05,06,07,08   
             
Tea\_Encrypt\_Address equ $-Jump\_Address\_Matrix             
          dd offset Tea\_Encrypt\_Process   
          dd offset Block\_Decrypt\_Process+2dh   
          dd offset Block\_Decrypt\_Process   
          dd offset xTea\_Decrypt\_Process   
            dd offset xBlock\_Encrypt\_Process             
          dd offset Block\_Decrypt\_Process-340dh   
          dd offset Block\_Encrypt\_Process   
          db 09h,0ah,0bh,0ch   
             
Block\_Decrypt\_Address equ $-Jump\_Address\_Matrix           
          dd offset Block\_Decrypt\_Process   
             
db 032h,0A4h,068h,04Eh,080h,0DCh,02Ah,06Ah,0F6h,028h,008h,01Dh,081h,090h,0FAh,0DFh,0FFh   
db 089h,0h,046h,002h,000h,000h,068h,062h,04Eh,080h,088h,007h,01Dh,081h,03Ch,02Bh,06Ah,0F6h   
             
MD5\_String      db 32 dup(0)                ;show   
                db 3  dup(0)   
                db 16 dup("\*")   
                   
Hexctr          db 0,0,'$'   
                db 32 dup("$")   
Xlatab          db 30h,31h,32h,33h,34h,35h,36h,37h,38h,39h   
                db 41h,42h,43h,44h,45h,46h   
                db "What"   
Charx           db 0,0   
                db "^^^"   
  
Current\_Time SYSTEMTIME <>  ; Current   "    
          
                db "Time"   
  
; Tiny Encryption Algorithm data area   
; keep 093eah and replace other 3 word with   
; 3 decryption word 4-2  deef 2212 efa8    
;   
Decryption\_Block\_Password equ Tea\_Data   
Decryption\_Block\_Cipher   equ Tea\_Key   
  
;correct values : 0d33fh, 02212h, 0efa8h, 007afh... address:00404a64   
Tea\_Key       dw 04efdh, 03216h, 09adfh, 007afh   
Tea\_Data      dw 00e54h, 02acch, 032afh, 049cdh     ;4 words changeless values   
  
  
IF INITIALIZE\_ENCRYPTION\_INFORMATION   
  
; original calues:  0D33FH, 02212H, 0EFA8H, 007AFH   
;                   00E54H, 02ACCH, 032AFH, 049CDH   
;   
; initial encrypted values:    
;                  0d33fh, 02212h, 0efa8h, 007afh,    
;           089DEh, 0A91Ah, 00939h, 0F3A9h    
  
  
  
Tea\_Encrypt\_Key  dw 0d33fh,02212h,0efa8h,007afh   
Tea\_Encrypt\_Data dw 00e54h,02acch,032afh,049cdh   
  
ENDIF   
               
           dw 0acbeh                ;chaos codes   
           dw 043a9h,0cfeah            ;chaos codes   
           db 013h,0a6h,0b3h,08eh          ;chaos codes   
           
sum     equ     eax   
y       equ     ebx   
z       equ     ecx   
delta   equ     edx   
rounds  equ     di   
t       equ     ebp   
v0      equ     dword ptr \[edi\]   
v1      equ     dword ptr \[edi+4\]   
k0      equ     dword ptr \[esi\]   
k1      equ     dword ptr \[esi+4\]   
k2      equ     dword ptr \[esi+8\]   
k3      equ     dword ptr \[esi+12\]   
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Uninitialized data                                                     \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
.data?   
hInstance      HINSTANCE ?   
CommandLine    LPSTR ?   
szCharCount    db ?   
MD5HASH\_RESULT dd ?,?,?,?   
               db ?,?,?,?   
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
.code   
start:   
  invoke GetModuleHandle, NULL   
  mov hInstance,eax   
  invoke DialogBoxParam,hInstance,IDD\_DIALOG\_MAIN,NULL,ADDR Dialog\_Process,NULL   
  invoke ExitProcess,eax   
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
;\* MAIN DIALOG                                                          \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
Dialog\_Process proc hWnd:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM   
      LOCAL hdc:HDC   
      LOCAL ps:PAINTSTRUCT   
      LOCAL hfont:HFONT   
        
         
    .IF uMsg==WM\_INITDIALOG   
               
            invoke GetDlgItem,hWnd,IDC\_EDIT\_USER\_NAME   
            invoke SetFocus,eax   
            invoke GetMenu,hWnd   
  
  
      .ELSEIF uMsg==WM\_PAINT   
                      
            invoke BeginPaint,hWnd,ADDR ps   
            mov hdc,eax   
            invoke CreateFont,14,9,0,0,100,0,0,0,OEM\_CHARSET,   
                   OUT\_DEFAULT\_PRECIS,CLIP\_DEFAULT\_PRECIS,PROOF\_QUALITY,   
                   VARIABLE\_PITCH OR FF\_ROMAN,ADDR FontNameSmall   
            invoke SelectObject,hdc,eax   
            mov hfont,eax   
;            RGB 4,50,232   
;            invoke SetTextColor,hdc,eax   
;            RGB 233,233,216   
;            invoke SetBkColor,hdc,eax   
;            invoke TextOut,hdc,135,136,ADDR TitleText,SIZEOF TitleText   
            invoke SelectObject,hdc,hfont   
  
     
      .ELSEIF uMsg==WM\_CLOSE   
           
    invoke SendMessage,NULL,WM\_COMMAND,IDC\_BUTTON\_EXIT,0   
    invoke ExitProcess,eax   
                     
                     
      .ELSEIF uMsg==WM\_COMMAND   
    mov eax,wParam   
    .IF lParam==0   
    .ELSE   
             mov edx,wParam   
       shr edx,16   
  
            Check:   
      .IF dx==BN\_CLICKED   
            
                        .IF ax==IDC\_BUTTON\_SUBMIT   
  
                              pushad   
                              mov ecx,StringBufferSize   
                              mov esi,offset Dialog\_Main\_Welcome\_String   
                              mov edi,offset StringBuffer   
                              rep movsb     
                      invoke GetDlgItemText,hWnd,IDC\_EDIT\_USER\_NAME,   
                                     ADDR StringBuffer,32   
                              cmp eax,sizeOf Hello\_String   
                              mov szCharCount,al   
                              jb  GetErrorBox   
                              mov edi,offset StringBuffer   
                              add edi,eax   
                              mov ecx,32   
                              sub ecx,eax   
                              mov al,0f6h   
                              rep stosb                                 
                     StartProc:   
                        
                     ;initialize once original info with default encryption word                        
  
                     IF INITIALIZE\_ENCRYPTION\_INFORMATION   
                              invoke Encryption\_Process, ADDR Show\_Info1, Success\_Info\_Length, ADDR FUNCTION\_1\_ENCRYPT\_VALUE\_1\_ADDRESS, ADDR Show\_Info1   
                              invoke Encryption\_Process, ADDR Show\_Info2, Success\_Info\_Length, ADDR FUNCTION\_1\_ENCRYPT\_VALUE\_2\_ADDRESS, ADDR Show\_Info2   
                              invoke Encryption\_Process, ADDR Show\_Info3, Success\_Info\_Length, ADDR FUNCTION\_1\_ENCRYPT\_VALUE\_3\_ADDRESS, ADDR Show\_Info3   
                              invoke Encryption\_Process, ADDR Show\_Info4, Success\_Info\_Length, ADDR FUNCTION\_1\_ENCRYPT\_VALUE\_4\_ADDRESS, ADDR Show\_Info4   
                              invoke Tea\_Encrypt\_Process   
                              invoke Block\_Encrypt\_Process   
                              popad   
                              ret   
                     ENDIF    
                        
                              invoke MD5HASH\_Process,ADDR StringBuffer,64,   
                                     ADDR MD5HASH\_RESULT   
                              invoke HEX2ASCII, ADDR MD5HASH\_RESULT, 16, ADDR MD5\_String   
                      invoke GetDlgItemText,hWnd,IDC\_EDIT\_REGISTRATION\_NUMBER,   
                                     ADDR StringBuffer,33       ;32+1 registration number=32,0 33 bytes   
                              mov esi, offset StringBuffer            
                              mov edi, offset MD5\_String   
                              mov eax, dword ptr \[esi\]   
                              mov ebx, dword ptr \[edi\]   
                              xor eax,ebx   
                              jnz GetErrorBox   
                              mov eax, dword ptr \[esi+4\]   
                              mov ebx, dword ptr \[edi+4\]   
                              xor eax,ebx   
                              jnz GetErrorBox   
                              jmp EndProc   
                
                  GetErrorBox:        
                              invoke MessageBox,hWnd,ADDR Error\_Input\_String,   
                                     ADDR Error\_Name,MB\_OK or MB\_TASKMODAL   
                  Reset\_Dlg:     
                              invoke SetDlgItemText,hWnd,IDC\_EDIT\_USER\_NAME,NULL   
                              invoke SetDlgItemText,hWnd,IDC\_EDIT\_EMAIL,NULL   
                              invoke SetDlgItemText,hWnd,IDC\_EDIT\_REGISTRATION\_NUMBER,NULL   
                              invoke GetDlgItem,hWnd,IDC\_EDIT\_USER\_NAME   
                              invoke SetFocus,eax    
                              mov ecx,StringBufferSize   
                              mov esi,offset Dialog\_Main\_Welcome\_String   
                              mov edi,offset StringBuffer   
                              rep movsb     
                              popad   
                              mov eax,1    ; exit process   
                              jmp Finish   
                                        
                      EndProc:   
                              invoke Decryption\_Word\_Process   
                              invoke Judge\_Process, hWnd, ADDR Success\_Info1, Success\_Info\_Length, ADDR Decryption\_Word1, ADDR Success\_Info1   
                              cmp eax,1   
                              jz FinishProc   
                              invoke Judge\_Process, hWnd, ADDR Success\_Info2, Success\_Info\_Length, ADDR Decryption\_Word1, ADDR Success\_Info2   
                              cmp eax,1   
                              jz FinishProc   
                              invoke Judge\_Process, hWnd, ADDR Success\_Info3, Success\_Info\_Length, ADDR Decryption\_Word1, ADDR Success\_Info3   
                              cmp eax,1   
                              jz FinishProc   
                              invoke Judge\_Process, hWnd, ADDR Success\_Info4, Success\_Info\_Length, ADDR Decryption\_Word1, ADDR Success\_Info4   
                   FinishProc:                 
                              jmp Reset\_Dlg   
                       Finish:                                              
                              nop                                
  
                        .ELSEIF     ax==IDC\_BUTTON\_CLEAR   
  
                              mov ecx,32   
                              mov esi,offset Dialog\_Main\_Welcome\_String   
                              mov edi,offset StringBuffer   
                              rep stosb     
                              invoke SetDlgItemText,hWnd,IDC\_EDIT\_USER\_NAME,NULL   
                              invoke SetDlgItemText,hWnd,IDC\_EDIT\_EMAIL,NULL   
                              invoke SetDlgItemText,hWnd,IDC\_EDIT\_REGISTRATION\_NUMBER,NULL   
    
  
                        .ELSEIF     ax==IDC\_BUTTON\_EXIT   
  
                          invoke SendMessage,NULL,WM\_COMMAND,   
                                 IDC\_BUTTON\_EXIT,0   
                          invoke ExitProcess,eax   
     
                        .ELSEIF     ax==IDC\_BUTTON\_ABOUT   
  
               invoke SendMessage,hWnd,WM\_COMMAND,IDC\_BUTTON\_ABOUT,0   
              invoke DialogBoxParam,hInstance,IDD\_DIALOG\_ABOUT,NULL,ADDR Dialog\_About\_Process,NULL   
                  
                        .ENDIF               
      .ENDIF   
    .ENDIF   
  .ELSE   
    mov eax,FALSE   
    ret   
  
  .ENDIF   
  mov eax,TRUE   
  ret   
  
    Dialog\_Process endp    
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
;\* ABOUT DIALOG                                                         \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
Dialog\_About\_Process proc hWnd :DWORD,uMsg:DWORD,wParam:DWORD,lParam:DWORD   
      LOCAL hdc:HDC   
      LOCAL ps:PAINTSTRUCT   
      LOCAL hfont:HFONT   
      LOCAL rect:RECT   
                           
      .IF uMsg==WM\_INITDIALOG   
  
            invoke GetDlgItem,hWnd,IDC\_BUTTON\_ABOUT\_OK   
            invoke SetFocus,eax   
  
      .ELSEIF uMsg==WM\_PAINT   
  
            invoke BeginPaint,hWnd,ADDR ps   
            mov hdc,eax            
  
            invoke SelectObject,hdc,eax   
            mov hfont,eax   
            RGB 63,118,192   
            invoke SetTextColor,hdc,eax   
            RGB 233,233,216   
            invoke SetBkColor,hdc,eax   
            invoke SetRect,ADDR rect,75,15,400,300   
            invoke DrawText,hdc,ADDR Dialog\_Main\_Welcome\_String,-1,ADDR rect,   
                            DT\_LEFT or DT\_WORDBREAK   
                   
            invoke EndPaint,hWnd,ADDR ps   
  
      .ELSEIF uMsg==WM\_COMMAND   
           invoke EndDialog,hWnd,NULL   
      .ENDIF   
           
        xor eax,eax   
        ret   
Dialog\_About\_Process endp   
  
  
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; this proccess is a conversion of all hex                              \*   
; to ascii and display it                                               \*   
; al is needed to get the hex                                           \*   
;                                                                       \*   
; input buffer  hex value                                               \*   
; output buffer hex value ascii                                         \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
  
HEX2ASCII proc IptBuffer:dword,IptBufferLength:dword,OptBuffer:dword   
  
    LOCAL LOOP\_COUNTER:DWORD   
  
    mov esi,IptBuffer   
    mov edi,OptBuffer   
    mov eax,IptBufferLength   
    mov LOOP\_COUNTER,eax   
       
LOOP\_HEX2ASC:       
  
    mov al,byte ptr \[esi\]   
    mov Hexctr,al   
    mov ecx,04   
    mov ah,0   
    shr ax,cl   
    lea ebx,Xlatab   
    xlat   
    mov Charx,al   
    mov al,byte ptr Hexctr   
    shl ax,cl   
    shr al,cl   
    xlat   
    mov Charx+1,al   
    mov ax,word ptr Charx   
    mov word ptr \[edi\],ax   
    add edi,2   
    add esi,1   
  mov eax,LOOP\_COUNTER   
  dec eax   
  jz EXIT\_HEX2ASC   
  mov LOOP\_COUNTER,eax   
  jmp LOOP\_HEX2ASC   
     
EXIT\_HEX2ASC:   
       
    ret   
  
HEX2ASCII endp   
  
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; procMD5hash : hashes a string using the md5 algorithm   
;   
; input  :   
;   
;  ptBuffer: pointer to the string buffer    
;  (doesn't have to be zero-terminated, must be at least 64bytes large)   
;   
;  dtBufferLength: length of the buffer   
;  ptMD5Result: pointer to a MD5RESULT structure   
;   
; output :   
;   
;  ptMD5Result: contains the hash dwords in dtA, dtB, dtC, dtD   
;   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
  
MD5RESULT  STRUCT   
 dtA  dd ?   
 dtB  dd ?   
 dtC  dd ?   
 dtD  dd ?   
MD5RESULT  ENDS   
  
  
FF   MACRO dta,dtb,dtc,dtd,x,s,t    ; a = b + ((a + F(b,c,d) + x + t) << s )   
  
   mov eax,dtb   
   mov ebx,dtc   
   mov ecx,dtd   
  
   ; F(x,y,z) = (x and y) or ((not x) and z)   
   and ebx,eax   
   not eax   
   and eax,ecx   
   or eax,ebx   
  
   add eax,dta   
   add eax,x   
   add eax,t   
  
   mov cl,s   
   rol eax,cl   
  
   add eax,dtb   
  
   mov dta,eax   
  
ENDM   
  
  
GG   MACRO dta,dtb,dtc,dtd,x,s,t    ; a = b + ((a + G(b,c,d) + x + t) << s)   
  
   mov eax,dtb   
   mov ebx,dtc   
   mov ecx,dtd   
  
   ; G(x,y,z) = (x and z) or (y and (not z))   
  
   and eax,ecx   
   not ecx   
   and ecx,ebx   
   or eax,ecx   
  
   add eax,dta   
   add eax,x   
   add eax,t   
  
   mov cl,s   
   rol eax,cl   
  
   add eax,dtb   
  
   mov dta,eax   
  
ENDM   
  
  
HH   MACRO dta,dtb,dtc,dtd,x,s,t    ; a = b + ((a + H(b,c,d) + x + t) << s)   
  
   mov eax,dtb   
   mov ebx,dtc   
   mov ecx,dtd   
  
   ; H(x,y,z) = x xor y xor z   
  
   xor eax,ebx   
   xor eax,ecx   
  
   add eax,dta   
   add eax,x   
   add eax,t   
  
   mov cl,s   
   rol eax,cl   
  
   add eax,dtb   
  
   mov dta,eax   
  
ENDM   
  
  
II   MACRO dta,dtb,dtc,dtd,x,s,t    ; a = b + ((a + I(b,c,d) + x + t) << s)   
  
   mov eax,dtb   
   mov ebx,dtc   
   mov ecx,dtd   
  
   ; I(x,y,z) = y xor (x or (not z))   
  
   not ecx   
   or eax,ecx   
   xor eax,ebx   
  
   add eax,dta   
   add eax,x   
   add eax,t   
  
   mov cl,s   
   rol eax,cl   
  
   add eax,dtb   
  
   mov dta,eax   
  
ENDM   
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; MD5hash process: hashes a string using the md5 algorithm              \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
MD5HASH\_Process  proc uses eax ebx ecx edx edi esi,   
                      IptBuffer:dword,IptBufferLength:dword,OptMD5Result:dword   
   Local dta:dword,   
         dtb:dword,   
         dtc:dword,   
         dtd:dword   
  
  
   ; phase I &¤ padding   
   mov edi,IptBuffer   
   mov eax,IptBufferLength   
  
   inc eax   
   add edi,eax   
   mov byte ptr \[edi-1\],080h   
  
   xor edx,edx   
  
   mov ebx,64   
   div ebx   
  
   neg edx   
   add edx,64   
  
   cmp edx,8   
   jae Loop1   
  
   add edx,64   
  
Loop1:   
   mov ecx,edx   
   xor al,al   
   rep stosb   
  
   mov eax,dtBufferLength   
  
   inc edx   
   add IptBufferLength,edx   
  
   xor edx,edx   
  
   mov ebx,8   
   mul ebx   
  
   mov dword ptr \[edi-8\],eax   
   mov dword ptr \[edi-4\],edx   
  
   mov edx,IptBufferLength   
  
   mov edi,IptBuffer   
  
  
   ; phase II &¤ chaining variables initialization   
   mov esi,OptMD5Result   
   assume esi:ptr MD5RESULT   
  
   mov \[esi\].dtA,08b562301h   
   mov \[esi\].dtB,0facdab89h   
   mov \[esi\].dtC,098badcafh   
   mov \[esi\].dtD,0103265b8h   
  
   ; phase III &¤ hashing   
hashloop:     
  
   mov eax,\[esi\].dtA   
   mov dta,eax   
   mov eax,\[esi\].dtB   
   mov dtb,eax   
   mov eax,\[esi\].dtC   
   mov dtc,eax   
   mov eax,\[esi\].dtD   
   mov dtd,eax   
  
   ; round 1   
   FF dta,dtb,dtc,dtd,dword ptr \[edi+00\*4\],07,0d76aa478h   
   FF dtd,dta,dtb,dtc,dword ptr \[edi+01\*4\],12,0e8c7b756h   
   FF dtc,dtd,dta,dtb,dword ptr \[edi+02\*4\],17,0242070dbh   
   FF dtb,dtc,dtd,dta,dword ptr \[edi+03\*4\],22,0c1bdceeeh   
   FF dta,dtb,dtc,dtd,dword ptr \[edi+04\*4\],07,0f57c0fafh   
   FF dtd,dta,dtb,dtc,dword ptr \[edi+05\*4\],12,04787c62ah   
   FF dtc,dtd,dta,dtb,dword ptr \[edi+06\*4\],17,0a8304613h   
   FF dtb,dtc,dtd,dta,dword ptr \[edi+07\*4\],22,0fd469501h   
   FF dta,dtb,dtc,dtd,dword ptr \[edi+08\*4\],07,0698098d8h   
   FF dtd,dta,dtb,dtc,dword ptr \[edi+09\*4\],12,08b44f7afh   
   FF dtc,dtd,dta,dtb,dword ptr \[edi+10\*4\],17,0ffff5bb1h   
   FF dtb,dtc,dtd,dta,dword ptr \[edi+11\*4\],22,0895cd7beh   
   FF dta,dtb,dtc,dtd,dword ptr \[edi+12\*4\],07,06b901122h   
   FF dtd,dta,dtb,dtc,dword ptr \[edi+13\*4\],12,0fd987193h   
   FF dtc,dtd,dta,dtb,dword ptr \[edi+14\*4\],17,0a679438eh   
   FF dtb,dtc,dtd,dta,dword ptr \[edi+15\*4\],22,049b40821h   
  
   ; round 2   
   GG dta,dtb,dtc,dtd,dword ptr \[edi+01\*4\],05,0f61e2562h   
   GG dtd,dta,dtb,dtc,dword ptr \[edi+06\*4\],09,0c040b340h   
   GG dtc,dtd,dta,dtb,dword ptr \[edi+11\*4\],14,0265e5a51h   
   GG dtb,dtc,dtd,dta,dword ptr \[edi+00\*4\],20,0e9b6c7aah   
   GG dta,dtb,dtc,dtd,dword ptr \[edi+05\*4\],05,0d62f105dh   
   GG dtd,dta,dtb,dtc,dword ptr \[edi+10\*4\],09,002441453h   
   GG dtc,dtd,dta,dtb,dword ptr \[edi+15\*4\],14,0d8a1e681h   
   GG dtb,dtc,dtd,dta,dword ptr \[edi+04\*4\],20,0e7d3fbc8h   
   GG dta,dtb,dtc,dtd,dword ptr \[edi+09\*4\],05,021e1cde6h   
   GG dtd,dta,dtb,dtc,dword ptr \[edi+14\*4\],09,0c33707d6h   
   GG dtc,dtd,dta,dtb,dword ptr \[edi+03\*4\],14,0f4d50d87h   
   GG dtb,dtc,dtd,dta,dword ptr \[edi+08\*4\],20,0455a14edh   
   GG dta,dtb,dtc,dtd,dword ptr \[edi+13\*4\],05,0a9e3e905h   
   GG dtd,dta,dtb,dtc,dword ptr \[edi+02\*4\],09,0fcefa3f8h   
   GG dtc,dtd,dta,dtb,dword ptr \[edi+07\*4\],14,0676f02d9h   
   GG dtb,dtc,dtd,dta,dword ptr \[edi+12\*4\],20,08d2a4c8ah   
  
   ; round 3   
   HH dta,dtb,dtc,dtd,dword ptr \[edi+05\*4\],04,0fffa3942h   
   HH dtd,dta,dtb,dtc,dword ptr \[edi+08\*4\],11,08771f681h   
   HH dtc,dtd,dta,dtb,dword ptr \[edi+11\*4\],16,06d9d6122h   
   HH dtb,dtc,dtd,dta,dword ptr \[edi+14\*4\],23,0fde5380ch   
   HH dta,dtb,dtc,dtd,dword ptr \[edi+01\*4\],04,0a4beea44h   
   HH dtd,dta,dtb,dtc,dword ptr \[edi+04\*4\],11,04bdecfa9h   
   HH dtc,dtd,dta,dtb,dword ptr \[edi+07\*4\],16,0f6bb4b60h   
   HH dtb,dtc,dtd,dta,dword ptr \[edi+10\*4\],23,0bebfbc70h   
   HH dta,dtb,dtc,dtd,dword ptr \[edi+13\*4\],04,0289b7ec6h   
   HH dtd,dta,dtb,dtc,dword ptr \[edi+00\*4\],11,0eaa127fah   
   HH dtc,dtd,dta,dtb,dword ptr \[edi+03\*4\],16,0d4ef3085h   
   HH dtb,dtc,dtd,dta,dword ptr \[edi+06\*4\],23,004881d05h   
   HH dta,dtb,dtc,dtd,dword ptr \[edi+09\*4\],04,0d9d4d039h   
   HH dtd,dta,dtb,dtc,dword ptr \[edi+12\*4\],11,0e6db99e5h   
   HH dtc,dtd,dta,dtb,dword ptr \[edi+15\*4\],16,01fa27cf8h   
   HH dtb,dtc,dtd,dta,dword ptr \[edi+02\*4\],23,0c4ac5665h   
  
   ; round 4   
   II dta,dtb,dtc,dtd,dword ptr \[edi+00\*4\],06,0f4292244h   
   II dtd,dta,dtb,dtc,dword ptr \[edi+07\*4\],10,0432aff97h   
   II dtc,dtd,dta,dtb,dword ptr \[edi+14\*4\],15,0ab9423a7h   
   II dtb,dtc,dtd,dta,dword ptr \[edi+05\*4\],21,0fc93a039h   
   II dta,dtb,dtc,dtd,dword ptr \[edi+12\*4\],06,0655b59c3h   
   II dtd,dta,dtb,dtc,dword ptr \[edi+03\*4\],10,08f0ccc92h   
   II dtc,dtd,dta,dtb,dword ptr \[edi+10\*4\],15,0ffeff47dh   
   II dtb,dtc,dtd,dta,dword ptr \[edi+01\*4\],21,085845dd1h   
   II dta,dtb,dtc,dtd,dword ptr \[edi+08\*4\],06,06fa87e4fh   
   II dtd,dta,dtb,dtc,dword ptr \[edi+15\*4\],10,0fe2ce6e0h   
   II dtc,dtd,dta,dtb,dword ptr \[edi+06\*4\],15,0a3014314h   
   II dtb,dtc,dtd,dta,dword ptr \[edi+13\*4\],21,04e0811a1h   
   II dta,dtb,dtc,dtd,dword ptr \[edi+04\*4\],06,0f7537e82h   
   II dtd,dta,dtb,dtc,dword ptr \[edi+11\*4\],10,0bd3af235h   
   II dtc,dtd,dta,dtb,dword ptr \[edi+02\*4\],15,02ad7d2bbh   
   II dtb,dtc,dtd,dta,dword ptr \[edi+09\*4\],21,0eb86d391h   
  
   mov eax,dta   
   add \[esi\].dtA,eax   
   mov eax,dtb   
   add \[esi\].dtB,eax   
   mov eax,dtc   
   add \[esi\].dtC,eax   
   mov eax,dtd   
   add \[esi\].dtD,eax   
  
   add edi,64   
  
   sub edx,64   
   jnz hashloop   
  
  
   ; phase IV &¤ results   
  
   mov ecx,4   
  
  
R5:      
   mov  eax,dword ptr \[esi\]   
   xchg al,ah   
   rol  eax,16   
   xchg al,ah   
   mov  dword ptr \[esi\],eax   
  
   add  esi,4   
   loop R5   
  
   ret   
  
MD5HASH\_Process  endp   
  
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; decrypt function encryption word                                      \*   
;                                                                       \*   
; change 4 function encryption word user inputted from asc to hex       \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
Decryption\_Word\_Process proc   
  
    mov cx,4   
  
    mov esi,offset StringBuffer+16+2   
    mov edi,offset Decryption\_Word1+1   
    call DW\_1   
    mov esi,offset StringBuffer+16   
    mov edi,offset Decryption\_Word1   
    call DW\_1   
  
    mov esi,offset StringBuffer+16+6   
    mov edi,offset Decryption\_Word1+3   
    call DW\_1   
    mov esi,offset StringBuffer+16+4   
    mov edi,offset Decryption\_Word1+2   
    call DW\_1   
  
    mov esi,offset StringBuffer+16+10   
    mov edi,offset Decryption\_Word1+5   
    call DW\_1   
    mov esi,offset StringBuffer+16+8   
    mov edi,offset Decryption\_Word1+4   
    call DW\_1   
  
    mov esi,offset StringBuffer+16+14   
    mov edi,offset Decryption\_Word1+7   
    call DW\_1   
    mov esi,offset StringBuffer+16+12   
    mov edi,offset Decryption\_Word1+6   
    call DW\_1   
    
  
    mov esi,offset MD5HASH\_RESULT+8   
    mov edi,offset Decryption\_Word1   
    mov ax, word ptr \[esi\]      
    xor word ptr\[edi\],ax   
    mov ax,word ptr \[edi\]   
    xchg ah,al   
    mov word ptr \[edi\],ax   
    mov edi,offset Decryption\_Word2   
    mov ax, word ptr \[esi+2\]      
    xor word ptr\[edi\],ax   
    mov ax,word ptr \[edi\]   
    xchg ah,al   
    mov word ptr \[edi\],ax   
    mov edi,offset Decryption\_Word3   
    mov ax, word ptr \[esi+4\]      
    xor word ptr\[edi\],ax   
    mov ax,word ptr \[edi\]   
    xchg ah,al   
    mov word ptr \[edi\],ax   
    mov edi,offset Decryption\_Word4   
    mov ax, word ptr \[esi+6\]      
    xor word ptr\[edi\],ax   
    mov ax,word ptr \[edi\]   
    xchg ah,al   
    mov word ptr \[edi\],ax   
    jmp DW\_END   
           
DW\_1:                       ;ASCII to HEX  : convert 2 bytes ASCII to ax register WORD   
    xor bx,bx   
    mov al,byte ptr \[esi\]   
    sub al,30h              ;obtain ASCII 0-9   
    cmp al,10h   
    jb  DW\_2   
    sub al,07h              ;obtain ASCII A-F   
DW\_2:   
    mov bl,al   
    shl bl,cl   
    mov al,byte ptr \[esi+1\]   
    sub al,30h   
    cmp al,10h   
    jb  DW\_3   
    sub al,07h              ;obtain ASCII A-F   
DW\_3:   
    or  bl,al   
    mov byte ptr \[edi\],bl   
    ret   
  
DW\_END:   
  
; put 3 decryption word to key for TEA decryption   
; 3 decryption is  d33f  2212 efa8 ==> Tea\_Key                                        
  
  mov edi,offset Tea\_Key   
  mov esi,offset Decryption\_Word4   
  mov ax,word ptr \[esi\]   
  mov word ptr \[edi\],ax   
  mov ax,word ptr \[esi-2\]   
  mov word ptr \[edi+2\],ax   
  mov ax,word ptr \[esi-4\]   
  mov word ptr \[edi+4\],ax   
    ret   
           
Decryption\_Word\_Process endp   
  
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Decrypting Process                                                    \*   
;                                                                       \*   
; Loop decrypt object input buffer length must be a multiple of 8       \*   
;                                                                       \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
Decryption\_Process Proc IptBuffer:dword,IptBufferLength:dword, Decryption\_Word\_Address:dword, OptBuffer:dword   
  
    Local DECRYPT\_WORD:dword   
    
    MOV ESI,Decryption\_Word\_Address   
    MOV AX,WORD PTR \[ESI\]   
    MOV WORD PTR DECRYPT\_WORD,AX   
  MOV  ESI,IptBuffer   
  MOV  EDI,OptBuffer   
     
Decryption\_P1:   
  
  CLD   
  LODSW   
  PUSH  AX   
  LODSW   
  MOV    BX,AX   
  LODSW   
  MOV    CX,AX   
  LODSW   
  MOV    DX,AX   
  POP    AX   
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; decrypt codes process                     \*   
;                                           \*   
; annotated instructions are used for encrypt process    \*   
;                                           \*   
;                                           \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
  
;  XOR  AX,WORD PTR DECRYPT\_WORD   
;  XOR  BX,WORD PTR DECRYPT\_WORD   
;  XOR  CX,WORD PTR DECRYPT\_WORD   
;  XOR  DX,WORD PTR DECRYPT\_WORD   
  
;  XCHG  AH,DL   
;  XCHG  AL,DH   
;  XCHG  BH,CL   
;  XCHG  BL,CH   
  
  XCHG  AH,DL                          
  XCHG  AL,DH                          
  XCHG  BH,CL                          
  XCHG  BL,CH                          
                                           
  XOR  DX,WORD PTR DECRYPT\_WORD           
  XOR  CX,WORD PTR DECRYPT\_WORD           
  XOR  BX,WORD PTR DECRYPT\_WORD           
  XOR  AX,WORD PTR DECRYPT\_WORD           
  
  
  PUSH  AX   
  MOV  AX,DX   
  STOSW   
  MOV  AX,CX   
  STOSW   
  MOV  AX,BX   
  STOSW   
  POP  AX   
  STOSW   
  
  DEC IptBufferLength   
    JNZ Decryption\_P1   
  
    RET   
  
Decryption\_Process    ENDP   
       
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Encrypting Process                                                    \*   
;                                                                       \*   
; Loop encrypt object input buffer length must be a multiple of 8       \*   
;                                                                       \*   
; this process is a reverse process relative to decryption process      \*   
;                                                                       \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
Encryption\_Process Proc IptBuffer:dword,IptBufferLength:dword, Encryption\_Word\_Address:dword, OptBuffer:dword   
  
    Local ENCRYPT\_WORD:word            ;Encrypt\_Word equal Decrypt\_Word   
    
    MOV ESI,Encryption\_Word\_Address   
    MOV AX,WORD PTR \[ESI\]   
    MOV WORD PTR ENCRYPT\_WORD,AX   
  MOV  ESI,IptBuffer   
  MOV  EDI,OptBuffer   
     
Encryption\_P1:   
  
  CLD   
  LODSW   
  PUSH  AX   
  LODSW   
  MOV    BX,AX   
  LODSW   
  MOV    CX,AX   
  LODSW   
  MOV    DX,AX   
  POP    AX   
  
  XOR  AX,WORD PTR ENCRYPT\_WORD   
  XOR  BX,WORD PTR ENCRYPT\_WORD   
  XOR  CX,WORD PTR ENCRYPT\_WORD   
  XOR  DX,WORD PTR ENCRYPT\_WORD   
  
  XCHG  AH,DL                          
  XCHG  AL,DH                          
  XCHG  BH,CL                          
  XCHG  BL,CH                          
                                           
  PUSH  AX   
  MOV  AX,DX   
  STOSW   
  MOV  AX,CX   
  STOSW   
  MOV  AX,BX   
  STOSW   
  POP  AX   
  STOSW   
  
  DEC IptBufferLength   
    JNZ Encryption\_P1   
  
    RET   
  
Encryption\_Process    ENDP   
       
  
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Check\_Process                                                         \*   
;                                                                       \*   
; check process if 12 bytes is equal between test and object buffer     \*   
; return ex=1 else ex,0                                                 \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
Check\_Process Proc TestBuffer:dword,TestLength:dword, ObjectBuffer:dword   
  
       
      mov ecx,TestLength   
      mov esi, TestBuffer   
      mov edi, ObjectBuffer   
      repz cmpsb   
      mov eax,0   
      jnz Check\_Error   
        
      mov eax,1   
      jmp Check\_End   
  
Check\_Error:   
      xor eax,eax   
         
Check\_End:             
      ret   
  
Check\_Process endp   
  
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Judgement Process                                                     \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
Judge\_Process Proc hWnd :DWORD,Ipt\_Buffer:dword, Ipt\_Info\_Length:dword, Ipt\_Decryption\_Word\_Address:dword, Opt\_Buffer:dword   
  
    LOCAL Source\_Address:dword   
    LOCAL Source\_Length:dword   
    LOCAL Decrypt\_Word\_Address:dword   
    LOCAL Object\_Address:dword   
  LOCAL Save\_Temp\_Value:word   
  LOCAL Save\_Temp\_Value2:dword   
       
    mov esi, dword ptr Ipt\_Decryption\_Word\_Address   
    mov ax,word ptr \[esi\]   
    mov Current\_Decrypt\_Word, ax   
       
    invoke Decryption\_Process, Ipt\_Buffer, Ipt\_Info\_Length, Addr Current\_Decrypt\_Word, Opt\_Buffer   
    invoke Check\_Process, ADDR Test\_Success\_Info,12, Ipt\_Buffer   
    mov byte  ptr \[Success\_Info\_Status\],0   
    cmp eax,1   
    jnz Judge\_Next   
       
; magic number 1 :d33f xor 2212 xor efa8 = 1e85   
; magic number 2 :d33f sub 2212 xor efa8 = 5e85   
; magic number 2-1 (ah xchg al)= 40   
; magic number 3 :efa8 sub 2212 xor d33f = 1ea9   
  pushad   
    mov esi, offset Decryption\_Block\_Cipher    ; equal Tea\_Key   
    xor eax,eax   
    mov ax,word ptr \[esi\]            ;   
    xor ax,word ptr\[esi+2\]            ;   
    xor ax,word ptr\[esi+4\]            ; eax=1e85   
    xchg ah,al   
    mov word ptr Save\_Temp\_Value,ax        ; eax=851e   
    mov ax,word ptr \[esi\]   
    sub ax,word ptr\[esi+2\]            ;   
    xor ax,word ptr\[esi+4\]            ; eax=5e85   
    xchg ah,al                  ; eax=855e   
     sub ax,Save\_Temp\_Value            ; eax=40   
  mov esi,offset Jump\_Address\_Matrix-40h+Exit\_Address   
  add eax,esi   
  mov eax,\[eax\]   
    mov dword ptr Save\_Temp\_Value2,eax       ; save exit process address   
  mov esi,offset Jump\_Address\_Matrix+Tea\_Encrypt\_Address-0851eh   
  xchg esi,eax   
  xor  esi,esi   
  mov  si,Save\_Temp\_Value   
  add eax,esi   
     call dword ptr \[eax\]             ;call Tea\_Decrypt\_Process   
    mov esi, offset Decryption\_Block\_Cipher    ; equal Tea\_Key   
    xor eax,eax   
    mov ax,word ptr \[esi+4\]            ;   
    sub ax,word ptr\[esi+2\]            ;   
    xor ax,word ptr\[esi\]            ; eax=1ea9   
  mov esi,offset Jump\_Address\_Matrix+1ea9h+Block\_Decrypt\_Address   
  sub esi,eax   
    call dword ptr \[esi\]            ;Call Block\_Decrypt\_Process   
    popad   
    mov eax,Ipt\_Buffer   
    mov dword ptr \[Success\_Info\_Address\],eax   
    mov byte  ptr \[Success\_Info\_Status\],1       ;set success starus    
    mov ax,word ptr \[Current\_Decrypt\_Word\]   
    mov word  ptr \[Success\_Decryption\_Word\],ax   
    invoke MessageBox,hWnd,ADDR Finial\_Success\_Info,   
           ADDR Password\_Name,MB\_OK or MB\_TASKMODAL   
    invoke Encryption\_Process, Ipt\_Buffer, Ipt\_Info\_Length, Addr Current\_Decrypt\_Word, Opt\_Buffer   
  push dword ptr Save\_Temp\_Value2   
  pop  eax   
  call dword ptr eax                 ; exit process   
    mov eax,1   
    jmp Judge\_End   
  
Judge\_Next:   
    invoke Encryption\_Process, Ipt\_Buffer, Ipt\_Info\_Length, Addr Current\_Decrypt\_Word, Opt\_Buffer   
  
    xor eax,eax   
  
Judge\_End:   
  
    ret   
  
  
Judge\_Process endp   
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Function incalid              \*   
;                  \*   
; chaos codes                \*   
;                   \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
xBlock\_Decrypt\_Process proc    
  
   LOCAL xDecrypt\_Word:word   
  
  
  mov  esi,offset Finial\_Success\_Info                
  mov  edi,offset Decryption\_Block\_Cipher   
  mov ax,word ptr \[edi\]   
  mov word ptr xDecrypt\_Word,ax   
  mov ecx,Decryption\_Block\_Length/2  ;input string byte length convert to word length    
  xor ebx,ebx               
xdb\_2:                            
  mov  ax,word ptr \[esi\]                         
  xor  ax,word ptr xDecrypt\_Word         
  xchg  ah,al            
  add  ax,word ptr \[edi+ebx\]                
  add  ebx,2                                
  cmp  ebx,08h      ;wrong number                              
  jbe  xdb\_3                                    
  xor  ebx,ebx                                
xdb\_3:                                    
  mov  word ptr \[esi\],ax                              
  add  esi,2                                
  loop  xdb\_2       
  ret   
  
xBlock\_Decrypt\_Process endp   
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Function 1   
;   
; decrypt an encrypted block with method 1   
;    
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
Block\_Decrypt\_Process proc    
  
   LOCAL Decrypt\_Word:word   
  
  
  mov  esi,offset Finial\_Success\_Info                
  mov  edi,offset Decryption\_Block\_Cipher   
  mov ax,word ptr \[edi\]   
  mov word ptr Decrypt\_Word,ax   
  mov ecx,Decryption\_Block\_Length/2  ;input string byte length convert to word length    
  xor ebx,ebx               
db\_2:                            
  mov  ax,word ptr \[esi\]                         
  xor  ax,word ptr Decrypt\_Word         
  xchg  ah,al            
  add  ax,word ptr \[edi+ebx\]                
  add  ebx,2                                
  cmp  ebx,0eh      ;encrypted block encryption length  8 word                              
  jbe  db\_3                                    
  xor  ebx,ebx                                
db\_3:                                    
  mov  word ptr \[esi\],ax                              
  add  esi,2                                
  loop  db\_2       
  ret   
  
Block\_Decrypt\_Process endp   
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Function 2   
;   
; encrypt an encrypted block with method 1   
;    
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
Block\_Encrypt\_Process proc    
  
   LOCAL Encrypt\_Word:word   
  
IF INITIALIZE\_ENCRYPTION\_INFORMATION   
  
    mov esi,offset Finial\_Success\_Info   
    mov edi,offset Tea\_Encrypt\_Key   
  jmp eb\_1   
     
ENDIF   
     
    mov esi,offset Finial\_Success\_Info   
    mov edi,offset Decryption\_Block\_Cipher   
       
eb\_1:       
  mov ax,word ptr \[edi\]   
  mov word ptr Encrypt\_Word,ax   
  mov ecx,Decryption\_Block\_Length/2    ;input string byte length convert to word length    
  xor ebx,ebx               
eb\_2:                            
  mov  ax,word ptr \[esi\]                         
  sub  ax,word ptr \[edi+ebx\]                
  xchg  ah,al            
  xor  ax,word ptr Encrypt\_Word         
  add  ebx,2                                
  cmp  ebx,0eh    ;encrypted block encryption length = 8 word    
  jbe  eb\_3                                    
  xor  ebx,ebx                                
eb\_3:                                    
  mov  word ptr \[esi\],ax                              
  add  esi,2                                
  loop  eb\_2       
  ret   
Block\_Encrypt\_Process endp   
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Function incalid                            \*   
;                                    \*   
; chaos codes                              \*   
;                                     \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
xBlock\_Encrypt\_Process proc    
  
   LOCAL xEncrypt\_Word:word   
  
    mov esi,offset Finial\_Success\_Info   
    mov edi,offset Decryption\_Block\_Cipher   
       
xeb\_1:       
  mov ax,word ptr \[edi\]   
  mov word ptr xEncrypt\_Word,ax   
  mov ecx,Decryption\_Block\_Length/2    ;input string byte length convert to word length    
  xor ebx,ebx               
xeb\_2:                            
  mov  ax,word ptr \[esi\]                         
  sub  ax,word ptr \[edi+ebx\]                
  xchg  ah,al            
  xor  ax,word ptr xEncrypt\_Word         
  add  ebx,2                                
  cmp  ebx,08h    ; chaos number    
  jbe  xeb\_3                                    
  xor  ebx,ebx                                
xeb\_3:                                    
  mov  word ptr \[esi\],ax                              
  add  esi,2                                
  loop  xeb\_2       
  ret   
xBlock\_Encrypt\_Process endp   
  
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Function 4   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Tiny Encryption Algorithm   
; A public domain block cipher    
; TEA uses a 128 bit key and operates on 64 bit data blocks   
;Key:   
;        db      "0000","0000","0000","0000"   
;Data:   
;        db      "0000","0000"   
;   
;sum     equ     eax   
;y       equ     ebx   
;z       equ     ecx   
;delta   equ     edx   
;rounds  equ     di   
;t       equ     ebp   
;v0      equ     dword ptr \[edi\]   
;v1      equ     dword ptr \[edi+4\]   
;k0      equ     dword ptr \[esi\]   
;k1      equ     dword ptr \[esi+4\]   
;k2      equ     dword ptr \[esi+8\]   
;k3      equ     dword ptr \[esi+12\]   
  
Tea\_Encrypt\_Process proc   
  
IF INITIALIZE\_ENCRYPTION\_INFORMATION   
  
        mov     esi,offset Tea\_Encrypt\_Key   
        mov     edi,offset Tea\_Encrypt\_Data   
        call    Encrypt   
        ret   
ENDIF   
  
  
        mov     esi,offset Tea\_Key   
        mov     edi,offset Tea\_Data   
        call    Encrypt   
        ret   
  
Encrypt:   
        push    edi   
        mov     y,v0   
        mov     z,v1   
        xor     sum,sum   
        mov     delta,9e3779b9h ; sqr(5)-1 \* 2³¹   
        mov     rounds,32   
ELoopR:   
        add     sum,delta   
        mov     t,z   
        shl     t,4   
        add     y,t   
        mov     t,k0   
        xor     t,z   
        add     y,t   
        mov     t,z   
        shr     t,5   
        xor     t,sum   
        add     y,t   
        add     y,k1   
        ;   
        mov     t,y   
        shl     t,4   
        add     z,t   
        mov     t,k2   
        xor     t,y   
        add     z,t   
        mov     t,y   
        shr     t,5   
        xor     t,sum   
        add     z,t   
        add     z,k3   
        dec     rounds   
        jnz     ELoopR   
  
        pop     edi   
        mov     v0,y   
        mov     v1,z   
        ret   
  
Tea\_Encrypt\_Process endp   
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
;invalid function chaos codes   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
xTea\_Encrypt\_Process proc   
  
        mov     esi,offset Tea\_Key   
        mov     edi,offset Tea\_Data   
        call    xEncrypt   
        ret   
  
xEncrypt:   
        push    edi   
        mov     y,v0   
        mov     z,v1   
        xor     sum,sum   
        mov     delta,9e3779b9h-1045h ; sqr(5)-1 \* 2³¹   
        mov     rounds,32   
xELoopR:   
        add     sum,delta   
        mov     t,z   
        shl     t,4   
        add     y,t   
        mov     t,k0   
        xor     t,z   
        add     y,t   
        mov     t,z   
        shr     t,5   
        xor     t,sum   
        add     y,t   
        add     y,k1   
        ;   
        mov     t,y   
        shl     t,4   
        add     z,t   
        mov     t,k2   
        xor     t,y   
        add     z,t   
        mov     t,y   
        shr     t,4   
        xor     t,sum   
        add     z,t   
        add     z,k3   
        dec     rounds   
        jnz     xELoopR   
  
        pop     edi   
        mov     v0,y   
        mov     v1,z   
        ret   
  
xTea\_Encrypt\_Process endp   
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Function 5   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Tiny Decryption Algorithm   
; A public domain block cipher    
; TEA uses a 128 bit key and operates on 64 bit data blocks   
  
Tea\_Decrypt\_Process proc   
  
Tea\_001:           
        mov     esi,offset Tea\_Key   
        mov     edi,offset Tea\_Data   
        call    Decrypt   
        ret   
  
Decrypt:   
        push    edi   
        mov     y,v0   
        mov     z,v1   
        mov     delta,9e3779b9h ; sqr(5)-1 \* 2³¹   
        mov     sum,delta   
        shl     sum,5   
        mov     rounds,32   
DLoopR:   
        mov     t,y   
        shl     t,4   
        sub     z,t   
        mov     t,k2   
        xor     t,y   
        sub     z,t   
        mov     t,y   
        shr     t,5   
        xor     t,sum   
        sub     z,t   
        sub     z,k3   
        ;   
        mov     t,z   
        shl     t,4   
        sub     y,t   
        mov     t,k0   
        xor     t,z   
        sub     y,t   
        mov     t,z   
        shr     t,5   
        xor     t,sum   
        sub     y,t   
        sub     y,k1   
        sub     sum,delta   
        dec     rounds   
        jnz     DLoopR   
  
        pop     edi   
        mov     v0,y   
        mov     v1,z   
        ret   
  
  
Tea\_Decrypt\_Process endp   
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Function 6 INVALID   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
xTea\_Decrypt\_Process proc   
  
xTea\_001:           
        mov     esi,offset Tea\_Key   
        mov     edi,offset Tea\_Data   
        call    xDecrypt   
        ret   
  
xDecrypt:   
        push    edi   
        mov     y,v0   
        mov     z,v1   
        mov     delta,9e3779b9h-1045h ; sqr(5)-1 \* 2³¹   
        mov     sum,delta   
        shl     sum,4   
        mov     rounds,32   
xDLoopR:   
        mov     t,y   
        shl     t,4   
        sub     z,t   
        mov     t,k2   
        xor     t,y   
        sub     z,t   
        mov     t,y   
        shr     t,5   
        xor     t,sum   
        sub     z,t   
        sub     z,k3   
        ;   
        mov     t,z   
        shl     t,4   
        sub     y,t   
        mov     t,k0   
        xor     t,z   
        sub     y,t   
        mov     t,z   
        shr     t,4   
        xor     t,sum   
        sub     y,t   
        sub     y,k1   
        sub     sum,delta   
        dec     rounds   
        jnz     xDLoopR   
  
        pop     edi   
        mov     v0,y   
        mov     v1,z   
        ret   
  
  
xTea\_Decrypt\_Process endp   
  
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
; Function 7 jmup to exit process          \*   
;                  \*   
; note: this function is invalid, just provde a jump address     \*   
;       point to exit process for aother program        \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
Function\_Jump\_Exit\_Process proc   
  
F\_7:   
  cmp eax,347fh   
  jnz chaos\_001   
  sub eax,0321h   
  xor eax,0dfa2h   
  jmp dword ptr \[eax\]   
  db 066h   
   invoke ExitProcess,eax   
chaos\_001:      
  xor eax,0993ah   
  jmp dword ptr \[eax\]     
   jmp F\_7\_2   
   invoke ExitProcess,eax   
  jmp F\_7\_2   
  invoke ExitProcess,eax   
  db 0e6h   
     
Function\_Jump\_Exit\_Address equ $   
  
  invoke ExitProcess,eax   
  db 45h   
  jmp F\_7   
  invoke ExitProcess,037h   
  jmp F\_7   
  invoke ExitProcess,eax   
  jmp F\_7   
  db 0eah   
F\_7\_2:   
  ret   
      
Function\_Jump\_Exit\_Process endp   
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
;\* End Program                                                          \*   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
  
  
  
end start   
  
  
;---------------------------------------------------------------------  
          
 4dd466d0e1e0cc123afbe504da2462ca
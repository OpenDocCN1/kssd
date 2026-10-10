

# Mover98 3.1

> 原文：[`www.pediy.com/kssd/pediy03/forum731.htm`](https://www.pediy.com/kssd/pediy03/forum731.htm)

目的：破解 Mover98 3.1 的自校验（初学）   
工具：W32Dasm 8.93   
方法：瞎猫撞上死耗子   
程序介绍：Mover98 是专门给软件搬家的工具，而不会影响程序的运行。电脑报上有介绍，我就不罗嗦了。   
前言：事实上，我并不是诚心要跟它过不去，我是个做汉化的，这么好的软件我当然不能放过。下载、安装、注册之后，先开始汉化“关于”对话框（这是我的惯例，也是先验证一下能否汉化，因为我经常碰到不能汉化的情况），然后退出运行程序，鼠标指针晃了两下就没动静了，再改回来，程序正常运行。哦，原来有自校验！   
分析：软件的这种自校验肯定是在程序开始运行的时候就要比较的，所以这段代码一定在程序的入口不远处。   
用 W32Dasm 反汇编出代码后，开始瞪大眼睛仔细寻找……   
  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* Program Entry Point \*\*\*\*\*\*\*\*   
:0047DC1C 55                      push ebp   
:0047DC1D 8BEC                    mov ebp, esp   
:0047DC1F 83C4F0                  add esp, FFFFFFF0   
:0047DC22 33C0                    xor eax, eax   
:0047DC24 8945F0                  mov dword ptr \[ebp\-10\], eax   
:0047DC27 B87CD94700              mov eax, 0047D97C   
:0047DC2C E8977FF8FF              call 00405BC8   
:0047DC31 33C0                    xor eax, eax   
:0047DC33 55                      push ebp   
:0047DC34 6817DD4700              push 0047DD17   
:0047DC39 64FF30                  push dword ptr fs:\[eax\]   
:0047DC3C 648920                  mov dword ptr fs:\[eax\], esp   
:0047DC3F 8D55F0                  lea edx, dword ptr \[ebp\-10\]   
:0047DC42 33C0                    xor eax, eax   
:0047DC44 E84B4CF8FF              call 00402894   
:0047DC49 8B45F0                  mov eax, dword ptr \[ebp\-10\]   
:0047DC4C B98B3B0000              mov ecx, 00003B8B   
:0047DC51 33D2                    xor edx, edx   
:0047DC53 E80843FBFF              call 00431F60   
:0047DC58 84C0                    test al, al      <----开始判断   
:0047DC5A 7505                    jne 0047DC61    <----跳过下面这个 call   
:0047DC5C E86F5BF8FF              call 004037D0    <----让它运行你就死定了   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0047DC5A(U)   
|   
:0047DC61 A1D4F34700              mov eax, dword ptr \[0047F3D4\]   
:0047DC66 8B00                    mov eax, dword ptr \[eax\]   
:0047DC68 E82334FBFF              call 00431090   
:0047DC6D 8D55F0                  lea edx, dword ptr \[ebp\-10\]   
:0047DC70 B801000000              mov eax, 00000001   
:0047DC75 E81A4CF8FF              call 00402894   
:0047DC7A 8B45F0                  mov eax, dword ptr \[ebp\-10\]   
  
    纵观上下十几行就只有这一个跳转，肯定就是它了。原来这么简单，只要跳过 call 004037D0 就行了。好办，把 jne 改成 jmp（7505 改成 EB05）。OK!  4dd466d0e1e0cc123afbe504da2462ca
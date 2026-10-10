

# vfp&exe1.70

> 原文：[`www.pediy.com/kssd/pediy03/forum1043.htm`](https://www.pediy.com/kssd/pediy03/forum1043.htm)

vfp&exe1.72C 注册码计算   
  
    据其帮助文件，知道注册方式为 keyfile 保护。keyfile 应在文件安装目录下。   
    所以用 filemon 监视，知道 keyfile 为 vfp&exe.bin，伪造它。   
    经过一番艰苦跟踪后终于知道关键点   
  
:0045640F E8B4150000              call 004579C8   
  
F8 进入，比对关键   
......   
:004579E5 8A27    mov ah, byte ptr \[edi\] <--依次取密码字串“好   
                                          孩子 8 好孩子 8...”   
                                          ascii 码对应为 BA C3   
                                          BA A2 D7 D3 38....   
:004579E7 8A03    mov al, byte ptr \[ebx\] <--依次取 vfp&exe.bin 字串 ascii 码   
:004579E9 80F800  cmp al, 00   
:004579EC 7406    je 004579F4   
:004579EE 38E0    cmp al, ah   
:004579F0 7402    je 004579F4   
:004579F2 30E0    xor al, ah            <--两者异或   
:004579F4 8806    mov byte ptr \[esi\], al <--值存入 ptr\[ebx\]   
:004579F6 43      inc ebx   
:004579F7 46      inc esi   
:004579F8 47      inc edi   
:004579F9 80FC38  cmp ah, 38   
:004579FC 7502    jne 00457A00   
:004579FE 89D7    mov edi, edx   
:00457A00 E2E3    loop 004579E5          <--循环   
......   
  
    由动态跟踪知道，   
1.当 xor 后 dword ptr \[006EFBF4\]==机器码，则注册成功。   
2.mov al, byte ptr \[ebx\]  <--ebx 初值为 006EF8B0   
  6EFBF4-6EF8B0+1==837(BCD),   
  所以 VFP&EXE.bin 文件第 837～844 字符对应计算后的机器码。此时的密码字顺序为“A2 D7 D3 38 BA C3 BA A2....”，由 XOR 原理知道：a ＝ (a xor b) xor b   
所以演算如下（我的机器码为 1DDD0D4E）：   
  
机器码 对应 ASCII 码  密码 ascii 码  vfp&exe.bin 字符位置  hex 值   
  1    31            A2          第 837 位          93   
  D    44            D7          第 838 位          93   
  D    44            D3          第 839 位          97   
  D    44            38          第 840 位          7C   
  0    30            BA          第 841 位          8A   
  D    44            C3          第 842 位          87   
  4    34            BA          第 843 位          8E   
  E    45            A2          第 844 位          E7   
  
    这样只要你构建一个 vfp&exe.bin 文件，其字符第 837～844 位依次填入上述 HEX 值，你就拥有注册版了。   
  
    不知谁能写出一个可以自动生成 vfp&exe.bin 文件的注册机来，请一定将源代码提供，最好是 DELPHI 编译的。  4dd466d0e1e0cc123afbe504da2462ca
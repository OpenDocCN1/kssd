

# Windows 优化大师 v2.9+

> 原文：[`www.pediy.com/kssd/pediy02/forum268.htm`](https://www.pediy.com/kssd/pediy02/forum268.htm)

 Windows 优化大师 v2.9+的注册码加密算法   
  
                              written by dr0, 2000/08/22   
  
本文将以汇编代码为例，解释一下“Windows 优化大师” v2.9+中用到的 RSA 算法。   
先简单介绍一下 RSA 算法，便于和汇编代码对照。具体的数学理论我们在此并不太关心，有兴趣的可以去查参考资料。   
  
RSA 算法简述：   
1、取两个素数 p 和 q。   
2、计算 n=pq，f=(p-1)(q-1)。   
3、随机选取整数 e，满足条件 gcd(e, f)=1，其中 gcd 为最大公约数。   
4、计算 d，使得乘积 de 对 f 求余的结果为 1，即 de 和 1 对 f 同余。   
上述只有 e 和 n 对外公开，用于加密。   
  
加密过程（符号^表示乘幂，mod 表示求余）：   
    e 为加密密钥，假定明文为 m，则密文 c = (m ^ e) mod n。   
  
解密过程：   
    d 为解密密钥，则解密得到的明文 m'= (c ^ d) mod n。   
  
RSA 被攻破的一个充分条件是 n 被因子分解，即得到 p 和 q。因为得到 p 和 q 之后便可以计算出 f，从而根据“de 和 1 对 f 同余”这个条件计算出解密密钥 d 来。   
  
根据上面的加密和解密算法可以看出：   
1、RSA 的加密和解密是对称的，即加密和解密可以使用同一个函数；   
2、RSA 加密主要依靠模幂运算，因此这部分运算的复杂度对算法的效率影响最大。关于模幂运算，在《计算机密码学》（清华出版社，卢开澄）一书中讲了一种算法，该算法很容易理解，其指导思想就是不停地降阶，从而降低时间复杂度。用类 C 语言的伪码描述如下：   
  
//下面这个函数计算(m ^ e) mod n   
ReturnValueType  encrypt\_decrypt(m, e, n)   
{   
  LocalVariables a, b, c;   
  
  a = m;   
  b = e;   
  c = 1;   
  
  while(b)   
  {   
    if ((b mod 2) == 0)   
        {   
        b = b / 2;        //降阶   
                a = (a \* a) mod n;   
        }   
        else   
        {   
                b = b - 1;   
                c = (a \* c) mod n;   
        }   
  }          
      
  return c;   
}   
  
Windows 优化大师的破解过程已有很多文章讲过，本文重点分析其对我们输入的注册码进行加密运算的部分，因为只有这一部分求逆的难度比较大些，其它部分可以直接拷贝或者很容易求逆（例如环形移位运算）。我们将会看到，其使用的模幂算法就是上述的算法。   
  
用 IDA 反汇编 OctoDll.dll，可以看到一个名为 Registed( )的函数，这显然是在判断注册码。这个函数将对我们输入的两部分注册码分别进行 RSA 加密，再移位，把移位的结果和一个数（这个数是根据“You are big pig.”、“1234567”以及注册申请码计算出来的，这个计算过程无需求逆）进行比较，然后返回另一个值（它的主程序会判断该值是否为 0x14）。   
  
Registed    proc near   
  
    var\_10      = dword    ptr -10h   
    var\_A        = word    ptr -0Ah   
    var\_8        = dword    ptr -8   
    var\_4        = dword    ptr -4   
    arg\_0        = dword    ptr  8   
    arg\_8        = dword    ptr  10h   
  
        push  ebp   
        mov    ebp, esp   
        add    esp, 0FFFFFFF0h   
        push  ebx   
        xor    edx, edx   
        mov    \[ebp+var\_10\], edx   
        mov    ebx, eax   
        xor    eax, eax   
        push  ebp   
        push  offset loc\_4043FE   
        push  dword ptr fs:\[eax\]   
        mov    fs:\[eax\], esp   
        mov    eax, \[ebp+arg\_8\]   
        mov    \[ebp+var\_8\], eax   
        mov    eax, \[ebp+arg\_0\]   
        mov    \[ebp+var\_4\], eax   
        lea    eax, \[ebp+var\_10\]   
        mov    edx, ebx   
        call  unknown\_libname\_17   
        mov    eax, \[ebp+var\_10\]   
        lea    edx, \[ebp+var\_A\]   
        lea    ecx, \[ebp+var\_8\]   
        call  sub\_4041D8      //核心判断    
        test  eax, eax   
        jnz    short loc\_4043E4   
        xor    ebx, ebx        //bad guy   
        jmp    short loc\_4043E8   
  
loc\_4043E4:   
        movzx  ebx, \[ebp+var\_A\] //return value   
  
loc\_4043E8:   
        xor    eax, eax   
        pop    edx   
        pop    ecx   
        pop    ecx   
        mov    fs:\[eax\], edx   
        push  offset loc\_404405   
  
loc\_4043F5:   
        lea    eax, \[ebp+var\_10\]   
        call  sub\_402E5C   
        retn   
  
很显然，核心判断在 call  sub\_4041D8 中。跟进去，看见函数体如下。这个函数的前半部分是在根据“You are big pig.”、“1234567”以及注册申请码通过哈希运算计算出几个数来。后半部分才是最值得关心的。注释如下：   
  
sub\_4041D8    proc near   
  
var\_24        = dword    ptr -24h   
var\_20        = dword    ptr -20h   
var\_1C        = dword    ptr -1Ch   
var\_18        = dword    ptr -18h   
var\_14        = dword    ptr -14h   
var\_10        = dword    ptr -10h   
var\_C        = dword    ptr -0Ch   
var\_8        = dword    ptr -8   
var\_4        = dword    ptr -4   
  
        push  ebp   
        mov    ebp, esp   
        add    esp, 0FFFFFFDCh   
        push  ebx   
        push  esi   
        push  edi   
        xor    ebx, ebx   
        mov    \[ebp+var\_14\], ebx   
        mov    esi, ecx   
        lea    edi, \[ebp+var\_10\]   
        movsd   
        movsd   
        mov    \[ebp+var\_8\], edx   
        mov    \[ebp+var\_4\], eax   
        mov    eax, \[ebp+var\_4\]   
        call  sub\_402FC0   
        xor    eax, eax   
        push  ebp   
        push  offset loc\_40435F   
        push  dword ptr fs:\[eax\]   
        mov    fs:\[eax\], esp   
        lea    eax, \[ebp+var\_14\]   
        mov    edx, offset \_str\_You\_are\_big\_pig.Text   
        call  sub\_402EC4   
        mov    eax, \[ebp+var\_4\]   
        call  unknown\_libname\_18   
        and    eax, 80000007h   
        jns    short loc\_40422A   
        dec    eax   
        or    eax, 0FFFFFFF8h   
        inc    eax   
  
loc\_40422A:   
        test    eax, eax   
        jz    short loc\_40425A   
        lea    eax, \[ebp+var\_4\]   
        mov    edx, offset \_str\_1234567.Text   
        call    @System@@LStrCat$qqrv ;    System \_\_linkproc\_\_ LStrCat(void)   
        mov    eax, \[ebp+var\_4\]   
        call    unknown\_libname\_18   
        test    eax, eax   
        jns    short loc\_40424A   
        add    eax, 7   
  
loc\_40424A:   
        sar    eax, 3   
        mov    edx, eax   
        shl    edx, 3   
        lea    eax, \[ebp+var\_4\]   
        call    @System@@LStrSetLength$qqrv ; System \_\_linkproc\_\_ LStrSetLength(void)   
  
loc\_40425A:   
        xor    esi, esi   
        lea    eax, \[ebp+var\_4\]   
        call    sub\_402FD0   
        mov    edi, eax   
        lea    eax, \[ebp+var\_14\]   
        call    sub\_402FD0   
        mov    ebx, eax   
        jmp    short loc\_4042A2   
  
loc\_404272:   
        mov    eax, \[edi+esi\*4\]   
        mov    \[ebp+var\_1C\], eax   
        mov    eax, \[edi+esi\*4+4\]   
        mov    \[ebp+var\_18\], eax   
        mov    edx, ebx   
        lea    eax, \[ebp+var\_1C\]   
        call    sub\_404170   
        mov    eax, \[ebx\]   
        mov    \[ebx+8\], eax   
        mov    eax, \[ebx+4\]   
        mov    \[ebx+0Ch\], eax   
        mov    eax, \[ebp+var\_1C\]   
        mov    \[ebx\], eax   
        mov    eax, \[ebp+var\_18\]   
        mov    \[ebx+4\], eax   
        add    esi, 2   
  
loc\_4042A2:   
        mov    eax, \[ebp+var\_4\]   
        call    unknown\_libname\_18   
        test    eax, eax   
        jns    short loc\_4042B1   
        add    eax, 3   
  
loc\_4042B1:   
        sar    eax, 2   
        cmp    esi, eax   
        jb    short loc\_404272  //以上为前半部分，不用求逆    
  
        mov    eax, \[ebp+var\_10\]   
        xor    edx, edx   
        push    edx            //注册码的第一部分 m1 的高位（恒为 0）   
        push    eax            //注册码的第一部分 m1 的低位   
        push    ds:dword\_4050D0 //加密密钥 e 的高位（0）    
        push    ds:dword\_4050CC //加密密钥 e 的低位（3B442AF9）    
        push    ds:dword\_4050D8 //n 的高位（0）   
        push    ds:dword\_4050D4 //n 的低位（69AAA0E3）    
        call    sub\_4040B8      //encrypt\_decrypt(m1, e, n)   
        sub    eax, 2   
        mov    \[ebp+var\_24\], eax   
        mov    eax, \[ebp+var\_C\]   
        xor    edx, edx   
        push    edx              //注册码的第二部分 m2 的高位（恒为 0）   
        push    eax              //注册码的第二部分 m2 的低位   
        push    ds:dword\_4050D0  //加密密钥 e 的高位（0）    
        push    ds:dword\_4050CC  //加密密钥 e 的低位（3B442AF9）    
        push    ds:dword\_4050D8  //n 的高位（0）    
        push    ds:dword\_4050D4  //n 的低位（69AAA0E3）    
        call    sub\_4040B8      //encrypt\_decrypt(m2, e, n)   
        sub    eax, 2   
        mov    \[ebp+var\_20\], eax   
        shl    \[ebp+var\_24\], 2  //移位   
        lea    ecx, \[ebp+var\_24\]   
        mov    eax, \[ecx\]   
        mov    edx, \[ecx+4\]   
        shrd  eax, edx, 2    //这条和下面一条完成 64  bit 环形移位   
        shr    edx, 2   
        mov    \[ecx\], eax   
        mov    \[ecx+4\], edx   
        mov    eax, \[ebp+var\_24\]   
        cmp    eax, \[ebp+var\_1C\] //比较   
        jz    short loc\_404330   
        xor    ebx, ebx   
        jmp    short loc\_404341   
  
loc\_404330:   
        mov    ax, word ptr \[ebp+var\_20\] //返回值    
        and    ax, 0FFFFh   
        mov    edx, \[ebp+var\_8\]   
        mov    \[edx\], ax   
        or    ebx, 0FFFFFFFFh   
  
loc\_404341:   
        xor    eax, eax   
        pop    edx   
        pop    ecx   
        pop    ecx   
        mov    fs:\[eax\], edx   
        push    offset loc\_404366   
  
loc\_40434E:   
        lea    eax, \[ebp+var\_14\]   
        call    sub\_402E5C   
        lea    eax, \[ebp+var\_4\]   
        call    sub\_402E5C   
        retn   
  
即上面的两个 call sub\_4040B8 分别等价于   
      encrypt\_decrypt(m1, e, n)   
      encrypt\_decrypt(m2, e, n)   
只需要把 sub\_4040B8 的函数体和上面的算法伪码描述对照一下就明白为什么等价了（注意 windows 优化大师使用的是 64 bit 的运算，每个数要用两个 32 bit 的数来表示，因此操作每个 64 bit 的数至少要两条指令）：   
  
sub\_4040B8    proc near   
  
var\_8        = dword    ptr -8    //局部变量 c 的低位（c 中将存放返回值）    
var\_4        = dword    ptr -4    //局部变量 c 的高位    
arg\_0        = dword    ptr  8      //m 的低 32 位   
arg\_4        = dword    ptr  0Ch    //m 的高 32 位   
arg\_8        = dword    ptr  10h    //e 的低 32 位   
arg\_C        = dword    ptr  14h    //e 的高 32 位   
arg\_10        = dword    ptr  18h  //n 的低 32 位    
arg\_14        = dword    ptr  1Ch  //n 的高 32 位    
  
        push    ebp   
        mov    ebp, esp   
        add    esp, 0FFFFFFF8h   
        mov    \[ebp+var\_8\], 1    //  c = 1;   
        mov    \[ebp+var\_4\], 0   
        jmp    short loc\_40414A   
  
loc\_4040CE:   
  
        push    0   
        push    2   
        mov    eax, \[ebp+arg\_8\]   
        mov    edx, \[ebp+arg\_C\]   
        call    \_\_LLMOD          //b mod 2   
        cmp    edx, 0   
        jnz    short loc\_40411B   
        cmp    eax, 0   
        jnz    short loc\_40411B   
        push    0   
        push    2   
        mov    eax, \[ebp+arg\_8\]   
        mov    edx, \[ebp+arg\_C\]   
        call    \_\_LLDIV          // b = b / 2   
        mov    \[ebp+arg\_8\], eax   
        mov    \[ebp+arg\_C\], edx   
        push    \[ebp+arg\_14\]   
        push    \[ebp+arg\_10\]   
        push    \[ebp+arg\_14\]   
        push    \[ebp+arg\_10\]   
        push    \[ebp+arg\_4\]   
        push    \[ebp+arg\_0\]   
        call    sub\_404060      // a = (a \* a) mod n   
        mov    \[ebp+arg\_10\], eax   
        mov    \[ebp+arg\_14\], edx   
        jmp    short loc\_40414A   
  
loc\_40411B:   
  
        mov    eax, \[ebp+arg\_8\]   
        mov    edx, \[ebp+arg\_C\]   
        sub    eax, 1            // b = b - 1   
        sbb    edx, 0   
        mov    \[ebp+arg\_8\], eax   
        mov    \[ebp+arg\_C\], edx   
        push    \[ebp+arg\_14\]   
        push    \[ebp+arg\_10\]   
        push    \[ebp+var\_4\]   
        push    \[ebp+var\_8\]   
        push    \[ebp+arg\_4\]   
        push    \[ebp+arg\_0\]   
        call    sub\_404060    //c =  (a \* c) mod n   
        mov    \[ebp+var\_8\], eax   
        mov    \[ebp+var\_4\], edx   
  
loc\_40414A:   
        cmp    \[ebp+arg\_C\], 0    //b 等于 0 吗？    
        jnz    short loc\_40415C   
        cmp    \[ebp+arg\_8\], 0   
        ja    loc\_4040CE        //继续循环   
        jmp    short loc\_404162    
  
loc\_40415C:   
        jg    loc\_4040CE        //继续循环   
  
loc\_404162:   
        mov    eax, \[ebp+var\_8\]  //返回值 c   
        mov    edx, \[ebp+var\_4\]   
        pop    ecx   
        pop    ecx   
        pop    ebp   
        retn    18h   
sub\_4040B8    endp   
  
至此我们已经搞清其加密算法，且得到如下结果：   
e = 0x000000003B442AF9   
n = 0x0000000069AAA0E3   
剩下的事情就是对 n 进行因子分解，写出注册机.  4dd466d0e1e0cc123afbe504da2462ca


# 翻译 FLEXlm9.2 的破解教学六

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-706.htm`](https://www.pediy.com/kssd/pediy07/pediy7-706.htm)

  对于 4 个 Vendor 密钥，很明显它们是初始化时在 l\_n36\_buf()中被模糊的，在 l\_xorname()中进行模糊逆向，l\_xorname()未做别的仅进行了一些与或操作。我们都知道与或的特征：x ^ x = 0, x ^ 0 = x, x ^ 1 = ~x，因此进行两次相同的与或操作将相互取消。这个好特性将与或适用于编码和译码。如果我们的猜测对的话，那么 l\_n36\_buf()将采用与 l\_good\_lic\_key()相同的方式调用 l\_xorname()。因为 l\_n36\_buf()驻留于 lm\_new.c，它由 lmnewgen.exe 产生，因此直接检查 lmnewgen.c 更好。  
  
  
#include "l\_strkey.h"  
... ...  
int main(int argc, char \*\*argv)  
{  
      ... ...  
      strcpy(outfile, "lm\_new.c");  
      ... ...  
/\* 初始化工作，能报告错误并能退出 \*/  
if (lc\_init(0, vendor\_name, &vendorkeys\[0\], &job))  
      {    
fprintf(stderr, "lc\_init failed: %s\\n", lc\_errstring(job));  
         exit(1);  
      }    
/\* 通过 sb\_rngFIPS186Session()产生的随机数 \*/  
l\_genrand(job, lmseed1, lmseed2, lmseed3, NEWSEEDSIZ, newseeds);  
      ... ... /\* 把新种子转换为 seed1 – seed4,及其它\*/  
/\* 把 4 个加密种子写进 lmseeds.h \*/  
if (!(ofp = fopen("lmseeds.h", "w")))  
      {    
perror("Can't write lmseeds.h file, exiting");  
         exit(1);  
      }    
fprintf(ofp, "... ...", seed1, seed2, seed3, seed4);  
      fclose(ofp);  
  
/\*现在是 lm\_new.c \*/  
if (!(ofp = fopen(outfile, "w")))  
      {    
perror("Can't open output file, exiting");  
         exit(1);  
      }    
      ... ...  
/\* 确定缺省的和弱的种子被排除在外 \*/  
if (!l\_reasonable\_seed(seed3) || !l\_reasonable\_seed(seed4) ||  
         !l\_reasonable\_seed(lmseed1) || !l\_reasonable\_seed(lmseed2) ||  
         !l\_reasonable\_seed(lmseed3))  
      {    
         ... ...  
         fprintf(stderr, "Existing.\\n");  
         exit(1);  
      }    
      ... ...  
      fputs("#include <time.h>\\n", ofp);  
      do\_real(); /\* 写 lm\_new.c 的主要内容\*/  
      fclose(ofp);  
      return 0;  
}  
  
static void do\_real()  
{  
      ... ... /\* 产生随机的参数和函数名称 \*/  
      while (!key5\_uniqx)  
      {    
key5\_uniqx = our\_rand(256) + (our\_rand(256) << 8) +  
                  (our\_rand(256) << 16) + (our\_rand(256) << 24);  
      }    
      ... ... /\* 产生随机的 key5\_order\[\] \*/  
for (i = 0; i < 200; i += 2)  
         random\_garbage();  
      ... ... /\* 更多的随机垃圾 \*/  
for (counter=0; counter<keysize; counter++)/\* 一般 keysize=1, 只有一个 vendor \*/  
      {    
         key5(&vendorkeys\[counter\]);  /\* 模糊化 vendorkeys->data\[\] \*/  
         l\_xorname(vname, &vendorkeys\[counter\]);/\* 模糊化 vendorkeys->keys\[\] \*/  
/\* 真正 VENDORCODE 的初始化 \*/  
         do\_ulong(vendorkeys\[counter\].data\[0\], "data\[0\]", counter);  
         do\_ulong(vendorkeys\[counter\].data\[1\], "data\[1\]", counter);  
         do\_ulong(vendorkeys\[counter\].keys\[0\], "keys\[0\]", counter);  
         do\_ulong(vendorkeys\[counter\].keys\[1\], "keys\[1\]", counter);  
         do\_ulong(vendorkeys\[counter\].keys\[2\], "keys\[2\]", counter);  
         do\_ulong(vendorkeys\[counter\].keys\[3\], "keys\[3\]", counter);  
         ... ...  
      }    
      l\_puts\_rand(ofp, fpVar, numvars); /\* 以随机的顺序输出行 \*/  
      fflush(ofp);  
      uniqcode(); /\* 输出 lm\_new.c!l\_n36\_buff()的源代码 \*/  
      fflush(ofp);  
      ... ...  
}  
  
static void do\_ulong(unsigned long ul, char \*varname, int counter)  
{  
      ... ...  
randvarname(b1, "b1"); bnames\[0\] = b1; /\* 产生混乱的参数名称 \*/  
randvarname(b2, "b2"); bnames\[1\] = b2;  
randvarname(b3, "b3"); bnames\[2\] = b3;  
randvarname(b4, "b4"); bnames\[3\] = b4;  
  
for (i = 0; i < 4; i++)  
      {    
shift = i \* 8;  
  
         CLEARV; /\* 把 ul 分成 4 字节并且把它们指派为 4 个糊乱命名的参数 \*/  
sprintf(vBuf, "static unsigned int %s = %d;\\n",  
bnames\[i\], (ul & (0xff << shift)) >> shift );  
         fwrite(vBuf, sizeof(char), sizeof(vBuf), fpVar);  
  
         CLEARF; /\* 重新组装回 4 个参数，使 v->varname = ul \*/  
sprintf(fBuf, "\\tif (%s == %d) v->%s += (%s << %d);\\n",  
               countervar, counter, varname, bnames\[i\], shift);  
         fwrite(fBuf, sizeof(char), sizeof(fBuf), fpFunc);  
      }    
}  
  
static void uniqcode()  
{  
      ... ...  
int idx = \*vendor\_name % XOR\_SEEDS\_ARRAY\_SIZ; /\* idx = V % 20 = 86 % 20 = 6 \*/  
      XOR\_SEEDS\_INIT\_ARRAY(xor\_arr) /\* l\_strkey.h \*/  
  
fprintf(ofp, "static void %s(job, vendor\_id, key) \\n\\  
            ... ...  
unsigned long x = 0x%x; \\n\\  
struct s\_tmp {int i; char \*cp; unsigned char a\[12\];} \*t, t2; \\n\\  
if (job) t = (struct s\_tmp \*)job; \\n\\  
               else t = &t2; \\n\\  
            ... ...", key5\_fname, key5\_uniqx); /\* key5\_fname 和 key5\_uniqx 都是随机数\*/  
for(i = 0; i < SEEDS\_XOR\_NUM; i++) /\* 解码 t->a\[\], i.e. job->mem\_ptr2\_bytes\[\] \*/  
      {    
         unsigned char num;  
         cpp\[i\] = cp;  
sprintf(cp, "t->cp=(char \*)(((long)t->cp) ^ (time(0) ^ ((0x%x << 16) + 0x%x)));\\n\\  
t->a\[%d\] = (time(0) & 0xff) ^ 0x%x;\\n", /\* 运行时随机化 \*/  
               our\_rand(0xff), our\_rand(0xff), i, our\_rand(0xff));  
cp += strlen(cp) + 1;  
      }    
      l\_puts\_rand1(ofp, SEEDS\_XOR\_NUM, cpp); /\* 以随机的顺序输出行\*/  
  
fprintf(ofp, "for (i = 0; i < %d; i++) \\n\\  
            { \\n\\  
               if (sig\[i%%SIGSIZE\] != vendor\_id\[i%%len\]) \\n\\  
                  sig\[i%%SIGSIZE\] ^= vendor\_id\[i%%len\]; \\n\\  
            } \\n\\  
unsigned long y = ((((long)sig\[0\] << %d) \\n\\  
| ((long)sig\[1\] << %d) \\n\\  
   | ((long)sig\[2\] << %d) \\n\\  
| ((long)sig\[3\] << %d)) \\n\\  
^ ((long)(t->a\[%d\]) <<  0) \\n\\  
^ ((long)(t->a\[%d\]) <<  8) \\n\\  
^ ((long)(t->a\[%d\]) << 16) \\n\\  
^ ((long)(t->a\[%d\]) << 24) \\n\\  
^ x \\n\\  
^ key->keys\[0\] \\n\\  
^ key->keys\[1\]) & 0xffffffff; \\n\\  
key->data\[0\] ^= y; \\n\\  
key->data\[1\] ^= y; \\n\\  
      ... ...", MAX\_DAEMON\_NAME, /\* MAX\_DAEMON\_NAME = 10 在 lmclient.h 中定义 \*/  
key5\_order\[0\], key5\_order\[1\], key5\_order\[2\], key5\_order\[3\],  
xor\_arr\[idx\]\[0\], xor\_arr\[idx\]\[1\], xor\_arr\[idx\]\[2\], xor\_arr\[idx\]\[3\]));  
}  
  
static void key5(VENDORCODE \*k) /\* 模糊化加密种子, 即 k->data\[\] \*/  
{  
      ... ...  
/\* 和 uniqcode()中相同的 key5\_uniqx, key5\_order\[\], sig\[\]和 MAX\_DAEMON\_NAME\*/  
for (i = 0; i < MAX\_DAEMON\_NAME; i++)  
      {    
if (sig\[i%SIGSIZE\] != vname\[i % len\])  
            sig\[i%SIGSIZE\] ^= vname\[i % len\];  
      }    
y = ((((long)sig\[0\] << key5\_order\[0\])  
         | ((long)sig\[1\] << key5\_order\[1\])  
         | ((long)sig\[2\] << key5\_order\[2\])  
         | ((long)sig\[3\] << key5\_order\[3\]))  
         ^ key5\_uniqx  
         ^ k->keys\[0\]  
         ^ k->keys\[1\]) & 0xffffffff;  
      k->data\[0\] ^= y; /\* k->data\[0\] = 加密种子 SEED1 ^ y \*/  
      k->data\[1\] ^= y; /\* k->data\[1\] = 加密种子 SEED2 ^ y \*/  
}  
  
  
  源码有点长，但是它的结构一旦被我们揭开就不再非常难懂。主要的过程在 lmseeds.h 中花费大量时间，并且 lm\_new.c 主要是在 do\_real()中产生的。在几个子程序的帮助下，do\_real()本身是在制造 l\_n36\_buf()，它把 l\_n36\_buff()留给 uniqcode()。那么 l\_xorname()干什么用呢? 它虽然没有出现在 l\_n36\_buf()中，但是在 do\_real()中出现了。因此这种说法是正确的：4 个 vendor key 在不同的位置被同一个异或操作来进行加密和解密，它们分别是 do\_real() / l\_n36\_buf() 和 l\_good\_lic\_key()   
  注意 lm\_new.c 本身被高度模糊化：这里有大量完全无用的垃圾代码, 标识符是随机、无任何意义的， 真代码和垃圾代码看上去相似并且混在一起，真代码的行顺序被打乱……。很明显，它以这种方式设计的目的是用来迷惑读者，并且它成功了，因为理解 C 源码几乎是不可能的，更不用说反编译码了。这正是我们为什么要把注意力放在它的起源 lmnewgen.c 上的原因, 因为后者能给我们更多提示。例如，l\_xorname()在 do\_real()中被调用,而不是 l\_n36\_buf()。这意味着密钥的模糊化是在 vendor site 的内部进行的，并且加密后的密钥被链接到传送到终端用户的目标中，然后它们在 l\_good\_lic\_key()中被实时解码。这种方案是为了最小化真实密钥的暴露。  
  但是，我们对加密种子（encryption seeds）比 vendor key 更感兴趣。回想一下，job->mem\_ptr2\_bytes\[\] 和 code->data\[\] 在验证和密钥产生的过程中是不一样的。很可能 FLEXlm 也使用了异或进行种子加密，但是远不止这些，因为我们在运行时看到它是随机的。我们将逐步分析。  
  恰好在 do\_real()->l\_xorname()前面一行，这儿有一处对 key5()的调用, 它在 vendor 那边对原有的加密种子进行模糊化 (在工作结构上没做任何事情). vendor key 的经验告诉我们它们一定在用户方的某个地方进行了反向模糊。这时从事这项工作的不再是 l\_good\_lic\_key(), 而是 l\_n36\_buff(). 它在 lm\_new.c 中的源码是可读的，但我们仍然依然顷向于研究它的产生器 uniqcode()。不多久我们发现 l\_n36\_buff()对一些和 key5()中一样的参数进行了异或， 并加上额外的 t->a\[\]。那么它又是什么东西呢？t->a\[\]正是 job->mem\_ptr2\_bytes\[\]的另外一个名称. 怎样去解决那些额外的异或呢？让我们翻回去几页…，噢，在 l\_string\_key()中宏 L\_MOVELONG()处，正好在散列（hashing）开始之前。  
  现在我们看到种子加密/解密在两个步骤中完成， l\_n36\_buf()/l\_n36\_buff() 和 l\_n36\_buff()/ l\_string\_key()。在相应函数中的 xor 代码相互镜像，从而保证清除掉所有的干扰。l\_n36\_buff()中的 time(0)因子引入了运行时的随机性，它同时影响了我们在 l\_string\_key()中看到的 code->data\[\]和 job->mem\_ptr2\_bytes\[\] . 确实，与 vendor keys 相对而言，FLEXlm 以一种更加模糊的方式隐藏了加密种子。我们现在知道了 lm\_new.c 中两个名字神秘的函数的意义（事实上只有两个）: l\_n36\_buf() – 初始化 VENDORCODE 、加密种子和密钥；l\_n36\_buff() – 解开 l\_n36\_buf()，并进行第二步加密。  
  我们需要强调模糊化只用于 checkout 过程。在 keygen 过程中 lmcrypt.exe 并不调用 l\_n36\_buf()、l\_n36\_buff()，或者 l\_good\_lic\_key()，并且 L\_MOVELONG 对原始种子无任何影响，因为 job->mem\_ptr2\_bytes\[\]总是 0。在 checkout 的加密/解密过程中，xor 操作涉及了众多的常数、变量和数组。它们常被放置在两个地方，一个用于编码，另一个用于解码。为了方便，下表对它们进行了概括。  
  
  
  
  相关的对象  编码  解码  
VENDOR\_KEY  l\_xorname(),  
VENDORNAME,  do\_real(), l\_n36\_buf()  l\_good\_lic\_key()  
ENCRYPTION\_SEED,步骤 1  
  VENDORMAGIC\_V7  do\_real(),key5(), l\_n36\_buf()  uniqcode(), l\_n36\_buff()  
ENCRYPTION\_SEED,步骤 2  x = key5\_uniqx,   
key5\_order\[\],   
sig\[\], MAX\_DAEMON\_NAME  
idx, xor\_arr  uniqcode(), l\_n36\_buff()  L\_MOVELONG(), l\_string\_key()  
  
  现在是我们从 cmath.exe 中找回真正的加密种子的时候了。真正的种子作为 L\_MOVELONG()的第一个参数被恢复。使用 idx=6，和前面所述的置换表 xor\_arr\[\]，xor 操作数包含 4 字节,，在 job->mem\_ptr2\_bytes\[\]中被编址在 7、3、5、11。经过汇编后是 00A0A000.因此, ENCRYPTION\_SEED1 =  code->data\[0\] ^ 00A0A000 = 52ED15B8 ^ 00A0A000 = 524DB5B8，ENCRYPTION\_SEED2 =  code->data\[1\] ^ 00A0A000 = 75CF780F ^ 00A0A000 = 756FD80F。  
  在 lmseeds.h 中试用它们.呜呼! lmcrypt.exe 产生了亲爱的 6D5C01FD71C9。改变版本号为 5.0，我们得到 3F23BE3056E4, 又是正确的鉴名档! 这无疑确定我们已经获得 Visual Numerics 的可信的加密种子(encryption seeds)和 vendor keys。 最终,我们能生成任何我们所想要的 VNI license keys, 可以应用于其它的版本, 其它的 features,其它的产品…只要 FLEXLM 没做重大的修改, 它应能够正常工作而不会有任何故障发生.  
  
F:\\flexlm>type license.dat  
FEATURE CMATH VNI 5.0 permanent uncounted 0 HOSTID=ANY  
FEATURE CSTAT VNI 5.0 permanent uncounted 0 HOSTID=ANY  
FEATURE CMATH VNI 5.5 permanent uncounted 0 HOSTID=ANY  
FEATURE CSTAT VNI 5.5 permanent uncounted 0 HOSTID=ANY  
FEATURE CMATH VNI 7.1 permanent uncounted 0 HOSTID=ANY  
FEATURE CSTAT VNI 8.3 permanent uncounted 0 HOSTID=ANY  
FEATURE Hello VNI 2.9 permanent uncounted 0 HOSTID=ANY  
FEATURE cRaCk VNI 4.0 permanent uncounted 0 HOSTID=ANY  
FEATURE CMATH VNI 5.5 permanent uncounted HOSTID=ANY SIGN=0  
  
E:\\flexlm>utils\\lmcrypt -i license.dat  
FEATURE CMATH VNI 5.0 permanent uncounted 3F23BE3056E4 HOSTID=ANY  
FEATURE CSTAT VNI 5.0 permanent uncounted 2C60CD4570B0 HOSTID=ANY  
FEATURE CMATH VNI 5.5 permanent uncounted 6D5C01FD71C9 HOSTID=ANY  
FEATURE CSTAT VNI 5.5 permanent uncounted 369B56AC8B35 HOSTID=ANY  
FEATURE CMATH VNI 7.1 permanent uncounted F218B30D7129 HOSTID=ANY  
FEATURE CSTAT VNI 8.3 permanent uncounted CC5FA3C48B85 HOSTID=ANY  
FEATURE Hello VNI 2.9 permanent uncounted 505E4E243D1B HOSTID=ANY  
FEATURE cRaCk VNI 4.0 permanent uncounted 93D0E20E2D20 HOSTID=ANY  
FEATURE CMATH VNI 5.5 permanent uncounted HOSTID=ANY \\  
        SIGN=B5E1542279DC  
  
进一步的讨论  
  
尽管我们已经对 FLEXLM 保护系统进行了完全的反向工程,这里仍有许多东西值得讨论. 最显著的一个是, VENDOR\_KEY5 在哪里?  
  
（待续。。。。。。） 

 4dd466d0e1e0cc123afbe504da2462ca
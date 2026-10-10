

# SemCAD1.4

> 原文：[`www.pediy.com/kssd/pediy05/pediy50153.htm`](https://www.pediy.com/kssd/pediy05/pediy50153.htm)

软接缝的弱点——从 SemCAD1.4 的暴破谈 FlexLM 7.2 保护的破解   
Passion 抛砖引玉   
2001 年底   
（感谢看雪提供的 FlexLM 7.2 的 SDK 和其中的 SIG 文件）   
  
早听说过高手关于狗加密弱点的论述，说许多作者过于相信狗的保护能力，在程序的保护部分中没有作过多的手脚，甚至只读狗验证一次即通过，狗的多种加密与保护本领没有得到利用。遗憾的是运气不好，这种“接缝”的弱点我偏偏没有碰到过。不过最近在对付 Flexible Licence Manager 保护的软件的时候，倒是领会了一点点这个“接缝”的弱点，这里写出来让大伙儿“斧正斧正”，算是抛砖引玉，嘿嘿。^\_^   
Flexible Licence Manager 的保护不同于加壳，后者是对软件产品的可执行文件进行处理，而前者提供的是源代码的形式，需要开发者手工将加密保护代码加入产品中再重新编译。这种源代码保护比加壳在形式上更复杂，至少不存在对这种保护“脱壳”的说法。据我的一点点见识，FlexLM 所保护的软件大多是 CAD 等或者其他大型一点的工程软件，像我初学的时候对付的 Rational Rose 2000（当然，当时没破出来，^\_^）和最近看到的 SemCAD 便是这样。   
FlexLM 同样有其弱点。SDK 中即使源代码不公开，其加密接口也算是公开的，只是不同软件中使用的加密种子和 Features 等参数不同，因此 Licence 文件也就不通用，不是随便生成一个冒牌货就能蒙混过关。   
用 FlexLM 进行保护，需要在自己的源代码中加入 FlexLM 带的各种库和头文件，然后在需要保护的部分加保护代码，FlexLM 7.2 的 SDK 文档中的例子程序是这样的：   
  
#include "lmpolicy.h"   
LP\_HANDLE \*lp\_handle;   
      /\*...\*/   
      if (lp\_checkout(LPCODE, LM\_RESTRICTIVE|LM\_MANUAL\_HEARTBEAT,   
                      "myfeature","1.0", 1, "license.dat", &lp\_handle))   
      {   
              fprintf(stderr, "Checkout failed: %s",   
                                    lp\_errstring(lp\_handle));   
              exit(-1);   
      /\* FlexLM 校验出错 \*/   
      }   
/\*   
\*    检验通过，执行程序。   
\*/   
      /\*...\*/   
/\*   
\*    校验完毕，释放 Licence 占用的资源。   
\*/   
      lp\_checkin(lp\_handle);   
  
既然正儿八经的文档里头是这么指导的，那么有几个软件的保护代码会和这里的流程不同？倘若找到了调用 lp\_checkout 的地方再人为修改掉返回值，那么 FlexLM 的保护就算被暴力砍掉了，不管 lp\_checkout 里面算法多复杂都没用。   
  
（IDA 中加载 FLEXLM7.2 SDK 的 SIG 文件，反 SEMCad.exe，18M，费了半个上午、一个中午加一个下午。）   
  
反出来的代码中有这么一段：   
  
.text:006A19E4                push    offset a1\_4    ; "1.4"   
.text:006A19E9                push    offset aSemcad\_core ; "SEMCAD\_CORE"   
.text:006A19EE                push    eax   
.text:006A19EF                push    edx   
.text:006A19F0                call    sub\_69DCE0   
  
.text:006A0210                push    offset a1\_4    ; "1.4"   
.text:006A0215                push    offset aSemcad\_gui ; "SEMCAD\_GUI"   
.text:006A021A                push    eax   
.text:006A021B                push    edx   
.text:006A021C                call    sub\_69DCE0   
  
（这里 sub\_69DCE0 大概是初始化。暴露了 Features 是"SEMCAD\_GUI"和"SEMCAD\_CORE"）   
  
lp\_checkout 函数的一般调用形式是：   
  
lp\_checkout(LPCODE, policy, feature, version, 1, path, &lp)   
  
  
正巧对应到程序中的这段代码：   
  
.text:0069DDDB                push    esp   
.text:0069DDDC                push    eax                // lp 地址   
.text:0069DDDD                push    edx          // Licence 文件路径   
.text:0069DDDE                push    1          // 就是那个 1   
.text:0069DDE0                push    ecx                // Feature 版本地址   
.text:0069DDE1                push    ebx                // Feature 地址    
.text:0069DDE2                push    0A00h              // policy   
.text:0069DDE7                push    offset off\_1409940  // LPCODE 地址   
.text:0069DDEC                call    \_lp\_checkout   
  
  
//.text:00841820 处是\_lp\_checkout 的地址。   
  
.text:0069DDF1                add    esp, 20h   
.text:0069DDF4                test    eax, eax   
.text:0069DDF6                jnz    loc\_69E618   
  
再看看 loc\_69E618：嘿嘿！   
  
  .text:0069E618 loc\_69E618:                            ; CODE XREF: sub\_69DCE0+116j   
  .text:0069E618                lea    ecx, \[ebp+var\_94\]   
  .text:0069E61E                lea    eax, \[ebp+var\_84\]   
  .text:0069E624                push    eax   
  .text:0069E625                push    offset aLicenseCheckFa ;   
                "License check failed: "   
这里是多么的明显哇。   
  
.text:0069ddec 处调用\_lp\_checkout。   
  
.data1:015DB960 aLicense\_dat    db '\\license.dat',0    ; DATA XREF: sub\_69DCE0+60o   
.data1:015DB960                                        ; sub\_6A1940+45o   
  
  
因此，要暴力解决 SemCAD，只需要把.text:0069DDF4 处的 test eax,eax 和 jnz loc\_69e618 改为 mov eax,0 和三个 nop 即可。   
  
\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_   
  
  
如果要跟踪出正确的 Key 和 Seed 以生成永久 Licence 文件，可以从 lp\_checkout 的参数处分析：   
  
lp\_checkout(LPCODE, policy, feature, version, 1, path, &lp)   
  
#define LPCODE &\_lpcode 所以第一个参数是一个叫\_lpcode 变量的地址。   
  
static LPCODE\_HANDLE \_lpcode  = { &lp\_code,   
#if defined (WINNT) && defined(FLEXLM\_DLL)   
    VENDOR\_NAME   
#else   
    0   
#endif   
    };   
这里定义了\_lpcode 变量，它是一个静态的结构，其第一个属性是一个叫 lp\_code 变量的地址。   
  
#if defined (WINNT) && defined(FLEXLM\_DLL)   
LM\_CODE(lp\_code, ENCRYPTION\_SEED1, ENCRYPTION\_SEED2, VENDOR\_KEY1,   
    VENDOR\_KEY2, VENDOR\_KEY3, VENDOR\_KEY4, VENDOR\_KEY5);   
// 这里其实也就是定义一个 VENDERCODE 型的变量 lp\_code，具体见下一个宏定义。   
#else   
static VENDORCODE lp\_code;   
#endif   
// 到这里为止，不管条件如何，总归定义了一个全局变量 lp\_code   
  
#define LM\_CODE(name, x, y, k1, k2, k3, k4, k5)  \\   
                    static VENDORCODE name = \\   
                        { VENDORCODE\_6, \\   
                          { (x)^(k5), (y)^(k5) }, \\   
                          { (k1), (k2), (k3), (k4) }, \\   
                          {0}, \\   
                          {0}, \\   
                          LM\_STRENGTH, \\   
                          0, \\   
                          FLEXLM\_VERSION, \\   
                          FLEXLM\_REVISION, \\   
                          FLEXLM\_PATCH, \\   
                          LM\_VER\_BEHAVIOR, \\   
                          CRO\_KEY1, \\   
                          CRO\_KEY2 \\   
                          }   
  
这里常数 VENDORCODE\_6 的值是 4，因为有一句#define VENDORCODE\_6    4   
  
VENDORCODE 结构定义如下：   
  
#define VENDORCODE VENDORCODE6   
// 这里说明 VENDORCODE 其实就是 VENDORCODE6 型的结构   
  
下面是核心 VENDORCODE6 结构的定义：   
  
typedef struct vendorcode6 {   
                short type;      /\* Type of structure \*/   
                unsigned long data\[2\];   
                            /\* 64-bit code 放两个 ENCRYPTION\_SEED 和 VENDOR\_KEY5 异或后的数值\*/   
                unsigned long keys\[4\]; /\* 放四个 VENDOR\_KEY\*/   
#define LM\_PUBKEYS     3   
#define LM\_MAXPUBKEYSIZ 40   
/\*   
\*                pubkey is for both the public and private keys   
\*                The public key goes here when authenticating   
\*                the private key when generating licenses   
\*/   
                int pubkeysize\[LM\_PUBKEYS\];   
                unsigned char pubkey\[LM\_PUBKEYS\]\[LM\_MAXPUBKEYSIZ\];   
                int pubkeyinfo1;   
                int (\*pubkey\_fptr)();   
                short flexlm\_version;   
                short flexlm\_revision;   
                char flexlm\_patch\[2\];   
#define LM\_MAX\_BEH\_VER 4   
                char behavior\_ver\[LM\_MAX\_BEH\_VER + 1\];   
                unsigned long crokeys\[2\];   
              } VENDORCODE6, \*VENDORCODE\_PTR;   
  
// 以上定义了 VENDORCODE 型也就是 VENDORCODE6 的结构   
  
（这里 defined 里头是根据开发平台上的条件定义的，被保护软件采用的是何种 define 可根据软件本身适应的平台、是否使用了 FLEXLM 的 DLL 等来稍微确定一下。）   
  
现在颗颗珠子终于串到一起来了。   
  
从调用实现过程上来说，lp\_checkout 传入的第一个参数是一个地址，根据这个地址找到一个指向的内存区域，该区域是 LPCODE\_HANDLE 型结构，变量名为 \_lpcode，它的第一个属性还是一个地址，指向 lp\_code 结构（第二个属性不是一个不相干的数便是 0，因此不用管了）。所以，将 lp\_checkout 传入的第一个参数进行二次寻址便能找到 lp\_code 结构，该结构的第一个部分是 type，从第二个部分开始的 24 个字节也就是 6 个 DWord 分别储存了两个 ENCRYPTION\_SEED 和 VENDOR\_KEY5 异或后的数值加上四个 VENDOR\_KEY。   
由于变量是自左至右推入堆栈的，因此程序中最近的一个 push 便是 LPCODE 参数：   
  
.text:0069DDE7                push    offset off\_1409940   
.text:0069DDEC                call    \_lp\_checkout   
  
由此可知，地址 1409940 处是 LPCODE\_HANDLE 结构，1409940 处的值在本程序中是 1552260，它是 VENDORCODE6 结构的地址。   
值得注意的是，根据 FlexLM SDK 的声明，该处是存放一个 VENDORCODE6 结构，但并没规定调用\_lp\_checkout 前此结构必须要填充必要的数值。事实上刚进入\_lp\_checkout 的时候该内存区域全是 0，VendorKey 等数值是\_lp\_checkout 内部才进行填充的，如下：   
  
\_lp\_checkout 内部：   
  
.text:00841820                push    ebp   
.text:00841821                mov    ebp, esp   
.text:00841823                sub    esp, 0E4h   
.text:00841829                mov    \[ebp+var\_68\], 0   
.text:00841830                mov    \[ebp+var\_C\], 0   
.text:00841837                mov    \[ebp+var\_10\], 0   
.text:0084183E                mov    eax, \[ebp+arg\_18\]   
.text:00841841                mov    dword ptr \[eax\], offset unk\_157D6B8   
.text:00841847                lea    ecx, \[ebp+var\_5C\]   
.text:0084184A                push    ecx   
.text:0084184B                mov    edx, \[ebp+arg\_0\]   
.text:0084184E                mov    eax, \[edx\]   
.text:00841850                push    eax   
.text:00841851                push    0   
.text:00841853                push    0   
.text:00841855                call    \_lc\_new\_job   
  
这里调用完\_lc\_new\_job 后，EAX 所指的区域（也就是 VendorCode 结构所在地）才被填入两个加密 Seed 和四个 Key。其值如下：   
  
5E51FD8:   
  
04 00 00 00 A2 0A BD 51 84 0F BE CC FA FE 77 FF   
32 46 AC C8 CF DF D0 35 F8 AD 1E 83   
  
倒序后：   
00000004 51BD0AA2 CCBE0F84 FAFE77FF C8AC4632 35D0DFCF 831EADF8   
  
这里刚好对应到：   
static VENDORCODE name = \\   
                      { VENDORCODE\_6, \\            // 其实就是一个 4。   
               { (x)^(k5), (y)^(k5) }, \\   
              { (k1), (k2), (k3), (k4) }, \\   
                      ……   
                      （下面的不管了）   
  
所以可知：   
VENDOR\_KEY1 0xFAFE77FF   
VENDOR\_KEY2 0xC8AC4632   
VENDOR\_KEY3 0x35D0DFCF   
VENDOR\_KEY4 0x831EADF8   
VENDOR\_KEY5 0x0 //还不知道，^\_^   
  
写到这里，本人水平有限，在跟踪下面的代码过程中始终找不到第五个 Key 的校验处，也就没法子完成这篇文章，请高手指教指教。   
  
——————————————————————————————————————   
  
  
下面是另外一些有用的定义：   
  
typedef struct \_lp\_handle {   
    char feature\[MAX\_FEATURE\_LEN + 1\];   
    char lickey\[MAX\_CRYPT\_LEN + 1\];   
#ifdef LM\_INTERNAL   
    long last\_failed\_reconnect;   
    long \*recent\_reconnects;   
    int num\_minutes;   
    LM\_HANDLE \*job;   
    int policy;   
#endif /\* LM\_INTERNAL \*/   
} LP\_HANDLE, FAR \* LP\_HANDLE\_PTR;   
  
  
typedef struct \_lpcode\_handle {   
    VENDORCODE \* code;   
    char \*        vendor\_name;   
} LPCODE\_HANDLE;   
  
VENDORCODE vendorkeys\[\] = {        /\* Possible keys for vendor daemons \*/   
        { VENDORCODE\_5,   
        ENCRYPTION\_SEED1^VENDOR\_KEY5, ENCRYPTION\_SEED2^VENDOR\_KEY5,   
          VENDOR\_KEY1, VENDOR\_KEY2, VENDOR\_KEY3, VENDOR\_KEY4,   
          {0}, {0}, LM\_STRENGTH,  0,   
          FLEXLM\_VERSION, FLEXLM\_REVISION, FLEXLM\_PATCH,   
          LM\_BEHAVIOR\_CURRENT, {CRO\_KEY1, CRO\_KEY2}},   
                  };  4dd466d0e1e0cc123afbe504da2462ca
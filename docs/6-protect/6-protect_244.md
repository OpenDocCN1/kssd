

# 制作 GL Studio v3.02 的 license

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-47.htm`](https://www.pediy.com/kssd/pediy09/pediy09-47.htm)

> 制作 GL Studio v3.02 的 license  
>   
> 按：好像没什么技术含量，仅供参考！  
>   
> 手头的这个软件，试用版 license 已经过期，故不得不自己动手找一个不过期的 license。  
> license 中签名的长度是 12 个字符，可断定没使用公钥保护，不需打补丁就可以找出关键  
> 参数，做出正确的 license。  
>   
> 1、确定版本号（用 lmtools.exe 或 lmver.exe 都可以）  
> FLEXlm v9.2.0 (lmgr.lib)  
>   
> 2、生成 Vendor Key（用 lmkg.exe）  
>   
> /\* Version 9 keys \*/  
> #define VENDOR\_KEY1 0x109922eb  
> #define VENDOR\_KEY2 0x9ca04eba  
> #define VENDOR\_KEY3 0xc3934b79  
> #define VENDOR\_KEY4 0xdbcfd49c  
> #define VENDOR\_KEY5 0x623a63f1  
> #define CRO\_KEY1 0x14b920e3  
> #define CRO\_KEY2 0xda822c3c  
>   
> #define VENDOR\_NAME "DISTI"  
>   
> 3、找 FEATURE  
> 这个很好找，在试用版 license 里面就有。或者在 IDA 里面也能找到。有两个：  
> glStudio  
> glStudioRuntime  
>   
> 4、找 ENCRYPTION\_SEED  
>   
> 反汇编主程序 glStudio.exe，其中有一段是进行 license 检查的代码：  
>   
> .text:004642CA                 push    offset aAcquiringLicen ; "Acquiring License..."  
> .text:004642CF                 call    sub\_476F60  
> .text:004642D4                 add     esp, 4  
> .text:004642D7  
> .text:004642D7 loc\_4642D7:                             ; CODE XREF: \_main+328j  
> .text:004642D7                 push    edi  
> .text:004642D8                 push    offset a3\_0     ; "3.0"  
> .text:004642DD                 push    offset aGlstudio ; "glStudio"  
> .text:004642E2                 mov     ecx, offset unk\_653090  
> .text:004642E7                 call    ds:?Checkout@License@disti@@QAEXPAD0\_N@Z ; disti::License::Checkout(char \*,char \*,bool)  
> .text:004642ED                 mov     ecx, offset unk\_653090  
> .text:004642F2                 call    ds:?Valid@License@disti@@QAE\_NXZ ; disti::License::Valid(void)  
> .text:004642F8                 test    al, al  
> .text:004642FA                 jnz     short loc\_46430A  
> .text:004642FC                 push    offset aGlStudioDevelo ; "GL Studio Development license not found"...  
> .text:00464301                 call    ds:?fl\_message@@YAXPBDZZ ; fl\_message(char const \*,...)  
> .text:00464307                 add     esp, 4  
>   
> 上面这段代码调用了 glstudio3\_0\_2\_vc71.dll 中的几个函数。  
>   
> 反汇编 glstudio3\_0\_2\_vc71.dll，寻找两个关键函数 lc\_checkout、l\_sg。  
> 这两个函数的原型（见 lm\_ckout.c）:  
>   
> API\_ENTRY  
> lc\_checkout(  
>   LM\_HANDLE\_PTR      job,    /\* Current license job \*/  
>   const LM\_CHAR\_PTR    feature,  /\* The feature to be checked in \*/  
>   const LM\_CHAR\_PTR    version,  /\* Feature's version \*/  
>   int            nlic,    /\* Number of licenses to checkout \*/  
>   int            flag,    /\* Checkout flag \*/  
>   const VENDORCODE\_PTR  key,    /\* The vendor's key \*/  
>   int            dup\_group)  /\* Duplicate license grouping criteria \*/  
> {  
>   
> void  
> l\_sg(  
>   LM\_HANDLE \*    job,  
>   char \*      vendor\_id,  
>   VENDORCODE \*  key) /\*- l\_sg means "signature vendor\_key5" \*/  
>   
> lc\_checkout 的定位比较简单，因为它有 7 个参数，在 feature 出现的地方就可以找到它。  
>   
> .text:10079F20 \_lc\_checkout    proc near               ; CODE XREF: sub\_10037C9A+54p  
> .text:10079F20                                         ; sub\_10037C9A+A9p ...  
> .text:10079F20  
> .text:10079F20 var\_8           = dword ptr -8  
> .text:10079F20 var\_4           = dword ptr -4  
> .text:10079F20 arg\_0           = dword ptr  8  
> .text:10079F20 arg\_4           = dword ptr  0Ch  
> .text:10079F20 arg\_8           = dword ptr  10h  
> .text:10079F20 arg\_C           = dword ptr  14h  
> .text:10079F20 arg\_10          = dword ptr  18h  
> .text:10079F20 arg\_14          = dword ptr  1Ch  
> .text:10079F20 arg\_18          = dword ptr  20h  
> .text:10079F20  
> .text:10079F20                 push    ebp  
> .text:10079F21                 mov     ebp, esp  
> .text:10079F23                 sub     esp, 8  
> .text:10079F26                 mov     \[ebp+var\_4\], 0  
> .text:10079F2D                 mov     eax, \[ebp+arg\_0\]  
> .text:10079F30                 push    eax  
> .text:10079F31                 call    sub\_100862CE  
> .text:10079F36                 add     esp, 4  
> .text:10079F39                 mov     ecx, \[ebp+arg\_0\]  
> .text:10079F3C                 mov     edx, \[ecx+3FCh\]  
> .text:10079F42                 or      dh, 40h  
> .text:10079F45                 mov     eax, \[ebp+arg\_0\]  
> .text:10079F48                 mov     \[eax+3FCh\], edx  
> .text:10079F4E                 push    95h  
> .text:10079F53                 push    offset aLm\_ckout\_c ; "lm\_ckout.c"      // 看看，多好的提示  
> .text:10079F58                 mov     ecx, \[ebp+arg\_0\]  
> .text:10079F5B                 push    ecx  
> .text:10079F5C                 call    \_\_initp\_misc\_winxfltr\_0  
> .text:10079F61                 add     esp, 0Ch  
> .text:10079F64                 mov     edx, \[ebp+arg\_0\]  
> .text:10079F67                 add     edx, 494h  
> .text:10079F6D                 push    0  
> .text:10079F6F                 push    edx  
> .text:10079F70                 call    \_setjmp3  
> .text:10079F75                 add     esp, 8  
> .text:10079F78                 test    eax, eax  
> .text:10079F7A                 jz      short loc\_10079F87  
> .text:10079F7C                 mov     eax, \[ebp+arg\_0\]  
> .text:10079F7F                 mov     eax, \[eax+14h\]  
> .text:10079F82                 jmp     loc\_1007A057  
>   
>   
> 在 IDA 中搜索立即数 0x6f7330b8，可找到 l\_sg。  
>   
> .text:1007DABB \_l\_sg           proc near               ; CODE XREF: sub\_10073B7E+C63p  
> .text:1007DABB                                         ; sub\_1007BC9F+C4p ...  
> .text:1007DABB  
> .text:1007DABB var\_24          = dword ptr -24h  
> .text:1007DABB var\_20          = dword ptr -20h  
> .text:1007DABB var\_1C          = dword ptr -1Ch  
> .text:1007DABB var\_14          = byte ptr -14h  
> .text:1007DABB var\_13          = word ptr -13h  
> .text:1007DABB var\_11          = byte ptr -11h  
> .text:1007DABB var\_10          = dword ptr -10h  
> .text:1007DABB var\_C           = dword ptr -0Ch  
> .text:1007DABB var\_8           = dword ptr -8  
> .text:1007DABB var\_4           = dword ptr -4  
> .text:1007DABB arg\_0           = dword ptr  8  
> .text:1007DABB arg\_4           = dword ptr  0Ch  
> .text:1007DABB arg\_8           = dword ptr  10h  
> .text:1007DABB  
> .text:1007DABB                 push    ebp  
> .text:1007DABC                 mov     ebp, esp  
> .text:1007DABE                 sub     esp, 24h  
> .text:1007DAC1                 mov     \[ebp+var\_14\], 0  
> .text:1007DAC5                 xor     eax, eax  
> .text:1007DAC7                 mov     \[ebp+var\_13\], ax  
> .text:1007DACB                 mov     \[ebp+var\_11\], al  
> .text:1007DACE                 mov     \[ebp+var\_C\], 6F7330B8h    // 这个是比较重要的特征参数  
> .text:1007DAD5                 mov     \[ebp+var\_4\], 0  
> .text:1007DADC                 mov     \[ebp+var\_8\], 0  
> .text:1007DAE3                 mov     \[ebp+var\_10\], 3  
> .text:1007DAEA                 mov     ecx, \[ebp+arg\_0\]  
> .text:1007DAED                 mov     edx, \[ecx+6Ch\]  
> .text:1007DAF0                 mov     eax, \[edx+0F54h\]  
> .text:1007DAF6                 and     eax, 8000h  
> .text:1007DAFB                 test    eax, eax  
> .text:1007DAFD                 jz      short loc\_1007DB22  
> .text:1007DAFF                 cmp     dword\_10213CB4, 0  
> .text:1007DB06                 jz      short loc\_1007DB22  
> .text:1007DB08                 mov     ecx, \[ebp+arg\_8\]  
> .text:1007DB0B                 push    ecx  
> .text:1007DB0C                 mov     edx, \[ebp+arg\_4\]  
> .text:1007DB0F                 push    edx  
> .text:1007DB10                 mov     eax, \[ebp+arg\_0\]  
> .text:1007DB13                 push    eax  
> .text:1007DB14                 call    dword\_10213CB4            // 在这个地方下断  
> .text:1007DB1A                 add     esp, 0Ch                  // 单步运行到这里  
>   
>   
> 然后就是动态跟踪，找到以下参数：  
>   
> 输入 dd \[esp+8\]  
> 4D917F04               data\[0\]  
> 4D9C696B               data\[1\]  
>   
> 输入 dd \[esp+4\]  
> DISTI                  vendor\_id  
>   
> 输入 dd \[esp\]  
> 341F91CA               job+08  
> 00E80685               job+0c  
> 004D0000               job+10  
>   
> 使用工具 calcseed.exe，输入上述信息，可计算得到：  
> #define ENCRYPTION\_SEED1 0x0000face  
> #define ENCRYPTION\_SEED2 0x000deca1  
>   
> ENCRYPTION\_SEED3、ENCRYPTION\_SEED4 是 SDK 中的默认值，不需要改动，对计算 license 没有影响。  
>   
> 至此,分析已经基本完成,可以使用 SDK 计算 license 了。  
>   
> 5、写注册机  
>   
> 按照上面的信息,编辑 lm\_code.h,修改 makefile，编译 lmcrypt.c，大功告成!  
>   
> 注意：要将  
> #define ENCRYPTION\_SEED1 0x0000face  
> #define ENCRYPTION\_SEED2 0x000deca1  
> #define ENCRYPTION\_SEED3 0x22334455  
> #define ENCRYPTION\_SEED4 0x33221100  
> 这几行放到 lm\_code.h 文件中，  
> #define VENDOR\_KEY5 。。。  
> 后面即可。  
>   
> 后记：  
> 第一次碰 FlexLM，没想到运气真好，照着 tulipfan\[CCG\]的大作  
> “制作 Compuware.SoftIce.Driver.Suite.3.0.1.StinkyD 的 license”就轻松地搞定了它。  
> 原计划花一个星期的时间，实际上只用了两天，总共 10 小时不到，还包括搜集资料和学习的  
> 时间。  
> 感谢 tulipfan\[CCG\]！  
>   
> \--------------------------------------------------------------------------------  
>   
> 附录：FlexLM SDK 使用说明  
>   
> 以下以 FLEXnet v10.1 为例，简单介绍 FlexLM SDK 的使用流程。  
> 这个流程也基本适用于 FlexLM SDK 的早期版本。  
>   
> 1、安装 FlexLM SDK  
>    刚安装完的 SDK 肯定是不能用的。若不信可运行 i86\_n3\\lmwin.exe 看看，肯定一个  
>    feature 都不能 checkout，呵呵。  
>      
> 2、怎样才能让 demo 运行起来呢？  
>    首先用工具 lmkg.exe 计算出一组 vendor key。以下是一个例子：     
>   
> /\* Version 10 keys \*/  
> #define VENDOR\_KEY1 0x321cd48b  
> #define VENDOR\_KEY2 0xe6e4652c  
> #define VENDOR\_KEY3 0x2076c1a2  
> #define VENDOR\_KEY4 0xbbb34234  
> #define VENDOR\_KEY5 0x0b165dca  
> #define CRO\_KEY1 0x363cd683  
> #define CRO\_KEY2 0xa0c607aa  
>   
> #define VENDOR\_NAME "demo"  
>   
>    然后在命令行下运行 i86\_n3\\lmrand1.exe，生成一组随机数种子。  
>      
> #define LM\_SEED1 0xcdfeb228  
> #define LM\_SEED2 0x1b111a77  
> #define LM\_SEED3 0xfe02cc3d  
>   
>    最后修改 machind\\m\_code.h 中相应的行。  
>    （注意：如果 VENDOR\_NAME 不是 demo，还要修改 makefile 中相应行。）  
>      
> 注意：  
>   
> A.从 v10.0 以后，CRO\_KEY1、CRO\_KEY2 改名为 TRL\_KEY1、TRL\_KEY1。  
>   
> B.关于 LM\_STRENGTH 的可能定义值  
> 可根据需要定义以下之一，推荐使用公钥保护，强度更高。  
>   
> 不使用公钥保护  
> LM\_STRENGTH\_DEFAULT:  签名长 12 字符  
> 使用公钥保护（ECC）  
> LM\_STRENGTH\_113BIT, LOW:     签名长  58 字符  
> LM\_STRENGTH\_163BIT, MEDIUM:  签名长  84 字符  
> LM\_STRENGTH\_239BIT, HIGH:    签名长 120 字符  
> 兼容 v7.1 以前的版本（不做变动）  
> LM\_STRENGTH\_LICENSE\_KEY  
>   
>   
> 3、写一个调用 Visual C++ 6.0 的脚本编译 SDK  
>   
> @echo off  
> set MSVCDir="C:\\Program Files\\Microsoft Visual Studio\\VC98"  
> set PATH=%MSVCDir%\\Bin;%PATH%  
> @call vcvars32.bat  
> @call build.bat  
> echo ............. Done!  
> pause  
>   
> 4、生成 licences  
>   
> lmcrypt counted.lic  
> lmcrypt uncounted.lic  
> lmcrypt expired.lic  
> pause  
>   
> 注意：需要先修改 lic 文件中除签名字符串以外的其它各项，然后再用以上命令  
> 生成新的 lic 文件。上面的命令只更改签名字符串。  
>   
> 5、测试 lincense  
>    首先用 i86\_n3\\lmtools.exe 配置一个 license server，可参考 FlexLM 编程指南。  
>    然后运行 i86\_n3\\lmwin.exe，看看所有的 feature 是不是都能 checkout。  
>      
>    我这里是没有问题的，不知道你那里怎么样呢？  
>   
> 6、在自己的软件中调用 FlexLM  
>    仿照 lmvendor.c 调用 API 即可，很简单了。 

 4dd466d0e1e0cc123afbe504da2462ca
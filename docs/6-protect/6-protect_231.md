

# 整理 flexlm\_v9.2\_full\_source 不完全编译手册

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-72.htm`](https://www.pediy.com/kssd/pediy09/pediy09-72.htm)

> 下载 flexlm\_v9.2\_full\_source 后，编译好久才把 demo 编译过去，考虑到可能会有同样辛苦的同志，所以把编译过程记录下来共享：  
>   
> 1)解压后从 dos 窗口进入 v9.2 目录下运行环境变量 vc.bat(在 vc 的安装目录 VC98\\Bin 下)  
> 2)修改 pc.mak 中 27 行为 CERTDIR = certicom\\lib\\$(FLEXLM\_MAKE\_PLATFORM)\\  
> 3)注释第 28 行 #CERTDIR = $(CERTICOM)  
> 4)在 src\\pc.mak 的 490 行下面添加一行  
>   copy ..\\flexlm-java-tng\\c\\i86\_n3\\lGetNativeHostId.obj .  
> 5)进入 utils 目录下面运行  
>   lmrand1.exe -seed  
>   生成 3 个随机种子，覆盖 h\\lm\_code.h 中对应宏定义  
> 6)运行 lmkg,生成以 vonder name 为 demo 的 vonder key 值，覆盖 h\\lm\_code.h 中对应宏定义  
> 7)src\\l\_ctype.c 替换'?'为'\*'(注意是注意其中是注释符号/\*\*/匹配，不涉及注释匹配的不要修改)  
> 8)copy certicom\\lib\\i86\_n3\\include\\\*.\* h /y  
> 9)copy certicom\\lib\\i86\_n3\\lib\\\*.\* i86\_n3 /y  
> 10)copy flexlm-java-tng\\c\\i86\_n3\\\*.\* src /y  
>   
> 11)编译设置环境变量  
>   a) cl makesetup.c  
>   b) makesetup RELEASE NONDEBUG i86\_n3 MT  
>         c) setup.bat  
>   
> 12)编译 SDK  
>   nmake /fpc.mak release  
>   看到\[Finished building kit! Don't forget to build lmtools utility.\]表示编译完全通过  
> 13)生成 license  
>   cd i86\_n3  
>   lmcrypt.exe counted.lic  
>   lmcrypt.exe uncounted.lic  
>   lmcrypt.exe expired.lic  
> 14)测试运行  
>   lmtools.exe 配置一个 license server。然后运行 lmwin.exe 将所有的 feature 都 checkout 一次  
>     
>   
> 附:vondor\_key1-5 与 lm\_seed1-3  
> #define VENDOR\_KEY1 0x4054cac1  
> #define VENDOR\_KEY2 0x4500538e  
> #define VENDOR\_KEY3 0x542a6830  
> #define VENDOR\_KEY4 0x3951f9f9  
> #define VENDOR\_KEY5 0x0b165dca  
> /\*  
>  \*   Vendor name.      Leave "demo" if evaluating.  Otherwise,  
>  \*         set to your vendor daemon name.  
>  \*/  
> #define VENDOR\_NAME "demo"  
> /\*  
>  \*   Private SEEDs:     Make up 3, 8-hex-char numbers, replace and  
>  \*        guard securely.  You can also use the command  
>  \*        'lmrand1 -seed' to make these numbers up  
>  \*/  
> #define LM\_SEED1 0x82395728  
> #define LM\_SEED2 0xac86853f  
> #define LM\_SEED3 0x13765cfb  
>   
>   
> 注：说是不完全编译是因为还有一些问题没有解决，genlic32 一直运行不起来正在查找资料  
> 参考不少朋友的文章，没有记下来，再次不提名感谢:) 

 4dd466d0e1e0cc123afbe504da2462ca
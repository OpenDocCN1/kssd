

# 让 VC7 系列生成 VC6 支持 PDB 格式,以便 VC6IDE 可以调试

> 原文：[`www.pediy.com/kssd/pediy10/98020.html`](https://www.pediy.com/kssd/pediy10/98020.html)

让 VC7 系列生成 VC6 支持 PDB 格式,以便 VC6IDE 可以调试  
morning.ye 2009-9-18  
  
我们知道 阅读 VC7 系列的 atl/mfc 源代码可以看到,VC7 似乎具备生成 vc6 pdb 文件的能力,其中参数分别是  
cl.exe:  /Zvc6  
link.exe: /debugtype:vc6  
  
但是,很遗憾,如果我们把这个参数加上,它们纷纷表示,不认识.  
当然,到底是否支持,调试一下才知道,通过分析,发现只是前端接口关掉了,不接受这些参数,但是内部实现确实支持的.  
OK,patch 它们.  
  
准备工作:  
拷贝一份 vc2002/2003bin 目录到 vc6bin 目录,把 vc6 的改名,vc2003toolkit 也可以  
从 PSDK/DDK2003 等地方拷贝 mspdb60.dll,版本号应该是 6.2  
  
开始修改  
1.1 修改编译前端  
分别用 od 加载 c1.dll,c1xx.dll,搜索字符串 mspdb60.dll,在前方不远处有一个比较,记下这个全局变量的地址,在 LoadPE 里转成 offset,并将原来的 0 改成 1  
c1.dll 在偏移 10678a11 处,将原来的 0 改成 1  
c1xx.dll 在偏移 1058D3B1 处,将原来的 0 改成 1  
1.2 修改编译后端:c2 略麻烦一点,不过调试类似,  
c2.dll 在偏移 1070744C 处,将原来的 1D 改成 3D  
  
2.修改连接器  
link.exe  
0042A4DC      8A98 60010000 mov     bl, byte ptr \[eax+160\]  
0042A4E2      84DB          test    bl, bl  
0042A4E4  |.  56            push    esi  
0042A4E5  |.  57            push    edi  
0042A4E6      74 0B         je      short link.0042A4F3  
0042A4E8  |.  BF AC3C4000   mov     edi, link.00403CAC                   ;  mspdb60.dll  
0042A4ED  |.  897C24 0C     mov     dword ptr \[esp+C\], edi  
0042A4F1  |.  EB 0C         jmp     short link.0042A4FF  
0042A4F3  |>  C74424 0C A03>mov     dword ptr \[esp+C\], link.00403CA0     ;  mspdb71.dll  
0042A4FB  |.  8B7C24 0C     mov     edi, dword ptr \[esp+C\]  
0042A4FF  |>  57            push    edi                                  ; /FileName  
0042A500  |.  FF15 A4104000 call    dword ptr \[<&KERNEL32.LoadLibraryA>\] ; \\LoadLibraryA  
  
改成  
0042A4DC      B3 01         mov     bl, 1              //把原来的判断改成赋值  
0042A4DE      8898 60010000 mov     byte ptr \[eax+160\], bl      //  
0042A4E4  |.  56            push    esi  
0042A4E5  |.  57            push    edi  
0042A4E6      90            nop                    //不再跳转,其实跳转也没关系,因为后面我们会替换这个名字  
0042A4E7      90            nop  
0042A4E8  |.  BF AC3C4000   mov     edi, link.00403CAC                         ;  mspdb60.dll  
0042A4ED  |.  897C24 0C     mov     dword ptr \[esp+C\], edi  
0042A4F1  |.  EB 0C         jmp     short link.0042A4FF  
0042A4F3  |>  C74424 0C A03>mov     dword ptr \[esp+C\], link.00403CA0           ;  mspdb71.dll  
0042A4FB  |.  8B7C24 0C     mov     edi, dword ptr \[esp+C\]  
0042A4FF  |>  57            push    edi                                        ; /FileName  
0042A500  |.  FF15 A4104000 call    dword ptr \[<&KERNEL32.LoadLibraryA>\]       ; \\LoadLibraryA  
  
  
3.用 16 进制编译器打开 c1.dll,c1xx.dll,c2.dll,cl.exe,link.exe  
搜索 pdb71.dll 全部改成 pdb60.dll  
  
4.在 link.exe 中把 ?Open@PDB@@SAHPBD0KPAJQADPAPAU1@@Z 改成 ?Open@PDB@@SAHPAD0KPAJQADPAPAU1@@Z  
因为 pdb60.dll 把参数声明为 char\*,而 pdb71.dll 把参数声明为 const char\*,这导致修改符不同  
  
5.从 PSDK/DDK 任何能找到 mspdb60.dll 的地方把 6.2 版的 mspdb60.dll 拷贝进去  
  
  
OK,一切搞定,现在新建一个 console 工程,编译 debug 配置,可以看到 生成的文件内部已经是 NB10 标记的 PDB 文件了,开始调试,不再提示没有符号文件,可以正常显示变量等信息.  
  
  
附件已上传,是基于**VC7.1SP1**修改的,不打算自己挨个改一遍的朋友,找个 VC2003bin 目录拷贝后,用压缩包里的文件覆盖一下就 OK 了.

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || bin.rar \[附件请到论坛下载：[`bbs.pediy.com/showthread.php?t=98020 `](http://bbs.pediy.com/showthread.php?t=98020)\]   
 |
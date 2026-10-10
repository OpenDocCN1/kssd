

# 超星 PDG 转换

> 原文：[`www.pediy.com/kssd/pediy05/pediy50331.htm`](https://www.pediy.com/kssd/pediy05/pediy50331.htm)

  
＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝  
“超星读书卡”持卡用户可以通过 Internet 下载超星数字图书馆中的数字图书。

令很多用户困扰的是在一台机器上下载的文件，在另一台机器上无法直接阅读。

虽然可以通过获取离线注册码的方式解决，但总感觉不尽方便，能否妥善解决？

也许我们注意到在线阅读时是不分那台机器的，可以想象其 PDG 文件在从网上传

到机器内时是不具备机器识别特征的，这些文件是下载到本地时在本地加密后存

盘的。

因此我们有可能在分析出其本地加密算法，然后推演出其逆算法，将其还原

成原始文件，那么就可以实现自由的离线阅读了。

经过算法分析，证明此方法是可行的，并做出这个 demo 版的小工具：

1\. 此工具暂没有批量转换功能，一次只能转换一个选定文件

2\. 我只做了不全面的简单测试，不一定适用于所有的 PDG 文件

3\. 我用的是 SSReader3.6 及以前版下载的 PDG 文件，通过测试

4\. 现在 SSReader3.7 的加密方式已经改变，肯定不支持，有待以后分析

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝

以下论述是我跟踪分析的大致结果,表述不是很精确,有些是猜测   
所有分析是基于 3.6 版的,新版的格式可能会增加内容或有所改变,   
我的分析是针对于两种主要类型(格式)的,其实也是目前绝大部分   
图书所采用的类型,一种是我们在线阅览的那种 pdgtype=02h   
另一种是下载存储到本地硬盘文件的那种 pdgtype=10h   
因此以下描述的结构只能保证对这两种文件有效,其它类型的还需   
进一步的分析验证.   
  
\==========================================================================================   
1\. PDG 文件的大致结构:   
  
PDG2\_file STRUCT   
    HH\_header<>   
    optional header<>   
    PDG\_data<>   
PDG2\_file ENDS   
  
HH\_header STRUCT   
    dw    4848h            ;超星文件特征标志   
    db    02h            ;PDG\_VERSION=2   
    db    00h            ;我见到的 PDG2 这里都是 0,就算默认吧   
    dd    ?            ;不知道,对我们似乎没有价值   
    dd    offset\_optional\_header    ;optional\_header 在文件中的偏移地址,应该等于 0Ch 吧   
HH\_header ENDS   
  
optional\_header STRUCT   
    db    80h,00h,00h        ;没什么好说的,算是标志吧   
    db    pdgtype            ;文档类型,我只分析 02h 和 10h 两种类型   
    dw    x\_pix            ;扫描图像的横向参数   
    dw    y\_pix            ;扫描图像的纵向参数   
    db    01h,00h,00h,00h        ;按默认算吧   
    dd    offset\_PDG\_data        ;扫描图像数据在文件中的偏移地址,大概都是 8Ch 吧   
    dd    size\_PDG\_data        ;扫描图像数据的字节数   
    dd    8 dup (?)        ;作用可能不大,把它们添上 0 不影响页面的显示   
    key\_data<>   
    db    1Ch dup (?)        ;作用可能不大,把它们添上 0 不影响页面的显示   
optional\_header ENDS   
  
key\_data STRUCT   
    db    1Eh dup (?)        ;这里有时是一个有关超星公司字符信息,有时为空(全 0)   
    dw    ?            ;我不清楚什么含义   
    db    8 dup (?)        ;可能有几处默认是 0   
    dd    SS\_user\_key        ;重要的数据,可用于解码还原 x\_pix 和 y\_pix   
    dd    ?            ;   
key\_data ENDS   
\==========================================================================================   
2\. 我所想象的图书阅览管理过程(基于简单的分析,未必准确,仅供参考)   
    超星图书服务器上所存储的图书是扫描生成的(废话,大家都知道),是按 pdgtype=02h 存储的,   
文件结构与上述结构相符,单其扫描数据 PDG\_data 是经过加密的,加密过程如下:   
    30h 字节长的 key\_data 经过 md5 运算得到 128 位即 16 字节的数据,此 16 字节的数据作为密钥,采用   
一种分组加密算法以 16 字节为单位进行加密运算,直到数据结束,此分组加密算法我根据手头的不多   
的密码学算法比较,我没有识别出来,我暂且称之为 encode\_sub,我将它放在后面了,大家有兴趣的可   
以帮忙看一下,有知道可以的告诉我,万分感谢.   
    当我们在线阅览时,ssreader.exe 中的相对应 decode\_sub 将数据解码后,就可以显示出来了.   
  
    在下载到本地硬盘时,文件存储成 pdgtype=10h 类型的了,PDG\_data 扫描数据是用解码后的原始   
数据存储的,但它把 optional\_header 中部分数据加密了,加密过程如下:   
    跟据每台机器的硬盘 C:分区的卷序列号和空间大小对应此机器的 SS 机器码,由 SS 机器码可以算   
出一个 word 值,暂时称为 SS\_w,这段算法我没去分析,对我们用处也不大.在下载存盘时还需要一个随   
机 word 值 RANDOM\_w,RANDOM\_w 实际就作为 SS\_user\_key 的高 16 位,由 SS\_w 和 RANDOM\_w 再算出一个 word   
值作为 SS\_user\_key 的低 16 位,因其算法可逆,所以可以用 SS\_user\_key 还原出原来的 SS\_w,这也是原程   
序用来检测当前硬盘的 SS\_w 和下载文件的 SS\_w 是否一致的方法,不一致则提示你用户不对不是无法阅   
读.   
    接下来用 SS\_user\_key(用 SS\_w 和 RANDOM\_w 算法稍变也可以)算出两个 word 值,分别去减 x\_pix 和   
y\_pix,最后把 pdgtype 添上 10h.这样与我们有用的处理就算结束了,存盘后就可以了.   
\==========================================================================================   
3\. 我们需要做的工作   
    我们只要把 pdgtype=10h 的文件转变成 pdgtype=02h 的文件,就可以实现不受硬盘的限制脱机自由   
阅读了.过程大致这样吧,大家也应该可以想到了,我用语言描述很费尽的,因此这里用汇编代码描述吧:   
  
下面这段算法在 pdg2.dll 可以找到,我贴的稍有调整,并去掉了 SS\_w 的检测   
  
          mov ebp,SS\_user\_key   
          shr eax, 10h   
          xor edx, edx   
          mov esi,1FFh   
          div esi   
          mov ecx, edx   
          mov eax,ebp   
          shr eax,10h   
          xor edx, edx   
          mov esi,0DBh   
          div esi   
          imul ecx, edx   
          lea edx, dword ptr \[ecx-000050EEh\]   
          mov ebx, ebp   
          sub ebx, edx   
          and ebx, 0000FFFFh   
          shr ebp, 10h   
          mov eax, ebx   
          xor edx, edx   
          mov ecx, 000003FBh   
          mov edi, 00000083h   
          div ecx   
          imul ebx, ebp   
          mov eax, ebp   
          mov ecx, edx   
          xor edx, edx   
          div edi   
          mov eax, edx   
          sub eax, ebx   
          sub eax, ecx   
          sub ecx, ebx   
          sub ecx, edx   
      add x\_pix,ax        ;还原 x\_pix   
      add y\_pix,cx        ;还原 y\_pix   
  
为了看起来干净利索,我们把 key\_data 区域都清 0,不这样也没问题,不过有几处可能不要乱数   
      lea edi,key\_data   
      mov ecx,30h   
      xor eax,eax   
          rep stosb   
然后对 30h 字节长度的 key\_data 进行一次 md5,md5 我就不贴在这里了,太费篇幅了,大家都可以   
找到,我们下面要用到这 128 位(16 字节)的结果作为分组加密算法的密钥,实际上我们的 key\_data   
已经是清 0 的了,结果也是固定不变的了,也可以直接用现成的结果.我假设结果存在 key\_128 处.   
  
为转成 pdgtype=02h 文件,PDG\_data 要用 encode\_sub 加密   
  
                mov    edi, size\_PDG\_data        ;数据长度   
                cmp    edi, 10h   
                jl      data\_encode\_end   
                mov    ebx, key\_128   
                mov    esi, offset PDG\_data   
                shr    edi, 4   
@@:            push    ebx                ;密钥指针   
                push    esi                ;数据指针,加密结果也存储在这里   
                call    sub\_encode            ;分组算法,每次加密 10h 字节   
                add    esp, 8   
                add    esi, 10h   
                dec    edi   
                jnz    @B   
data\_encode\_end:                    ;处理结束   
  
;下面是完整的分组加密算法,在\*\*\*\*\*\*\*\*.exe 中可以找到   
;其中注释是我分析时加的,不尽准确,本想删去,后来一想算了,献丑也罢,只当交流了   
;开始看时感觉上有点象 RC5 族算法,但越看越不象,资料有限,还是确认不了,望高手指教   
  
encode\_sub      proc near   
var\_2C          = dword ptr -2Ch   
var\_28          = dword ptr -28h   
var\_24          = dword ptr -24h   
var\_20          = dword ptr -20h   
var\_1C          = dword ptr -1Ch   
var\_18          = dword ptr -18h   
var\_14          = dword ptr -14h   
var\_10          = dword ptr -10h   
var\_C          = dword ptr -0Ch   
var\_8          = dword ptr -8   
var\_4          = dword ptr -4   
arg\_0          = dword ptr  8            ;pData   
arg\_4          = dword ptr  0Ch        ;pKey   
                push    ebp   
                mov    ebp, esp   
                add    esp, 0FFFFFFD4h   
                mov    eax, \[ebp+arg\_0\]    ;pData   
                mov    edx, \[eax\]        ;A   
                mov    \[ebp+var\_4\], edx    ;sa=A   
                mov    edx, \[eax+4\]        ;B   
                mov    \[ebp+var\_8\], edx    ;sb=B   
                mov    edx, \[eax+8\]        ;C   
                mov    \[ebp+var\_C\], edx    ;sc=C   
                mov    edx, \[eax+0Ch\]        ;D   
                mov    \[ebp+var\_10\], edx    ;sd=D   
                mov    eax, \[ebp+arg\_4\]    ;pKey   
                mov    edx, \[eax\]        ;ka   
                mov    \[ebp+var\_14\], edx    ;ka   
                mov    edx, \[eax+4\]        ;kb   
                mov    \[ebp+var\_18\], edx    ;kb   
                mov    edx, \[eax+8\]        ;kc   
                mov    \[ebp+var\_1C\], edx    ;kc   
                mov    edx, \[eax+0Ch\]        ;kd   
                mov    \[ebp+var\_20\], edx    ;kd   
                xor    edx, edx        ;c1   
                mov    \[ebp+var\_24\], edx    ;c1   
                mov    \[ebp+var\_28\], 9E3779B9h    ;c0   
                mov    \[ebp+var\_2C\], 10h    ;r   
@@:            mov    ecx, \[ebp+var\_28\]    ;c0   
                add    \[ebp+var\_24\], ecx    ;c1+c0   
                mov    eax, \[ebp+var\_8\]    ;sb   
                shl    eax, 4            ;sb<<4   
                add    eax, \[ebp+var\_14\]    ;sb<<4+ka   
                mov    edx, \[ebp+var\_8\]    ;sb   
                add    edx, \[ebp+var\_24\]    ;sb+c1   
                xor    eax, edx        ;(sb<<4+ka) xor (sb+c1)   
                mov    ecx, \[ebp+var\_8\]    ;sb   
                shr    ecx, 5            ;sb>>5   
                add    ecx, \[ebp+var\_18\]    ;sb>>5+kb   
                xor    eax, ecx        ;(sb<<4+ka) xor (sb+c1) xor (sb>>5+kb)   
                add    \[ebp+var\_4\], eax    ;sa=sa+\[(sb<<4+ka) xor (sb+c1) xor (sb>>5+kb)\]   
                mov    eax, \[ebp+var\_C\]    ;sc   
                shl    eax, 4            ;sc<<4   
                add    eax, \[ebp+var\_1C\]    ;sc<<4+kc   
                mov    edx, \[ebp+var\_C\]    ;sc   
                add    edx, \[ebp+var\_24\]    ;sc+c1   
                xor    eax, edx        ;(sc<<4+kc) xor (sc+c1)   
                mov    ecx, \[ebp+var\_C\]    ;sc   
                shr    ecx, 5            ;sc>>5   
                add    ecx, \[ebp+var\_20\]    ;sc>>5+kd   
                xor    eax, ecx        ;(sc<<4+kc) xor (sc+c1) xor (sc>>5+kd)   
                add    \[ebp+var\_8\], eax    ;sb=sb+\[(sc<<4+kc) xor (sc+c1) xor (sc>>5+kd)\]   
                mov    eax, \[ebp+var\_10\]    ;sd   
                shl    eax, 4            ;sd<<4   
                add    eax, \[ebp+var\_14\]    ;sd<<4+ka   
                mov    edx, \[ebp+var\_10\]    ;sd   
                add    edx, \[ebp+var\_24\]    ;sd+c1   
                xor    eax, edx        ;(sd<<4+ka) xor (sd+c1)   
                mov    ecx, \[ebp+var\_10\]    ;sd   
                shr    ecx, 5            ;sd>>5   
                add    ecx, \[ebp+var\_20\]    ;sd>>5+kd   
                xor    eax, ecx        ;(sd<<4+ka) xor (sd+c1) xor (sd>>5+kd)   
                add    \[ebp+var\_C\], eax    ;sc=sc+\[(sd<<4+ka) xor (sd+c1) xor (sd>>5+kd)\]   
                mov    eax, \[ebp+var\_4\]    ;sa   
                shl    eax, 4            ;sa<<4   
                add    eax, \[ebp+var\_1C\]    ;sa<<4+kc   
                mov    edx, \[ebp+var\_4\]    ;sa   
                add    edx, \[ebp+var\_24\]    ;sa+c1   
                xor    eax, edx        ;(sa<<4+kc) xor (sa+c1)   
                mov    ecx, \[ebp+var\_4\]    ;sa   
                shr    ecx, 5            ;sa>>5   
                add    ecx, \[ebp+var\_18\]    ;sa>>5+kb   
                xor    eax, ecx        ;(sa<<4+kc) xor (sa+c1) xor (sa>>5+kb)   
                add    \[ebp+var\_10\], eax    ;sd=sd+\[(sa<<4+kc) xor (sa+c1) xor (sa>>5+kb)\]   
                mov    eax, \[ebp+var\_2C\]    ;r   
                dec    eax            ;r-1   
                mov    \[ebp+var\_2C\], eax    ;r=r-1   
                jnz    @B   
                mov    eax, \[ebp+arg\_0\]    ;pData   
                mov    edx, \[ebp+var\_4\]    ;sa   
                mov    \[eax\], edx        ;A=sa   
                mov    edx, \[ebp+var\_8\]    ;sb   
                mov    \[eax+4\], edx        ;B=sb   
                mov    edx, \[ebp+var\_C\]    ;sc   
                mov    \[eax+8\], edx        ;C=sc   
                mov    edx, \[ebp+var\_10\]    ;sd   
                mov    \[eax+0Ch\], edx        ;D=sd   
                mov    esp, ebp   
                pop    ebp   
                retn   
encode\_sub      endp   
\==========================================================================================   
后记:文章我是最懒得写的,写起来真是辛苦,花了数个小时,总感觉表述的不是很满意,无奈已经尽力了   
    分析水平有限,错误之处在所难免,唯恐误人子弟,读者朋友们见谅了,希望大家能提出自己的主见.   
\==========================================================================================   
          heXer/iPB   
          2002.10.26 

 4dd466d0e1e0cc123afbe504da2462ca
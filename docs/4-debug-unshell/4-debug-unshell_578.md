

# CrypKey v5.3.218 解密分析 

> 原文：[`www.pediy.com/kssd/pediy04/kanxue246.htm`](https://www.pediy.com/kssd/pediy04/kanxue246.htm)

我从来没有找到过，(但肯定有，因为我从下载的注册机中得到的 NFO 文件中确定了)。我现在用的是一个 DEMO 版，使用原来旧版本的注册机+旧版本的 Cryp9516x.DLL 以欺骗方式解密的。你如果想下载最新版的 DEMO 请到 http://www.crypkey.com/。前段时间我曾在此寻求有关 CrypKey 破解的帮助(注意：是算法破解而非脱壳-脱壳太 Easy 了)，可惜没人理我，亏我还是看着这个论坛火起来的，哎...   
  
再附欺骗手法：   
  
    安装 CrypKey 最新 DEMO-版本 5.3.218 至 C:\\CKI，备份一个至 D:\\CKI，拷贝旧版本的 Cryp9516c.DLL 至 D:\\CKI 并重命名为 Cryp9516e.DLL，使用 Duelist/CORE 的注册机对 D:\\CKI 的 CrypKey Instant 及 Site Key Generator 进行注册。再将 License 导回 C:\\CKI，这样你就拥有一个可使用的 CrypKey 了。详细的使用方法可参看 Duelist/CORE 的注册机中的 NFO 文件。我在 Win98 下确实试验成功了！！！   
  
再次寻求帮助!!!   
  
    请问有那位高手对 CrypKey 的加密算法有研究，请不吝赐教。我只解决了校验初期及末期的简单异或运算，对于核心部分的运算(经过对被加密程序及 DEMO 版中的 Site Key Generator 进行分析)还毫无心得，反正其中使用了浮点运算指令。如果有哪位高手有兴趣一起研究，请 E-Mail 联系。我的 E-Mail 地址：hzhprivate@yahoo.com.cn。为了大家研究方便我简要给出一些在分析 SKW.EXE 时的关键数据：   
  
//CrypKey v5.3.218 解密分析   
//   
//以下程序片段来自 CrypKey Site Key Generator v5.4   
//   
//脱壳后进行反编译。(注意：脱壳后的程序不可运行，我想是对入口表做了特殊处理)   
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
/\*    程序开始 - 主程序部分   
/\*    这段程序将输入的 SITE CODE 转化为 SITE KEY   
/\*    为了统一观察结果，我采用下面这个 SITE CODE：   
/\*    D268 96B6 2C22 B0AD AF   
/\*    采用生成 1 个无限制协议   
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
/\*    D268 96B6 2C22 B0AD AF  -   
/\*                            | ->这一步运算已经实现(004166F2 call 00416EED)   
/\*    04CF 0C7E 8401 2B4E 49  -   
/\*   
/\*    F936 0004 1728 9259            - (这组数字纯粹与 SITE CODE 有关)   
/\*                                  | ->(00416701 - 0073F384)   
/\*    7901 5465 0000 0000 0000 9E5A  - (这组数字是 SITE CODE 与 Liscence 的信息综合)   
/\*      \*\* ->这就是生成 Liscence 的 Copy Number(0 - 127)   
/\*    \*\* ->决定了 Liscence 的属性(可叠加性等)   
/\*                            \*\*\*\* ->相当于前两项属性的校验   
/\*    (004176ED - 0073F4D8)   
/\*   
/\*    下面两组字符串实际上是在同一段程序中产生的   
/\*    93AE CE84 0000 0000 0000 94A5   
/\*   
/\*    2193 AECE 8400 0000 0000 0094 A5  -   
/\*                                      | ->这一步的运算已经实现   
/\*    F711 68A4 9C06 C66A 1806 C6FE 29  -   
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00417400(C)   
|   
:00417410 8D95F0FEFFFF            lea edx, dword ptr \[ebp+FFFFFEF0\]    //EDX -> 变换的结果   
:00417416 52                      push edx   
:00417417 68300EFE40              push 40FE0E30   
:0041741C 6A00                    push 00000000   
:0041741E 6800B49040              push 4090B400   
:00417423 6A00                    push 00000000   
:00417425 56                      push esi                //ESI -> 已经去除了空格的 SITE CODE   
  
\* Possible Reference to String Resource ID=00001: "English"   
                                  |   
:00417426 6A01                    push 00000001                //重要参数，决定了下面调用的内部路由   
:00417428 E8A3F2FFFF              call 004166D0                //对已经去除了空格的 SITE  CODE 进行变换   
                                    //(关键调用，着重分析对象)   
:0041742D 83C41C                  add esp, 0000001C   
:00417430 8D4DF8                  lea ecx, dword ptr \[ebp\-08\]        //ECX -> 变换的结果    
:00417433 51                      push ecx   
:00417434 8D85F0FEFFFF            lea eax, dword ptr \[ebp+FFFFFEF0\]    //EAX -> 上一步得到的字符串   
:0041743A 50                      push eax   
:0041743B E812F7FFFF              call 00416B52                //将输入的字符串转为相应的数字串    
:00417440 83C408                  add esp, 00000008   
:00417443 8BF0                    mov esi, eax   
:00417445 56                      push esi   
:00417446 8D45F8                  lea eax, dword ptr \[ebp\-08\]        //EAX -> 上一步得到的数字串    
:00417449 50                      push eax   
:0041744A E803FAFFFF              call 00416E52                //对输入的数字串作 CRC 校验，结果返回 AX    
:0041744F 83C408                  add esp, 00000008   
:00417452 6685C0                  test ax, ax                //校验结果    
:00417455 7410                    je 00417467                //通过校验   
:00417457 8B5530                  mov edx, dword ptr \[ebp+30\]   
:0041745A C60200                  mov byte ptr \[edx\], 00   
:0041745D B8FEFFFFFF              mov eax, FFFFFFFE            //置错误代码   
:00417462 E999020000              jmp 00417700                //结束   
///////////////////////////////////////////////////////////   4dd466d0e1e0cc123afbe504da2462ca
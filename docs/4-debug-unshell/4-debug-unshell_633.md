

# netsowell 的壳的 IAT 简要分析的修复【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-552.htm`](https://www.pediy.com/kssd/pediy08/pediy8-552.htm)

netsowell 的壳的 IAT 简要分析的修复  
  
原贴在这里  
http://bbs.pediy.com/showthread.php?s=&threadid=22815  
原程序下载：ennotepad.rar   
  
由于没有花指令，分析起来好看多了，感谢 netsowell 大侠给我等菜鸟调试分析的机会  
  
  
野猪力量 --- 注入!  
  
  
      002C0264            8945 90           mov dword ptr ss:\[ebp-70\],eax  
      002C0267            33C0              xor eax,eax  
      002C0269            8945 8C           mov dword ptr ss:\[ebp-74\],eax  
      002C026C            6A 2C             push 2C  
      002C026E            8D85 D0FEFFFF     lea eax,dword ptr ss:\[ebp-130\]              //这里的 ebp-130 是每次处理时在堆栈中的数据结构的起始  
      002C0274            50                push eax  
      002C0275            53                push ebx  
      002C0276            FFD6              call esi                                    //从这里到最后所有的 esi 都是 memcpy(src, dst, size)  
      002C0278            E9 CC020000       jmp       002C0549  
这个是准备处理第一个调用输入表变形的地方， 也就是把 call \[IAT\]，变成 call 2X0000  
壳的思路就是，将一块一块关于变形 call 的数据结构 copy 到堆栈的局部变量中，根据  
数据结构完成每一个变形  
  
这个数据结构大致是这样的  
struct ss  
{  
    BYTE      key;          //解密输入表 dll 和函数名字的密钥，算法是异或  
    DWORD     IATRva;       //相关的在输入表(IAT)中的位置的 RVA    
    DWORD     IsOrdinal;    //1= 通过 Ordinal 取导入函数, 0=通过名字取导入函数  
    DWORD     Unknown1;  
    DWORD     Ordinal;      //Ordinal 的值，如果没有，这里是-1  
    BYTE      code1\[7\];     //1 个 E8 的 call, 目的地址是处理输入表变形的公共部分  
                              BYTE\[7\]的起始地址是 IAT 中的值  
    WORD      temp1;        //临时写入 E9  
    DWORD     ff25addr;     //如果不为 0，则这个值加上主代码段起始地址得到需要变形  
                              的地址，且这个变形是 FF25 的变形  
    WORD      temp2;        //临时写入 90 E8  
    DWORD     ff15addr;     //如果不为 0，则这个值加上主代码段起始地址得到需要变形  
                              的地址，且这个变形是 FF15 的变形  
    DWORD     UnKnown2;  
  
    //以上共 0x2C 长  
    enum  
    {  
        BYTE      EnDllName\[\];  //加密的 dll 的名字，密钥是上面的 key, 算法是异或  
        DWORD     EnFunAddr;    //将\* (LPDWORD)code1\[1\]和导入函数的地址异或  
    }  
    BYTE      EnFunName\[\];  //加密的导入函数的名字，密钥是上面的 key, 算法是异或  
       
}  
  
  
      002C027D            8BC3              mov eax,ebx  
      002C027F            0345 8C           add eax,dword ptr ss:\[ebp-74\]  
      002C0282            83C0 11           add eax,11  
      002C0285            8B55 90           mov edx,dword ptr ss:\[ebp-70\]  
      002C0288            2BD0              sub edx,eax  
      002C028A            83EA 05           sub edx,5  
      002C028D            8955 88           mov dword ptr ss:\[ebp-78\],edx  
      002C0290            C685 E1FEFFFF E8  mov byte ptr ss:\[ebp-11F\],0E8  
      002C0297            8B45 88           mov eax,dword ptr ss:\[ebp-78\]  
      002C029A            8985 E2FEFFFF     mov dword ptr ss:\[ebp-11E\],eax             //完成对数据结构 ss 中 ss.code1 的计算  
      002C02A0            6A 2C             push 2C  
      002C02A2            8BC3              mov eax,ebx  
      002C02A4            0345 8C           add eax,dword ptr ss:\[ebp-74\]  
      002C02A7            50                push eax  
      002C02A8            8D85 D0FEFFFF     lea eax,dword ptr ss:\[ebp-130\]  
      002C02AE            50                push eax  
      002C02AF            FFD6              call esi                                   //将更新后的 ss.code1 部分写回原来的地方  
      002C02B1            8BC3              mov eax,ebx  
      002C02B3            0345 8C           add eax,dword ptr ss:\[ebp-74\]  
      002C02B6            83C0 11           add eax,11  
      002C02B9            8945 88           mov dword ptr ss:\[ebp-78\],eax  
      002C02BC            6A 04             push 4  
      002C02BE            8B45 AC           mov eax,dword ptr ss:\[ebp-54\]              //程序基址  
      002C02C1            0385 D1FEFFFF     add eax,dword ptr ss:\[ebp-12F\]             //再加上 ss.IATRva 就是这个变形相关的 IAT 的地址  
      002C02C7            50                push eax  
      002C02C8            8D45 88           lea eax,dword ptr ss:\[ebp-78\]  
      002C02CB            50                push eax  
      002C02CC            FFD6              call esi                                   //将 ss.code1 的起始地址写入相关的 IAT 中  
                                                                                       //这样以后每次 call \[IAT\]，相当于 call ss.code1  
                                                                                       //ss.code1 再 call 入处理变形的公共代码部分  
      002C02CE            8345 8C 2C        add dword ptr ss:\[ebp-74\],2C  
      002C02D2            8B45 8C           mov eax,dword ptr ss:\[ebp-74\]  
      002C02D5            03C3              add eax,ebx  
      002C02D7            8945 A8           mov dword ptr ss:\[ebp-58\],eax  
      002C02DA            83BD 54FFFFFF 00  cmp dword ptr ss:\[ebp-AC\],0  
      002C02E1            0F85 F6010000     jnz       002C04DD  
      002C02E7            8BFB              mov edi,ebx  
      002C02E9            037D 8C           add edi,dword ptr ss:\[ebp-74\]                
      002C02EC            0FB685 D0FEFFFF   movzx eax,byte ptr ss:\[ebp-130\]           //取出 ss.key 密钥  
      002C02F3            50                push eax  
      002C02F4            57                push edi                                  //ss.EnDllName  
      002C02F5            E8 6D020000       call       002C0567                       //解出 dll 的名字，返回名字的长度  
      002C02FA            8945 80           mov dword ptr ss:\[ebp-80\],eax  
      002C02FD            8B45 80           mov eax,dword ptr ss:\[ebp-80\]  
      002C0300            0145 8C           add dword ptr ss:\[ebp-74\],eax  
      002C0303            57                push edi  
      002C0304            FF55 98           call dword ptr ss:\[ebp-68\]                //Call GetModuleHandle，取 dll 的基础  
      002C0307            8945 84           mov dword ptr ss:\[ebp-7C\],eax  
      002C030A            837D 84 00        cmp dword ptr ss:\[ebp-7C\],0  
      002C030E            75 07             jnz short       002C0317  
      002C0310            57                push edi  
      002C0311            FF55 A0           call dword ptr ss:\[ebp-60\]                //如果 dll 不存在，则 LoadLibrary 这个 dll  
      002C0314            8945 84           mov dword ptr ss:\[ebp-7C\],eax  
      002C0317            57                push edi  
      002C0318            E8 96020000       call       002C05B3                       //取得 dll 基址后，将 dll 的名字加密回去  
      002C031D            837D 84 00        cmp dword ptr ss:\[ebp-7C\],0  
      002C0321            0F84 11020000     je       002C0538  
  
  
      002C0327            83BD D5FEFFFF 00  cmp dword ptr ss:\[ebp-12B\],0              //ss.IsOrdinal 是否为 1  
      002C032E            75 2F             jnz short       002C035F  
      002C0330            8BFB              mov edi,ebx  
      002C0332            037D 8C           add edi,dword ptr ss:\[ebp-74\]  
      002C0335            0FB685 D0FEFFFF   movzx eax,byte ptr ss:\[ebp-130\]  
      002C033C            50                push eax  
      002C033D            57                push edi  
      002C033E            E8 24020000       call       002C0567                       //ss.IsOrdinal 为 0 则解密导入函数的名字  
      002C0343            8945 80           mov dword ptr ss:\[ebp-80\],eax  
      002C0346            57                push edi  
      002C0347            8B45 84           mov eax,dword ptr ss:\[ebp-7C\]  
      002C034A            50                push eax  
      002C034B            FF55 9C           call dword ptr ss:\[ebp-64\]                //GetProcAddress 取导入函数地址  
      002C034E            8945 84           mov dword ptr ss:\[ebp-7C\],eax  
      002C0351            57                push edi  
      002C0352            E8 5C020000       call       002C05B3                       //将导入函数名字加密回去  
      002C0357            8B45 80           mov eax,dword ptr ss:\[ebp-80\]  
      002C035A            0145 8C           add dword ptr ss:\[ebp-74\],eax  
      002C035D            EB 11             jmp short       002C0370  
  
      002C035F            8BBD DDFEFFFF     mov edi,dword ptr ss:\[ebp-123\]            //ss.Ordinal  
      002C0365            57                push edi  
      002C0366            8B45 84           mov eax,dword ptr ss:\[ebp-7C\]  
      002C0369            50                push eax  
      002C036A            FF55 9C           call dword ptr ss:\[ebp-64\]                //ss.IsOrdinal 为 1 则通过 ss.Ordinal 取导入函数  
      002C036D            8945 84           mov dword ptr ss:\[ebp-7C\],eax  
      002C0370            8B85 D9FEFFFF     mov eax,dword ptr ss:\[ebp-127\]  
      002C0376            83E8 01           sub eax,1  
      002C0379            72 1D             jb short       002C0398  
      002C037B            0F84 EC000000     je       002C046D  
      002C0381            2D FF0F0000       sub eax,0FFF  
      002C0386            0F84 0D010000     je       002C0499  
      002C038C            48                dec eax  
      002C038D            0F84 28010000     je       002C04BB  
      002C0393            E9 A0010000       jmp       002C0538  
  
      002C0398            8B85 E2FEFFFF     mov eax,dword ptr ss:\[ebp-11E\]  
      002C039E            3145 84           xor dword ptr ss:\[ebp-7C\],eax            //ss.EnFunAddr = ApiName ^ \*(LPDWORD)ss.code1\[1\]       
      002C03A1            6A 04             push 4  
      002C03A3            8B45 A8           mov eax,dword ptr ss:\[ebp-58\]  
      002C03A6            50                push eax  
      002C03A7            8D45 84           lea eax,dword ptr ss:\[ebp-7C\]  
      002C03AA            50                push eax  
      002C03AB            FFD6              call esi                                 //将更新后的 ss.EnFunAddr 写回原来的地方  
  
  
      002C03AD            83BD EAFEFFFF 00  cmp dword ptr ss:\[ebp-116\],0             //检查 ss.ff25addr 是否为 0  
      002C03B4            76 51             jbe short       002C0407  
      002C03B6            66:C785 EEFEFFFF >mov word ptr ss:\[ebp-112\],0E9            //ff25.addr 不为 0，则 ss.temp1 写入 e9  
      002C03BF            8B45 A8           mov eax,dword ptr ss:\[ebp-58\]  
      002C03C2            83E8 1B           sub eax,1B  
      002C03C5            8B95 78FFFFFF     mov edx,dword ptr ss:\[ebp-88\]  
      002C03CB            0395 EAFEFFFF     add edx,dword ptr ss:\[ebp-116\]  
      002C03D1            2BC2              sub eax,edx  
      002C03D3            83E8 05           sub eax,5                                //通过 e9 的目的地址 ss.code1 计算偏移地址  
      002C03D6            8945 84           mov dword ptr ss:\[ebp-7C\],eax              
      002C03D9            6A 02             push 2  
      002C03DB            8B85 78FFFFFF     mov eax,dword ptr ss:\[ebp-88\]  
      002C03E1            0385 EAFEFFFF     add eax,dword ptr ss:\[ebp-116\]  
      002C03E7            50                push eax  
      002C03E8            8D85 EEFEFFFF     lea eax,dword ptr ss:\[ebp-112\]  
      002C03EE            50                push eax  
      002C03EF            FFD6              call esi                                 //写入 e9  
      002C03F1            6A 04             push 4  
      002C03F3            8B85 78FFFFFF     mov eax,dword ptr ss:\[ebp-88\]  
      002C03F9            0385 EAFEFFFF     add eax,dword ptr ss:\[ebp-116\]  
      002C03FF            40                inc eax  
      002C0400            50                push eax  
      002C0401            8D45 84           lea eax,dword ptr ss:\[ebp-7C\]  
      002C0404            50                push eax  
      002C0405            FFD6              call esi                                 //写入 e9 的偏移地址  
  
  
      002C0407            83BD F0FEFFFF 00  cmp dword ptr ss:\[ebp-110\],0             //检查 ss.ff15addr 是否为 0  
      002C040E            0F86 24010000     jbe       002C0538  
      002C0414            66:C785 F4FEFFFF >mov word ptr ss:\[ebp-10C\],0E890          //ff15.addr 不为 0，则 ss.temp1 写入 90 e8  
      002C041D            8B45 A8           mov eax,dword ptr ss:\[ebp-58\]  
      002C0420            83E8 1B           sub eax,1B  
      002C0423            8B95 78FFFFFF     mov edx,dword ptr ss:\[ebp-88\]  
      002C0429            0395 F0FEFFFF     add edx,dword ptr ss:\[ebp-110\]  
      002C042F            42                inc edx  
      002C0430            2BC2              sub eax,edx  
      002C0432            83E8 05           sub eax,5                                //通过 e8 的目的地址 ss.code1 计算偏移地址  
      002C0435            8945 84           mov dword ptr ss:\[ebp-7C\],eax  
      002C0438            6A 02             push 2  
      002C043A            8B85 78FFFFFF     mov eax,dword ptr ss:\[ebp-88\]  
      002C0440            0385 F0FEFFFF     add eax,dword ptr ss:\[ebp-110\]  
      002C0446            50                push eax  
      002C0447            8D85 F4FEFFFF     lea eax,dword ptr ss:\[ebp-10C\]  
      002C044D            50                push eax  
      002C044E            FFD6              call esi                                 //写入 90 e8  
      002C0450            6A 04             push 4  
      002C0452            8B85 78FFFFFF     mov eax,dword ptr ss:\[ebp-88\]  
      002C0458            0385 F0FEFFFF     add eax,dword ptr ss:\[ebp-110\]  
      002C045E            83C0 02           add eax,2  
      002C0461            50                push eax  
      002C0462            8D45 84           lea eax,dword ptr ss:\[ebp-7C\]  
      002C0465            50                push eax  
      002C0466            FFD6              call esi                                 //写入 e8 的偏移地址  
      002C0468            E9 CB000000       jmp       002C0538                       //当前变形的数据结构处理完毕  
  
      002C046D            8B45 A8           mov eax,dword ptr ss:\[ebp-58\]  
      002C0470            83E8 1B           sub eax,1B  
      002C0473            8985 7CFFFFFF     mov dword ptr ss:\[ebp-84\],eax  
      002C0479            6A 04             push 4  
      002C047B            8D45 84           lea eax,dword ptr ss:\[ebp-7C\]  
      002C047E            50                push eax  
      002C047F            8B45 84           mov eax,dword ptr ss:\[ebp-7C\]  
      002C0482            50                push eax  
      002C0483            FFD6              call esi  
      002C0485            6A 04             push 4  
      002C0487            8B85 7CFFFFFF     mov eax,dword ptr ss:\[ebp-84\]  
      002C048D            50                push eax  
      002C048E            8D45 84           lea eax,dword ptr ss:\[ebp-7C\]  
      002C0491            50                push eax  
      002C0492            FFD6              call esi  
      002C0494            E9 9F000000       jmp       002C0538  
      002C0499            E8 B1030000       call       002C084F  
      002C049E            8985 7CFFFFFF     mov dword ptr ss:\[ebp-84\],eax  
      002C04A4            6A 04             push 4  
      002C04A6            8B45 AC           mov eax,dword ptr ss:\[ebp-54\]  
      002C04A9            0385 D1FEFFFF     add eax,dword ptr ss:\[ebp-12F\]  
      002C04AF            50                push eax  
      002C04B0            8D85 7CFFFFFF     lea eax,dword ptr ss:\[ebp-84\]  
      002C04B6            50                push eax  
      002C04B7            FFD6              call esi  
      002C04B9            EB 7D             jmp short       002C0538  
      002C04BB            E8 BF030000       call       002C087F  
      002C04C0            8985 7CFFFFFF     mov dword ptr ss:\[ebp-84\],eax  
      002C04C6            6A 04             push 4  
      002C04C8            8B45 AC           mov eax,dword ptr ss:\[ebp-54\]  
      002C04CB            0385 D1FEFFFF     add eax,dword ptr ss:\[ebp-12F\]  
      002C04D1            50                push eax  
      002C04D2            8D85 7CFFFFFF     lea eax,dword ptr ss:\[ebp-84\]  
      002C04D8            50                push eax  
      002C04D9            FFD6              call esi  
      002C04DB            EB 5B             jmp short       002C0538  
      002C04DD            8BFB              mov edi,ebx  
      002C04DF            037D 8C           add edi,dword ptr ss:\[ebp-74\]  
      002C04E2            0FB685 D0FEFFFF   movzx eax,byte ptr ss:\[ebp-130\]  
      002C04E9            50                push eax  
      002C04EA            57                push edi  
      002C04EB            E8 9B000000       call       002C058B  
      002C04F0            8945 80           mov dword ptr ss:\[ebp-80\],eax  
      002C04F3            8B45 80           mov eax,dword ptr ss:\[ebp-80\]  
      002C04F6            0145 8C           add dword ptr ss:\[ebp-74\],eax  
      002C04F9            E8 D1000000       call       002C05CF  
      002C04FE            8945 A8           mov dword ptr ss:\[ebp-58\],eax  
      002C0501            50                push eax  
      002C0502            8B45 A8           mov eax,dword ptr ss:\[ebp-58\]  
      002C0505            C700 33C033C0     mov dword ptr ds:\[eax\],C033C033  
      002C050B            C740 04 33C0C300  mov dword ptr ds:\[eax+4\],0C3C033  
      002C0512            58                pop eax  
      002C0513            83BD D5FEFFFF 00  cmp dword ptr ss:\[ebp-12B\],0  
      002C051A            75 1C             jnz short       002C0538  
      002C051C            8BFB              mov edi,ebx  
      002C051E            037D 8C           add edi,dword ptr ss:\[ebp-74\]  
      002C0521            0FB685 D0FEFFFF   movzx eax,byte ptr ss:\[ebp-130\]  
      002C0528            50                push eax  
      002C0529            57                push edi  
      002C052A            E8 5C000000       call       002C058B  
      002C052F            8945 80           mov dword ptr ss:\[ebp-80\],eax  
      002C0532            8B45 80           mov eax,dword ptr ss:\[ebp-80\]  
      002C0535            0145 8C           add dword ptr ss:\[ebp-74\],eax  
  
  
      002C0538            6A 2C             push 2C                                
      002C053A            8D85 D0FEFFFF     lea eax,dword ptr ss:\[ebp-130\]  
      002C0540            50                push eax  
      002C0541            8BC3              mov eax,ebx  
      002C0543            0345 8C           add eax,dword ptr ss:\[ebp-74\]  
      002C0546            50                push eax  
      002C0547            FFD6              call esi                                   //将下一个 ss 数据结构 copy 入堆栈  
  
      002C0549            83BD D1FEFFFF 00  cmp dword ptr ss:\[ebp-12F\],0               //通过检测 ss.IATRva 看是否全部处理完毕  
      002C0550          ^ 0F85 27FDFFFF     jnz       002C027D                         //ss.IATRva 不为 0 则跳上去处理  
      002C0556            8B85 40FFFFFF     mov eax,dword ptr ss:\[ebp-C0\]  
      002C055C            0345 AC           add eax,dword ptr ss:\[ebp-54\]  
      002C055F            5F                pop edi  
      002C0560            5E                pop esi  
      002C0561            5B                pop ebx  
      002C0562            8BE5              mov esp,ebp  
      002C0564            5D                pop ebp  
      002C0565            C3                retn  
  
  
经过分析后，修复起来很方便，因为数据结构我们需要的信息都有  
  
1. 还原 IAT 的真实名字  
      002C034B          /E9 B0060000       jmp 002C0A00                                 //跳到自己的空间  
      002C0350          |90                nop  
  
      002C0A00           FF55 9C           call dword ptr ss:\[ebp-64\]                   //取得导入函数的地址  
      002C0A03           8945 84           mov dword ptr ss:\[ebp-7C\],eax                //原句照抄  
      002C0A06           8B95 D1FEFFFF     mov edx,dword ptr ss:\[ebp-12F\]               //ss.IATRva 取出  
      002C0A0C           0355 AC           add edx,dword ptr ss:\[ebp-54\]                //加上基址得到 IAT 的地址  
      002C0A0F           8902              mov dword ptr ds:\[edx\],eax                   //导入函数地址写入 IAT  
      002C0A11         ^ E9 3AF9FFFF       jmp 002C0350                                 //跳回去  
  
2. 还原变形的代码  
  
      002C0414           66:C785 F4FEFFFF F>mov word ptr ss:\[ebp-10C\],15FF              //原来的 90 e8 改成 ff 15  
      002C041D           8B85 D1FEFFFF      mov eax,dword ptr ss:\[ebp-12F\]              //取出相应的 ss.IATRva  
      002C0423           0345 AC            add eax,dword ptr ss:\[ebp-54\]               //得到 IAT 地址  
      002C0426           90                 nop  
      002C0427           90                 nop  
      002C0428           90                 nop  
      002C0429           90                 nop  
      002C042A           90                 nop  
      002C042B           90                 nop  
      002C042C           90                 nop  
      002C042D           90                 nop  
      002C042E           90                 nop  
      002C042F           90                 nop  
      002C0430           90                 nop  
      002C0431           90                 nop  
      002C0432           90                 nop  
      002C0433           90                 nop  
      002C0434           90                 nop  
  
  
      002C03B6           66:C785 EEFEFFFF E>mov word ptr ss:\[ebp-112\], 25FF             //e9 改成 ff 25 也是一样，不过要多动几行  
                                                                                          因为后面是 inc eax，我们需要 eax+2  
  
  
经过这样处理后，代码段都修复完成， 输入表只有一个未能识别，那个就是 MessageBoxW  
由于 netwowell 大侠没有把 mov IAT 的部分打开，所以这里就没有仔细分析  
  
谢谢大家看完 

 4dd466d0e1e0cc123afbe504da2462ca
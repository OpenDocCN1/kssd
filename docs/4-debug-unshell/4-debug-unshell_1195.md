

# yoda's cryptor 1.2

> 原文：[`www.pediy.com/kssd/pediy06/pediy6674.htm`](https://www.pediy.com/kssd/pediy06/pediy6674.htm)

两步快速脱 yoda's cryptor 1.2 壳--esp 定律和内存断点完美组合应用            
     
软件名称：  
  两步快速脱 yoda's cryptor 1.2 壳--esp 定律和内存断点完美组合应用  
     
软件简介：  
  yoda's cryptor 1.2，老壳了。公开了源码，谢谢作者无私!  
  
     
下载地址：  
  本地下载  
\------------------------------------------------------------  
     
破解作者：  
  springkang\[DFCG\]  
     
破解工具：  
  OD，loadpe,impr,winxp  
     
破解目的：  
  学习和推广 esp 定律。  
     
\------------------------------------------------------------  
     
\[破解过程\]  
     
详细过程：  
首先，忽略所有异常，再载入加壳的 winxp 记事本。  
  
01013060 >  60              PUSHAD       //载入加壳程序后停在这里。首先使用 esp 定律!  
01013061    E8 00000000     CALL NOTEPAD.01013066   //esp 为 0006ffa4  
01013066    5D              POP EBP    //esp 为 0006ffa0    
01013067    81ED F31D4000   SUB EBP,401DF3    //esp 为 0006ffa4,从这里开始运行几步 esp 的值均无变化。好了，可以在这里下 0006ffa4 硬件访问--word(dword)断点了，也可以再走几步下断，问题不大。F9 运行。  
0101306D    B9 7B090000     MOV ECX,97B  
01013072    8DBD 3B1E4000   LEA EDI,DWORD PTR SS:\[EBP+401E3B\]  
  
  
0101375D    50              PUSH EAX                                 ; NOTEPAD.0101370C    //中断到这里了，取消硬件断点，下内存访问断点。  
0101375E    33C0            XOR EAX,EAX  
01013760    64:FF30         PUSH DWORD PTR FS:\[EAX\]  
01013763    64:8920         MOV DWORD PTR FS:\[EAX\],ESP  
01013766    EB 01           JMP SHORT NOTEPAD.01013769  
01013768    8700            XCHG DWORD PTR DS:\[EAX\],EAX  
0101376A    0000            ADD BYTE PTR DS:\[EAX\],AL  
0101376C    0000            ADD BYTE PTR DS:\[EAX\],AL  
  
内存镜像，项目 27  
 地址=01001000  
 大小=00007000 (28672.)  
 Owner=NOTEPAD  01000000  
 区段=.text  
 包含=code   //老规矩了，在这里下内存断点  ，f9 运行  
 类型=Imag 01001008  
 访问=RW  CopyOnWr  
 初始访问=RWE  
  
  
01006AE0    6A 70           PUSH 70      //很脸熟吧，用 loadpe 和 impr 就可以 dump 和修复了，运行正常。  
01006AE2    68 88180001     PUSH NOTEPAD.01001888  
01006AE7    E8 BC010000     CALL NOTEPAD.01006CA8  
01006AEC    33DB            XOR EBX,EBX  
01006AEE    53              PUSH EBX  
  
最后补充一下，如果用 OD 的插件 dump 加壳的 yoda's cryptor 1.2 主程序，impr 修复，虽然运行正常，但无法加密。而用 loadpe 的 dump 再修复就没有这种问题。具体原因偶是个大菜鸟，也无法得知。  
  
  
  
     
破解小结：  
  此法对加壳的主程序同样有效! 下硬件断点的地址有很多，如 0012ffa0,12ffc0,12ffac,12ffa4 等，不一而足,不同的壳有不同的下法。关键是看载入加壳的程序后运行几步观察寄存器的 esp 的值，多试几次，就会有收获。例如 petite2.2 的壳就是下在 0012ffc0 处。  
  最后感谢 weiyi75,fly,loveboom(偶的老乡)，shinegood,forgot,temerata 等等高手，还有 DFCG 的我要大哥，你们的文章给予我很多帮助，谢谢你们!!!  
     
\------------------------------------------------------------  
     
版权公有，人权私有。如有转载，注明作者。

 4dd466d0e1e0cc123afbe504da2462ca
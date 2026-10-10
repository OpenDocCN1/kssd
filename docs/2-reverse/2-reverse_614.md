

# 神泣

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-518.htm`](https://www.pediy.com/kssd/pediy08/pediy8-518.htm)

【文章标题】: 神泣逆向工程—我的处女破文  
【文章作者】: BUG  
【作者邮箱】: \*\*\*\*\*\*\*\*  
【作者 QQ 号】: \*\*\*\*\*\*\*\*  
【软件名称】: 神泣  
【下载地址】: 自己搜索下载  
【加壳方式】: Asprotect  
【编写语言】: VC++  
【使用工具】: PEID,OD,IDA 5.0  
【操作平台】: WINDOWS XP  
【软件介绍】: 神泣是光通代理的一款韩国泡菜式游戏  
【作者声明】: 最近国内对游戏破解开发搞得有点严,本来我也不想发布的,但是这是我的第一次:),不管怎样也得记录一下嘛,失误之处还请各位高手指正,谢谢!  
\--------------------------------------------------------------------------------  
【前言】  
     破解此游戏并非用在商业用途，只是想自己脱机外挂升升级，让电脑自己去跟电脑说话，我就可以静心研究了，不过也是通过这一次逆向，让我感觉进步了不少。  
     这是我第一写破文，还请大家批评指正！  
【详细过程】  
      首先是脱壳,这个游戏是 Asprotect 的壳子，我现在还是新手，要不是有 Asprotect 的脱壳机，我想我在这一关上又得研究好几天了。脱掉它的"外衣"之后,就开始对它逆向工程了，要分析发送数据的来源，就得对 send 下断点，然后输入帐号密码，登陆。这时断点断在了 WS\_32 模块,回到用户领空,  
  到达这里:  
  0051B740  /$  B8 00100000   MOV EAX,1000  
  0051B745  |.  E8 A68C0200   CALL \_game.005443F0  
  0051B74A  |.  53            PUSH EBX  
  0051B74B  |.  8B9C24 0C1000>MOV EBX,DWORD PTR SS:\[ESP+100C\]  
  0051B752  |.  8BCB          MOV ECX,EBX  
  0051B754  |.  55            PUSH EBP  
  0051B755  |.  56            PUSH ESI  
  0051B756  |.  8BB424 101000>MOV ESI,DWORD PTR SS:\[ESP+1010\]  
  0051B75D  |.  8BC1          MOV EAX,ECX  
  0051B75F  |.  57            PUSH EDI  
  0051B760  |.  8D6B 02       LEA EBP,DWORD PTR DS:\[EBX+2\]  
  0051B763  |.  8D7C24 12     LEA EDI,DWORD PTR SS:\[ESP+12\]  
  0051B767  |.  C1E9 02       SHR ECX,2  
  0051B76A  |.  66:896C24 10  MOV WORD PTR SS:\[ESP+10\],BP  
  0051B76F  |.  53            PUSH EBX  
  0051B770  |.  F3:A5         REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS>  
  0051B772  |.  8BC8          MOV ECX,EAX  
  0051B774  |.  83E1 03       AND ECX,3  
  0051B777  |.  F3:A4         REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[>  
  0051B779  |.  8D4C24 16     LEA ECX,DWORD PTR SS:\[ESP+16\]  
  0051B77D  |.  51            PUSH ECX  
  0051B77E  |.  E8 2DB5FFFF   CALL \_game.00516CB0                      ;  加密函数  
  0051B783  |.  A1 A0661002   MOV EAX,DWORD PTR DS:\[21066A0\]  
  0051B788  |.  83C4 08       ADD ESP,8  
  0051B78B  |.  83C3 02       ADD EBX,2  
  0051B78E  |.  8D5424 10     LEA EDX,DWORD PTR SS:\[ESP+10\]  
  0051B792  |.  6A 00         PUSH 0                                   ; /Flags = 0  
  0051B794  |.  53            PUSH EBX                                 ; |DataSize  
  0051B795  |.  52            PUSH EDX                                 ; |Data  
  0051B796  |.  50            PUSH EAX                                 ; |Socket => EC  
  0051B797  |.  FF15 D8135B00 CALL DWORD PTR DS:\[<&ws2\_32.send>\]       ; \\send  
    
  这里还挺近的,PUSH ECX 就是原始数据,  
  这以后我就用 IDA 看了,动态调试时用 OD,静态分析还是 IDA 强大,  
  跟进\_game.00516CB0 到达:  
    
  .text:00516CB0 sub\_516CB0      proc near               ; CODE XREF: sub\_51B740+3Ep  
  .text:00516CB0  
  .text:00516CB0 arg\_0           = dword ptr  4  
  .text:00516CB0 arg\_4           = dword ptr  8  
  .text:00516CB0  
  .text:00516CB0                 mov     al, ds:byte\_2100685  
  .text:00516CB5                 test    al, al  
  .text:00516CB7                 jz      short locret\_516CEB  
  .text:00516CB7  
  .text:00516CB9                 mov     eax, ds:dword\_210067C  
  .text:00516CBE                 test    eax, eax  
  .text:00516CC0                 jnz     short loc\_516CD7  
  .text:00516CC0  
  .text:00516CC2                 mov     eax, \[esp+arg\_4\]  
  .text:00516CC6                 mov     ecx, \[esp+arg\_0\]  
  .text:00516CCA                 push    eax  
  .text:00516CCB                 push    ecx  
  .text:00516CCC                 mov     ecx, offset unk\_20FF4D8  
  .text:00516CD1                 call    sub\_517CC0                          ;先不管那 2 个全  
  
局变量地址判断了什么,发送登陆封包是通过了这个函数  
  .text:00516CD1  
  .text:00516CD6                 retn  
  .text:00516CD6  
  .text:00516CD7 ; -----------------------------------------------------------------------  
  
\----  
  .text:00516CD7  
  .text:00516CD7 loc\_516CD7:                             ; CODE XREF: sub\_516CB0+10j  
  .text:00516CD7                 mov     edx, \[esp+arg\_4\]  
  .text:00516CDB                 mov     eax, \[esp+arg\_0\]  
  .text:00516CDF                 push    edx  
  .text:00516CE0                 push    eax  
  .text:00516CE1                 mov     ecx, offset unk\_20FF628  
  .text:00516CE6                 call    sub\_518000  
  .text:00516CE6  
  .text:00516CEB  
  .text:00516CEB locret\_516CEB:                          ; CODE XREF: sub\_516CB0+7j  
  .text:00516CEB                 retn  
  .text:00516CEB  
  .text:00516CEB sub\_516CB0      endp  
    
  分析可知 arg\_4 = 长度,arg\_0 为原始内容,unk\_20FF4D8 就是一个偏移量,从代码上看来,我猜测它是 this 指针,类是一个全局变量,所以直接赋值了一个偏移量,在后面的代码上看来我的分析是正确的.  
    
  .........  
  以上是水话.现在说点实在的.  
  最终到达 sub\_517B20:  
  .text:00517B20 sub\_517B20      proc near               ; CODE XREF: sub\_517CC0+Bp  
  .text:00517B20  
  .text:00517B20 dest            = dword ptr  10h  
  .text:00517B20 src             = dword ptr  14h  
  .text:00517B20 datalen         = dword ptr  18h  
  .text:00517B20  
  .text:00517B20                 push    ebx  
  .text:00517B21                 push    ebp  
  .text:00517B22                 push    esi  
  .text:00517B23                 mov     esi, ecx        ; esi = this 指针  
  .text:00517B25                 xor     ebp, ebp  
  .text:00517B27                 push    edi  
  .text:00517B28                 mov     edx, \[esi+104h\]  
  .text:00517B2E                 cmp     edx, ebp  
  .text:00517B30                 jle     else            ; 如果 esi+104h <= 0 就跳到另一个分支  
  .text:00517B30  
  .text:00517B36                 mov     ecx, \[esp+4+datalen\]  
  .text:00517B3A                 mov     eax, 10h  
  .text:00517B3F                 sub     eax, edx  
  .text:00517B41                 cmp     ecx, eax  
  .text:00517B43                 jg      short \_len\_jg\_else  
  .text:00517B43  
  .text:00517B45                 mov     eax, ecx  
  .text:00517B47                 dec     ecx  
  .text:00517B48                 test    eax, eax  
  .text:00517B4A                 jz      short \_small\_return  
  .text:00517B4A  
  .text:00517B4C                 mov     eax, \[esp+4+dest\]  
  .text:00517B50                 lea     edi, \[ecx+1\]  
  .text:00517B53                 mov     ecx, \[esp+4+src\]  
  .text:00517B53  
  .text:00517B57  
  .text:00517B57 \_loc\_for:                               ; CODE XREF: sub\_517B20+5Aj  
  .text:00517B57                 mov     edx, \[esi+104h\]  
  .text:00517B5D                 mov     bl, \[ecx\]  
  .text:00517B5F                 mov     dl, \[esi+edx+108h\]  
  .text:00517B66                 xor     dl, bl  
  .text:00517B68                 mov     \[eax\], dl  
  .text:00517B6A                 mov     ebx, \[esi+104h\]  
  .text:00517B70                 inc     ebx  
  .text:00517B71                 inc     eax  
  .text:00517B72                 inc     ecx  
  .text:00517B73                 dec     edi  
  .text:00517B74                 mov     \[esi+104h\], ebx  
  .text:00517B7A                 jnz     short \_loc\_for  
  .text:00517B7A  
  .text:00517B7C  
  .text:00517B7C \_small\_return:                          ; CODE XREF: sub\_517B20+2Aj  
  .text:00517B7C                 cmp     dword ptr \[esi+104h\], 10h  
  .text:00517B83                 jl      \_loc\_return  
  .text:00517B83  
  .text:00517B89                 mov     \[esi+104h\], ebp  
  .text:00517B8F                 pop     edi  
  .text:00517B90                 pop     esi  
  .text:00517B91                 pop     ebp  
  .text:00517B92                 pop     ebx  
  .text:00517B93                 retn    0Ch  
  .text:00517B93  
  .text:00517B96 ; -----------------------------------------------------------------------  
  
\----  
  .text:00517B96  
  .text:00517B96 \_len\_jg\_else:                           ; CODE XREF: sub\_517B20+23j  
  .text:00517B96                 mov     ebx, \[esp+4+src\]  
  .text:00517B9A                 mov     edi, \[esp+4+dest\]  
  .text:00517B9E                 sub     ecx, eax  
  .text:00517BA0                 cmp     edx, 10h  
  .text:00517BA3                 mov     \[esp+4+datalen\], ecx  
  .text:00517BA7                 jge     short loc\_517BD0  
  .text:00517BA7  
  .text:00517BA9  
  .text:00517BA9 \_loc\_for\_1:                             ; CODE XREF: sub\_517B20+AEj  
  .text:00517BA9                 mov     eax, \[esi+104h\]  
  .text:00517BAF                 mov     dl, \[esi+eax+108h\]  
  .text:00517BB6                 mov     al, \[ebx\]  
  .text:00517BB8                 xor     dl, al  
  .text:00517BBA                 mov     \[edi\], dl  
  .text:00517BBC                 mov     eax, \[esi+104h\]  
  .text:00517BC2                 inc     eax  
  .text:00517BC3                 inc     edi  
  .text:00517BC4                 inc     ebx  
  .text:00517BC5                 cmp     eax, 10h  
  .text:00517BC8                 mov     \[esi+104h\], eax  
  .text:00517BCE                 jl      short \_loc\_for\_1  
  .text:00517BCE  
  .text:00517BD0  
  .text:00517BD0 loc\_517BD0:                             ; CODE XREF: sub\_517B20+87j  
  .text:00517BD0                 mov     \[esi+104h\], ebp  
  .text:00517BD6                 jmp     short loc\_517BE4  
  .text:00517BD6  
  .text:00517BD8 ; -----------------------------------------------------------------------  
  
\----  
  .text:00517BD8  
  .text:00517BD8 else:                                   ; CODE XREF: sub\_517B20+10j  
  .text:00517BD8                 mov     ebx, \[esp+4+src\]  
  .text:00517BDC                 mov     edi, \[esp+4+dest\]  
  .text:00517BE0                 mov     ecx, \[esp+4+datalen\]  
  .text:00517BE0  
  .text:00517BE4  
  .text:00517BE4 loc\_517BE4:                             ; CODE XREF: sub\_517B20+B6j  
  .text:00517BE4                 cmp     ecx, 10h  
  .text:00517BE7                 jl      short loc\_517C58  
  .text:00517BE7  
  .text:00517BE9                 mov     ebp, ecx  
  .text:00517BEB                 shr     ebp, 4  
  .text:00517BEE                 mov     eax, ebp  
  .text:00517BF0                 neg     eax  
  .text:00517BF2                 shl     eax, 4  
  .text:00517BF5                 add     ecx, eax  
  .text:00517BF7                 mov     \[esp+4+datalen\], ecx  
  .text:00517BF7  
  .text:00517BFB  
  .text:00517BFB loc\_517BFB:                             ; CODE XREF: sub\_517B20+132j  
  .text:00517BFB                                         ; 这个循环就是加密了.  
  .text:00517BFB                 lea     eax, \[esi+0F4h\] ; DWORD m\_key1\[4\]  
  .text:00517C01                 mov     ecx, esi        ; KeyCode 函数里面会压入 this 指针进入真正执行操作的函数  
  .text:00517C03                 push    eax  
  .text:00517C04                 lea     eax, \[esi+108h\] ; DWORD m\_key2\[4\]  
  .text:00517C0A                 push    eax  
  .text:00517C0B                 call    KeyCode         ; 这个函数是通过循环将 edx,eax,ecx(ecx 这里虽然是一个指针,其实是指向第一个成员变量)  
  .text:00517C0B                                         ; 这个函数太长,我写程序时是用的内嵌汇编,我写这篇文章时正在还原这个函数.  
  .text:00517C0B  
  .text:00517C10                 mov     ecx, esi  
  .text:00517C12                 call    convKey         ; 呵呵,小动作.把一个 KEY 数组的值全部加 1  
  .text:00517C12  
  .text:00517C17                 mov     ecx, \[ebx\]  
  .text:00517C19                 mov     edx, \[esi+108h\]  
  .text:00517C1F                 xor     ecx, edx  
  .text:00517C21                 add     ebx, 10h  
  .text:00517C24                 mov     \[edi\], ecx  
  .text:00517C26                 mov     edx, \[ebx-0Ch\]  
  .text:00517C29                 mov     eax, \[esi+10Ch\]  
  .text:00517C2F                 add     edi, 10h  
  .text:00517C32                 xor     edx, eax  
  .text:00517C34                 mov     \[edi-0Ch\], edx  
  .text:00517C37                 mov     eax, \[ebx-8\]  
  .text:00517C3A                 xor     eax, \[esi+110h\]  
  .text:00517C40                 mov     \[edi-8\], eax  
  .text:00517C43                 mov     ecx, \[ebx-4\]  
  .text:00517C46                 mov     eax, \[esi+114h\]  
  .text:00517C4C                 xor     ecx, eax  
  .text:00517C4E                 dec     ebp  
  .text:00517C4F                 mov     \[edi-4\], ecx  
  .text:00517C52                 jnz     short loc\_517BFB   
  .text:00517C52  
  .text:00517C54                 mov     ecx, \[esp+4+datalen\]  
  .text:00517C54  
  .text:00517C58  
  .text:00517C58 loc\_517C58:                             ; CODE XREF: sub\_517B20+C7j  
  .text:00517C58                 test    ecx, ecx  
  .text:00517C5A                 jle     short \_loc\_return  
  .text:00517C5A  
  .text:00517C5C                 lea     edx, \[esi+0F4h\]  
  .text:00517C62                 lea     eax, \[esi+108h\]  
  .text:00517C68                 push    edx  
  .text:00517C69                 push    eax  
  .text:00517C6A                 mov     ecx, esi  
  .text:00517C6C                 call    KeyCode  
  .text:00517C6C  
  .text:00517C71                 mov     ecx, esi  
  .text:00517C73                 call    convKey  
  .text:00517C73  
  .text:00517C78                 mov     ecx, \[esp+4+datalen\]  
  .text:00517C7C                 mov     eax, \[esi+104h\]  
  .text:00517C82                 cmp     eax, ecx  
  .text:00517C84                 jge     short \_loc\_return  
  .text:00517C84  
  .text:00517C86  
  .text:00517C86 loc\_517C86:                             ; CODE XREF: sub\_517B20+18Cj  
  .text:00517C86                 mov     edx, \[esi+104h\]  
  .text:00517C8C                 mov     al, \[esi+edx+108h\]  
  .text:00517C93                 mov     dl, \[ebx\]  
  .text:00517C95                 xor     al, dl  
  .text:00517C97                 mov     \[edi\], al  
  .text:00517C99                 mov     ebp, \[esi+104h\]  
  .text:00517C9F                 inc     ebp  
  .text:00517CA0                 inc     edi  
  .text:00517CA1                 mov     eax, ebp  
  .text:00517CA3                 inc     ebx  
  .text:00517CA4                 cmp     eax, ecx  
  .text:00517CA6                 mov     \[esi+104h\], ebp  
  .text:00517CAC                 jl      short loc\_517C86  
  .text:00517CAC  
  .text:00517CAE  
  .text:00517CAE \_loc\_return:                            ; CODE XREF: sub\_517B20+63j  
  .text:00517CAE                                         ; sub\_517B20+13Aj  
  .text:00517CAE                                         ; sub\_517B20+164j  
  .text:00517CAE                 pop     edi  
  .text:00517CAF                 pop     esi  
  .text:00517CB0                 pop     ebp  
  .text:00517CB1                 pop     ebx  
  .text:00517CB2                 retn    0Ch  
  .text:00517CB2  
  .text:00517CB2 sub\_517B20      endp  
    
  代码太多,我又很懒,我主要还原一下 loc\_517BFB 加密这个循环部分吧。  
  要逆向这个游戏涉及的地方太多，又不能一一贴出，我讲解一下我分析出来的结果好了  
    
  class CCode//20FF4D8  
  {  
    BYTE m\_keydata\[240\];//这是一个 KEY 表，加密的条件之 1  
    //+F0  
    int m\_num;//这个还不知道它的完全用途  
    //+F4  
    DWORD m\_key1\[4\];//这是个 DWORD 数组，加密的条件之 1  
    //+104  
    DWORD m\_nopnum;//现在仅知道通过它来判断加密还是解密，不过从值上来看不像是它的完全用途  
    //+108  
    DWORD m\_key2\[4\];//这是个 DWORD 数组，加密的条件之 1  
  //----------------------------------------------------------  
  这里是 loc\_517BFB 还原出来的代码  
    
    else//加密  
    {  
      if( len >= 10 )  
      {  
        int nnum;  
        \_\_asm  
        {  
          mov    eax,len;  
          shr    eax,4  
          mov    nnum,eax  
          neg    eax  
          shl    eax,4  
          add    len,eax  
        }  
        for( ; nnum > 0; nnum-- )//一次循环加密 16 字节，所以上面会 shr eax,4  
        {  
          KeyCode(m\_key2,m\_key1);//每一次循环都会变换 m\_key1,m\_key2 和 m\_keydata 的值(m\_keydata 的值是否变化有待深入考究)  
          convKey();  
          DWORD tmp;  
          memcpy(&tmp,( src ),sizeof(DWORD));  
          tmp ^= m\_key2\[0\];  
          \*(DWORD\*)( dest ) = tmp;  
    
          memcpy(&tmp,( src + 4 ),sizeof(DWORD));  
          tmp ^= m\_key2\[1\];  
          \*(DWORD\*)( dest + 4) = tmp;  
    
          memcpy(&tmp,( src + 8 ),sizeof(DWORD));  
          tmp ^= m\_key2\[2\];  
          \*(DWORD\*)( src + 8) = tmp;  
    
          memcpy(&tmp,( src + 12 ),sizeof(DWORD));  
          tmp ^= m\_key2\[3\];  
          \*(DWORD\*)( dest + 12) = tmp;  
    
          src += 16;  
          dest += 16;  
        }  
      }  
    
  这就是加密的全过程了，解密部分我也还原出来了，但是这里还是不贴的好，以免某些人搞破坏。  
  虽然这里加密就完了，其实远远还没这么简单，上面提到加密需要的三个数据 m\_key1,m\_key2,m\_keydata。m\_keydata 和 m\_key1 会在游戏开始时初始化，首先，连接游戏服务器，连接成功后服务端会主动热情的发送给你一个 KEY 封包，它的结构是这样子的：  
  //消息头结构  
  typedef struct \_MsgHeader  
  {  
    WORD  len;  
    WORD  cmd;  
    \_MsgHeader()  
    {  
      len = 0;  
      cmd = 0x0;  
    }  
  }MsgHeader;  
    
  //接收密钥结构  
  typedef struct \_MsgKey  
  {  
    BYTE  d1;  
    BYTE  d2;  
    BYTE  d3;  
    BYTE  key1\[0x40\];  
    BYTE  key2\[0x80\];  
  }MsgKey;  
    
  MsgKey 的长度 = C7-MsgHeader  
    
  然后通过这个 KEY 就可以变形得出 m\_keydata 和 key1 的值  
    
  我的表达能力不是很好，写到这里的时候，已经用了我 1 个多小时的时间了，深切的感到自己语文成绩之差，这里我已经不好描述了，所以我还是贴代码描述好了，抱歉！  
    
  从 DeCodeKey 函数开始  
    
  DWORD dword\_210067C;// =dekey2 中的 d1,只有这次写入操作;  
  DWORD dword\_2100688 = 0x456FC070;//本来是通过时间来生成一个 KEY,只有这次写入操作,应该不会有副本做判断,直接赋值常量可能也能成功  
  
  CCode::CCode()  
  {  
    //m\_num = 0x0a;  
    //memcpy((void\*)m\_key1,"\\x81\\x28\\x59\\x9E\\x00\\xBD\\xC7\\xB7\\xD1\\xBB\\x9F\\xF1\\xA8\\x37\\xD8\\xE2",16);  
    //m\_nopnum = 0;  
    //strcpy((char\*)m\_key2,"");  
  }  
  CCode::~CCode()  
  {  
  }  
    
  //设置 KEY  
  //data : 接收的密钥封包去掉头 2 字节  
  void CCode::DeCodeKey(BYTE\* data)//接受到的 KEY 封包  
  {  
    MsgKey msgkey;  
    msgkey.d1 = (BYTE)data\[0\];  
    msgkey.d2 = (BYTE)data\[2\];  
    msgkey.d3 = (BYTE)data\[1\];  
    memcpy( msgkey.key1, &data\[0x3\],0x40 );//复制到结构里去  
    memcpy( msgkey.key2, &data\[0x43\],0x80 );  
    dekey1(msgkey.d1,msgkey.d3,msgkey.d2,msgkey.key1,msgkey.key2 );  
  }  
    
  void CCode::dekey1( BYTE d1,BYTE d2,BYTE d3,BYTE\* key1,BYTE\* key2 )//dekey1 在游戏程序中是一个虚函数  
  {  
    DWORD var\_84 = 0;  
    BYTE var\_80\[0x80\];  
    
    dekey2(var\_80,&var\_84,d1 & 0xFF,key1,d2 & 0xFF,key2,d3 & 0xFF);  
  }  
  //一些无关痛痒的结构  
  typedef struct \_Unstname//暂时不知道干什么的,dekey2 中把这个结构进行变形  
  {  
    BYTE var\_8C\[0x7F\];  
    BYTE var\_D;  
    \_Unstname()  
    {  
      memset(var\_8C,0,0x80);//一次性把 var\_D 也刷掉  
    }  
  }Unstname;  
    
  DWORD d\_dword\_210065C\[4\];  
  void CCode::dekey2(OUT BYTE\* var\_80,OUT DWORD\* var\_84,BYTE d1,BYTE\* key1,BYTE d2,BYTE\* key2,BYTE d3)  
  {  
    //DWORD var\_C4 = 0;  
    //DWORD var\_C0 = 0;  
    //DWORD var\_BC = 0;  
    //DWORD var\_B8 = 0;  
    //DWORD var\_B4 = 0;  
    //DWORD var\_B0 = 0;  
    //DWORD var\_AC = 0;  
    //DWORD var\_A8 = 0;  
    DWORD var\_C4\[8\] = {0};  
    BYTE var\_A4\[0x24\];  
    Unstname unstname;  
    BYTE var\_C\[0x8\];  
    DWORD var\_4;  
    
    dword\_210067C = d1;  
    outkey80((BYTE\*)&unstname,sizeof(Unstname)/\*0x80\*/);//这个函数通过了 dword\_2100688 变  
  
量来生成一个 80 字节的数据，那个变量是跟时间有关的数据变形得到的，不过我用常量好象也能使用，待考究  
      
    unstname.var\_D &= 0x7F;//还原代码而已，还没看到用途  
    
    SHA256(var\_C4,(unsigned char \*)&unstname,0x80,key2,d3);//这里就通过 SHA 算法生成一个数据  
                                                                 //SHA256 函数里面我就直接按游戏中的加密顺序套用了标准函数了  
    
    if( dword\_210067C == 0 )  
    {  
      //快到 InitKey 函数了...快打通了..  
      InitKey(var\_C4,0x10);//变形得到 m\_keydata.  
      d\_dword\_210065C\[0\] = var\_C4\[4\];  
      d\_dword\_210065C\[1\] = var\_C4\[5\];  
      d\_dword\_210065C\[2\] = var\_C4\[6\];  
      d\_dword\_210065C\[3\] = var\_C4\[7\];  
      setkey1(d\_dword\_210065C);//设置 m\_key1 的值  
    }  
    else  
    {  
    }  
  }  
  void CCode::setkey1(DWORD key\[4\])  
  {  
    m\_key1\[0\] = key\[0\];  
    m\_key1\[1\] = key\[1\];  
    m\_key1\[2\] = key\[2\];  
    m\_key1\[3\] = key\[3\];  
    //memcpy(m\_key1,key,0x10);  
    m\_nopnum = 0;  
  }   
  void CCode::SHA256(DWORD\* var\_C4,unsigned char \*input,DWORD len,BYTE\* key2,BYTE d3)  
  {  
    //BYTE var\_114\[0x68\];//可能是 0x28+0x40  
    sha256\_context ctx;  
    BYTE sdata1\[0x40\];  
    BYTE sdata2\[0x40\];  
    BYTE var\_sha\_outstr\[32\];  
    BYTE var\_C\[8\];//跟异常有关  
    DWORD var\_4;//完全没用到  
    
    //call    sub\_517E80 //关于 var\_114  
    var\_4 = 0;  
    
    BYTE\* s = NULL;  
    if(len > 0x40)  
    {  
      sha256\_starts(&ctx);  
      sha256\_update(&ctx,input,len);  
      sha256\_finish(&ctx,var\_sha\_outstr);  
    
      s = var\_sha\_outstr;  
    }  
    else  
    {  
      s = input;  
    }  
    memset(sdata1,0,0x40);  
    memset(sdata2,0,0x40);  
    
    memcpy(sdata1, s, 32 );  
    memcpy(sdata2, s, 32 );  
      
    for(int idx = 0; idx < 0x40; idx++)  
    {  
      sdata1\[idx\] = sdata1\[idx\] ^ 0x36;  
      sdata2\[idx\] = sdata2\[idx\] ^ 0x5C;    
    }  
      
    sha256\_starts( &ctx );  
    sha256\_update( &ctx, (uint8 \*) sdata1, 0x40 );  
    sha256\_update( &ctx, (uint8 \*) key2, d3 );  
    sha256\_finish( &ctx, (uint8 \*)var\_C4);  
    
    sha256\_starts( &ctx );              
    sha256\_update( &ctx, (uint8 \*) sdata2, 0x40 );   
    sha256\_update( &ctx, (uint8 \*) var\_C4, 0x20 );   
    sha256\_finish( &ctx, (uint8 \*)var\_C4);  
    
  }  
    
  //初始化 KEY 表  
  //addr:可能是通过密钥封包运算出来的  
  void CCode::InitKey(DWORD\* addr,int num)  
  {  
    InitKeys(addr,num\*8,this);//汇编代码.....太长了加我太懒了..  
  }  
    
\--------------------------------------------------------------------------------  
【经验总结】  
  通过这次逆向，熟悉了 OD 和 IDA，终于对破解和逆向有了一点经验。  
  为了迎接即将到来的 17 岁年华，所以先写人生的第一篇破文鼓励自己，多谢大家支持！：）  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 12 月 03 日 17:43:21\[CODE\] 

 4dd466d0e1e0cc123afbe504da2462ca
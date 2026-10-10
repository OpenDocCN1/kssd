

# QQ 自动登录器

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-472.htm`](https://www.pediy.com/kssd/pediy08/pediy8-472.htm)

【文章标题】 : 逆向小试 —— QQ 自动登录器原理分析  
【调试环境】 : WINXP OllyDbg1.10    
【软件名称】 : QQ 自动登录器  
【调试目的】 : 学习逆向  
【文章作者】 : figo  
【作者声明】 : 纯技术交流，没有其他目的，不针对任何软件。失误之处恳请批评指正，或有更好的方法或者技巧，欢迎互相交流。    
【附件下载】 : 下载  
【分析过程】 ;  
  
QQ 自动登录器是一款很不错的操纵多 QQ 帐号的软件。  
分析这款软件是因为整理以前的光盘是无意发现的，刚好有同学要我帮他写个 QQ 木马，  
于是顺便把它调试了一下，不多说了，进入正题吧：  
  
  
我们先运行一下程序，添加一个自动登录的 QQ 帐号。设 QQ 号为 382174647，密码为 123123。  
  
再将所有的 WinExec 函数置断点，出现了 QQ 自动登录器的主界面，启动帐号 382174647，这时程序中断在如下代码：  
  
00404B20   .  6A 00         PUSH 0                                      ; /ShowState = SW\_HIDE  
00404B22   .  52            PUSH EDX                                    ; |CmdLine   
00404B23   .  FF15 38804000 CALL DWORD PTR DS:\[<&KERNEL32.WinExec>\]     ; \\WinExec  
  
这时 EDX 指向的字符串内容为：   
"C:\\Program Files\\Tencent\\QQ\\qq.exe /START QQUIN:382174647 PWDHASH:Qpf0SxOVUjUkWySXOZ16kw== /STAT:41"  
  
上面的代码还是比较好理解的，唯一需要讲解的是 EDX 所指向的字符串是怎样生成的。  
  
将代码页上翻，可以发现不远出有调用 CString.Format 函数的代码，入栈参数为字符串：  
   
"%s /START QQUIN:%d PWDHASH:%s /STAT:%d"  
  
不难分析出:  
第一个 %s 为 QQ.EXE  的路径.  
第一个 %d 为 QQ 号码.  
第二个 %s 为 QQ 密码的密文  
第二个 %d 为 QQ 登陆状态 ， 41 是代表正常登陆,40 是隐藏登陆  
  
  
下面分析一下密文的生成：  
  
程序目录下有个 LoginData.dat，用于保存 QQ 自动登录数据的 ，由于程序必会读取该文件的，所以  
先把所有的 CFile.Read 置断点，然后运行程序，程序中断在如下代码：  
  
  
  
00402731  |.  6A 10         PUSH 10  
00402733  |.  50            PUSH EAX  
00402734  |.  8D4C24 20     LEA ECX,DWORD PTR SS:\[ESP+20\]  
00402738  |.  E8 1F3E0000   CALL <JMP.&MFC42.#5442\_?Read@CFile@@UAEI>;  读文件头, LoginData.dat 的前 10H 个字节  
0040273D  |>  817C24 08 000>CMP DWORD PTR SS:\[ESP+8\],10400           ;  校验 LoginData.dat 文件的有效性  
00402745  |.  74 3C         JE SHORT QAL1\_4\_1.00402783         ;   偏移量 0 的值固定为 10400H  
  
  
  
0040468A  |.  6A 10         PUSH 10  
0040468C  |.  50            PUSH EAX  
0040468D  |.  8D4C24 1C     LEA ECX,DWORD PTR SS:\[ESP+1C\]  
00404691  |.  E8 C61E0000   CALL <JMP.&MFC42.#5442\_?Read@CFile@@UAEIPAXI@Z>    ;  读文件头  
00404696  |.  396C24 2C     CMP DWORD PTR SS:\[ESP+2C\],EBP                      ;  已记录的 QQ 数，位于偏移量 08H  
0040469A  |.  75 34         JNZ SHORT QAL1\_4\_1.004046D0  
  
  
00404759  |.  6A 40         |PUSH 40  
0040475B  |.  50            |PUSH EAX  
0040475C  |.  8D4C24 1C     |LEA ECX,DWORD PTR SS:\[ESP+1C\]  
00404760  |.  E8 F71D0000   |CALL <JMP.&MFC42.#5442\_?Read@CFile@@UAEIPAXI@Z>   ;  读取每个 QQ 的登录数据  
00404765  |.  83F8 40       |CMP EAX,40                                        ;  QQ 登录数据块固定尺寸为 40H  
00404768  |.  0F85 E8000000 |JNZ QAL1\_4\_1.00404856  
  
这时候 D ESP + 34 来查看 QQ 登录数据块的内容，数据如下：  
  
0012F6D8  E8 8A 40 00 01 00 00 00 D6 07 09 00 01 00 12 00  鑺@....?....  
0012F6E8  17 00 1A 00 09 00 00 00 29 00 00 00 42 97 F4 4B  ......)...B 楐 K  
0012F6F8  13 95 52 35 24 5B 24 97 39 9D 7A 93 B7 85 C7 16  昍 5$\[$?漽摲吳  
0012F708  33 38 32 31 37 34 36 34 37 00 00 00 00 00 00 00  382174647.......  
  
大家注意到了没？ 地址 0012F6F4 处的内容 ：  
  
42 97 F4 4B 13 95 52 35 24 5B 24 97 39 9D 7A 93  
  
这正是密码 “123123” 的标准 MD5 码。  
  
我们把 0012F6F4 处的内容设硬件断点，再运行程序，程序中断在如下代码：  
  
0040478B  |.  8B4C24 50     |MOV ECX,DWORD PTR SS:\[ESP+50\]           ;  ESP + 50 值为 0012F6F4  
0040478F  |.  8D442A 1C     |LEA EAX,DWORD PTR DS:\[EDX+EBP+1C\]       ;  \[EDX + EBP + 1C\] 指向的值为 0038F2C8  
00404793  |.  894C2A 1C     |MOV DWORD PTR DS:\[EDX+EBP+1C\],ECX  
00404797  |.  8B5424 54     |MOV EDX,DWORD PTR SS:\[ESP+54\]  
0040479B  |.  8950 04       |MOV DWORD PTR DS:\[EAX+4\],EDX  
0040479E  |.  8B4C24 58     |MOV ECX,DWORD PTR SS:\[ESP+58\]  
004047A2  |.  8948 08       |MOV DWORD PTR DS:\[EAX+8\],ECX  
004047A5  |.  8B5424 5C     |MOV EDX,DWORD PTR SS:\[ESP+5C\]  
004047A9  |.  83C9 FF       |OR ECX,FFFFFFFF  
004047AC  |.  8950 0C       |MOV DWORD PTR DS:\[EAX+C\],EDX  
004047AF  |.  8B43 60       |MOV EAX,DWORD PTR DS:\[EBX+60\]  
004047B2  |.  8D5428 30     |LEA EDX,DWORD PTR DS:\[EAX+EBP+30\]  
004047B6  |.  33C0          |XOR EAX,EAX  
004047B8  |.  F2:AE         |REPNE SCAS BYTE PTR ES:\[EDI\]  
  
上面代码的功能是将 0012F6F4 处的内容复制到 0038F2C8。  
这时要把 0012F6F4 处的硬件断点删除，改设 0038F2C8 。  
大家一定对此感疑惑吧？这是因为 0012F6F4 是属于栈地址，保存的是临时变量。  
很多人对数据进行置硬件断点时，程序被莫名其妙的中断再 7CXXXXXX 处，那可能是把硬件断点置在临时变量上了。  
一般程序都会把一些重要的临时变量复制到全局变量或固定的内存空间。所以在一般的调试过程中要尽量避免在栈地址  
中置硬件断点，除非情况特殊（比如 脱壳或分析算法之类的）。  
  
继续运行程序，程序主界面出现了，启动帐号，程序中断在以下代码：  
  
  
  
00401AF0  |.  8B5C24 1C     MOV EBX,DWORD PTR SS:\[ESP+1C\]  
00401AF4  |>  8A043B        /MOV AL,BYTE PTR DS:\[EBX+EDI\]            ;  EBX 指向密码的 MD5 码  
00401AF7  |.  8D6F 01       |LEA EBP,DWORD PTR DS:\[EDI+1\]            ;  EBP 为临时计数变量  
00401AFA  |.  884424 10     |MOV BYTE PTR SS:\[ESP+10\],AL  
00401AFE  |.  8B4C24 10     |MOV ECX,DWORD PTR SS:\[ESP+10\]  
00401B02  |.  81E1 FF000000 |AND ECX,0FF  
00401B08  |.  8BC1          |MOV EAX,ECX  
00401B0A  |.  C1E8 02       |SHR EAX,2  
00401B0D  |.  8A80 F8864000 |MOV AL,BYTE PTR DS:\[EAX+4086F8\]  
00401B13  |.  880432        |MOV BYTE PTR DS:\[EDX+ESI\],AL  
00401B16  |.  8B4424 20     |MOV EAX,DWORD PTR SS:\[ESP+20\]  
00401B1A  |.  3BE8          |CMP EBP,EAX  
00401B1C  |.  0F8D A0000000 |JGE QAL1\_4\_1.00401BC2                   ;  判断是否将 MD5 码的最后一个字节加密完  
00401B22  |.  8A441F 01     |MOV AL,BYTE PTR DS:\[EDI+EBX+1\]  
00401B26  |.  83E1 03       |AND ECX,3  
00401B29  |.  884424 11     |MOV BYTE PTR SS:\[ESP+11\],AL  
00401B2D  |.  8B4424 11     |MOV EAX,DWORD PTR SS:\[ESP+11\]  
00401B31  |.  25 FF000000   |AND EAX,0FF  
00401B36  |.  8BD8          |MOV EBX,EAX  
00401B38  |.  C1EB 04       |SHR EBX,4  
00401B3B  |.  C1E1 04       |SHL ECX,4  
00401B3E  |.  0BD9          |OR EBX,ECX  
00401B40  |.  45            |INC EBP  
00401B41  |.  8A8B F8864000 |MOV CL,BYTE PTR DS:\[EBX+4086F8\]  
00401B47  |.  884C32 01     |MOV BYTE PTR DS:\[EDX+ESI+1\],CL  
00401B4B  |.  8B4C24 20     |MOV ECX,DWORD PTR SS:\[ESP+20\]  
00401B4F  |.  3BE9          |CMP EBP,ECX  
00401B51  |.  7D 3C         |JGE SHORT QAL1\_4\_1.00401B8F             ;  不会产生的跳转  
00401B53  |.  8B5C24 1C     |MOV EBX,DWORD PTR SS:\[ESP+1C\]  
00401B57  |.  83E0 0F       |AND EAX,0F  
00401B5A  |.  C1E0 02       |SHL EAX,2  
00401B5D  |.  8A4C1F 02     |MOV CL,BYTE PTR DS:\[EDI+EBX+2\]  
00401B61  |.  884C24 12     |MOV BYTE PTR SS:\[ESP+12\],CL  
00401B65  |.  8B4C24 12     |MOV ECX,DWORD PTR SS:\[ESP+12\]  
00401B69  |.  81E1 FF000000 |AND ECX,0FF  
00401B6F  |.  8BE9          |MOV EBP,ECX  
00401B71  |.  C1ED 06       |SHR EBP,6  
00401B74  |.  0BE8          |OR EBP,EAX  
00401B76  |.  83E1 3F       |AND ECX,3F  
00401B79  |.  8A85 F8864000 |MOV AL,BYTE PTR SS:\[EBP+4086F8\]  
00401B7F  |.  884432 02     |MOV BYTE PTR DS:\[EDX+ESI+2\],AL  
00401B83  |.  8A89 F8864000 |MOV CL,BYTE PTR DS:\[ECX+4086F8\]  
00401B89  |.  884C32 03     |MOV BYTE PTR DS:\[EDX+ESI+3\],CL  
00401B8D  |.  EB 17         |JMP SHORT QAL1\_4\_1.00401BA6  
00401B8F  |>  8B5C24 1C     |MOV EBX,DWORD PTR SS:\[ESP+1C\]  
00401B93  |.  83E0 0F       |AND EAX,0F  
00401B96  |.  8A0485 F88640>|MOV AL,BYTE PTR DS:\[EAX\*4+4086F8\]  
00401B9D  |.  884432 02     |MOV BYTE PTR DS:\[EDX+ESI+2\],AL  
00401BA1  |.  C64432 03 3D  |MOV BYTE PTR DS:\[EDX+ESI+3\],3D  
00401BA6  |>  8B4424 20     |MOV EAX,DWORD PTR SS:\[ESP+20\]  
00401BAA  |.  83C7 03       |ADD EDI,3                               ;  被加密的 MD5 码计数变量  
00401BAD  |.  83C2 04       |ADD EDX,4                               ;  加密后的 MD5 码计数变量  
00401BB0  |.  3BF8          |CMP EDI,EAX                             ;  判断是否加密完成  
00401BB2  |.^ 0F8C 3CFFFFFF \\JL QAL1\_4\_1.00401AF4  
00401BB8  |.  8BC6          MOV EAX,ESI  
00401BBA  |.  5F            POP EDI  
00401BBB  |.  5E            POP ESI  
00401BBC  |.  5D            POP EBP  
00401BBD  |.  5B            POP EBX  
00401BBE  |.  83C4 08       ADD ESP,8  
00401BC1  |.  C3            RETN  
  
  
  
00401BC2  |>  8A4424 10     MOV AL,BYTE PTR SS:\[ESP+10\]  
00401BC6  |.  83E0 03       AND EAX,3  
00401BC9  |.  C1E0 04       SHL EAX,4  
00401BCC  |.  8A88 F8864000 MOV CL,BYTE PTR DS:\[EAX+4086F8\]  
00401BD2  |.  B0 3D         MOV AL,3D  
00401BD4  |.  884C32 01     MOV BYTE PTR DS:\[EDX+ESI+1\],CL  
00401BD8  |.  884432 02     MOV BYTE PTR DS:\[EDX+ESI+2\],AL  
00401BDC  |.  884432 03     MOV BYTE PTR DS:\[EDX+ESI+3\],AL           ;  最后两个字符固定为 '='  
00401BE0  |>  8BC6          MOV EAX,ESI  
  
  
  
上面的的代码就是将 MD5 码加密成密文，代码看起来有点乱  
为了便于理解，我用 C 代码写了一个登录 QQ 的演示程序：  
  
#include "windows.h"  
#include "stdio.h"  
#include "md5.h"  
  
char szTable\[100\] = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";  
  
void login(char \* szNum,char \* szPwd, char \* szPath,int iPwdlen);  
void QQHash(unsigned char\* szMDin,unsigned char\* szQQout);  
  
void main()  
{  
    
  int icnt;  
  char szQQnum\[20\];  
  char szQQpwd\[20\];  
  char szQQpath\[255\] = "" ;  
  char szPathfmt\[10\] = "%s %s";  
  char szTmp\[255\] = "";  
  unsigned long lpPath;  
  
  printf("请输入 QQ.exe 所在的路径\\n (例如 c:\\\\program files\\\\tencent\\\\qq\\\\) : \\n");  
    
  do  
  {  
    scanf("%s",szTmp);  
    wsprintf(szQQpath,szPathfmt,szQQpath,szTmp);  
  
    
  }while(getchar() != 0x0a);  
    
  lpPath = (unsigned  long )(szQQpath) + 1;  
  
  
    
  printf("请输入 QQ 号码 :\\n");  
  scanf("%s",szQQnum);  
      
  printf("请输入 QQ 密码 : \\n");  
  
  scanf("%s",szQQpwd);  
  icnt = strlen(szQQpwd);  
    
    login(szQQnum,szQQpwd,(char\*)(lpPath),icnt);  
  
  
}  
  
void login(char \* szNum,char \* szPwd, char \* szPath, int iPwdlen)  
{    
    
  char szCmd\[255\] = "";  
  char szMDHash\[30\] = "";  
  char szQQHash\[30\] = "";  
  MD5 MD;    
    
  MD.Hash ((unsigned char\*)szMDHash,(unsigned char\*)szPwd,(unsigned long) iPwdlen);  
  QQHash((unsigned char\*)szMDHash,(unsigned char\*)szQQHash);  
  
    
  wsprintf((LPSTR)szCmd,"%sQQ.exe /START QQUIN:%s PWDHASH:%s /STAT:41",   
  szPath,szNum,szQQHash);  
  WinExec((LPCSTR)szCmd,SW\_SHOWNORMAL);  
  
  
  
}  
  
  
  
  
  
void QQHash(unsigned char\* szMDin,unsigned char\* szQQout)  
{  
  
  int i = 0 , j=0;  
  unsigned char c1,c2,c3,t1;  
  c1=0;  
  c2=0;  
  c3=0;  
  t1=0;  
  
  
  while(true)  
  {  
    c1 = szMDin\[j\];  
    t1 = c1;  
    c1 = c1 >> 2;  
    szQQout\[i\] =szTable\[c1\];  
    j++;  
    i++;  
      
    if(j >= 16)  
    {  
      c1 = szMDin\[j\];  
      c1 = c1 & 0x03;  
      c1 = c1 << 4;  
      szQQout\[i\] = szTable\[c1\];  
      i++;  
      szQQout\[i\] = 0x3d;  
      i++;  
      szQQout\[i\] = 0x3d;  
      break;  
  
  
    }  
      
    c2 = szMDin\[j\];  
    c1 = t1 & 0x03;  
    c1 = c1 << 4;  
    t1 = c2;  
    c2 = c2 >> 4;  
    c2 = c2 | c1;  
    szQQout\[i\] =szTable\[c2\];  
      
    j++;  
    i++;  
  
    c1 = t1 & 0x0f;  
    c1 = c1 << 2;  
    c3 = szMDin\[j\];  
    t1 = c3 >> 6;  
    t1 = c1 | t1;  
    c3 = c3 & 0x3f;  
    szQQout\[i\] =szTable\[t1\];  
    i++;  
    szQQout\[i\] =szTable\[c3\];  
      
    j++;  
    i++;  
  
  
    
  
    
  }  
  
  
}   
  
由于代码写的比较仓促，注释也懒的加，所以上面代码也不好理解，建议读代码之前，自己跟踪一下关键代码。  
  
当然，上面的代码只是 QQ 自动登录器的一个简单演示，功能并不完善。  
有兴趣编写 QQ 自动登录器的，可以参考参考上面代码（详细代码 见附件）。  
  
QQ 自动登录器很方便，但最好不要在别人电脑上使用，因为它保存的密码只用 MD5 加密一次，容易被爆破。  
在我的 Athlon XP 2500+ 上大约花了两个小时就把一个 8 位的纯数字密码爆破完成。  
而 QQ 的本地保存密码就相对安全多了，多次 MD5 循环加密，很难爆破的。  
  
  
  
特别声明：  
    
关于代码中的 MD5.H 非本人编写的，而是摘自 《看雪论坛精华 7》中的 “QQ 本地密码文件暴力破解” 一文。  
感谢原作者 —— lotusroots。  
没办法，人懒啊，自己去写个 MD5 代码很费时的。  
转载时请保持文章的完整性。 

 4dd466d0e1e0cc123afbe504da2462ca
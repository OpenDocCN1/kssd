
```
00401BCC  |.  8A88 F8864000 MOV CL,BYTE PTR DS:[EAX+4086F8]  
00401BD2  |.  B0 3D         MOV AL,3D  
00401BD4  |.  884C32 01     MOV BYTE PTR DS:[EDX+ESI+1],CL  
00401BD8  |.  884432 02     MOV BYTE PTR DS:[EDX+ESI+2],AL  
00401BDC  |.  884432 03     MOV BYTE PTR DS:[EDX+ESI+3],AL         ;  最后两个字符固定为 '='  
00401BE0  |>  8BC6          MOV EAX,ESI  
```

上面的代码就是将 MD5 码加密成密文，代码看起来有点乱

为了便于理解，我用 C 代码写了一个登录 QQ 的演示程序：

```c
#include "windows.h"  

#include "stdio.h"  

#include "md5.h"  

char szTable[100] = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";  

void login(char * szNum,char * szPwd, char * szPath,int iPwdlen);  

void QQHash(unsigned char* szMDin,unsigned char* szQQout);  

void main()  

{  

  int icnt;  

  char szQQnum[20];  

  char szQQpwd[20];  

  char szQQpath[255] = "" ;  

  char szPathfmt[10] = "%s %s";  

  char szTmp[255] = "";  

  unsigned long lpPath;  

  printf("请输入 QQ.exe 所在的路径\\n (例如 c:\\\\program files\\\\tencent\\\\qq\\\\) : \\n");  

  do  

  {  

    scanf("%s",szTmp);  

    wsprintf(szQQpath,szPathfmt,szQQpath,szTmp);  

  }while(getchar() != 0x0a);  

  lpPath = (unsigned  long )(szQQpath) + 1;  

  printf("请输入 QQ 号码 :\\n");  

  scanf("%s",szQQnum);  

  printf("请输入 QQ 密码 : \\n");  

  scanf("%s",szQQpwd);  

  icnt = strlen(szQQpwd);  

    login(szQQnum,szQQpwd,(char*)(lpPath),icnt);  

}  

void login(char * szNum,char * szPwd, char * szPath, int iPwdlen)  

{   

  char szCmd[255] = "";  

  char szMDHash[30] = "";  

  char szQQHash[30] = "";  

  MD5 MD;   

  MD.Hash ((unsigned char*)szMDHash,(unsigned char*)szPwd,(unsigned long) iPwdlen);  

  QQHash((unsigned char*)szMDHash,(unsigned char*)szQQHash);  

  wsprintf((LPSTR)szCmd,"%sQQ.exe /START QQUIN:%s PWDHASH:%s /STAT:41",   

  szPath,szNum,szQQHash);  

  WinExec((LPCSTR)szCmd,SW_SHOWNORMAL);  

}  

void QQHash(unsigned char* szMDin,unsigned char* szQQout)  

{  

  int i = 0 , j=0;  

  unsigned char c1,c2,c3,t1;  

  c1=0;  

  c2=0;  

  c3=0;  

  t1=0;  

  while(true)  

  {  

    c1 = szMDin[j];  

    t1 = c1;  

    c1 = c1 >> 2;  

    szQQout[i] =szTable[c1];  

    j++;  

    i++;  

    if(j >= 16)  

    {  

      c1 = szMDin[j];  

      c1 = c1 & 0x03;  

      c1 = c1 << 4;  

      szQQout[i] = szTable[c1];  

      i++;  

      szQQout[i] = 0x3d;  

      i++;  

      szQQout[i] = 0x3d;  

      break;  

    }  

    c2 = szMDin[j];  

    c1 = t1 & 0x03;  

    c1 = c1 << 4;  

    t1 = c2;  

    c2 = c2 >> 4;  

    c2 = c2 | c1;  

    szQQout[i] =szTable[c2];  

    j++;  

    i++;  

    c1 = t1 & 0x0f;  

    c1 = c1 << 2;  

    c3 = szMDin[j];  

    t1 = c3 >> 6;  

    t1 = c1 | t1;  

    c3 = c3 & 0x3f;  

    szQQout[i] =szTable[t1];  

    i++;  

    szQQout[i] =szTable[c3];  

    j++;  

    i++;  

  }  

} 
```

由于代码写的比较仓促，注释也懒的加，所以上面代码也不好理解，建议读代码之前，自己跟踪一下关键代码。

当然，上面的代码只是 QQ 自动登录器的一个简单演示，功能并不完善。

有兴趣编写 QQ 自动登录器的，可以参考参考上面代码（详细代码 见附件）。

QQ 自动登录器很方便，但最好不要在别人电脑上使用，因为它保存的密码只用 MD5 加密一次，容易被爆破。

在我的 Athlon XP 2500+ 上大约花了两个小时就把一个 8 位的纯数字密码爆破完成。

而 QQ 的本地保存密码就相对安全多了，多次 MD5 循环加密，很难爆破的。

### 特别声明

关于代码中的 `MD5.H` 非本人编写的，而是摘自《看雪论坛精华 7》中的“QQ 本地密码文件暴力破解”一文。

感谢原作者 —— `lotusroots`。

没办法，人懒啊，自己去写个 MD5 代码很费时的。

转载时请保持文章的完整性。

`4dd466d0e1e0cc123afbe504da2462ca`

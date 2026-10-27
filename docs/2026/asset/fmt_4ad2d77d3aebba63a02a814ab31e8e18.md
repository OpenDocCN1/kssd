
```
    MD5STEP(F4, b, c, d, a, in[1] + 0x85845dd1, 21);
MD5STEP(F4, a, b, c, d, in[8] + 0x6fa87e4f, 6);
     MD5STEP(F4,
d, a, b, c, in[15] + 0xfe2ce6e0, 10);
     MD5STEP(F4, c, d, a,
b, in[6] + 0xa3014314, 15);
     MD5STEP(F4, b, c, d, a, in[13]
+ 0x4e0811a1, 21);
     MD5STEP(F4, a, b, c, d, in[4] + 0xf7537e82,
6);
     MD5STEP(F4, d, a, b, c, in[11] + 0xbd3af235, 10);
    MD5STEP(F4, c, d, a, b, in[2] + 0x2ad7d2bb, 15);
MD5STEP(F4, b, c, d, a, in[9] + 0xeb86d391, 21);
     buf[0]
+= a;
     buf[1] += b;
     buf[2] += c;
  buf[3] += d;
 }

 char eulav(long i);

 void zhconvert(verylong
a, char *dbuff, int CodeLen){
     static
char *b;
     register long i,j,cnt = 0;
     b
= (char *)malloc( (size_t)(a[0] << 3) );
     if (!b)
return;
     do    {
  b[cnt] = eulav(a[1] & 15);
         cnt++;
         zrshift(a, (long) 4, &a);
    }
     while ( (a[1] != 0) || (a[0] != 1)
|| (cnt < CodeLen) );
     b[cnt]&=0x00;
  j=cnt-1;
     for    (i=0; i<cnt;
i++)    {
         dbuff[i]=b[j];
         j--;
     }
    dbuff[i]=0x00;
     free(b);
 }

 void GenerateCode(HWND hDlg){
     static char modulus_n[]="24DFDA27FA14D3F27DDF62CEA5D2381F9";
     static char private_d[]="1E2D9B52ADCBC20DCCDE3C721AA740E83";
     static char hextab[]="0123456789ABCDEF";
     char        szName[31],
szOrg[31], szH[62], signature[16], md[33];
     struct MD5Context
md5c;
         char
  Regcode[42];
     int
  i,j,nLen,oLen;
     BYTE
  t, u;
     verylong    d = 0;
    verylong    m = 0;
     verylong
  n = 0;
     nLen=GetDlgItemTextA(hDlg, EDIT_NAME,
szName, 31);
         oLen=GetDlgItemTextA(hDlg, EDIT_ORG,
szOrg, 31);
     if    (nLen < 1) {
         SetDlgItemTextA(hDlg, EDIT_CODE, "Name
must contain at least 1 char.");
         return;
     }
     if    (oLen <
1) {
         SetDlgItemTextA(hDlg, EDIT_CODE,
"Organization must contain at least 1 char.");
  return;
     }
  //concatenate the strings
     for (i=0; i<nLen;
i++) szH[i] = szName[i];
     for (i=0; i<oLen; i++) szH[nLen+i]
= szOrg[i];
     szH[nLen+oLen] = '\0';
     MD5Init(&md5c);
     MD5Update(&md5c, szH, nLen+oLen);
     MD5Final(signature,
&md5c);
     j=0;
     for
  (i=0; i<16; i++){
         u=t=signature[i];
         md[j]=hextab[((t & 0xF0) >> 4)];
         md[j+1]=hextab[(u & 0x0F)];
        j+=2;
     }
  md[j]&=0x00;
      /* Read in Bignums
*/
     zstrtozbas(modulus_n, (long) 16, &n);
  zstrtozbas(private_d, (long) 16, &d);
     zstrtozbas(md,
(long) 16, &m);
     /* m = m^d % n */
  zexpmod(m,d,n,&m);
     /* Convert m to Hex
String, 32 Chars long */
     zhconvert(m, md, 32);
         /* Clean up */
     zfree(&d);
     zfree(&m);
     zfree(&n);
     /* Show Regcode */
    SetDlgItemTextA(hDlg, EDIT_CODE, md);
 }

BOOL CALLBACK DlgP (HWND hDlg, UINT message, WPARAM wParam, LPARAM lParam)
{
     switch (message)
     {
      case WM_INITDIALOG :
      SendDlgItemMessageA(hDlg, EDIT_NAME, EM_SETLIMITTEXT,
(WPARAM) 30, 0);
             SetWindowTextA(hDlg,
"KeyGen For Bishop Lockless CrackMe4 By 娃娃/[CCG]");
      SetDlgItemTextA(hDlg, EDIT_CODE, "请输入用户名和组织名.");
             return TRUE;
        case WM_COMMAND :
      switch (LOWORD (wParam))
      {
      case BT_GENERATE :
              GenerateCode(hDlg);
              SetFocus(GetDlgItem(hDlg,
EDIT_NAME));
      break;
      case IDCANCEL :
              EndDialog
(hDlg, 0);
      break;
  }
             break;
     }
     return FALSE;
 }

 int WINAPI
WinMain (HINSTANCE hInstance, HINSTANCE hPrevInstance, PSTR szCmdLine, int iCmdShow)
 {
     DialogBoxA(hInstance, MAKEINTRESOURCE(DLG_MAIN),
0, DlgP);
     return 0;
 }
```

```
********************************************************************KeyGen.RC********************************************************************

111 DIALOG 24, 50, 210, 119
STYLE DS_ABSALIGN | DS_MODALFRAME |
0xA04L | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU
CAPTION "KeyGen
By 娃娃/[CCG]"
FONT 9, "宋体"
LANGUAGE LANG_FRENCH, SUBLANG_NEUTRAL
{
CONTROL "", 1009, "EDIT", ES_LEFT | ES_AUTOHSCROLL | ES_OEMCONVERT | WS_CHILD
| WS_VISIBLE | WS_BORDER | WS_TABSTOP, 4, 13, 202, 12
CONTROL "(&G)生成",
1008, "BUTTON", BS_PUSHBUTTON | WS_CHILD | WS_VISIBLE | WS_TABSTOP, 7, 101, 45,
13
CONTROL "(&X)退出", 2, "BUTTON", BS_PUSHBUTTON | WS_CHILD | WS_VISIBLE
| WS_TABSTOP, 59, 101, 45, 13
CONTROL "", 1010, "EDIT", ES_LEFT | ES_AUTOHSCROLL
| ES_READONLY | WS_CHILD | WS_VISIBLE | WS_BORDER | WS_TABSTOP, 4, 73, 202, 12

CONTROL "用户名称:", -1, "STATIC", SS_LEFT | WS_CHILD | WS_VISIBLE | WS_GROUP,
5, 4, 36, 8
CONTROL "您的注册码:", -1, "STATIC", SS_LEFT | WS_CHILD | WS_VISIBLE
| WS_GROUP, 6, 64, 60, 8
CONTROL "组织名称:", -1, "STATIC", SS_LEFT | WS_CHILD
| WS_VISIBLE | WS_GROUP, 6, 32, 64, 8
CONTROL "", 1011, "EDIT", ES_LEFT |
ES_AUTOHSCROLL | ES_OEMCONVERT | WS_CHILD | WS_VISIBLE | WS_BORDER | WS_TABSTOP,
4, 41, 202, 12
}
*******************************************************************
```

好久都没有写这种破解文章了 看着以前破解界的朋友现在或忙于工作或投身于编程或者逆向工程 也不知道自己心里是因为技不如人而感到羞愧 还是因为失去一些可以互相交流和帮助的老朋友而感到惋惜 看雪论坛陪我度过了3年的时光 无数精彩的回忆都留在这里 不免有的时候想想再过三年之后的看雪论坛 想想三年之后的[CCG] 又或者想想三年之后的我 ……  ……  无限惆怅 不好意思 一时间有感而发 说多了

还是那句话：仅以此文献给我可爱的组织CCG 希望他能蒸蒸日上

作者：娃娃 QQ:15110296 EMAil:Nydoll@etang.com
属于中国破解组织CCG(CHiNA CrACKiNG GrOUp)
写于2002年11月8日

*/转贴请保持完整 并写明出处:"看雪软件调试论坛" 作者:"娃娃[CCG]"/*

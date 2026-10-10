

# 看雪论坛读书月第一题 Crackme01 分析和注册机

> 原文：[`www.pediy.com/kssd/pediy10/68344.html`](https://www.pediy.com/kssd/pediy10/68344.html)

简单分析：  
  
1、使用读写禁止读取的内存来触发异常，异常中进行反调试检测及相应处理。  

代码:

```
mov     \[ebp+SEH\_trigger\_ep\], 29Ch
mov     ecx, \[ebp+SEH\_trigger\]
mov     byte ptr \[ecx\], 6

```

地址\[29c\]内写入 6，来触发异常。  
  
2、反调试，使用查找父进程的方法，如找到父进程非 explorer.exe，则终止进程。同时代码中，  
有反 HideOD 的 Process32NextW 的代码，当发现 Process32NextW 返回异常，同样终止进程，因此无  
法使用 HideOD 来 bypass 该反调试。  
   处理方式为修改一个跳转使程序不要 TerminateProcess 就可。  
  
3、花指令  
  通过花指令来阻碍调试。花指令只使用了一种。  
代码:

```
\_\_asm
{
        jmp dword ptr\[label1\]
label2:  stc
        jnb dword ptr\[label3\]
        retn
label3：\_\_emit 0xff
label1:  call dword ptr\[label2\]
}

```

处理方式为，OD 中在 label1 处 f8，就可越过。IDA 也可以正确识别该部分代码。  
  
4、MessageBoxA  
在通往成功的路径上，使用 MessageBoxA 拦截  
代码:

```
  if ( MessageBoxA((HWND)(Regcode\[10\] == 'U'), "继续努力", "错了", 0); )
  {
    pID\_currentprocess = GetCurrentProcess();
    TerminateProcess(pID\_currentprocess, 0);
  }
  

```

如果 Regcode\[10\]=='U'则可跨过此障碍。  
  
5、算法  
按钮事件,进行用户名和注册码的合法性初步校验,如合法，进入 chkreg()  
代码:

```
int \_\_cdecl onCheckBtnClick()
{
  int ecx0; // ecx@0
  int result; // eax@2
  char \*regcode; // ST08\_4@1
  int regcodeItem; // eax@1
  int len\_regcode; // eax@1
  int regnameItem; // eax@1
  int len\_regname; // eax@1
  int regname1; // eax@1
  int this; // \[sp+5Ch\] \[bp-4h\]@1
  int regcode1; // \[sp+58h\] \[bp-8h\]@1
  char \*regname2; // \[sp+54h\] \[bp-Ch\]@1
  signed int tmpi; // \[sp+50h\] \[bp-10h\]@5
  int v12; // \[sp+0h\] \[bp-60h\]@12
  int Func\_addr; // \[sp+4Ch\] \[bp-14h\]@12

  this = ecx0;
  regcode = (char \*)(ecx0 + 0x60);
  regcodeItem = CWnd\_\_GetDlgItem(1001);
  CWnd\_\_GetWindowTextA(regcodeItem, regcode);
  len\_regcode = getstrlen1();
  regcode1 = CString\_\_GetBuffer(this + 96, len\_regcode);
  regnameItem = CWnd\_\_GetDlgItem(1002);
  CWnd\_\_GetWindowTextA(regnameItem, this + 0x64);
  len\_regname = getstrlen1();
  regname1 = CString\_\_GetBuffer(this + 100, len\_regname);
  regname2 = (char \*)regname1;
  if ( strlen((const char \*)regname1) )
  {
    if ( strlen((const char \*)regcode1) )
    {
      tmpi = 0;
      while ( regname2\[tmpi\] && regname2\[tmpi\] > 'a' && regname2\[tmpi\] < 'z' )
        ++tmpi;
      if ( tmpi == 6 )
      {
        strcpy(&Name, regname2);
        strcpy(Regcode, (const char \*)regcode1);
        Func\_addr = (int)&v12;
        result = chkreg((int)&v12, (int)regname2, regcode1);
      }
      else
      {
        result = CWnd\_\_MessageBoxA(this, "非法用户", 0, 0);
      }
    }
    else
    {
      result = CWnd\_\_MessageBoxA(this, "请输入注册码!", 0, 0);
    }
  }
  else
  {
    result = CWnd\_\_MessageBoxA(this, "请输入用户名", 0, 0);
  }
  return result;
}

```

第一级算法  
要求:regcode\[2n+1\]+0x1B=regname\[n\];  
     regcode\[2n+2\]+0x20<regname\[n\];  
如不符合，修改函数的返回地址为错误提示函数 wrong()的地址。  
如正确，设置返回地址为 chkregcode()。  
在函数最后，使用 mov \[29c\],6 来触发异常，进入父进程检验的反调试函数。  
代码:

```
int \_\_cdecl chkreg(int Func\_addr, int regname, int regcode)
{
  int result; // eax@8
  int v4; // \[sp+0h\] \[bp-6Ch\]@1
  int \*v5; // \[sp+5Ch\] \[bp-10h\]@1
  int chkregcode\_addr; // \[sp+58h\] \[bp-14h\]@1
  signed int tmpi; // \[sp+54h\] \[bp-18h\]@1
  int tmpj; // \[sp+50h\] \[bp-1Ch\]@1
  int v9; // \[sp+68h\] \[bp-4h\]@8

  v5 = &v4;
  chkregcode\_addr = Func\_addr - 16;
  \*(\_DWORD \*)(Func\_addr - 16) = chkregcode;
  tmpi = 0;
  tmpj = 0;
  loc\_401BF4(Func\_addr - 16);
  while ( tmpi < 6 )
  {
    if ( \*(\_BYTE \*)(tmpi + regname) != \*(\_BYTE \*)(tmpj + regcode) + 0x1B )
      wrong();
    loc\_401C30();
    if ( \*(\_BYTE \*)(tmpi + regname) > \*(\_BYTE \*)(tmpj + regcode + 1) + 0x20 )
      \*(\_DWORD \*)chkregcode\_addr = wrong;
    tmpj += 2;
    ++tmpi;
  }
  v9 = 0;
  result = loc\_401C7A();
  v29c = 6;
  return result;
}

```

第二级算法  
跨过父进程检测的反调试后，进入第二级算法 chkregcode()  
代码:

```
int \_\_cdecl chkregcode()
{
  int tmpB; // ecx@4
  signed int tmpChar; // eax@15
  int v3; // edx@6
  int v4; // eax@10
  HANDLE pID\_currentprocess; // eax@11
  signed int v6; // edx@15
  int v7; // ecx@23
  char \*charlist; // \[sp+88h\] \[bp-4h\]@1
  char \*charlist1; // \[sp+84h\] \[bp-8h\]@1
  signed int tmpi; // \[sp+64h\] \[bp-28h\]@1
  \_BYTE chartable\[28\]; // \[sp+68h\] \[bp-24h\]@6
  HANDLE hProcess; // \[sp+60h\] \[bp-2Ch\]@11
  signed int v13; // \[sp+5Ch\] \[bp-30h\]@12
  signed int tmpj; // \[sp+58h\] \[bp-34h\]@12
  int v15; // \[sp+54h\] \[bp-38h\]@14
  char \*v16; // \[sp+50h\] \[bp-3Ch\]@20
  char \*v17; // \[sp+4Ch\] \[bp-40h\]@20

  charlist  = "ABCDEFGHIJKLMNOPQRSTUVWXY";
  charlist1 = "ABCDEFGHIJKLMNOPQRSTUVWXY";
  tmpi = 0;
  while ( tmpi != 24 )
  {
    loc\_40182B();
    if ( Name\[0\] == \*charlist + 32 || Name\[5\]== \*charlist + 32) )
      tmpB = (int)(charlist++ + 1);
    LOBYTE(tmpB) = \*charlist;
    chartable\[tmpi\] = \*charlist;
    v3 = tmpi++ + 1;
    charlist += 2;
    loc\_401893(tmpB, v3);
    if ( !\*charlist )
    {
      if ( tmpi < 24 )
        charlist = charlist1 + 1;
    }
  }
  if ( MessageBoxA((HWND)(Regcode\[10\] == 'U'), "继续努力", "错了", 0))
  {
    pID\_currentprocess = GetCurrentProcess();
    TerminateProcess(pID\_currentprocess, 0);
  }
  tmpi = 0;
  v13 = 5;
  tmpj = 0;
  while ( tmpi != 12 )
  {
    v15 = 4 \* v13 - 4;
    do
    {
      tmpChar = Regcode\[tmpi\];
      v6 = chartable\[v15++\];
      if ( tmpChar == v6 )
        break;
      ++tmpj;
    }
    while ( tmpj <= 4 );
    if ( tmpj == 5 )
    {
      v16 = "ABCDEFGHIJKLMN";
      v17 = "OPQRSTUVWXYZ";
      tmpj = tmpChar;
      if ( tmpChar > 0 )
        wrong();
      wrong();
    }
    tmpi += 2;
    v7 = v13-- - 1;
    loc\_4019B2(v7);
    if ( !v13 )
      v13 = 6;
    tmpj = 0;
  }
  return succeed();
}

```

在第一个循环中为打乱字母表。  
取其中的 24 个字母，每 4 个为一组，共六组，分别对应名字的 6 个字符。  
  
注册机  
根据以上算法，计算得出名字有如下限制：  
1、名字必须为 6 位字母，名字中字母不能为'a'  
2、Name\[0\]必须为 defghij,  
3、Name\[5\]必须为 p  
4、所有字符必须为打乱的字母表对应组（4 个 1 组）中的一个字母.见算法。  
由于不是所有名字均有解。因此采用注册机随机生成注册码的方法，每点一次随机生成一个。  
具体算法为  
  
代码:

```
void CkeygenDlg::OnBnClickedOk()
{
  // TODO: 在此添加控件通知处理程序代码
  char Name\[7\];
  char RegCode\[13\];
  char Ctable\[26\]="ABCDEFGHIJKLMNOPQRSTUVWXY";
  char KeyCtable\[26\]="";
  char NameTable\[6\]\[4\];
  char tmpB;

  int tmpi,tmpj,tmpk;
  int tmpRand;

  Name\[5\]='p';
  Name\[6\]=0x00;

  //GetDlgItemText(IDC\_NAME,&Name\[0\],2);
  srand( (unsigned)time( NULL ) );
  tmpRand=rand()%7;
  Name\[0\]='d'+tmpRand;
  if(Name\[0\]<'d' || Name\[0\]>'j')
  {
    AfxMessageBox("请设置名字的首字母为 defghij 其中的一个，否则无解");
    return;
  }
  //Create Keytable
  tmpj=0;
  for(tmpi=0;tmpi<=24;tmpi++)
  {
    if(Name\[0\]==(Ctable\[tmpj\]+0x20)||Name\[5\]==(Ctable\[tmpj\]+0x20))
    {
      tmpj=(tmpj+1)%25;
    }
    tmpB=Ctable\[tmpj\];
    KeyCtable\[tmpi\]=tmpB;
    tmpj=(tmpj+2)%25;
  }
  //Create Nametable
  for(tmpj=0;tmpj<4;tmpj++)
    NameTable\[0\]\[tmpj\]=KeyCtable\[16+tmpj\]+0x1B;
  for(tmpj=0;tmpj<4;tmpj++)
    NameTable\[1\]\[tmpj\]=KeyCtable\[12+tmpj\]+0x1B;
  for(tmpj=0;tmpj<4;tmpj++)
    NameTable\[2\]\[tmpj\]=KeyCtable\[8+tmpj\]+0x1B;
  for(tmpj=0;tmpj<4;tmpj++)
    NameTable\[3\]\[tmpj\]=KeyCtable\[4+tmpj\]+0x1B;
  for(tmpj=0;tmpj<4;tmpj++)
    NameTable\[4\]\[tmpj\]=KeyCtable\[0+tmpj\]+0x1B;
  for(tmpj=0;tmpj<4;tmpj++)
    NameTable\[5\]\[tmpj\]=KeyCtable\[20+tmpj\]+0x1B;
  //Generate Name
  for(tmpi=1;tmpi<5;tmpi++)
  {
    srand( (unsigned)time( NULL ) );
    tmpRand=rand()%4;
    for(tmpj=tmpRand;tmpj<(tmpRand+4);tmpj++)
    {
      if(NameTable\[tmpi\]\[tmpj%4\]>'a'&&NameTable\[tmpi\]\[tmpj%4\]<'z')
        Name\[tmpi\]=NameTable\[tmpi\]\[tmpj%4\];
    }
  }
  Name\[5\]='p';
  Name\[6\]=0x00;
  SetDlgItemText(IDC\_NAME,&Name\[0\]);
  //Generate Regcode
  for(tmpi=0;tmpi<6;tmpi++)
  {
    RegCode\[2\*tmpi\]=Name\[tmpi\]-0x1B;
    RegCode\[2\*tmpi+1\]=Name\[tmpi\]-0x20;
  }
  RegCode\[12\]=0x00;
  SetDlgItemText(IDC\_REGCODE,&RegCode\[0\]);
  SetDlgItemText(IDOK,"NextOne");
}

```

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || keygen.rar  
 |

第一次写 keygen。  
里面的随机种子用的 time 取的秒，所以每秒只能生成一组注册码，按钮只能每隔一秒点一次，  
当时如果取 ms 做种子就好了。 

随机种子用 GetTickCount()就好，分析写的不错，赞
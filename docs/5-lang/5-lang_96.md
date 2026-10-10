

# WinXP 总管

> 原文：[`www.pediy.com/kssd/pediy06/pediy6889.htm`](https://www.pediy.com/kssd/pediy06/pediy6889.htm)

菜鸟也玩.net(WinXP 总管)  
  
算法部分未完成  
  
目标：WinXP 总管  
OS：win2000+sp4 ，.net 平台（那个单词太长不会写）  
工具：OllyDbg   Reflector V 4.12 5  
目的：今天发现 WinXP 总管支持 2000 了所以...........  
  
下面开始：  
运行程序输入用户名：winroot  
输入注册码：78787878  
程序会提示你，重新启动他自己。  
打开注册表：  

> ```
> 代码:
> Windows Registry Editor Version 5.00
> 
> \[HKEY\_CURRENT\_USER\\Software\\Yanicsoft\]
> 
> \[HKEY\_CURRENT\_USER\\Software\\Yanicsoft\\WinXP Manager\]
> "CreateSR"="True"
> "Date"="78787878"
> "Name"="winroot" 
> ```
> 
>   

  
看到了吧！  
准备启动的时候校验。现在不管他！  
打开 OD  
OD 忽略所有异常，自定义异常从头忽略到尾。  
OD 加载停留在此，多漂亮！只有一行代码 

> ```
> 代码:
> 112B46FE >- FF25 00200011   JMP DWORD PTR DS:\[<&mscoree.\_CorExeMain>>; mscoree.\_CorExeMain
> 112B4704    0000            ADD BYTE PTR DS:\[EAX\],AL
> 112B4706    0000            ADD BYTE PTR DS:\[EAX\],AL
>  
> ```
> 
>   

  
F9 运行-什么断点都不要下（我是 2000 他会检测系统）  
弹出一个对话框说 2000 下功能有限制  
先不管他  
到 OD  
在 RegQueryValueExW 下断点  
点那个提示框的确定，应该会中断在 RegQueryValueExW（winxp 下可能麻烦点在开始就下这个断点一直 F9 出现下面的堆栈，要不就下条件断点）  
注意堆栈 遇到异常一定要 shift+F9 

> ```
> 代码:
> 0012F584   00ADA795  /CALL to RegQueryValueExW from 00ADA792
> 0012F588   0000031C  |hKey = 31C
> 0012F58C   00C38884  |ValueName = "Date"
> 0012F590   00000000  |Reserved = NULL
> 0012F594   0012F5FC  |pValueType = 0012F5FC
> 0012F598   00000000  |Buffer = NULL
> 0012F59C   0012F5F8  \\pBufSize = 0012F5F8
> Date 出现两次才读注册码继续运行
> 
> 0012F060   791BD0F8  /CALL to RegQueryValueExW from mscorwks.791BD0F2
> 0012F064   0000017C  |hKey = 17C
> 0012F068   7933AD00  |ValueName = "Name"
> 0012F06C   00000000  |Reserved = NULL
> 0012F070   0012F0CC  |pValueType = 0012F0CC
> 0012F074   00000000  |Buffer = NULL
> 0012F078   0012F0E0  \\pBufSize = 0012F0E0
> Name 出现两次 
> ```
> 
>   

  
alt+m 从头开始搜索内存 UNICODE 填入“78787878”  
搜索到后选择找到的 37 00 38 00 37 00 38 00 37 00 38 00 37 00 38 00  7.8.7.8.7.8.7.8.  
下内存访问断点！  
清除以前的 API 断点  
只保留内存断点  
F9 中断在内存断点  
看一下寄存器~~~呵呵表说你没有看到注册码！ecx 指向的就是！  
  
最简单的方法：  
OD 中断在这 

> ```
> 代码:
> 112B46FE >- FF25 00200011   JMP DWORD PTR DS:\[<&mscoree.\_CorExeMain>>; mscoree.\_CorExeMain
>  
> ```
> 
>   

  
F7 跟进  
来到这里 

> ```
> 代码:
> 7917D4A5 >  55              PUSH EBP
> 7917D4A6    8BEC            MOV EBP,ESP
> 7917D4A8    51              PUSH ECX
> 7917D4A9    56              PUSH ESI
> 7917D4AA    6A 00           PUSH 0
> 7917D4AC    8D45 FC         LEA EAX,DWORD PTR SS:\[EBP\-4\]
> 7917D4AF    50              PUSH EAX
> 7917D4B0    6A 01           PUSH 1
> 7917D4B2    E8 A9A6FFFF     CALL mscoree.79177B60                    ;F8
> 7917D4B7    8BF0            MOV ESI,EAX
> 7917D4B9    85F6            TEST ESI,ESI
> 7917D4BB    7C 17           JL SHORT mscoree.7917D4D4
> 7917D4BD    68 D8D41779     PUSH mscoree.7917D4D8                    ; ASCII "\_CorExeMain"
> 7917D4C2    FF75 FC         PUSH DWORD PTR SS:\[EBP\-4\]
> 7917D4C5    FF15 E4101779   CALL DWORD PTR DS:\[<&KERNEL32.GetProcAdd>; F8
> 7917D4CB    85C0            TEST EAX,EAX
> 7917D4CD    74 05           JE SHORT mscoree.7917D4D4
> 7917D4CF    FFD0            CALL EAX                                 ; F7 跟进
>  
> ```
> 
>   

  
来到这里 

> ```
> 代码:
> 791C7067 >  56              PUSH ESI
> 791C7068    57              PUSH EDI
> 791C7069    33FF            XOR EDI,EDI
> 791C706B    57              PUSH EDI
> 791C706C    E8 2155FFFF     CALL mscorwks.791BC592
> 791C7071    50              PUSH EAX
> 791C7072    E8 DDA5FEFF     CALL mscorwks.791B1654
> 791C7077    3BC7            CMP EAX,EDI
> 791C7079    0F8C F7FB0700   JL mscorwks.79246C76
> 791C707F    E8 B7D70000     CALL mscorwks.791D483B
> 791C7084    8BF0            MOV ESI,EAX
> 791C7086    56              PUSH ESI
> 791C7087    E8 07A6FEFF     CALL mscorwks.791B1693
> 791C708C    56              PUSH ESI
> 791C708D    FF15 04131B79   CALL DWORD PTR DS:\[<&KERNEL32.LocalFree>>; KERNEL32.LocalFree
> 791C7093    6A 02           PUSH 2
> 791C7095    E8 820D0000     CALL mscorwks.CoInitializeEE
> 
>  
> ```
> 
>   

  
  
搜索代码  
  
MOV EBX,DWORD PTR DS:\[EAX+ECX\]  
SUB EBX,DWORD PTR DS:\[ECX\]  
  
找到这里  
7926B423    8B1C08          MOV EBX,DWORD PTR DS:\[EAX+ECX\]      ；下断点  
7926B426    2B19            SUB EBX,DWORD PTR DS:\[ECX\]  
注意 ECX 中断几次~遇到异常一定要 shift+F9  
看着那个像注册码的东西就是注册码  
  
  
作者主要是明码比对  
算法过程~  
1，取用户名，和注册码  
2，注册码，和 F（用户名）比对  

> ```
> 代码:
> StringType.StrCmp(注册码, this.FromHDSNGetRegisterCode(用户名)；
> 
> public string FromHDSNGetRegisterCode(string strCodeWord)
> {
>       Encrypt.MyClsEncrypt encrypt1 = new Encrypt.MyClsEncrypt(strCodeWord);
>       return encrypt1.Encrypt(Encrypt.MyClsRegister.GetPySerialNum());
> }
> 
> public MyClsEncrypt(string strCodeWord)
> {
>       this.strCryptMatrix = new string\[0x62\];
>       this.CodeWord = this.EncryptCode("Chinese" + strCodeWord + publicVarFun.ainfo.Version);
>       this.Matrix = "8x3p5BeabcdfghijklmnoqrstuvwyzACDEFGHIJKLMNOPQRSTUVWXYZ1246790";
>       int num1 = 1;
>       this.LAM = Strings.Len(this.Matrix);
>       this.strCryptMatrix\[1\] = this.Matrix;
>       int num3 = this.LAM;
>       for (int num2 = 2; num2 <= num3; num2++)
>       {
>             this.mov1 = Strings.Left(this.strCryptMatrix\[num1\], 1);
>             this.mov2 = Strings.Right(this.strCryptMatrix\[num1\], this.LAM - 1);
>             this.strCryptMatrix\[num2\] = this.mov2 + this.mov1;
>             num1++;
>       }
> }
> 
> /////////////////////////////////////////
> public string Encrypt(string strEncrypted)
> {
>       string text2;
>       try
>       {
>             string text3 = strEncrypted;
>             this.LS2E = strEncrypted.Length;
>             this.LCW = this.CodeWord.Length;
>             this.EncryptedLetter = "";
>             this.EncryptedString = "";
>             int num2 = 1;
>             int num5 = this.LS2E - 1;
>             for (int num1 = 1; num1 <= num5; num1++)
>             {
>                   string text1 = Strings.Mid(text3, num1, 1);
>                   this.MP = Strings.InStr(1, this.Matrix, text1, CompareMethod.Binary);
>                   this.CWL = Strings.Mid(this.CodeWord, num2, 1);
>                   int num4 = this.LAM;
>                   for (int num3 = 1; num3 <= num4; num3++)
>                   {
>                         if (StringType.StrCmp(Strings.Mid(this.strCryptMatrix\[num3\], this.MP, 1), this.CWL, false) == 0)
>                         {
>                               this.EncryptedLetter = Strings.Left(this.strCryptMatrix\[num3\], 1);
>                               this.EncryptedString = this.EncryptedString + this.EncryptedLetter;
>                               break;
>                         }
>                   }
>                   num2++;
>                   if (num2 > this.LCW)
>                   {
>                         num2 = 1;
>                   }
>             }
>             text2 = Strings.Trim(this.EncryptedString).ToUpper();
>       }
>       catch (Exception exception1)
>       {
>             ProjectData.SetProjectError(exception1);
>             ProjectData.ClearProjectError();
>       }
>       return text2;
> }
> 
> //////////////////////////////////////////////////////
> public string Version
> {
>       get
>       {
>             string\[\] textArray1 = new string\[6\] { "V", StringType.FromInteger(this.myType.Assembly.GetName().Version.Major), ".", StringType.FromInteger(this.myType.Assembly.GetName().Version.Minor), ".", StringType.FromInteger(this.myType.Assembly.GetName().Version.Build) } ;
>             return string.Concat(textArray1);
>       }
> }
> /////////////////////////////////////////////
> private string EncryptCode(string strEncrypted)
> {
>       string text1;
>       MD5CryptoServiceProvider provider1 = new MD5CryptoServiceProvider();
>       UTF8Encoding encoding1 = new UTF8Encoding();
>       byte\[\] buffer1 = provider1.ComputeHash(encoding1.GetBytes(strEncrypted));
>       byte\[\] buffer2 = buffer1;
>       for (int num2 = 0; num2 < buffer2.Length; num2++)
>       {
>             byte num1 = buffer2\[num2\];
>             text1 = text1 + StringType.FromByte(num1);
>       }
>       return StringType.FromInteger(Math.Abs(text1.GetHashCode()));
> }
>  
> ```
> 
>   

  
  
以上代码都是用 Reflector V 4.12 5 反编译出来的，偶主要是菜鸟，看着算法难受~看着是明码，就问 Fly 大虾，大虾曰：“OD 是个好同志！”  
只好是用 OD 跟一下~~~不得不佩服 Fly~~~~  
  
                                                                    WiNrOOt  
                                                            web:[www.winroot.org](http://www.winroot.org)  
                                                            Email:winroot@126.com  
                                                            2004-12-28 21:38  4dd466d0e1e0cc123afbe504da2462ca
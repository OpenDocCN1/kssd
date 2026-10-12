
# 一款 PHP 编辑器 DSV PHP Editor 1.2.1 的算法分析

> 原文：[`www.pediy.com/kssd/pediy10/59577.html`](https://www.pediy.com/kssd/pediy10/59577.html)

【文章标题】: 一款 PHP 编辑器 DSV PHP Editor 1.2.1 的算法分析

【文章作者】: iawen

【作者邮箱】: 913221@163.com

【作者 QQ 号】: 160193626

【软件名称】: DSV PHP Editor 1.2.1

【软件大小】: 9291KB

【下载地址】: 自己搜索下载

【加壳方式】: 无

【保护方式】: 注册码

【编写语言】: Borland Delphi 6.0 - 7.0

【使用工具】: OD

【操作平台】: XP SP2

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

DSV PHP Editor 是一款功能全面且强劲的 PHP 编辑器，内置 PHP 语法模板，支持语法快速输入和语法检查，语法加亮显示功能，可实时运行 PHP 代码，内置了一个全面的 PHP 代码资料库，并具备强大的搜索和替换等编辑功能，同时还支持 Java Script，Pascal Script，XML，SQL，Html，CSS 等多种语法，可以帮助你快速的完成 PHP\*\*\*页的开发和制作！

--------------------------------------------------------------------------------

## 详细过程

首先运用 F12 堆栈调用法找到关键段的段首：

```
006649C4   /$  55               push ebp
006649C5   |.  8BEC             mov ebp,esp
006649C7   |.  33C9             xor ecx,ecx
```

F8 单步跟踪：

```
00664A19   |.  8BD6             mov edx,esi
00664A1B   |.  8B86 34010000     mov eax,dword ptr ds:[esi+134]
00664A21   |.  FF96 30010000     call dword ptr ds:[esi+130]
          //CALL 运行完毕，输入的假码与用户名已经出现在了堆栈，关键的地方也不去远了，留意堆栈与寄存器
00664A35   |.  8D4D F8           lea ecx,dword ptr ss:[ebp-8]
00664A38   |.  8B96 9C000000     mov edx,dword ptr ds:[esi+9C]
00664A3E   |.  8BC6             mov eax,esi
00664A40   |.  E8 D3010000       call dsvPHPed.00664C18              ;  关键 CALL，也是算法 CALL，F7 跟进
00664A45   |.  837D F8 00        cmp dword ptr ss:[ebp-8],0          ;  返回标志值到堆栈
00664A49   |.  74 2C            je short dsvPHPed.00664A77          ;  这里跳向出错，不跳再注册成功
```

//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*跟进关键 CALL\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
00664C18   /$  55               push ebp
00664C19   |.  8BEC             mov ebp,esp
00664C1B   |.  51               push ecx
……
00664C54   |.  837D F8 00        cmp dword ptr ss:[ebp-8],0         ;  测试用户名是否为空
00664C58   |.  75 1A            jnz short dsvPHPed.00664C74
00664C5A   |.  6A 00            push 0
00664C5C   |.  66:8B0D 2C4F6600 mov cx,word ptr ds:[664F2C]
00664C63   |.  B2 02            mov dl,2
00664C65   |.  B8 384F6600      mov eax,dsvPHPed.00664F38          ;  ASCII "User name is empty. Please set this propery before!"
……
00664C74   |> \8D55 D8          lea edx,dword ptr ss:[ebp-28]
00664C77   |.  8B45 F8          mov eax,dword ptr ss:[ebp-8]
00664C7A   |.  E8 6159DAFF      call dsvPHPed.0040A5E0              ;  将用户名转为大写
……
00664CA5   |.  8BF0             mov esi,eax
00664CA7   |.  85F6             test esi,esi                        ;  小循环，ESI 为计数器
00664CA9   |.  7E 16            jle short dsvPHPed.00664CC1         ;  次数为用户名的长度
00664CAB   |.  BB 01000000      mov ebx,1
00664CB0   |>  8B45 F8          /mov eax,dword ptr ss:[ebp-8]
00664CB3   |.  0FB64418 FF      |movzx eax,byte ptr ds:[eax+ebx-1]   ;  按位取转换后的用户名的 ASCII 值
00664CB8   |.  F7EB             |imul ebx                            ;  乘以 EBX
00664CBA   |.  0145 DC          |add dword ptr ss:[ebp-24],eax       ;  累加
00664CBD   |.  43               |inc ebx                              ;  EBX 加一
00664CBE   |.  4E               |dec esi
00664CBF   |.^ 75 EF            \jnz short dsvPHPed.00664CB0         ;  结果累加保存到[ebp-24],此为 846
00664CC1   |>  8B45 FC          mov eax,dword ptr ss:[ebp-4]
```

//其作用就是把转换为大写的用户名的每位的 ASCII 值乘以一个变量 EDX，然后相累加
//用 C 来表达如：
//这里令用户名为：strName;注册码为：strCode

```c
int sum=0;
int k=1;
int len=strlen(strName);
for(int i=0;i<len;i++){
  if(strName[i]>='a' && strName[i]<='z')
    strName[i]-=32;
  sum+=(strName[i]*k);
  k++;
}
```

sum 里面保存的就是上面循环后所得的值了，相当于上面的 dword ptr ss:[ebp-4]

```
00664CC4   |.  F680 99000000 08 test byte ptr ds:[eax+99],8
           //byte ptr ds:[eax+99]的值总是为 7，故下面的跳转实现
00664CCB   |.  0F84 DC000000  je dsvPHPed.00664DAD
>>>>>>>>>>>>>跳转来到：
00664DAD   |> \BB 01000000     mov ebx,1                           ;  设变量为 k，另设一累加器 i=0
00664DB2   |>  8B45 DC          /mov eax,dword ptr ss:[ebp-24]      ;  计算结果也就是上面有 sum 送 EAX，设为 tmp
00664DB5   |.  03C3             |add eax,ebx                       ;  tmp+=k
00664DB7   |.  B9 C8000000      |mov ecx,0C8
00664DBC   |.  99               |cdq
00664DBD   |.  F7F9             |idiv ecx                           ;  
00664DBF   |.  8BC2             |mov eax,edx                        ;  tmp=% 0xC8
00664DC1   |.  8BD3             |mov edx,ebx                        ;  设变量 j,j=k
00664DC3   |.  03D2             |add edx,edx                        ;  j+=j,
00664DC5   |.  8B4D FC          |mov ecx,dword ptr ss:[ebp-4]
00664DC8   |.  8B89 90000000    |mov ecx,dword ptr ds:[ecx+90]
      //这里出现了一个常量字符串：{0E8E3B8C-6E33-4266-A116-8E72E5295C9F}
      //后来查询注册表，才知是程序写入注册表的主键名，这里设为：strS
00664DCE   |.  8A5411 FF        |mov dl,byte ptr ds:[ecx+edx-1]     ;  strS[j-1]
00664DD2   |.  8B4D FC          |mov ecx,dword ptr ss:[ebp-4]
00664DD5   |.  8B89 90000000    |mov ecx,dword ptr ds:[ecx+90]
00664DDB   |.  8A4C19 FF        |mov cl,byte ptr ds:[ecx+ebx-1]     ;  strS[k-1]
00664DDF   |.  32D1             |xor dl,cl                          ;  tmp2=strS[j-1] ^ strS[k-1]
00664DE1   |.  8B4D F8          |mov ecx,dword ptr ss:[ebp-8]
00664DE4   |.  8B75 F0          |mov esi,dword ptr ss:[ebp-10]
00664DE7   |.  8A4C31 FF        |mov cl,byte ptr ds:[ecx+esi-1]
00664DEB   |.  32D1             |xor dl,cl                          ;  tmp2= tmp2 ^ strName[i]
00664DED   |.  81E2 FF000000    |and edx,0FF
00664DF3   |.  33C2             |xor eax,edx                        ;  tmp^=tmp2
00664DF5   |.  8D4D D0          |lea ecx,dword ptr ss:[ebp-30]
00664DF8   |.  BA 02000000      |mov edx,2
00664DFD   |.  E8 7E63DAFF      |call dsvPHPed.0040B180             ;  转换为字符，设一临时变量 strTmp
00664E02   |.  8B55 D0          |mov edx,dword ptr ss:[ebp-30]      ;  sprintf(strTmp,"%02X",tmp);
00664E05   |.  8D45 E8          |lea eax,dword ptr ss:[ebp-18]
00664E08   |.  E8 9308DAFF      |call dsvPHPed.004056A0             ;  连接到 strCode 里
00664E0D   |.  8B45 F8          |mov eax,dword ptr ss:[ebp-8]       ;  strncat(strCode,strTmp,2)
00664E10   |.  8B55 F0          |mov edx,dword ptr ss:[ebp-10]
00664E13   |.  0FB64410 FF      |movzx eax,byte ptr ds:[eax+edx-1]  ;  取用户名：strName[i]
00664E18   |.  33C3             |xor eax,ebx                        ;  tmp2=strName[i] ^ k
00664E1A   |.  0145 DC          |add dword ptr ss:[ebp-24],eax      ;  累加到 sum:sum+=tmp2
00664E1D   |.  8B45 FC          |mov eax,dword ptr ss:[ebp-4]
00664E20   |.  F680 99000000 08 |test byte ptr ds:[eax+99],8
     //如上：byte ptr ds:[eax+99]的值总是为 7，故下面的跳转实现
00664E27   |.  74 0D            |je short dsvPHPed.00664E36
00664E29   |.  8B45 E4          |mov eax,dword ptr ss:[ebp-1C]
00664E2C   |.  0FB64418 FF      |movzx eax,byte ptr ds:[eax+ebx-1]
00664E31   |.  33C3             |xor eax,ebx
00664E33   |.  0145 DC          |add dword ptr ss:[ebp-24],eax
00664E36   |>  FF45 F0          |inc dword ptr ss:[ebp-10]         ;  这里的[ebp-10]即为变量 i
00664E39   |.  FF45 EC          |inc dword ptr ss:[ebp-14]
00664E3C   |.  8B45 F8          |mov eax,dword ptr ss:[ebp-8]
00664E3F   |.  E8 5408DAFF      |call dsvPHPed.00405698             ;  取用户的长度，设为 len,strlen(strName)
```



### 汇编代码分析

```asm
  00664E44   |.  3B45 F0          |cmp eax,dword ptr ss:[ebp-10]      ;  if(i>=len)  
  00664E47   |.  7D 07            |jge short dsvPHPed.00664E50  
  00664E49   |.  C745 F0 01000000 |mov dword ptr ss:[ebp-10],1       ;  则重新计数，这里是赋 1，但是 C++的数组从 0 开始  
  00664E50   |>  837D EC 0B       |cmp dword ptr ss:[ebp-14],0B       ;  所以这里置：i=0  
  00664E54   |.  7E 07            |jle short dsvPHPed.00664E5D  
  00664E56   |.  C745 EC 01000000 |mov dword ptr ss:[ebp-14],1  
  00664E5D   |>  43              |inc ebx  
  00664E5E   |.  83FB 0D         |cmp ebx,0D  
  00664E61   |.^ 0F85 4BFFFFFF    \\jnz dsvPHPed.00664DB2  
```

用伪代码描述逻辑如下：

```c
k=1;  
for(int i=0;k<=12;){  
  int tmp=sum;  
  tmp+=k;  
  tmp%=0xC8;  
  int j=k;  
  j+=j;  
  int tmp2=strS[j-1] ^strS[k-1];  
  tmp2^=strName[i];  
  tmp^=tmp2;  
  char strTmp[5];  
  sprintf(strTmp,"%02X",tmp);  
  strncat(strCode,strTmp,2);  
  tmp=k^strName[i];  
  sum+=tmp;  
  k++;  
  i++;  
  if(i==len)  
    i=0;  
}  
```

后续汇编代码：

```asm
  00664E67   |.  8B45 E8         mov eax,dword ptr ss:[ebp-18]  
  00664E6A   |.  E8 2908DAFF     call dsvPHPed.00405698              ;  取得到字符串的长度  
  00664E6F   |.  83F8 18        cmp eax,18                            ;  与 0x18 相比  
  00664E72   |.  7E 16          jle short dsvPHPed.00664E8A  
  00664E8A   |> \\8D45 CC       lea eax,dword ptr ss:[ebp-34]  
  00664E8D   |.  50            push eax  
  00664E8E   |.  B9 08000000   mov ecx,8  
  00664E93   |.  BA 01000000   mov edx,1  
  00664E98   |.  8B45 E8       mov eax,dword ptr ss:[ebp-18]  
  00664E9B   |.  E8 580ADAFF   call dsvPHPed.004058F8  
  00664EA0   |.  FF75 CC       push dword ptr ss:[ebp-34]  
  00664EA3   |.  68 744F6600   push dsvPHPed.00664F74  
  00664EA8   |.  8D45 C8       lea eax,dword ptr ss:[ebp-38]  
  00664EAB   |.  50            push eax  
  00664EAC   |.  B9 08000000   mov ecx,8  
  00664EB1   |.  BA 09000000   mov edx,9  
  00664EB6   |.  8B45 E8       mov eax,dword ptr ss:[ebp-18]  
  00664EB9   |.  E8 3A0ADAFF   call dsvPHPed.004058F8  
  00664EBE   |.  FF75 C8       push dword ptr ss:[ebp-38]  
  00664EC1   |.  68 744F6600   push dsvPHPed.00664F74  
  00664EC6   |.  8D45 C4       lea eax,dword ptr ss:[ebp-3C]  
  00664EC9   |.  50            push eax  
  00664ECA   |.  B9 08000000   mov ecx,8  
  00664ECF   |.  BA 11000000   mov edx,11  
  00664ED4   |.  8B45 E8       mov eax,dword ptr ss:[ebp-18]  
  00664ED7   |.  E8 1C0ADAFF   call dsvPHPed.004058F8  
```

上述三次调用将得到的 24 位字符用 `-` 分隔成三部分，每部分 8 位。描述如下：

```c
char strTmp2[STR_LEN]={0};  
strncpy(strTmp2,&strCode[0],8);  
strcat(strReg,strTmp2);  
strReg[8]='-';  
strncpy(strTmp2,&strCode[8],8);  
strcat(strReg,strTmp2);  
strReg[17]='-';  
strncpy(strTmp2,&strCode[16],8);  
strcat(strReg,strTmp2);  
```

函数结束部分：

```asm
  00664F23    .  5E            pop esi  
  00664F24    .  5B            pop ebx  
  00664F25    .  8BE5          mov esp,ebp  
  00664F27    .  5D            pop ebp  
  00664F28    .  C3            retn  
```

`****************************** 算法 CALL 结束 ******************************`

完整 C 描述如下：

```c
#define STR_LEN    64  
char strName[STR_LEN];  
char strCode[STR_LEN];  
char strS[]="{0E8E3B8C-6E33-4266-A116-8E72E5295C9F}";  
char strReg[STR_LEN];  

void CreateRegCode(HWND hDlg){  
  int result=GetDlgItemText(hDlg,IDC_USERNAME,strName,20);  
  if(result==0){  
    MessageBox(hDlg,TEXT("请输入用户名！"),TEXT("Error!"),MB_OK);  
    return;  
  }  
  memset(strReg,0,STR_LEN);  
  memset(strCode,0,STR_LEN);  
  int sum=0;  
  int k=1;  
  int len=strlen(strName);  
  for(int i=0;i<len;i++){  
    if(strName[i]>='a' && strName[i]<='z')  
      strName[i]-=32;  
    sum+=(strName[i]*k);  
    k++;  
  }  
  k=1;  
  for(int i=0;k<=12;){  
    int tmp=sum;  
    tmp+=k;  
    tmp%=0xC8;  
    int j=k;  
    j+=j;  
    int tmp2=strS[j-1] ^strS[k-1];  
    tmp2^=strName[i];  
    tmp^=tmp2;  
    char strTmp[5];  
    sprintf(strTmp,"%02X",tmp);  
    strncat(strCode,strTmp,2);  
    tmp=k^strName[i];  
    sum+=tmp;  
    k++;  
    i++;  
    if(i==len)  
      i=0;  
  }  
  char strTmp2[STR_LEN]={0};  
  strncpy(strTmp2,&strCode[0],8);  
  strcat(strReg,strTmp2);  
  strReg[8]='-';  
  strncpy(strTmp2,&strCode[8],8);  
  strcat(strReg,strTmp2);  
  strReg[17]='-';  
  strncpy(strTmp2,&strCode[16],8);  
  strcat(strReg,strTmp2);  
  SetDlgItemText(hDlg,IDC_PWD,strReg);  
  return;  
}  
```

--------------------------------------------------------------------------------

【版权声明】: 转载请注明作者并保持文章的完整, 谢谢!

2008 年 02 月 13 日 1:27:11

附件为注册机的源码！

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `DSVeditorKeyGen.rar` |
| :--- | :--- | :--- |

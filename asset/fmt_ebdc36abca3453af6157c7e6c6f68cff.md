
```
1000AFB0   |.   83F9 38           |cmp ecx,38                                ;  ECX<38 吗  

1000AFB3   |.^ 7C AD             \jl short UIClass.1000AF62  

1000AFB5   |.   8D4C24 28        lea ecx,dword ptr ss:[esp+28]  

1000AFB9   |.   8D5424 10        lea edx,dword ptr ss:[esp+10]  

1000AFBD   |.   51             push ecx  

1000AFBE   |.   68 F4300210    push UIClass.100230F4                       ;  ASCII "%s"  

1000AFC3   |.   52             push edx  

1000AFC4   |.   E8 09CA0000    call <jmp.&MFC42.#2818_CString::Format>  

1000AFC9   |.   83C4 0C        add esp,0C  

1000AFCC   |>   8D4C24 10      lea ecx,dword ptr ss:[esp+10]  

1000AFD0   |.   E8 F7C90000    call <jmp.&MFC42.#4204_CString::MakeUpper>  

1000AFD5   |.   8B7424 44      mov esi,dword ptr ss:[esp+44]  
```

注册机算法：

```c
#include<iostream.h>  

#include <conio.h>  

#include <stdlib.h>  

#include<math.h>  

#include<string.h>  

void main()  

{  

  char jqm[8]={0}/*  这是我的机器码 AP8UFEPM   */;  

  char name[12]={0}, zhucema[12]={0};  

  unsigned int len, tmp=0, sum=0, ebp=0, edx=0, ecx=0, tt=0;  

  double res;  

    cout<<"请输入 3 位用户名"<<endl;  

  cin>>name;  

  len=strlen(name);  

  if(len!=3)  

  {  

    cout<<"请输入 3 位用户名"<<endl;  

      exit(0);  

  }  

    cout<<"请输入 8 位机器码"<<endl;  

  cin>>jqm;  

  cout<<"name is:"<<name<<endl<<"jqm is :"<<jqm<<endl;  

  strcat(name, jqm);  

  len=strlen(name);  

  cout<<"cat name is:"<<name<<endl;  

  for(unsigned int i=0; i<len; i++)  

  {  

    tmp=name[i];  

    res=sqrt(tmp);//用户名求平方根  

    res*=(i+1);  

    res+=1;  

    tmp*=(i+1);  

    tmp*=(i+1);  

    res*=tmp;  

    tt=res;//取整保存  

    sum+=tt;  

    sum=sum%0x186A0;  

    tmp=pow(name[i],2)*(i+1);//后面利用  

    ///////////////////////////////////////////////  

    ecx=i+1;  

    ecx*=ebp;  

    tmp+=ecx;  

        res=sqrt(sum);  

        ebp=tmp%0x186A0;//后面利用  

    ///////////////////////////////////////////  

    tmp=res;//取整  

    tmp+=ebp;  

        edx=tmp%0x186A0;//后面用  

  }  

  ecx=0;  

  for(i=0; i<3; i++)  

  {  

    ecx=i;  

    tmp=sum;  

    ecx*=i;  

    ecx*=i;  

    ecx=ecx+tmp+0x1f;  

    ecx=ecx&0x8000007F;  

    zhucema[i]=ecx;  

  }  

  for(i=3; i<6; i++)  

  {  

    ecx=i;  

    tmp=ebp;  

    ecx*=i;  

    ecx*=i;  

    ecx=ecx+tmp+0x1f;  

    ecx=ecx&0x8000007F;  

    zhucema[i]=ecx;  

  }  

  for(i=3; i<8; i++)  

  {  

    ecx=i;  

    tmp=edx;  

    ecx*=i;  

    ecx*=i;  

    ecx=ecx+tmp+0x1f;  

    ecx=ecx&0x8000007F;  

    zhucema[i]=ecx;  

  }  

  ecx=0;  

  for(i=0; i<8; i++)  

  {  

    if(zhucema[i]<'0'||zhucema[i]>'9')//  

    {  

      if(zhucema[i]<'A'||zhucema[i]>'Z')//  

      {  

        if(zhucema[i]<'a'||zhucema[i]>'z')//  

        {  

          tmp=zhucema[i];  

            tmp=ecx+tmp+0x1f;  

          tmp=tmp&0x8000007F;  

          zhucema[i]=tmp;  

          if(zhucema[i]<'0'||zhucema[i]>'9')//  

          {  

            if(zhucema[i]<'A'||zhucema[i]>'Z')//  

            {  

              if(zhucema[i]<'a'||zhucema[i]>'z')//  

              {  

                tmp=zhucema[i];  

                tmp=ecx+tmp+0x1f;  

                tmp=tmp&0x8000007F;  

                zhucema[i]=tmp;  

              }  

            }  

          }  

        }  

      }  

      ecx+=7;  

    }  

  }  

  for(i=0; i<8; i++)  

  {  

    if(zhucema[i]==0x4f)  

      zhucema[i]=0x44;  

    else  

      if(zhucema[i]==0x6f)  

      zhucema[i]=0x48;  

  }  

  cout<<zhucema<<endl;  

}
```

\==============================================================================================  

如有得罪或疏漏之处请谅解，下次再会！

```
0041812B  |.  8B69 F8          mov ebp,dword ptr ds:[ecx-8]  

0041812E  |.  896C24 34        mov dword ptr ss:[esp+34],ebp  

00418132  |.  E8 9FF30100      call tcsearch.004374D6  

00418137  |.  8B00             mov eax,dword ptr ds:[eax]  

00418139  |.  8D4C24 28        lea ecx,dword ptr ss:[esp+28]  

0041813D  |.  68 9CD74700      push tcsearch.0047D79C                  ; /Arg2 = 0047D79C ASCII "%Y%m%d"  

00418142  |.  51               push ecx                                ; |Arg1  

00418143  |.  8D4C24 34        lea ecx,dword ptr ss:[esp+34]           ; |  

00418147  |.  894424 34        mov dword ptr ss:[esp+34],eax          ; |  

0041814B  |.  E8 99F30100      call tcsearch.004374E9                  ; \EAX=系统日期 20050430  

00418150  |.  3BEF             cmp ebp,edi  

00418152  |.  C64424 58 03      mov byte ptr ss:[esp+58],3  

00418157  |.  0F84 34020000     je tcsearch.00418391  

0041815D  |.  BE 01000000      mov esi,1                               ;  ESI=1  

00418162  |.  3BEE             cmp ebp,esi  

00418164  |.  897424 1C        mov dword ptr ss:[esp+1C],esi  

00418168  |.  0F8C 37010000     jl tcsearch.004182A5  

0041816E  |.  8B4424 28        mov eax,dword ptr ss:[esp+28]           ;  EAX=系统日期 20050430  

00418172  |.  DD05 80664400    fld qword ptr ds:[446680]  

00418178  |.  8A58 06          mov bl,byte ptr ds:[eax+6]              ;  BL=date[6]  

0041817B  |.  8A48 04          mov cl,byte ptr ds:[eax+4]              ;  CL=date[4]  

0041817E  |.  885C24 13        mov byte ptr ss:[esp+13],bl             ;  [ESP+13]=BL=date[6]  

00418182  |.  8A58 07          mov bl,byte ptr ds:[eax+7]              ;  BL=date[7]  

00418185  |.  0FBEDB           movsx ebx,bl                            ;  ebx=bl  

00418188  |.  895C24 24        mov dword ptr ss:[esp+24],ebx           ;  [esp+24]=ebx=date[7]  

0041818C  |.  8A50 05          mov dl,byte ptr ds:[eax+5]              ;  DL=date[5]  

0041818F  |.  DB4424 24        fild dword ptr ss:[esp+24]             ;  ST0=[esp+24]=date[7]  

00418193  |.  0FBE5C24 13      movsx ebx,byte ptr ss:[esp+13]          ;  EBX=[ESP+13]=date[6]  

00418198  |.  D9FA             fsqrt                                   ;  ST1=SQRT（ST0）  

0041819A  |.  895C24 24        mov dword ptr ss:[esp+24],ebx           ;  [ESP+24]=EBX=date[6]  

0041819E  |.  8A40 03          mov al,byte ptr ds:[eax+3]              ;  AL=date[3]  

004181A1  |.  0FBEC0           movsx eax,al                            ;  eax=date[3]  

004181A4  |.  0FBEEA           movsx ebp,dl                            ;  EBP=DL=date[5]  

004181A7  |.  0FBEC9           movsx ecx,cl                            ;  ECX=CL  

004181AA  |.  DB4424 24        fild dword ptr ss:[esp+24]             ;  ST0=date[6]  

004181AE  |.  0FAFE9           imul ebp,ecx                            ;  EBP=EBP*ECX 这个数据要记住 ！  

004181B1  |.  D9FA             fsqrt                                   ;  ST0=SQRT（ST0）  

004181B3  |.  894424 24        mov dword ptr ss:[esp+24],eax           ;  [ESP+24]=EAX=date[3]  

004181B7  |.  DEC9             fmulp st(1),st                          ;  RES=ST0*ST1  

004181B9  |.  DD5C24 38        fstp qword ptr ss:[esp+38]              ;  这个数据要记住 ！  

004181BD  |.  DB4424 24        fild dword ptr ss:[esp+24]             ;  ST0=date[3]****这个重要  

004181C1  |.  D9FA             fsqrt                                   ;  SQRT（ST0）  

004181C3  |.  DCC0             fadd st,st  

004181C5  |.  DD5C24 40        fstp qword ptr ss:[esp+40]  

004181C9  |>  8B5424 64        /mov edx,dword ptr ss:[esp+64]          ;  EDX=NAME  

004181CD  |.  DB4424 1C        |fild dword ptr ss:[esp+1C]  

004181D1  |.  8A5C32 FF       |mov bl,byte ptr ds:[edx+esi-1]         ;  BL=NAME[I]  

004181D5  |.  0FBEC3          |movsx eax,bl                           ;  EAX=NAME[I]  

004181D8  |.  DD5C24 1C       |fstp qword ptr ss:[esp+1C]  

004181DC  |.  894424 24       |mov dword ptr ss:[esp+24],eax  
```

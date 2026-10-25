
### 汇编分析

`:00477C58 43`

`inc ebx`

`:00477C59 4E`

`dec esi`

`:00477C5A 75E0`

`jne 00477C3C` ->比较是否取完，没有继续

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

`|:00477B80(U)`

|

`:00477C5C`

`837DFC00                cmp dword ptr [ebp-04], 00000000` ->比较end是否取完

`:00477C60 0F851FFFFFFF`

`jne 00477B85`

->没有则继续，后面位数的取法，见注册机源码

`:00477C66 33C0`

`xor eax, eax`

`:00477C68`

`5A                      pop edx`

`:00477C69 59`

`pop ecx`

`:00477C6A 59`

`pop ecx`

`:00477C6B 648910`

`mov dword ptr fs:[eax], edx`

`:00477C6E`

`688B7C4700              push 00477C8B`

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

`|:00477C89(U)`

|

`:00477C73`

`8D45F0                  lea eax, dword ptr [ebp-10]`

`:00477C76 E829BFF8FF`

`call 00403BA4`

`:00477C7B 8D45FC`

`lea eax, dword ptr [ebp-04]`

`:00477C7E E821BFF8FF`

`call 00403BA4`

`:00477C83 C3`

`ret`

下面为注册机源码：CB

v6.0 win98 SE下调试通过

呵呵，可能是自己的编程水平不高，只能写出这样的源程序，我想它应该可以更简化一些。

```cpp
void
__fastcall Tform1::OKBtnClick(TObject *Sender)
{
 int key_11[29]={'~','T','u','r','n','I','d','e','@','$','I',
             'n','t','o','P','r','o','f','i','t','$','w',
             '/','e','B','o','o','k'},*p=key_11;
int key_22[15]={'0','0','0','0','0','0','0','0','0','0','0',
            '0','0','0','0'} ,*p1=key_22;
String name,key_1,key_2,T,key_3,name_key,code_1,code_2,code_3,
        code_4,code_5,code_6,code_7,code_8,code_9,
          code_10,code_11,code_12,code_13,code_14,
            code_15,code_16;
 int a,b=1,c=1,d=0,d_1=0,j=1,k=1;
unsigned long e=0,f=0,g=0,h=0,i=0,l=0,m=0,x=0;
 if(UEdit->Text=="")
{CEdit->Text="未输入注册名！";return;}
 if(UEdit->Text.Length()>10)
{CEdit->Text="注册名不能大于10位！";return;}
 if(UEdit->Text!="")
  {
    name=UEdit->Text;a=UEdit->Text.Length();
   while(c<=12)
         {
     if(c>a) {name=name+name;d++;e=name[d];}
         else e=name[c];
   f=e^a;g=f^b;
          c++;b++;
         key_1=key_1+char(g);
  }
   while(a>=1)
         {
       h=name[a]^key_1[j];
 a--;j++;
          key_2=key_2+char(h);
        }
   if(j<12)
     {
   while(j<=12)
           {
         x=key_1[j];
   name_key=name_key+char(x);j++;
  }
     key_2=key_2+name_key;
     while(k<=12)
        {
          i=key_2[k]^ *p1;
         l=i^ *p;m=l^k;k++;p++;p1++;
         key_3=key_3+char(m);
  }
     unsigned long a_1=0,b_1=0,c_1=0,a_11=0,b_11=0,c_11=0,
                 c_12=0,c_13=0,c_14=0,c_15=0;
   a_1=key_3[3]&0x3f;
    b_1=key_3[2]<<2;
   b_11=key_3[2]>>6;
    c_15=((b_11|b_1)^0x100)&0x3f;
   c_1=((b_11|b_1)&0x100)+key_3[1];
    c_11=c_1<<4;c_12=key_3[2]>>4;
   c_13=(c_11|c_12)&0x3f;
    c_14=(c_1&0xff)>>2;
   char key_111[]={'I','Y','A','G','P','X','D','J','Q','W',
                 'M','H','V','C','N','F','U','Z','R','B',
                   'K','E','S','O','L','T','t','f','k','y',
                   's','b','o','h','l','u','j','w','e','c',
                   'p','m','i','a','q','n','d','x','z','v',
                   'g','r','4','6','+','0','2','5','7','3',
                   '/','8','1','=','9'};
   code_1=key_111[c_14];code_2=key_111[c_13];
    code_3=key_111[c_15];code_4=key_111[a_1];
   unsigned long aa_1=0,ab_1=0,ac_1=0,aa_11=0,ab_11=0,ac_11=0,
               ac_12=0,ac_13=0,ac_14=0,ac_15=0;
   aa_1=key_3[6]&0x3f;
    ab_1=key_3[5]<<2;
   ab_11=key_3[6]>>6;
    ac_15=((ab_11|ab_1)^0x100)&0x3f;
   ac_1=((ab_11|ab_1)&0x100)+key_3[4];
    ac_11=ac_1<<4;ac_12=key_3[5]>>4;
   ac_13=(ac_11|ac_12)&0x3f;
    ac_14=(ac_1&0xff)>>2;
   code_5=key_111[ac_14];code_6=key_111[ac_13];
    code_7=key_111[ac_15];code_8=key_111[aa_1];
   unsigned long ba_1=0,bb_1=0,bc_1=0,ba_11=0,bb_11=0,bc_11=0,
               bc_12=0,bc_13=0,bc_14=0,bc_15=0;
   ba_1=key_3[9]&0x3f;
    bb_1=key_3[8]<<2;
   bb_11=key_3[9]>>6;
    bc_15=((bb_11|bb_1)^0x100)&0x3f;
   bc_1=((bb_11|bb_1)&0x100)+key_3[7];
    bc_11=bc_1<<4;bc_12=key_3[8]>>4;
   bc_13=(bc_11|bc_12)&0x3f;
    bc_14=(bc_1&0xff)>>2;
   code_9=key_111[bc_14];code_10=key_111[bc_13];
 code_11=key_111[bc_15];code_12=key_111[ba_1];
    unsigned
long ca_1=0,cb_1=0,cc_1=0,ca_11=0,cb_11=0,cc_11=0,
         cc_12=0,cc_13=0,cc_14=0,cc_15=0;
 ca_1=key_3[12]&0x3f;
    cb_1=key_3[11]<<2;
 cb_11=key_3[12]>>6;
    cc_15=((cb_11|cb_1)^0x100)&0x3f;
   cc_1=((cb_11|cb_1)&0x100)+key_3[10];
    cc_11=cc_1<<4;cc_12=key_3[11]>>4;
   cc_13=(cc_11|cc_12)&0x3f;
    cc_14=(cc_1&0xff)>>2;
   code_13=key_111[cc_14];code_14=key_111[cc_13];
 code_15=key_111[cc_15];code_16=key_111[ca_1];
    CEdit->Text=CEdit->Text+code_1+code_2+code_3+code_4
   +code_5+code_6+code_7+code_8+code_9+code_10+code_11
    +code_12+code_13+code_14+code_15+code_16;
```

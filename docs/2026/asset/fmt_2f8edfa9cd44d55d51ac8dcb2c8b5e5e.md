
# 反汇编代码

```asm
00409D98   |.  C74424 1C 01000000 mov dword ptr ss:[esp+1C],1

00409DA0   |.  894424 0C          mov dword ptr ss:[esp+C],eax

00409DA4   |.  8B4C24 28          mov ecx,dword ptr ss:[esp+28]

00409DA8   |.  C64424 1C 02       mov byte ptr ss:[esp+1C],2

00409DAD   |.  8B69 F8            mov ebp,dword ptr ds:[ecx-8]

00409DB0   |.  8D7D 01            lea edi,dword ptr ss:[ebp+1]

00409DB3   |.  57                 push edi

00409DB4   |.  E8 8CC70100        call Registry.00426545

00409DB9   |.  8BCF               mov ecx,edi

00409DBB   |.  8BF0               mov esi,eax

00409DBD   |.  8BD1               mov edx,ecx

00409DBF   |.  33C0               xor eax,eax

00409DC1   |.  8BFE               mov edi,esi

00409DC3   |.  83C4 04            add esp,4

00409DC6   |.  C1E9 02            shr ecx,2

00409DC9   |.  F3:AB              rep stos dword ptr es:[edi]

00409DCB   |.  8BCA               mov ecx,edx

00409DCD   |.  83E1 03            and ecx,3

00409DD0   |.  F3:AA              rep stos byte ptr es:[edi]

00409DD2   |.  33C9               xor ecx,ecx

00409DD4   |.  85ED               test ebp,ebp

00409DD6   |.  7E 37              jle short Registry.00409E0F

00409DD8   |>  8B4424 28          /mov eax,dword ptr ss:[esp+28]

00409DDC   |.  8A0401             |mov al,byte ptr ds:[ecx+eax]

00409DDF   |.  3C 41              |cmp al,41

00409DE1   |.  74 22              |je short Registry.00409E05                    ;  如果RES[I]是41=A或61=a就RES[I]+=1

00409DE3   |.  3C 61              |cmp al,61

00409DE5   |.  74 1E              |je short Registry.00409E05

00409DE7   |.  3C 5A              |cmp al,5A                                     ;  如果RES[I]是5A=Z或7A=z就RES[I]-=1

00409DE9   |.  74 16              |je short Registry.00409E01

00409DEB   |.  3C 7A              |cmp al,7A

00409DED   |.  74 12              |je short Registry.00409E01

00409DEF   |.  0FBED0             |movsx edx,al

00409DF2   |.  81E2 01000080      |and edx,80000001                              ;  res[i] and 80000001

00409DF8   |.  79 05              |jns short Registry.00409DFF

00409DFA   |.  4A                 |dec edx

00409DFB   |.  83CA FE            |or edx,FFFFFFFE

00409DFE   |.  42                 |inc edx

00409DFF   |>  74 04              |je short Registry.00409E05                    ;  res[i] and 80000001等于0就res[i]+1

00409E01   |>  FEC8               |dec al                                        ;  res[i] and 80000001不等于0就res[i]-1

00409E03   |.  EB 02              |jmp short Registry.00409E07

00409E05   |>  FEC0               |inc al

00409E07   |>  880431             |mov byte ptr ds:[ecx+esi],al                  ;  形成真码

00409E0A   |.  41                 |inc ecx

00409E0B   |.  3BCD               |cmp ecx,ebp

00409E0D   |.^ 7C C9              \jl short Registry.00409DD8

00409E0F   |>  56                 push esi

00409E10   |.  8D4C24 10          lea ecx,dword ptr ss:[esp+10]

00409E14   |.  E8 30CB0100        call Registry.00426949

00409E19   |.  56                 push esi

00409E1A   |.  E8 4FC70100        call Registry.0042656E

00409E1F   |.  8B7424 28          mov esi,dword ptr ss:[esp+28]

00409E23   |.  83C4 04            add esp,4

00409E26   |.  8D4424 0C          lea eax,dword ptr ss:[esp+C]

00409E2A   |.  8BCE               mov ecx,esi
```

////////////////////////////////////////////////////////////////////////////////////////////////////

## 总结

注册方式为对机器码进行BASE64加密后对其结果进行了一些处理形成注册码，详见注册机C源码。

BASE64C代码参考了网上不知哪位大虾编的。

## BASE64 代码参考

```c
#include  <stdio.h>

#include  <stdlib.h>

#include  <string.h>

#include  <conio.h>  /*  Windows  special  */

char*  alphabet  =  "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=";

char*  encode(char  data[],  int  length)

{

        int  i,  index,  val;

        int  quad,  trip;    /*  bool  type:  0  or  1  */

        char*  out;

    //out = (char*)malloc((((length+2)/3)*4)*sizeof(char));
    out=new char[(((length+2)/3)*4)*sizeof(char)];

        for  (i=0,  index=0;  i<length;  i+=3,  index+=4)

        {

               quad  =  0;
               trip  =  0;
                           val=(0xFF  &  (int)  data[i]);
               val<<=8;
               if((i+1)<length)
                           {
                       val|=(0xFF&(int)data[i+1]);
                       trip = 1;
               }
               val<<=8;
               if  ((i+2)<length)
                           {
                       val|=(0xFF&(int)data[i+2]);
                       quad  =  1;
               }
               out[index+3]  =  alphabet[(quad ?(val&0x3F):64)];
               val>>=6;
               out[index+2]  =  alphabet[(trip?(val&0x3F):64)];
               val>>=6;
               out[index+1]  =  alphabet[val&0x3F];
               val>>=6;
               out[index+0]  =  alphabet[val&0x3F];
       }

        out[index]  =  '\0';
       return  out;

}

void  init_codes(char  codes[])

{

int  i;
for  (i =0; i< 256; i++)
           codes[i]  = -1;
       for  (i = 'A'; i <= 'Z'; i++)
                   codes[i] = i -'A';
       for  (i='a'; i<='z';i++)
                   codes[i] = 26 + i- 'a';
       for  (i='0';i<='9'; i++)
                   codes[i]= 52+i-'0';
       codes['+'] = 62;
       codes['/'] = 63;
}

char*  decode(char  data[],int  length)

{

        int  value,  ix;
       int  shift  =  0;      /*  #  of  excess  bits  stored  in  accum  */
       int  accum  =  0;      /*  excess  bits  */
       int  index  =  0;
       int  len;
           char  codes[256];
           char*  out;
           len=((length + 3)/4)*3;
       if(length>0 && data[length-1] =='=')
                   --len;
       if  (length>1&&data[length-2] =='=')
                   --len;
           printf("%d\n",sizeof(char));
           //out  =  (char*)malloc(len  *  sizeof(char));
        out=new char[len*sizeof(char)];
           init_codes(codes);
       for  (ix=0;  ix<length;  ix++)
           {
               value = codes[data[ix] & 0xFF];  /*  ignore  high  byte  of  char  */
               if  (  value  >=  0  )
                           {          /*  skip  over  non-code  */
                       accum  <<= 6;                        /*  bits  shift  up  by  6  each  time  thru  */
                       shift  +=  6;                          /*  loop,  with  new  bits  being  put  in  */
                       accum  |=  value;                  /*  at  the  bottom.  */
                       if  (  shift  >=  8  )
                                           {          /*  whenever  there  are  8  or  more  shifted  in,  */
                               shift  -=  8;                  /*  write  them  out  (from  the  top,  leaving  any  */
                               out[index++]  =  ((accum  >>  shift)  &  0xff);            /*  excess  at  the  bottom  for  next  iterati  on.  */
     }
   }
}

           out[index]  =  '\0';
           if  (index  !=  len)
                   printf("miscalculated  data  length!\n");
return  out;

}

char*  fu1(char  *data,  int  length)

{

        int    index;
        char*  out1;
     out1=new char[length*sizeof(char)];
         strcpy(out1,data);
        for  (index=0;  index<length; index++)
        {
                if(out1[index]=='a'||out1[index]=='A')
          out1[index]=out1[index]+1;
                else if(out1[index]=='Z'||out1[index]=='z')
              out1[index]=out1[index]-1;
                else if((out1[index]&0x80000001)==0)
              out1[index]=out1[index]+1;
                else out1[index]=out1[index]-1;
       }
        out1[index-1]  =  '\0';
       return  out1;

}

void   main()

{

        char data1[255]={0},data2[255]={0};
printf("请输入机器码： \n");
        scanf( "%s",  &data2 );

char*  out1;
char*  out2;
char*  res;
out2  =  encode(data2,strlen(data2)); //加密
```

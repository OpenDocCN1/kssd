
```
0058AD9E         |.   6A 04             push 4        //关闭锁

0058ADA0         |.   FF15 94E16400    call dword ptr ds:[64E194]

0058ADA6         |.   83C4 24          add esp,24

0058ADA9         |.   33C0             xor eax,eax

0058ADAB         |.   E9 6F010000      jmp d1.0058AF1F

0058ADB0         |>   66:8B55 F4      mov dx,word ptr ss:[ebp-C]

0058ADB4         |.   66:2BD6         sub dx,si

0058ADB7         |.   66:3355 F2      xor dx,word ptr ss:[ebp-E]

0058ADBB         |.   66:8953 04      mov word ptr ds:[ebx+4],dx

0058ADBF         |.   C745 FC 1000000>mov dword ptr ss:[ebp-4],10

0058ADC6         |.   8B4D 0C         mov ecx,dword ptr ss:[ebp+C]

0058ADC9         |.   8D0C89          lea ecx,dword ptr ds:[ecx+ecx*4]

0058ADCC         |.   41               inc ecx

0058ADCD         |.   894D F8         mov dword ptr ss:[ebp-8],ecx

0058ADD0         |.   E8 E3FCFFFF     call d1.0058AAB8

0058ADD5         |.   8BF8             mov edi,eax

0058ADD7         |.   66:8945 F4      mov word ptr ss:[ebp-C],ax

0058ADDB         |.   E8 D8FCFFFF     call d1.0058AAB8

0058ADE0         |.   8BF0             mov esi,eax

0058ADE2         |.   66:8945 F2      mov word ptr ss:[ebp-E],ax

0058ADE6         |.   E8 CDFCFFFF     call d1.0058AAB8

0058ADEB         |.   66:8945 CE      mov word ptr ss:[ebp-32],ax

0058ADEF         |.   66:8945 F0      mov word ptr ss:[ebp-10],ax

0058ADF3         |.   E8 C0FCFFFF     call d1.0058AAB8

0058ADF8         |.   66:8945 CC      mov word ptr ss:[ebp-34],ax

0058ADFC         |.   66:8945 EE      mov word ptr ss:[ebp-12],ax

0058AE00         |.   8D55 D4         lea edx,dword ptr ss:[ebp-2C]

0058AE03         |.   52               push edx

0058AE04         |.   8D4D EE         lea ecx,dword ptr ss:[ebp-12]  //输入值 4  返回值 4

0058AE07         |.   51               push ecx

0058AE08         |.   8D45 F0         lea eax,dword ptr ss:[ebp-10]  //输入值 3  返回值 3

0058AE0B         |.   50               push eax

0058AE0C         |.   8D55 F2         lea edx,dword ptr ss:[ebp-E]  //输入值 2  返回值 2

0058AE0F         |.   52               push edx

0058AE10         |.   8D4D F4         lea ecx,dword ptr ss:[ebp-C]  //输入值 1  返回值 1

0058AE13         |.   51               push ecx

0058AE14         |.   8D45 F8         lea eax,dword ptr ss:[ebp-8]  //密码超始地址

0058AE17         |.   50               push eax

0058AE18         |.   8D55 FC         lea edx,dword ptr ss:[ebp-4]  //计算起始点

0058AE1B         |.   52               push edx

0058AE1C         |.   8D4D F6         lea ecx,dword ptr ss:[ebp-A]

0058AE1F         |.   51               push ecx      //锁的句柄

0058AE20         |.   6A 10            push 10        //计算 3

0058AE22         |.   FF15 94E16400    call dword ptr ds:[64E194]

/*

WORD Rockey(WORD function, WORD* handle, DWORD* lp1,  DWORD* lp2, WORD* p1, WORD* p2, WORD* p3, WORD* p4, BYTE* buffer)

(16) 计算 3

     function = 15

     *handle = 锁的句柄

     *lp1 = 计算起始点

     *lp2 = 密码起始地址

     *p1 = 输入值 1

     *p2 = 输入值 2

     *p3 = 输入值 3

     *p4 = 输入值 4

     返回:

     *p1 = 返回值 1

     *p2 = 返回值 2

     *p3 = 返回值 3

     *p4 = 返回值 4

     返回为 0 表示成功, 其它为错误码

A  P1

B  P2

C  P3

D  P4

E  lp2 中指定的模块内容

F  lp2+1 中指定的模块内容

G  lp2+2 中指定的模块内容

H  lp2+3 中指定的模块内容

*/

0058AE28         |.   83C4 24          add esp,24

0058AE2B         |.   66:85C0         test ax,ax

0058AE2E         |.   74 32            je short d1.0058AE62

0058AE30         |.   8D45 D4         lea eax,dword ptr ss:[ebp-2C]

0058AE33         |.   50               push eax

0058AE34         |.   8D55 EE         lea edx,dword ptr ss:[ebp-12]

0058AE37         |.   52               push edx

0058AE38         |.   8D4D F0         lea ecx,dword ptr ss:[ebp-10]

0058AE3B         |.   51               push ecx

0058AE3C         |.   8D45 F2         lea eax,dword ptr ss:[ebp-E]

0058AE3F         |.   50               push eax

0058AE40         |.   8D55 F4         lea edx,dword ptr ss:[ebp-C]

0058AE43         |.   52               push edx

0058AE44         |.   8D4D F8         lea ecx,dword ptr ss:[ebp-8]

0058AE47         |.   51               push ecx

0058AE48         |.   8D45 FC         lea eax,dword ptr ss:[ebp-4]

0058AE4B         |.   50               push eax

0058AE4C         |.   8D55 F6         lea edx,dword ptr ss:[ebp-A]

0058AE4F         |.   52               push edx

0058AE50         |.   6A 04            push 4        //关闭锁

0058AE52         |.   FF15 94E16400    call dword ptr ds:[64E194]

0058AE58         |.   83C4 24          add esp,24

0058AE5B         |.   33C0             xor eax,eax

0058AE5D         |.   E9 BD000000     jmp d1.0058AF1F

0058AE62         |>   6A 05            push 5                                  ; /Arg2 = 00000005

0058AE64         |.   66:337D F4      xor di,word ptr ss:[ebp-C]             ; |

0058AE68         |.   57               push edi                               ; |Arg1

0058AE69         |.   E8 6DFCFFFF     call d1.0058AADB                       ; \d1.0058AADB

0058AADB         /$   55               push ebp

0058AADC         |.   8BEC             mov ebp,esp

0058AADE         |.   53               push ebx

0058AADF         |.   8B55 0C         mov edx,dword ptr ss:[ebp+C]  //参数 00000007>edx

0058AAE2         |.   8B45 08         mov eax,dword ptr ss:[ebp+8]  //参数 001229c3>eax

0058AAE5         |.   8BCA             mov ecx,edx     //拷贝

0058AAE7         |.   0FB7D8          movzx ebx,ax     //ebx=p4=  29C3

0058AAEA         |.   D3FB             sar ebx,cl  //cl=7  //循环右移 7 位=00000053

0058AAEC         |.   B9 10000000     mov ecx,10  //10

0058AAF1         |.   2BCA             sub ecx,edx  //ecx=9

0058AAF3         |.   66:D3E0         shl ax,cl  //左移 p4 9 位

0058AAF6         |.   66:0BD8         or bx,ax  //或 p4 第一次运数  ，p4 第三次运数  最后结果 00008653

0058AAF9         |.   8BC3             mov eax,ebx  //将最后结果放入 eax

0058AAFB         |.   5B               pop ebx    //出

0058AAFC         |.   5D               pop ebp

0058AAFD         \.   C3               retn

0058AE6E         |.   83C4 08          add esp,8
```


### 汇编代码

```
:005C7FD4 8BF9                    mov edi, ecx     // edi=ecx
:005C7FD6 C1E70D                  shl edi, 0D      // edi=edi << 0xd
:005C7FD9 33CF                    xor ecx, edi     // ecx=ecx xor edi
:005C7FDB 03C1                    add eax, ecx     // eax=eax+ecx
:005C7FDD 03C8                    add ecx, eax     // ecx=ecx+eax
:005C7FDF 8BF8                    mov edi, eax     // edi=eax
:005C7FE1 C1EF11                  shr edi, 11      // edi=edi >> 0x11
:005C7FE4 33C7                    xor eax, edi     // eax=eax xor edi
:005C7FE6 03D8                    add ebx, eax     // ebx=ebx+eax
:005C7FE8 03C3                    add eax, ebx     // eax=eax+ebx
:005C7FEA 8BFB                    mov edi, ebx     // edi=ebx
:005C7FEC C1E709                  shl edi, 09      // edi=edi << 0x9
:005C7FEF 33DF                    xor ebx, edi     // ebx=ebx xor edi
:005C7FF1 03D3                    add edx, ebx     // edx=edx+ebx
:005C7FF3 03DA                    add ebx, edx     // ebx=ebx+edx
:005C7FF5 8BFA                    mov edi, edx     // edi=edx
:005C7FF7 C1EF03                  shr edi, 03      // edi=edi >> 0x3
:005C7FFA 33D7                    xor edx, edi     // edx=edx xor edi
:005C7FFC 03CA                    add ecx, edx     // ecx=ecx+edx
:005C7FFE 8BD1                    mov edx, ecx     // edx=ecx
:005C8000 C1E207                  shl edx, 07      // edx=edx << 0x7
:005C8003 33CA                    xor ecx, edx     // ecx=ecx xor edx
:005C8005 03C1                    add eax, ecx     // eax=eax+ecx
:005C8007 8BD3                    mov edx, ebx     // edx=ebx
:005C8009 C1EA0F                  shr edx, 0F      // edx=edx >> 0xf
:005C800C 33C2                    xor eax, edx     // eax=eax xor edx
:005C800E 03D8                    add ebx, eax     // ebx=ebx+eax
:005C8010 8BC3                    mov eax, ebx     // eax=ebx
:005C8012 C1E00B                  shl eax, 0B      // eax=eax << 0xb
:005C8015 33D8                    xor ebx, eax     // ebx=ebx xor eax
:005C8017 8B442410                mov eax, dword ptr [esp+10]  // eax=序列号
:005C801B 33C3                    xor eax, ebx     // eax=eax xor ebx
:005C801D 8B54240C                mov edx, dword ptr [esp+0C]  // edx=0x00003c69
:005C8021 89542410                mov dword ptr [esp+10], edx  // [esp+10]=edx
:005C8025 8944240C                mov dword ptr [esp+0C], eax  // [esp+0c]=eax
:005C8029 83C60C                  add esi, 0000000C
:005C802C FF4C2414                dec [esp+14]     // [esp+14]-1
:005C8030 0F8561FFFFFF            jne 005C7F97     // 不等于 0 继续循环
:005C8036 8B442404                mov eax, dword ptr [esp+04]
:005C803A 8B542410                mov edx, dword ptr [esp+10]  // edx=激活码第一部分
:005C803E 8910                    mov dword ptr [eax], edx
:005C8040 8B442404                mov eax, dword ptr [esp+04]
:005C8044 8B54240C                mov edx, dword ptr [esp+0C]  // edx=激活码第二部分
:005C8048 895004                  mov dword ptr [eax+04], edx
:005C804B 83C418                  add esp, 00000018
:005C804E 5F                      pop edi
:005C804F 5E                      pop esi
:005C8050 5B                      pop ebx
:005C8051 C3                      ret
```

### 分析总结

运算前 `eax=0xd92da051` `ebx=0xb1a48361` `ecx=0x9f638ce6` `edx=0x00003c69`

`h1=0xbc614e` (序列号 16 进制)   `h2=0x00003c69`

`k1=edx+ebx`   `k2=ebx+k1`  `k3=edx >> 0x7`  `k4=k1 xor k3`

`m1=ecx+k4`    `m2=k4+m1`   `m3=m1 << 0x0d`  `m4=m1 xor m3`

`s1=eax+m4`    `s2=m4+s1`   `s3=s1 >> 0x11`  `s4=s1 xor s3`

`u1=k2+s4`     `u2=s4+u1`   `u3=u1 << 0x09`  `u4=u1 xor u3`

`f1=m2+u4`     `f2=u4+f1`   `f3=f1 >> 0x3`   `f4=f1 xor f3`

`g1=s2+f4`                     `g2=g1 << 0x7`  `g3=g1 xor g2`

`l1=u2+g3`                    `l2=f2 >> 0xf`  `l3=l1 xor l2`

`n1=f2+l3`                    `n2=n1 << 0xb`  `n3=n1 xor n2`

`eax=h1 xor n3`   `h1=h2`  `h2=eax`  `edx=h2`

以上是算法的第一步，将以上计算再循环 3 遍（哎，作者也不累啊，我都转晕了）

第 1 次循环前 `eax=0xb1a48361`  `ebx=0xd92da051`   `ecx=0x91057ebe`  `edx=h2(1)`

第 2 次循环前 `eax=0x91057ebe`  `ebx=0x9f638ce6`   `ecx=0xd92da051`  `edx=h2(2)`

第 3 次循环前 `eax=0x9f638ce6`  `ebx=0x91057ebe`   `ecx=0xb1a48361`  `edx=h2(3)`

上面这些参数除 `edx` 外都是固定的。

经过循环运算后得出：

`h1=20ed1b35` 依次从后两位取得出 `351bed20` = 注册码第一部分。

`h2=e35b8793` 依次从后两位取得出 `93875be3` = 注册码第二部分。

两部分合起来就是注册码了。

### 一组可用的注册信息

- 序列号：`12345678`
- 激活码：`351BED2093875BE3`

注册信息保存：`REGISTRATION.DAT` 文件中

### 注册机

- 中断地址：`643688`
- 中断次数：`2`
- 第一字节：`58`
- 指令长度：`1`
- 内存方式：`EAX`

【版权信息】`sunboy`  
2003-05-28

### 补充

CB 算法源程序，有点乱。

```c
void __fastcall Tform1::OKBtnClick(TObject *Sender)
{
    unsigned long k1,k2,k3,k4,m1,m2,m3,m4,s1,s2,s3,s4,u1,u2,u3,u4,f1,f2,f3,f4,
    g1,g2,g3,l1,l2,l3,n1,n2,n3;
    int h1,h2=0x3c69;
    unsigned long i=1,edx=0x3c69,kk,yy,t;
    unsigned long eax[5]={0,0xd92da051,0xb1a48361,0x91057ebe,0x9f638ce6};
    unsigned long ebx[5]={0,0xb1a48361,0xd92da051,0x9f638ce6,0x91057ebe};
    unsigned long ecx[5]={0,0x9f638ce6,0x91057ebe,0xd92da051,0xb1a48361};
    String sn1,sn="";
    h1=StrToInt(Edit1->Text);
    while(i<5)
    {
        {
            k1=edx+ebx[i];k2=ebx[i]+k1;k3=k1>>7;k4=k1^k3;
            m1=ecx[i]+k4;m2=k4+m1;m3=m1<<13;m4=m1^m3;
            s1=eax[i]+m4;s2=m4+s1;s3=s1>>17;s4=s1^s3;
            u1=k2+s4;u2=s4+u1;u3=u1<<9;u4=u1^u3;
            f1=m2+u4;f2=u4+f1;f3=f1>>3;f4=f1^f3;
            g1=s2+f4;g2=g1<<7;g3=g1^g2;
            l1=u2+g3;l2=f2>>15;l3=l1^l2;
            n1=f2+l3;n2=n1<<11;n3=n1^n2;
        }
        eax[i]=h1^n3;
        h1=h2;
        h2=eax[i];
        edx=h2;
        i++;
    }
    sn1=IntToHex(h2,2)+IntToHex(h1,2);
    kk=sn1.Length();
    yy=kk-1;
    t=8;
    while(t>0)
    {
        sn=sn+sn1[yy]+sn1[kk];
        kk=kk-2;
        yy=yy-2;
        t--;
    }
    Edit2->Text=sn;
}
```

`4dd466d0e1e0cc123afbe504da2462ca`

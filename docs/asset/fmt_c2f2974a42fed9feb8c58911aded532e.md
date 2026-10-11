
# 注册码判断过程

//下面是注册码的判断过程：

```
:00401060 83EC08                   sub esp, 00000008
:00401063 8B54240C                 mov edx, dword ptr [esp+0C]
:00401067 53                       push ebx
:00401068 56                       push esi
:00401069 57                       push edi
:0040106A 8A4204                   mov al, byte ptr [edx+04]       //edx 中是刚才处理过的假注册码
:0040106D 8BF9                     mov edi, ecx                    //同样两位两位处理
:0040106F 8A4A02                   mov cl, byte ptr [edx+02]
:00401072 8844240C                 mov byte ptr [esp+0C], al       //第五位放入第 0 位
:00401076 8A4207                   mov al, byte ptr [edx+07]
:00401079 884C240D                 mov byte ptr [esp+0D], cl       //第三位放入第 1 位
:0040107D 8A0A                     mov cl, byte ptr [edx]
:0040107F 8844240E                 mov byte ptr [esp+0E], al       //第八位放入第 2 位
:00401083 8A4206                   mov al, byte ptr [edx+06]
:00401086 884C240F                 mov byte ptr [esp+0F], cl       //第零位放入第 3 位
:0040108A 8A4A03                   mov cl, byte ptr [edx+03]
:0040108D 88442410                 mov byte ptr [esp+10], al       //第七位放入第 4 位
:00401091 8A4205                   mov al, byte ptr [edx+05]
:00401094 884C2411                 mov byte ptr [esp+11], cl       //第四位放入第 5 位
:00401098 8A4A01                   mov cl, byte ptr [edx+01]
:0040109B 88442412                 mov byte ptr [esp+12], al       //第六位放入第 6 位
:0040109F 8D74240C                 lea esi, dword ptr [esp+0C]     //传地址给 esi
:004010A3 33C0                     xor eax, eax
:004010A5 884C2413                 mov byte ptr [esp+13], cl       //第二位放入第 7 位
:004010A9 2BF2                     sub esi, edx                    //用 edx 作计数器
```

**Referenced by a (U)nconditional or (C)onditional Jump at Address:**

`| :004010BB(C)`

```
:004010AB 8A4C3814                 mov cl, byte ptr [eax+edi+14]   //开始将换过位的假注册码和 edi+14 中的数逐个异或
:004010AF 8A1C16                   mov bl, byte ptr [esi+edx]      //edi+14 中的数为 312ACF04C76BF4DA
:004010B2 32CB                     xor cl, bl
:004010B4 40                       inc eax
:004010B5 880A                     mov byte ptr [edx], cl           //异或出来的东东保存
:004010B7 42                       inc edx
:004010B8 83F808                   cmp eax, 00000008                //是否已结束
:004010BB 7CEE                     jl 004010AB                      //否则继续
:004010BD 5F                       pop edi
:004010BE 5E                       pop esi
:004010BF 5B                       pop ebx
:004010C0 83C408                   add esp, 00000008
:004010C3 C20400                   ret 0004
```

## 分析

其实已经很明显了，这是一个用注册码推 `ID` 的软件。它将我们输入的注册码转成 16 进制数的形式，然后进行换位，这一步得到的东东再和 16 进制数 `31 2A CF 04 C7 6B F4 DA` 逐位异或，然后这个东东和你的 `ID` 比较，相同说明是正确的，否则就让你检查。

我们可以用计算器来推一个你自己的注册码，例如:我的 `ID` 为 `bbbsl`,16 进制为 `62 62 62 73 6c`,假设注册码为数组 `a[8]`,则经它折腾之后实际上数组变为 `b[8]={a[5],a[3],a[8],a[0],a[7],a[4],a[6],a[2]}`,若要让推出来的 `ID` 前五位同 `bbbsl` 相同，就必须是数组 `b` 前五位和 `31 2a cf 04 c7` 相异或后得到的东东==`62 62 62 73 6c`,so 拿起计算器算算吧，`62^31=53`,`62^2a=48`,`62^cf=AD`,`73^04=77`,`6c^c7=AB`,即 `a[5]=53`,`a[3]=48`,`a[8]=AD`,`a[0]=77`,`a[7]=AB`,其余各位随意，所以我的注册码为：

```
id: bbbsl
sn: 770048005300ABAD 4dd466d0e1e0cc123afbe504da2462ca
```

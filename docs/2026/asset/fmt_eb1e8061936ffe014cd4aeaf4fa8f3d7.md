
# [RSA的另类爆破] RM to AVI MPEG WMV VCD SVCD DVD Converter 2.7

RM to AVI MPEG WMV VCD SVCD DVD Converter 2.7

说明：程序使用RSA加密的，`N`有82位十六进制，太难分解了，于是想到捏造一个`N`替换它，然后逆推注册码就完美的爆破它了。

终于在除夕夜完成爆破，以此文辞旧迎新！祝大家春节快乐，身体健康，合家平安，万事如意！

## 1. 重启验证

注册后生成一个文件：`rmtoall.ini`

```
[Options]
User=cyto
Pass=87654321012345678
```

## 2. 查壳脱壳

PEID:ASPack 2.12 -> Alexey Solodovnikov

OD载入：

```
00595001 r>  60             pushad                          ; 入口点
00595002     E8 03000000    call rmtoall.0059500A
...
005953AF     61             popad
005953B0     75 08          jnz short rmtoall.005953BA
005953B2     B8 01000000    mov eax,1
005953B7     C2 0C00        retn 0C
005953BA     68 3C4E5000    push rmtoall.00504E3C
005953BF     C3             retn
```

dump，Borland Delphi 6.0 - 7.0

运行ok。

## 3. 注册码存在`ini`文件里

下断`kernel.GetPrivateProfileStringA`

断下返回，上下翻翻，发现是读ini文件的内容，比如`user`、`pass`：

```
0050481F   mov ecx,up－rmto.005048E4                        ;  ASCII "User"
00504851   mov ecx,up－rmto.00504904                        ;  ASCII "Pass"
```

然后返回到：

```
00504FC3   |.  E8 E8F7FFFF     call up－rmto.005047B0               ;  取得ini内容
00504FC8   |.  A1 58815000     mov eax,dword ptr ds:[508158]
00504FCD   |.  8338 00         cmp dword ptr ds:[eax],0            ;  用户名是否为空？
00504FD0   |.  0F84 EE000000   je up－rmto.005050C4
00504FD6   |.  A1 18825000     mov eax,dword ptr ds:[508218]
00504FDB   |.  8338 00         cmp dword ptr ds:[eax],0            ;  注册码是否为空？
00504FDE   |.  0F84 E0000000   je up－rmto.005050C4
00504FE4   |.  B8 60A05000     mov eax,up－rmto.0050A060
00504FE9   |.  8B15 18825000   mov edx,dword ptr ds:[508218]       ;  up－rmto.00509FA8
00504FEF   |.  8B12            mov edx,dword ptr ds:[edx]
00504FF1   |.  E8 B2F4EFFF     call up－rmto.004044A8
00504FF6   |.  BA 44A05000     mov edx,up－rmto.0050A044
00504FFB   |.  A1 60A05000     mov eax,dword ptr ds:[50A060]
00505000   |.  E8 23CBFFFF     call up－rmto.00501B28               ;  输入的注册码处理
00505005   |.  B8 60A05000     mov eax,up－rmto.0050A060
0050500A   |.  E8 45F4EFFF     call up－rmto.00404454
0050500F   |.  8D55 E8         lea edx,dword ptr ss:[ebp-18]
00505012   |.  A1 AC815000     mov eax,dword ptr ds:[5081AC]
00505017   |.  8B00            mov eax,dword ptr ds:[eax]
00505019   |.  8B80 50040000   mov eax,dword ptr ds:[eax+450]
0050501F   |.  E8 8CF5F5FF     call up－rmto.004645B0              ; 取字符串A＝RSA（E）
00505024   |.  8B45 E8         mov eax,dword ptr ss:[ebp-18]       ; E=65537=10001(H)
00505027   |.  8D55 EC         lea edx,dword ptr ss:[ebp-14]
0050502A   |.  E8 B53AF0FF     call up－rmto.00408AE4
0050502F   |.  8B45 EC         mov eax,dword ptr ss:[ebp-14]
00505032   |.  BA 48A05000     mov edx,up－rmto.0050A048
00505037   |.  E8 E0CEFFFF     call up－rmto.00501F1C              ;  对E的处理
0050503C   |.  8D55 E0         lea edx,dword ptr ss:[ebp-20]
0050503F   |.  A1 AC815000     mov eax,dword ptr ds:[5081AC]
00505044   |.  8B00            mov eax,dword ptr ds:[eax]
00505046   |.  8B80 54040000   mov eax,dword ptr ds:[eax+454]
0050504C   |.  E8 5FF5F5FF     call up－rmto.004645B0              ;取字符串B＝RSA（N）
00505051   |.  8B45 E0         mov eax,dword ptr ss:[ebp-20]       ; N
00505054   |.  8D55 E4         lea edx,dword ptr ss:[ebp-1C]
00505057   |.  E8 883AF0FF     call up－rmto.00408AE4
0050505C   |.  8B45 E4         mov eax,dword ptr ss:[ebp-1C]
0050505F   |.  BA 50A05000     mov edx,up－rmto.0050A050
00505064   |.  E8 B3CEFFFF     call up－rmto.00501F1C              ;  对N的处理
00505069   |.  68 58A05000     push up－rmto.0050A058
0050506E   |.  68 58A05000     push up－rmto.0050A058
00505073   |.  68 58A05000     push up－rmto.0050A058
00505078   |.  68 58A05000     push up－rmto.0050A058
0050507D   |.  68 60A05000     push up－rmto.0050A060
00505082   |.  B9 50A05000     mov ecx,up－rmto.0050A050
00505087   |.  BA 48A05000     mov edx,up－rmto.0050A048
0050508C   |.  A1 44A05000     mov eax,dword ptr ds:[50A044]       ;  转换后的十六进制
00505091   |.  E8 EAF2FFFF     call up－rmto.00504380              ;  RSA加密
00505096   |.  A1 18825000     mov eax,dword ptr ds:[508218]
0050509B   |.  8B15 60A05000   mov edx,dword ptr ds:[50A060]
005050A1   |.  E8 02F4EFFF     call up－rmto.004044A8
005050A6   |.  B8 48A05000     mov eax,up－rmto.0050A048
005050AB   |.  E8 E4D0FFFF     call up－rmto.00502194
005050B0   |.  B8 50A05000     mov eax,up－rmto.0050A050
005050B5   |.  E8 DAD0FFFF     call up－rmto.00502194
005050BA   |.  B8 58A05000     mov eax,up－rmto.0050A058
005050BF   |.  E8 D0D0FFFF     call up－rmto.00502194
005050C4   |>  A1 58815000     mov eax,dword ptr ds:[508158]
005050C9   |.  8B00            mov eax,dword ptr ds:[eax]
005050CB   |.  8B15 18825000   mov edx,dword ptr ds:[508218]       ;  up－rmto.00509FA8
005050D1   |.  8B12            mov edx,dword ptr ds:[edx]
005050D3   |.  E8 88F7EFFF     call up－rmto.00404860              ;  比较
005050D8   |.  75 2D           jnz short up－rmto.00505107         ;  关键跳转
```

以上就是注册码计算判断的主线。经过计算后的注册码最后与用户名比较，相等就成功了。

## 4. 从关键比较入手，分析比较值与存放地址

```
005050C4   |>  A1 58815000     mov eax,dword ptr ds:[508158]
005050C9   |.  8B00            mov eax,dword ptr ds:[eax]
005050CB   |.  8B15 18825000   mov edx,dword ptr ds:[508218]       ;  up－rmto.00509FA8
005050D1   |.  8B12            mov edx,dword ptr ds:[edx]
005050D3   |.  E8 88F7EFFF     call up－rmto.00404860              ;  比较
005050D8   |.  75 2D           jnz short up－rmto.00505107         ;  关键跳转
```

比较的长度：

```
00404877   |.  8B46 FC    mov eax,dword ptr ds:[esi-4]    ;  用户名长度＝4
0040487A   |.  8B57 FC    mov edx,dword ptr ds:[edi-4]    ;  计算值长度＝26
```

比较的东西：

```
00508158  A4 9F 50 00              ?P.0
00509FA4  8C 3F EF 00              ??(帮
00EF3F8C  63 79 74 6F                cyto
00508218  A8 9F 50 00              ?P.膀
00509FA8  28 B0 EF 00              帮..
00EFB028  03 90 19 BD 6A D3 87 3F E4 B3 D9 C9 50 79 9F B8  ?疥??涑偕Py?
00EFB038  11 77 77 C9 E6 C7 8F 56 4B FB 7C 35 2D 62 D5 86  ww涉?VK?5-b?
00EFB048  6C 8A 10 D9 16 57                                  l??W..????
```

通过修改`ini`的注册码那如发现：只要注册码不变上面比较的地址都是固定的。如果注册码有变的话，存放地址也会跟着变，值也会变，比较的长度也变。

## 5. 定位加密算法

从上面的比较结果可以看出，太像RSA运算了，于是上cryptosearcher 0.97分析，发现`FGIntRSA`，查找google：

RSA加密一共需要三个单元`FGInt`, `FGIntPrimeGeneration`, `FGIntRSA`：

`FGint`是一个大型数运算库；`GIntPrimeGeneration`是一个找大质数的类库；`FGIntRSA`是实现加密和解密以及验证的单元。

而且主线对输入的注册码处理后紧接着出现两个字符串：

`65537＝10001（H）`

`63790510521550840388844862178357891512889443215420175471639593023445165917945890380913786330112957＝1DDD59DD5BE4EB02D560C324464193E0369A5ADAA8776B560E6ECA51F550EFD32B3FAE992B478963BD（H）`

估计第一个就是RSA的`E`，第二个应该就是传说中的`N`。

## 6. 算法过程分析

对注册码下硬件访问断点，分析程序对注册码的处理：

从`ini`中取得注册码，存放地址：

```
00EFB1CC  38 37 36 35 34 33 32 31 30 31 32 33 34 35 36 37  38  87654321012345678
```

`6.1 00505000 call up－rmto.00501B28`：对输入的注册码进行处理

首先根据注册码的ASCII码取出表中的值：

```
00501B99   |> /8D45 F8          /lea eax,dword ptr ss:[ebp-8]
00501B9C   |. |8B55 FC          |mov edx,dword ptr ss:[ebp-4]
00501B9F   |. |0FB65432 FF      |movzx edx,byte ptr ds:[edx+esi-1]     ;edx＝注册码ASCII码
00501BA4   |. |8B9495 F4FBFFFF  |mov edx,dword ptr ss:[ebp+edx*4-40C]  ;取值，ebp=0012FF4C
00501BAB   |. |E8 6C2BF0FF      |call up－rmto.0040471C
00501BB0   |. |46               |inc esi
00501BB1   |. |4B               |dec ebx
00501BB2   |.^\75 E5            \jnz short up－rmto.00501B99
```

在`00501BA4`处先根据顺取到的注册码ASCII码计算得到存放表单的地址。

对应关系：

```
0＝110100；1＝110101；2＝110110；3＝110111；4＝111000；5＝111001；
6＝111010；7＝111011；8＝111100；9＝111101。

a＝000000 ；b＝000010 ；c＝000100 ；d＝000110 ；e＝001000 ；f＝001010 ；
g＝001100 ；h＝001110 ；i＝010000 ；j＝010010 ；k＝010100 ；l＝010110 ；
m＝011000 ；n＝011010 ；o＝011100 ；p＝011110 ；q＝100000 ；r＝100010 ；
s＝100100 ；t＝100110 ；u＝101000 ；v＝101010 ；w＝101100 ；x＝101110 ；
y＝110000 ；z＝110010 。

A＝000001 ；B＝000011 ；C＝000101 ；D＝000111 ；E＝001001 ；F＝001011 ；
G＝001101 ；H＝001111 ；I＝010001 ；J＝010011 ；K＝010101 ；L＝010111 ；
M＝011001 ；N＝011011 ；O＝011101 ；P＝011111 ；Q＝100001 ；R＝100011 ；
S＝100101 ；T＝100111 ；U＝101001 ；V＝101011 ；W＝101101 ；X＝101111 ；
Y＝110001 ；Z＝110011
```

```
注册码：87654321012345678；38 37 36 35 34 33 32 31 30 31 32 33 34 35 36 37 38
```

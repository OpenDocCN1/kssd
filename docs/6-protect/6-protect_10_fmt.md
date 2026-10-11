
# 【看雪读书月】【学习心得】软件加密保护演示

> 原文：[`www.pediy.com/kssd/pediy10/68698.html`](https://www.pediy.com/kssd/pediy10/68698.html)

响应看雪读书活动

搞定不题目，只好另辟新路。。。，再者文章貌似心得文章较少。。。

此文章加密演示以`CrackMe`做为示例。

源代码在被破解后或是月底仍没人破解后放出密码。（暴破除外）

此`CM`应该比较有意思的~~

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `CrackMe.rar` |
| :--- | :--- | :--- |

看到`vxin`说这个`CM`应该比较有意思的，刚好有空，拿来`XX`一下

原程序请看：[`bbs.pediy.com/showthread.php?t=68698`](http://bbs.pediy.com/showthread.php?t=68698)

先`PEiD`一下，`DELPHI`的程序，再用`DEDE`看一下，居然没有`Anti DeDe`，于是祭出`OD`，在`OD`里对比着`DEDE`看源代码，怎一个爽字了得！

在`DEDE`里看程序的过程，除了`FormShow`外只有一个`Button1Click`：`0047F89C`，直奔`OD`里的`0047F89C`处下断，`F9`运行后输入用户名：`lelfei` 注册码：`14141414`，确定后中断下来了！

提示：注意对比`DEDE`看`CALL`过程的作用~~

代码：

```
0047F8CA  lea     edx, dword ptr [ebp-4]
0047F8CD  mov     eax, dword ptr [ebx+308]
0047F8D3  call    <GetText>                    ;  Edit_Name.GetText
0047F8D8  mov     eax, dword ptr [ebp-4]
0047F8DB  call    <StrLen>                     ;  用户名长度必须大于 2
0047F8E0  cmp     eax, 2
0047F8E3  jle     0047F9F8
0047F8E9  mov     eax, 0047FA7C                ;  ASCII "3w.bmp"
0047F8EE  call    <FileExists>                 ;  检测 KeyFile 是否存在
0047F8F3  test    al, al
0047F8F5  je      0047F9F8
0047F8FB  lea     edx, dword ptr [ebp-C]
0047F8FE  mov     eax, dword ptr [ebx+310]
0047F904  call    <GetText>                    ;  Edit_Code.GetText
0047F909  mov     eax, dword ptr [ebp-C]
0047F90C  lea     edx, dword ptr [ebp-8]
0047F90F  call    0047F520                     ;  -----------@1，Code 转换为 16 进制
0047F914  mov     eax, dword ptr [ebp-8]
0047F917  push    eax
0047F918  lea     edx, dword ptr [ebp-10]
0047F91B  mov     eax, dword ptr [ebx+308]
0047F921  call    <GetText>                    ;  Edit_Name.GetText
0047F926  mov     edx, dword ptr [ebp-10]
0047F929  pop     eax
0047F92A  call    <StrCmp>                     ;  比较 Name 与处理后的 Code
0047F92F  jnz     0047F9F8
0047F935  mov     eax, dword ptr [ebx+304]
0047F93B  mov     eax, dword ptr [eax+168]
0047F941  mov     edx, 0047FA7C                ;  ASCII "3w.bmp"
0047F946  call    <LoadFromFile>               ;  载入 KeyFile 文件
0047F94B  call    0047F34C                     ;  ------------@2，获取硬盘 SN
0047F950  lea     edx, dword ptr [ebp-18]
0047F953  call    <FixFileName>                ;  修复字符串
0047F958  mov     eax, dword ptr [ebp-18]
0047F95B  lea     edx, dword ptr [ebp-14]
0047F95E  call    <Trim>                       ;  去除空格
0047F963  mov     eax, dword ptr [ebp-14]
0047F966  call    0047F4BC                     ;  ------------@3，累加硬盘 SN 的 ASC 码
0047F96B  push    eax
0047F96C  mov     eax, dword ptr [ebx+304]
0047F972  mov     eax, dword ptr [eax+168]
0047F978  call    <GetBitmap>                  ;  Image2.GetBitmap
0047F97D  pop     edx
0047F97E  call    0047F684                     ;  -------------@4，用累加的 ASC 码对图片 XOR 解密
0047F983  mov     eax, dword ptr [ebx+304]
0047F989  call    004350D4                     ;  Image2.PaintRequest
0047F98E  mov     eax, dword ptr [ebx+304]
0047F994  mov     eax, dword ptr [eax+168]
0047F99A  call    <GetBitmap>                  ;  Image1.GetBitmap
0047F99F  push    eax
0047F9A0  mov     eax, dword ptr [ebx+2FC]
0047F9A6  mov     eax, dword ptr [eax+168]
0047F9AC  call    <GetBitmap>                  ;  Image1.GetBitmap
0047F9B1  pop     edx
0047F9B2  call    0047F758                     ;  -------------@5，比较图片内容
0047F9B7  test    al, al
0047F9B9  je      short 0047FA10
0047F9BB  push    40
0047F9BD  lea     edx, dword ptr [ebp-20]
0047F9C0  mov     eax, dword ptr [ebx+308]
0047F9C6  call    <GetText>                    ;  Edit_Name.GetText
0047F9CB  mov     ecx, dword ptr [ebp-20]
0047F9CE  lea     eax, dword ptr [ebp-1C]
0047F9D1  mov     edx, 0047FA8C                ;  '注册成功！正式授权给：'
0047F9D6  call    <StrCat>
0047F9DB  mov     eax, dword ptr [ebp-1C]
0047F9DE  call    <StrToChar>
0047F9E3  mov     edx, eax
0047F9E5  mov     ecx, 0047FAA4
0047F9EA  mov     eax, dword ptr [4812F4]
0047F9EF  mov     eax, dword ptr [eax]
0047F9F1  call    <MessageBox>
0047F9F6  jmp     short 0047FA10
0047F9F8  push    40
0047F9FA  mov     ecx, 0047FAAC
0047F9FF  mov     edx, 0047FAB4
0047FA04  mov     eax, dword ptr [4812F4]
0047FA09  mov     eax, dword ptr [eax]         ;  '非法用户！请与软件开发商联系。'
0047FA0B  call    <MessageBox>
```

可以看出这个过程大部分都是调用系统过程，只有标注的`@1`--`@5` 这 5 个`CALL`不知道用途。下面分别分析每个`CALL`的作用：

第一个`CALL 0047F520`代码就不贴了，主要作用是把用户输入的`Code`转换为16进制值，比如我输入的`Code='14141414'`，输出结果=`0x14141414`，然后与用户名的`ASC`码逐位比较。这就是说只需要把用户名的`ASC`码连接成字符串就是注册码了。

第二个`call 0047F34C`作用为读取硬盘序列号：

代码：



```
0047F34C  push    ebp
0047F34D  mov     ebp, esp
0047F34F  add     esp, -23C
0047F355  push    ebx
0047F356  mov     ebx, 0047F494
0047F35B  mov     eax, dword ptr [481468]
0047F360  cmp     dword ptr [eax], 2              ;  判断操作系统
0047F363  jnz     short 0047F386
0047F365  push    0                               ; /hTemplateFile = NULL
0047F367  push    0                               ; |Attributes = 0
0047F369  push    3                               ; |Mode = OPEN_EXISTING
0047F36B  push    0                               ; |pSecurity = NULL
0047F36D  push    3                               ; |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
0047F36F  push    C0000000                        ; |Access = GENERIC_READ|GENERIC_WRITE
0047F374  push    0047F498                        ; |FileName = "\\\\.\\PhysicalDrive0"
0047F379  call    <jmp.&kernel32.CreateFileA>     ; \\CreateFileA
0047F37E  mov     dword ptr [ebp-214], eax        ;  连接第 1 个物理硬盘
0047F384  jmp     short 0047F3A2
0047F386  push    0                               ; /hTemplateFile = NULL
0047F388  push    0                               ; |Attributes = 0
0047F38A  push    1                               ; |Mode = CREATE_NEW
0047F38C  push    0                               ; |pSecurity = NULL
0047F38E  push    0                               ; |ShareMode = 0
0047F390  push    0                               ; |Access = 0
0047F392  push    0047F4AC                        ; |FileName = "\\\\.\\SMARTVSD"
0047F397  call    <jmp.&kernel32.CreateFileA>     ; \\CreateFileA
0047F39C  mov     dword ptr [ebp-214], eax
0047F3A2  cmp     dword ptr [ebp-214], -1
0047F3A9  je      0047F48B
0047F3AF  xor     eax, eax
0047F3B1  push    ebp
0047F3B2  push    0047F463
0047F3B7  push    dword ptr fs:[eax]
0047F3BA  mov     dword ptr fs:[eax], esp
0047F3BD  lea     eax, dword ptr [ebp-239]
0047F3C3  xor     ecx, ecx
0047F3C5  mov     edx, 20                         ;  Clear InBuffer=20
0047F3CA  call    <FillChar>
0047F3CF  lea     eax, dword ptr [ebp-210]
0047F3D5  xor     ecx, ecx
0047F3D7  mov     edx, 210
0047F3DC  call    <FillChar>                      ;  Clear OutBuffer=210
0047F3E1  xor     eax, eax
0047F3E3  mov     dword ptr [ebp-218], eax
0047F3E9  mov     dword ptr [ebp-239], 200
0047F3F3  mov     byte ptr [ebp-234], 1
0047F3FA  mov     byte ptr [ebp-233], 1
0047F401  mov     byte ptr [ebp-230], 0A0
0047F408  mov     byte ptr [ebp-22F], 0EC
0047F40F  push    0                               ; /pOverlapped = NULL
0047F411  lea     eax, dword ptr [ebp-218]        ; |
0047F417  push    eax                              ; |pBytesReturned
0047F418  push    210                              ; |OutBufferSize = 210 (528.)
0047F41D  lea     eax, dword ptr [ebp-210]        ; |
0047F423  push    eax                              ; |OutBuffer
0047F424  push    20                               ; |InBufferSize = 20 (32.)
0047F426  lea     eax, dword ptr [ebp-239]        ; |
0047F42C  push    eax                              ; |InBuffer
0047F42D  push    7C088                            ; |IoControlCode = SMART_RCV_DRIVE_DATA
0047F432  mov     eax, dword ptr [ebp-214]        ; |
0047F438  push    eax                              ; |hDevice
0047F439  call    <jmp.&kernel32.DeviceIoContro>; \\DeviceIoControl
0047F43E  test    eax, eax                        ;  读取硬盘驱动数据
0047F440  jnz     short 0047F449
0047F442  call    0040399C
0047F447  jmp     short 0047F48B
0047F449  xor     eax, eax
0047F44B  pop     edx
0047F44C  pop     ecx
0047F44D  pop     ecx
0047F44E  mov     dword ptr fs:[eax], edx
0047F451  push    0047F46A
0047F456  mov     eax, dword ptr [ebp-214]
0047F45C  push    eax                              ; /hObject
0047F45D  call    <jmp.&kernel32.CloseHandle>     ; \\CloseHandle
```

获取硬盘数据后会先用`FixFileName`修复一下字符串，再`Trim`去除空格，然后进行第 3 个`CALL`进行处理。

第三个`call 0047F4BC`跟进后只是简单的对硬盘序列号累加，结果记为`sum`，代码也不贴了。

第四个`call 0047F684`作用是用`sum`对一幅图片进行解密：

代码:

```
0047F684  push    ebp
0047F685  mov     ebp, esp
0047F687  add     esp, -10
0047F68A  push    ebx
0047F68B  push    esi
0047F68C  push    edi
0047F68D  mov     ebx, edx
0047F68F  mov     dword ptr [ebp-4], eax
0047F692  xor     eax, eax
0047F694  push    ebp
0047F695  push    0047F6CE
0047F69A  push    dword ptr fs:[eax]
0047F69D  mov     dword ptr fs:[eax], esp
0047F6A0  mov     edx, 1
0047F6A5  mov     eax, dword ptr [ebp-4]
0047F6A8  call    <GetScanline>                   ;  获取图片第 1 线数据
0047F6AD  push    eax
0047F6AE  xor     edx, edx
0047F6B0  mov     eax, dword ptr [ebp-4]
0047F6B3  call    <GetScanline>                   ;  获取图片第 0 线数据
0047F6B8  pop     edx
0047F6B9  xchg    eax, edx
0047F6BA  sub     eax, edx
0047F6BC  cdq
0047F6BD  xor     eax, edx
0047F6BF  sub     eax, edx
0047F6C1  mov     dword ptr [ebp-8], eax          ;  相减得到每一线数据长度
0047F6C4  xor     eax, eax
0047F6C6  pop     edx
0047F6C7  pop     ecx
0047F6C8  pop     ecx
0047F6C9  mov     dword ptr fs:[eax], edx
0047F6CC  jmp     short 0047F6EE
……异常处理
0047F6EE  mov     eax, dword ptr [48140C]
0047F6F3  mov     dword ptr [eax], ebx            ;  保存累加结果 sum
0047F6F5  mov     eax, dword ptr [ebp-4]
0047F6F8  mov     edx, dword ptr [eax]
0047F6FA  call    dword ptr [edx+20]              ;  图片高度
0047F6FD  dec     eax
0047F6FE  test    eax, eax
0047F700  jl      short 0047F73E
0047F702  inc     eax
0047F703  mov     dword ptr [ebp-10], eax
0047F706  mov     dword ptr [ebp-C], 0            ;  i=0
0047F70D  mov     edx, dword ptr [ebp-C]          ;  ----循环开始
0047F710  mov     eax, dword ptr [ebp-4]
0047F713  call    <GetScanline>                   ;  获取图片每一线数据
0047F718  mov     edi, eax
0047F71A  mov     ebx, dword ptr [ebp-8]          ;  取每一线数据长度
0047F71D  dec     ebx
0047F71E  test    ebx, ebx
0047F720  jl      short 0047F736
0047F722  inc     ebx
0047F723  xor     esi, esi
0047F725  mov     eax, 100
0047F72A  call    00402B84                        ;  sum=sum*8088405+1,al=sum 高 8 位
0047F72F  xor     byte ptr [edi+esi], al          ;  对数据进行 XOR 解密
0047F732  inc     esi
0047F733  dec     ebx
0047F734  jnz     short 0047F725                  ;  对每一线的每一字节解密
0047F736  inc     dword ptr [ebp-C]               ;  i=i+1
0047F739  dec     dword ptr [ebp-10]              ;  取下一线
0047F73C  jnz     short 0047F70D                  ;  循环结束----
0047F73E  pop     edi
0047F73F  pop     esi
0047F740  pop     ebx
0047F741  mov     esp, ebp
0047F743  pop     ebp
0047F744  retn
```

可以看出对图片数据的每一字节都进行了`XOR`加密了。

最后一个`call 0047F758`就是比较图片内容了：

代码:



```
0047F758   push    ebp
0047F759   mov     ebp, esp
0047F75B   add     esp, -1C
0047F75E   push    ebx
0047F75F   push    esi
0047F760   push    edi
0047F761   mov     edi, edx
0047F763   mov     esi, eax
0047F765   mov     byte ptr [ebp-1], 0
0047F769   mov     eax, esi
0047F76B   mov     edx, dword ptr [eax]
0047F76D   call    dword ptr [edx+20]          ;  Image2.GetImageHeight
0047F770   mov     ebx, eax
0047F772   mov     eax, edi
0047F774   mov     edx, dword ptr [eax]
0047F776   call    dword ptr [edx+20]          ;  Image1.GetImageHeight=5B
0047F779   cmp     ebx, eax
0047F77B   jnz     0047F892                    ;  高度是否相同？
0047F781   mov     eax, esi
0047F783   mov     edx, dword ptr [eax]
0047F785   call    dword ptr [edx+2C]          ;  Image2.GetImageWidth
0047F788   mov     ebx, eax
0047F78A   mov     eax, edi
0047F78C   mov     edx, dword ptr [eax]
0047F78E   call    dword ptr [edx+2C]          ;  Image.GetImageWidth=157
0047F791   cmp     ebx, eax
0047F793   jnz     0047F892                    ;  宽度是否相同？
0047F799   mov     dl, 1
0047F79B   mov     eax, dword ptr [41C194]
0047F7A0   call    <CreateBitmap>
0047F7A5   mov     dword ptr [ebp-10], eax
0047F7A8   mov     dl, 1
0047F7AA   mov     eax, dword ptr [41C194]
0047F7AF   call    <CreateBitmap>
0047F7B4   mov     dword ptr [ebp-14], eax
0047F7B7   xor     ecx, ecx
0047F7B9   push    ebp
0047F7BA   push    0047F88B
0047F7BF   push    dword ptr fs:[ecx]
0047F7C2   mov     dword ptr fs:[ecx], esp
0047F7C5   mov     edx, esi
0047F7C7   mov     eax, dword ptr [ebp-10]
0047F7CA   mov     ecx, dword ptr [eax]
0047F7CC   call    dword ptr [ecx+8]           ;  Bitmap.Assign
0047F7CF   mov     edx, edi
0047F7D1   mov     eax, dword ptr [ebp-14]
0047F7D4   mov     ecx, dword ptr [eax]
0047F7D6   call    dword ptr [ecx+8]           ;  Bitmap.Assign
0047F7D9   mov     dl, 6
0047F7DB   mov     eax, dword ptr [ebp-10]
0047F7DE   call    <SetPixelFormat>
0047F7E3   mov     dl, 6
0047F7E5   mov     eax, dword ptr [ebp-14]
0047F7E8   call    <SetPixelFormat>
0047F7ED   mov     eax, dword ptr [ebp-10]
0047F7F0   mov     edx, dword ptr [eax]
0047F7F2   call    dword ptr [edx+20]          ;  GetImageHeight
0047F7F5   dec     eax
0047F7F6   test    eax, eax
0047F7F8   jl      short 0047F86D
0047F7FA   inc     eax
0047F7FB   mov     dword ptr [ebp-18], eax
0047F7FE   mov     dword ptr [ebp-8], 0
0047F805   mov     edx, dword ptr [ebp-8]      ;  循环开始====
0047F808   mov     eax, dword ptr [ebp-10]
0047F80B   call    <GetScanline>              ;  获取 Image1 的每一线
0047F810   mov     ebx, eax
0047F812   mov     edx, dword ptr [ebp-8]
0047F815   mov     eax, dword ptr [ebp-14]
0047F818   call    <GetScanline>              ;  获取 Image2 的每一线
0047F81D   mov     dword ptr [ebp-C], eax
0047F820   mov     eax, dword ptr [ebp-10]
0047F823   mov     edx, dword ptr [eax]
0047F825   call    dword ptr [edx+2C]          ;  GetImageWidth
0047F828   dec     eax
0047F829   test    eax, eax
0047F82B   jl      short 0047F865
0047F82D   inc     eax
0047F82E   mov     dword ptr [ebp-1C], eax
0047F831   xor     eax, eax
0047F833   lea     edx, dword ptr [eax+eax*2]  ;  循环开始----每次取 3 位
0047F836   mov     cl, byte ptr [ebx+edx]
0047F839   mov     esi, dword ptr [ebp-C]
0047F83C   cmp     cl, byte ptr [esi+edx]      ;  比较第 1 位
0047F83F   jnz     short 0047F865
0047F841   mov     cl, byte ptr [ebx+edx+1]
0047F845   mov     esi, dword ptr [ebp-C]
0047F848   cmp     cl, byte ptr [esi+edx+1]     ;  比较第 2 位
0047F84C   jnz     short 0047F865
0047F84E   mov     cl, byte ptr [ebx+edx+2]
0047F852   mov     esi, dword ptr [ebp-C]
0047F855   cmp     cl, byte ptr [esi+edx+2]     ;  比较第 3 位
0047F859   jnz     short 0047F865
0047F85B   mov     byte ptr [ebp-1], 1          ;  当 3 位都相同时置标志位
0047F85F   inc     eax
0047F860   dec     dword ptr [ebp-1C]
0047F863   jnz     short 0047F833             ;  继续循环----
0047F865   inc     dword ptr [ebp-8]
0047F868   dec     dword ptr [ebp-18]
0047F86B   jnz     short 0047F805             ;  继续循环====取下一位
0047F86D   xor     eax, eax
0047F86F   pop     edx
0047F870   pop     ecx
0047F871   pop     ecx
0047F872   mov     dword ptr fs:[eax], edx
0047F875   push    0047F892
0047F87A   mov     eax, dword ptr [ebp-10]
0047F87D   call    00403164
0047F882   mov     eax, dword ptr [ebp-14]
0047F885   call    00403164
0047F88A   retn
0047F88B   jmp     004038B8
0047F890   jmp     short 0047F87A
0047F892   mov     al, byte ptr [ebp-1]         ;  取标志位
0047F895   pop     edi
0047F896   pop     esi
0047F897   pop     ebx
0047F898   mov     esp, ebp
0047F89A   pop     ebp
0047F89B   retn
```

从这个 `CALL` 返回后就直接判断 `AL` 值，并判断注册是否成功的对话框。

由于是用 `XOR` 进行加密的，只需要把 `Image1` 的图片数据另存出来就可以作为 `KeyFile` 使用了。注意存出来后要加上一个 `Bitmap` 的头格式，先自己用画图程序建立一个 `343*91*24b` 大小的图片（如何知道长宽和色深？在比较图片数据时可以得到长与宽，色深就是在用工具查看程序资源 `Image1` 和 `Image2` 时得到的），把前 `0x36` 字节的头格式附到图片数据前就 `OK` 了。建议用 `010Editor` 进行查看，会自动把头格式用其他颜色标记出来。

如果要做注册机的话，可以自己获取到硬盘 `SN` 后累加 `ASC` 值，用累加结果 `sum*8088405+1` 取高位后，对得到的 `KeyFile` 进行 `XOR` 操作，只需要得到前 `3` 字节就 `OK` 了。在此略过注册机了，反正 `vxin` 要开放源代码了~~~~

PS：这个 `CM` 界面不错~~~~

lelfei

`6C656C666569`

强，学习，一会源码放在你的源子后面。。。

代码:



以下是一个 Delphi 单元的代码实现，展示了软件保护思路，通过 KeyFile 验证并利用图片加密技术：

```delphi
unit Unit1;
{=====================================
软件保护思路 ：
  KeyFile 验证而且是利用图片加密比较
                            by:VxinLv
          HomePage:www.3WorkRoom.cn
\=====================================}

interface

uses
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
  Dialogs,Registry, WinSkinData, StdCtrls,  ExtCtrls, Mask;

type
  TForm1 = class(TForm)
    Button1: TButton;
    Image1: TImage;
    SkinData1: TSkinData;
    Image2: TImage;
    Edit_Name: TEdit;
    Label1: TLabel;
    Edit_Code: TEdit;
    Label3: TLabel;
    procedure Button1Click(Sender: TObject);
    procedure FormShow(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

var
  Form1: TForm1;

implementation

{$R *.dfm}
function GetIdeSerialNumber: pchar;  //=============获取硬盘的出厂系列号
  const IDENTIFY_BUFFER_SIZE = 512;
type
   TIDERegs = packed record
     bFeaturesReg: BYTE;
     bSectorCountReg: BYTE;
     bSectorNumberReg: BYTE;
     bCylLowReg: BYTE;
     bCylHighReg: BYTE;
     bDriveHeadReg: BYTE;
     bCommandReg: BYTE;
     bReserved: BYTE;
  end;
  TSendCmdInParams = packed record
    cBufferSize: DWORD;
    irDriveRegs: TIDERegs;
    bDriveNumber: BYTE;
    bReserved: array[0..2] of Byte;
    dwReserved: array[0..3] of DWORD;
    bBuffer: array[0..0] of Byte;
  end;
  TIdSector = packed record
    wGenConfig: Word;
    wNumCyls: Word;
    wReserved: Word;
    wNumHeads: Word;
    wBytesPerTrack: Word;
    wBytesPerSector: Word;
    wSectorsPerTrack: Word;
    wVendorUnique: array[0..2] of Word;
    sSerialNumber: array[0..19] of CHAR;
    wBufferType: Word;
    wBufferSize: Word;
    wECCSize: Word;
    sFirmwareRev: array[0..7] of Char;
    sModelNumber: array[0..39] of Char;
    wMoreVendorUnique: Word;
    wDoubleWordIO: Word;
    wCapabilities: Word;
    wReserved1: Word;
    wPIOTiming: Word;
    wDMATiming: Word;
    wBS: Word;
    wNumCurrentCyls: Word;
    wNumCurrentHeads: Word;
    wNumCurrentSectorsPerTrack: Word;
    ulCurrentSectorCapacity: DWORD;
    wMultSectorStuff: Word;
    ulTotalAddressableSectors: DWORD;
    wSingleWordDMA: Word;
    wMultiWordDMA: Word;
    bReserved: array[0..127] of BYTE;
  end;
  PIdSector = ^TIdSector;
  TDriverStatus = packed record
    bDriverError: Byte;
    bIDEStatus: Byte;
    bReserved: array[0..1] of Byte;
    dwReserved: array[0..1] of DWORD;
  end;
  TSendCmdOutParams = packed record
    cBufferSize: DWORD;
    DriverStatus: TDriverStatus;
    bBuffer: array[0..0] of BYTE;
  end;
var
  hDevice: Thandle;
  cbBytesReturned: DWORD;
  SCIP: TSendCmdInParams;
  aIdOutCmd: array[0..(SizeOf(TSendCmdOutParams) + IDENTIFY_BUFFER_SIZE-1)-1] of Byte;
  IdOutCmd: TSendCmdOutParams absolute aIdOutCmd;
procedure ChangeByteOrder(var Data; Size: Integer);
var
  ptr: Pchar;
  i: Integer;
  c: Char;
begin
  ptr := @Data;
  for I := 0 to (Size shr 1) - 1 do begin
    c := ptr^;
    ptr^ := (ptr + 1)^;
    (ptr + 1)^ := c;
    Inc(ptr, 2);
  end;
end;
begin
Result := '';
if SysUtils.Win32Platform = VER_PLATFORM_WIN32_NT then begin // Windows NT, Windows 2000
hDevice := CreateFile('\\\\.\PhysicalDrive0', GENERIC_READ or GENERIC_WRITE,
FILE_SHARE_READ or FILE_SHARE_WRITE, nil, OPEN_EXISTING, 0, 0);
end else // Version Windows 95 OSR2, Windows 98
hDevice := CreateFile('\\\\.\SMARTVSD', 0, 0, nil, CREATE_NEW, 0, 0);
if hDevice = INVALID_HANDLE_VALUE then Exit;
try
FillChar(SCIP, SizeOf(TSendCmdInParams) - 1, #0);
FillChar(aIdOutCmd, SizeOf(aIdOutCmd), #0);
cbBytesReturned := 0;
with SCIP do begin
cBufferSize := IDENTIFY_BUFFER_SIZE;
with irDriveRegs do begin
bSectorCountReg := 1;
bSectorNumberReg := 1;
bDriveHeadReg := $A0;
bCommandReg := $EC;
end;
end;
if not DeviceIoControl(hDevice, $0007C088, @SCIP, SizeOf(TSendCmdInParams) - 1,
@aIdOutCmd, SizeOf(aIdOutCmd), cbBytesReturned, nil) then Exit;
finally
CloseHandle(hDevice);
end;
with PIdSector(@IdOutCmd.bBuffer)^ do begin
ChangeByteOrder(sSerialNumber, SizeOf(sSerialNumber));
(Pchar(@sSerialNumber) + SizeOf(sSerialNumber))^:= #0;
Result := Pchar(@sSerialNumber);
end;
end;

function ASCII10ADD(s: string): Integer;  //=============取 10 进制累加值
var i,sum:integer;
begin
   sum:=0;  for i:=1 to length(s) do
begin
   sum:=sum+ord(s[i]);
end;
   Result :=sum;
end;

function myHextoStr(S: string): string;    //=============16 进制字符串转原字符串
var hexS,tmpstr:string;
    i:integer;
    a:byte;
begin
    hexS  :=s;//应该是该字符串
    if length(hexS) mod 2=1 then
    begin
        hexS:=hexS+'0';
    end;
    tmpstr:='';
    for i:=1 to (length(hexS) div 2) do
    begin
        a:=strtoint('$'+hexS[2*i-1]+hexS[2*i]);
        tmpstr := tmpstr+chr(a);
    end;
    result :=tmpstr;
end;

procedure EncryptBMP(const BMP: TBitmap; Key: Integer);  //=============加密图片
var
  BytesPorScan: Integer;
  w, h: integer;
  p: pByteArray;
begin
  try
    BytesPorScan := Abs(Integer(BMP.ScanLine[1]) -
      Integer(BMP.ScanLine[0]));
  except
    raise Exception.Create('Error');
  end;
  RandSeed := Key;
  for h := 0 to BMP.Height - 1 do
  begin
    P := BMP.ScanLine[h];
    for w := 0 to BytesPorScan - 1 do
      P^[w] := P^[w] xor Random(256);
  end;
end;

function IsHomology(bmp1, bmp2: TbitMap): boolean;    //=============图片验证
type
  PRGBTripleArray = ^TRGBTripleArray;
  TRGBTripleArray = array[0..32767] of TRGBTriple;
var
  x, y: integer;
  p0, p1: PRGBTripleArray;
  sBmp, dBmp: TBitMap;
begin
  Result := False;

  if (bmp1.Height <> bmp2.Height) or
    (bmp1.Width <> bmp2.Width) then
  begin
    Exit;
  end;

  sBmp := TBitmap.Create;
  dBmp := TBitmap.Create;
  try
    sBmp.Assign(bmp1);
    dBmp.Assign(bmp2);
    sBmp.PixelFormat := pf24bit;
    dBmp.PixelFormat := pf24bit;
    for y := 0 to sBmp.Height - 1 do
    begin
      p0 := sBmp.ScanLine[y];
      p1 := dBmp.ScanLine[y];
      for x := 0 to sBmp.Width - 1 do
        if (p0[x].rgbtBlue = p1[x].rgbtBlue) and
          (p0[x].rgbtGreen = p1[x].rgbtGreen) and
          (p0[x].rgbtRed = p1[x].rgbtRed) then
        begin
          Result := True;
        end
        else
        begin
          Break;
        end;
    end;
  finally
    sBmp.Free;
    dBmp.Free;
  end;
end;

procedure TForm1.Button1Click(Sender: TObject);
begin
     try
      if (Length(Edit_Name.Text)>2) and (FileExists('3w.bmp')) //=========忽悠一下 Cracker
         and (myHextoStr(Edit_Code.Text)=Edit_Name.Text)  then //====重点就是验证图片
      begin
       Image2.Picture.LoadFromFile('3w.bmp'); //=========载入图片
       EncryptBMP(Image2.Picture.Bitmap, ASCII10ADD(Trim(strpas(GetIdeSerialNumber)))); //===恢复加密过的图片
       Image2.Refresh;
       if IsHomology(Image1.Picture.Bitmap,Image2.Picture.Bitmap) then  //============验证图片
         begin
           Application.MessageBox(PChar('注册成功！正式授权给：'+Edit_name.text),'提示',MB_OK+64);
         end;
         end
     else
      Application.MessageBox(PChar('非法用户！请与软件开发商联系。'),'错误',MB_OK+64);
       except
     end;
end;
        
procedure TForm1.FormShow(Sender: TObject);
begin
     Edit_Name.SetFocus;
end;

end.
```

上传的附件

|  || |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `CrackMe_src.rar` |

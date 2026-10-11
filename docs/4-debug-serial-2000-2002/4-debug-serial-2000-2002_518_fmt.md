
# UltraEdit-32 8.10.1.0

> 原文：[`www.pediy.com/kssd/pediy03/forum983.htm`](https://www.pediy.com/kssd/pediy03/forum983.htm)

UltraEdit-32 8.10.1.0 的破解及注册机的生成

1、这个软件的注册机网上早就有了，我写这篇教程主要是用来交流。希望起到抛砖引玉的功效^O^

2、下载：`http://www.ultraedit.com/`

3、破解必备工具：`TRW2000` 或者 `SoftICE`，`W32Dasm`，大脑:)

4、因为这个程序没有加壳，所以有 1M 多。未注册版好像有个时间限制，Hmmm 我对去除时间限制不感兴趣，我要的是注册机。这个程序注册后会在主程序目录下生成一个与主程序同名的×.reg 的文件。Ok，Let's go!

5、先在 `Help` 菜单下找到 `Register UltraEdit-32`，点击会弹出一个对话框。填入你的大名，还有注册码。我用，CoolBob，12345－54321-67890-09876。为什么知道注册码是这个格式呢，因为我跟踪过。你也可以在内存中看到这个格式。这里就不罗嗦了。填好后，点击 `OK` 按钮。呼啦一下，那个目录下多出来一个文件，这个文件名跟主程序的名字一样，扩展名为 `reg`。嘿嘿，这个时候可以猜想这个文件就是它的 keyfile，程序启动的时候肯定会去检验的，下什么断点呢？试试 `Bpx readfile do "d esp-100"`，用 `Symbol Loader` 装载 `UltraEdit-32`。按几次 `F5` 你会在 `SoftICE` 的数据窗口看到 `C:\Program Files\UltraEdit\UEDIT32.reg` 字样，这个时候就要小心跟踪了，`F12` 跳出来。

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00468949(C)

|

:00468925 8A8C3579FDFFFF                 mov cl, byte ptr [ebp+esi-00000287]
:0046892C 8A843578FDFFFF                 mov al, byte ptr [ebp+esi-00000288]
:00468933 FEC9                             dec cl
:00468935 FEC8                             dec al
:00468937 888C357CFEFFFF                 mov byte ptr [ebp+esi-00000184], cl
:0046893E 8884357DFEFFFF                 mov byte ptr [ebp+esi-00000183], al
:00468945 46                               inc esi
:00468946 46                               inc esi
:00468947 3BF7                             cmp esi, edi
:00468949 7EDA                             jle 00468925
:0046894B 8D857CFEFFFF                     lea eax, dword ptr [ebp+FFFFFE7C]
```

在这里把 `UEDIT32.reg` 里的数据解码，现在下 `d eax` 呵呵，看到什么了？

接下来就简单了，`bpr` 上面的那段数据。再 `F5` 让 `SoftICE` 尽情狂奔，一步一步跟到这里

```
:004136AE 0FBE740DAF                       movsx esi, byte ptr [ebp+ecx-51]
:004136B3 0FBE7C0DB1                       movsx edi, byte ptr [ebp+ecx-4F]
:004136B8 46                               inc esi
:004136B9 0FAFF7                            imul esi, edi
:004136BC 03F1                              add esi, ecx
:004136BE 0175FC                            add dword ptr [ebp-04], esi
:004136C1 41                               inc ecx
:004136C2 3BC8                             cmp ecx, eax
:004136C4 7CE8                             jl 004136AE
```

把你的名字经过一番运算，后保存在 `ebp-04` 这个地方，看看 `ebp-04` 上面来头，下 `Bpm ebp-04`。接下来的运算还真变态，看看吧

```
:004136C6 33FF                             xor edi, edi
:004136C8 8955F8                            mov dword ptr [ebp-08], edx
:004136CB 8D45B0                            lea eax, dword ptr [ebp-50]
:004136CE 33C9                             xor ecx, ecx
:004136D0 2945F8                            sub dword ptr [ebp-08], eax
:004136D3 897D10                            mov dword ptr [ebp+10], edi
:004136D6 891D8C3E5000                      mov dword ptr [00503E8C], ebx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004137C9(C)

|

:004136DC 8B45F8                            mov eax, dword ptr [ebp-08]
:004136DF 8D740DB0                          lea esi, dword ptr [ebp+ecx-50]
:004136E3 03C6                             add eax, esi
:004136E5 83F83C                            cmp eax, 0000003C
:004136E8 7D42                             jge 0041372C
:004136EA 8BC1                             mov eax, ecx

* Possible Reference to Dialog: DialogID_0604, CONTROL_ID:0004, "Replace All"
                                        |
* Possible Reference to String Resource ID=00004: "*.MAC"
                                        |
:004136EC 6A04                             push 00000004//注意这些数字
:004136EE 99                               cdq
:004136EF 5F                               pop edi
:004136F0 F7FF                              idiv edi
:004136F2 8BC1                             mov eax, ecx

* Possible Reference to Dialog: DialogID_0605, CONTROL_ID:0020, ""
                                        |
* Possible Reference to String Resource ID=00032: "
Any changes will be lost and the file deleted!"
                                        |
:004136F4 6A20                             push 00000020//注意这些数字
:004136F6 5B                               pop ebx

* Possible Reference to String Resource ID=00059: "Select File to Compare"
                                        |
:004136F7 6A3B                             push 0000003B//注意这些数字
:004136F9 8BFA                             mov edi, edx
:004136FB 99                               cdq
:004136FC F7FB                              idiv ebx
:004136FE 8BC2                             mov eax, edx
:00413700 99                               cdq
:00413701 2BC2                             sub eax, edx
:00413703 8B14BD00A04F00                   mov edx, dword ptr [4*edi+004FA000]//CRC
:0041370A D1F8                             sar eax, 1
:0041370C 5F                               pop edi
:0041370D 0FB60402                         movzx eax, byte ptr [edx+eax]//取一些数据来
:00413711 8AD1                             mov dl, cl
:00413713 0255FC                            add dl, byte ptr [ebp-04]
:00413716 0FB6D2                            movzx edx, dl
:00413719 33C2                             xor eax, edx
:0041371B 99                               cdq
:0041371C F7FF                              idiv edi
:0041371E 8B7D10                            mov edi, dword ptr [ebp+10]
:00413721 8A4415B0                          mov al, byte ptr [ebp+edx-50]
:00413725 02C1                             add al, cl
:00413727 324601                            xor al, byte ptr [esi+01]
:0041372A 8806                             mov byte ptr [esi], al//保存 al 的值，后面有用

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004136E8(C)

|
:0041372C 83FF3C                            cmp edi, 0000003C
:0041372F 7D71                             jge 004137A2
:00413731 8BC1                             mov eax, ecx

* Possible Reference to String Resource ID=00005: "ULTRAEDT.MAC"
                                        |
:00413733 6A05                             push 00000005//注意这些数字，无非就是整除、求余
:00413735 99                               cdq
:00413736 5B                               pop ebx
:00413737 F7FB                              idiv ebx
:00413739 8BDA                             mov ebx, edx
:0041373B 85DB                             test ebx, ebx//ebx=0,dl=esi%1a+0x41
:0041373D 895DF0                            mov dword ptr [ebp-10], ebx
:00413740 740A                             je 0041374C
:00413742 83FB02                            cmp ebx, 00000002//和 ebx＝0 一样的运算
:00413745 7405                             je 0041374C
:00413747 83FB04                            cmp ebx, 00000004//和 ebx＝0 一样的运算
:0041374A 751A                             jne 00413766

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00413740(C), :00413745(C)
|
:0041374C 0FB606                          movzx eax, byte ptr [esi]

* Possible Reference to String Resource ID=00026: "Run Windows Program"
                                        |
:0041374F 6A1A                             push 0000001A
:00413751 99                               cdq
:00413752 5B                               pop ebx
:00413753 F7FB                              idiv ebx
:00413755 8B5DF0                            mov ebx, dword ptr [ebp-10]
:00413758 80C241                            add dl, 41
:0041375B 88943D30FFFFFF                   mov byte ptr [ebp+edi-000000D0], dl//这里存放注册码
:00413762 47                               inc edi
:00413763 897D10                            mov dword ptr [ebp+10], edi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041374A(C)
|
:00413766 83FF3C                            cmp edi, 0000003C
:00413769 7D37                             jge 004137A2
:0041376B 83FB01                            cmp ebx, 00000001//ebx=1,3 时，dl＝esi%0xa+0x30
:0041376E 7405                             je 00413775
:00413770 83FB03                            cmp ebx, 00000003
:00413773 7517                             jne 0041378C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041376E(C)
|
:00413775 0FB606                          movzx eax, byte ptr [esi]

* Possible Reference to String Resource ID=00010: "
 Thank you for supporting Shareware."
                                        |
```



### 6、注册机

```c
#include <stdio.h>

main()
{ int i,n,nm=0;
  int j=0;
  int k=0;
  int ebp_04=0;int ebp_08=0;
  int esi;
  char code[23];
  char name[60];
  unsigned char a0[]={0x2f,0,0xde,0,0x43,0,0xa5,0,0xd8,0,0xef,0,0x8e,0,0x66,0};
  unsigned char a1[]={0x38,0,0x8f,0,0x65,0,0x5a,0,0xd3,0,0x1f,0,0x78,0,0x6f,0};
  unsigned char a2[]={0,0x1f,0,0xcb,0,0x98,0,0xf3,0,0x10,0,0xf1,0,0xa5,0,0x2f};
  unsigned char a3[]={0,0x73,0,0x34,0,0x77,0,0x33,0,0xfe,0,0xe0,0,0x73,0,0x78};

  printf("UltraEdit-32 8.10.1.0 keymaker by CoolBob[CCG],written at 2001-5-15\n");
  printf("\nyour name(at least 6 numbers and  not exceed 60):");
  scanf("%s",name);

  n=strlen(name);

  for (i=1,j=2;(i<n)&&(j<n);i++,j++){
    if((i%5==4)&&(i>0)) j++;
    ebp_08=ebp_08+name[i-1]*name[j];
  };
  ebp_08=ebp_08&0x00ff;

  for (i=0;i<n-1;i++) {
    ebp_04=ebp_04+(name[i]+1)*name[i+2]+i;
  };
  ebp_04=ebp_04&0x00ff;

  name[strlen(name)]=0;
  for (i=strlen(name)+1;i<0x3c;i++){name[i]=i;};

  for (i=0;i<20;i++){
    if(i%4==0){
      esi=(a0[i/2]^((i+ebp_04)&0x00ff))%0x3b;
      esi=name[esi]+i;
      esi=esi^name[i+1];
      name[i]=esi;
      if ((i%5==0)||(i%5==2)||(i%5==4)) esi=esi%0x1a+0x41;
      else  esi=esi%0xa+0x30;
    };

    if(i%4==1){
      esi=(a1[i/2]^((i+ebp_04)&0x00ff))%0x3b;
      esi=name[esi]+i;
      esi=esi^name[i+1];
      name[i]=esi;
      if ((i%5==0)||(i%5==2)||(i%5==4)) esi=esi%0x1a+0x41;
      else  esi=esi%0xa+0x30;
    };

    if(i%4==2){
      esi=(a2[i/2]^((i+ebp_04)&0x00ff))%0x3b;
      esi=name[esi]+i;
      esi=esi^name[i+1];
      name[i]=esi;
      if ((i%5==0)||(i%5==2)||(i%5==4)) esi=esi%0x1a+0x41;
      else  esi=esi%0xa+0x30;
    };

    if(i%4==3){
      esi=(a3[i/2]^((i+ebp_04)&0x00ff))%0x3b;
      esi=name[esi]+i;
      esi=esi^name[i+1];
      name[i]=esi;
      if ((i%5==0)||(i%5==2)||(i%5==4)) esi=esi%0x1a+0x41;
      else  esi=esi%0xa+0x30;
    };

    if((i%5==0)&&(i>0)) {code[k++]='-';} code[k++]=esi;
  };

  code[22]=ebp_08%0x19+0x41;
  code[7]=ebp_08%0x9+0x30;
  code[12]=ebp_08%0xd+0x41;
  code[23]=0;

  printf("code is: %s",code);
  printf("\nPress any key to exit!\n");
  getch();
}
```

### 核心逻辑说明

上面是一个嵌套循环，第一次找到注册码后，验证过程中发现 `UEDIT.reg` 被删除，说明程序中还有其他地方比较注册码，并调用 `Deletefilea` 等 API 来删掉文件。这是一个很好的突破口。

`Deletefilea` 调用处分析（SoftICE 调试发现）：

```asm
:00468B64 A18C3E5000            mov eax, dword ptr [00503E8C] // 这个 00503E8C 还记得吗？
:00468B69 FF35983D5000         push dword ptr [00503D98]
:00468B6F 8945F0               mov dword ptr [ebp-10], eax
:00468B72 E8F98E0200           call 00491A70
:00468B77 59                   pop ecx
:00468B78 83F80F               cmp eax, 0000000F
:00468B7B 59                   pop ecx
:00468B7C 0F8206010000         jb 00468C88
:00468B82 391D6C7E5000         cmp dword ptr [00507E6C], ebx
:00468B88 0F85FA000000         jne 00468C88
:00468B8E 391D707E5000         cmp dword ptr [00507E70], ebx
:00468B94 0F84EE000000         je 00468C88
:00468B9A 0FB645F0             movzx eax, byte ptr [ebp-10] // 取出的是上面第二次 name 运算值

* Possible Reference to String Resource ID=00025: "Dos Command"
:00468B9E 6A19                 push 00000019 // 这里就很呛眼
:00468BA0 8B3D983D5000         mov edi, dword ptr [00503D98]
:00468BA6 99                   cdq
:00468BA7 59                   pop ecx
:00468BA8 F7F9                 idiv ecx
:00468BAA 0FBE4716             movsx eax, byte ptr [edi+16] // 我们输入注册码的第 23 位
:00468BAE 83C241               add edx, 00000041
:00468BB1 3BC2                 cmp eax, edx
:00468BB3 7530                 jne 00468BE5
:00468BB5 0FB645F0             movzx eax, byte ptr [ebp-10]

* Possible Reference to String Resource ID=00009: "This copy of UltraEdit-32 is licensed to : "
:00468BB9 6A09                 push 00000009 // 还有
:00468BBB 99                   cdq
:00468BBC 59                   pop ecx
:00468BBD F7F9                 idiv ecx
:00468BBF 0FBE4707             movsx eax, byte ptr [edi+07] // 我们输入注册码的第 8 位
:00468BC3 83C230               add edx, 00000030
:00468BC6 3BC2                 cmp eax, edx
:00468BC8 751B                 jne 00468BE5
:00468BCA 0FB645F0             movzx eax, byte ptr [ebp-10]
:00468BCE 8A4F0C               mov cl, byte ptr [edi+0C] // 我们输入注册码的第 13 位

* Possible Reference to String Resource ID=00013: "Mod: "
:00468BD1 6A0D                 push 0000000D // 第三个
:00468BD3 99                   cdq
:00468BD4 5F                   pop edi
:00468BD5 F7FF                 idiv edi
:00468BD7 0FBEC1               movsx eax, cl
:00468BDA 83C241               add edx, 00000041
:00468BDD 3BC2                 cmp eax, edx
:00468BDF 0F84A3000000         je 00468C88
```

### 验证逻辑总结

核心验证逻辑是从 `00503E8C` 取出的数值（即之前保存的 `edx` 值）经过三次运算进行校验：

1. `edi%0x19+0x41` 与 `byte ptr [edi+16]` 比较
2. `eax%0x9+0x30` 与 `byte ptr [edi+07]` 比较
3. `eax%0xD+0x41` 与 `byte ptr [edi+0C]` 比较

如果有任何一项不相等，就会删除 `.reg` 文件。基于此逻辑，注册机生成代码如下所示。


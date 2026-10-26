
### 汇编代码

```
00452613     83C6 28                         add esi,28

00452616     AD                              lods dword ptr ds:[esi] //就是这个dword ptr ds:[esi],值00062934就是我的入口,读者可以用Peid看看就知道了，原程序这里是51000 ,这句执行后 EAX=00062934

00452617     8AD8                            mov bl,al

00452619     02DC                            add bl,ah

0045261B     C1E8 10                         shr eax,10

0045261E     02D8                            add bl,al

00452620     02DC                            add bl,ah  //后面某处和原程序入口51000对比，正确则解压004526E1处需要的正确代码，否则是错误的垃圾代码。

00452622     889D FC234000                   mov byte ptr ss:[ebp+4023FC],bl

00452628     59                              pop ecx

..................................................................................

004526CE     C1C0 09                         rol eax,9

004526D1     2BC2                            sub eax,edx

004526D3     81C2 820378F5                   add edx,F5780382

004526D9     89048E                          mov dword ptr ds:[esi+ecx*4],eax

004526DC     49                              dec ecx

004526DD   ^ EB E1                           jmp short fsfd.004526C0

004526DF     61                              popad

004526E0     61                              popad

004526E1     C3                              retn //返回14XXXX低内存段，此时是错误的垃圾代码。

0014334A     FD                              std //执行OVER

0014334B     33DA                            xor ebx,edx

0014334D     25 FD7E2DB5                     and eax,B52D7EFD

00143352     9F                              lahf

00143353     A0 00071C00                     mov al,byte ptr ds:[1C0700]

00143358     0000                            add byte ptr ds:[eax],al //空地址

0014335A     0000                            add byte ptr ds:[eax],al

0014335C     0000                            add byte ptr ds:[eax],al

..................................................................................
```

基本分析清楚，可以用Fly大侠的方法修改为原入口然后跳到脱壳程序入口。

也可以这样，麻烦但是可以锻炼技术。

#### 修改1

```
0045240C     60                              pushad
0045240D     78 01                           js short fsfd.00452410
0045240F     FC                              cld
00452410     72 03                           jb short fsfd.00452415
00452412     73 01                           jnb short fsfd.00452415
00452414     7A 66                           jpe short fsfd.0045247C
```

修改为

```
0045240C     60                              pushad
0045240D     E9 A5010000                     jmp fsfd.004525B7
00452412     73 01                           jnb short fsfd.00452415
00452414     7A 66                           jpe short fsfd.0045247C
```

下面的代码是通过跟踪原程序得到，可以通过二进制复制粘贴覆盖原代码，这个二哥的

`ASF-AVI-RM-WMV Repair V1.41` 脱壳去暗桩+汉化完美爆破

`ASProtect 1.1b Registered SDK` 之神奇挂挂3.7版脱壳+去暗桩

都详细写过，是`OD`的基本操作，必须掌握，不再赘述。

#### 修改2,动态代码

```
004525B7     E8 A4BE0000                     call fsfd.0045E460
004525BC     8B4424 20                       mov eax,dword ptr ss:[esp+20]
004525C0     33C9                            xor ecx,ecx
004525C2     8B9C8D E2264000                 mov ebx,dword ptr ss:[ebp+ecx*4+4026E2]
004525C9     039D AAD24000                   add ebx,dword ptr ss:[ebp+40D2AA]
004525CF     3BC3                            cmp eax,ebx
004525D1     74 07                           je short fsfd.004525DA
004525D3     90                              nop
004525D4     90                              nop
004525D5     90                              nop
004525D6     90                              nop
004525D7     41                              inc ecx
004525D8   ^ EB E8                           jmp short fsfd.004525C2
004525DA     C7848D E2264000 00000000        mov dword ptr ss:[ebp+ecx*4+4026E2],0
004525E5     8DB5 C2554000                   lea esi,dword ptr ss:[ebp+4055C2]
004525EB     B8 0A000000                     mov eax,0A
004525F0     F7E1                            mul ecx
004525F2     03F0                            add esi,eax
004525F4     56                              push esi
004525F5     51                              push ecx
004525F6     8A85 FC234000                   mov al,byte ptr ss:[ebp+4023FC]
004525FC     0AC0                            or al,al
004525FE     75 28                           jnz short fsfd.00452628
00452600     90                              nop
00452601     90                              nop
00452602     90                              nop
00452603     90                              nop
```

```
00452604     8B85 AAD24000                   mov eax,dword ptr ss:[ebp+40D2AA]
0045260A     8B70 3C                         mov esi,dword ptr ds:[eax+3C]
0045260D     03B5 AAD24000                   add esi,dword ptr ss:[ebp+40D2AA]
00452613     83C6 28                         add esi,28
00452616     AD                              lods dword ptr ds:[esi] //需要动外科手术,针对dword ptr ds:[esi]
```

修改为

```
00452604     B8 00100500                     mov eax,51000 //原入口
00452609     90                              nop
0045260A     90                              nop
0045260B     90                              nop
0045260C     90                              nop
0045260D     90                              nop
0045260E     90                              nop
0045260F     90                              nop
00452610     90                              nop
00452611     90                              nop
00452612     90                              nop
00452613     90                              nop
00452614     90                              nop
00452615     90                              nop
00452616     90                              nop
```

```
00452617     8AD8                            mov bl,al
00452619     02DC                            add bl,ah
0045261B     C1E8 10                         shr eax,10
0045261E     02D8                            add bl,al
00452620     02DC                            add bl,ah
00452622     889D FC234000                   mov byte ptr ss:[ebp+4023FC],bl
00452628     59                              pop ecx
00452629     5E                              pop esi
0045262A     60                              pushad
0045262B     B8 02000000                     mov eax,2
00452630     E8 B5BB0000                     call fsfd.0045E1EA
00452635     0BC0                            or eax,eax
00452637     75 24                           jnz short fsfd.0045265D
00452639     90                              nop
0045263A     90                              nop
0045263B     90                              nop
0045263C     90                              nop
0045263D     61                              popad
0045263E     8BBD AED24000                   mov edi,dword ptr ss:[ebp+40D2AE]
00452644     B8 0A000000                     mov eax,0A
00452649     F7E1                            mul ecx
0045264B     03F8                            add edi,eax
0045264D     B9 0A000000                     mov ecx,0A
00452652     8A9D FC234000                   mov bl,byte ptr ss:[ebp+4023FC]
00452658     EB 11                           jmp short fsfd.0045266B
0045265A     90                              nop
0045265B     90                              nop
0045265C     90                              nop
```

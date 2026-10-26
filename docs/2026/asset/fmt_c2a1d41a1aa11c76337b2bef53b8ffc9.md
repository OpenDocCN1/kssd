

``` 
004A0691    BF 01000000     mov edi,1
004A0696    8B45 F8         mov eax,dword ptr ss:[ebp-8]        ;// happy
004A0699    E8 DE40F6FF     call colorpic.0040477C              ; 取字符串长度的函数
004A069E    8B55 F8         mov edx,dword ptr ss:[ebp-8]
004A06A1    0FB6543A FF     movzx edx,byte ptr ds:[edx+edi-1]   ; 68(h)...
004A06A6    0FAF55 FC       imul edx,dword ptr ss:[ebp-4]       ; edx=68\*1E392E=C473AB0;...
004A06AA    6BD2 53         imul edx,edx,53                     ; edx\*53=(3)FB180710;...A1AD2F9A
004A06AD    03F2            add esi,edx                         ; esi=3388684+FB180710=FE508D94...
004A06AF    2BF0            sub esi,eax                         ; esi-Len(happy)=FE508D8F...最后:E976AB7F
004A06B1    47              inc edi
004A06B2    4B              dec ebx                             :\\
004A06B3  ^ 75 E1           jnz short colorpic.004A0696
004A06B5    FF75 F4         push dword ptr ss:[ebp-C]
004A06B8    68 80074A00     push colorpic.004A0780              ; /下面是取绝对值
004A06BD    8BC6            mov eax,esi                         ; /eax=E976AB7F;16895481=(not E976AB7F)+1
004A06BF    99              cdq                                 ; edx(的每一位)=eax的符号位扩展(cdq:Convert Doubleword to Quadword)
004A06C0    33C2            xor eax,edx
004A06C2    2BC2            sub eax,edx                         ; \serial2:16895481[十进制378098817]
004A06C4    8D55 F0         lea edx,dword ptr ss:[ebp-10]
004A06C7    E8 B082F6FF     call colorpic.0040897C              ; 注册码第二部分：378098817
004A06CC    FF75 F0         push dword ptr ss:[ebp-10]
004A06CF    8D45 F4         lea eax,dword ptr ss:[ebp-C]
004A06D2    BA 03000000     mov edx,3
004A06D7    E8 6041F6FF     call colorpic.0040483C
004A06DC    8B45 F8         mov eax,dword ptr ss:[ebp-8]        ; happy
004A06DF    E8 9840F6FF     call colorpic.0040477C              ; Len(happy)
004A06E4    8BD8            mov ebx,eax
004A06E6    85DB            test ebx,ebx
004A06E8    7E 2D           jle short colorpic.004A0717
004A06EA    BF 01000000     mov edi,1
004A06EF    8B45 F8         mov eax,dword ptr ss:[ebp-8]        ;//happy
004A06F2    E8 8540F6FF     call colorpic.0040477C              ; eax=Len(happy)
004A06F7    8B55 F8         mov edx,dword ptr ss:[ebp-8]        ; happy
004A06FA    0FB6543A FF     movzx edx,byte ptr ds:[edx+edi-1]   ; 68(h),61(a)...
004A06FF    F7EA            imul edx                            ; eax\*=68;\*61
004A0701    69C0 3B010000   imul eax,eax,13B                    ; eax\*=13B...
004A0707    03F0            add esi,eax                         ; E976AB7F+27FD8=E9792B57;...E983CAC6
004A0709    8B45 F8         mov eax,dword ptr ss:[ebp-8]
004A070C    E8 6B40F6FF     call colorpic.0040477C
004A0711    03F0            add esi,eax                          ;esi+Len(happy)
004A0713    47              inc edi
004A0714    4B              dec ebx
004A0715  ^ 75 D8           jnz short colorpic.004A06EF          ;\\
004A0717    0375 FC         add esi,dword ptr ss:[ebp-4]        ; esi=E983CAC6+1E392E=E9A203F4
004A071A    FF75 F4         push dword ptr ss:[ebp-C]           ; 54036100-378098817
004A071D    68 80074A00     push colorpic.004A0780
004A0722    8BC6            mov eax,esi                         ; /给E9A203F4取绝对值=serial3(十进制)
004A0724    99              cdq
004A0725    33C2            xor eax,edx
004A0727    2BC2            sub eax,edx                         ; \
004A0729    8D55 EC         lea edx,dword ptr ss:[ebp-14]
004A072C    E8 4B82F6FF     call colorpic.0040897C              ; 注册码第三部分：375258124
```

好了注册码的生成过程出来了，详见注册机源代码。

这里给出两组可用的`Name`和`RegKey`：

`Name:happy`
`RegKey:54036100-378098817-375258124`

注册文件：

``` 
[reginfo]
userinfo=qxUvU1s=
licence=9h1UF4GBUVD45Wr0nEZg4eyaHlhKxbMMinOBYw==
```

`Name:HappyTown`
`RegKey:168200108-243803457-239163428`

注册文件：

``` 
[reginfo]
userinfo=iwkpRXaw0kSm
licence=8gVR5zdgXmUFG7HSBo15KWhF3LLw6uV4bnu24jM=
```

---

## 加密`Name`和`RegKey`

下面我简单说一下这个过程：

///////////////////////////////////

`qxUvU1s=`的形成过程：

密钥在`00484600`处，长度`40h`位。

`cast256(name)` // 在`00483CC0`处

把结果和`Name(happy)`逐字符`xor`(在`00483CCB`处)生成`Name`长度个字节，然后把这几个字节通过移位等简单运算得到相应的数字序列(共`Name`长度个)，最终根据数字序列查表(从`004818C8`开始的子程序)得到`qxUvU1s=`

///////////////////////////////////

`9h1UF4GBUVD45Wr0nEZg4eyaHlhKxbMMinOBYw==`的形成过程：

跟上面的`qxUvU1s=`的形成过程一模一样(因为调用的是同一个子程序，只是`Name`换成了`RegKey`)，你可以跟进去看看，详细的跟进分析我就不贴了，仅贴出一部分，我实在太困了。

密钥：

``` 
00484600  AC 69 41 00 8C E5 41 00 C8 68 41 00 64 E6 41 00  ?A.?A.辱A.d媪.
00484610  30 E5 41 00 74 E6 41 00 6C E7 41 00 9C E9 41 00  0辶.t媪.l缌.?A.
00484620  84 E6 41 00 7C E6 41 00 A4 E9 41 00 80 41 48 00  ?A.|媪.らA.?H.
00484630  88 47 48 00 90 47 48 00 78 47 48 00 B4 47 48 00  ?H.?H.xGH.辞H.
```

表：

``` 
004A404C  41 42 43 44 45 46 47 48 49 4A 4B 4C 4D 4E 4F 50  ABCDEFGHIJKLMNOP
004A405C  51 52 53 54 55 56 57 58 59 5A 61 62 63 64 65 66  QRSTUVWXYZabcdef
004A406C  67 68 69 6A 6B 6C 6D 6E 6F 70 71 72 73 74 75 76  ghijklmnopqrstuv
004A407C  77 78 79 7A 30 31 32 33 34 35 36 37 38 39 2B 2F  wxyz0123456789+/
```

``` 
004A0382    8B4D EC         mov ecx,dword ptr ss:[ebp-14]
004A0385    B2 01           mov dl,1
004A0387    A1 C0904600     mov eax,dword ptr ds:[4690C0]
004A038C >  E8 DF8DFCFF     call colorpic.00469170              ; ->AxCtrls.TOleStream.Create(TOleStream;boolean;IStream);<+>
004A0391    8945 E8         mov dword ptr ss:[ebp-18],eax
004A0394    33C0            xor eax,eax
004A0396    55              push ebp
004A0397    68 97044A00     push <colorpic.->System.@HandleFina>
004A039C    64:FF30         push dword ptr fs:[eax]
004A039F    64:8920         mov dword ptr fs:[eax],esp
004A03A2    6A 00           push 0
004A03A4    BA 745A4A00     mov edx,colorpic.004A5A74
004A03A9    8B45 FC         mov eax,dword ptr ss:[ebp-4]
004A03AC >  8B80 30030000   mov eax,dword ptr ds:[eax+330]      ; \*cast:N.A.//指向密钥地址:00484600
004A03B2    B9 40000000     mov ecx,40
004A03B7    8B18            mov ebx,dword ptr ds:[eax]
004A03B9    FF53 40         call dword ptr ds:[ebx+40]          ; case256初始化
004A03BC    8B45 FC         mov eax,dword ptr ss:[ebp-4]
004A03BF >  8B80 30030000   mov eax,dword ptr ds:[eax+330]      ; \*cast:N.A.
004A03C5    8B10            mov edx,dword ptr ds:[eax]
004A03C7    FF52 48         call dword ptr ds:[edx+48]          ; 判断是否已经初始化
004A03CA    8D4D F0         lea ecx,dword ptr ss:[ebp-10]
004A03CD    8B45 FC         mov eax,dword ptr ss:[ebp-4]
004A03D0 >  8B80 30030000   mov eax,dword ptr ds:[eax+330]      ; \*cast:N.A.
004A03D6    8B55 F8         mov edx,dword ptr ss:[ebp-8]        ; happy
004A03D9    8B18            mov ebx,dword ptr ds:[eax]          ; 指向密钥:00484600
004A03DB    FF53 54         call dword ptr ds:[ebx+54]          ; 形成qxUvU1s=
004A03DE    8B45 F0         mov eax,dword ptr ss:[ebp-10]
004A03E1    50              push eax
004A03E2    B9 34054A00     mov ecx,colorpic.004A0534           ; ASCII "userinfo"
004A03E7    BA 48054A00     mov edx,colorpic.004A0548           ; ASCII "reginfo"
004A03EC    8B45 E8         mov eax,dword ptr ss:[ebp-18]
004A03EF    8B18            mov ebx,dword ptr ds:[eax]
004A03F1    FF53 04         call dword ptr ds:[ebx+4]
004A03F4    8B45 FC         mov eax,dword ptr ss:[ebp-4]
004A03F7 >  8B80 30030000   mov eax,dword ptr ds:[eax+330]      ; \*cast:N.A.
004A03FD    8B10            mov edx,dword ptr ds:[eax]
004A03FF    FF52 48         call dword ptr ds:[edx+48]
004A0402    8D4D F0         lea ecx,dword ptr ss:[ebp-10]
004A0405    8B45 FC         mov eax,dword ptr ss:[ebp-4]
004A0408 >  8B80 30030000   mov eax,dword ptr ds:[eax+330]      ; \*cast:N.A.
```


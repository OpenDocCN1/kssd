

```
.text:0048735B                 push    offset off_523498  ; "%1X"
.text:00487360                 push    edx
.text:00487361                 call    _sprintf              //转换成字符串
.text:00487366                 mov     eax, [edi+0DCh]
.text:0048736C                 add     esp, 0Ch
.text:0048736F                 mov     eax, [eax]            //取注册码
.text:00487371                 cmp     dword ptr [eax-8], 5
.text:00487375                 jl      short loc_487382      //长度小于5则跳
.text:00487377                 mov     al, [eax+4]           //取注册码第5个字节
.text:0048737A                 mov     cl, byte ptr [esp+28h+var_18] //取字符串第1个字节
.text:0048737E                 cmp     al, cl                //比较
.text:00487380                 jz      short loc_48738E
.text:00487382
.text:00487382 loc_487382:
.text:00487382                 mov     ecx, [edi+0E4h]
.text:00487388                 mov     dword ptr [ecx], 1    //不等则置标志
.text:0048738E
.text:0048738E loc_48738E:
```

这一段逆向如下：

```
int fun5(char *name,char *code)
{
  int sum,i;
  char buf[12];
  if(strlen(name) <= 0)return 0;
  sum = 1;
  for(i = 0; i < strlen(name); i++)sum += name[i];
  sum = ((sum * 9 + 10) / 3 + 36) % 16;
  sprintf(buf,"%1X",sum);
  if(strlen(code) < 5)return 0;
  if(code[4] != buf[0])return 0;
  else return 1;
}
```

### 第6处：

```
.text:004876F0 fun_4876F0      proc near
.text:004876F0
.text:004876F0 var_C           = dword ptr -0Ch
.text:004876F0 arg_0_name      = dword ptr  4
.text:004876F0 arg_4_code      = dword ptr  8
.text:004876F0
.text:004876F0                 sub     esp, 0Ch
.text:004876F3                 or      ecx, 0FFFFFFFFh
.text:004876F6                 xor     eax, eax
.text:004876F8                 push    esi
.text:004876F9                 mov     esi, [esp+10h+arg_0_name]
.text:004876FD                 push    edi
.text:004876FE                 mov     edi, esi
.text:00487700                 repne scasb
.text:00487702                 not     ecx
.text:00487704                 dec     ecx
.text:00487705                 mov     edi, ecx
.text:00487707                 jnz     short loc_48770F ; 用户名长度不为0则跳
.text:00487709                 pop     edi
.text:0048770A                 pop     esi
.text:0048770B                 add     esp, 0Ch
.text:0048770E                 retn
.text:0048770F loc_48770F:
.text:0048770F                 call    InitTable1_487650       //初始化CRC16表
.text:00487714                 push    edi             ; 长度
.text:00487715                 push    esi             ; 用户名
.text:00487716                 push    0
.text:00487718                 call    XorTable1_4876A0        //对用户名计算CRC16
.text:0048771D                 add     esp, 0Ch
.text:00487720                 and     eax, 0FFFFh
.text:00487725                 push    eax
.text:00487726                 lea     eax, [esp+18h+var_C]
.text:0048772A                 push    offset a02x_0   ; "%02X"
.text:0048772F                 push    eax
.text:00487730                 call    _sprintf               //CRC16结果转成字符串
.text:00487735                 mov     esi, [esp+20h+arg_4_code]
.text:00487739                 mov     al, byte ptr [esp+20h+var_C]
.text:0048773D                 add     esp, 0Ch
.text:00487740                 mov     cl, [esi+2]
.text:00487743                 lea     edx, [esi+2]
.text:00487746                 cmp     cl, al              //注册码第3个字节和字符串第1个字节比较
.text:00487748                 jz      short loc_487752
.text:0048774A                 pop     edi
.text:0048774B                 xor     eax, eax
.text:0048774D                 pop     esi
.text:0048774E                 add     esp, 0Ch
.text:00487751                 retn
.text:00487752 loc_487752:
.text:00487752                 mov     al, [esi+3]
.text:00487755                 mov     cl, byte ptr [esp+14h+var_C+1]
.text:00487759                 cmp     al, cl             //注册码第4个字节和字符串第2个字节比较
.text:0048775B                 jz      short loc_487765
.text:0048775D                 pop     edi
.text:0048775E                 xor     eax, eax
.text:00487760                 pop     esi
.text:00487761                 add     esp, 0Ch
.text:00487764                 retn
.text:00487765 loc_487765:
.text:00487765                 mov     edi, esi        ; 注册码
.text:00487767                 or      ecx, 0FFFFFFFFh
.text:0048776A                 xor     eax, eax
.text:0048776C                 repne scasb
.text:0048776E                 not     ecx
.text:00487770                 add     ecx, -3
.text:00487773                 push    ecx             ; len
.text:00487774                 push    edx             ; &RegCode[2]
.text:00487775                 push    eax             ; 0
.text:00487776                 call    XorTable1_4876A0  //对注册码第3个字节开始计算CRC16
.text:0048777B                 add     esp, 0Ch
.text:0048777E                 and     eax, 0FFFFh
.text:00487783                 lea     ecx, [esp+14h+var_C]
.text:00487787                 push    eax
.text:00487788                 push    offset a02x_0   ; "%02X"
.text:0048778D                 push    ecx
.text:0048778E                 call    _sprintf          //CRC16结果转成字符串
.text:00487793                 mov     dl, [esi]
.text:00487795                 mov     al, byte ptr [esp+20h+var_C]
.text:00487799                 add     esp, 0Ch
.text:0048779C                 cmp     dl, al            //注册码第1个字节和字符串第1个字节比较
.text:0048779E                 jz      short loc_4877A8
.text:004877A0                 pop     edi
.text:004877A1                 xor     eax, eax
.text:004877A3                 pop     esi
.text:004877A4                 add     esp, 0Ch
.text:004877A7                 retn
.text:004877A8 loc_4877A8:
.text:004877A8                 mov     cl, [esi+1]
.text:004877AB                 mov     dl, byte ptr [esp+14h+var_C+1]
.text:004877AF                 xor     eax, eax
.text:004877B1                 cmp     cl, dl            //注册码第2个字节和字符串第2个字节比较
.text:004877B3                 pop     edi
.text:004877B4                 pop     esi
.text:004877B5                 setz    al
.text:004877B8                 add     esp, 0Ch
.text:004877BB                 retn
.text:004877BB fun_4876F0      endp
```

这一段也是在注册对话框部分验证的。这一段逆向如下：

```
int fun6(char *name,char *code)
{
  int sum,i;
  char buf[12];
  short int tmp;
  tmp = crc16(name,strlen(name);
  tmp = tmp & 0xffff;
  sprintf(buf,"%02X",tmp);
  if(code[2] != buf[0])return 0;
  if(code[3] != buf[1])return 0;
  tmp = crc16(&code[2],strlen(code)-2);
  tmp = tmp & 0xffff;
  sprintf(buf,"%02X",tmp);
  if(code[0] != buf[0])return 0;
  if(code[1] != buf[1])return 0;
  return 1;
}
```

### 第7处：

```
.text:004A4220 sub_4A4220      proc near
.text:004A4220
.text:004A4220 var_1C          = dword ptr -1Ch
.text:004A4220 var_18          = dword ptr -18h
.text:004A4220 var_C           = dword ptr -0Ch
.text:004A4220 var_4           = dword ptr -4
.text:004A4220
.text:004A4220                 push    0FFFFFFFFh
.text:004A4222                 push    offset unknown_libname_1063
.text:004A4227                 mov     eax, large fs:0
.text:004A422D                 push    eax
.text:004A422E                 mov     large fs:0, esp
.text:004A4235                 sub     esp, 10h
.text:004A4238                 push    ebp
.text:004A4239                 push    esi
.text:004A423A                 mov     esi, ecx
.text:004A423C                 push    edi
.text:004A423D                 mov     [esp+28h+var_1C], esi
.text:004A4241                 mov     dword ptr [esi], offset off_503E48
.text:004A4247                 mov     eax, [esi+2D4h]
.text:004A424D                 mov     [esp+28h+var_4], 0Ch
.text:004A4255                 mov     cx, [eax]
.text:004A4258                 cmp     cx, 7D0h
.text:004A425D                 ja      short loc_4A4270
```


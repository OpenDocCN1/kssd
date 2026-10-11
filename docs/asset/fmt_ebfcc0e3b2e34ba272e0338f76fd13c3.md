
00406B29 `0F85 2D010000` `jnz Setup.00406C5C`

00406B2F `80F9 4E` `cmp cl,4E` ;N 则第 3 位取 6

00406B32 `75 09` `jnz short Setup.00406B3D`

00406B34 `80FA 36` `cmp dl,36`

00406B37 `0F85 1F010000` `jnz Setup.00406C5C`

00406B3D `33C0` `xor eax,eax`

00406B3F `8A46 04` `mov al,byte ptr ds:[esi+4]` ; 第 5 位

00406B42 `50` `push eax`

00406B43 `E8 B8FCFFFF` `call Setup.00406800` ;字符转换为数字

00406B48 `8BD0` `mov edx,eax`

00406B4A `33C0` `xor eax,eax`

00406B4C `8A46 03` `mov al,byte ptr ds:[esi+3]` ; 第 4 位

00406B4F `895424 1C` `mov dword ptr ss:[esp+1C],edx`

00406B53 `50` `push eax`

00406B54 `E8 A7FCFFFF` `call Setup.00406800` ;字符转换为数字

00406B59 `8BE8` `mov ebp,eax` ;

00406B5B `8D0492` `lea eax,dword ptr ds:[edx+edx*4]`

00406B5E `8D4445 00` `lea eax,dword ptr ss:[ebp+eax*2]`

00406B62 `99` `cdq`

00406B63 `B9 16000000` `mov ecx,16` ;mod $16

00406B68 `F7F9` `idiv ecx`

00406B6A `83C4 08` `add esp,8`

00406B6D `3A9A 58CC4300` `cmp bl,byte ptr ds:[edx+43CC58]` ;余数查表"ABCDEFGHKLMNPQRSTUVWXY"得到结果

00406B73 `0F85 E3000000` `jnz Setup.00406C5C`

00406B79 `33C0` `xor eax,eax`

00406B7B `8A46 07` `mov al,byte ptr ds:[esi+7]` ; 第 8 位

00406B7E `50` `push eax`

00406B7F `E8 7CFCFFFF` `call Setup.00406800` ;字符转换为数字

00406B84 `8BD0` `mov edx,eax`

00406B86 `33C0` `xor eax,eax`

00406B88 `8A46 06` `mov al,byte ptr ds:[esi+6]` ; 第 7 位

00406B8B `895424 18` `mov dword ptr ss:[esp+18],edx`

00406B8F `50` `push eax`

00406B90 `E8 6BFCFFFF` `call Setup.00406800` ;字符转换为数字

00406B95 `8BF8` `mov edi,eax`

00406B97 `8D1492` `lea edx,dword ptr ds:[edx+edx*4]`

00406B9A `8D0457` `lea eax,dword ptr ds:[edi+edx*2]`

00406B9D `99` `cdq`

00406B9E `B9 16000000` `mov ecx,16` ;mod $16

00406BA3 `F7F9` `idiv ecx`

00406BA5 `8A4424 1B` `mov al,byte ptr ss:[esp+1B]`

00406BA9 `83C4 08` `add esp,8`

00406BAC `3A82 58CC4300` `cmp al,byte ptr ds:[edx+43CC58]` ;余数查表"ABCDEFGHKLMNPQRSTUVWXY"得到结果

00406BB2 `0F85 A4000000` `jnz Setup.00406C5C`

00406BB8 `33C0` `xor eax,eax`

00406BBA `8A46 0A` `mov al,byte ptr ds:[esi+A]` ; 第 11 位

00406BBD `50` `push eax`

00406BBE `E8 3DFCFFFF` `call Setup.00406800` ;字符转换为数字

00406BC3 `8BD8` `mov ebx,eax`

00406BC5 `33C0` `xor eax,eax`

00406BC7 `8A46 09` `mov al,byte ptr ds:[esi+9]` ; 第 10 位

00406BCA `50` `push eax`

00406BCB `E8 30FCFFFF` `call Setup.00406800` ;字符转换为数字

00406BD0 `8BF0` `mov esi,eax`

00406BD2 `8D0C9B` `lea ecx,dword ptr ds:[ebx+ebx*4]`

00406BD5 `8D044E` `lea eax,dword ptr ds:[esi+ecx*2]`

00406BD8 `99` `cdq`

00406BD9 `B9 16000000` `mov ecx,16` ;mod $16

00406BDE `F7F9` `idiv ecx`

00406BE0 `8B4C24 38` `mov ecx,dword ptr ss:[esp+38]`

00406BE4 `8A41 0B` `mov al,byte ptr ds:[ecx+B]` ; 第 12 位

00406BE7 `83C4 08` `add esp,8`

00406BEA `3A82 58CC4300` `cmp al,byte ptr ds:[edx+43CC58]` ;余数查表"ABCDEFGHKLMNPQRSTUVWXY"得到结果

00406BF0 `75 6A` `jnz short Setup.00406C5C`

00406BF2 `33C0` `xor eax,eax`

00406BF4 `8A41 02` `mov al,byte ptr ds:[ecx+2]` ; 第 8 位

00406BF7 `50` `push eax`

00406BF8 `E8 03FCFFFF` `call Setup.00406800` ;字符转换为数字

00406BFD `0FAF7C24 18` `imul edi,dword ptr ss:[esp+18]` ; EDI=实际为第 7 位*第 8 位)

00406C02 `0FAF6C24 1C` `imul ebp,dword ptr ss:[esp+1C]` ; ebp=(实际为第 4 位*第 5 位)

00406C07 `0FAFF3` `imul esi,ebx` ; 第 10 位*第 11 位

00406C0A `03C6` `add eax,esi` ; +第 3 位

00406C0C `03C7` `add eax,edi`

00406C0E `03C5` `add eax,ebp`

00406C10 `99` `cdq`

00406C11 `B9 64000000` `mov ecx,64` ;mod $64

00406C16 `F7F9` `idiv ecx`

00406C18 `52` `push edx`

00406C19 `8D5424 24` `lea edx,dword ptr ss:[esp+24]`

00406C1D `68 A0E94200` `push Setup.0042E9A0`

00406C22 `52` `push edx`

00406C23 `E8 59CC0000` `call Setup.00413881` ;把最后结果进行处理为 10 进制即最后两位注册码

00406C28 `8B4424 40` `mov eax,dword ptr ss:[esp+40]`

00406C2C `8A48 0C` `mov cl,byte ptr ds:[eax+C]`

00406C2F `8A5424 2C` `mov dl,byte ptr ss:[esp+2C]`

00406C33 `83C4 10` `add esp,10`

00406C36 `3ACA` `cmp cl,dl`

00406C38 `75 22` `jnz short Setup.00406C5C`

00406C3A `8A50 0D` `mov dl,byte ptr ds:[eax+D]`

00406C3D `3A5424 1D` `cmp dl,byte ptr ss:[esp+1D]`

00406C41 `75 19` `jnz short Setup.00406C5C`

00406C43 `5F` `pop edi`

00406C44 `5E` `pop esi`

00406C45 `5D` `pop ebp`

00406C46 `B8 01000000` `mov eax,1`

00406C4B `5B` `pop ebx`

00406C4C `8B4C24 18` `mov ecx,dword ptr ss:[esp+18]`

00406C50 `334C24 1C` `xor ecx,dword ptr ss:[esp+1C]`

00406C54 `83C4 1C` `add esp,1C`

00406C57 `E9 8ACB0000` `jmp Setup.004137E6`

\====================================================================

第一位必须为 `Y`

第 2 位分别对应不同的字符`"BEFJLN"`

第 3 位分别对应不同版本下的取值`"1,2,3,4,5,6,7,8,9"`

第 4，5，7，8，10，11 位任意数字

第 6 位为`(第 5 位数字*5*2+第 4 位) mod &16` 结果查表`"ABCDEFGHKLMNPQRSTUVWXY"`

第 9 位为`(第 8 位数字*5*2+第 7 位) mod &16` 结果查表`"ABCDEFGHKLMNPQRSTUVWXY"`

第 12 位为`)第 10 为数字*5*2+第 9 位) mod &16` 结果查表`"ABCDEFGHKLMNPQRSTUVWXY"`

第 13，14 为`(第 3 位+第 4 位*第 5 位+第 7 位*第 8 位+第 10 位*第 11 位) MOD &64`，然后转换为 10 进制,如果小于 10 则前面补 0

\====================================================================

因为不会算法，东拼西凑的看了点 delphi 资料，凑出这个注册机来，谁笑话我我跟谁急！![](img/45777b5dbd6b901752ab7df42fce6892.jpg) ![](img/45777b5dbd6b901752ab7df42fce6892.jpg) ![](img/45777b5dbd6b901752ab7df42fce6892.jpg) 你们只可以当笑话或者笨蛋的典型来看待这个，OK？

\----------------------------------------------------------------

```pascal
;Delhi source

var
   s1,s2,s3,s4,s5,s6,s,sn:string;
 
 m:integer;

begin
 
 s1:='ABCDEFGHKLMNPQRSTUVWXYZ';
 
 s2:='BEFJLN';
 
 s3:='0123456789';
 
 s4:='5789';
 
 s5:='1789';
 
 s6:='3789';
 
 sn:='000000000000';
 
 if suiComboBox1.text='Standard Edition, Single User License' then
 
 begin
 
 sn[2]:=s2[1];
 
 sn[3]:=s5[random(4)+1];
 
 end;
 
 if suiComboBox1.text='Standard Edition, Upgrade License' then
 
 begin
 
 sn[2]:=s2[2];
 
 sn[3]:='2';
 
 end;
 
 if suiComboBox1.text='Professional Edition, Site License' then
 
 begin
 
 sn[2]:=s2[3];
 
 sn[3]:=s6[random(4)+1];
 
 end;
 
 if suiComboBox1.text='Professional Edition, Upgrade License' then
 
 begin
 
 sn[2]:=s2[4];
 
 sn[3]:='4';
 
 end;
 
 if suiComboBox1.text='Enterprise Edition, Site License' then
 
 begin
 
 sn[2]:=s2[5];
 
 sn[3]:=s4[random(4)+1];
 
 end;
```

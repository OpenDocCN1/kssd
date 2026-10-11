
# 音乐贺卡厂 4.10

> 原文：[`www.pediy.com/kssd/pediy03/forum1215.htm`](https://www.pediy.com/kssd/pediy03/forum1215.htm)

## 音乐贺卡厂 4.10 破解过程

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00433373(C)

:00433387 8B857CFFFFFF             mov eax, dword ptr [ebp+FFFFFF7C]<-你输入的假码,我们把它改成[ebp-50]（真正的注册码的内存地址），嘿嘿，让它变成真的和真的比，不就....
:0043338D 8B4DB0                   mov ecx, dword ptr [ebp-50]<-真正的注册码
:00433390 50                       push eax
:00433391 51                       push ecx

* Reference To: MSVBVM60.__vbaStrCmp, Ord:0000h
|
:00433392 FF1510114000             Call dword ptr [00401110]
:00433398 8BD8                     mov ebx, eax
:0043339A 8D8D7CFFFFFF             lea ecx, dword ptr [ebp+FFFFFF7C]
:004333A0 F7DB                     neg ebx
:004333A2 1BDB                     sbb ebx, ebx
:004333A4 F7DB                     neg ebx
:004333A6 F7DB                     neg ebx

* Reference To: MSVBVM60.__vbaFreeStr, Ord:0000h
|
:004333A8 FF159C124000             Call dword ptr [0040129C]
:004333AE 8D8D70FFFFFF             lea ecx, dword ptr [ebp+FFFFFF70]
```

```
* Possible StringData Ref from Code Obj ->"\\reguser.mcm"
|
:004334DE 6804664100               push 00416604<-进栈

* Reference To: MSVBVM60.__vbaStrCat, Ord:0000h
|
:004334E3 FF155C104000             Call dword ptr [0040105C]<-生成 reguser.mcm 文件
:004334E9 8D9560FFFFFF             lea edx, dword ptr [ebp+FFFFFF60]
:004334EF 8D4D80                   lea ecx, dword ptr [ebp-80]
:004334F2 898568FFFFFF             mov dword ptr [ebp+FFFFFF68], eax
:004334F8 C78560FFFFFF08000000     mov dword ptr [ebp+FFFFFF60], 00000008
:00433502 FFD7                     call edi
:00433504 6A00                     push 00000000
:00433506 8D9560FFFFFF             lea edx, dword ptr [ebp+FFFFFF60]
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004335AF(C)

:004335C3 83EC10                   sub esp, 00000010
:004335C6 8B857CFFFFFF             mov eax, dword ptr [ebp+FFFFFF7C]<-将你的注册名送入 eax,准备写如 reguser.mcm
:004335CC 8BD4                     mov edx, esp
:004335CE B908000000               mov ecx, 00000008
:004335D3 898D60FFFFFF             mov dword ptr [ebp+FFFFFF60], ecx
:004335D9 898568FFFFFF             mov dword ptr [ebp+FFFFFF68], eax
:004335DF 890A                     mov dword ptr [edx], ecx
:004335E1 8B8D64FFFFFF             mov ecx, dword ptr [ebp+FFFFFF64]
:004335E7 6A01                     push 00000001

* Possible StringData Ref from Code Obj ->"WWriteLine"
|
:004335E9 6840664100               push 00416640
:004335EE 894A04                   mov dword ptr [edx+04], ecx
:004335F1 8D4D90                   lea ecx, dword ptr [ebp-70]
:004335F4 51                       push ecx
:004335F5 C7857CFFFFFF00000000     mov dword ptr [ebp+FFFFFF7C], 00000000
:004335FF 894208                   mov dword ptr [edx+08], eax
:00433602 8B856CFFFFFF             mov eax, dword ptr [ebp+FFFFFF6C]
:00433608 89420C                   mov dword ptr [edx+0C], eax

* Reference To: MSVBVM60.__vbaObjVar, Ord:0000h
|
:0043360B FF151C114000             Call dword ptr [0040111C]
:00433611 50                       push eax

* Reference To: MSVBVM60.__vbaLateMemCall, Ord:0000h
|
:00433612 FF1514124000             Call dword ptr [00401214]
:00433618 83C41C                   add esp, 0000001C
:0043361B 8D8D70FFFFFF             lea ecx, dword ptr [ebp+FFFFFF70]

* Reference To: MSVBVM60.__vbaFreeObj, Ord:0000h
|
:00433621 FF1598124000             Call dword ptr [00401298]
:00433627 8D8D60FFFFFF             lea ecx, dword ptr [ebp+FFFFFF60]

* Reference To: MSVBVM60.__vbaFreeVar, Ord:0000h
|
:0043362D FF151C104000             Call dword ptr [0040101C]
:00433633 8B17                     mov edx, dword ptr [edi]
:00433635 57                       push edi
:00433636 FF9204030000             call dword ptr [edx+00000304]
:0043363C 50                       push eax
:0043363D 8D8570FFFFFF             lea eax, dword ptr [ebp+FFFFFF70]
:00433643 50                       push eax
:00433644 FFD3                     call ebx
:00433646 8BF0                     mov esi, eax
:00433648 8D957CFFFFFF             lea edx, dword ptr [ebp+FFFFFF7C]
:0043364E 52                       push edx
:0043364F 56                       push esi
:00433650 8B0E                     mov ecx, dword ptr [esi]
:00433652 FF91A0000000             call dword ptr [ecx+000000A0]
:00433658 85C0                     test eax, eax
:0043365A DBE2                     fclex
:0043365C 7D12                     jge 00433670
:0043365E 68A0000000               push 000000A0

* Possible StringData Ref from Code Obj ->"酦?檉??"
|
:00433663 68DC644100               push 004164DC
:00433668 56                       push esi
:00433669 50                       push eax

* Reference To: MSVBVM60.__vbaHresultCheckObj, Ord:0000h
|
:0043366A FF156C104000             Call dword ptr [0040106C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043365C(C)

:00433670 83EC10                   sub esp, 00000010
:00433673 8B857CFFFFFF             mov eax, dword ptr [ebp+FFFFFF7C]<-将你的注册码（你输入的假码）送入 eax,准备写如 reguser.mcm。我们把它改成[ebp-50]
```

附上 RPP 文件：

```
O=cr-ecardiy.exe:
F=ecardiy.exe:
p=433387/8b,85,7c,ff,ff,ff/8b,45,b0,90,90,90:
p=433673/8b,85,7c,ff,ff,ff/8b,45,b0,90,90,90:

$
jieao[CCG] 2001.8.11
```

### 标题:<音乐贺卡工场 4.10>注册。 (71 字)

发信人:`CrackerABC[BCG]`

时 间:`2001-8-10 21:15:06`

详细信息:

```
bpx 433390 do "d ecx"
```

这可是为了给外面的老婆做贺卡的时候跟的，很简单。

### 代码实现

```
//code with delphi5
var
  name,machine,code,a,d,e:string;
  len,halflen,i,b,f:integer;
//reg generate
begin
  name:=editbox1.text;
  len:=length(name);
  machine:=editbox2.text+editbox2.text;
  i:=1;
  code:='ECD';
  if len<4 then editbox3.text:='用户名至少为 4 位，且不能为中文。'
  else
    begin
      if len/2<>round(len/2) then
        begin
          while i<=len do
            begin
              a:=int2str(ord(name[i]) xor ord(machine[i+2]));
              code:=code+a;
              i:=i+1;
            end;
          if length(code)>=12 then editbox3.text:=code[1]+code[2]+code[3]+code[4]+code[5]+code[6]+code[7]+code[8]+code[9]+code[10]+code[11]+code[12]
            else
              begin
                e:=''; i:=1;
                while i<=(length(code)-3) do
                begin
                  d:=code[i+3];
                  e:=e+d;
                  i:=i+1;
                end;
                e:=e+e;
                e:=e+e;
                code:='ECD'+e;
```



```
editbox3.text:=code[1]+code[2]+code[3]+code[4]+code[5]+code[6]+code[7]+code[8]+code[9]+code[10]+code[11]+code[12];

end;

end;

if len/2=round(len/2) then

begin

.......//以下省略，抱歉，答应作者的，不能无信

end;

end;

4dd466d0e1e0cc123afbe504da2462ca
```

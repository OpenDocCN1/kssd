
# PCWorker 破解(算法)分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-153.htm`](https://www.pediy.com/kssd/pediy07/pediy7-153.htm)

## 基本信息

**破文标题**：PCWorker 破解(算法)分析  
**破文作者**：Skyer  
**软件名称**：`PCWorker`  
**软件大小**：325 KB  
**使用平台**：Win9x/Me/NT/2000/XP  
**发布公司**：http://www.pcworker.net  
**软件简介**：像按键精灵可自动按键 (可免费注册)  
**加密方式**：License File  
**编译语言**：VC 6  
**功能限制**：时间限制  
**调试环境**：WinXP, `OllyDBG 1.10`  
**破解日期**：2005-04-02  
**破解目的**：研究算法分析 (此软件注册不用钱, 作者只是想了解有那些人在使用这软件)  
**作者声明**：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

---

## 破解过程

第一次发文，请多指教..

1. 先执行一次 `pcworker`，记下 `PCKEY`，并去除 `-`，关闭
2. `OD` 载入 `pcworker.exe` 执行到

`PCWorker.ii` 是注册档，向 `PCWorker` 注册后，作者会发 email 到你的信箱

```assembly
00432BC2   > 68 24E74400    push PCWorker.0044E724             ;  ASCII "\PCWorker.ii"
00432BC7   . 8D4424 1C      lea eax,dword ptr ss:[esp+1C]
00432BCB   . 68 4C144500    push offset <PCWorker.EXEPath>
00432BD0   . 50             push eax
00432BD1   . E8 3C570000    call <jmp.&MFC42.#924_operator+>
00432BD6   . 8B4C24 18      mov ecx,dword ptr ss:[esp+18]
00432BDA   . C68424 C412000>mov byte ptr ss:[esp+12C4],8
00432BE2   . 51             push ecx
00432BE3   . FF15 80F84300  call dword ptr ds:[<&PCWorker.KMCF_Acc>;  PCWork_1.KMCF_AccessFile
00432BE9   . 8B2D 54F84300  mov ebp,dword ptr ds:[<&PCWorker.CKMC_>;  PCWork_1.CKMC_IniFile::GetValue
00432BEF   . 83C4 04        add esp,4
00432BF2   . 84C0           test al,al
00432BF4   . 0F84 BC030000  je PCWorker.00432FB6
```

继续往下执行到

```assembly
0043363B   . 68 1CE74400    push PCWorker.0044E71C             ;  ASCII "PCKEY"
00433640   . E8 794C0000    call <jmp.&MFC42.#537_CString::CString>
00433645   . 8B15 D0E54400  mov edx,dword ptr ds:[44E5D0]     ;  PCWorker.0044E5D4
0043364B   . 51             push ecx
0043364C   . 8BCC           mov ecx,esp
0043364E   . 896424 40      mov dword ptr ss:[esp+40],esp
00433652   . 52             push edx
00433653   . C68424 D412000>mov byte ptr ss:[esp+12D4],34
0043365B   . E8 5E4C0000    call <jmp.&MFC42.#537_CString::CString>
00433660   . 8D4424 1C      lea eax,dword ptr ss:[esp+1C]
00433664   . 8BCD           mov ecx,ebp
00433666   . 50             push eax
00433667   . C68424 D412000>mov byte ptr ss:[esp+12D4],7
0043366F   . FF15 54F84300  call dword ptr ds:[<&PCWorker.CKMC_Ini>;  PCWork_1.CKMC_IniFile::GetValue
```

这里会从 `pcworker.ini` 读取 `[Registration]` 下的注册信息，但我们的设定文件无此信息, 手动加上

```ini
[Registration]
PCKEY=填入第一次执行 pcworker 记录的 pckey
Name=Skyer
Birthday=1981/01/30
Serial=kKR7sky $
```

本文主要要找出 `Serial` 的算法

重新分析 `pcworker.exe` (`Ctrl+F2`)，执行到

```assembly
004337E1   . 8B0F           mov ecx,dword ptr ds:[edi]        ;  Serial
004337E3   . BB 0A000000    mov ebx,0A                        ;  长度一定要为 10
004337E8   . 3959 F8        cmp dword ptr ds:[ecx-8],ebx
004337EB   . 7D 1A          jge short PCWorker.00433807
```

了解 `Serial` 长度必为 10

再往下到

```assembly
004338E8   > /8A0408        mov al,byte ptr ds:[eax+ecx]
004338EB   . |8B1D 24144500  mov ebx,dword ptr ds:[<PCKeyXOR>]
004338F1   . |884424 10     mov byte ptr ss:[esp+10],al
004338F5   . |8B4424 10     mov eax,dword ptr ss:[esp+10]
004338F9   . |25 FF000000   and eax,0FF
004338FE   . |33D8          xor ebx,eax
00433900   . |41           inc ecx
00433901   . |891D 24144500  mov dword ptr ds:[<PCKeyXOR>],ebx
00433907   . |8B02         mov eax,dword ptr ds:[edx]
00433909   . |3B48 F8      cmp ecx,dword ptr ds:[eax-8]
0043390C   . ^\7C DA       jl short PCWorker.004338E8
```

这里是算 `pckey` 的 `xor` 编码 (由左至右一个一个 `xor`) => 取名 `PCKeyXOR`

转成代码大概是：

```c
int sum=0;
for (i=0; i<PCKeyLength; i++)
  sum = sum ^ PCKey[ i ]
```

```assembly
0043392C   > /8A1408        mov dl,byte ptr ds:[eax+ecx]
0043392F   . |8B2D 18144500  mov ebp,dword ptr ds:[<PCKeyAdd>]
00433935   . |885424 10     mov byte ptr ss:[esp+10],dl
00433939   . |8B5424 10     mov edx,dword ptr ss:[esp+10]
0043393D   . |81E2 FF000000  and edx,0FF
00433943   . |03EA         add ebp,edx
00433945   . |40          inc eax
00433946   . |892D 18144500  mov dword ptr ds:[<PCKeyAdd>],ebp
0043394C   . |8B51 F8      mov edx,dword ptr ds:[ecx-8]
0043394F   . |3BC2         cmp eax,edx
00433951   . ^\7C D9      jl short PCWorker.0043392C
```

`pckey` 的 `add` 编码 (由左至右一个一个 `add`) => 取名 `PCKeyAdd`

转成代码大概是：

```c
int sum=0;
for (i=0; i<PCKeyLength; i++)
  sum = sum + PCKey[ i ]
```

```assembly
00433955   . 8B96 11010000  mov edx,dword ptr ds:[esi+111]    ;  Serial
0043395B   . 8BE8           mov ebp,eax
0043395D   . 33C0           xor eax,eax
0043395F   . B9 01000000    mov ecx,1
00433964   > 0FBE1C0A       movsx ebx,byte ptr ds:[edx+ecx]
00433968   . 03C3           add eax,ebx
0043396A   . 41             inc ecx
0043396B   . 83F9 08        cmp ecx,8
0043396E   . ^ 7E F4        jle short PCWorker.00433964
00433970   . 8B8E 19010000  mov ecx,dword ptr ds:[esi+119]
00433976   . BB 5F000000    mov ebx,5F
0043397B   . 41             inc ecx
0043397C   . 898E 19010000  mov dword ptr ds:[esi+119],ecx
00433982   . 8A0A           mov cl,byte ptr ds:[edx]
00433984   . 33D2           xor edx,edx
00433986   . F7F3           div ebx
00433988   . 0FBEC9         movsx ecx,cl
0043398B   . 0FBEC2         movsx eax,dl                    ;  编码 / 5F 的余数
0043398E   . 83C0 20        add eax,20
00433991   . 3BC8           cmp ecx,eax
00433993   . 74 08          je short PCWorker.0043399D
```

检查 `Serial` 是否正确

转成代码大概是：

```c
int key1 = Serial[0];
int sum=0;
int key2;
for (i=1; i<9; i++)
  sum += Serial[ i ]
key2 = (sum % 0x5f) + 0x20
if (key1 == key2)
  正确!!
```

```assembly
004339DA   > \2BFD          sub edi,ebp                     ;  比较执行时间!! 有没有低于 0.5 s
004339DC   . 81FF F4010000  cmp edi,1F4
004339E2   . 76 08         jbe short PCWorker.004339EC
```

这里判断有没有被调试，强迫跳到 `004339EC` 吧

```assembly
00433C55   > /8A0408        mov al,byte ptr ds:[eax+ecx]
00433C58   . |884424 10    mov byte ptr ss:[esp+10],al
00433C5C   . |8B5424 10    mov edx,dword ptr ss:[esp+10]
00433C60   . |A1 28144500  mov eax,dword ptr ds:[<NameADD>]
00433C65   . |81E2 FF000000  and edx,0FF
00433C6B   . |03C2         add eax,edx
00433C6D   . |41          inc ecx
00433C6E   . |A3 28144500  mov dword ptr ds:[<NameADD>],eax
00433C73   . |8B07         mov eax,dword ptr ds:[edi]
00433C75   . |3B48 F8      cmp ecx,dword ptr ds:[eax-8]
00433C78   . ^\7C DB      jl short PCWorker.00433C55
```

`Name` 的 `add` 编码 (由左至右一个一个 `add`) => 取名 `NameADD`

```assembly
00433C7A   > \A1 04144500   mov eax,dword ptr ds:[<PCKey>]   ;  PCKey
00433C7F   . 8A48 02        mov cl,byte ptr ds:[eax+2]
00433C82   . 8D4424 10      lea eax,dword ptr ss:[esp+10]
00433C86   . 884C24 10      mov byte ptr ss:[esp+10],cl
00433C8A   . 8B5424 10      mov edx,dword ptr ss:[esp+10]
00433C8E   . 52             push edx                         ;  第 3 byte 为 rand key
00433C8F   . 50             push eax
00433C90   . E8 8BE9FFFF    call <PCWorker.算 PCKey>
00433C95   . 8B00           mov eax,dword ptr ds:[eax]
00433C97   . 8B8E 05010000  mov ecx,dword ptr ds:[esi+105]
00433C9D   . 50             push eax
00433C9E   . 51             push ecx
00433C9F   . FFD5           call ebp
00433CA1   . 83C4 10        add esp,10
00433CA4   . 8D4C24 10      lea ecx,dword ptr ss:[esp+10]
00433CA8   . 85C0           test eax,eax
00433CAA   . 0F95C3         setne bl
```

这里是验证 `PCKey` 的正确性，因为我们的 `PCKey` 是一开始执行时由 `pcworker` 产生的

一定正确! 有兴趣研究的朋友们，可进入 `00433C90` 这个 `call` 研究研究

```assembly
00433CD1   > /8A1408        mov dl,byte ptr ds:[eax+ecx]
00433CD4   . |885424 10    mov byte ptr ss:[esp+10],dl
```



### Name 的 XOR 编码

*   说明：由左至右一个一个 `xor` => 取名 `NameXOR`

```asm
00433CD8   .  |8B4424 10       mov eax,dword ptr ss:[esp+10]  
00433CDC   .  |8B15 28144500   mov edx,dword ptr ds:[<NameXOR>]  
00433CE2   .  |25 FF000000     and eax,0FF  
00433CE7   .  |33D0            xor edx,eax  
00433CE9   .  |41              inc ecx  
00433CEA   .  |8915 28144500   mov dword ptr ds:[<NameXOR>],edx  
00433CF0   .  |8B07            mov eax,dword ptr ds:[edi]  
00433CF2   .  |3B48 F8         cmp ecx,dword ptr ds:[eax-8]  
00433CF5   .^\7C DA            jl short PCWorker.00433CD1  
```

### Serial 校验代码逻辑

转成代码大概是：

```c
int key1 = Serial[9];  

int sum=0;  

int key2;  

for (i=1; i<9; i++)  
  sum = sum ^ Serial[ i ]  

key2 = (sum % 0x5f) + 0x20  

if (key1 == key2)  
  正确!!  
```

### Birthday 的 XOR 编码

*   说明：由左至右一个一个 `xor` => 取名 `BirthdayXOR`

```asm
00433D9A   .  8B96 11010000   mov edx,dword ptr ds:[esi+111]  
00433DA0   .  33C0            xor eax,eax  
00433DA2   .  8BCB            mov ecx,ebx  
00433DA4   >  0FBE3C11        movsx edi,byte ptr ds:[ecx+edx]  
00433DA8   .  33C7            xor eax,edi  
00433DAA   .  41              inc ecx  
00433DAB   .  83F9 08         cmp ecx,8  
00433DAE   .^ 7E F4           jle short PCWorker.00433DA4  
00433DB0   .  8A4A 09         mov cl,byte ptr ds:[edx+9]  
00433DB3   .  33D2            xor edx,edx  
00433DB5   .  BF 5F000000     mov edi,5F  
00433DBA   .  F7F7            div edi  
00433DBC   .  80C2 20         add dl,20  
00433DBF   .  3ACA            cmp cl,dl  
00433DC1   .  74 07           je short PCWorker.00433DCA  
```

### Birthday 的 ADD 编码

*   说明：由左至右一个一个 `add` => 取名 `BirthdayAdd`
*   提示：在 `4513f8` 下内存访问断点，可找到所有比较 `Serial` 的规则

```asm
00434141   >  /8A1408         mov dl,byte ptr ds:[eax+ecx]  
00434144   .  |8B1D 20144500  mov ebx,dword ptr ds:[<BirthdayAdd>]  
0043414A   .  |885424 10      mov byte ptr ss:[esp+10],dl  
0043414E   .  |8B5424 10      mov edx,dword ptr ss:[esp+10]  
00434152   .  |81E2 FF000000  and edx,0FF  
00434158   .  |03DA           add ebx,edx  
0043415A   .  |40             inc eax  
0043415B   .  |891D 20144500  mov dword ptr ds:[<BirthdayAdd>],ebx  
00434161   .  |8B51 F8        mov edx,dword ptr ds:[ecx-8]  
00434164   .  |3BC2           cmp eax,edx  
00434166   .^\7C D9           jl short PCWorker.00434141  
```

### PCKey 与 Serial 关联校验

```asm
00408674   .  8B85 1B0C0000   mov eax,dword ptr ss:[ebp+C1B]         ;  PCKeyAdd 编码  
0040867A   .  8B15 20144500   mov edx,dword ptr ds:[<BirthdayAdd 编码>]  
00408680   .  8B3D 1C144500   mov edi,dword ptr ds:[<NameAdd 编码>]  
00408686   .  03C2            add eax,edx  
00408688   .  03C7            add eax,edi  
0040868A   .  33D2            xor edx,edx  
0040868C   .  BE 5F000000     mov esi,5F  
00408691   .  8B0D F8134500   mov ecx,dword ptr ds:[<Serial>]  
00408697   .  F7F6            div esi  
00408699   .  8A49 01         mov cl,byte ptr ds:[ecx+1]  
0040869C   .  0FBEC1         movsx eax,cl  
0040869F   .  0FBED2         movsx edx,dl  
004086A2   .  83C2 20         add edx,20  
004086A5   .  3BC2            cmp eax,edx  
004086A7   .  74 07           je short PCWorker.004086B0  
```

#### 比较 `Serial[1]`

转换成代码如下：

```c
int sum = PCKeyAdd + BirthdayAdd + NameAdd  

char s1 = (sum % 0x5f) + 0x20  
```

#### 比较 `Serial[2]`

转换成代码如下：

```c
int sum = PCKeyXor ^ BirthdayXor ^ NameXor  

char s2 = (sum % 0x5f) + 0x20  
```

#### 比较 `Serial[3]`

转换成代码如下：

```c
int sum = PCKeyAdd ^ BirthdayAdd ^ NameAdd  

char s3 = (sum % 0x5f) + 0x20  
```

#### 比较 `Serial[4]`

转换成代码如下：

```c
int sum = PCKeyXor + BirthdayXor + NameXor  

char s4 = (sum % 0x5f) + 0x20  
```

#### 比较 `Serial[8]`

转换成代码如下：

```c
int sum = BirthdayXor & NameAdd | PCKeyXor  

char s8 = (sum % 0x5f) + 0x20  
```

其中 `Serial[5]` ~ `Serial[7]` 随便，不重要。

### 算法总结

最后，来整理一下算法：

1. `Serial` 是依 `PCKey`, `Name`, `Birthday` 算出来的。长度为 10 (`Serial[0]` ~ `Serial[9]`)。
2. 算出 `pckey`, `name`, `birthday` 的 `add`, `xor` 值。
3. 算出 `Serial[1]`, `Serial[2]`, `Serial[4]`, `Serial[4]`, `Serial[8]`。
4. `Serial[5]` ~ `Serial[7]` 随便填。
5. 算出 `Serial[0]` & `Serial[9]`。
6. 结束。

### Python 注册机 (Python 2.4)

**使用方式：**

```bash
D:\Data\Python>python PcworkerReg.py F-QYUC-TFGY-NGBN Skyer 1932/01/34
```

会产生 `pcworker.ii`，放到 `pcworker` 下即可。

```python
import sys, string

def CalAddSum(s):
  sum = 0
  smap = map(lambda x: ord(x), s)
  for i in smap:
    sum += i
  return sum

def CalXorSum(s):
  xors = 0
  smap = map(lambda x: ord(x), s)
  for i in smap:
    xors = xors ^ i
  return xors

def main(args):
  if len(args) != 4:
    print 'Usage: %s [PCKEY] [Name] [Birthday]' % (args[0])
    return
  pckey = string.replace(args[1], '-', '')
  pckey = string.upper(pckey)
  nameadd = CalAddSum(args[2])
  namexor = CalXorSum(args[2])
  birthadd = CalAddSum(args[3])
  birthxor = CalXorSum(args[3])
  pckeyadd = CalAddSum(pckey)
  pckeyxor = CalXorSum(pckey)

  #Serial, 2nd byte
  c1 = chr(((nameadd + birthadd + pckeyadd) % 0x5f) + 0x20)
  #Serial, 3rd byte
  c2 = chr(((namexor ^ birthxor ^ pckeyxor) % 0x5f) + 0x20)
  #Serial, 4th byte
  c3 = chr(((nameadd ^ birthadd ^ pckeyadd) % 0x5f) + 0x20)
  #Serial, 5th byte
  c4 = chr(((namexor + birthxor + pckeyxor) % 0x5f) + 0x20)
  #Serial, 9th byte
  c8 = chr(((birthxor & nameadd | pckeyxor) % 0x5f) + 0x20)

  key = c1+c2+c3+c4+'sky'+c8
  c0 = chr((CalAddSum(key) % 0x5f) + 0x20)
  c9 = chr((CalXorSum(key) % 0x5f) + 0x20)
  key = c0 + key + c9

  o = open('pcworker.ii', 'w')
  o.write('[Registration]\n')
  o.write('PCKEY=%s\n' % (pckey))
  o.write('Name=%s\n' % (args[2]))
  o.write('Birthday=%s\n' % (args[3]))
  o.write('Serial=%s\n' % (key))

if __name__ == '__main__':
  main(sys.argv)
```

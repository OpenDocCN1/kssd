
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

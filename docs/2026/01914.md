
# ToggleMOUSE 4.4.6 (5千字)

http://www.toggle.com

这是个很好的鼠标增强工具。它可以输入注册码，不过这里我们不是从输入注册码时开始跟踪，而是跟踪它启动时的判断过程，从而找到注册码。

## 启动时读取的注册表键

用RegMon看一下，发现它启动的时候要读如下几个键：

- `HKEY_CURRENT_USER\Software\ToggleSoftware\ToggleMOUSE\Registration\Name`
- `HKEY_CURRENT_USER\Software\ToggleSoftware\ToggleMOUSE\Registration\Company`
- `HKEY_CURRENT_USER\Software\ToggleSoftware\ToggleMOUSE\Registration\RegNumber`

## 反汇编代码片段

反汇编其主程序，找到如下的代码片段：

```
* Possible StringData Ref from Data Obj ->"Registration"
            |
:004108AB BEB8F64300              mov esi, 0043F6B8
* Possible StringData Ref from Data Obj ->"RegNumber"
            |
:004108B0 68ACF64300              push 0043F6AC
:004108B5 56                      push esi
:004108B6 8BC8                    mov ecx, eax
:004108B8 E83AE30100              call 0042EBF7
  //读取注册码
:004108BD 894508                  mov dword ptr [ebp+08], eax  //保存注册码
:004108C0 E82DDF0100              call 0042E7F2
:004108C5 8B4004                  mov eax, dword ptr [eax+04]
:004108C8 53                      push ebx
* Possible StringData Ref from Data Obj ->"Name"
            |
:004108C9 68A4F64300              push 0043F6A4
:004108CE 8D4DE8                  lea ecx, dword ptr [ebp-18]
:004108D1 56                      push esi
:004108D2 51                      push ecx
:004108D3 8BC8                    mov ecx, eax
:004108D5 E889E30100              call 0042EC63
  //读取Name
:004108DA 50                      push eax
:004108DB 8D4DF0                  lea ecx, dword ptr [ebp-10]
:004108DE E880480100              call 00425163
:004108E3 8D4DE8                  lea ecx, dword ptr [ebp-18]
:004108E6 E88B470100              call 00425076
:004108EB E802DF0100              call 0042E7F2
:004108F0 8B4004                  mov eax, dword ptr [eax+04]
:004108F3 53                      push ebx
* Possible StringData Ref from Data Obj ->"Company"
            |
:004108F4 689CF64300              push 0043F69C
:004108F9 8D4DE8                  lea ecx, dword ptr [ebp-18]
:004108FC 56                      push esi
:004108FD 51                      push ecx
:004108FE 8BC8                    mov ecx, eax
:00410900 E85EE30100              call 0042EC63
//读取Company
:00410905 50                      push eax
:00410906 8D4DEC                  lea ecx, dword ptr [ebp-14]
:00410909 E855480100              call 00425163
:0041090E 8D4DE8                  lea ecx, dword ptr [ebp-18]
:00410911 E860470100              call 00425076
:00410916 E8D7DE0100              call 0042E7F2
:0041091B 8B4004                  mov eax, dword ptr [eax+04]
:0041091E 33DB                    xor ebx, ebx
:00410920 53                      push ebx
* Possible StringData Ref from Data Obj ->"LastPageIndex"
            |
:00410921 688CF64300              push 0043F68C
* Possible StringData Ref from Data Obj ->"Settings"
            |
:00410926 685CE34300              push 0043E35C
:0041092B 8BC8                    mov ecx, eax
:0041092D E8C5E20100              call 0042EBF7
//读取注册码的校验和
:00410932 8945E8                  mov dword ptr [ebp-18], eax //保存校验和
:00410935 8D45F0                  lea eax, dword ptr [ebp-10]
:00410938 50                      push eax
:00410939 8D4DF8                  lea ecx, dword ptr [ebp-08]
:0041093C E822480100              call 00425163
:00410941 8D4DF8                  lea ecx, dword ptr [ebp-08]
:00410944 E8E34B0100              call 0042552C
:00410949 8B45F8                  mov eax, dword ptr [ebp-08]
:0041094C 8B37                    mov esi, dword ptr [edi]
:0041094E 3958F8                  cmp dword ptr [eax-08], ebx
:00410951 7E21                    jle 00410974
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00410972(C)
|
:00410953 0FBE0403                movsx eax, byte ptr [ebx+eax] //以下计算注册码
:00410957 50                      push eax
:00410958 E8B27A0000              call 0041840F
:0041095D 85C0                    test eax, eax
:0041095F 8B45F8                  mov eax, dword ptr [ebp-08]
:00410962 59                      pop ecx
:00410963 7409                    je 0041096E
:00410965 0FBE0C03                movsx ecx, byte ptr [ebx+eax]
:00410969 03CB                    add ecx, ebx
:0041096B 0FAFF1                  imul esi, ecx
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00410963(C)
|
:0041096E 43                      inc ebx
:0041096F 3B58F8                  cmp ebx, dword ptr [eax-08]
:00410972 7CDF                    jl 00410953    //循环
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00410951(C)
:00410974 6A64                    push 00000064
:00410976 8BC6                    mov eax, esi
:00410978 33D2                    xor edx, edx
:0041097A 59                      pop ecx
:0041097B F7F1                    div ecx
:0041097D 8B4D08                  mov ecx, dword ptr [ebp+08] //你输入的假注册码
:00410980 81F121332153            xor ecx, 53213321
:00410986 3B4DE8                  cmp ecx, dword ptr [ebp-18] //检查校验和
:00410989 7512                    jne 0041099D
:0041098B 394508                  cmp dword ptr [ebp+08], eax //比较真假注册码
:0041098E 750D                    jne 0041099D
:00410990 85C0                    test eax, eax
:00410992 7409                    je 0041099D
:00410994 C7470801000000          mov [edi+08], 00000001
//good guy
:0041099B EB04                    jmp 004109A1
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00410989(C), :0041098E(C), :00410992(C)
|
:0041099D 83670800                and dword ptr [edi+08],00000000  //bag guy
```

## 注册码验证逻辑

根据以上代码可知校验和等于注册码和常数`0x53213321`异或，校验和放在这里：

`HKEY_CURRENT_USER\Software\ToggleSoftware\ToggleMOUSE\Settings\LastPageIndex`

注册机也很好写，因其计算过程很简单。

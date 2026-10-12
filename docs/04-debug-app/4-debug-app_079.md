
# WINZIP8.0 PASSWORD 追踪部分认识

> 原文：[`www.pediy.com/kssd/pediy03/forum1279.htm`](https://www.pediy.com/kssd/pediy03/forum1279.htm)

## 操作步骤

1. 加密加压一个 `ABC.ZIP`
2. 解压 `ABC.ZIP`，输入 `PASSWORD`
3. `BPX HMEMCPY` 确认 `PASSWORD`
4. 按 `F12` 进入无模块提醒领空
5. `U 20001EC6`
6. `BPX 20001EC6`
7. `F5` 拦截成功如下

```assembly
:20001EC6 A1CC4D0620                  mov eax, dword ptr [20064DCC]--passwod
:20001ECB 56                          push esi
:20001ECC 50                          push eax
:20001ECD E83EFDFFFF                  call 20001C10----计算 1
:20001ED2 8B4D08                      mov ecx, dword ptr [ebp+08]
:20001ED5 83C404                      add esp, 00000004
:20001ED8 33F6                        xor esi, esi
:20001EDA 8B11                        mov edx, dword ptr [ecx]
:20001EDC 8955F4                      mov dword ptr [ebp-0C], edx
:20001EDF 8B4104                      mov eax, dword ptr [ecx+04]
:20001EE2 8945F8                      mov dword ptr [ebp-08], eax
:20001EE5 8B4908                      mov ecx, dword ptr [ecx+08]
:20001EE8 894DFC                      mov dword ptr [ebp-04], ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :20001F0E(C)
|
:20001EEB E890FCFFFF                  call 20001B80---计算 2
:20001EF0 8A5435F4                    mov dl, byte ptr [ebp+esi-0C]
:20001EF4 32D0                        xor dl, al
:20001EF6 8AC2                        mov al, dl
:20001EF8 885435F4                    mov byte ptr [ebp+esi-0C], dl
:20001EFC 25FF000000                  and eax, 000000FF
:20001F01 50                          push eax
:20001F02 E899FCFFFF                  call 20001BA0----计算 3
:20001F07 83C404                      add esp, 00000004
:20001F0A 46                          inc esi
:20001F0B 83FE0C                      cmp esi,0000000C--oc=12,
                                      ( the encryption header=12 bytes )
:20001F0E 7CDB                        jl 20001EEB--计算 2. 计算 3 循环 12 计算
:20001F10 8B15240F0320                mov edx, dword ptr [20030F24]
:20001F16 660FB645FF                  movzx ax, byte ptr [ebp-01]--计算结果
:20001F1B F6422002                    test [edx+20], 02
:20001F1F 7414                        je 20001F35
:20001F35 8B156A170820                mov edx, dword ptr [2008176A]--crc32
:20001F3B C1EA18                      shr edx, 18
:20001F3E 663BC2                      cmp ax, dx
:20001F41 7407                        je 20001F4A
```

### 计算 1

可能是 PAUL 文章如下部分：

```pseudocode
process_keys(key):
  key0_{1-l} <-- 0x12345678
  key1_{1-l} <-- 0x23456789
  key2_{1-l} <-- 0x34567890

  loop for i <-- 1 to l
    update_keys_{i-l}(key_{i})
  end loop
end process_keys
```

```assembly
:20001C10 55                          push ebp
:20001C11 8BEC                        mov ebp, esp
:20001C13 56                          push esi
:20001C14 8B7508                      mov esi, dword ptr [ebp+08]--password
:20001C17 C7058840032078563412        mov dword ptr [20034088], 12345678
:20001C21 C7058C40032089674523        mov dword ptr [2003408C], 23456789
:20001C2B C7059040032090785634        mov dword ptr [20034090], 34567890
:20001C35 8A06                        mov al, byte ptr [esi]--password
:20001C37 84C0                        test al, al
:20001C39 7416                        je 20001C51

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :20001C4F(C)
|
:20001C3B 25FF000000                  and eax, 000000FF
:20001C40 50                          push eax
:20001C41 E85AFFFFFF                  call 20001BA0 -----计算 1.1
:20001C46 8A4601                      mov al, byte ptr [esi+01]--next password
:20001C49 83C404                      add esp, 00000004
:20001C4C 46                          inc esi---password 长度+1
:20001C4D 84C0                        test al, al
:20001C4F 75EA                        jne 20001C3B  --password 长度=0 NO JUMP

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :20001C39(C)
:20001C51 5E                          pop esi
:20001C52 5D                          pop ebp
:20001C53 C3                          ret
```

### 计算 1.1

```assembly
* Referenced by a CALL at Addresses:
| :20001C41 , :20001CB7 , :20001D0B , :20001D44 , :20001D78
| :20001DEE , :20001F02 , :20001F93 , :20008ACC
|
:20001BA0 55                          push ebp
:20001BA1 8BEC                        mov ebp, esp
:20001BA3 8B1588400320                mov edx, dword ptr [20034088]-12345678
:20001BA9 8B4508                      mov eax, dword ptr [ebp+08]- password
:20001BAC 8BCA                        mov ecx, edx
:20001BAE 56                          push esi
:20001BAF 33C8                        xor ecx, eax
:20001BB1 81E1FF000000                and ecx, 000000FF
:20001BB7 C1EA08                      shr edx, 08
:20001BBA 8B0C8D04110320             mov ecx, dword ptr [4*ecx+20031104]
:20001BC1 33CA                        xor ecx, edx
:20001BC3 8B158C400320                mov edx, dword ptr [2003408C]-23456789
:20001BC9 890D88400320                mov dword ptr [20034088], ecx
:20001BCF 81E1FF000000                and ecx, 000000FF
:20001BD5 03CA                        add ecx, edx
:20001BD7 8B1590400320                mov edx, dword ptr [20034090]-3456790
:20001BDD 69C905840808                imul ecx, 08088405
:20001BE3 41                          inc ecx
:20001BE4 8BF2                        mov esi, edx
:20001BE6 890D8C400320                mov dword ptr [2003408C], ecx
:20001BEC 81E6FF000000                and esi, 000000FF
:20001BF2 C1E918                      shr ecx, 18
:20001BF5 33CE                        xor ecx, esi
:20001BF7 5E                          pop esi
:20001BF8 C1EA08                      shr edx, 08
:20001BFB 8B0C8D04110320             mov ecx, dword ptr [4*ecx+20031104]
:20001C02 33CA                        xor ecx, edx
:20001C04 890D90400320                mov dword ptr [20034090], ecx --[20034090]
                                      结果存放地址
:20001C0A 5D                          pop ebp
:20001C0B C3                          ret
```

### 计算 2

```assembly
:20001B80 8B0D90400320                mov ecx, dword ptr [20034090]
:20001B86 83C902                      or ecx, 00000002
:20001B89 8BC1                        mov eax, ecx
:20001B8B 83F001                      xor eax, 00000001
:20001B8E 0FAFC1                      imul eax, ecx
:20001B91 25FFFF0000                  and eax, 0000FFFF
:20001B96 C1E808                      shr eax, 08
:20001B99 C3                          ret
```

### 计算 3

```assembly
:20001BA0 55                          push ebp
:20001BA1 8BEC                        mov ebp, esp
:20001BA3 8B1588400320                mov edx, dword ptr [20034088]
:20001BA9 8B4508                      mov eax, dword ptr [ebp+08]
:20001BAC 8BCA                        mov ecx, edx
:20001BAE 56                          push esi
:20001BAF 33C8                        xor ecx, eax
:20001BB1 81E1FF000000                and ecx, 000000FF
:20001BB7 C1EA08                      shr edx, 08
:20001BBA 8B0C8D04110320             mov ecx, dword ptr [4*ecx+20031104]
:20001BC1 33CA                        xor ecx, edx
:20001BC3 8B158C400320                mov edx, dword ptr [2003408C]
:20001BC9 890D88400320                mov dword ptr [20034088], ecx
:20001BCF 81E1FF000000                and ecx, 000000FF
:20001BD5 03CA                        add ecx, edx
:20001BD7 8B1590400320                mov edx, dword ptr [20034090]
:20001BDD 69C905840808                imul ecx, 08088405
:20001BE3 41                          inc ecx
:20001BE4 8BF2                        mov esi, edx
:20001BE6 890D8C400320                mov dword ptr [2003408C], ecx
:20001BEC 81E6FF000000                and esi, 000000FF
:20001BF2 C1E918                      shr ecx, 18
:20001BF5 33CE                        xor ecx, esi
:20001BF7 5E                          pop esi
```



:20001BF8 C1EA08`shr edx, 08`   

:20001BFB 8B0C8D04110320`mov ecx, dword ptr [4*ecx+20031104]`   

:20001C02 33CA`xor ecx, edx`   

:20001C04 890D90400320`mov dword ptr [20034090], ecx`--好象最终结果是解密解压的 key   

:20001C0A 5D`pop ebp`   

:20001C0B C3`ret`   

KINGSUN   

版权所有 4dd466d0e1e0cc123afbe504da2462ca

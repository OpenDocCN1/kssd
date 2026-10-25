
```asm
:0041CB78 0FBE2C0E                movsx
ebp, byte ptr [esi+ecx]
:0041CB7C 8D5101
lea edx, dword ptr [ecx+01]
:0041CB7F 0FAFDD
imul ebx, ebp
:0041CB82 0FBE898CDB4200          movsx ecx, byte ptr
[ecx+0042DB8C]
:0041CB89 0FAFD9
imul ebx, ecx
:0041CB8C 0FAFDA
imul ebx, edx
:0041CB8F 03FB
  add edi, ebx
:0041CB91 8BCA
  mov ecx, edx
:0041CB93 3BC2
  cmp eax, edx
:0041CB95 7FD9
  jg 0041CB70
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0041CB63(C), :0041CB67(C), :0041CB6E(C)
|
:0041CB97 8BC7
  mov eax, edi
:0041CB99 5D
  pop ebp
:0041CB9A 5F
  pop edi
:0041CB9B 5E
  pop esi
:0041CB9C 5B
  pop ebx
:0041CB9D C3
  ret
```

### 五
由于本人能力有限，不能写出注册机，还请大客们指教呀！它好象有一个对照表。

总结：

```
User Name:RATARICE[BCG]
Organization:peNcil grOup
Registration:4105493258
```

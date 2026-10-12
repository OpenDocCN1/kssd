
## 汇编代码段 1

```
:004C9C18 E85344F4FF         call 0040E070 
  <!-- EAX=07/11/2002-P4X266E-8233A-6A6LWSNGC-00 -->
  <!-- 剔除字符串中非数字、字母的符号 -->

:004C9C1D 8B55F0             mov edx, dword ptr [ebp-10]
  <!-- EDX=07112002-P4X266E-8233A-6A6LWSNGC-00 -->

:004C9C20 8D45F4             lea eax, dword ptr [ebp-0C]
:004C9C23 E8B0ABF3FF         call 004047D8
:004C9C28 A0749D4C00         mov al, byte ptr [004C9D74]
:004C9C2D 50                 push eax
:004C9C2E 8D45EC             lea eax, dword ptr [ebp-14]
:004C9C31 50                 push eax
:004C9C32 33C9               xor ecx, ecx
:004C9C34 BA8C9D4C00         mov edx, 004C9D8C
:004C9C39 8B45F4             mov eax, dword ptr [ebp-0C]
:004C9C3C E82F44F4FF         call 0040E070
:004C9C41 8B55EC             mov edx, dword ptr [ebp-14]
:004C9C44 8D45F4             lea eax, dword ptr [ebp-0C]
:004C9C47 E88CABF3FF         call 004047D8
:004C9C4C A0749D4C00         mov al, byte ptr [004C9D74]
:004C9C51 50                 push eax
:004C9C52 8D45E8             lea eax, dword ptr [ebp-18]
:004C9C55 50                 push eax
:004C9C56 33C9               xor ecx, ecx
:004C9C58 BA989D4C00         mov edx, 004C9D98
:004C9C5D 8B45F4             mov eax, dword ptr [ebp-0C]
  <!-- EDX=07112002-P4X266E-8233A-6A6LWSNGC-00 -->

:004C9C60 E80B44F4FF         call 0040E070
  <!-- 剔除字符串中非数字、字母的符号 -->

:004C9C65 8B55E8             mov edx, dword ptr [ebp-18]
  <!-- EDX=07112002P4X266E8233A6A6LWSNGC00 -->

:004C9C68 8D45F4             lea eax, dword ptr [ebp-0C]
:004C9C6B E868ABF3FF         call 004047D8
:004C9C70 A0749D4C00         mov al, byte ptr [004C9D74]
:004C9C75 50                 push eax
:004C9C76 8D45E4             lea eax, dword ptr [ebp-1C]
:004C9C79 50                 push eax
:004C9C7A 33C9               xor ecx, ecx
:004C9C7C BAA49D4C00         mov edx, 004C9DA4
:004C9C81 8B45F4             mov eax, dword ptr [ebp-0C]
:004C9C84 E8E743F4FF         call 0040E070
:004C9C89 8B55E4             mov edx, dword ptr [ebp-1C]
:004C9C8C 8D45F4             lea eax, dword ptr [ebp-0C]
:004C9C8F E844ABF3FF         call 004047D8
:004C9C94 8B4DF8             mov ecx, dword ptr [ebp-08]
* Possible StringData Ref from Code Obj ->"1978"

:004C9C97 BAB09D4C00         mov edx, 004C9DB0
  <!-- EDX=004C9DB0=1978 -->

:004C9C9C 8B45F4             mov eax, dword ptr [ebp-0C]
  <!-- EAX=07112002P4X266E8233A6A6LWSNGC00 -->

:004C9C9F E8FC89FFFF         call 004C26A0
  <!-- 用 1978 和上面的主板信息循环运算得出一组新值 -->

:004C9CA4 8B45F8             mov eax, dword ptr [ebp-08]
:004C9CA7 50                 push eax
:004C9CA8 8B45F8             mov eax, dword ptr [ebp-08]
:004C9CAB 8B00               mov eax, dword ptr [eax]
  <!-- EAX=0809799DF6187196F0709DC2CC3251A1E12561A3DC225AA7CB125C9DDC11789F -->

:004C9CAD E84EADF3FF         call 00404A00
  <!-- 取上面字符串的长度 -->

:004C9CB2 8BD8               mov ebx, eax
  <!-- EBX=40 -->

:004C9CB4 8B45F4             mov eax, dword ptr [ebp-0C]
  <!-- EAX=07112002P4X266E8233A6A6LWSNGC00 -->

:004C9CB7 E844ADF3FF         call 00404A00
  <!-- 取上面字符串的长度 EAX=1F -->

:004C9CBC 2BD8               sub ebx, eax
  <!-- EBX=40 - 1F=21（H）=33（D） -->

:004C9CBE 53                 push ebx
:004C9CBF 8B45F4             mov eax, dword ptr [ebp-0C]
  <!-- EAX=07112002P4X266E8233A6A6LWSNGC00 -->

:004C9CC2 E839ADF3FF         call 00404A00
  <!-- 取上面字符串的长度 -->

:004C9CC7 8BC8               mov ecx, eax
  <!-- ECX=1F（H）=31（D） -->

:004C9CC9 8B45F8             mov eax, dword ptr [ebp-08]
:004C9CCC 8B00               mov eax, dword ptr [eax]
  <!-- EAX=0809799DF6187196F0709DC2CC3251A1E12561A3DC225AA7CB125C9DDC11789F -->

:004C9CCE 5A                 pop edx
  <!-- EDX=21（H）=33（D） -->

:004C9CCF E88CAFF3FF         call 00404C60
  <!-- 从第 33 位开始取上面字符串中的 31 位字符 -->
  <!-- E12561A3DC225AA7CB125C9DDC11789 这就是认证码了 -->

:004C9CD4 33C0               xor eax, eax
:004C9CD6 5A                 pop edx
:004C9CD7 59                 pop ecx
:004C9CD8 59                 pop ecx
:004C9CD9 648910             mov dword ptr fs:[eax], edx
:004C9CDC 68FE9C4C00         push 004C9CFE

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C9CFC(U)

:004C9CE1 8D45E4             lea eax, dword ptr [ebp-1C]
:004C9CE4 BA05000000         mov edx, 00000005
:004C9CE9 E876AAF3FF         call 00404764
:004C9CEE 8D45FC             lea eax, dword ptr [ebp-04]
:004C9CF1 E84AAAF3FF         call 00404740
:004C9CF6 C3                 ret
```

## 算法 CALL：`4C2C92` 调用 `004C9DB8`

* Referenced by a CALL at Address:
|:004C2C92

```
:004C9DB8 55                 push ebp
:004C9DB9 8BEC               mov ebp, esp
:004C9DBB 6A00               push 00000000
:004C9DBD 6A00               push 00000000
:004C9DBF 6A00               push 00000000
:004C9DC1 6A00               push 00000000
:004C9DC3 6A00               push 00000000
:004C9DC5 6A00               push 00000000
:004C9DC7 53                 push ebx
:004C9DC8 8BD9               mov ebx, ecx
:004C9DCA 8955FC             mov dword ptr [ebp-04], edx
:004C9DCD 8B45FC             mov eax, dword ptr [ebp-04]
  <!-- EAX=E12561A3DC225AA7CB125C9DDC11789 -->

:004C9DD0 E81BAEF3FF         call 00404BF0
:004C9DD5 33C0               xor eax, eax
:004C9DD7 55                 push ebp
:004C9DD8 68F99E4C00         push 004C9EF9
:004C9DDD 64FF30             push dword ptr fs:[eax]
:004C9DE0 648920             mov dword ptr fs:[eax], esp
:004C9DE3 8BC3               mov eax, ebx
:004C9DE5 E856A9F3FF         call 00404740
:004C9DEA A0089F4C00         mov al, byte ptr [004C9F08]
:004C9DEF 50                 push eax
:004C9DF0 8D45F4             lea eax, dword ptr [ebp-0C]
:004C9DF3 50                 push eax
:004C9DF4 33C9               xor ecx, ecx
:004C9DF6 BA149F4C00         mov edx, 004C9F14
:004C9DFB 8B45FC             mov eax, dword ptr [ebp-04]
:004C9DFE E86D42F4FF         call 0040E070
:004C9E03 8B55F4             mov edx, dword ptr [ebp-0C]
:004C9E06 8D45FC             lea eax, dword ptr [ebp-04]
:004C9E09 E8CAA9F3FF         call 004047D8
:004C9E0E A0089F4C00         mov al, byte ptr [004C9F08]
:004C9E13 50                 push eax
:004C9E14 8D45F0             lea eax, dword ptr [ebp-10]
:004C9E17 50                 push eax
:004C9E18 33C9               xor ecx, ecx
:004C9E1A BA209F4C00         mov edx, 004C9F20
:004C9E1F 8B45FC             mov eax, dword ptr [ebp-04]
```

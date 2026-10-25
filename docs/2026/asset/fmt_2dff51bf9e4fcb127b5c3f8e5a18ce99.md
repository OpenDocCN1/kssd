
```asm
:004A14BD 8B45E4 mov eax, dword ptr [ebp-1C]  <====从这里可以整理出计算公式:(第二组+第五组)-第四组=第二组
:004A14C0 03C3   add eax, ebx                <====有这个公式你就可以编出注册机了
:004A14C2 2BC7   sub eax, edi
:004A14C4 3BF0   cmp esi, eax
:004A14C6 741A   je 004A14E2
:004A14C8 6A00   push 00000000
:004A14CA 668B0DBC164A00          mov cx, word ptr [004A16BC]
:004A14D1 33D2   xor edx, edx

\* Possible StringData Ref from Code Obj ->"您输入的序列号有误，请检查输入。"
            |
:004A14D3 B8C8164A00              mov eax, 004A16C8
:004A14D8 E8B739F9FF              call 00434E94
:004A14DD E989010000              jmp 004A166B
```

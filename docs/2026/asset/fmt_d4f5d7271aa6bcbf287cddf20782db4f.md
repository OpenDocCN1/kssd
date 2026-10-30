
```asm
.text:0000555555554E4B                 public challenge5
.text:0000555555554E4B challenge5      proc near               ; CODE XREF: start+1B3↓p
.text:0000555555554E4B
.text:0000555555554E4B var_58          = qword ptr -58h
.text:0000555555554E4B var_48          = dword ptr -48h
.text:0000555555554E4B var_44          = dword ptr -44h
.text:0000555555554E4B var_40          = dword ptr -40h
.text:0000555555554E4B var_20          = dword ptr -20h
.text:0000555555554E4B var_8           = qword ptr -8
.text:0000555555554E4B
.text:0000555555554E4B                 push    rbp
.text:0000555555554E4C                 mov     rbp, rsp
.text:0000555555554E4F                 sub     rsp, 60h
.text:0000555555554E53                 mov     [rbp+var_58], rdi
.text:0000555555554E57                 mov     rax, fs:28h
.text:0000555555554E60                 mov     [rbp+var_8], rax
.text:0000555555554E64                 xor     eax, eax
.text:0000555555554E66                 mov     edi, 0          ; timer
.text:0000555555554E6B                 call    _time
.text:0000555555554E70                 mov     edi, eax        ; seed
.text:0000555555554E72                 call    _srand
.text:0000555555554E77                 mov     [rbp+var_48], 0
.text:0000555555554E7E                 jmp     short loc_555555554EA1
.text:0000555555554E80 ; ----------------------------------------
.text:0000555555554E80
.text:0000555555554E80 loc_555555554E80:                       ; CODE XREF: challenge5+5A↓j
.text:0000555555554E80                 mov     eax, [rbp+var_48]
.text:0000555555554E83                 cdqe
.text:0000555555554E85                 mov     [rbp+rax*4+var_40], 0
.text:0000555555554E8D                 call    _rand
.text:0000555555554E92                 mov     edx, eax
.text:0000555555554E94                 mov     eax, [rbp+var_48]
.text:0000555555554E97                 cdqe
.text:0000555555554E99                 mov     [rbp+rax*4+var_20], edx
.text:0000555555554E9D                 add     [rbp+var_48], 1
.text:0000555555554EA1
.text:0000555555554EA1 loc_555555554EA1:                       ; CODE XREF: challenge5+33↑j
.text:0000555555554EA1                 cmp     [rbp+var_48], 4
.text:0000555555554EA5                 jle     short loc_555555554E80
.text:0000555555554EA7                 mov     [rbp+var_44], 0
.text:0000555555554EAE                 jmp     short loc_555555554ED3
.text:0000555555554EB0 ; ----------------------------------------
.text:0000555555554EB0
.text:0000555555554EB0 loc_555555554EB0:                       ; CODE XREF: challenge5+8C↓j
.text:0000555555554EB0                 mov     eax, [rbp+var_44]
.text:0000555555554EB3                 cdqe
.text:0000555555554EB5                 mov     edx, [rbp+rax*4+var_40]
.text:0000555555554EB9                 mov     eax, [rbp+var_44]
.text:0000555555554EBC                 cdqe
.text:0000555555554EBE                 mov     eax, [rbp+rax*4+var_20]
.text:0000555555554EC2                 cmp     edx, eax
.text:0000555555554EC4                 jz      short loc_555555554ECF
.text:0000555555554EC6                 mov     rax, [rbp+var_58]
.text:0000555555554ECA                 mov     byte ptr [rax], 0
.text:0000555555554ECD                 jmp     short loc_555555554EE0
.text:0000555555554ECF ; ----------------------------------------
.text:0000555555554ECF
.text:0000555555554ECF loc_555555554ECF:                       ; CODE XREF: challenge5+79↑j
.text:0000555555554ECF                 add     [rbp+var_44], 1
.text:0000555555554ED3
.text:0000555555554ED3 loc_555555554ED3:                       ; CODE XREF: challenge5+63↑j
.text:0000555555554ED3                 cmp     [rbp+var_44], 4
.text:0000555555554ED7                 jle     short loc_555555554EB0
.text:0000555555554ED9                 mov     rax, [rbp+var_58]
.text:0000555555554EDD                 mov     byte ptr [rax], 1
.text:0000555555554EE0
.text:0000555555554EE0 loc_555555554EE0:                       ; CODE XREF: challenge5+82↑j
.text:0000555555554EE0                 mov     rax, [rbp+var_8]
.text:0000555555554EE4                 xor     rax, fs:28h
.text:0000555555554EED                 jz      short locret_555555554EF4
.text:0000555555554EEF                 call    ___stack_chk_fail
.text:0000555555554EF4 ; ----------------------------------------
.text:0000555555554EF4
.text:0000555555554EF4 locret_555555554EF4:                    ; CODE XREF: challenge5+A2↑j
.text:0000555555554EF4                 leave
.text:0000555555554EF5                 retn
.text:0000555555554EF5 challenge5      endp
```

challenge 5 伪代码如下：

```c
unsigned __int64 __fastcall challenge5(_BYTE *a1)
{
  unsigned int v1; // eax
  int i; // [rsp+18h] [rbp-48h]
  int j; // [rsp+1Ch] [rbp-44h]
  int v5[14]; // [rsp+20h] [rbp-40h]
  unsigned __int64 v6; // [rsp+58h] [rbp-8h]

  v6 = __readfsqword(0x28u);
  v1 = time(0LL);
  srand(v1); // srand 使用时间作为种子，生成真正的随机数
  for ( i = 0; i <= 4; ++i )
  {
    v5[i] = 0; // 0,1,2,3,4
    v5[i + 8] = rand(); // 8,9,19,11,12
  }
  for ( j = 0; j <= 4; ++j ) // 我们需要绕过此循环
  {
    if ( v5[j] != v5[j + 8] )
    {
      *a1 = 0;
      return __readfsqword(0x28u) ^ v6;
    }
  }
  *a1 = 1;
  return __readfsqword(0x28u) ^ v6;
}
```

分析代码逻辑，我们想要最后得到`*a1 = 1`的话，需要绕过第二个for循环。让`rand()`得到的值都为0就可以了。hook方法在官方手册 [Hijacking OS API (POSIX)](https://bbs.kanxue.com/elink@ae9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2M7h3W2D9K9h3&6Y4i4K6u0W2K9h3!0Q4x3V1k6W2L8W2)9J5c8X3I4S2N6r3g2K6N6q4)9J5c8X3S2A6K9X3q4U0K9#2)9J5c8R3`.`.) 。

## writeup 5

```python
def hook_rand(ql):
    ql.arch.regs.rax = 0 # 在x86-64架构中，函数的返回值通常保存在RAX寄存器中。rand函数的返回值会存储在rax寄存器
def challenge5(ql):
    ql.os.set_api('rand', hook_rand)
```

运行后没有显示第五个挑战是否被解决，因为被第六题的死循环卡住了。

# Challenge 6：修改寄存器值

> Challenge 6: Avoid the infinite loop.

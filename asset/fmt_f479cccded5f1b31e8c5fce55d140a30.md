
```assembly
.text:00401030               ; ================ S U B R O U T I N E =======================================
.text:00401030
.text:00401030               ; Attributes: bp-based frame
.text:00401030
.text:00401030               _main_0 proc near            ; CODE XREF: _main j
.text:00401030
.text:00401030               var_48= byte ptr -48h
.text:00401030               var_8= dword ptr -8
.text:00401030               var_4= dword ptr -4
.text:00401030
.text:00401030 55            push    ebp
.text:00401031 8B EC         mov     ebp, esp
.text:00401033 83 EC 48       sub     esp, 48h
.text:00401036 53            push    ebx
.text:00401037 56            push    esi
.text:00401038 57            push    edi
.text:00401039 8D 7D B8      lea     edi, [ebp+var_48]
.text:0040103C B9 12 00 00 00  mov     ecx, 12h
.text:00401041 B8 CC CC CC CC mov     eax, 0CCCCCCCCh
.text:00401046 F3 AB         rep stosd
.text:00401048 6A 0C         push    0Ch            ; Size
.text:0040104A E8 41 71 00 00 call    _malloc
.text:0040104F 83 C4 04      add     esp, 4
.text:00401052 89 45 FC      mov     [ebp+var_4], eax  //eax 寄存器返回数组的起始地址
.text:00401055 C7 45 F8 02 00 00+ mov     [ebp+var_8], 2
.text:0040105C 8B 45 FC      mov     eax, [ebp+var_4]
.text:0040105F C7 00 0A 00 00 00 mov     dword ptr [eax], 0Ah
.text:00401065 8B 4D FC      mov     ecx, [ebp+var_4]
.text:00401068 C7 41 04 14 00 00+ mov     dword ptr [ecx+4], 14h
.text:0040106F 8B 55 FC      mov     edx, [ebp+var_4]
.text:00401072 C7 42 08 1E 00 00+ mov     dword ptr [edx+8], 1Eh
.text:00401079 8B 45 F8      mov     eax, [ebp+var_8]
.text:0040107C 8B 4D FC      mov     ecx, [ebp+var_4]
.text:0040107F C7 04 81 28 00 00+ mov     dword ptr [ecx+eax*4], 28h
.text:00401086 33 C0         xor     eax, eax
.text:00401088 5F            pop     edi
.text:00401089 5E            pop     esi
.text:0040108A 5B            pop     ebx
.text:0040108B 83 C4 48      add     esp, 48h
.text:0040108E 3B EC         cmp     ebp, esp
.text:00401090 E8 8B 8F 00 00 call    __chkesp
.text:00401095 8B E5         mov     esp, ebp
.text:00401097 5D            pop     ebp
.text:00401098 C3            retn
.text:00401098               _main_0 endp
.text:00401098
.text:00401098               ; ---------------------------------------------------------------------------
```

## 学习总结

1.  关于数组的使用，通过上面的分析能够得到唯一的确定结论是:只有当变量被用作数组的索引时，才最容易确定数组的存在。但其实我们可以反过来想，一般我们写程序使用数组都是用变量索引来访问数组元素的，很少有常量索引来访问数组元素，所以我们要把握重点，对于具体问题具体分析。

2.  要访问数组中的元素，首先需要用索引乘以数组元素的大小，计算出相应元素的偏移量，然后将得到的偏移量与数组的基址相加，得到数组元素的访问地址。

3.  对于变量索引访问数组的我们一般可以得到数组元素的大小，这一点对于分析也有一定的帮助。

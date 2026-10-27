
仔细检查后，第一个字节的值(以前为`E9`)将在`0x004032CA`被`VMInstruction`数据数组的第一个字节覆盖。我在解压的VM字节码(样本中的操作码`0x06`)中找到了此指令的一个实例，并查看指令数据中第一个字节的值，即`0xEB`(实际上，每个操作码为`0x06`的VM指令其第一个数据字节都是`0xEB`)。然后，用`EB`代替`E9`，我们最终得到了更合理的反汇编代码：

```
.text:0041EFBC EB 02              jmp     short loc_41EFC0

.text:0041EFBC

.text:0041EFBC                ; --------------------------------

.text:0041EFBE F8                 db 0F8h ; °

.text:0041EFBF B0                 db 0B0h ; ¦

.text:0041EFC0                ; --------------------------------

.text:0041EFC0

.text:0041EFC0                loc_41EFC0:

.text:0041EFC0 F9                 stc     ; Set carry flag

.text:0041EFC1 B8 C9 33 40 00     mov     eax, offset loc_4033C9

.text:0041EFC6 FF E0              jmp     eax
```

现在可以分析上面代码中引用的位置，即`loc_4033C9`：

```
.text:00403371         jb      short loc_4032FD ; JB will be taken, since carry flag was set previously

; JB path

    ; The code here is identical to the VM conditional branch taken re-entry sequence

; JNB path

    ; No real point in analyzing this, as the JB will always be taken
    .text:0040327C         add     [ebx+VMContext.pCurrInsn], size VMInstruction
    .text:00403426         mov     eax, [ebx+VMContext.fpVMEntry] 
    .text:004033E6         mov     esp, [ebx+VMContext.SavedESP1] 
    .text:00403345         jmp     [ebx+VMContext.fpVMReEntryToFuncPtr]
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2018-3-19 11:05

被`houjingyi`编辑，原因：

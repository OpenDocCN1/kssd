
```
 1 // Hook information.

 2 typedef struct _HOOK_ENTRY

 3 {

 4     LPVOID TargetFunctionAddress;        //目标地址

 5     LPVOID FakeFunctionAddress;          //Fake地址即覆盖地址

 6     LPVOID TrampolineMemorySlot;         // Address of the trampoline function.

 7     UINT8  OriginalDataBackup[8];        // Original prologue of the target function.目标功能的原始序幕- //恢复Hook使用的存放原先数据

 8 

 9     UINT8  PatchAbove : 1;    // Uses the hot patch area.  备份原函数的5字节，重要！！！

10     UINT8  IsEnabled : 1;     // Enabled.启用或者关闭

11     UINT8  queueEnable : 1;   // Queued for enabling/disabling when != isEnabled.

12     

13     UINT   IP : 4;            // Count of the instruction boundaries.索引 想到汇编的IP就很明白了

14     UINT8  OldIPs[8];         // Instruction boundaries of the target function.原地址的字节变化就靠它了

15     UINT8  NewIPs[8];         // Instruction boundaries of the trampoline function 用在后续解释的MemorySlot中

16 } HOOK_ENTRY, *PHOOK_ENTRY;   //44字节

17 

18 

19 typedef struct _HOOK_INFORMATION_

20 {

21     PHOOK_ENTRY Items;            // Data heap

22     UINT        MaximumLength;    // Size of allocated data heap, items

23     UINT        Length;           // Actual number of data items

24 }HOOK_INFORMATION,*PHOOK_INFORMATION;
```

当有了这个结构体后就可以去`CreateHook`了，下面是构建过程:

```
 1     if (CreateTrampoline(&Tl))

 2     {

 3     PHOOK_ENTRY HookEntry = AddHookEntry(); //填充一个HookInfo信息

 4        if (HookEntry != NULL)

 5     {

 6 HookEntry->TargetFunctionAddress = Tl.TargetFunctionAddress;       

 7 #if defined(_M_X64) || defined(__x86_64__)

 8             HookEntry->FakeFunctionAddress = Tl.pRelay;//跳转在trampolinex64重要

 9 #else

10 HookEntry->FakeFunctionAddress = Tl.FakeFunctionAddress;

11 #endif

12 HookEntry->TrampolineMemorySlot = Tl.MemorySlot;

13 HookEntry->PatchAbove = Tl.PatchAbove

14 HookEntry->IsEnabled = FALSE;

15     //HookEntry->QueueEnable = FALSE;

16 HookEntry->IP = Tl.IP;

17 

18 memcpy(HookEntry->OldIPs, Tl.OldIPs, ARRAYSIZE(Tl.OldIPs));

19 memcpy(HookEntry->NewIPs, Tl.NewIPs, ARRAYSIZE(Tl.NewIPs));

20 

21 // Back up the target function.

22 

23 if (Tl.PatchAbove)//这就是热补丁

24 {

25 memcpy(

26 HookEntry->OriginalDataBackup,

27 (LPBYTE)TargetFunctionAddress - sizeof(JMP_REL),

28 sizeof(JMP_REL) + sizeof(JMP_REL_SHORT));

29 }

30 else

31 {    //存储源函数的数据内容    

32 memcpy(HookEntry->OriginalDataBackup, TargetFunctionAddress, sizeof(JMP_REL));

33 }

34 if (OriginalWhitelist != NULL)//白名单，用来恢复

35 {

36 *OriginalWhitelist = HookEntry->TrampolineMemorySlot;

37 }
```

到这里为止终于是创建了Hook

## 第二幕 `EnableHook`

顾名思义就是启动Hook，显而易见得知它的作用无非就是覆盖原函数我们记录的那7字节，如下:

```
1 //SHELLCODE

2         PJMP_REL jmp = (PJMP_REL)PatchData;

3         jmp->Opcode = 0xE9;//跳转

4         jmp->Operand = (UINT32)((LPBYTE)HookEntry->FakeFunctionAddress - (PatchData + sizeof(JMP_REL)));

5
```

当需要解除Hook时候我们就可以用到在前面说过的`OriginalDataBackup`去恢复原函数，或者直接调用`MemorySlot`中记录下的原始序幕

```
1         else

2         {

3             memcpy(PatchData, HookEntry->OriginalDataBackup, sizeof(JMP_REL));

4         }
```

## 第三幕 `MessageBoxW`测试

```
 1 if (CreateHook(&MessageBoxW, &FakeMessageBox,

 2         reinterpret_cast<LPVOID*>(&__OriginalMessageBoxW)) != STATUS_SUCCESS)//告知要hook成什么样子

 3     {

 4         return;

 5     }

 6 

 7     MessageBoxW(0, L"MessageBoxW", L"MessageBoxW", 0);//没有Hook还是原先,不要也行

 8     if (EnableHook(MessageBoxW) != STATUS_SUCCESS)

 9     {

10         printf("EnableHook is wrong\r\n");

11         return;

12     }

13     MessageBoxW(NULL, L"CreateHook()", L"CreateHook()", 0);//启动Hook后，现在是FakeHOOK

14 

15     printf("Input AnyKey To Exit\r\n");

16     getchar();

17 

18     Uninitialize();//返回释放

19 }

20 

21 int WINAPI FakeMessageBox(

22     _In_opt_ HWND    DialogHwnd,

23     _In_opt_ WCHAR*  DialogText,

24     _In_opt_ WCHAR*  DialogCaption,

25     _In_     UINT    Type

26 )

27 {

28     __OriginalMessageBoxW(DialogHwnd, L"FakeMessageBox", L"FakeMessageBox", Type);

29     return 0;

30 }
```

编译运行后出结果，先是原先的`MessageBoxW`:

![](https://bbs.kanxue.com/upload/attach/201709/4BQ965WQAGW5J4A.jpg)

![](https://bbs.kanxue.com/upload/attach/201709/KGUMANJT8W8BTYG.jpg)

一切顺利，没有白费功夫，下面是我对`EB`，`call`，热补丁的汇编源码，我们仿照`MessageBoxW`的形式在`test.cpp`中定义函数指针，与Fake函数的输出形式。

在这里花费了功夫探索出了热补丁的简单定义是申请5字节空的内存然后 `mov edi,edi`，能应用正确，汇编代码如下

```
  .DATA

	MessageBoxW dq 0

.CODE	







Asm_OnInitMember PROC



	mov  qword ptr[rsp+8h],rcx

	push        rbp  

	push        rdi  

	sub         rsp,28h

	mov         rax,qword ptr[rsp+28h+8h+8h+8h]  

	mov         MessageBoxW,rax

	add         rsp,28h

	pop         rdi

	pop         rbp



ret

Asm_OnInitMember ENDP







Asm_1 PROC



   	mov  qword ptr[rsp+8h],rcx

	push        rbp  

	push        rdi 

	sub         rsp,28h

	xor         rbx,rbx 

	;00007FF77A8012BC E9 7A 0B 00 00       jmp         Asm_4 (07FF77A801E3Bh)  

    mov     rax,qword ptr[rsp+28h+8h+8h+8h]

	mov     ebx,dword ptr[rax+1]

	add     rax,rbx

	add     rax,5

	add         rsp,28h

	pop     rdi

	pop     rbp

ret

Asm_1 ENDP





Asm_3 PROC

	jmp Label1

Label1:

    jmp Label2

Label2:

	mov eax,-3

ret

Asm_3 ENDP





Asm_4 PROC

	call Label0

	jmp  Exit;

Label0:

	mov rcx,0;

	call Label1;    //Call 

		db 'H'

		db 0

		db 'e'

		db 0

		db 'l'

		db 0

		db 'l'

		db 0

		db 'o'

		db 0

		db 'S'

		db 0

		db 'u'

		db 0

		db 'b'

		db 0 

		db '_'

		db 0 

		db '4'

		db 0

		db 0

		db 0

Label1:

    pop rdx

	call Label2;

		db 'H'

		db 0

		db 'e'

		db 0

		db 'l'

		db 0

		db 'l'

		db 0

		db 'o'

		db 0

		db 'S'

		db 0

		db 'u'

		db 0

		db 'b'

		db 0 

		db '_'

		db 0 

		db '4'

		db 0

		db 0

		db 0

Label2:

     pop r8

	 mov r9,0

	    call MessageBoxW

		ret

Exit:

ret

Asm_4 ENDP





Asm_10  PROC

   db 0CCh

   db 0CCh

   db 0CCh

   db 0CCh

   db 0CCh

  

   mov edi,edi

   ret

Asm_10 ENDP

END
```

```
 1 //热补丁测试

      typedef void(*LPFN_SUB_10)();

      void FakeSub_10();   //热补丁

      LPFN_SUB_10       __OriginalSub_10 = NULL;

 2     

       PVOID v10 = Asm_1(Asm_10);

 3 

 4     if (SeCreateHook((PVOID)((ULONG_PTR)v10 + 5), &FakeSub_10,

 5         reinterpret_cast<LPVOID*>(&__OriginalSub_10)) != STATUS_SUCCESS)

 6     {

 7         return;

 8     }

 9     //对于热补丁函数调用

10     ((LPFN_SUB_10)(((ULONG_PTR)v10 + 5)))();

11 if (SeEnableHook(ALL_HOOKS) != STATUS_SUCCESS)

12     {

13         printf("SeEnableHook() Error\r\n");

14         return;

15     }

16 ((LPFN_SUB_10)(((ULONG_PTR)v10 + 5)))();
```

E9的测试只需要自写一个函数调用测试调用即可，如下面这样就行了然后在仿照上面自行测试即可

```
1 1 //E9指令，这样就行了

2 2 

3 3 void Sub_2()

4 4 {

5 5     printf("Sub_2\n\r");

6 6 }
```

下面是所有的正确输出结果:

![](https://bbs.kanxue.com/upload/attach/201709/SMTZ5SZAQG7DFNG.jpg)

好了,x64下的MiniHook终于是测试完了，写了一遍后又是更懂了，如果有什么差错，望大家纠正

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [Buffer.cpp](javascript:void(0);)

  （4.99kb，135次下载）

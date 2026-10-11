
## 逆向分析与脱壳步骤

### 1. 修改跳转逻辑

> `00AB087F 55 push ebp 00AB0880 8BEC mov ebp,esp 00AB0882 83C4 FC add esp,-4 00AB0885 53 push ebx 00AB0886 57 push edi 00AB0887 56 push esi 00AB0888 E8 00000000 call 00AB088D 00AB088D 5B pop ebx 00AB088E 81EB B0180010 sub ebx,100018B0 00AB0894 8B45 0C mov eax,dword ptr ss:[ebp+C] 00AB0897 8983 9A180010 mov dword ptr ds:[ebx+1000189A],eax 00AB089D 33C0 xor eax,eax 00AB089F 8983 9E180010 mov dword ptr ds:[ebx+1000189E],eax 00AB08A5 33F6 xor esi,esi 00AB08A7 46 inc esi 00AB08A8 46 inc esi 00AB08A9 8B93 2E1B0010 mov edx,dword ptr ds:[ebx+10001B2E] 00AB08AF FF75 0C push dword ptr ss:[ebp+C] 00AB08B2 FFD2 call edx 00AB08B4 8945 FC mov dword ptr ss:[ebp-4],eax 00AB08B7 8B93 3A1B0010 mov edx,dword ptr ds:[ebx+10001B3A] 00AB08BD 85C0 test eax,eax 00AB08BF 75 05 jnz short 00AB08C6 00AB08C1 4E dec esi 00AB08C2 74 73 je short 00AB0937 00AB08C4 EB E9 jmp short 00AB08AF 00AB08C6 8B75 10 mov esi,dword ptr ss:[ebp+10] 00AB08C9 8B7D 08 mov edi,dword ptr ss:[ebp+8] 00AB08CC 8B55 14 mov edx,dword ptr ss:[ebp+14] 00AB08CF 85D2 test edx,edx 00AB08D1 75 02 jnz short 00AB08D5 00AB08D3 8BD6 mov edx,esi 00AB08D5 85F6 test esi,esi 00AB08D7 75 02 jnz short 00AB08DB 00AB08D9 8BF2 mov esi,edx 00AB08DB C783 9E180010 00000>mov dword ptr ds:[ebx+1000189E],0 00AB08E5 8B02 mov eax,dword ptr ds:[edx] 00AB08E7 85C0 test eax,eax 00AB08E9 74 43 je short 00AB092E 00AB08EB 52 push edx 00AB08EC 8983 9E180010 mov dword ptr ds:[ebx+1000189E],eax 00AB08F2 A9 00000080 test eax,80000000 00AB08F7 74 0C je short 00AB0905 00AB08F9 25 FFFFFF7F and eax,7FFFFFFF 00AB08FE 68 00000000 push 0 00AB0903 EB 0E jmp short 00AB0913 00AB0905 8B4D 08 mov ecx,dword ptr ss:[ebp+8] 00AB0908 0341 08 add eax,dword ptr ds:[ecx+8] 00AB090B 33C9 xor ecx,ecx 00AB090D 66:8B08 mov cx,word ptr ds:[eax] 00AB0910 51 push ecx 00AB0911 40 inc eax 00AB0912 40 inc eax 00AB0913 50 push eax 00AB0914 FF75 FC push dword ptr ss:[ebp-4] 00AB0917 FF93 3E1B0010 call dword ptr ds:[ebx+10001B3E] 00AB091D 5A pop edx 00AB091E 85C0 test eax,eax 00AB0920 74 15 je short 00AB0937 00AB0922 8906 mov dword ptr ds:[esi],eax //修改为：mov eax,dword ptr ds:[edx] 00AB0924 8902 mov dword ptr ds:[edx],eax //修改为：mov dword ptr ds:[esi],eax //这样就自动还原 ThunkRVA 了 00AB0926 83C2 04 add edx,4 00AB0929 83C6 04 add esi,4 00AB092C EB AD jmp short 00AB08DB 00AB092E 33C0 xor eax,eax 00AB0930 5E pop esi 00AB0931 5F pop edi 00AB0932 5B pop ebx 00AB0933 C9 leave 00AB0934 C2 1000 retn 10`

修改之后就可以运行到 `00AB02F6` 处了。

代码:

```
00AB02F6 85C0 test eax,eax //输入表处理完毕
00AB02F8 74 7B je short 00AB0375
```

### 2. Dump 最佳时机

此时程序已经解密完毕，而壳还没有把输入表填充系统函数地址，正是 `dump` 的最佳时机！

运行 `LordPE` 完全 `dump` 目标进程，此时 `dump` 出来的文件不需要使用 `ImportREC` 修复输入表。

### 3. OEP 定位

`dump` 后直接按 `F4` 至 `00AB03B5` 处 `retn`。

代码:

```
00AB0375 8BDE mov ebx,esi
00AB0377 837B 48 01 cmp dword ptr ds:[ebx+48],1
00AB037B 75 16 jnz short 00AB0393
00AB037D 8B43 0C mov eax,dword ptr ds:[ebx+C]
00AB0380 8B4B 40 mov ecx,dword ptr ds:[ebx+40]
00AB0383 8BF1 mov esi,ecx
00AB0385 0373 08 add esi,dword ptr ds:[ebx+8]
00AB0388 C606 E9 mov byte ptr ds:[esi],0E9
00AB038B 83C1 05 add ecx,5
00AB038E 2BC1 sub eax,ecx
00AB0390 8946 01 mov dword ptr ds:[esi+1],eax
00AB0393 8BF3 mov esi,ebx
00AB0395 8B46 28 mov eax,dword ptr ds:[esi+28]
00AB0398 8B7E 08 mov edi,dword ptr ds:[esi+8]
00AB039B 03C7 add eax,edi
00AB039D 68 00800000 push 8000
00AB03A2 6A 00 push 0
00AB03A4 FFB5 421B0010 push dword ptr ss:[ebp+10001B42]
00AB03AA FF10 call dword ptr ds:[eax]
00AB03AC 8B46 0C mov eax,dword ptr ds:[esi+C]
00AB03AF 03C7 add eax,edi
00AB03B1 5D pop ebp
00AB03B2 5E pop esi
00AB03B3 5F pop edi
00AB03B4 5B pop ebx
00AB03B5 C3 retn //返回 005165C5，继续
```

### 4. 后续执行与返回

代码:

```
005165C5 8985 23120010 mov dword ptr ss:[ebp+10001223],eax
005165CB 8BF0 mov esi,eax
005165CD 59 pop ecx
005165CE 5A pop edx
005165CF 03CA add ecx,edx
005165D1 68 00800000 push 8000
005165D6 6A 00 push 0
005165D8 57 push edi
005165D9 FF11 call dword ptr ds:[ecx] ; kernel32.VirtualFree
005165DB 8BC6 mov eax,esi
005165DD 5A pop edx
005165DE 5E pop esi
005165DF 5F pop edi
005165E0 59 pop ecx
005165E1 5B pop ebx
005165E2 5D pop ebp
005165E3 FFE0 jmp eax ; Hard_Boo.00407674 //飞向光明之巅
```

### 5. 最终 OEP 修正

代码:

```
00407674 68 40404100 push 414040 //OEP
00407679 E8 EEFFFFFF call 0040766C ; Hard_Boo.0040766C
```

用 `LordPE` 修正 `dump.exe` 的 `OEP RVA=00007674` / `Import Table RVA=000E1914`。

脱壳完成。

### 六. CopyMinder 的亮点

`CopyMinder` 的 `Key` 校验部分采用了 `Int3` 来控制流程。

代码:

```
```

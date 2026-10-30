

```text
.text:3E586F6B push ebp
.text:3E586F6C mov ebp, esp
.text:3E586F6E mov eax, [ebp+pCElementObject]
.text:3E586F71 test eax, eax
.text:3E586F73 mov [ecx], eax
.text:3E586F75 jz short loc_3E586F7D
.text:3E586F75
.text:3E586F77 mov al, [eax+14h]
.text:3E586F7A mov [ecx+8], al
.text:3E586F7A
.text:3E586F7D
.text:3E586F7D loc_3E586F7D: ; CODE XREF: CTreeNode::SetElement(CElement *)+Cj
.text:3E586F7D pop ebp
.text:3E586F7E retn 4
.text:3E586F7E
.text:3E586F7E public: void __thiscall CTreeNode::SetElement(class CElement *) endp
.text:3E586F7E
```

步过以上部分，在此之后，网页代码使用`createEventObject`创建了`event`的副本，很囧的是，这个新的`CEventObj`类，当然copy了原来对象的`EVENTPARAM`结构，但是根据[1]的解释，在`EVENTPARAM`结构内容被copy的时候，并没有增加`CTreeNode`的访问计数！

`CEvent::Create`中调用`EVENTPARAM::EVENTPARAM`复制`EVENTPARAM`结构，但后者的代码中的确没有包括操作其中`CTreeNode`（在后面的`CEventObj::get_srcElement`函数分析中可以看出，`CTreeNode`类指针在`EVENTPARAM`结构的头部）增加访问计数的内容。

```text
.text:3E6B4BC9
.text:3E6B4BC9 loc_3E6B4BC9: ; CODE XREF: CEventObj::Create(IHTMLEventObj * *,CDoc *,CElement *,CMarkup *,int,ushort *,EVENTPARAM *,int)+84j
.text:3E6B4BC9 push 0D8h ; dwBytes
.text:3E6B4BCE call _MemAlloc(x)
.text:3E6B4BCE
.text:3E6B4BD3 test eax, eax
.text:3E6B4BD5 jz short loc_3E6B4BE3
.text:3E6B4BD5
.text:3E6B4BD7 push [ebp+arg_18]
.text:3E6B4BDA mov ecx, eax
.text:3E6B4BDC call EVENTPARAM::EVENTPARAM(EVENTPARAM const *)
.text:3E6B4BDC
.text:3E6B4BE1 jmp short loc_3E6B4BE5
.text:3E6B4BE1

.text:3E582A41
.text:3E582A41 public: __thiscall EVENTPARAM::EVENTPARAM(struct EVENTPARAM const *) proc near
.text:3E582A41 ; CODE XREF: CElement::get_nodeType(long *)-164050p
.text:3E582A41 ; .text:3E5BFDC3p
.text:3E582A41 ; .text:3E5C2363p
.text:3E582A41 ; CElement::FireStdEvent_KeyHelper(CTreeNode *,CMessage *,int *,EVENTINFO *)+7Ap
.text:3E582A41 ; CElement::get_nodeType(long *)-7B645p
.text:3E582A41 ; CElement::get_nodeType(long *)+5FEBp ...
.text:3E582A41
.text:3E582A41 pEventParam = dword ptr 8
.text:3E582A41
.text:3E582A41 mov edi, edi
.text:3E582A43 push ebp
.text:3E582A44 mov ebp, esp
.text:3E582A46 push ebx
.text:3E582A47 mov ebx, ecx
.text:3E582A49 push esi
.text:3E582A4A mov esi, [ebp+pEventParam]
.text:3E582A4D push edi
.text:3E582A4E xor eax, eax
.text:3E582A50 lea edi, [ebx+48h]
.text:3E582A53 stosd
.text:3E582A54 stosd
.text:3E582A55 stosd
.text:3E582A56 stosd
.text:3E582A57 xor edx, edx
.text:3E582A59 mov [ebx+0C4h], edx
.text:3E582A5F mov [ebx+0C8h], edx
.text:3E582A65 mov [ebx+0CCh], edx
.text:3E582A6B mov [ebx+0D0h], edx
.text:3E582A71 push 36h
.text:3E582A73 pop ecx
.text:3E582A74 mov edi, ebx
.text:3E582A76 rep movsd ; 直接copy了内容
.text:3E582A78 mov eax, [ebx+64h]
.text:3E582A7B and dword ptr [ebx+0A8h], 0FFFFFBFFh
.text:3E582A85 add dword ptr [eax+8], 8
.text:3E582A89 mov eax, [ebx+68h]
.text:3E582A8C cmp eax, edx
.text:3E582A8E jz short loc_3E582A94
.text:3E582A8E
.text:3E582A90 add dword ptr [eax+8], 8
.text:3E582A90
.text:3E582A94
.text:3E582A94 loc_3E582A94: ; CODE XREF: EVENTPARAM::EVENTPARAM(EVENTPARAM const *)+4Dj
.text:3E582A94 mov eax, [ebx+6Ch]
.text:3E582A97 cmp eax, edx
.text:3E582A99 jnz loc_3E6A4170
.text:3E582A99
.text:3E582A9F
.text:3E582A9F loc_3E582A9F: ; CODE XREF: CElement::get_nodeType(long *)+13875j
.text:3E582A9F mov eax, [ebx+84h]
.text:3E582AA5 cmp eax, edx
.text:3E582AA7 jnz loc_3E6A4179
.text:3E582AA7
.text:3E582AAD jmp loc_3E6A4181
.text:3E582AAD
.text:3E582AAD public: __thiscall EVENTPARAM::EVENTPARAM(struct EVENTPARAM const *) endp
.text:3E582AAD
.text:3E582AB2
.text:3E582AB2 loc_3E582AB2: ; CODE XREF: CElement::get_nodeType(long *)+138EEj
.text:3E582AB2 pop edi
.text:3E582AB3 pop esi
.text:3E582AB4 mov eax, ebx
.text:3E582AB6 pop ebx
.text:3E582AB7 pop ebp
.text:3E582AB8 retn 4

.text:3E6A4170
.text:3E6A4170 loc_3E6A4170: ; CODE XREF: EVENTPARAM::EVENTPARAM(EVENTPARAM const *)+58j
.text:3E6A4170 add dword ptr [eax+8], 8
.text:3E6A4174 jmp loc_3E582A9F
.text:3E6A4174
.text:3E6A4179 ; ---------------------------------------------------------------------------
.text:3E6A4179
.text:3E6A4179 loc_3E6A4179: ; CODE XREF: EVENTPARAM::EVENTPARAM(EVENTPARAM const *)+66j
.text:3E6A4179 mov ecx, [eax]
.text:3E6A417B push eax
.text:3E6A417C call dword ptr [ecx+4]
.text:3E6A417F xor edx, edx
.text:3E6A417F
.text:3E6A4181
.text:3E6A4181 loc_3E6A4181: ; CODE XREF: EVENTPARAM::EVENTPARAM(EVENTPARAM const *)+6Cj
.text:3E6A4181 mov eax, [ebp+lpMem]
.text:3E6A4184 add eax, 0C4h
.text:3E6A4189 lea ecx, [ebx+0C4h]
.text:3E6A418F push eax
.text:3E6A4190 mov [ecx], edx
.text:3E6A4192 call CStr::Set(CStr const &)
.text:3E6A4192
.text:3E6A4197 mov esi, [ebp+lpMem]
.text:3E6A419A lea eax, [esi+0C8h]
.text:3E6A41A0 lea ecx, [ebx+0C8h]
.text:3E6A41A6 xor edi, edi
.text:3E6A41A8 push eax
.text:3E6A41A9 mov [ecx], edi
.text:3E6A41AB call CStr::Set(CStr const &)
.text:3E6A41AB
.text:3E6A41B0 lea eax, [esi+0CCh]
.text:3E6A41B6 lea ecx, [ebx+0CCh]
.text:3E6A41BC push eax
.text:3E6A41BD mov [ecx], edi
.text:3E6A41BF call CStr::Set(CStr const &)
.text:3E6A41BF
.text:3E6A41C4 lea eax, [esi+0D0h]
.text:3E6A41CA lea ecx, [ebx+0D0h]
.text:3E6A41D0 push eax
.text:3E6A41D1 mov [ecx], edi
.text:3E6A41D3 call CStr::Set(CStr const &)
.text:3E6A41D3
.text:3E6A41D8 lea ecx, [ebx+48h]
.text:3E6A41DB xor eax, eax
.text:3E6A41DD mov edi, ecx
.text:3E6A41DF stosd
.text:3E6A41E0 stosd
.text:3E6A41E1 add esi, 48h
.text:3E6A41E4 stosd
.text:3E6A41E5 push esi ; pvargSrc
.text:3E6A41E6 push ecx ; pvargDest
.text:3E6A41E7 stosd
.text:3E6A41E8 call VariantCopy(x,x)
.text:3E6A41E8
.text:3E6A41ED jmp loc_3E582AB2
.text:3E6A41ED
```

于是通过调用`document.getElementById("sp1").innerHTML = ""`，导致`CTreeNode`和`CImgElement`的释放，看看`CTreeNode`开头处的`CImgElement`类指针是什么时候被改掉的，对`CTreeNode`类头部下硬件写入断点：

```text
0:005> ba w 4 02b60880
0:005> g
Breakpoint 2 hit
eax=feeefeee ebx=00000000 ecx=00000011 edx=02b60360 esi=02b60360 edi=02b60884
eip=7c922c53 esp=019df8cc ebp=019df9b4 iopl=0 nv up ei pl nz ac pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00010216
ntdll!RtlFillMemoryUlong+0x10:
7c922c53 f3ab rep stos dword ptr es:[edi]
0:005> kb
ChildEBP RetAddr Args to Child
019df8cc 7c96be06 02b60370 00000558 feeefeee ntdll!RtlFillMemoryUlong+0x10
019df9b4 7c98f7fa 00150000 50000061 02b60880 ntdll!RtlFreeHeapSlowly+0x2ea
019dfa28 7c96bc5a 00150000 50000061 02b60880 ntdll!RtlDebugFreeHeap+0x193
019dfb10 7c946045 00150000 40000060 02b60880 ntdll!RtlFreeHeapSlowly+0x37
019dfbe0 3e5b2038 00150000 00000000 02b60880 ntdll!RtlFreeHeap+0xf9
019dfbf4 3e7179d2 02b60880 3e593ed1 02b60880 mshtml!_MemFree+0x1c
019dfbfc 3e593ed1 02b60880 001dbff0 019dfc1c mshtml!CTreeNode::Release+0xe
019dfc0c 3e5b592b 02b60880 001dbff0 019dfd48 mshtml!CTreeNode::RemoveRef+0x22
019dfc1c 3e5c9a5c 001dbff0 001c8ae8 002033d0 mshtml!PlainRelease+0x33
019dfc2c 3e5d65bf 001c8ae8 001db6c0 00000000 mshtml!CTreeNode::NodeRelease+0x24
019dfd48 3e5a358a 000003eb 00000001 00000000 mshtml!CElement::FireEvent+0x36f
019dfd64 3e5a0fbb 001db6c0 019dfdd8 3e5a34ff mshtml!CImgElement::Fire_onload+0x13
019dfd70 3e5a34ff 00000004 001c7fb0 019dfe1c mshtml!CImgHelper::SetReadyStateImg+0x5c
019dfdd8 3e5a355e 001f6820 019dfdf4 3e59f3ff mshtml!CImgHelper::OnDwnChan+0x436
019dfde4 3e59f3ff 001f6820 001db6c0 019dfe28 mshtml!CImgHelper::OnDwnChanCallback+0x10
019dfdf4 3e5b79e2 001f6820 00000000 00000000 mshtml!CDwnChan::OnMethodCall+0x19
019dfe28 3e5b1602 019dfeb0 3e5b1551 00000000 mshtml!GlobalWndOnMethodCall+0x101
019dfe48 77d18734 000e021c 000000d4 00000000 mshtml!GlobalWndProc+0x181
```


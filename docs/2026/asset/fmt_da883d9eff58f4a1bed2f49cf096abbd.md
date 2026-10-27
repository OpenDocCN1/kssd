
```c
kd> uf PspSetCreateProcessNotifyRoutine
nt!PspSetCreateProcessNotifyRoutine:
83fd87f9 8bff mov edi,edi
83fd87fb 55 push ebp
83fd87fc 8bec mov ebp,esp
83fd87fe 807d0c00 cmp byte ptr [ebp+0Ch],0 //判断是设置回调还是清除回调.
83fd8802 53 push ebx
83fd8803 56 push esi //保存寄存器环境.
83fd8804 57 push edi
83fd8805 0f84fa000000 je nt!PspSetCreateProcessNotifyRoutine+0x10a (83fd8905) 是否删除回调 == 0 跳转.也就是设置回调. 删除回调走上边.
====================> 删除回调走的位置
nt!PspSetCreateProcessNotifyRoutine+0x12:
83fd880b 648b3524010000 mov esi,dword ptr fs:[124h]
83fd8812 66ff8e84000000 dec word ptr [esi+84h]
83fd8819 33db xor ebx,ebx
83fd881b c7450ce06cf883 mov dword ptr [ebp+0Ch],offset nt!PspCreateProcessNotifyRoutine (83f86ce0) //获得回调函数数组地址.
nt!PspSetCreateProcessNotifyRoutine+0x29:
83fd8822 ff750c push dword ptr [ebp+0Ch] //数组地址压栈.
83fd8825 e88ea90d00 call nt!ExReferenceCallBackBlock (840b31b8) //逆向ExReferenceCallBackBlock
83fd882a 8bf8 mov edi,eax
83fd882c 85ff test edi,edi
83fd882e 7439 je nt!PspSetCreateProcessNotifyRoutine+0x6e (83fd8869)
nt!PspSetCreateProcessNotifyRoutine+0x37:
83fd8830 8b4f08 mov ecx,dword ptr [edi+8]
83fd8833 e81b3cf2ff call nt!ExGetCallBackBlockRoutine (83efc453)
83fd8838 3b4508 cmp eax,dword ptr [ebp+8]
83fd883b 7522 jne nt!PspSetCreateProcessNotifyRoutine+0x64 (83fd885f)
nt!PspSetCreateProcessNotifyRoutine+0x44:
83fd883d 85c9 test ecx,ecx
83fd883f 7509 jne nt!PspSetCreateProcessNotifyRoutine+0x4f (83fd884a)
nt!PspSetCreateProcessNotifyRoutine+0x48:
83fd8841 384d10 cmp byte ptr [ebp+10h],cl
83fd8844 7519 jne nt!PspSetCreateProcessNotifyRoutine+0x64 (83fd885f)
nt!PspSetCreateProcessNotifyRoutine+0x4d:
83fd8846 eb08 jmp nt!PspSetCreateProcessNotifyRoutine+0x55 (83fd8850)
nt!PspSetCreateProcessNotifyRoutine+0x4f:
83fd884a 807d1000 cmp byte ptr [ebp+10h],0
83fd884e 740f je nt!PspSetCreateProcessNotifyRoutine+0x64 (83fd885f)
nt!PspSetCreateProcessNotifyRoutine+0x55:
83fd8850 8b450c mov eax,dword ptr [ebp+0Ch]
83fd8853 57 push edi
83fd8854 33c9 xor ecx,ecx
83fd8856 e8abfeffff call nt!ExCompareExchangeCallBack (83fd8706)
83fd885b 84c0 test al,al
83fd885d 7547 jne nt!PspSetCreateProcessNotifyRoutine+0xab (83fd88a6)
nt!PspSetCreateProcessNotifyRoutine+0x64:
83fd885f 8b450c mov eax,dword ptr [ebp+0Ch]
83fd8862 8bcf mov ecx,edi
83fd8864 e883aa0d00 call nt!ExDereferenceCallBackBlock (840b32ec)
nt!PspSetCreateProcessNotifyRoutine+0x6e:
83fd8869 83450c04 add dword ptr [ebp+0Ch],4
83fd886d 43 inc ebx
83fd886e 83fb40 cmp ebx,40h
83fd8871 72af jb nt!PspSetCreateProcessNotifyRoutine+0x29 (83fd8822)
nt!PspSetCreateProcessNotifyRoutine+0x78:
83fd8873 66ff8684000000 inc word ptr [esi+84h]
83fd887a 0fb78684000000 movzx eax,word ptr [esi+84h]
83fd8881 6685c0 test ax,ax
83fd8884 7516 jne nt!PspSetCreateProcessNotifyRoutine+0xa1 (83fd889c)
nt!PspSetCreateProcessNotifyRoutine+0x8b:
83fd8886 8d4640 lea eax,[esi+40h]
83fd8889 3900 cmp dword ptr [eax],eax
83fd888b 740f je nt!PspSetCreateProcessNotifyRoutine+0xa1 (83fd889c)
nt!PspSetCreateProcessNotifyRoutine+0x92:
83fd888d 6683be8600000000 cmp word ptr [esi+86h],0
83fd8895 7505 jne nt!PspSetCreateProcessNotifyRoutine+0xa1 (83fd889c)
nt!PspSetCreateProcessNotifyRoutine+0x9c:
83fd8897 e8fd3be9ff call nt!KiCheckForKernelApcDelivery (83e6c499)
nt!PspSetCreateProcessNotifyRoutine+0xa1:
83fd889c b87a0000c0 mov eax,0C000007Ah
83fd88a1 e9c5000000 jmp nt!PspSetCreateProcessNotifyRoutine+0x170 (83fd896b)
nt!PspSetCreateProcessNotifyRoutine+0xab:
83fd88a6 83c9ff or ecx,0FFFFFFFFh
83fd88a9 807d1000 cmp byte ptr [ebp+10h],0
83fd88ad b8e46df883 mov eax,offset nt!PspCreateProcessNotifyRoutineCount (83f86de4)
83fd88b2 7405 je nt!PspSetCreateProcessNotifyRoutine+0xbe (83fd88b9)
nt!PspSetCreateProcessNotifyRoutine+0xb9:
83fd88b4 b8e06df883 mov eax,offset nt!PspCreateProcessNotifyRoutineExCount (83f86de0)
nt!PspSetCreateProcessNotifyRoutine+0xbe:
83fd88b9 f00fc108 lock xadd dword ptr [eax],ecx
83fd88bd 8d049de06cf883 lea eax,nt!PspCreateProcessNotifyRoutine (83f86ce0)[ebx*4]
83fd88c4 8bcf mov ecx,edi
83fd88c6 e821aa0d00 call nt!ExDereferenceCallBackBlock (840b32ec)
83fd88cb 66ff8684000000 inc word ptr [esi+84h]
83fd88d2 0fb78684000000 movzx eax,word ptr [esi+84h]
83fd88d9 6685c0 test ax,ax
83fd88dc 7516 jne nt!PspSetCreateProcessNotifyRoutine+0xf9 (83fd88f4)
nt!PspSetCreateProcessNotifyRoutine+0xe3:
83fd88de 8d4640 lea eax,[esi+40h]
83fd88e1 3900 cmp dword ptr [eax],eax
83fd88e3 740f je nt!PspSetCreateProcessNotifyRoutine+0xf9 (83fd88f4)
nt!PspSetCreateProcessNotifyRoutine+0xea:
83fd88e5 6683be8600000000 cmp word ptr [esi+86h],0
83fd88ed 7505 jne nt!PspSetCreateProcessNotifyRoutine+0xf9 (83fd88f4)
nt!PspSetCreateProcessNotifyRoutine+0xf4:
83fd88ef e8a53be9ff call nt!KiCheckForKernelApcDelivery (83e6c499)
nt!PspSetCreateProcessNotifyRoutine+0xf9:
83fd88f4 8bcf mov ecx,edi
83fd88f6 e890f41600 call nt!ExWaitForCallBacks (84147d8b)
83fd88fb 57 push edi
83fd88fc e814780e00 call nt!SepFreeCapturedString (840c0115)
nt!PspSetCreateProcessNotifyRoutine+0x106:
83fd8901 33c0 xor eax,eax
83fd8903 eb66 jmp nt!PspSetCreateProcessNotifyRoutine+0x170 (83fd896b)
// 设置回调会走的路线.
nt!PspSetCreateProcessNotifyRoutine+0x10a:
83fd8905 807d1000 cmp byte ptr [ebp+10h],0 //判断第三个参数是否为0
83fd8909 7413 je nt!PspSetCreateProcessNotifyRoutine+0x123 (83fd891e)
nt!PspSetCreateProcessNotifyRoutine+0x110:
83fd890b ff7508 push dword ptr [ebp+8]
83fd890e e856affdff call nt!MmVerifyCallbackFunction (83fb3869)
83fd8913 85c0 test eax,eax
83fd8915 7507 jne nt!PspSetCreateProcessNotifyRoutine+0x123 (83fd891e)
nt!PspSetCreateProcessNotifyRoutine+0x11c:
83fd8917 b8220000c0 mov eax,0C0000022h
83fd891c eb4d jmp nt!PspSetCreateProcessNotifyRoutine+0x170 (83fd896b)
//第三个参数是默认Push 0.会到这里.
nt!PspSetCreateProcessNotifyRoutine+0x123:
83fd891e 33c0 xor eax,eax
83fd8920 384510 cmp byte ptr [ebp+10h],al
83fd8923 0f95c0 setne al // == 0 则设置al位. 如果比较滞后 ==0 相等.(zf=1) 则设置al位0 .如果不相等则al = 1.
83fd8926 50 push eax 0
83fd8927 ff7508 push dword ptr [ebp+8] 回调函数地址.
83fd892a e8a8fdffff call nt!ExAllocateCallBack (83fd86d7)
83fd892f 8bd8 mov ebx,eax
83fd8931 85db test ebx,ebx //判断ebx结果是否是0. 不是0 跳转.
83fd8933 7507 jne nt!PspSetCreateProcessNotifyRoutine+0x141 (83fd893c)
nt!PspSetCreateProcessNotifyRoutine+0x13a:
83fd8935 b89a0000c0 mov eax,0C000009Ah
83fd893a eb2f jmp nt!PspSetCreateProcessNotifyRoutine+0x170 (83fd896b)
nt!PspSetCreateProcessNotifyRoutine+0x141:
83fd893c bee06cf883 mov esi,offset nt!PspCreateProcessNotifyRoutine (83f86ce0) //不是0跳转到此位置. 获取数组.
83fd8941 33ff xor edi,edi
nt!PspSetCreateProcessNotifyRoutine+0x148:
83fd8943 6a00 push 0
83fd8945 8bcb mov ecx,ebx ecx = ExAllocateCallBack 返回值.
83fd8947 8bc6 mov eax,esi eax = 数组
83fd8949 e8b8fdffff call nt!ExCompareExchangeCallBack (83fd8706) //应该是遍历数组.进行比较.设置回调.
83fd894e 84c0 test al,al
83fd8950 7520 jne nt!PspSetCreateProcessNotifyRoutine+0x177 (83fd8972)
nt!PspSetCreateProcessNotifyRoutine+0x157:
83fd8952 83c704 add edi,4
83fd8955 83c604 add esi,4
83fd8958 81ff00010000 cmp edi,100h
83fd895e 72e3 jb nt!PspSetCreateProcessNotifyRoutine+0x148 (83fd8943)
nt!PspSetCreateProcessNotifyRoutine+0x165:
83fd8960 53 push ebx
83fd8961 e8af770e00 call nt!SepFreeCapturedString (840c0115)
83fd8966 b80d0000c0 mov eax,0C000000Dh
nt!PspSetCreateProcessNotifyRoutine+0x170:
83fd896b 5f pop edi
83fd896c 5e pop esi
83fd896d 5b pop ebx
83fd896e 5d pop ebp
83fd896f c20c00 ret 0Ch
nt!PspSetCreateProcessNotifyRoutine+0x177:
83fd8972 33c9 xor ecx,ecx
83fd8974 41 inc ecx
83fd8975 807d1000 cmp byte ptr [ebp+10h],0
83fd8979 7525 jne nt!PspSetCreateProcessNotifyRoutine+0x1a5 (83fd89a0)
nt!PspSetCreateProcessNotifyRoutine+0x180:
83fd897b b8e46df883 mov eax,offset nt!PspCreateProcessNotifyRoutineCount (83f86de4)
83fd8980 f00fc108 lock xadd dword ptr [eax],ecx
83fd8984 a1786bf883 mov eax,dword ptr [nt!PspNotifyEnableMask (83f86b78)]
83fd8989 a802 test al,2
83fd898b 0f8570ffffff jne nt!PspSetCreateProcessNotifyRoutine+0x106 (83fd8901)
nt!PspSetCreateProcessNotifyRoutine+0x196:
83fd8991 b8786bf883 mov eax,offset nt!PspNotifyEnableMask (83f86b78)
83fd8996 f00fba2801 lock bts dword ptr [eax],1
83fd899b e961ffffff jmp nt!PspSetCreateProcessNotifyRoutine+0x106 (83fd8901)
nt!PspSetCreateProcessNotifyRoutine+0x1a5:
83fd89a0 b8e06df883 mov eax,offset nt!PspCreateProcessNotifyRoutineExCount (83f86de0)
83fd89a5 f00fc108 lock xadd dword ptr [eax],ecx
83fd89a9 a1786bf883 mov eax,dword ptr [nt!PspNotifyEnableMask (83f86b78)]
83fd89ae a804 test al,4
83fd89b0 0f854bffffff jne nt!PspSetCreateProcessNotifyRoutine+0x106 (83fd8901)
nt!PspSetCreateProcessNotifyRoutine+0x1bb:
83fd89b6 b8786bf883 mov eax,offset nt!PspNotifyEnableMask (83f86b78)
83fd89bb f00fba2802 lock bts dword ptr [eax],2
83fd89c0 e93cffffff jmp nt!PspSetCreateProcessNotifyRoutine+0x106 (83fd8901)
<===================================================================================>
```

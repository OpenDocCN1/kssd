
在`windbg`验证结论,共享内存`CPerContext`的地址为`0617a350`,可见它继承与`CSmAllocator`,它的`CPerContext->CSharedMemoryBlockObj(+0x4)`也就是`CSharedMemoryBlock`地址为`060def40`,`CSharedMemoryBlock->_pbBase`地址为`03e40000`,同时可以看到`ReservedForOle`指针指向的地址为`03e40008`,验证了结论.

```
0:018> dt ntdll!\_TEB ReservedForOle @$teb
+0xf80 ReservedForOle : 0x031a9e98 Void
0:018> dc 0x031a9e98
031a9e98 03e40008 0617a350 00000000 00001002 ....P...........
031a9ea8 0000000c 02f86f80 00000000 00000000 .....o..........
031a9eb8 00000001 03112648 0506e850 00000000 ....H&..P.......
031a9ec8 00000000 00002f4c 03109f80 03109f80 ....L/..........
031a9ed8 00000000 00000000 0000013c 00000000 ........<.......
031a9ee8 00000000 0313c658 00000000 00000000 ....X...........
031a9ef8 00000000 00000000 00000000 00000000 ................
031a9f08 00000000 00000000 00000000 00000000 ................
0:018> dps 0x031a9e98
//ReservedForOle指针指向的地址=CSharedMemoryBlock->_pbBase+8
031a9e98 03e40008
//CPerContext的地址
031a9e9c 0617a350
031a9ea0 00000000
031a9ea4 00001002
031a9ea8 0000000c
031a9eac 02f86f80
...
//CPerContext的地址为0617a350
0:018> dps 0617a350
0617a350 750c11a8 coml2!CSmAllocator::`vftable'
//CPerContext->_psmb也就是CSharedMemoryBlock地址
0617a354 060def40
//NtCurrentTeb()->ReservedForOle赋值成CPerContext->_pbBase
0617a358 03e40008
....
//CSharedMemoryBlock地址
0:018> dps 060def40
060def40 000010b8
//CSharedMemoryBlock->_pbBase地址
060def44 03e40000
...
```

设置好共享内存后会进入`UnmarshalContext`,其中先从`SDfMarshalPacket->CBasedGlobalContextObj`中通过`CContextList::_Find`找第一个`CGlobalContext`,其实`CGlobalContext`继承于`CContextList`结构,会根据`CContextList->pctxNext`找下一个`CContextList`直到`CContextList->ctxid`为要找的进程`id`为止.之后会验证下`CPerContext`句柄是否有效,其实只要构造一个名为`"\\Sessions\\0\\BaseNamedObjects\\OleDfRoot%X%08lX"`的内核`Event`通过`NtCreateEvent`,Bits服务的`Sessions`为`0`,详见`poc`代码.

```
SDfMarshalPacket * __userpurge UnmarshalContext@<eax>(CPerContext **a1@<edx>, SDfMarshalPacket *pckfrom@<ecx>, struct SDfMarshalPacket *pvkOrg, struct CPerContext **a4, unsigned int a5, int a6, int a7)
{
SDfMarshalPacket *pck; // edi
CGlobalContext *pCGlobalContextOffset; // ecx
CGlobalContext *pCGlobalContext; // esi
unsigned int NowPid; // eax
CPerContext *pCPerContextFound; // eax
CContext *pContext; // ebx
struct _GUID v13; // ST04_16
struct CPerContext **v14; // ecx
CPerContext *v15; // eax
CSmAllocator *v17; // eax
unsigned int NowPidNext; // eax
int CBasedGlobalFileStreamBaseObjOffset; // edx
CGlobalFileStream *pCBasedGlobalFileStreamBase; // ecx
int v21; // eax
char *v22; // edx
int v23; // esi
CGlobalFileStream *fstmFromGlobal; // ecx
SDfMarshalPacket *v25; // eax
char *v26; // edx
CGlobalFileStream *v27; // ecx
SDfMarshalPacket *v28; // eax
struct ILockBytes *v29; // edi
struct CFileStream *v30; // ebx
struct ILockBytes *v31; // esi
unsigned int v32; // edx
void **v33; // [esp+0h] [ebp-44h]
void **v34; // [esp+0h] [ebp-44h]
void **v35; // [esp+0h] [ebp-44h]
struct IMalloc *v36; // [esp+4h] [ebp-40h]
unsigned int v37; // [esp+4h] [ebp-40h]
unsigned int *v38; // [esp+4h] [ebp-40h]
CPerContext **v39; // [esp+10h] [ebp-34h]
CPerContext *v40; // [esp+14h] [ebp-30h]
int v41; // [esp+18h] [ebp-2Ch]
unsigned int v42; // [esp+1Ch] [ebp-28h]
BOOL pCPerContextFoundRef; // [esp+20h] [ebp-24h]
CGlobalContext *pCGlobalContextRef; // [esp+24h] [ebp-20h]
struct CFileStream *v45; // [esp+28h] [ebp-1Ch]
struct ILockBytes *ppvRet; // [esp+2Ch] [ebp-18h]
struct ILockBytes *v47; // [esp+30h] [ebp-14h]
struct CFileStream *v48; // [esp+34h] [ebp-10h]
struct ILockBytes *v49; // [esp+38h] [ebp-Ch]
SDfMarshalPacket *pckRef; // [esp+3Ch] [ebp-8h]
v39 = a1;
v42 = 0;
pck = pckfrom;
v41 = 0;
pckRef = pckfrom;
ppvRet = 0;
v48 = 0;
pCGlobalContextOffset = pckfrom->CBasedGlobalContextObj;
v45 = 0;
v49 = 0;
v47 = 0;
v40 = 0;
if ( pCGlobalContextOffset )
{
pCGlobalContext = (pCGlobalContextOffset + *NtCurrentTeb()->ReservedForOle);
v41 = ppvRet;
v48 = v45;
v49 = v47;
}
else
{
pCGlobalContext = 0;
}
pCGlobalContextRef = pCGlobalContext;
NowPid = GetCurrentProcessId();
// 找第一个PerContext
pCPerContextFound = CContextList::_Find(&pCGlobalContext->CContextListbase, NowPid);
pContext = &pCPerContextFound->baseclass_CContext;
pCPerContextFoundRef = !pCPerContextFound
// 先验证下找到的PerContext必须成功
|| (pContext = (CPerContext::IsHandleValid(pCPerContextFound) != 0 ? pCPerContextFound : 0)) == 0;
// 如果不是当前进程检查CProcessSecret,不相符也没关系
if ( GetCurrentProcessId() != pck->ProcessContextId )
{
* &v13.Data1 = * &pck->cntxkey.Data1;
*v13.Data4 = *pck->cntxkey.Data4;
if ( CProcessSecret::VerifyMatchingSecret(v13) < 0 && pContext )
// 不相符也没关系只是不设置状态
CPerContext::SetThreadAllocatorState(pContext, v14);
pCGlobalContext = pCGlobalContextRef;
pck = pckRef;
}
if ( !pCPerContextFoundRef )
{
LABEL_21:
v17 = GetTlsSmAllocator();
CSmAllocator::SetState(v17, pContext[5].pctxNext, pContext[6].ctxid, pContext[6].pctxNext, 0, pContext);
NowPidNext = GetCurrentProcessId();
// 继续找下一个PerContext
pContext = CContextList::_Find(&pCGlobalContext->CContextListbase, NowPidNext);
if ( !pContext )
{
pckRef = 0x800300FD;
goto LABEL_23;
}
CBasedGlobalFileStreamBaseObjOffset = pck->CBasedGlobalFileStreamBaseObj;
if ( CBasedGlobalFileStreamBaseObjOffset )
{
pCBasedGlobalFileStreamBase = (CBasedGlobalFileStreamBaseObjOffset + *NtCurrentTeb()->ReservedForOle);
v49 = v47;
}
else
{
pCBasedGlobalFileStreamBase = 0;
}
// 进入关键步骤
hr = CFileStream::Unmarshal(&ppvRet, pCBasedGlobalFileStreamBase, pCBasedGlobalFileStreamBase, v33, v36);
....
}
struct CContext * __thiscall CContextList::_Find(CContextList *this, unsigned int pid)
{
CContext *HeadCtx; // eax
char *HeadCtxFoundRet; // edx
CContext *v4; // ecx
CContext *HeadCtxlookup; // ecx
CContext *v6; // ecx
CContext *HeadCtxFound; // ecx
HeadCtx = this->_pctxHead;
HeadCtxFoundRet = 0;
while ( 1 )
{
v4 = HeadCtx ? (HeadCtx + *NtCurrentTeb()->ReservedForOle) : 0;
if ( !v4 )
break;
HeadCtxlookup = HeadCtx ? (HeadCtx + *NtCurrentTeb()->ReservedForOle) : 0;
if ( HeadCtxlookup->ctxid )
{
v6 = HeadCtx ? (HeadCtx + *NtCurrentTeb()->ReservedForOle) : 0;
if ( v6->ctxid == pid )
break;
}
if ( HeadCtx )
HeadCtxFound = (HeadCtx + *NtCurrentTeb()->ReservedForOle);
else
HeadCtxFound = 0;
// 继续找下一个
HeadCtx = HeadCtxFound->pctxNext;
}
if ( HeadCtx )
HeadCtxFoundRet = HeadCtx + *NtCurrentTeb()->ReservedForOle;
return HeadCtxFoundRet;
}
int __thiscall CPerContext::IsHandleValid(CPerContext *this)
{
CPerContext *that; // esi
int result; // eax
wchar_t Dest; // [esp+8h] [ebp-48h]
that = this;
StringCchPrintfW(
&Dest,
0x20u,
L"OleDfRoot%X%08lX",
this->CGlobalContextPtr->_luidMutexName.HighPart,
this->CGlobalContextPtr->_luidMutexName.LowPart);
// 在poc中使用NtCreateEvent使它验证成功
result = CDfMutex::IsHandleValid(&that->_dmtx, &Dest);
if ( !result )
// 失败就返回0
that->baseclass_CContext.ctxid = 0;
return result;
}
```

之后进入 `CFileStream::Unmarshal`,同样通过`CContextList::_Find`和方式通过设置和`bits`服务相同的`pid`从`SDfMarshalPacket->CBasedGlobalFileStreamBaseObj`找第一个`CFileStream`也就是`poc`中往共享内存`0x7279 - 0x10`写入的`CFileStream`,它的`CFileStream->_hFile`被赋值为`0`导致`GetFileType`返回`-1`导致验证失败(不影响程序运行),这个时候它的`ctxid`已经可以检测到为`0`,这时会进入`poc`中新建线程的第一个`if`语句断点.由于验证失败进入`else`,其中`fstmOut->_CGlobalFileStreamPtr`被赋值成`SDfMarshalPacket->CBasedGlobalFileStreamBaseObj`, `fstmGlobal`赋值成`SDfMarshalPacket->CGlobalFileStreamPtr`,之后它的`fstmGlobal->_CGlobalFileStreamPtr->_pctxHead=0x7279`被 `CContextList::Add`设置成`fstmGlobal`的相对地址,可以去判断读取`SDfMarshalPacket->CBasedGlobalFileStreamBaseObj->_pctxHead`已经不是`0x7279`,其实是`fstmGlobal`的相对地址.笔者发现一个小`tips`, `fstmGlobal->_CGlobalFileStreamPtr`也是可以在`SDfMarshalPacket`中读取到的,这样实际上求`fstmGlobal->_CGlobalFileStreamPtr - SDfMarshalPacket->CBasedGlobalFileStreamBaseObj`值就可以计算出`bits`进程的映射共享内存的映射基址也就是(`NtCurrentTeb()->ReservedForOle->pvThreadBase - 8`)的值,从而实现读写`bits`进程的任意指定真实地址(非通过偏移内存计算)内存数据在共享内存区域,有兴趣的读者可以自行尝试后续研究.

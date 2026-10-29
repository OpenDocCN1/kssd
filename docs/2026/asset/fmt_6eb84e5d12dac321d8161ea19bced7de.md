
# [原创]CVE-2015-2370之DCOM DCE/RPC协议原理详细分析

## 漏洞成因分析

这个`CVE-2015-2370`漏洞是一种`DCOM` `DCE/RPC`协议中`ntlm`认证后数据包重放导致的权限提升漏洞

分析的重点`DCOM` `DCE/RPC`协议原理,这个协议主要由2块内容组成,`dcom`的远程激活机制和`ntlm`身份认证

### 1.dcom的远程激活机制

[微软官方解释](<mark class="encrypted">a63K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6U0j5K6t1J5y4U0R3H3x3g2)9J5k6h3q4K6M7s2R3`.</mark> "微软官方解释")

有一个运行在`135`端口的`rpcss`服务也就是`dcom`的激活服务负责协调本机所有`com`对象的激活,当本机激活时采用通过内核通信,无法捕获数据包,属于内部操作,只有当远程激活或远程重定向到本机激活这种方式才可以捕获到数据包.远程激活有2种方式,一种是采用`CoCreateInstanceEx`方式指定远程服务器和激活身份等参数调用`rpscss`的`IRemoteSCMActivator`接口的`RemoteCreateInstance`方法激活,或者`CoGetClassObject` =调用`rpscss`的`IRemoteSCMActivator`接口的`RemoteGetClassObject`方法激活同样可选指定远程服务器和激活身份等参数.还有一种方式方式是客户端`marshal`服务端`unmarshal`方式,在`marshal`的`stream`中写入`[OBJREF](<mark class="encrypted">236K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6U0j5K6t1J5y4U0R3J5z5q4)9J5k6h3q4K6M7s2R3`.</mark> "OBJREF")通过其中的`DUALSTRINGARRAY`字段指定远程解析的服务器或端口,远程服务器`rpcss`服务采用`IObjectExporter`接口中的`ResolveOxid`或`ResolveOxid2`方法实现反序列化出来需要`unmarshal`的远程`com`对象`remunkown`指针.`CVE-2015-2370`是通过在`ntlm`身份认证后在`ResolveOxid2`之后又重放了一个`RemoteCreateInstance`请求导致以客户端的高权限创建出来一个`OLE Packager`的文件实现了权限提升.用户也可以创建一个`rpc`服务实现自己实现这2个接口自定义的`rpcss`解析和激活服务.这2个模块的可以在`rpcss`服务加载的`rpcss.dll`中找到具体实现,以下是接口定义:

```
[
uuid(99fcfec4-5260-101b-bbcb-00aa0021347a),
pointer_default(unique)
]
//marshal方式
interface IObjectExporter
{
[idempotent] error_status_t ResolveOxid
(
[in] handle_t hRpc,
[in] OXID *pOxid,
[in] unsigned short cRequestedProtseqs,
[in, ref, size_is(cRequestedProtseqs)]
unsigned short arRequestedProtseqs[],
[out, ref] DUALSTRINGARRAY **ppdsaOxidBindings,
[out, ref] IPID *pipidRemUnknown,
[out, ref] DWORD *pAuthnHint
);
[idempotent] error_status_t SimplePing
(
[in] handle_t hRpc,
[in] SETID *pSetId
);
[idempotent] error_status_t ComplexPing
(
[in] handle_t hRpc,
[in, out] SETID *pSetId,
[in] unsigned short SequenceNum,
[in] unsigned short cAddToSet,
[in] unsigned short cDelFromSet,
[in, unique, size_is(cAddToSet)] OID AddToSet[],
[in, unique, size_is(cDelFromSet)] OID DelFromSet[],
[out] unsigned short *pPingBackoffFactor
);
[idempotent] error_status_t ServerAlive
(
[in] handle_t hRpc
);
[idempotent] error_status_t ResolveOxid2
(
[in] handle_t hRpc,
[in] OXID *pOxid,
[in] unsigned short cRequestedProtseqs,
[in, ref, size_is(cRequestedProtseqs)]
unsigned short arRequestedProtseqs[],
[out, ref] DUALSTRINGARRAY **ppdsaOxidBindings,
[out, ref] IPID *pipidRemUnknown,
[out, ref] DWORD *pAuthnHint,
[out, ref] COMVERSION *pComVersion
);
[idempotent] error_status_t ServerAlive2
(
[in] handle_t hRpc,
[out, ref] COMVERSION *pComVersion,
[out, ref] DUALSTRINGARRAY **ppdsaOrBindings,
[out, ref] DWORD *pReserved
);
}
```

```
[
uuid(000001A0-0000-0000-C000-000000000046),
pointer_default(unique)
]
//CoCreateInstanceEx方式
interface IRemoteSCMActivator
{
void Opnum0NotUsedOnWire(void);
void Opnum1NotUsedOnWire(void);
void Opnum2NotUsedOnWire(void);
HRESULT RemoteGetClassObject(
[in] handle_t rpc,
[in] ORPCTHIS *orpcthis,
[out] ORPCTHAT *orpcthat,
[in,unique] MInterfacePointer *pActProperties,
[out] MInterfacePointer **ppActProperties
);
HRESULT RemoteCreateInstance(
[in] handle_t rpc,
[in] ORPCTHIS *orpcthis,
[out] ORPCTHAT *orpcthat,
[in,unique] MInterfacePointer *pUnkOuter,
[in,unique] MInterfacePointer *pActProperties,
[out] MInterfacePointer **ppActProperties
);
}
```

### 2.ntlm身份认证机制分析

<a href="elink@c71K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8S2N6X3g2F1M7r3!0J5N6q4)9J5k6i4y4G2N6i4u0U0k6h3k6G2M7X3N6W2i4K6u0W2L8X3g2@1i4K6u0r3L8Y4c8D9L8g2)9J5k6h3S2@1L8h3H3`.">ntlm官方解释</a>

`CVE-2015-2370`采用`CoGetInstanceFromIStorage`方式触发服务器从`IStorage`自身实现的`IMarhal`接口的`MarshalInterface`方法往`stream`中写入`marshaldata`

```
HRESULT CoGetInstanceFromIStorage(
COSERVERINFO *pServerInfo,
CLSID *pClsid,
IUnknown *punkOuter,
DWORD dwClsCtx,
IStorage *pstg,
DWORD dwCount,
MULTI_QI *pResults
);
```

`marshaldata`是一个`OBJREF`可以通过如下脚本使用`010editor`解析

```
local unsigned short sizetp;
local unsigned short sizesec;
typedef struct GUID
{
unsigned int Data1;
unsigned ushort Data2;
unsigned ushort Data3;
byte Data4[8];
} IID;
typedef struct tagDUALSTRINGARRAY {
local short sizetp;
local short sizesec;
local int aNetworkAddrLen;
local int aPrincNameLen;
unsigned short wNumEntries;
unsigned short wSecurityOffset;
sizetp=wSecurityOffset-2;
sizesec=(wNumEntries-wSecurityOffset-1);
Printf("wNumSTRINGBINDING is %d\n",sizetp);
Printf("wNumSECURITYBINDING is %d\n",sizesec);
while(sizetp>0)
{
struct tagSTRINGBINDING {
unsigned short wTowerId;
aNetworkAddrLen= ReadWStringLength(FTell());
Printf("aNetworkAddrLen is %d\n",aNetworkAddrLen);
char aNetworkAddr[aNetworkAddrLen*2];
sizetp= sizetp-aNetworkAddrLen-1;
} STRINGBINDING;
}
byte nullterm1[2];
while(sizesec>1)
{
struct tagSECURITYBINDING{
byte wAuthnSvc[2];
byte Reserved [2];
aPrincNameLen=ReadWStringLength(FTell());
Printf("aPrincNameLen is %d \n",aPrincNameLen);
char aPrincName[aPrincNameLen*2];
sizesec=sizesec-2-aPrincNameLen;
}SECURITYBINDING;
}
byte nullterm2[2];
} DUALSTRINGARRAY ;
struct tagOBJREF {
byte signature[4];
unsigned long flags;
GUID _iid;
if(OBJREF.flags==01h)
{
struct tagOBJREF_standard {
unsigned long flags;
unsigned long cPublicRefs;
struct oxid {
DWORD LowPart;
LONG HighPart;
} _oxid;
struct oid {
DWORD LowPart;
LONG HighPart;
} _oid;
struct ipid
{
unsigned int Data1;
unsigned ushort Data2;
unsigned ushort Data3;
byte Data4[8];
} _ipid;
DUALSTRINGARRAY dualstringarray;
} OBJREF_standard;
}
if(OBJREF.flags==02h)
{
struct tagOBJREF_handler {
unsigned char std[40];
struct clsid
{
unsigned int Data1;
unsigned ushort Data2;
unsigned ushort Data3;
byte Data4[8];
} _clsid;
unsigned char saResAddr[8];
} OBJREF_handler;
}
if(OBJREF.flags==04h)
{
struct tagOBJREF_custom {
struct clsid_custom
{
unsigned int Data1;
unsigned ushort Data2;
unsigned ushort Data3;
byte Data4[8];
} _clsid_custom;
unsigned long cbExtension;
unsigned long size;
unsigned byte pData;
} OBJREF_custom;
}
if(OBJREF.flags==08h)
{
struct tagOBJREF_EXTENDED{
byte std[40];
byte Signature1[4];
DUALSTRINGARRAY dualstringarray;
unsigned int nElms;
byte Signature2[4];
struct tagDATAELEMENT{
GUID dataID;
unsigned int cbSize;
unsigned int cbRounded;
struct ContextProperties{
unsigned ushort MajorVersion ;
unsigned ushort MinVersion;
GUID ContextId ;
unsigned int Flags ;
unsigned int Reserved ;
unsigned int dwNumExtents;
unsigned int cbExtents ;
unsigned int MshlFlags ;
unsigned int PROPMARSHALHEADERCount ;
unsigned int Frozen;
struct PROPMARSHALHEADER{
GUID clsidHEADER;
GUID policyId;
byte PROPMARSHALHEADEflags[4];
unsigned int cbctxProperty ;
byte ctxProperty[cbctxProperty];
} _PROPMARSHALHEADER[PROPMARSHALHEADERCount];
} _data;
} _DATAELEMENT;
}OBJREF_EXTENDED;
}
} OBJREF;
```

结果是一个`standard`的`matshal`模式,其中的`DUALSTRINGARRAY`字段指定远程解析的服务器为`127.0.0.1`的`6666`端口,也就是我们要使用中间人攻击监听端口,如下图: ![](upload/attach/201907/609565_5ADDWF3UF7DEBXJ.png)

`127.0.0.1`的`6666`监听的数据包经过中转后最终发送至`135`端口的`rpcss`服务,服务端先进行`ServerAlive`进行服务器时候在线确认,之后进行`ntlm`身份认证.

`NTLM`认证共需要三个消息完成：

![](upload/attach/201907/609565_2CBGGYSY4472ZNQ.png)

(1). Type1 消息: Negotiate 协商消息。

客户端在发起认证时，是首先向服务器发送协商消息,协商需要认证的服务类型从数据包中`UUID`为`IOXIDResolver(99fcfec4-5260-101b-bbcb-00aa0021347a)`代表协商的服务是`IObjectExporter`,如图它被我们替换成了`ISystemActivator(000001a0-0000-0000-c000-000000000046)`代表协商的服务替换成`IRemoteSCMActivator`方式,这里`CVE-2015-2370`为之后重放了一个`RemoteCreateInstance`请求做铺垫,告诉`rpcss`服务要最终要激活和请求是`RemoteCreateInstance`数据包中的内容,`Type1` 消息中的`Negotiate Flags`代表客户端要和服务器端协商加密等级

(2). Type2 消息: Challenge 挑战消息。

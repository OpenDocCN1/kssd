
```cpp
HRESULT CDownloadUtilities::MarshalBindContextToStream(CInterThreadMarshal **ppITM<eax>, CDownloadThreadParam *pDTP, wchar_t *szURL, IBindCtx *pBC)

{

	IEUserBroker *pIEUB;

	IUnknown *pUnk;

	HRESULT hr;

	if (SUCCEEDED(hr = pBC->QueryInterface(&IID_IUnknown, &pUnk)))

	{

		if (!_wcsnicmp(szURL, L"mk:", 3))

		{

			pUnk->Release();

			pUnk = NULL;

			hr = CreateAsyncBindCtxEx(0, 0, 0, 0, &pBC, 0);

			if (*ppITM)

			{

				(*ppITM)->~CInterThreadMarshal();

				delete *ppITM;

				*ppITM = NULL;

			}

		}

		if (SUCCEEDED(hr))

		{

			if (*ppITM)

			{

				*(pDTP + 18h) = *ppITM;

				*ppITM = NULL;

			}

			else

			{

				IStream **ppStm = new LPSTREAM;

				if (ppStm)

				{

					*ppStm = 0;

				}

				else

				{

					ppStm = NULL;

				}

				*(pDTP + 18h) = ppStm;

				hr = 8007000Eh;

				if (ppStm)

				{

					hr = CoMarshalInterThreadInterfaceInStream(&IID_IBindCtx, pUnk, ppStm);

				}

			}

			pUnk->Release();

		}

	}

	//omit

	return hr;

}
```

首先函数调用`pBC->QueryInterface`获得`pBC`对象的`IUnknown`接口`pUnk`，由于`IBindCtx`只继承自`IUnknown`，所以`IUnknown`接口和`IBindCtx`接口地址实际相同，当下载CHM内嵌的文件时，`CDownloadUtilities::MarshalBindContextToStream`函数的`szURL`参数为形如`mk:XXX`的字符串，所以函数调用`pUnk->Release`又设`pUnk=NULL`，而`pUnk`原本的计数应为`1`，`pUnk->Release`彻底释放了`pBC`对象，于是函数调用`CreateAsyncBindCtxEx`重新获得`pBC`对象，`pBC`是更新了但`pUnk`仍然为`NULL`，此外函数又彻底释放了`*ppITM`对象并设`*ppITM=NULL`，于是之后函数会调用`CoMarshalInterThreadInterfaceInStream`列集接口指针，但调用时第二个参数为`pUnk`，但`pUnk=NULL`，实际应为`pBC`，这还不是大问题，紧跟着的`pUnk->Release`才是祸根，由于`pUnk=NULL`，取`pUnk`的虚函数表指针造成访问违例，程序报错，不过幸好之前`pUnk`设为了`NULL`，不然后果就更糟糕了![](https://bbs.kanxue.com/view/img/face/1.gif)，解决方法应该是换用`pBC`。当`szURL`参数不是形如`mk:XXX`的字符串时就不会产生问题了。

附件是编译好的一个CHM，其中的`1.rar`是本人写的一个小工具，说明见本人博客`939K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6H3M7X3!0E0K9i4y4W2k6q4)9#2k6X3I4#2`[https://bbs.kanxue.com/elink@62bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6H3M7X3!0E0K9i4y4W2k6q4)9#2k6X3I4#2](https://bbs.kanxue.com/elink@62bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6H3M7X3!0E0K9i4y4W2k6q4)9#2k6X3I4#2)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [chm.7z](javascript:void(0);)

（729.63kb，29次下载）

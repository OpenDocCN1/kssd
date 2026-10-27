
# [原创]反虚拟机技术总结

大概是在一年半以前我在自己CSDN博客上写了[详解反虚拟机技术](https://bbs.kanxue.com/elink@805K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3x3K6t1@1x3o6l9^5y4o6N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0f1J5z5o6x3H3z5e0V1H3)和[详解反调试技术](https://bbs.kanxue.com/elink@fabK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3x3K6t1@1x3o6l9^5y4o6N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0f1J5y4K6V1^5x3o6f1H3)，没想到看的人还很多，有人甚至给我发私信发邮件，在百度和谷歌搜索“反调试”和“反虚拟机”，第一条结果就是我的文章。我决定在看雪也分享一下。当然我只是做了个整理收集的工作，没有一条技术和一行代码是我原创的，参考链接会附在最后。

恶意代码编写者经常使用反虚拟机技术逃避分析，这种技术可以检测自己是否运行在虚拟机中。如果恶意代码探测到自己在虚拟机中运行，它会执行与其本身行为不同的行为，其中最简单的行为是停止自身运行。近年来，随着虚拟化技术的使用不断增加，采用反虚拟机技术的恶意代码数量逐渐下降。恶意代码编写者已经开始意识到，目标主机是虚拟机，也并不意味着它就没有攻击价值。随着虚拟化技术的不断发展和普通应用，反虚拟机技术可能变得更加少见。这里研究最常见的反虚拟机技术（包括VMware、virtualbox和virtualpc，重点是最常用的VMware），并且介绍一些如何防御它们的办法。

## 一、检测虚拟机痕迹

### 1.根据MAC地址

通常，MAC地址的前三个字节标识一个提供商。以`00:05:69`、`00:0c:29`和`00:50:56`开始的MAC地址与VMware相对应；以`00:03:ff`开始的MAC地址与virtualpc对应；以`08:00:27`开始的MAC地址与virtualbox对应。

```
BOOL CheckVMWare()  

{  

    string mac;  

    get_3part_mac(mac);  

    if (mac=="00-05-69" || mac=="00-0c-29" || mac=="00-50-56")  

    {  

        return TRUE;  

    }  

    else  

    {  

        return FALSE;  

    }  

}
```

```
BOOL CheckVirtualPC()  

{  

    string mac;  

    get_3part_mac(mac);  

    if (mac=="00-03-ff")  

    {  

        return TRUE;  

    }  

    else  

    {  

        return FALSE;  

    }  

}
```

```
BOOL CheckVirtualBox()  

{  

    string mac;  

    get_3part_mac(mac);  

    if (mac=="08-00-27")  

    {  

        return TRUE;  

    }  

    else  

    {  

        return FALSE;  

    }  

}
```

```
typedef struct _ASTAT_  

{  

    ADAPTER_STATUS adapt;  

    NAME_BUFFER NameBuff[30];  

} ASTAT, *PASTAT;
```

```
void get_3part_mac(string &mac)  

{  

    NCB Ncb;  

    ASTAT Adapter;  

    UCHAR uRetCode;  

    LANA_ENUM lenum;  

    memset(&Ncb, 0, sizeof(Ncb));  

    Ncb.ncb_command = NCBENUM;  

    Ncb.ncb_buffer = (UCHAR *)&lenum;  

    Ncb.ncb_length = sizeof(lenum);  

    uRetCode = Netbios(&Ncb);  

    for (int i = 0; i < lenum.length; i++)  

    {  

        memset(&Ncb, 0, sizeof(Ncb));  

        Ncb.ncb_command = NCBRESET;  

        Ncb.ncb_lana_num = lenum.lana[i];  

        uRetCode = Netbios(&Ncb);  

        memset(&Ncb, 0, sizeof(Ncb));  

        Ncb.ncb_command = NCBASTAT;  

        Ncb.ncb_lana_num = lenum.lana[i];  

        strcpy((char *)Ncb.ncb_callname, "*");  

        Ncb.ncb_buffer = (unsigned char *)&Adapter;  

        Ncb.ncb_length = sizeof(Adapter);  

        uRetCode = Netbios(&Ncb);  

        if (uRetCode == 0)  

        {  

            char tmp[128];  

            sprintf(tmp, "%02x-%02x-%02x",  

                Adapter.adapt.adapter_address[0],  

                Adapter.adapt.adapter_address[1],  

                Adapter.adapt.adapter_address[2]  

            );  

            mac = tmp;  

        }  

    }  

}
```

### 2.基于主板序列号、主机型号、系统盘所在磁盘名称等其他硬件信息

```
//通过WMI获取主机信息  

BOOL ManageWMIInfo(string &result, string table, wstring wcol)  

{     

    HRESULT hres;  

    char bord[1024];  

    //初始化COM   

    hres = CoInitialize(0);  

    //获得WMI连接COM接口    

    IWbemLocator *pLoc = NULL;  

    hres = CoCreateInstance(  

        CLSID_WbemLocator,   

        0,   

        CLSCTX_INPROC_SERVER,   

        IID_IWbemLocator, (LPVOID *) &pLoc);  

    if (FAILED(hres))  

    {  

        cout << "Failed to create IWbemLocator object."  

            << "Err code = 0x"  

            << hex << hres << endl;  

        CoUninitialize();  

        return false;   

    }  

    //通过连接接口连接WMI的内核对象名ROOT//CIMV2    

    IWbemServices *pSvc = NULL;  

    hres = pLoc->ConnectServer(  

        _bstr_t(L"ROOT\\CIMV2"), // Object path of WMI namespace  

        NULL, // User name. NULL = current user  

        NULL, // User password. NULL = current  

        0, // Locale. NULL indicates current  

        NULL, // Security flags.  

        0, // Authority (e.g. Kerberos)  

        0, // Context object   

        &pSvc // pointer to IWbemServices proxy  

        );  

    if (FAILED(hres))  

    {  

        cout << "Could not connect. Error code = 0x"  

            << hex << hres << endl;  

        pLoc->Release();   

        CoUninitialize();  

        return false;   

    }  

    //设置请求代理的安全级别     

    hres = CoSetProxyBlanket(  

        pSvc, // Indicates the proxy to set  

        RPC_C_AUTHN_WINNT, // RPC_C_AUTHN_xxx  

        RPC_C_AUTHZ_NONE, // RPC_C_AUTHZ_xxx  

        NULL, // Server principal name   

        RPC_C_AUTHN_LEVEL_CALL, // RPC_C_AUTHN_LEVEL_xxx   

        RPC_C_IMP_LEVEL_IMPERSONATE, // RPC_C_IMP_LEVEL_xxx  

        NULL, // client identity  

        EOAC_NONE // proxy capabilities   

        );  

    if (FAILED(hres))  

    {  

        cout << "Could not set proxy blanket. Error code = 0x"  

            << hex << hres << endl;  

        pSvc->Release();  

        pLoc->Release();   

        CoUninitialize();  

        return false;   

    }  

    //通过请求代理来向WMI发送请求  

    IEnumWbemClassObject* pEnumerator = NULL;  

    string select = "SELECT * FROM "+ table;  

    hres = pSvc->ExecQuery(  

        bstr_t("WQL"),    

        bstr_t(select.c_str()),  

        WBEM_FLAG_FORWARD_ONLY | WBEM_FLAG_RETURN_IMMEDIATELY,   

        NULL,  

        &pEnumerator);  

    if (FAILED(hres))  

    {  

        cout << "Query for Network Adapter Configuration failed."  

            << " Error code = 0x”"  

            << hex << hres << endl;  

        pSvc->Release();  

        pLoc->Release();  

        CoUninitialize();  

        return false;   

    }  

    //循环枚举所有的结果对象  

    ULONG uReturn = 0;  

    IWbemClassObject *pclsObj;  

    while (pEnumerator)  

    {  

        HRESULT hr = pEnumerator->Next(WBEM_INFINITE, 1,   

            &pclsObj, &uReturn);  

        if(0 == uReturn)  

        {  

            break;  

        }  

        VARIANT vtProp;  

        VariantInit(&vtProp);  

        hr = pclsObj->Get(wcol.c_str(), 0, &vtProp, 0, 0);  

        if(!FAILED(hr))  

        {  

            CW2A tmpstr1(vtProp.bstrVal);  

            strcpy_s(bord,200,tmpstr1);  

            result = bord;  

        }  

        VariantClear(&vtProp);  

    }  

    //释放资源    

    pSvc->Release();  

    pLoc->Release();  

    pEnumerator->Release();  

    pclsObj->Release();  

    CoUninitialize();  

    return true;   

}
```

```
BOOL CheckVMWare()  

{  

    string table = "Win32_BaseBoard";  

    wstring wcol = L"SerialNumber";  

    string ret;  

    ManageWMIInfo(ret, table, wcol);  

    if (ret == "None")  

    {  

        return TRUE;  

    }  

    else  

    {  

        return FALSE;  

    }  

}
```

```
BOOL CheckVMWare()  

{  

    string table = "Win32_DiskDrive";  

    wstring wcol = L"Caption";  

    string ret;  

    ManageWMIInfo(ret, table, wcol);  

    if (ret.find("VMware") != string::npos)  

    {  

        return TRUE;  

    }  

    else  

    {  

        return FALSE;  

    }  

}
```

```
BOOL CheckVMWare()  

{  

    string table = "Win32_computersystem";  

    wstring wcol = L"Model";  

    string ret;  

    ManageWMIInfo(ret, table, wcol);  

    if (ret.find("VMware") != string::npos)  

    {  

        return TRUE;  

    }  

    else  

    {  

        return FALSE;  

    }  

}
```

```
BOOL CheckVirtualBox()  

{     

    string table = "Win32_computersystem";  

    wstring wcol = L"Model";  

    string ret;  

    ManageWMIInfo(ret, table, wcol);  

    if (ret.find("VirtualBox") != string::npos)  

    {  

        return TRUE;  

    }  

    else  

    {  

        return FALSE;  

    }  

}
```

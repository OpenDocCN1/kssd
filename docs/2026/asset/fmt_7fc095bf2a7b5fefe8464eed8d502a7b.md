
```
                NULL,

                PROV_RSA_FULL,

                CRYPT_NEWKEYSET))

            {

                _tprintf(TEXT("CryptAcquireContext succeeded.\n"));

            }

            else

            {

                MyHandleError(TEXT("Could not create the default ")

                    TEXT("key container.\n"));

            }

        }

        else

        {

            MyHandleError(TEXT("A general error running ")

                TEXT("CryptAcquireContext."));

        }

    }

    CHAR pszName[1000];

    DWORD cbName;

    cbName = 1000;

    if(CryptGetProvParam(

        hCryptProv,                                     //CSP模块句柄

        PP_NAME,                                     //指定获取哪些信息，这里是指定获取CSP名字的信息

        (BYTE*)pszName,                               //缓冲区接受信息返回值

        &cbName,

        0))

    {

        _tprintf(TEXT("CryptGetProvParam succeeded.\n"));

        printf("Provider name: %s\n", pszName);

    }

    else

    {

        MyHandleError(TEXT("Error reading CSP name.\n"));

    }

    //---------------------------------------------------------------

    // Read the name of the key container.

    cbName = 1000;

    if(CryptGetProvParam(

        hCryptProv,

        PP_CONTAINER,                                       //获取容器名字

        (BYTE*)pszName,

        &cbName,

        0))

    {

        _tprintf(TEXT("CryptGetProvParam succeeded.\n"));

        printf("Key Container name: %s\n", pszName);

    }

    else

    {

        MyHandleError(TEXT("Error reading key container name.\n"));

    }

    if(CryptContextAddRef(                       // CryptContextAddRef是向一个CSP的引用计数器增加一个值的函数

        hCryptProv,

        NULL,                                //保留值，必须为NULL

        0))                                                                           //保留值，必须为0

    {

        _tprintf(TEXT("CryptcontextAddRef succeeded.\n"));

    }

    else

    {

        MyHandleError(TEXT("Error during CryptContextAddRef!\n"));

    }

    //---------------------------------------------------------------

    //  The reference count on hCryptProv is now greater than one.

    //  The first call to CryptReleaseContext will not release the

    //  provider handle.

    //---------------------------------------------------------------

    //  Release the context once.

    if (CryptReleaseContext(hCryptProv, 0))                // CryptReleaseContext是用来释放CSP句柄的，当这个函数调用一次的时候，CSP里面的引用计数就减少一，当引用计数减少的0的时候。CSP将不能再被这个程序中的任何函数调用了。

    {

        _tprintf(TEXT("The first call to CryptReleaseContext ")

            TEXT("succeeded.\n"));

    }

    else

    {

        MyHandleError(TEXT("Error during ")

            TEXT("CryptReleaseContext #1!\n"));

    }

    if (CryptReleaseContext(hCryptProv, 0))                //再次释放CSP模块

    {

        _tprintf(TEXT("The second call to CryptReleaseContext ")

            TEXT("succeeded.\n"));

    }

    else

    {

        MyHandleError(TEXT("Error during ")

            TEXT("CryptReleaseContext #2!\n"));

    }
```

下面是从`PROV_RSA_FULL`的CSP模块中创建一个自己的容器

```
    LPCTSTR pszContainerName = TEXT("My Sample Key Container");

    hCryptProv = NULL;

    if(CryptAcquireContext(

        &hCryptProv,

        pszContainerName,

        NULL,

        PROV_RSA_FULL,

        CRYPT_NEWKEYSET))

    {

        _tprintf(TEXT("CryptAcquireContext succeeded. \n"));

        _tprintf(TEXT("New key set created. \n"));

        //-----------------------------------------------------------

        // Release the provider handle and the key container.

        if(hCryptProv)

        {

            if(CryptReleaseContext(hCryptProv, 0))

            {

                hCryptProv = NULL;

                _tprintf(TEXT("CryptReleaseContext succeeded. \n"));

            }

            else

            {

                MyHandleError(TEXT("Error during ")

                    TEXT("CryptReleaseContext!\n"));

            }

        }

    }

    else

    {

        if(GetLastError() == NTE_EXISTS)

        {

            _tprintf(TEXT("The named key container could not be ")

                TEXT("created because it already exists.\n"));

        }

        else

        {

            MyHandleError(TEXT("Error during CryptAcquireContext ")

                TEXT("for a new key container."));

        }

    }

    if(CryptAcquireContext(

        &hCryptProv,

        pszContainerName,

        NULL,

        PROV_RSA_FULL,

        0))

    {

        _tprintf(TEXT("Acquired the key set just created. \n"));

    }

    else

    {

        MyHandleError(TEXT("Error during CryptAcquireContext!\n"));

    }

    //---------------------------------------------------------------

    // Perform cryptographic operations.

    //---------------------------------------------------------------

       if(CryptReleaseContext(

        hCryptProv,

        0))

    {

        _tprintf(TEXT("CryptReleaseContext succeeded. \n"));

    }

    else

    {

        MyHandleError(TEXT("Error during CryptReleaseContext!\n"));

    }

    if(CryptAcquireContext(

        &hCryptProv,

        pszContainerName,

        NULL,

        PROV_RSA_FULL,

        CRYPT_DELETEKEYSET))                    //CRYPT_DELETEKEYSET意味着CryptAcquireContex删除一个指定的容器

    {

        _tprintf(TEXT("Deleted the key container just created. \n"));

    }

    else

    {

        MyHandleError(TEXT("Error during CryptAcquireContext!\n"));

    }

}
```

就先写到这，大家多提宝贵意见。附件中有这两个程序编译好的程序，源程序就是文章中的。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [cryptography api 学习笔记一.rar](javascript:void(0);)
  （20.20kb，191次下载）

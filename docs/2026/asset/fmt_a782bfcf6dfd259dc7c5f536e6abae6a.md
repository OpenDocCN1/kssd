
## 解密

### 3.4 获取最终密文

```cpp
        //3.4获取最终密文

        //先定义pTemp指向第二部分，循环查找0之后将其指向第三部分

        char*pTemp = g_DecodeText + 5;

        int tempcount = 0;

        while (*pTemp != 0)

        {

            tempcount++;

            pTemp++;

        }

        //将指针指向第三部分数据

        pTemp = g_DecodeText + 5 + tempcount + 1;

        //定义缓冲区接收最终密文

        char LastEncode[1024] = { 0 };

        int outvalue = 0;

        //完整代码放出

        GetLastEncode(pTemp, LastEncode,0x8000,&outvalue);

        printf("最终密文：%s\n", LastEncode);
```

### 4 获取明文

```cpp
    //4：获取明文
    CryptDecrypt(hKey,0,TRUE,0, (BYTE*)LastEncode, (DWORD*)&outvalue);
    printf("最终明文：%s\n", LastEncode);
```

## 测试

首先发送一条消息，然后使用wireshark抓包，将`data`部分复制下来，只保留从1001开始的部分！

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202002/868245_FUS7FXB4TFF8U7C.jpg)

打开工具进行测试

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202002/868245_Z8UWSD2MCMRFUB8.jpg)

## 源码

```cpp
// FeiQ.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。

//



#include "pch.h"

#include <stdio.h>

#include <windows.h>



//获取原始密文

char g_DecodeText[8000] = { 0 };



//算法

char Caculate(unsigned int BlobByte)

{

    char result; // al@3

    unsigned int BlobBytea; // [sp+8h] [bp+8h]@4



    if ((signed int)BlobByte < 48 || (signed int)BlobByte > 57)// 非数字

    {

        BlobBytea = toupper(BlobByte);              // 转大写

        if ((signed int)BlobBytea < 65 || (signed int)BlobBytea > 90)// 非A-Z

            result = 0;

        else

            result = BlobBytea - 55;                  // A-Z

    }

    else                                          // 数字

    {

        result = BlobByte - 48;

    }

    return result;

}



//获取BLOB

BOOL GetBlobBody(char *pBlob, char* pOutBlob, int fixvalue, int* pDwordValue)

{

    int Temp; // ebx@4

    int BlobLen; // [sp+4h] [bp-4h]@1



    BlobLen = strlen(pBlob);                      // 计算BLOB长度

    while (BlobLen >= 2 && *pDwordValue < fixvalue)// 判断BLOB长度是否大于2，参数三中的值是否小于0x3f4

    {

        Temp = Caculate(pBlob[BlobLen - 1]);// 倒序取pBlob[BlobLen-1]作为参数

        *((char*)(pOutBlob + *pDwordValue)) = 16 * Caculate(pBlob[BlobLen - 2]) | Temp;// 倒序取pBlob[BlobLen-2]作为参数

        BlobLen -= 2;

        ++*pDwordValue;

    }

    return 1;

}



//获取最终密文

int  GetLastEncode(char* EncodeText, char* OutDecodeText, int FixValue, int*_out_index)

{

    char v4; // bl@5



    for (*_out_index = 0; *EncodeText && *(EncodeText + 1) && *_out_index < FixValue; ++*_out_index)

    {

        v4 = 16 * Caculate(*EncodeText); // 取EncodeText位置字节

        *(char*)(OutDecodeText + *_out_index) = Caculate(*(EncodeText + 1)) | v4;// 取EncodeText+1处字节进行运算

        EncodeText += 2;                            // 指针自增2

    }

    return 1;

}





int main()

{

    printf("输入密文：\n");

    scanf_s("%s", g_DecodeText,8000);



    //1：按照0x3A进行分割分割

    int AllLen = strlen(g_DecodeText) + 1;

    for (int i = 0; i < AllLen; ++i)

    {

        if (*(g_DecodeText + i) == 0x3A)

        {

            *(g_DecodeText + i) = 0;

        }

    }



    //2：获取BLOB

    char BlobPart[1024] = { 0x01,0x02,0x00,0x00,0x02,0x66 ,0x00,0x00,0x00,0xA4,0x00,0x00};

    char*pOutBlob = BlobPart + 0xc;

    char*pInBlob = g_DecodeText + 5;

    int nCount = 0;

    GetBlobBody(pInBlob, pOutBlob, 0x3f4, &nCount);





    //3：获取最终密文

        //3.1获取CSP

        HCRYPTPROV hCspHandle = NULL;

        const char CspName[] = "feiq.rsa512.15pb-win7";

        const char CspProvider[] = "Microsoft Base Cryptographic Provider v1.0";

        CryptAcquireContextA(&hCspHandle, CspName, CspProvider,PROV_RSA_FULL,CRYPT_MACHINE_KEYSET);

        printf("CSP句柄：%08x\n", hCspHandle);



        //3.2获取公钥/私钥对句柄

        HCRYPTPROV hUserHandle = NULL;

        CryptGetUserKey(hCspHandle, AT_KEYEXCHANGE, &hUserHandle);

        printf("公/私钥对句柄：%08x\n", hUserHandle);



        //3.3导入密钥并获取句柄

        HCRYPTKEY hKey = NULL;

        CryptImportKey(hCspHandle,(const BYTE*)BlobPart, nCount+0XC, hUserHandle,0,&hKey);

        printf("CryptImportKey：%08x\n", hKey);





        //3.4获取最终密文

        char*pTemp = g_DecodeText + 5;

        int tempcount = 0;

        while (*pTemp != 0)

        {

            tempcount++;

            pTemp++;

        }



        pTemp = g_DecodeText + 5 + tempcount + 1;

        char LastEncode[1024] = { 0 };

        int outvalue = 0;

        GetLastEncode(pTemp, LastEncode,0x8000,&outvalue);

        printf("最终密文：%s\n", LastEncode);





    //4：获取明文

    CryptDecrypt(hKey,0,TRUE,0, (BYTE*)LastEncode, (DWORD*)&outvalue);

    printf("最终明文：%s\n", LastEncode);

    system("pause");

    return 0;

}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#加密算法](https://bbs.kanxue.com/forum-4-1-5.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)


```c
//获取机器码：计算MD5的函数实现本处省略

char* GetMachineID(char* pMac, char* pBios, char* pMachineID)
{
    BYTE uMD5Buf[32] = { 0 };
    BYTE uTmpBuf[32] = { 0 };
    char* pTmp = NULL;

    //连接字符串
    if (pMac == NULL)
    {
        if (pBios != NULL)
        {
            pTmp = pBios;
        }
    }
    else
    {
        if (pBios != NULL)
        {
            strcat(pMac, pBios);
            pTmp = pMac;
        }
        pTmp = pMac;
    }

    //计算MD5值：这个MD5不能出来空字符串，需要单独提出来处理
    if (pTmp == NULL)
    {
        strcpy((char*)uTmpBuf, "d41d8cd98f00b204e9800998ecf8427e");
    }
    else
    {
        MDString(pTmp, uMD5Buf);
        HexToStr(uTmpBuf, uMD5Buf, 16);  //将MD5数据转为16进制字符串
        strlwr((char*)uTmpBuf);          //将字符串转为小写
    }

    //尾部追加#
    strcat((char*)uTmpBuf, "#");
    //计算MD5值
    MDString((char*)uTmpBuf, uMD5Buf);
    HexToStr(uTmpBuf, uMD5Buf, 16);  //将MD5数据转为16进制字符串
    //截取前半部分字符串
    uTmpBuf[16] = '\0';
    memcpy(pMachineID, uTmpBuf, 16);
    return pMachineID;
}
```

---

#### 4. 生成注册码

生成注册码：

```c
//获取注册码的第一段
char* RegistrationCode_1(char* pSrc, char* pDest, int nLen)
{
    memcpy(pDest, pSrc, nLen);
    return pDest;
}

//获取注册码的第二/三段
char* RegistrationCode_2_3(char* pSrc, char* pDest, int nStart, int nLen)
{
    if (nStart < 0)
    {
        nStart = 0;
    }
    if (nLen < 0)
    {
        nLen = 0;
    }
    //....
    memcpy(pDest, pSrc + nStart, nLen);
    return pDest;
}

//获取注册码的第四段
char* RegistrationCode_4(char* pSrc, char* pDest, int nLen)
{
    if (nLen < 0)
    {
        nLen = 0;
    }
    int nSrcLen = strlen(pSrc);
    if (nLen < nSrcLen)
    {
        pSrc += nSrcLen;
        pSrc -= nLen;
        memcpy(pDest, pSrc, nLen);
    }
    return pDest;
}

//功能：根据机器码获取注册码
//参数：
//  pMachineID：传入参数，机器码
//  pKey：传出参数，存储计算完的注册码
char* GetSerialNumber(char* pMachineID, char* pKey)
{
    char szTmpBuf[5] = { 0 };
    BYTE uMD5Buf[CODE_LEN] = { 0 };
    BYTE uTmpBuf[CODE_LEN] = { 0 };

    //计算机器码的MD5值
    MDString(pMachineID, uMD5Buf);
    //将MD5数据转为16进制字符串
    HexToStr(uTmpBuf, uMD5Buf, 16);
    //将机器码的MD5字符串再次计算MD5
    MDString((char*)uTmpBuf, uMD5Buf);
    //将MD5数据转为16进制字符串
    HexToStr(uTmpBuf, uMD5Buf, 16);
    //将字符串转为小写
    strlwr((char*)uTmpBuf);

    //注册码的第一段
    RegistrationCode_1((char*)uTmpBuf, szTmpBuf, 4);
    strcat(pKey, szTmpBuf);
    //注册码的第二段
    RegistrationCode_2_3((char*)uTmpBuf, szTmpBuf, 0x0A, 0x4);
    sprintf(pKey, "%s-%s", pKey, szTmpBuf);
    //注册码的第三段
    RegistrationCode_2_3((char*)uTmpBuf, szTmpBuf, 0x16, 0x4);
    sprintf(pKey, "%s-%s", pKey, szTmpBuf);
    //注册码的第四段
    RegistrationCode_4((char*)uTmpBuf, szTmpBuf, 4);
    sprintf(pKey, "%s-%s", pKey, szTmpBuf);
    return pKey;
}
```

### 总结

之前从来没有写过KeyGen，该软件的注册码生成基本上没啥难度，倒是追机器码搞得我头大

说明：

> 1. KeyGen在虚拟机中可能无法获取硬盘信息和物理内存信息，我在XP虚拟机下正常获取，Win7下获取不到硬盘信息和物理内存信息（具体细节没研究），不过该软件在Win7虚拟机下也是获取不到硬盘信息和物理内存信息，KeyGen可以正常获取注册码
> 2. Win10下需要管理员权限运行才能拿到硬件信息
> 3. 工程中的KeyGen不能直接实现软件注册（只用于学习，务作它用），因为该软件除了本地验证外还有网络验证（Windows Defender会报毒直接杀掉，介意的务尝试），只要输入注册码出现以下截图就说明本地注册完成，填写邮箱是网络验证的事情
>
> ![](https://bbs.kanxue.com/upload/attach/201712/770162_nx4h7r157n6efmz.png)
>
> 参考文档：
>
> 获取硬盘序列号参考代码：`ce3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3N6r3!0V1P5g2)9#2k6X3N6#2L8#2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3U0j5H3z5o6b7I4y4o6x3`.
>
> 获取物理内存信息参考代码：`bddK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3N6$3q4F1k6%4S2$3k6X3g2F1k6K6p5H3x3g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4K6x3&6y4o6M7J5y4b7`.

---

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-7-14 21:30 被turtledove编辑，原因：

上传的附件：

- [`KeyGen.rar`](javascript:void(0);)（31.49kb，99次下载）

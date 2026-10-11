
# 使用 `CryptoAPI` 解析 `X509` 证书和 `P12` 证书

> 原文：[`www.pediy.com/kssd/pediy10/97663.html`](https://www.pediy.com/kssd/pediy10/97663.html)

## 一、解析 `X509` 证书

### 1. 从磁盘上的证书文件中读取证书数据

```
unsigned char* pbX509Data; // 证书数据
unsigned long ulX509DataLen; // 证书数据长度
```

### 2. 获取 `CertContext`

```
PCCERT_CONTEXT pCertContext = CertCreateCertificateContext(X509_ASN_ENCODING, pbX509Data, ulX509DataLen);
```

### 3. 获取证书信息

```
pCertContext->pCertInfo->dwVersion; // 证书版本号

CRYPT_INTEGER_BLOB snBlob = pCertContext->pCertInfo->SerialNumber; // 证书 SN

CERT_NAME_BLOB issuerBlob = pCertContext->pCertInfo->Issuer; // 证书颁发者

CERT_NAME_BLOB subjectBlob = pCertContext->pCertInfo->Subject; // 证书主题

// 证书有效起始日期

SYSTEMTIME sysTime;

memset(&sysTime, 0, sizeof(sysTime));

FileTimeToSystemTime(&pCertContext->pCertInfo->NotBefore, &sysTime);

char szTime[128] = {0};

sprintf_s(szTime, 128, "%d 年%d 月%d 日 %d:%d:%d", sysTime.wYear, sysTime.wMonth, sysTime.wDay, sysTime.wHour, sysTime.wMinute, sysTime.wSecond);

// 证书有效终止日期

memset(&sysTime, 0, sizeof(sysTime));

FileTimeToSystemTime(&pCertContext->pCertInfo->NotAfter, &sysTime);

memset(szTime, 0, sizeof(szTime));

sprintf_s(szTime, 128, "%d 年%d 月%d 日 %d:%d:%d", sysTime.wYear, sysTime.wMonth, sysTime.wDay, sysTime.wHour, sysTime.wMinute, sysTime.wSecond);
```

### 4. 创建临时密钥容器

```
HCRYPTPROV hTmpProv = NULL;

if (RCRYPT_FAILED(CryptAcquireContext(&hTmpProv, "My_Temporary_Container", NULL, PROV_RSA_AES, CRYPT_NEWKEYSET)))    // NULL 表示使用系统默认 CSP
{
    if (RCRYPT_FAILED(CryptAcquireContext(&hTmpProv, "My_Temporary_Container", NULL, PROV_RSA_AES, 0)))    // NULL 表示使用系统默认 CSP
    {
        CertFreeCertificateContext(pCertContext);
        return NTE_FAIL;
    }
}
```

### 5. 向容器中导入公钥，获取公钥句柄

```
HCRYPTKEY hKey = NULL;

CERT_PUBLIC_KEY_INFO certPubKeyInfo = pCertContext->pCertInfo->SubjectPublicKeyInfo;

CryptImportPublicKeyInfo(hTmpProv, X509_ASN_ENCODING|PKCS_7_ASN_ENCODING, &certPubKeyInfo, &hKey);
```

### 6. 导出公钥（最好采用二次调用方式）

```
unsigned char* pBuf = NULL;

unsigned long ulBufLen = 0;

CryptExportKey(hKey, 0, PUBLICKEYBLOB, 0, pBuf, &ulBufLen);

pBuf = new unsigned char[ulBufLen];

memset(pBuf, 0, ulBufLen);

CryptExportKey(hKey, 0, PUBLICKEYBLOB, 0, pBuf, &ulBufLen);
```

### 7. 获取公钥信息

```
unsigned char* p = pBuf + sizeof(PUBLICKEYSTRUC);

(*(RSAPUBKEY*)p).bitlen; // 公钥模长（以 bit 为单位）

(*(RSAPUBKEY*)p).pubexp; // 公钥的 e（注意字节顺序）

p += sizeof(RSAPUBKEY); // 公钥的 n（注意字节顺序）
```

### 8. 清理工作

```
delete[] pBuf;

pBuf = NULL;

CryptDestroyKey(hKey);

CryptReleaseContext(hTmpProv, 0);

CertFreeCertificateContext(pCertContext);
```

## 二、解析 `P12` 证书

### 1. 从磁盘上的证书文件中读取证书数据

```
unsigned char* pbP12Data; // 证书数据

unsigned long ulP12DataLen; // 证书数据长度
```

### 2. 让用户输入证书密码

```
char* szPwd; // 证书密码
```

### 3. 将证书密码转换成 `UNICODE` 格式（最好采用二次调用方式）

```
LPWSTR pWideChar = NULL;

int nWideChar = 0;

nWideChar = MultiByteToWideChar(CP_ACP, 0, szPwd, -1, pWideChar, nWideChar);

pWideChar = new WCHAR[nWideChar];

memset(pWideChar, 0, sizeof(WCHAR)*nWideChar);

MultiByteToWideChar(CP_ACP, 0, szPwd, -1, pWideChar, nWideChar);
```

### 4. 将证书数据导入临时 `store`

```
CRYPT_DATA_BLOB blob;

memset(&blob, 0, sizeof(blob));

blob.pbData = pbP12Data;

blob.cbData = ulP12DataLen;

HCERTSTORE hCertStore = NULL;

hCertStore = PFXImportCertStore(&blob, pWideChar, CRYPT_EXPORTABLE);
```

### 5. 在 `store` 中查找证书，获取 `CertContext`

```
PCCERT_CONTEXT pCertContext = CertFindCertificateInStore(hCertStore, X509_ASN_ENCODING|PKCS_7_ASN_ENCODING, 0, CERT_FIND_ANY, NULL, NULL);
```

### 6. 获取证书信息

```
CRYPT_INTEGER_BLOB snBlob = pCertContext->pCertInfo->SerialNumber; // 证书 SN

pCertContext->pbCertEncoded; // X509 格式证书数据

pCertContext->cbCertEncoded; // X509 格式证书数据长度
```

### 7. 获取 `CSP` 句柄

```
HCRYPTPROV hProv = NULL;

DWORD dwKeySpec = 0;

BOOL bCallerFreeProv = FALSE;

CryptAcquireCertificatePrivateKey(pCertContext, 0, NULL, &hProv, &dwKeySpec, &bCallerFreeProv);
```

### 8. 获取密钥句柄

```
HCRYPTKEY hKey = NULL;

CryptGetUserKey(hProv, dwKeySpec, &hKey);
```

### 9. 导出私钥（最好采用二次调用方式）

```
BYTE* pbData = NULL;

DWORD dwDataLen = 0;

CryptExportKey(hKey, NULL, PRIVATEKEYBLOB, 0, pbData, &dwDataLen);

pbData = new BYTE[dwDataLen];

memset(pbData, 0, dwDataLen);

CryptExportKey(hKey, NULL, PRIVATEKEYBLOB, 0, pbData, &dwDataLen);
```

### 10. 获取公私钥信息

```
BYTE *p = pbData+ sizeof(PUBLICKEYSTRUC);

(*(RSAPUBKEY*)p).bitlen; // 公私钥模长（以 bit 为单位）

(*(RSAPUBKEY*)p).pubexp; // 公钥的 e（注意字节顺序）

p += sizeof(RSAPUBKEY); // 公私钥的 n（注意字节顺序）

p += ((*(RSAPUBKEY*)p).bitlen)/8; // 私钥的 p（注意字节顺序）

p += ((*(RSAPUBKEY*)p).bitlen)/16; // 私钥的 q（注意字节顺序）

p += ((*(RSAPUBKEY*)p).bitlen)/16; // 私钥的 dp（注意字节顺序）

p += ((*(RSAPUBKEY*)p).bitlen)/16; // 私钥的 dq（注意字节顺序）

p += ((*(RSAPUBKEY*)p).bitlen)/16; // 私钥的 qu（注意字节顺序）

p += ((*(RSAPUBKEY*)p).bitlen)/16; // 私钥的 d（注意字节顺序）
```

### 11. 清理工作

```
delete[] pbData;

pbData = NULL;

CryptDestroyKey(hKey);

CryptReleaseContext(hProv, 0);

CertFreeCertificateContext(pCertContext);

CertCloseStore(hCertStore, CERT_CLOSE_STORE_FORCE_FLAG);

delete[] pWideChar;

pWideChar = NULL;
```

注：为了缩短篇幅，上述代码没有包含错误和异常处理。

上传源代码及 `exe` 文件，欢迎大家指正![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `ParseCert.rar` |
| :--- | :--- | :--- |


# Crypto API 学习笔记二

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-412.htm`](https://www.pediy.com/kssd/pediy08/pediy8-412.htm)

Cryptography API 学习笔记二

前面学习了密钥容器的建立的一些知识，现在我们接下来自然是学习如何获取一个密钥，首先是获取一个 session key，即对话密钥，是对称密钥。

这些学习笔记主要是从 MSDN 中的例子中，学习基本的 Cryptography API 的用法，了解一些用法的过程。欢迎大家多提宝贵意见。

下面依旧是从一个小程序开始。这个程序的学习任务有以下几个：

任务一：调用 `CryptAcquireContext`，获取一个缺省 CSP 和缺省密钥容器的句柄

任务二：用 `CryptCreateHash` 去创建一个空的哈希对象

任务三：用 `CryptHashData` 哈希密码

任务四：调用 `CryptDeriveKey` 获取一个对话密钥

任务五：销毁密码和哈希后的数据

任务六：释放 CSP

```c
#include <stdio.h>
#include <string.h>
#include <conio.h>
#include <windows.h>
#include <Wincrypt.h>

#define MY_ENCODING_TYPE  (PKCS_7_ASN_ENCODING | X509_ASN_ENCODING)

void MyHandleError(char *s);
void GetConsoleInput(char*, int);

void main()
{
HCRYPTPROV hCryptProv;            定义 CSP 句柄
HCRYPTKEY hKey;                  定义密钥句柄
HCRYPTHASH hHash;                定义一个 HASH 对象的句柄
CHAR szPassword[512] = "";      定义 512 大小的字符数组，用来保存密码
DWORD dwLength;                  保存密码长度

fprintf(stderr,"Enter a password to be used to create a key:");
GetConsoleInput(szPassword, 512); 获取密码，这个是自己写的函数，目的是在屏幕上显示的是*。
printf("The password has been stored.\n");

dwLength = strlen(szPassword);

if(CryptAcquireContext(          以下是获取一个缺省的 PROV_RSA_FULL CSP 句柄
   &hCryptProv,
   NULL,
   NULL,
   PROV_RSA_FULL,
   0))
{
    printf("A context has been acquired. \n");
}
else
{
     MyHandleError("Error during CryptAcquireContext!");
}

//--------------------------------------------------------------------

if(CryptCreateHash(             调用 CryptCreateHash 创建一个 HASH 对象
   hCryptProv,                 一个 CSP 句柄
   CALG_MD5,                    确定哈希算法
   0,   对于非密钥算法，这个参数一定是 0，如果是密钥算法，那么这个参数就是密钥
   0,                           保留参数，为 0
   &hHash))                     一个哈希对象的指针
{
    printf("An empty hash object has been created. \n");
}
else
{
    MyHandleError("Error during CryptCreateHash!");
}

//--------------------------------------------------------------------

if(CryptHashData(             调用 CryptHashData 哈希密码
   hHash,                    哈希对象
   (BYTE *)szPassword,       指向缓冲区的地址
   dwLength,                 密码长度
   0))
{
     printf("The password has been hashed. \n");
}
else
{
     MyHandleError("Error during CryptHashData!");
}

//--------------------------------------------------------------------

if(CryptDeriveKey(          调用 CryptDeriveKey 获取对话密码
   hCryptProv,              CSP 句柄
   CALG_RC2,               一个 ALG_ID 结构，用来指定对称密钥生成的算法
   hHash,                  哈希对象
   CRYPT_EXPORTABLE,       指定生成密钥的类型，CRYPT_EXPORTABLE 意味着这个程序生成的密钥可以被其它程序调用，而不是仅仅限于这个程序当中。但是它不能用于非对称密码中。
   &hKey))
{
    printf("The key has been derived. \n");
}
else
{
    MyHandleError("Error during CryptDeriveKey!");
}

if(hHash)                  销毁哈希对象
{
   if(!(CryptDestroyHash(hHash)))
       MyHandleError("Error during CryptDestroyHash");
}

if(hKey)                   销毁密钥句柄
{
    if(!(CryptDestroyKey(hKey)))
        MyHandleError("Error during CryptDestroyKey");
}

if(hCryptProv)             销毁 CSP 句柄
{
   if(!(CryptReleaseContext(hCryptProv, 0)))
       MyHandleError("Error during CryptReleaseContext");
}

printf("The program to derive a key completed without error. \n");
} // end main
```

`void MyHandleError` 函数的实现省略，在上一篇中有

```c
void GetConsoleInput(char* strInput,
               int   intMaxChars)
{
  char ch;
  char minChar = ' ';
  minChar++;
  ch = getch();
  while (ch != '\r')
  {
    if (ch == '\b' && strlen(strInput) > 0)
    {
      strInput[strlen(strInput)-1]   = '\0';
      printf("\b \b");
    }
    else if (ch >= minChar && strlen(strInput) < intMaxChars)
    {
      strInput[strlen(strInput)+1] = '\0';
      strInput[strlen(strInput)]   = ch;
      putch('*');
    }
    ch = getch();
  }
  putch('\n');
}
```

下面讲讲如何复制一个对话密钥，照例是从一个程序讲起，完成以下任务。获取一个 CSP 句柄，创建一个对话密钥，复制密钥。改变密钥生成过程，随机填充一个缓冲区，销毁密钥句柄，释放 CSP 句柄。

```c
#include <stdio.h>
#include <windows.h>
#include <Wincrypt.h>

#define MY_ENCODING_TYPE  (PKCS_7_ASN_ENCODING | X509_ASN_ENCODING)

void MyHandleError(char *s);
void main()
{
//-------------------------------------------------------------------
// Declare and initialize variables.
HCRYPTPROV   hCryptProv;       CSP 句柄
HCRYPTKEY    hOriginalKey;     源密钥句柄
HCRYPTKEY    hDuplicateKey;    复制后的密钥句柄
DWORD        dwMode;
BYTE         pbData[16];

printf("This program creates a session key and duplicates \n");
printf("that key. Next, parameters are added to the original \n");
printf("key. Finally, both keys are destroyed. \n\n");

if(CryptAcquireContext(          获取 CSP 句柄，前面叙述过了，这里就不叙述了
   &hCryptProv,
   NULL,
   NULL,
   PROV_RSA_FULL,
   0))
{     
    printf("CryptAcquireContext succeeded. \n");
}
else
{
    MyHandleError("Error during CryptAcquireContext!\n");
}

//-------------------------------------------------------------------

if (CryptGenKey(               生成一个 CALG_RC4 算法生成的密钥，保存在 hOriginalKey 中
     hCryptProv, 
     CALG_RC4,               ALG_ID 结构，指定生成这个密钥使用的算法
     0, 
     &hOriginalKey))
{
   printf("Original session key is created. \n");
}
else
{
   MyHandleError("ERROR - CryptGenKey.");
}

if (CryptDuplicateKey(          复制密钥
     hOriginalKey,             源密钥
     NULL,                     保留参数,必须为 NULL
     0,                        保留参数，必须为 0
     &hDuplicateKey))          副本密钥
{
   printf("The session key has been duplicated. \n");
}
else
{
   MyHandleError("ERROR - CryptDuplicateKey");
}

给源密钥设置附加参数

dwMode = CRYPT_MODE_ECB;       CRYPT_MODE_ECB 是一个没有反馈的块加密模式

if(CryptSetKeyParam(
   hOriginalKey, 
   KP_MODE,                  指定密钥的某种属性被改变，
KP_MODE 意味着改变的是加密模式
(BYTE*)&dwMode,            指向一个已经被初始化的缓冲区
   0))                      

{
     printf("Key Parameters set. \n");
}
else
{
     MyHandleError("Error during CryptSetKeyParam.");
}

if(CryptGenRandom(          随机填充一块缓冲区
   hCryptProv,             CSP 句柄
   8,                     缓冲区大小
   pbData))               缓冲区地址
{
     printf("Random sequence generated. \n");
}
else
{
     MyHandleError("Error during CryptGenRandom.");
}

if(CryptSetKeyParam(          再次给密钥设置属性
   hOriginalKey, 
     KP_IV,             KP_IV 意味着，这个函数的第三个参数指向一个 BYTE 数组，数组大小为块大小/8。
   pbData, 
   0)) 
{
     printf("Parameter set with random sequence as "
     "initialization vector. \n");
}
else
{
     MyHandleError("Error during CryptSetKeyParam.");
}

//-------------------------------------------------------------------

if (hOriginalKey)            以下依次销毁，释放源密钥句柄，副本句柄，CSP 句柄，
    if (!CryptDestroyKey(hOriginalKey))
        MyHandleError("Failed CryptDestroyKey\n");

if (hDuplicateKey)



`if (!CryptDestroyKey(hDuplicateKey))`  

`MyHandleError("Failed CryptDestroyKey\n");`  

`if(hCryptProv)`  

`if (!CryptReleaseContext(hCryptProv, 0))`  

`MyHandleError("Failed CryptReleaseContext\n");`  

`printf("\nThe program ran to completion without error. \n");`  

`}` // End of main.  

当我们已经生成了一个密钥，我们下面要做的是如何保存，导出，导入一个对话密钥。将一个密钥保存起来，导出，保存到硬盘上。这样，这个密钥就可以在其它应用程序上使用，而不会随着生成密钥程序的关闭而丢失了。  

下面这段程序就是用来演示如何保存，导出一个对话密钥的。  

```c
#include <stdio.h>  

#include <windows.h>  

#include <Wincrypt.h>  

#define MY_ENCODING_TYPE  (PKCS_7_ASN_ENCODING | X509_ASN_ENCODING)  

void MyHandleError(char *s);  

void main(void)  

{  

//--------------------------------------------------------------------  

// Declare and initialize variables.  

HCRYPTPROV hProv;       // CSP handle  

HCRYPTKEY hSignKey;     // Signature key pair handle  

HCRYPTKEY hXchgKey;     // Exchange key pair handle  

HCRYPTKEY hKey;         // Session key handle  

BYTE *pbKeyBlob;        // Pointer to a simple key BLOB  

DWORD dwBlobLen;        // The length of the key BLOB  

//--------------------------------------------------------------------  

// Acquire a cryptographic provider context handle.  

if(CryptAcquireContext(             // 获取一个缺省容器的 CSP 句柄  
   &hProv,  
   NULL,  
   NULL,  
   PROV_RSA_FULL,  
   0))  
{  
    printf("The CSP has been acquired. \n");  
}  
else  
{  
    MyHandleError("Error during CryptAcquireContext.");  
}  

if(CryptGetUserKey(                // 获取一个 AT_SIGNATURE 类型的密钥句柄  
   hProv,  
   AT_SIGNATURE,  
   &hSignKey))  
{  
    printf("The signature key has been acquired. \n");  
}  
else  
{  
    MyHandleError("Error during CryptGetUserKey for signkey.");  
}  

//--------------------------------------------------------------------  

if(CryptGetUserKey(                // 获取一个 AT_KEYEXCHANGE 类型的密钥句柄，保存在 hXchgKey 中  
   hProv,  
   AT_KEYEXCHANGE,  
   &hXchgKey))  
{  
    printf("The key exchange key has been acquired. \n");  
}  
else  
{  
    printf("Error during CryptGetUserKey exchange key.");  
}  

// Generate a session key.  

if (CryptGenKey(                   // 生成一个 CRYPT_EXPORTABLE（可导出的），CALG_RC4（指定算法）的密钥，保存在 hKey  
    hProv,      
    CALG_RC4,      
    CRYPT_EXPORTABLE,  
    &hKey))  
{    
    printf("Original session key is created. \n");  
}  
else  
{  
    MyHandleError("ERROR -- CryptGenKey.");  
}  

if(CryptExportKey(                 // CryptExportKey 导出一个密钥  
   hKey,                           // 将要导出的密钥的句柄  
   hXchgKey,                       // 用户最终使用到的密钥的句柄  
   SIMPLEBLOB,                     // 指定 BLOB 的类型，SIMPLEBLOB 说明是 用来导出对话密钥的  
   0,                              // 指定密钥的附加属性  
   NULL,                           //  
   &dwBlobLen))                   // 当时这个函数在这里的主要目的是得到这个 BLOB 的长度  
{  
     printf("Size of the BLOB for the session key determined. \n");  
}  
else  
{  
     MyHandleError("Error computing BLOB length.");  
}  

if(pbKeyBlob = (BYTE*)malloc(dwBlobLen))  
{  
    printf("Memory has been allocated for the BLOB. \n");  
}  
else  
{  
    MyHandleError("Out of memory. \n");  
}  

if(CryptExportKey(                // 这是这个函数才是真正的导出密钥  
   hKey,  
   hXchgKey,  
   SIMPLEBLOB,  
   0,  
   pbKeyBlob,  
   &dwBlobLen))  
{  
     printf("Contents have been written to the BLOB. \n");  
}  
else  
{  
    MyHandleError("Error during CryptExportKey.");  
}  

free(pbKeyBlob);                 // 释放内存  

// Destroy the session key.  

if(hKey)  
    CryptDestroyKey(hKey);  

// Destroy the signature key handle.  

if(hSignKey)  
    CryptDestroyKey(hSignKey);  

// Destroy the key exchange key handle.  

if(hXchgKey)  
     CryptDestroyKey(hXchgKey);  

// Release the provider handle.  

if(hProv)  
   CryptReleaseContext(hProv, 0);  

printf("The program ran to completion without error. \n");  
}  
```

附件是 DOC 文档，希望多提宝贵意见  

`4dd466d0e1e0cc123afbe504da2462ca`

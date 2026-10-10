

# Crypto API 学习笔记三

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-363.htm`](https://www.pediy.com/kssd/pediy08/pediy8-363.htm)

Crypto API 学习笔记三  
Encoding and Decoding Data  
下面逐渐进入主题了，现在来讲讲是如何对数据进行 Encoding and Decoding 的。依旧是从一段程序中开始。  
#include <stdio.h>  
#include <windows.h>  
#include <Wincrypt.h>  
#define MY\_ENCODING\_TYPE  (PKCS\_7\_ASN\_ENCODING | X509\_ASN\_ENCODING)  
void MyHandleError(char \*s);  
  
void main(void)  
{  
HCRYPTMSG hMsg;                              指向一个消息句柄  
BYTE\* pbContent;                   一个 BYTE 指针指向消息  
DWORD cbContent;                   消息长度  
DWORD cbEncodedBlob;                          ECODE 的 BLOB 的大小  
BYTE \*pbEncodedBlob;                           一个 BYTE 指针指向 ENCODE BLOB  
  
DWORD cbData = sizeof(DWORD);                 数据大小  
DWORD cbDecoded;                              Decode 内容大小  
BYTE \*pbDecoded;                               指向 Decode 的指针  
  
pbContent = (BYTE\*) "Security is our only business";  
cbContent = strlen((char \*) pbContent)+1;  
  
printf("The original message => %s\\n",pbContent);    
  
if(cbEncodedBlob = CryptMsgCalculateEncodedLength(  
             MY\_ENCODING\_TYPE,      指定 Encode 类型，在程序的开头已经预定义了，MY\_ENCODING\_TYPE 就是 PKCS\_7\_ASN\_ENCODING | X509\_ASN\_ENCODING  
             0,                      // Flags  
             CMSG\_DATA,                定义了数据的类型，这里指定为 BYTE 型的字符串  
             NULL,                     
             NULL,                     
             cbContent))               内容的大小  
这里的的函数的作用是计算指定消息 Encode 所需要的最大的长度，通过计算，为一个 BLOB 分配内存空间。  
{  
    printf("The length of the data has been calculated. \\n");  
}  
else  
{  
    MyHandleError("Getting cbEncodedBlob length failed");  
}  
为 encode blob 分配内存空间  
  
if(pbEncodedBlob = (BYTE \*) malloc(cbEncodedBlob))  
{  
   printf("Memory has been allocated for the signed message. \\n");  
}  
else  
{  
   MyHandleError("Memory allocation failed");  
}  
  
if(hMsg = CryptMsgOpenToEncode(                 CryptMsgOpenToEncode 为 Encode，开一个消息  
          MY\_ENCODING\_TYPE,            Encode 类型，文件开始有说明  
          0,                       // Flags  
          CMSG\_DATA,                        指定 Message 的类型，CMSG\_DATA 说明类型没用到  
          NULL,                                    现在没有到，为 NULL  
          NULL,                                    同上  
          NULL))                                   不是流加密，这个参数为 NULL  
{  
    printf("The message to be encoded has been opened. \\n");  
}  
else  
{  
     MyHandleError("OpenToEncode failed");  
}  
if(CryptMsgUpdate(                              CryptMsgUpdate 将数据加到消息中，可以通过循环，将数据一段段的加得到消息中  
        hMsg,                                  一个小心句柄  
        pbContent,                              指向数据的指针  
        cbContent,                               数据的大小  
        TRUE))                                 TRUE 表明这个是最后一段数据，在开个消息的时候，如果 CMSG\_DETACHED\_FLAG 有使用到，这设为 FALSE，否则为 TRUE。  
{  
     printf("Content has been added to the encoded message. \\n");  
}  
else  
{  
      MyHandleError("MsgUpdate failed");  
}  
  
if(CryptMsgGetParam(                             CryptMsgGetParam 是获取一个消息中的参数  
               hMsg,                          一个消息句柄  
               CMSG\_BARE\_CONTENT\_PARAM,  指定要获取的参数的类型  
               0,                          
               pbEncodedBlob,                   一个接受数据的内存地址  
               &cbEncodedBlob))                 BLOB 的大小，即是上面接受的数据的大小  
{  
    printf("Message encoded successfully. \\n");  
}  
else  
{  
      MyHandleError("MsgGetParam failed");  
}  
                                                释放消息句柄  
if(hMsg)  
    CryptMsgClose(hMsg);  
  
if(hMsg = CryptMsgOpenToDecode(                  开个 Decode 的小心句柄，参数和上面的 Encode 一样  
               MY\_ENCODING\_TYPE,        
               0,                                     
 CMSG\_DATA,               
               NULL,                    
               NULL,                   
               NULL))                  
{  
     printf("The message to decode is open. \\n");  
}  
else  
{  
    MyHandleError("OpenToDecode failed");  
}  
下面的过程和 Encode 类似，调用的函数和上面相同，只不过是过程逆向  
  
printf("\\nThe length of the encoded message is %d.\\n\\n",  
    cbEncodedBlob);  
  
if(CryptMsgUpdate(  
    hMsg,                 // Handle to the message  
    pbEncodedBlob,        // Pointer to the encoded BLOB  
    cbEncodedBlob,        // Size of the encoded BLOB  
    TRUE))                // Last call  
{  
      printf("The encoded BLOB has been added to the message. \\n");  
}  
else  
{  
    MyHandleError("Decode MsgUpdate failed");  
}  
if(CryptMsgGetParam(                            CryptMsgGetParam 的调用和上面有所不同，这里一共是调用两次，第一次的作用主要是得到消息的大小，第二次是得到消息所在的内存地址  
                  hMsg,                         消息句柄  
                  CMSG\_CONTENT\_PARAM,    // Parameter type  
                  0,                       
                  NULL,                  // Address for returned   
                                         // information  
                  &cbDecoded))           // Size of the returned  
                                         // information  
{  
    printf("The decoded message size is %d. \\n", cbDecoded);  
}  
else  
{  
    MyHandleError("Decode CMSG\_CONTENT\_PARAM failed");  
}  
if(pbDecoded = (BYTE \*) malloc(cbDecoded))  
{  
     printf("Memory has been allocated for the decoded message.\\n");  
}  
else  
{  
    MyHandleError("Decoding memory allocation failed.");  
}  
if(CryptMsgGetParam(  
                  hMsg,                  // Handle to the message  
                  CMSG\_CONTENT\_PARAM,    // Parameter type  
                  0,                     // Index  
                  pbDecoded,             // Address for returned   
                                         // information  
                  &cbDecoded))           // Size of the returned   
                                         // information  
{  
     printf("The message is %s.\\n",(LPSTR)pbDecoded);  
}  
else  
{  
     MyHandleError("Decode CMSG\_CONTENT\_PARAM #2 failed");  
}  
if(pbEncodedBlob)  
   free(pbEncodedBlob);  
if(pbDecoded)  
   free(pbDecoded);  
if(hMsg)  
    CryptMsgClose(hMsg);  
  
printf("This program ran to completion without error. \\n");  
  
} //  End of main  
  
下面我们来看看如何哈希一个对话密钥，这个密钥可以用来对一个消息，文件进行加密。我们依旧从一个程序开始。  
#include <stdio.h>  
#include <windows.h>  
#include <Wincrypt.h>  
#define MY\_ENCODING\_TYPE  (PKCS\_7\_ASN\_ENCODING | X509\_ASN\_ENCODING)  
void MyHandleError(char \*s);  
  
void main()  
{  
  
//--------------------------------------------------------------------  
// Copyright (c) Microsoft Corporation.  All rights reserved.  
// Declare variables.  
  
HCRYPTPROV hCryptProv;  
HCRYPTHASH hHash;  
HCRYPTKEY hKey;  
  
//--------------------------------------------------------------------  
// Begin processing.  
  
printf("Process beginning. Creating a session key. \\n");  
  
if(CryptAcquireContext(                           首先依旧是获取一个缺省的 CSP 句柄  
   &hCryptProv,   
   NULL,   
   NULL,   
   PROV\_RSA\_FULL,   
   0))   
{  
    printf("CryptAcquireContext complete. \\n");  
}  
else  
{  
     MyHandleError("Acquisition of context failed.");  
}  
  
if(CryptCreateHash(                                 创建一个 CALG\_MD5 算法的 HASH 对象，这个 hash 对象用的是 MD5 算法  
   hCryptProv,                                        指定一个 CSP 句柄  
   CALG\_MD5,                                      指定算法  
   0,                 
   0,   
   &hHash))   
{  
    printf("An empty hash object has been created. \\n");  
}  
else  
{  
    MyHandleError("Error during CryptBeginHash!\\n");  
}  
  
if(CryptGenKey(                                        创建密钥  
   hCryptProv,                                         传入一个 CSP 句柄  
   CALG\_RC2,                                        指明密钥身成所用算法  
   CRYPT\_EXPORTABLE,                              说明密钥是可以导出到 CSP，用于这个应用程序外的  
   &hKey))   
{  
     printf("A random session key has been created. \\n");  
}  
else  
{  
    MyHandleError("Error during CryptGenKey!\\n");  
}  
if(CryptHashSessionKey(                                   对生成的密钥进行 hash  
   hHash,   
   hKey,   
   0))  
{  
     printf("The session key has been hashed. \\n");  
}  
else  
{  
    MyHandleError("Error during CryptHashSessionKey!\\n");  
}  
在这里就可以添加代码，用生成的密钥进行加密  
  
if(hHash)  
{  
   if(!(CryptDestroyHash(hHash)))  
       MyHandleError("Error during CryptDestroyHash");  
}  
  
  
if(hKey)  
{  
   if(!(CryptDestroyKey(hKey)))  
       MyHandleError("Error during CryptDestroyKey");  
}  
  
// Release the CSP.  
  
if(hCryptProv)  
{  
   if(!(CryptReleaseContext(hCryptProv,0)))  
       MyHandleError("Error during CryptReleaseContext");  
}  
  
printf("Create random session key completed without error. \\n");  
} // end main  
附件是 DOC 文档，希望大家多提宝贵意见 

 4dd466d0e1e0cc123afbe504da2462ca
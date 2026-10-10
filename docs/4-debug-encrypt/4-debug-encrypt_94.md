

# openssl RSA 加密

> 原文：[`www.pediy.com/kssd/pediy11/111070.html`](https://www.pediy.com/kssd/pediy11/111070.html)

初学 openssl,大牛飘过  
程序运行过程  
1.生成密钥对,保存密钥对  
  
2.读取密钥对  
  
3.用经过读取获得的密钥对对数据进行加解密(公钥加密私钥解密)  
  
#include <openssl/rsa.h>  
#include <openssl/pem.h>  
#include<openssl/evp.h>  
#include <string.h>  
EVP\_PKEY\* open();  
EVP\_PKEY\* Popen();  
void print(const char \*promptStr,unsigned char \* data,int len);  
void generate\_key();  
void  main(){   
          
  EVP\_PKEY\* pkey=NULL,\*pkey1=NULL;  
        generate\_key();  
        pkey1=Popen();  
  if(pkey1==NULL)  
  {    
            printf("p 加密失败\\n");  
      exit(-1);  
  }  
      pkey=open();  
  if(pkey==NULL)  
  {    
  exit(-1);  
  }  
     int len=-1;  
     char plainText\[\]="\[For test to public/private key encryption/decryption\]";  
     char encData\[512\];  
      char decData\[512\];  
     OpenSSL\_add\_all\_ciphers();  
     len=EVP\_PKEY\_encrypt(  
       (unsigned char \*)encData,  
             (const unsigned char \*)plainText,  
          sizeof(plainText),  
          pkey1  
      );  
        if(len==-1)  
  {  
        printf("EVP\_PKEY\_encrypt 加密失败\\n");  
      exit(-1);  
  }  
        print("加密后的数据",(unsigned char \*)encData,len);  
        len=EVP\_PKEY\_decrypt(  
           (unsigned char \*)decData,  
                (const unsigned char \*)encData,  
             len,  
          pkey  
      );  
      if(len==-1)  
  {  
      printf("EVP\_PKEY\_decrypt 解密失败\\n");  
        exit(-1);  
  }  
      print("解密后的数据",(unsigned char \*)decData,len);  
     printf("\\n 明文是:\[长度=%d 字节\]:%s\\n",len,decData);  
     EVP\_PKEY\_free(pkey);  
}  
void generate\_key()  
{   
     RSA \*rsa=NULL;  
        printf("正在产生 RSA 密钥...\\n");  
        rsa = RSA\_generate\_key(1024, RSA\_F4, NULL, NULL);  
        if(rsa==NULL)  
        {  
                printf("gen rsa error\\n");  
                 exit(-1);  
        }  
        // 公钥  
        BIO \*bp = BIO\_new(BIO\_s\_file());  
        if(BIO\_write\_filename(bp, "E:\\\\毕业设计\\\\加密成功 2\\\\public.pem")<=0)  
        {  
                printf("write error\\n");  
                exit(-1);  
        }  
        if(PEM\_write\_bio\_RSAPublicKey(bp, rsa)!=1)  
        {  
                printf("write public key error\\n");  
                exit(-1);  
        }    
  printf("保存公钥成功\\n");  
        BIO\_free\_all(bp);  
          
        char passwd\[\]="1234";  
        // 私钥  
        bp = BIO\_new\_file("E:\\\\毕业设计\\\\加密成功 2\\\\private.pem", "w+");  
        if(PEM\_write\_bio\_RSAPrivateKey(bp, rsa, EVP\_des\_ede3(), (unsigned char\*)passwd, 4, NULL, NULL)!=1)  
        {  
                printf("write public key error\\n");  
                exit(-1);  
        }  
  printf("保存私钥成功\\n");  
        BIO\_free\_all(bp);   
  
}  
EVP\_PKEY\* open()  
{  
  EVP\_PKEY\* pkey=NULL;  
  RSA \*rsaK;  
  OpenSSL\_add\_all\_algorithms();  
  BIO \*BP=BIO\_new(BIO\_s\_file());;  
  char FileName\[\]="E:\\\\毕业设计\\\\加密成功 2\\\\private.pem";  
  BIO\_read\_filename(BP,FileName);   
  char PSW\[\]="1234";  
  rsaK=PEM\_read\_bio\_RSAPrivateKey(BP,NULL,NULL,PSW);  
  if(rsaK==NULL)  
  {  
      printf("读取失败!\\n");  
      BIO\_free(BP);  
      RSA\_free(rsaK);  
      exit(-1);  
  }  
  printf("私钥获取成功\\n");  
  pkey=EVP\_PKEY\_new();  
   if(NULL==pkey)  
   {  
    printf("EVP\_PKEY\_new failed\\n");  
    RSA\_free(rsaK);  
    return NULL;  
   }  
   EVP\_PKEY\_assign\_RSA(pkey,rsaK);  
   return pkey;  
}  
  
EVP\_PKEY\* Popen()  
{  
  EVP\_PKEY\* pkey=NULL;  
  RSA \*rsaK;  
  OpenSSL\_add\_all\_algorithms();  
  BIO \*BP=BIO\_new(BIO\_s\_file());;  
  char FileName\[\]="E:\\\\毕业设计\\\\加密成功 2\\\\public.pem";  
  BIO\_read\_filename(BP,FileName);   
  rsaK=PEM\_read\_bio\_RSAPublicKey(BP,NULL,NULL,NULL);  
  if(rsaK==NULL)  
  {  
      printf("读取失败!\\n");  
      BIO\_free(BP);  
      RSA\_free(rsaK);  
      exit(-1);  
  }  
  printf("私钥获取成功\\n");  
  pkey=EVP\_PKEY\_new();  
   if(NULL==pkey)  
   {  
    printf("EVP\_PKEY\_new failed\\n");  
    RSA\_free(rsaK);  
    return NULL;  
   }  
   EVP\_PKEY\_assign\_RSA(pkey,rsaK);  
   return pkey;  
}  
  
void print(const char \*promptStr,unsigned char \* data,int len)  
{  
 int i;  
 printf("\\n===%s\[长度=%d 字节\]======\\n",promptStr,len);  
 for(i=0;i<len;i++)printf("%02x",data\[i\]);  
 printf("\\n======================================\\n");  
}
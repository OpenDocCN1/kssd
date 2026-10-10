
# 第一阶段 第三题 keygen 代码(Ultimate)

> 原文：[`www.pediy.com/kssd/pediy11/123554.html`](https://www.pediy.com/kssd/pediy11/123554.html)

代码:

```
#include <windows.h>

void Gen(char \*szName,char \*szCode)
{
  typedef struct \_SHA\_CTX
  {
    DWORD Unknown\[6\];
    DWORD State\[5\];
    DWORD Count\[2\];
    BYTE Buffer\[64\];
  } SHA\_CTX, \*PSHA\_CTX;
  typedef void (WINAPI\* PSHAInit)(SHA\_CTX \*ctx);
  typedef void (WINAPI\* PSHAUpdate)(SHA\_CTX \*ctx,BYTE \*buf,DWORD len);
  typedef void (WINAPI\* PSHAFinal)(SHA\_CTX \*ctx,BYTE \*res);

  char szTmp\[0x200\];
  SHA\_CTX context;
  BYTE digest\[0x20\];
    
  wsprintf(szTmp,"%sf\*\*\*Tencent",szName);
  GetVolumeInformation("C:\\\\",NULL,NULL,(DWORD \*)&szTmp\[strlen(szName)\],NULL,NULL,NULL,NULL);

  HINSTANCE hDLL = LoadLibrary("advapi32.dll");
  PSHAInit SHAInit = (PSHAInit)GetProcAddress(hDLL,"A\_SHAInit");
  PSHAUpdate SHAUpdate = (PSHAUpdate)GetProcAddress(hDLL,"A\_SHAUpdate");
  PSHAFinal SHAFinal = (PSHAFinal)GetProcAddress(hDLL,"A\_SHAFinal");

  SHAInit(&context);
  context.State\[0\] = 0xB1CAB1CA;
  context.State\[1\] = 0xCCBFCCBF;
  context.State\[2\] = 0xBFB2D6BE;
  context.State\[3\] = 0xF8C7D8B5;
  context.State\[4\] = 0xEEC7BCCD;
  SHAUpdate(&context,(BYTE \*)szTmp,strlen(szName)+11);
  SHAFinal(&context,digest);

  UINT64 t;
  int i,j;
  char \*ta = "ABCDEFGHJKMNPQRSTVWXYZ1234567890";
  for (i=0;i<4;i++)
  {
    t = 0;
    for (j=0;j<5;j++)
    {
      t <<= 8;
      t += digest\[i\*5+j\];
    }
    for (j=0;j<8;j++)
    {
      szCode\[i\*9+j\] = ta\[(\*((DWORD \*)&t+1)>>3)&0x1F\];
      t <<= 5;
    }
    szCode\[i\*9+8\] = '-';
  }
  szCode\[0x23\] = 0;
}
```

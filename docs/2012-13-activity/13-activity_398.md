
# 我也发个第 2 阶段第 2 题答案

> 原文：[`www.pediy.com/kssd/pediy11/124314.html`](https://www.pediy.com/kssd/pediy11/124314.html)

```cpp
void GetHash(const char* pszUserName, BYTE* pHash)
{
  Integer a(-3);
  Integer modulus("C564EEF070E69193h");
  Integer b("22996B9C33AEEFDBH");
  ECP ecp(modulus, a,b);
  Integer var_378("1C4EEE9222DDDA62h");
  Integer var_328("2223A1D595845FA2h");
  ECP::Point base(var_328, var_378);
  Integer var_364("880198E3724F9FEEh");
  Integer var_350("297A4A1E5B1FC99Bh");
  ECP::Point y(var_350, var_364);    //pub-key
  Integer var_33C("C564EEF19A080B07h");    //n
  Integer d("54656E63656E7420H");    //ecc 密码是 "Tencent "
  ECDSA<ECP, SHA>::Signer priv(ecp, base, var_33C, d);
  AutoSeededRandomPool rng;
  priv.SignMessage(rng, (BYTE*)pszUserName, strlen(pszUserName), pHash);
}
```

```cpp
void Base32Convert(BYTE* pHash, BYTE* pData)
{
  int i;
  int j;
  BYTE bits[20*8];
  for(i=0; i<20; i++)
  {
    for(j=0; j<8; j++)
    {
      bits[i*8+7-j] = (pHash[i]>>j)&1;
    }
  }
  for(i=0; i<32; i++)
  {
    for(j=0; j<5; j++)
    {
      pData[i] |= bits[i*5+j] << (4-j);
    }
  }
}
```

```cpp
void DataToCode(BYTE* pData, char* pszCode)
{
  char* pstr = "ABCDEFGHJKMNPQRSTVWXYZ1234567890";
  int i;
  for(i=0; i<32; i++)
  {
    pszCode[i] = pstr[pData[i]];
  }
}
```

```cpp
void Keygen(const char* pszUserName, char* pszLicenseCode)
{
  BYTE cbHash[32];
  BYTE cbData[32];
  memset(cbHash, 0, sizeof(cbHash));
  memset(cbData, 0, sizeof(cbData));
  //ecdsa
  GetHash(pszUserName, cbHash);
  //blowfish
  UINT* ptmp = (UINT*)cbHash;
  *(UINT*)(cbData+0x10) = ptmp[0] ^ ptmp[1] ^ ptmp[2] ^ ptmp[3];
  ptmp[0] ^= *(UINT*)(cbData+0x10);
  BF_KEY key;
  My_BF_set_key(&key, 8,(BYTE*)"DEADBEEF");
  My_BF_decrypt(ptmp, &key);
  ptmp[2] ^= ptmp[0];
  ptmp[3] ^= ptmp[1];
  My_BF_decrypt(ptmp+2, &key);
  //置换
  int i;
  for(i=0; i<16; i++)
  {
    cbHash[i] = gtable[cbHash[i]];
  }
  for(i=0; i<4; i++)
  {
    cbData[0x10+i] = gtable[cbData[0x10+i]];
  }
  //SMS4
    INT32U rk[32];
    SMS4_KeyExpansion ((ULONG*)"Security@Tencent", rk);
  SMS4_Encryption((ULONG*)cbHash, rk, (ULONG*)cbData);
  //base32
  BYTE sndata[32] = {0};
  Base32Convert(cbData, sndata);
  char szCode[64] = {0};
  DataToCode(sndata, szCode);
  pszLicenseCode[8] = pszLicenseCode[0x11] = pszLicenseCode[0x1a] = '-';
  memcpy(pszLicenseCode, szCode, 8);
  memcpy(pszLicenseCode+9, szCode+8, 8);
  memcpy(pszLicenseCode+0x12, szCode+16, 8);
  memcpy(pszLicenseCode+0x1b, szCode+24, 8);
}
```

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `keygen_tencent.rar` |

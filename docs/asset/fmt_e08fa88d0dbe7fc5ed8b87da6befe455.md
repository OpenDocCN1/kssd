
```cpp
#include <iostream>

using namespace std;

typedef unsigned long DWORD;
typedef unsigned short WORD;

#pragma warning( disable : 4267 4018 4312 4311 4996)

typedef struct
{
  int nUnknow_1;
  int nUnknow_2;
  int nUnknow_3;
  int nUnknow_4;
  int nUnknow_5;
  int nUnknow_6;
  char szBuffer[64];
}tagKey;

int Account_Func_10(int nEax, int nEdx, int nEcx)
{
  return (~nEcx | nEax) ^ nEdx;
}

int Account_Func_8(int nEax, int nEdx, int nEcx)
{
  return nEdx ^ nEax ^ nEcx;
}

int Account_Func_6(int nEax, int nEdx, int nEcx)
{
  return ~nEcx & nEdx | nEcx & nEax;
}

void Account_Func_4(int nEax, char cEdx)
{
  *(DWORD*)nEax = (*(DWORD*)nEax << cEdx) | (*(DWORD*)nEax >> (32 - cEdx));
}

int Account_Func_3(int nEax, int nEdx, int nEcx)
{
  return ~nEax & nEcx | nEax & nEdx;
}

void Account_Func_9(unsigned int n, char m, unsigned int i, unsigned int j, int* nEax, int nEcx, int nEdx)
{
  *(DWORD*)nEax += n + i + Account_Func_10(nEdx, nEcx, j);
  Account_Func_4( (int)nEax, m);
  *nEax += nEdx;
}

void Account_Func_7(unsigned int n, char m, unsigned int i, unsigned int j, int* nEax, int nEcx, int nEdx)
{
  *(DWORD*)nEax += n + i + Account_Func_8(nEdx, nEcx, j);
  Account_Func_4( (int)nEax, m);
  *nEax += nEdx;
}

void Account_Func_5(unsigned int n, char m, unsigned int i, unsigned int j, int* nEax, int nEcx, int nEdx)
{
  *(DWORD*)nEax += n + i + Account_Func_6(nEdx, nEcx, j);
  Account_Func_4( (int)nEax, m);
  *nEax += nEdx;
}

void Account_Func_2(unsigned int n, char m, unsigned int i, unsigned int j, int* nEax, int nEcx, int nEdx)
{
  *nEax += i + n + Account_Func_3( nEdx, nEcx, j);
  Account_Func_4( (int)nEax, m);
  *nEax += nEdx;
}

void Account_Func_1(int* pEax, int* pEdx, int nEcx)
{
  int i, v4;
  for ( i = nEcx >> 2; i; --i )
  {
    *(DWORD*)pEdx = *(unsigned char*)pEax;
    v4 = (int)pEax + 1;
    *(DWORD*)pEdx |= *(unsigned char*)v4 << 8;
    v4++;
    *(DWORD*)pEdx |= *(unsigned char*)v4 << 16;
    v4++;
    *(DWORD*)pEdx |= *(unsigned char*)v4 << 24;
    pEax = (int*)(v4 + 1);
    //edx += 4;
    pEdx++;
  }

}

void Struct_Account(char* pEdx, char* pEax)
{
  int nAry[22] = {0};
  nAry[1] = (int)pEdx;
  nAry[0] = (int)pEax;
  Account_Func_1((int*)nAry[0], (int*)&nAry[6],0x40);

  nAry[2] = *(int*)pEdx;
  nAry[3] = *(int*)(pEdx + 4);
  nAry[4] = *(int*)(pEdx + 8);
  nAry[5] = *(int*)(pEdx + 0xC);

  Account_Func_2( 0x0D76AA478, 7, nAry[6], nAry[5], &nAry[2], nAry[4], nAry[3]);

  Account_Func_2( 0x0E8C7B756, 0xC, nAry[7], nAry[4], &nAry[5], nAry[3], nAry[2]);

  Account_Func_2( 0x242070DB, 0x11, nAry[8], nAry[3], &nAry[4], nAry[2], nAry[5]);

  Account_Func_2( 0xC1BDCEEE, 0x16, nAry[9], nAry[2], &nAry[3], nAry[5], nAry[4]);

  Account_Func_2( 0xF57C0FAF, 7, nAry[10], nAry[5], &nAry[2], nAry[4], nAry[3]);

  Account_Func_2( 0x4787C62A, 0xC, nAry[11], nAry[4], &nAry[5], nAry[3], nAry[2]);

  Account_Func_2( 0xA8304613, 0x11, nAry[12], nAry[3], &nAry[4], nAry[2], nAry[5]);

  Account_Func_2( 0xFD469501, 0x16, nAry[13], nAry[2], &nAry[3], nAry[5], nAry[4]);

  Account_Func_2( 0x698098D8, 7, nAry[14], nAry[5], &nAry[2], nAry[4], nAry[3]);

  Account_Func_2( 0x8B44F7AF, 0xC, nAry[15], nAry[4], &nAry[5], nAry[3], nAry[2]);

  Account_Func_2( 0xFFFF5BB1, 0x11, nAry[16], nAry[3], &nAry[4], nAry[2], nAry[5]);

  Account_Func_2( 0x895CD7BE, 0x16, nAry[17], nAry[2], &nAry[3], nAry[5], nAry[4]);

  Account_Func_2( 0x6B901122, 7, nAry[18], nAry[5], &nAry[2], nAry[4], nAry[3]);

  Account_Func_2( 0xFD987193, 0xC, nAry[19], nAry[4], &nAry[5], nAry[3], nAry[2]);

  Account_Func_2( 0xA679438E, 0x11, nAry[20], nAry[3], &nAry[4], nAry[2], nAry[5]);

  Account_Func_2( 0x49B40821, 0x16, nAry[21], nAry[2], &nAry[3], nAry[5], nAry[4]);

  //////////////////////////////////////////////////////////////////////

  Account_Func_5( 0xF61E2562, 5, nAry[7], nAry[5], &nAry[2], nAry[4], nAry[3]);

  Account_Func_5( 0xC040B340, 9, nAry[12], nAry[4], &nAry[5], nAry[3], nAry[2]);

  Account_Func_5( 0x265E5A51, 0xE, nAry[17], nAry[3], &nAry[4], nAry[2], nAry[5]);

  Account_Func_5( 0xE9B6C7AA, 0x14, nAry[6], nAry[2], &nAry[3], nAry[5], nAry[4]);

  Account_Func_5( 0xD62F105D, 5, nAry[11], nAry[5], &nAry[2], nAry[4], nAry[3]);

  Account_Func_5( 0x2441453, 9, nAry[16], nAry[4], &nAry[5], nAry[3], nAry[2]);

  Account_Func_5( 0xD8A1E681, 0xE, nAry[21], nAry[3], &nAry[4], nAry[2], nAry[5]);

  Account_Func_5( 0xE7D3FBC8, 0x14, nAry[10], nAry[2], &nAry[3], nAry[5], nAry[4]);

  Account_Func_5( 0x21E1CDE6, 5, nAry[15], nAry[5], &nAry[2], nAry[4], nAry[3]);

  Account_Func_5( 0xC33707D6, 9, nAry[20], nAry[4], &nAry[5], nAry[3], nAry[2]);

  Account_Func_5( 0xF4D50D87, 0xE, nAry[9], nAry[3], &nAry[4], nAry[2], nAry[5]);

  Account_Func_5( 0x455A14ED, 0x14, nAry[14], nAry[2], &nAry[3], nAry[5], nAry[4]);

  Account_Func_5( 0xA9E3E905, 5, nAry[19], nAry[5], &nAry[2], nAry[4], nAry[3]);

  Account_Func_5( 0xFCEFA3F8, 9, nAry[8], nAry[4], &nAry[5], nAry[3], nAry[2]);

  Account_Func_5( 0x676F02D9, 0xE, nAry[13], nAry[3], &nAry[4], nAry[2], nAry[5]);

  Account_Func_5( 0x8D2A4C8A, 0x14, nAry[18], nAry[2], &nAry[3], nAry[5], nAry[4]);

  //////////////////////////////////////////////////////////////////////

  Account_Func_7( 0xFFFA3942, 4, nAry[11], nAry[5], &nAry[2], nAry[4], nAry[3]);

  Account_Func_7( 0x8771F681, 0xB, nAry[14], nAry[4], &nAry[5], nAry[3], nAry[2]);

  Account_Func_7( 0x6D9D6122, 0x10, nAry[17], nAry[3], &nAry[4], nAry[2], nAry[5]);

  Account_Func_7( 0xFDE5380C, 0x17, nAry[20], nAry[2], &nAry[3], nAry[5], nAry[4]);

  Account_Func_7( 0xA4BEEA44, 4, nAry[7], nAry[5], &nAry[2], nAry[4], nAry[3]);

  Account_Func_7( 0x4BDECFA9, 0xB, nAry[10], nAry[4], &nAry[5], nAry[3], nAry[2]);

  Account_Func_7( 0xF6BB4B60, 0x10, nAry[13], nAry[3], &nAry[4], nAry[2], nAry[5]);

  Account_Func_7( 0xBEBFBC70, 0x17, nAry[16], nAry[2], &nAry[3], nAry[5], nAry[4]);

  Account_Func_7( 0x289B7EC6, 4, nAry[19], nAry[5], &nAry[2], nAry[4], nAry[3]);

  Account_Func_7( 0xEAA127FA, 0xB, nAry[6], nAry[4], &nAry[5], nAry[3], nAry[2]);

  Account_Func_7( 0xD4EF3085, 0x10, nAry[9], nAry[3], &nAry[4], nAry[2], nAry[5]);

  Account_Func_7( 0x4881D05, 0x17, nAry[12], nAry[2], &nAry[3], nAry[5], nAry[4]);

  Account_Func_7( 0xD9D4D039, 4, nAry[15], nAry[5], &nAry[2], nAry[4], nAry[3]);

  Account_Func_7( 0xE6DB99E5, 0xB, nAry[18], nAry[4], &nAry[5], nAry[3], nAry[2]);

  Account_Func_7( 0x1FA27CF8, 0x10, nAry[21], nAry[3], &nAry[4], nAry[2], nAry[5]);

  Account_Func_7( 0xC4AC5665, 0x17, nAry[8], nAry[2], &nAry[3], nAry[5], nAry[4]);

  //////////////////////////////////////////////////////////////////////

  Account_Func_9( 0xF4292244, 6, nAry[6], nAry[5], &nAry[2], nAry[4], nAry[3]);

  Account_Func_9( 0x432AFF97, 0xA, nAry[13], nAry[4], &nAry[5], nAry[3], nAry[2]);

  Account_Func_9( 0xAB9423A7, 0xF, nAry[20], nAry[3], &nAry[4], nAry[2], nAry[5]);

  Account_Func_9( 0xFC93A039, 0x15, nAry[11], nAry[2], &nAry[3], nAry[5], nAry[4]);

  Account_Func_9( 0x655B59C3, 6, nAry[18], nAry[5], &nAry[2], nAry[4], nAry[3]);

  Account_Func_9( 0x8F0CCC92, 0xA, nAry[9], nAry[4], &nAry[5], nAry[3], nAry[2]);

  Account_Func_9( 0xFFEFF47D, 0xF, nAry[16], nAry[3], &nAry[4], nAry[2], nAry[5]);

  Account_Func_9( 0x85845DD1, 0x15, nAry[7], nAry[2], &nAry[3], nAry[5], nAry[4]);

  Account_Func_9( 0x6FA87E4F, 6, nAry[14],  nAry[5], &nAry[2], nAry[4], nAry[3]);

  Account_Func_9( 0xFE2CE6E0, 0xA, nAry[21], nAry[4], &nAry[5], nAry[3], nAry[2]);

  Account_Func_9( 0xA3014314, 0xF, nAry[12], nAry[3], &nAry[4], nAry[2], nAry[5]);

  Account_Func_9( 0x4E0811A1, 0x15, nAry[19], nAry[2], &nAry[3], nAry[5], nAry[4]);

  Account_Func_9( 0xF7537E82, 6, nAry[10], nAry[5], &nAry[2], nAry[4], nAry[3]);

  Account_Func_9( 0xBD3AF235, 0xA, nAry[17], nAry[4], &nAry[5], nAry[3], nAry[2]);

  Account_Func_9( 0x2AD7D2BB, 0xF, nAry[8], nAry[3], &nAry[4], nAry[2], nAry[5]);

  Account_Func_9( 0xEB86D391, 0x15, nAry[15], nAry[2], &nAry[3], nAry[5], nAry[4]);

  //////////////////////////////////////////////////////////////////////

  *(DWORD*)pEdx += nAry[2];
  *(DWORD*)(pEdx + 4) += nAry[3];
  *(DWORD*)(pEdx + 8) += nAry[4];
  *(DWORD*)(pEdx + 0xC) += nAry[5];
}

void Struct_Edit_2(int nEax, int nEdx, int nEcx)
{
  int i;
  DWORD dwTemp;
  for ( i = nEcx; i != 0; --i )
  {
    *(char*)nEdx = *(char*)nEax;

    nEdx += 1;
    dwTemp = *(DWORD*)nEax;
    dwTemp >>= 8;
    *(char*)nEdx = char(dwTemp);

    nEdx += 1;
    dwTemp = *(DWORD*)nEax;
    dwTemp >>= 16;
    *(char*)nEdx = char(dwTemp);

    nEdx += 1;
    dwTemp = *(DWORD*)nEax;
    dwTemp >>= 24;
    *(char*)nEdx = char(dwTemp);

    nEax += 4;

  }
}

//pv1 = eax, pv2 = edx, nLenth = ecx
void Struct_Edit_1(char* pv1, char* pv2,int nLenth)
{
  int v2 = (*(DWORD*)(pv1 + 16) >> 3) & 0x3F;
  *(DWORD*)(pv1 + 16) += 8 * nLenth;
  if ( 8 * nLenth > *(DWORD*)(pv1 + 16) )
    ++*(DWORD*)(pv1 + 20);
  *(DWORD*)(pv1 + 20) += nLenth >> 29;
  int v5 = 64 - v2;
  if ( v5 <= nLenth )
  {
    memcpy((void*)(pv1 + v2 + 0x18),pv2,v5);
    Struct_Account(pv1, pv1+0x18);
    while ( nLenth > v5 + 63 )
    {
      Struct_Account(pv1, pv2 + v5);
      //ary_account();
      v5 += 64;
    }
    v2 = 0;
  }
  else
  {
    v5 = 0;
  }
  memcpy(pv1+v2+0x18, pv2+v5, nLenth - v5);

}

int g_nAry[64] = {0x80};

void Matrix_Account(tagKey& pSt,int* pAry)
{
  int v7[2] = {0};

  int v4 = (int)pAry;
  int v3 = (int)&pSt;
  Struct_Edit_2((int)&pSt.nUnknow_5, (int)&v7, 2);
  int v2 = (*(DWORD*)(&pSt.nUnknow_5) >> 3) & 0x3F;
  unsigned int v5;
  if ( (unsigned int)v2 >= 0x38 )
    v5 = 120 - v2;
  else
    v5 = 56 - v2;
  Struct_Edit_1((char*)v3, (char*)&g_nAry, v5);
  Struct_Edit_1((char*)v3, (char*)&v7, 8u);
  Struct_Edit_2(v3, v4, 4);
}

//第三次 key 运算
void Account_Key_3(string& sRealKey, string sKey)
{
  string sTempKey;
  int nLenth = sKey.length();
  int n;
  for ( n = 0; n < nLenth; n++)
  {
    switch (sKey[n])
    {
    case '0':
    case '1':
    case '2':
    case '3':
    case '4':
    case '5':
    case '6':
    case '7':
    case '8':
    case '9':
      sTempKey += sKey[n];
      break;
    default:
      break;
    }
  }

  nLenth = sTempKey.length();
  if ( nLenth > 0)
  {
    int nTemp = 1;
    char* pTempKey = (char*)sTempKey.c_str();
    do 
    {
      if ( (nTemp % 2) == 0 )
      {
        char c = *(char*)(pTempKey + nTemp - 1);
        sRealKey += c;
      }
      nTemp++;
      nLenth--;
    } while (nLenth);
  }
}

void Account_Key_2(string& sKey, char* pAry)
{
   
  for (int n = 0; n < 16; n++)
  {
    char buffer[5] = {0};
    sprintf_s(buffer,"%02x",unsigned char(pAry[n]));
    sKey += buffer;
  }
}

//第一次 key 运算
void Account_Key_1(string sId, int* pAry)
{
  //初始化结构体
  tagKey stKey = {0x61234509,0xEFCD5B89,0x13BA7CFE,0x18325476,0,0,""};
  int nLenth = sId.length();
  Struct_Edit_1((char*)&stKey,(char*)sId.c_str(),nLenth);
  Matrix_Account(stKey,pAry);
  memcpy(pAry, &stKey, 16);

}

void Account_Key(string sId,string& sRealKey)
{
  int nAry[16] = {0};
  Account_Key_1(sId, nAry);
  string si2a;
  Account_Key_2(si2a,(char*)&nAry);
  Account_Key_3(sRealKey,si2a);
}

int _tmain(int argc, _TCHAR* argv[])
{
  cout << "please enter machine id:";

  char szId[40] ={0};
  cin >> szId;
  string sId = szId;
  string sRealKey;
  try
  {
    Account_Key(sId,sRealKey);
    cout << '\n' << "SN:" << sRealKey.c_str() << '\n';
  }
  catch(...)
  {
    cout << "error" << endl;
  }
  return 0;
}

```

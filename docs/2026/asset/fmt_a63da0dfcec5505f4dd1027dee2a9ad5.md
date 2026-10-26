
```
00543B69   .  50          push eax

00543B6A   .  8B45 FC     mov eax,dword ptr ss:[ebp-0x4]

00543B6D   .  8B40 74     mov eax,dword ptr ds:[eax+0x74]

00543B70   .  50          push eax

00543B71   .  E8 6EE9FFFF call <Computer.[B]WinHttpQueryDataAvailable[/B]>           ;  jmp 到 WINHTTP.WinHttpQueryDataAvailable



0461FC10   00EF2000

0461FC14   0461FCB0



0471FCB0  00000318  .. [COLOR="blue"]size[/COLOR]



00543BE8   .  50          push eax

00543BE9   .  8B45 F0     mov eax,dword ptr ss:[ebp-0x10]

00543BEC   .  50          push eax

00543BED   .  8B45 E8     mov eax,dword ptr ss:[ebp-0x18]

00543BF0   .  50          push eax

00543BF1   .  8B45 FC     mov eax,dword ptr ss:[ebp-0x4]

00543BF4   .  8B40 74     mov eax,dword ptr ds:[eax+0x74]

00543BF7   .  50          push eax

00543BF8   .  E8 EFE8FFFF call <Computer.[B]WinHttpReadData[/B]>                     ;  jmp 到 WINHTTP.WinHttpReadData



0471FC08   02A42000

0471FC0C   014F8650

0471FC10   00000318

0471FC14   0471FC98



OfpoWeotGXv4mus5ga".)

堆栈 ss:[0471FCA8]=014F8650, (ASCII "DTZinZLmLsFmPP62dsGSMc9Omp/71r5s+4aDhh6rPDB2sNKKMLrMTKpsxkLCsDKGnMa2QEJNij1yFt+gMskkF1q8xN24FF98dPRpnFyr+HT+4LIQ0UhvIrYVA/z+o6u39hyw0S7yNvtnFMoldQ8AfTsRHOdu25KelI/G1M04nDvntTxUQCaec1pXMszF81eenuCLoSWJXiPu4TYi/t3PrsI/heCOfpoWeotGXv4mus5ga".)

edx=00000000
```

### 解密函数
```
00542361  |.  E8 4AFEFFFF   ||call <Computer.?Unit153.sub_005421B0>

00542366  |.  8945 9C       ||mov [local.25],eax

00542369  |.  33C0          ||xor eax,eax

0054236B  |.  837D A4 00    ||cmp [local.23],0x0

0054236F  |.  7E 11         ||jle short Computer.00542382
```

跟进解密函数，简单测试

```
int EntryptedData[]=
{
  0x0000003D,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,
  0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,
  0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,
  0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,
  0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,
  0x00000040,0x00000040,0x00000040,0x0000003E,0x00000040,0x00000040,0x00000040,0x0000003F,
  0x00000034,0x00000035,0x00000036,0x00000037,0x00000038,0x00000039,0x0000003A,0x0000003B,
  0x0000003C,0x0000003D,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,
  0x00000040,0x00000000,0x00000001,0x00000002,0x00000003,0x00000004,0x00000005,0x00000006,
  0x00000007,0x00000008,0x00000009,0x0000000A,0x0000000B,0x0000000C,0x0000000D,0x0000000E,
  0x0000000F,0x00000010,0x00000011,0x00000012,0x00000013,0x00000014,0x00000015,0x00000016,
  0x00000017,0x00000018,0x00000019,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040,
  0x00000040,0x0000001A,0x0000001B,0x0000001C,0x0000001D,0x0000001E,0x0000001F,0x00000020,
  0x00000021,0x00000022,0x00000023,0x00000024,0x00000025,0x00000026,0x00000027,0x00000028,
  0x00000029,0x0000002A,0x0000002B,0x0000002C,0x0000002D,0x0000002E,0x0000002F,0x00000030,
  0x00000031,0x00000032,0x00000033,0x00000040,0x00000040,0x00000040,0x00000040,0x00000040
};



int EntrypedFunAdd(char* key,char* tempOut,int strLen)
{
  char *pkey=key;
  for (unsigned int i=0;i<strLen;i++)
  {
    int TempEsi=0;
    unsigned int tempEdx=(int)*(tempOut+i*4);
    tempEdx=EntryptedData[tempEdx]*4;
    unsigned int tempEdi=(int)*(tempOut+1+i*4);
    unsigned int tempEcx=EntryptedData[tempEdi];
    tempEcx=tempEcx/0x10;
    unsigned char *tempCl=(unsigned char*)&tempEcx;
    unsigned char *tempDl=(unsigned char*)&tempEdx;
    *tempDl=*tempDl|*tempCl;
    *(pkey+i*4)=*tempDl;
    TempEsi++;
    *tempDl=*(tempOut+2+i*4);
    if (*tempDl!=0x3d)
    {
      TempEsi++;
      *tempCl=EntryptedData[tempEdi];
      tempEcx=tempEcx*0x10;
    }
    unsigned int tempEbx=0;
    unsigned char* tempBl=(unsigned char*)&tempEbx;
    *tempBl=*tempDl;
    tempEbx=EntryptedData[tempEbx];
    tempEbx=tempEbx/4;
    *tempCl=*tempCl|*tempBl;
    *(pkey+1+i*4)=*tempCl;
    *tempCl=*(tempOut+3+i*4);
    if (*tempCl!=0x3d)
    {
      TempEsi++;
      tempEdx=tempEdx&0xff;
      *tempDl=EntryptedData[tempEdx];
      tempEdx=tempEdx*0x40;
      unsigned int tempEax=0;
      unsigned char* tempAl=(unsigned char*)&tempEax;
      *tempAl=*tempCl;
      *tempDl=*tempDl|EntryptedData[tempEax];
      *(pkey+2+i*4)=*tempDl;
    }
    pkey--;
  }
  return 0;
}
```

### 测试
```
LPSTR pszOutBuffer;

char OutBuffer[0x1024]={0};

int OutNum=0;

DWORD getWriteSize=0;
while(1)
{
  dwSize = 0;
  if (!WinHttpQueryDataAvailable( hRequest, &dwSize)) 
  {
    printf( "Error %u in WinHttpQueryDataAvailable.\n", GetLastError());
  }
  if (!dwSize) break;
  pszOutBuffer = new char[dwSize+1];
  if (!pszOutBuffer)
  {
    printf("Out of memory \r\n");
  }
  ZeroMemory(pszOutBuffer, dwSize+1);
  if (ReadData(hRequest, pszOutBuffer, dwSize, &getWriteSize) == FALSE) {
    printf("Error:RedaData failed!\n");
  }
  if (getWriteSize <= 0) {
    printf("Error:WriteData failed!\n");
  }
  memcpy(OutBuffer+OutNum,pszOutBuffer,getWriteSize);
  OutNum=OutNum+getWriteSize;
  delete(pszOutBuffer);
  pszOutBuffer=NULL;
}

char* Testkey=NULL;
Testkey = new char[OutNum+1];
ZeroMemory(Testkey, OutNum+1);
EntrypedFunAdd(Testkey,OutBuffer,OutNum/4);
```

**获得结果**
```
0471FC2F  9D62360D  .6b
0471FC33  C12EE692  掓.
0471FC37  B6FE3C66  f<
0471FC3B  3192C176  v翏1
0471FC3F  9F9A4ECF  螻殶
0471FC43  6CBED6FB  緇
0471FC47  868386FB  麊儐
0471FC4B  303CAB1E  ?0
0471FC4F  8AD2B076  v耙
0471FC53  4CCCBA30  0禾L
0471FC57  42C66CAA  猯艬
0471FC5B  8632B0C2  掳2
0471FC5F  40B6C69C  溒禓
0471FC63  3D8A4D42  BM?
0471FC67  00000072  r...
```

...

继续解密
```
int UlrstrLen=OutNum/4*3;
unsigned int arg1=0x64;
for(int i=0;i<0x40;i++)
{
  arg1=arg1*2;
  unsigned int Temp1=(arg1/8)&1;
  unsigned int Temp2=(arg1/0x8000)&1;
  Temp1=Temp1^Temp2;
  Temp2=(arg1/0x800000)&1;
  Temp1=Temp1^Temp2;
  Temp2=(arg1/0x80)&1;
  Temp1=Temp1^Temp2;
  if((Temp1-1)==0) arg1=arg1|1;
}

int TempSet=0;
int TempNum=8;
for(int i=0;i<UlrstrLen;i++)
{
  TempSet=0;
  unsigned char* charTempSet=(unsigned char*)&TempSet;
  for(int j=0;j<TempNum;j++)
  {
    unsigned int Temp1=arg1;
    Temp1=Temp1/0x80000000;
    Temp1=Temp1&1;
    if((Temp1-1)==0) { *charTempSet=*charTempSet|1; }
    TempSet=TempSet*2;
    arg1=arg1*2;
    Temp1=(arg1/8)&1;
    unsigned int Temp2=(arg1/0x8000)&1;
    Temp1=Temp1^Temp2;
    Temp2=(arg1/0x800000)&1;
    Temp1=Temp1^Temp2;
    Temp2=(arg1/0x80)&1;
    Temp1=Temp1^Temp2;
    if((Temp1-1)==0) { arg1=arg1|1; }
  }
  Testkey[i]= (unsigned char)Testkey[i]^(unsigned char)(*charTempSet);
}
```

**最后结果**

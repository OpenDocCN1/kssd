

# 逆向 windows 数字签名验证过程

> 原文：[`www.pediy.com/kssd/pediy12/135090.html`](https://www.pediy.com/kssd/pediy12/135090.html)

临近期末要考试，目前只逆了一个函数，BOOL CryptCATAdminAcquireContext(HCATADMIN \*phCatAdmin,GUID \*pgSubsystem,DWORD dwFlags)，待有时间后再一一逆完。没什么技术含量，高手飘过。

代码:

```
BYTE guidtowstr\[\] = {0x3,0x2,0x1,0x0,0x2d,0x5,0x4,0x2d,0x7,0x6,0x2d,0x8,0x9,0x2d,0xa,0xb,0xc,0xd,0xe,0xf};
WCHAR UnicodeNumber\[\] = L"0123456789ABCDEF";
//FIX ME:THE PATH BELOW SHOULD BE PRODUCED ACCORDING TO YOUR OWN MACHINE!
WCHAR OrigPath\[\] = L"C:\\\\WINDOWS\\\\system32\\\\CatRoot\\\\";
WCHAR OrigPath2\[\] = L"C:\\\\WINDOWS\\\\system32\\\\CatRoot2\\\\";
WCHAR Slash\[\] = L"\\\\";

typedef struct \_CAT\_CONTEXT
{
  int cbSize;
  BOOL UseDefaultGUID;
  PWCHAR pwGUID;
  PWCHAR pwDirectoryPath;
  PWCHAR pwDirectoryPath2;
  int \_20;
  PVOID list1;
  PVOID list2;
  PVOID list3;
  int \_36;
  CRITICAL\_SECTION CriticalSection;
  int \_64;
  int \_68;
  HANDLE hWakeEventHandle;
  HANDLE hCleanWaitObject;
  int \_80;
  int \_84;
}CAT\_CONTEXT,\*PCAT\_CONTEXT;

BOOL CryptCATAdminAcquireContext(HCATADMIN \*phCatAdmin,GUID \*pgSubsystem,DWORD dwFlags);
BOOL CryptCATAdminAcquireContext\_Internal(HCATADMIN \*phCatAdmin,GUID \*pgSubsystem,DWORD dwFlags,int arg);
VOID LIST\_Initialize(PVOID pBegin);
BOOL guid2wstr(GUID \*guid,PWCHAR pwguid);
PWCHAR  \_CatAdminCreatePath(PWCHAR OrigPath,PWCHAR pwGUID,BOOL UseDefaultGUID);
BOOL \_CatAdminRecursiveCreateDirectory(PWCHAR pwDirectoryPath,LPSECURITY\_ATTRIBUTES lpSecurityAttributes);
VOID CALLBACK \_CatAdminWaitOrTimerCallback(PVOID lpParameter,BOOLEAN TimerOrWaitFired);;

BOOL CryptCATAdminAcquireContext(HCATADMIN \*phCatAdmin,GUID \*pgSubsystem,DWORD dwFlags)
{
  return CryptCATAdminAcquireContext\_Internal(phCatAdmin,pgSubsystem,dwFlags,0);
}

BOOL CryptCATAdminAcquireContext\_Internal(HCATADMIN \*phCatAdmin,GUID \*pgSubsystem,DWORD dwFlags,int arg)
{
  WCHAR wGUID\[0x100\];
  GUID DefaultGUID = {0x127d0a1d,0x4ef2,0x11d1,{0x86,0x8,0x0,0xc0,0x4f,0xc2,0x95,0xee}};
  BOOL ReturnValue;
  GUID \*pGUIDToUse;
  HCATADMIN \*\_phCatAdmin;
  GUID \*\_pgSubsystem;
  PCAT\_CONTEXT pCatContext;
  BOOL UseDefaultGUID;

  \_phCatAdmin = phCatAdmin;
  \_pgSubsystem = pgSubsystem;

  pGUIDToUse = &DefaultGUID;
  ReturnValue = 1;
  UseDefaultGUID = TRUE;
  //FIX ME: CHECK WHEN phCatAdmin == NULL!
  \*phCatAdmin = 0;

  pCatContext = (PCAT\_CONTEXT)LocalAlloc(LMEM\_ZEROINIT,0x54);

  memset((PVOID)pCatContext,0,0x54);
  //FIX ME: CHECK WHEN pCatAdmin == NULL!
  pCatContext->cbSize = 0x54;

  LIST\_Initialize(&(pCatContext->list1));

  if(\_pgSubsystem == NULL)
  {
    pCatContext->UseDefaultGUID = UseDefaultGUID;
  }
  else
  {
    UseDefaultGUID = FALSE;
    pGUIDToUse = \_pgSubsystem;
  }
  
  guid2wstr(pGUIDToUse,wGUID);

  InitializeCriticalSection(&(pCatContext->CriticalSection));

  pCatContext->\_64 = UseDefaultGUID;
  pCatContext->\_68 = 0;
  //FIX ME:WHEN NULL RETURNED!
  pCatContext->pwGUID = (PWCHAR)LocalAlloc(LMEM\_ZEROINIT,2 \* lstrlenW(wGUID) + 2);
  
  wcscpy(pCatContext->pwGUID,wGUID);
  //FIX ME:WHEN NULL RETURNED!
  pCatContext->pwDirectoryPath = \_CatAdminCreatePath(OrigPath,wGUID,TRUE);
  //FIX ME:WHEN NULL RETURNED!
  pCatContext->pwDirectoryPath2 = \_CatAdminCreatePath(OrigPath2,wGUID,TRUE);
  //FIX ME:WHEN FALSE RETURNED!
  if(\_CatAdminRecursiveCreateDirectory(pCatContext->pwDirectoryPath,NULL))
  {
    //FIX ME:WHEN FALSE RETURNED!
    if(\_CatAdminRecursiveCreateDirectory(pCatContext->pwDirectoryPath2,NULL))
    {
      //FIX ME:WHEN NULL RETURNED!
      pCatContext->hWakeEventHandle = CreateEvent(NULL,FALSE,FALSE,NULL);
      if(pCatContext->hWakeEventHandle)
      {
        //FIX ME:WHEN 0 RETURNED!
        //if(RegisterWaitForSingleObject(&(pCatContext->hCleanWaitObject),pCatContext->hWakeEventHandle,(WAITORTIMERCALLBACK)\_CatAdminWaitOrTimerCallback,pCatContext,INFINITE,WT\_EXECUTEDEFAULT))
        //{
          \*\_phCatAdmin = (HCATADMIN \*)pCatContext;
        //}
      }
    }
  }

  return ReturnValue;
}

VOID LIST\_Initialize(PVOID pBegin)
{
  \*(DWORD \*)pBegin = 0;
  \*(DWORD \*)((BYTE \*)pBegin + 4) = 0;
  \*(DWORD \*)((BYTE \*)pBegin + 8) = 0;
}

BOOL guid2wstr(GUID \*guid,PWCHAR pwguid)
{
  int i = 0;
  //FIX ME:WHEN guid OR pwguid == NULL!
  \*pwguid = L'{';
  pwguid += 1;

  for(;i < 0x14;i++,pwguid++)
  {
    if(guidtowstr\[i\] != '-')
    {
      \*pwguid = UnicodeNumber\[((DWORD)(\*((BYTE \*)guid + guidtowstr\[i\]))) / 16\];
      pwguid++;
      \*pwguid = UnicodeNumber\[((DWORD)(\*((BYTE \*)guid + guidtowstr\[i\]))) & 0xf\];
    }
    else
    {
      \*pwguid = L'-';
    }
  }

  \*pwguid = L'}';
  \*(pwguid + 1) = 0;

  return TRUE;
}

PWCHAR  \_CatAdminCreatePath(PWCHAR OrigPath,PWCHAR pwGUID,BOOL UseDefaultGUID)
{
  int length = 0;
  PWCHAR pwFinalPath;

  length += lstrlenW(OrigPath);
  length += lstrlenW(pwGUID);
  length += 2;
  if(UseDefaultGUID)
  {
    length += 1;
  }
  //FIX ME:WHEN NULL RETURNED!
  pwFinalPath = (PWCHAR)LocalAlloc(LMEM\_ZEROINIT,2 \* length);

  wcscpy(pwFinalPath,OrigPath);
  if(OrigPath\[lstrlenW(OrigPath) - 1\] != L'\\\\')
  {
    wcscat(pwFinalPath,Slash);
  }
  wcscat(pwFinalPath,pwGUID);

  if(UseDefaultGUID)
  {
    wcscat(pwFinalPath,Slash);
  }

  return pwFinalPath;
}

BOOL \_CatAdminRecursiveCreateDirectory(PWCHAR pwDirectoryPath,LPSECURITY\_ATTRIBUTES lpSecurityAttributes)
{
  PWCHAR pwPath = NULL;
  BOOL ret;

  if(pwDirectoryPath\[lstrlenW(pwDirectoryPath) - 1\] == L'\\\\')
  {
    //FIX ME:WHEN NULL RETURNED!
    pwPath = (PWCHAR)LocalAlloc(LMEM\_ZEROINIT,2 \* lstrlenW(pwDirectoryPath));
    memcpy((PVOID)pwPath,(PVOID)pwDirectoryPath,2 \* lstrlenW(pwDirectoryPath) - 2);
    pwPath\[lstrlenW(pwDirectoryPath) - 1\] = 0;
    ret = \_CatAdminRecursiveCreateDirectory(pwPath,lpSecurityAttributes);
  }
  else
  {
    //FIX ME:CHECK WHETHER IT'S WINNT FIRST!
    if(GetFileAttributesW(pwDirectoryPath) != 0xffffffff) //INVALID\_FILE\_ATTRIBUTES
    {
      //FIX ME:WHEN NOT!
      if(GetFileAttributesW(pwDirectoryPath) & FILE\_ATTRIBUTE\_DIRECTORY)
      {
        ret = TRUE;
      }
    }
    else
    {
      //FIX ME:WHEN ERROR CODE IS OTHERS!
      if(GetLastError() == ERROR\_PATH\_NOT\_FOUND || GetLastError() == ERROR\_FILE\_NOT\_FOUND)
      {
        //FIX ME:CHECK WHETHER IT'S WINNT FIRST!
        CreateDirectoryW(pwDirectoryPath,lpSecurityAttributes);
        SetFileAttributesW(pwDirectoryPath,FILE\_ATTRIBUTE\_NORMAL);
        ret = TRUE;
      }
    }
  }

  if(pwPath)
  {
    LocalFree(pwPath);
  }
  
  return ret;
}

VOID CALLBACK \_CatAdminWaitOrTimerCallback(PVOID lpParameter,BOOLEAN TimerOrWaitFired)
{
}

```

最后那个注册的回调函数，是用来清理释放资源的，由于我的 vc6 SDK 不够高级，没有 RegisterWaitForSingleObject，所以也就没有还原除 c 代码。实际使用时，即使不加上这些，也是可以成功验证的，只是可能有一些资源泄漏问题。 

直接从文件格式验证也是可以的  
  
function VerifySignature(Index: Integer): Integer;  
var  
  Signer: TElPKCS7Signer;  
  Certificate: TElX509Certificate;  
  Lookup: TElCertificateLookup;  
  I, TagID: Integer;  
  IncludedDigest, OriginalDigest, DecryptedDigest: string;  
  MD5: TMessageDigest128;  
  SHA1: TMessageDigest160;  
  Data: string;  
begin  
  try  
    // check for signature index validity  
    Result := SB\_AUTHENTICODE\_ERROR\_INVALID\_INDEX;  
    if (Index < 0) or (Index >= FMessage.SignedData.SignerCount) then  
      Exit;  
    Signer := FMessage.SignedData.Signers\[Index\];  
    if Signer = nil then  
      Exit;  
    // attempt to find signer's certificate  
    Result := SB\_AUTHENTICODE\_ERROR\_NO\_SIGNER\_CERTIFICATE;  
    Certificate := nil;  
    Lookup := TElCertificateLookup.Create(nil);  
    try  
      Lookup.Criteria := \[lcIssuer\];  
      Lookup.Options := \[loExactMatch, loMatchAll\];  
      Lookup.IssuerRDN.Assign(Signer.Issuer.Issuer);  
      I := FMessage.SignedData.Certificates.FindFirst(Lookup);  
      while I >= 0 do  
      begin  
        if FMessage.SignedData.Certificates.Certificates\[I\].SerialNumber = Signer.Issuer.SerialNumber then  
        begin  
          Certificate := FMessage.SignedData.Certificates.Certificates\[I\];  
          Break;  
        end;  
        I := FMessage.SignedData.Certificates.FindNext(Lookup);  
      end;  
    finally  
      Lookup.Free;  
    end;  
    if (Certificate = nil) and (FCertStorage <> nil) then  
    begin  
      Lookup := TElCertificateLookup.Create(nil);  
      try  
        Lookup.Criteria := \[lcIssuer\];  
        Lookup.Options := \[loExactMatch, loMatchAll\];  
        Lookup.IssuerRDN.Assign(Signer.Issuer.Issuer);  
        I := FCertStorage.FindFirst(Lookup);  
        while I >= 0 do  
        begin  
          if FCertStorage.Certificates\[I\].SerialNumber = Signer.Issuer.SerialNumber then  
          begin  
            Certificate := FCertStorage.Certificates\[I\];  
            Break;  
          end;  
          I := FCertStorage.FindNext(Lookup);  
        end;  
      finally  
        Lookup.Free;  
      end;  
    end;  
    if Certificate = nil then  
      Exit;  
    // calculate digest for authenticated attributes  
    if Signer.DigestAlgorithm = SB\_OID\_MD5 then  
    begin  
      MD5 := HashMD5(Signer.AuthenticatedAttributesPlain);  
      SetLength(OriginalDigest, SizeOf(MD5));  
      Move(MD5, OriginalDigest\[1\], SizeOf(MD5));  
    end  
    else  
      if Signer.DigestAlgorithm = SB\_OID\_SHA1 then  
    begin  
      SHA1 := HashSHA1(Signer.AuthenticatedAttributesPlain);  
      SetLength(OriginalDigest, SizeOf(SHA1));  
      Move(SHA1, OriginalDigest\[1\], SizeOf(SHA1));  
    end  
    else  
    begin  
      Result := SB\_AUTHENTICODE\_ERROR\_INVALID\_ALGORITHM;  
      Exit;  
    end;  
    // check for authenticated attributes digest validity  
    Result := SB\_AUTHENTICODE\_ERROR\_INVALID\_SIGNATURE;  
    if (Signer.DigestEncryptionAlgorithm = SB\_OID\_RSAENCRYPTION) and  
      (Certificate.PublicKeyAlgorithm = SB\_CERT\_ALGORITHM\_ID\_RSA\_ENCRYPTION) then  
    begin  
      DecryptedDigest := DecryptRSA(Signer.EncryptedDigest, Certificate);  
      if DecryptedDigest = '' then  
        Exit;  
      // compare the decrypted digest with calculated one  
      if DecryptedDigest <> OriginalDigest then  
        Exit;  
    end  
    else  
      if (Signer.DigestEncryptionAlgorithm = SB\_OID\_DSA) and  
      (Certificate.PublicKeyAlgorithm = SB\_CERT\_ALGORITHM\_ID\_DSA) then  
    begin  
      if not VerifyDSA(Signer.DigestAlgorithm, Signer.EncryptedDigest, OriginalDigest, Certificate) then  
        Exit;  
    end  
    else  
    begin  
      Result := SB\_AUTHENTICODE\_ERROR\_INVALID\_ALGORITHM;  
      Exit;  
    end;  
    // attempt to find included digest for authenticode data  
    Result := SB\_AUTHENTICODE\_ERROR\_SIGNATURE\_CORRUPTED;  
    IncludedDigest := '';  
    if Signer.AuthenticatedAttributes.Count > 0 then  
    begin  
      for I := 0 to Signer.AuthenticatedAttributes.Count - 1 do  
        if (Signer.AuthenticatedAttributes.Attributes\[I\] = SB\_OID\_MESSAGE\_DIGEST) and  
          (Signer.AuthenticatedAttributes.Values\[I\].Count > 0) then  
        begin  
          IncludedDigest := UnformatAttributeValue(Signer.AuthenticatedAttributes.Values\[I\].Strings\[0\], TagID);  
          Break;  
        end;  
    end;  
    if IncludedDigest = '' then  
      Exit;  
    // calculate digest for authenticode data  
    Result := SB\_AUTHENTICODE\_ERROR\_SIGNATURE\_CORRUPTED;  
    if Length(FMessage.SignedData.Content) < 2 then  
      Exit;  
    if Ord(FMessage.SignedData.Content\[1\]) <> $30 then  
      Exit;  
    case Ord(FMessage.SignedData.Content\[2\]) of  
      $81: Data := Copy(FMessage.SignedData.Content, 4, MaxInt);  
      $82: Data := Copy(FMessage.SignedData.Content, 5, MaxInt);  
      $83: Data := Copy(FMessage.SignedData.Content, 6, MaxInt);  
    else  
      if Ord(FMessage.SignedData.Content\[2\]) < $81 then  
        Data := Copy(FMessage.SignedData.Content, 3, MaxInt)  
      else  
        Exit;  
    end;  
    if Signer.DigestAlgorithm = SB\_OID\_MD5 then  
    begin  
      MD5 := HashMD5(Data);  
      SetLength(OriginalDigest, SizeOf(MD5));  
      Move(MD5, OriginalDigest\[1\], SizeOf(MD5));  
    end  
    else  
      if Signer.DigestAlgorithm = SB\_OID\_SHA1 then  
    begin  
      SHA1 := HashSHA1(Data);  
      SetLength(OriginalDigest, SizeOf(SHA1));  
      Move(SHA1, OriginalDigest\[1\], SizeOf(SHA1));  
    end  
    else  
    begin  
      Result := SB\_AUTHENTICODE\_ERROR\_INVALID\_ALGORITHM;  
      Exit;  
    end;  
    // check for authenticode data digest validity  
    if OriginalDigest <> IncludedDigest then  
    begin  
      Result := SB\_AUTHENTICODE\_ERROR\_INVALID\_SIGNATURE;  
      Exit;  
    end;  
    if (FActualDigest <> FDigest) then  
      Result := SB\_AUTHENTICODE\_ERROR\_INVALID\_AUTHENTICODE  
    else  
      Result := SB\_AUTHENTICODE\_ERROR\_SUCCESS;  
  except  
    Result := SB\_AUTHENTICODE\_ERROR\_UNKNOWN;  
  end;  
end; 

签名也是一个鸟样:  
  
:读取并 HASH"MZ"  
2:移动到$0000003C 位置,并 HASH 前面的数据.  
3:读入 4 字节到 FHeaderOffset,判断 FHeaderOffset + $A0 是否大于文件大小.并 HASH 这 FHeaderOffset.  
4:移动到 FHeaderOffset 位置,并 HASH 前面的数据.  
5:读入 4 字节到 PESign,判断 PESign 是否为 $00004550.并 HASH 这 PESign.  
6:  
(1)保存当前流位置  
(2)移动指针到当前位置+$14,并读入两字节.如果该内容为$020B,说明为 64 位 PE,那么后面位置要加$10  
(3)恢复流位置  
  
7:移动到 FHeaderOffset + $58 位置,并 HASH 前面的数据.  
8:FStream.Read(Checksum, 4)!!!注意,这里并没 HASH  
9:再次移动到 SeekStream(FHeaderOffset + $98 + FDelta64bit)位置,并 HASH 前面的数据.  
10:FStream.Read(SignatureOffset, 4)!!!注意,这里并没 HASH  
11:FStream.Read(SignatureSize, 4)!!!注意,这里并没 HASH  
12:根据 SignatureOffset 之类判断是否已经签名了.  
13:SeekStream(FStream.Size)HASH 余下的数据.  
14:(FStream.Size and $F)判断整个文件大小是否能被 16 整除.不能则后面补零填充  
  
(\*  
总结:  
校验和的偏移位置(唯一一个不参与校验和计算的数据)  
数字签名体的偏移位置  
数字签名大小  
  
\*)  
签名收尾:  
var  
  SignatureOffset, SignatureSize, Dummy: LongWord;  
  
FStream.Position := FStream.Size;//移动到文件尾部  
if FPadding <> '' then  
   FStream.WriteBuffer(FPadding\[1\], Length(FPadding)); //补齐 16 字节  
  
SignatureOffset := FStream.Position;//签名信息位置  
  
SignatureSize := Length(S) + 8;//签名信息大小+sizeof(SignatureSize)+sizeof(Dummy)  
FStream.WriteBuffer(SignatureSize, SizeOf(SignatureSize));//写入签名大小  
CalcChecksum(FChecksum, @SignatureSize, SizeOf(SignatureSize));  
  
Dummy := $00020200;  
FStream.WriteBuffer(Dummy, SizeOf(Dummy));//写入 Dummy  
CalcChecksum(FChecksum, @Dummy, SizeOf(Dummy));  
  
FStream.WriteBuffer(S\[1\], Length(S));//写入签名  
CalcChecksum(FChecksum, @S\[1\], Length(S));  
  
FStream.Position := FHeaderOffset + $98 +FDelta64bit;  
FStream.WriteBuffer(SignatureOffset, SizeOf(SignatureOffset));//改写签名位置  
CalcChecksum(FChecksum, @SignatureOffset, SizeOf(SignatureOffset));  
  
FStream.WriteBuffer(SignatureSize, SizeOf(SignatureSize)); //改写签名大小  
CalcChecksum(FChecksum, @SignatureSize, SizeOf(SignatureSize));  
  
FStream.Position := FHeaderOffset + $58;  
Dummy := FinalizeChecksum(FChecksum, FStream.Size);  
FStream.WriteBuffer(Dummy, SizeOf(Dummy));//最后一个总数据校验
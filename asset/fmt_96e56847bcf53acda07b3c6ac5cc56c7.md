
```c
BOOL WINAPI GetHaoTingWMAPath(char *lpFullPath, LPCTSTR lpURL, LPCTSTR lpFileName)
{
  char szBuffer[256];

  memset(szBuffer, 0, sizeof(char) * 256);

  if (S_OK == URLDownloadToFile(NULL,
    lpURL,
    lpFileName,
    0,
    NULL))
  {
    HANDLE hFile = CreateFile(lpFileName,
      GENERIC_READ,
      FILE_SHARE_READ,
      NULL, OPEN_EXISTING,
      FILE_FLAG_DELETE_ON_CLOSE,
      NULL);
    

    if (INVALID_HANDLE_VALUE == hFile)
    {
      hFile = NULL;
      return FALSE;
    }

    LARGE_INTEGER FileSize = {0};

    GetFileSizeEx(hFile, &FileSize);
    
    HANDLE hMemFile = CreateFileMapping(hFile, NULL, PAGE_READONLY, 0, 0, NULL);

    if (INVALID_HANDLE_VALUE == hMemFile)
    {
      hMemFile = NULL;
      return FALSE;
    }

    LPBYTE lpByte = (LPBYTE)MapViewOfFile(hMemFile, FILE_MAP_READ, 0, 0, 0);

    for (UINT i = 0; i < FileSize.u.LowPart; i++)
    {

      if ((*lpByte == '.')
        && (*(lpByte + 1) == 'w') 
        && (*(lpByte + 2) == 'm') 
        && (*(lpByte + 3) == 'a'))
      {

        for (;;)
        {
          
          lpByte -= 1;

          if (*(lpByte - 1) == '"')
          {
            
            for (UINT j = 0; ; j++)
            {
              szBuffer[j] = *lpByte;
              
              if ((*lpByte == 'a')
                && (*(lpByte - 1) == 'm')
                && (*(lpByte - 2) == 'w')
                && (*(lpByte - 3) == '.'))
              {
                break;
              }

              lpByte += 1;

            }

            break;
          }

          
        }
        
        for (UINT k = 0; ; k++)
        {
          
          if ((*lpByte == 'h')
            && (*(lpByte + 1) == 'a')
            && (*(lpByte + 2) == 'o')
            && (*(lpByte + 3) == 't')
            && (*(lpByte + 4) == 'i')
            && (*(lpByte + 5) == 'n')
            && (*(lpByte + 6) == 'g')
            && (*(lpByte + 7) == '.')
            && (*(lpByte + 8) == 'c')
            && (*(lpByte + 9) == 'o')
            && (*(lpByte + 10) == 'm'))
          {
            
            for (;;)
            {
              lpByte -= 1;

              if ((*(lpByte - 1) == '\\'))
              {
                
                for (UINT l = 0; ; l++)
                {
                  lpFullPath[l] = *lpByte;

                  if ((*lpByte == 'm')
                    && (*(lpByte - 1) == 'o')
                    && (*(lpByte - 2) == 'c')
                    && (*(lpByte - 3) == '.'))
                  {
                    break;
                  }

                  lpByte += 1;
                }

                break;
              }

            }
            
            break;
          }

          lpByte += 1;

        }

        break;
      }

      lpByte += 1;
    }
    
    UnmapViewOfFile((LPVOID)lpByte);

    CloseHandle(hMemFile);
    CloseHandle(hFile);
    
    lstrcatA(lpFullPath, szBuffer);

    return TRUE;

  }

  return FALSE;

}
```

为了方便，我写了个 GUI 界面来查看效果:

![](img/0275bd3d914c6c4749801e6be0e44095.jpg)

附件里是我写的 Demo 程序，我试过很多页面，都可以读出来。破船的那个程序有一个缺点，就是分析起来太久了，对于批量分析不太适用![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")。。。

PS:

刚刚编辑完帖子，就发现破船回复了。。我写的这个分析引擎貌似它的防下载没有作用,因为我只读到`.wma`就不再读了，它后面是什么内容都不用管了...

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `Demo.rar` |
|---|---|---|

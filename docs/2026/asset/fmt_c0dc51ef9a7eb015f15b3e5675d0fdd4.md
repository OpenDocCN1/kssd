
```
NULL,                         // no security

CREATE_ALWAYS,                  // open or create

FILE_ATTRIBUTE_NORMAL,        // normal file

NULL);

if(hFile == INVALID_HANDLE_VALUE)
{      printf("Can't open dump file");
           return 0;
  }

WriteFile(hFile, hdesc->buffer, hdesc->size,&dwBytesWritten, NULL);
if(dwBytesWritten != hdesc->size)
{
        printf("WriteHive error\n");
}
CloseHandle(hFile);
return 0;
}
```

```c
void CleanPatterns()
{
      int i;
      if(pattern!=NULL)
      {
           for(i = 0; i < pattern_count; i++)
           {
                 if(pattern[i]!=NULL)
                      free(pattern[i]);
           }
           free(pattern);
      }
}
```

```c
void GetPatterns()
{
      HANDLE hService;
      CHAR achKey[MAX_PATH];
      DWORD i;
      DWORD retCode;
      int Nohide = 1;
      DWORD SubKeyNum = 0;
      pattern_count = 0;
      if(RegOpenKeyEx(
           HKEY_LOCAL_MACHINE,
           "SYSTEM\\ControlSet001\\Services",
           NULL,
           KEY_ALL_ACCESS,
           &hService
           ) != ERROR_SUCCESS)
      {
           printf("sorry %d\n",GetLastError());
           return;
      }
    RegQueryInfoKey( hService,
        NULL,
        NULL,
        NULL,
        &SubKeyNum,
        NULL,
        NULL,
        NULL,
        NULL,
        NULL,
        NULL,
        NULL);
      if(SubKeyNum == 0)
      {
           printf("SubKey's Number is NULL, it's too strange.\n");
           return;
      }
      pattern = malloc(sizeof(char *) * SubKeyNum );
      for (i = 0, retCode = ERROR_SUCCESS; retCode == ERROR_SUCCESS; i++)
    {
        retCode = RegEnumKey(
                 hService,
                 i,
                 achKey,
                 MAX_PATH
                 );
        if (retCode == (DWORD)ERROR_SUCCESS)
        {
              pattern[ pattern_count ] = strdup ( achKey ) ;
              pattern_count++;
        }
       }
      CloseHandle(hService);
}
```

```c
void ShowPathImage(struct hive *hdesc, int nkofs, char *path)
{
  void *data;
  int len,i,type;
  char string[SZ_MAX+1];
  type = get_val_type(hdesc, nkofs, path);
  if (type == -1) {
    printf("No such value <%s>\n",path);
    return;
  }
  len = get_val_len(hdesc, nkofs, path);
  if (!len) {
    printf("Value <%s> has zero length\n",path);
    return;
  }
  data = (void *)get_val_data(hdesc, nkofs, path, 0);
  if (!data) {
    printf("Value <%s> references NULL-pointer (bad boy!)\n",path);
    abort();
    return;
  }
  switch (type) {
  case REG_SZ:
  case REG_EXPAND_SZ:
  case REG_MULTI_SZ:
    cheap_uni2ascii(data,string,len);
    for (i = 0; i < (len>>1)-1; i++) {
      if (string[i] == 0) string[i] = '\n';
      if (type == REG_SZ) break;
    }
    puts(string);
    break;
  case REG_DWORD:
    printf("0x%08x",*(unsigned short *)data);
    break;
  default:
    printf("Don't know how to handle type yet!\n");
  case REG_BINARY:
    hexdump((char *)data, 0, len, 1);
  }
}
```

```c
void EnablePriv(LPCTSTR lpName)
{
      HANDLE hToken;
      LUID sedebugnameValue;
      TOKEN_PRIVILEGES tkp;
      if ( ! OpenProcessToken( GetCurrentProcess(),
           TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &hToken ) )
      {    printf("open process error\n");
           return;
      }
      if ( ! LookupPrivilegeValue( NULL, lpName  , &sedebugnameValue ) ){
           printf("can't find privilege error\n");
           CloseHandle( hToken );
           return;
      }
      tkp.PrivilegeCount = 1;
      tkp.Privileges[0].Luid = sedebugnameValue;
      tkp.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;
      if ( ! AdjustTokenPrivileges( hToken, FALSE, &tkp, sizeof (tkp), NULL, NULL ) )
      {
           printf("adjust error\n");
           CloseHandle( hToken );
      }
}
```

```c
int DumpServiceInfo()
{
      HKEY hService;
      EnablePriv(SE_BACKUP_NAME);
      if(RegOpenKeyEx(
           HKEY_LOCAL_MACHINE,
           "SYSTEM\\ControlSet001\\Services",
           NULL,
           KEY_ALL_ACCESS,
           &hService
           ) != ERROR_SUCCESS)
      {    printf("can't get key handle\n");
           return 0;
      }
      if(RegSaveKey(hService,"C:\\tmp.hiv",NULL) != ERROR_SUCCESS)
      {
           printf("Can't dump Service info\n");
           CloseHandle(hService);
           return 0;
      }
      CloseHandle(hService);
      return 1;
}
```

```c
void ShowHideService(struct hive *hdesc, char *path, int vofs, int type)
{
  struct nk_key *key;
  int nkofs;
  struct ex_data ex;
  int count = 0, countri = 0;
  void *data;
  int nkofs_cat;
  int serviceno;
  serviceno = 1;
  nkofs = trav_path(hdesc, vofs, path, 0);
  if(!nkofs) {
    printf("nk_ls: Key <%s> not found\n",path);
    abort();
    return;
  }
  nkofs += 4;
  key = (struct nk_key *)(hdesc->buffer + nkofs);
  if (key->id != 0x6b6e) {
    printf("Error: Not a 'nk' node!\n");
    debugit(hdesc->buffer,hdesc->size);
  }
   if (key->no_subkeys) {
        while ((ex_next_n(hdesc, nkofs, &count, &countri, &ex) > 0)) {
             if(!CompareHive(ex.name) )
             {
                   nohideservice = 0;
                   if(!(serviceno - 1))
                        printf("Hide Service List:\n");
                   printf("\n%d.------------------------------------------------------------\n",serviceno++ );
                   printf("Hide Service : %s\n", ex.name );
                   nkofs_cat = trav_path(hdesc, vofs, ex.name, 0);
                   printf("Image Path   : ");
                   ShowPathImage(hdesc, nkofs_cat + 4, "ImagePath");
                   data = (void *) get_val_data(hdesc, nkofs_cat + 4, "Start", 0 );
                   if( data != NULL)
                   {
                        printf("Start Type   : ");
                        switch(*(unsigned short *)data)
                         {
                        case 0:
                              printf("SERVICE_BOOT_START");
                              break;
                        case 1:
                              printf("SERVICE_SYSTEM_START");
                              break;
                        case 2:
                              printf("SERVICE_AUTO_START");
                              break;
                        case 3:
                              printf("SERVICE_DEMAND_START");
                              break;
                        case 4:
                              printf("SERVICE_DISABLED");
                              break;
                        default:
                              printf("UNKOWN START TYPE");
                        }
                        if( *(unsigned short *)data != 4 )
                        {
                              printf("(Will be set to Disabled)");
                              put_dword(hdesc, nkofs_cat + 4, "Start", 4);
                              ischange = 1;
                        }
                        printf("\n");
                   }
                   data = (void *) get_val_data(hdesc, nkofs_cat + 4, "Type", 0 );
                   printf("Service Type : ");
                   if( data != NULL)
```

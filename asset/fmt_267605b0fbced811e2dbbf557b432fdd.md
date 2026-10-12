
# 代码实现与实战演练

## 代码结构定义

```c
PROC pNewProc;//新的函数指针  
PROC pOldProc;//老的函数指针  
}HOOKAPI, *LPHOOKAPI;
```

## 函数实现

### GetNamedImportDescriptor

```c
PIMAGE_IMPORT_DESCRIPTOR GetNamedImportDescriptor(HMODULE hModule, LPCSTR szImportMod)  
{  
  //首先是 DOS 头  
  PIMAGE_DOS_HEADER pDOSHeader = (PIMAGE_DOS_HEADER) hModule;  
  if(pDOSHeader->e_magic != IMAGE_DOS_SIGNATURE) return NULL;  
  PIMAGE_NT_HEADERS pNTHeader = (PIMAGE_NT_HEADERS)((DWORD)pDOSHeader + (DWORD)(pDOSHeader->e_lfanew));  
  if(pNTHeader->Signature != IMAGE_NT_SIGNATURE) return NULL;  
  //如果没有 Import 部分,返回失败  
  if(pNTHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress == 0)  
    return NULL;  
  //取 Import 部分  
  PIMAGE_IMPORT_DESCRIPTOR pImportDesc = (PIMAGE_IMPORT_DESCRIPTOR)  
    ((DWORD)pDOSHeader + (DWORD)(pNTHeader->OptionalHeader.  
    DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress));  
  //寻找与 szImportMod 相配部分  
  while (pImportDesc->Name)  
  {  
    PSTR szCurrMod = (PSTR)((DWORD)pDOSHeader + (DWORD)(pImportDesc->Name));  
    if (stricmp(szCurrMod, szImportMod) == 0)  
      break; //找到  
    pImportDesc++;  
  }  
  if(pImportDesc->Name == NULL) return NULL;  
  return pImportDesc;  
}
```

### HookAPIByName

```c
HookAPIByName(HMODULE hModule/*被 HOOK 的目标进程 MODULE*/, LPCSTR szImportMod/*如 GDI32.DLL*/,LPHOOKAPI pHookApi/*指定函数名,如"MessageBoxW"*/)  
{  
  PIMAGE_IMPORT_DESCRIPTOR pImportDesc =  
    GetNamedImportDescriptor(hModule, szImportMod);  
  if (pImportDesc == NULL)  
    return FALSE; //需要改换的 API 不能取到正确描述符  

  PIMAGE_THUNK_DATA pOrigThunk = (PIMAGE_THUNK_DATA)((DWORD)hModule + (DWORD)(pImportDesc->OriginalFirstThunk));  
  PIMAGE_THUNK_DATA pRealThunk =  
    (PIMAGE_THUNK_DATA)((DWORD)hModule + (DWORD)(pImportDesc->FirstThunk));  

  while(pOrigThunk->u1.Function)  
  {  
    if((pOrigThunk->u1.Ordinal & IMAGE_ORDINAL_FLAG) != IMAGE_ORDINAL_FLAG)  
    {  
      PIMAGE_IMPORT_BY_NAME pByName = (PIMAGE_IMPORT_BY_NAME)((DWORD)hModule + (DWORD)  
      (pOrigThunk->u1.AddressOfData));  
      if(pByName->Name[0] == '\0')  
        return FALSE; //失败  
      if(strcmpi(pHookApi->szFunc, (char*)pByName->Name) == 0)  
      {  
        //改变 thunk 保护属性  
        MEMORY_BASIC_INFORMATION mbi_thunk;  
        VirtualQuery(pRealThunk, &mbi_thunk, sizeof(MEMORY_BASIC_INFORMATION));  
        VirtualProtect(mbi_thunk.BaseAddress,mbi_thunk.RegionSize, PAGE_READWRITE,  
        &mbi_thunk.Protect);  
        //保存原来的 API 函数指针  
        if(pHookApi->pOldProc == NULL)  
          pHookApi->pOldProc = (PROC)pRealThunk->u1.Function;  
        //改变 API 函数指针  
        pRealThunk->u1.Function = (PDWORD)pHookApi->pNewProc;  
        //将 thunk 保护属性改回来  
        DWORD dwOldProtect;  
        VirtualProtect(mbi_thunk.BaseAddress, mbi_thunk.RegionSize,  
          mbi_thunk.Protect, &dwOldProtect);  
      }  
    }  
    pOrigThunk++;  
    pRealThunk++;  
  }  
  SetLastError(ERROR_SUCCESS);  
  return TRUE;  
}
```

## 五、实战演练

现在万事俱备，只欠一个我们反复提到的——自己的函数就可以完成全部的任务了。

那这个自己的函数要做些什么呢？其实很简单，就是把通过它发送的数据包一一记录下来备用即可。

而记录的数据，将要被我们自己的程序所使用，所以存在 `dll` 的共享数据区里面。

这是其一，对于解除代理和 `BT` 的封禁，这已经足够了。

但要解决路由器的问题，还需要再加一点功能。

我们会把正确的不带路由器的认证包（记为 `A` 包）存起来，然后在有路由器环境下，客户端会产生指示使用了路由器的认证包（记为 `B` 包），用 `A` 包代替 `B` 包发送出去，就起到了欺骗服务器的作用，解除了路由器的封禁。

```c
static int WINAPI Mysendto(  
  SOCKET s,                    
  const char FAR * buf,        
  int len,                     
  int flags,                   
  const struct sockaddr FAR * to,  
  int tolen                   
)  
{  
  int ii=0;  
  if(len!=26)  
  {  
    if(statue_client==0)  
    {  
      if(stat_authorize==0)  
      {  
        for(int tt=0;tt<len;tt++)  
        {  
          *(buf_authorize+tt)=*(buf+tt);  
        }  
        len_authorize=len;  
        tolen_authorize=tolen;  
        flags_authorize=flags;  
        strcpy(to_authorize.sa_data,to->sa_data);  
        to_authorize.sa_family=to->sa_family;  
        stat_authorize=1;  
        ii=::sendto(s,buf,len,flags,to,tolen);  
      }  
      else  
      {  
        ii=::sendto(s,buf_authorize,len_authorize,flags_authorize,&to_authorize,tolen_authorize);  
      }  
      statue_client=1;  
    }  
    else if(statue_client==1)  
    {  
      statue_client=2;  
      for(int tt=0;tt<len;tt++)  
      {  
        *(buf_keepconn+tt)=*(buf+tt);  
      }  
      len_keepconn=len;  
      tolen_keepconn=tolen;  
      flags_keepconn=flags;  
      strcpy(to_keepconn.sa_data,to->sa_data);  
      to_keepconn.sa_family=to->sa_family;  
      stat_keepconn=1;  
      statue_client=3;  
      ii=::sendto(s,buf,len,flags,to,tolen);  
    }  
  }  
  else  
  {  
    for(int tt=0;tt<len;tt++)  
    {  
      *(buf_getservice+tt)=*(buf+tt);  
    }  
    len_getservice=len;  
    tolen_getservice=tolen;  
    flags_getservice=flags;  
    strcpy(to_getservice.sa_data,to->sa_data);  
    to_getservice.sa_family=to->sa_family;  
    stat_getservice=1;  
    ii=::sendto(s,buf,len,flags,to,tolen);  
  }  
  return ii;  
}
```

## 六、屠城三日

至此，神州数码的三大封禁已基本被我们解决，剩下来的工作，就是愉快的上网，愉快的用代理，愉快的用路由器，愉快的 `BT` 了。

---

### 论坛互动记录

**版权声明**：本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
2006 年 05 月 13 日 3:00:40

**引用**: 最初由 `dezone` 发布

好文！支持

另外，想问问楼主一个关于 `dll` 注入的问题，有时候一些程序是在启动时调用相关 `API` 的(如 `findwindow`)，我们的 `dll` 还没来的及注入，它就已经调用了该 `api` 了，本来想 `hook` 这些 `api` 的，但是迟了，请问有什么办法可以以最快的速度把 `dll` 注入其它程序呢？

另：用 `CreateProcess` 插入代码应该可以，哪有例子看看啊。

**引用**: 最初由 `austiny` 发布

这个函数配合 `InjectLib` 函数可以完成这个功能：

```c
int APIENTRY WinMain(HINSTANCE hInstance,  
                     HINSTANCE hPrevInstance,  
                     LPSTR     lpCmdLine,  
                     int       nCmdShow)  
{  
LPPROCESS_INFORMATION p=new _PROCESS_INFORMATION;  
STARTUPINFO ss=  
{  
  sizeof(STARTUPINFO),0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0};  
  BOOL b=::CreateProcess(EXEPATH,"",NULL,NULL,TRUE,CREATE_SUSPENDED,NULL,NULL,&ss,p);  
  DWORD err;  
  if(!b)  
  {  
   err=GetLastError();  
   return -1;  
  }  
  InjectLib(p->dwProcessId,DLLPATH);  
  ::ResumeThread(p->hThread);  
  return 0;  
}
```

感谢 `austiny` 兄的回复。（不好意思，才上看雪，刚看到）

不过这个使用了 `CreateRemoteThread`，只能在 `2k`, `xp` 以上系统下使用，`98` 下不能使用。

另外，《windows 核心编程》中有提到用 `CreateProcess` 插入代码的方法来实现 `dll` 的注入（最后提到的一种方法，这种方法的好处是 `98`, `2k`, `xp` 均可），但却没有给出代码，只给出大概的思路，我做了一下没成功，网上也找不到相关例子，不知道哪位兄弟做过。

`4dd466d0e1e0cc123afbe504da2462ca`

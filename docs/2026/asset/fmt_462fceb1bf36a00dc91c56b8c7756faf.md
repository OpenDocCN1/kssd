
```
static uint64
inject\_gencode\_mapped(HANDLE phandle, char \*dynamo\_path, uint64 hook\_location,
byte hook\_buf[EARLY\_INJECT\_HOOK\_SIZE], void \*must\_reach,
bool x86\_code, bool late\_injection, uint old\_hook\_prot)
{
bool success = false;
NTSTATUS res;
HANDLE file = INVALID\_HANDLE\_VALUE;
HANDLE section = INVALID\_HANDLE\_VALUE;
byte \*map = NULL;
size\_t view\_size = 0;
wchar\_t dllpath[MAX\_PATH];
uint64 ret = 0;
if (!convert\_to\_NT\_file\_path(dllpath, dynamo\_path, BUFFER\_SIZE\_ELEMENTS(dllpath)))
goto done;
NULL\_TERMINATE\_BUFFER(dllpath);
res = nt\_create\_module\_file(&file, dllpath, NULL, FILE\_EXECUTE | FILE\_READ\_DATA,
FILE\_ATTRIBUTE\_NORMAL, FILE\_SHARE\_READ, FILE\_OPEN, 0);
if (!NT\_SUCCESS(res))
goto done;
res = nt\_create\_section(&section, SECTION\_ALL\_ACCESS, NULL, /\* full file size \*/
PAGE\_EXECUTE\_WRITECOPY, SEC\_IMAGE, file,
/\* XXX: do we need security options to put in other process?\*/
NULL /\* unnamed \*/, 0, NULL, NULL);
if (!NT\_SUCCESS(res))
goto done;
res = nt\_raw\_MapViewOfSection(section, phandle, &map, 0, 0 /\* not page-file-backed \*/,
NULL, (PSIZE\_T)&view\_size, ViewUnmap,
0 /\* no special top-down or anything \*/,
PAGE\_EXECUTE\_WRITECOPY);
if (!NT\_SUCCESS(res))
goto done;
ret =
inject\_gencode\_mapped\_helper(phandle, dynamo\_path, hook\_location, hook\_buf, map,
must\_reach, x86\_code, late\_injection, old\_hook\_prot);
done:
if (ret == 0) {
close\_handle(file);
close\_handle(section);
}
return ret;
}
```

此函数利用`NtCreateSection`和`NtMapViewOfSection`将`dynamorio.dll`注入到目标进程中：

![图片描述](upload/attach/202304/949068\_KBJ27NQS6ZBZU77.png)

之后调用`inject_gencode_mapped_helper`函数

### `inject_gencode_mapped_helper`

`inject_gencode_mapped_helper`主要执行流程如下：

```
typedef struct {
uint64 app\_xax;
uint64 dr\_base;
uint64 ntdll\_base;
uint64 tofree\_base;
uint64 hook\_location;
uint hook\_prot;
bool late\_injection;
char dynamorio\_lib\_path[MAX\_PATH];
} earliest\_args\_t;
static uint64
inject\_gencode\_mapped\_helper(HANDLE phandle, char \*dynamo\_path, uint64 hook\_location,
byte hook\_buf[EARLY\_INJECT\_HOOK\_SIZE], byte \*map,
void \*must\_reach, bool x86\_code, bool late\_injection,
uint old\_hook\_prot)
{
const size\_t remote\_alloc\_sz = 2 \* PAGE\_SIZE;
const size\_t code\_alloc\_sz = PAGE\_SIZE;
earliest\_args\_t args;
/\* 使用NTAllocateVirtualMemory在目标进程的虚拟空间中申请2 page内存\*/
remote\_code\_buf =
(uint64)allocate\_remote\_code\_buffer(phandle, remote\_alloc\_sz, must\_reach);
/\* 在本进程申请1 page内存 \*/
local\_code\_buf = allocate\_remote\_code\_buffer(NT\_CURRENT\_PROCESS, code\_alloc\_sz, NULL);
/\* hook\_code\_buf和remote\_code\_buf都指向目标进程第一个page \*/
hook\_code\_buf = remote\_code\_buf;
/\* remote\_data此时指向目标进程第二个page \*/
remote\_data = remote\_code\_buf + code\_alloc\_sz;
args.dr\_base = (uint64)map;
args.ntdll\_base = find\_remote\_dll\_base(phandle, target\_64, "ntdll.dll");
if (args.ntdll\_base == 0)
goto error;
args.tofree\_base = remote\_code\_buf;
args.hook\_location = hook\_location;
args.hook\_prot = old\_hook\_prot;
args.late\_injection = late\_injection;
strncpy(args.dynamorio\_lib\_path, dynamo\_path,
BUFFER\_SIZE\_ELEMENTS(args.dynamorio\_lib\_path));
/\* 将args写入到目标进程的第二个page也就是remote\_data中 \*/
write\_remote\_memory\_maybe64(phandle, remote\_data, &args, sizeof(args),
&num\_bytes\_out)
...
/\* 这里会在local\_code\_buf 构造的代码 \*/
构造的代码如下：
mov dword ptr ds:[remote\_data],eax ;也就是将eax存在args.app\_xax
mov eax,offset ntdll!RtlUserThreadStart (772641e0);hook\_location(目标进程主线程eip)
mov dword ptr [eax],0E9683D83h;恢复hook
mov dword ptr [eax+4],74007730h
mov dword ptr [eax+8],680D8B0Eh
mov byte ptr [eax+0Ch],0E9h
mov byte ptr [eax+0Dh],30h
mov eax,remote\_data
push offset ntdll!RtlUserThreadStart (772641e0);hook\_location
jmp dynamorio.dll!dynamorio\_earliest\_init\_takeover
/\* 将上面在local\_code\_buf 构造的代码 写入到remote\_code\_buf(hook\_code\_buf) \*/
write\_remote\_memory\_maybe64(phandle, hook\_code\_buf, local\_code\_buf,
cur\_local\_pos - local\_code\_buf, &num\_bytes\_out)
return hook\_code\_buf;
}
```

我们同样跟踪验证此函数。让大家对此过程更清晰。

内存分配后各个参数

`remote_code_buf = 0xb60000`

`remote_data = 0xb61000`

`local_code_buf = 0x010d0000`

#### 查看agrs：

![图片描述](upload/attach/202304/949068\_ZZ4RFXZX7XNWZ75.png)

#### `args`写入到目标进程的第二个page

![图片描述](upload/attach/202304/949068\_S2ZKG9N3KP9UDDA.png)

#### 查看构造的代码

![图片描述](upload/attach/202304/949068\_D9RT532UFVSVDUZ.png)

#### 写入`remote_code_buf`

![图片描述](upload/attach/202304/949068\_RRB8J7GE87TY6ZB.png)

我们现在已经分析完`inject_gencode_mapped_helper`，现在我们回到`inject_into_new_process`，`hook_target`就是我们的`remote_code_buf`：

![图片描述](upload/attach/202304/949068\_MR9U2SF2493URJS.png)

执行后如下：

![图片描述](upload/attach/202304/949068\_6PDHHNNE942TCQR.png)

### 总结

可以看到我们劫持目标进程是通过修改目标进程主线程的`eip`到`remote_code_buf`来实现的，我们在整个过程中都没有修改`hook_location`里的值，所以我认为那些恢复hook的操作是没有必要的。

## `dr_inject_process_run`

恢复由`dr_inject_process_create()`创建的进程中的暂停线程。

参数

[in] `data` 由`dr_inject_process_create()`返回的指针。

```
bool
dr\_inject\_process\_run(void\* data)
{
dr\_inject\_info\_t\* info = (dr\_inject\_info\_t\*)data;
if (info->attached == true) {
/\* detach the debugger \*/
DebugActiveProcessStop(info->pi.dwProcessId);
}
/\* resume the suspended app process so its main thread can run \*/
/\* 恢复主线程\*/
ResumeThread(info->pi.hThread);
close\_handle(info->pi.hThread);
return true;
}
```

执行`ResumeThread`后，我们切换到子进程，记住此时的寄存器信息：

![图片描述](upload/attach/202304/949068\_B6GUHRCWRFR2JQ9.png)

![图片描述](upload/attach/202304/949068\_6UFVUQ7TEQ8H78D.png)

```
00b60000 89050010b600 mov dword ptr ds:[0B61000h],eax
00b60006 b8e0412677 mov eax,772641E0h
00b6000b c74000833d68e9 mov dword ptr [eax],0E9683D83h
00b60012 c7400430770074 mov dword ptr [eax+4],74007730h
00b60019 c740080e8b0d68 mov dword ptr [eax+8],680D8B0Eh
00b60020 c6400ce9 mov byte ptr [eax+0Ch],0E9h
00b60024 c6400d30 mov byte ptr [eax+0Dh],30h
00b60028 b80010b600 mov eax,0B61000h ;remote\_data
00b6002d 68e0412677 push 772641E0h ;RtlUserThreadStart
00b60032 e9a634256f jmp dynamorio!dynamorio\_earliest\_init\_takeover (6fdb34dd)
```

主函数已经分析完了，我们来总结一下这个过程，首先使用暂停线程的方式打开目标进程，将`dynamorio.dll`注入到目标进程空间中，在目标进程申请空间并写入跳转到`dynamorio!dynamorio_earliest_init_takeover`的代码，修改目标进程线程`EIP`到这段代码中。这样之后当恢复线程运行的时候，就会运行到`dynamorio_earliest_init_takeover`中。

## `dynamorio_earliest_init_takeover`

`dynamorio_earliest_init_takeover`函数是在`x86.asm_core.s`中用汇编写的，如下：

```
1:003> u 6fdb34dd l20
dynamorio!dynamorio\_earliest\_init\_takeover [E:\dynamorio\build32\core\x86.asm\_core.s @ 4381]:
6fdb34dd 50 push eax
6fdb34de 8d6424fc lea esp,[esp-4]
6fdb34e2 8da424a8fdffff lea esp,[esp-258h] ;提高栈顶
6fdb34e9 ffb42460020000 push dword ptr [esp+260h]
6fdb34f0 9c pushfd ;构建priv\_mcontext\_t结构
6fdb34f1 60 pushad
6fdb34f2 8d0424 lea eax,[esp]
6fdb34f5 50 push eax
6fdb34f6 e825fdfcff call dynamorio!get\_simd\_vals (6fd83220)
6fdb34fb 8d642404 lea esp,[esp+4]
6fdb34ff 8d842480020000 lea eax,[esp+280h]
6fdb3506 8944240c mov dword ptr [esp+0Ch],eax
6fdb350a 8d1424 lea edx,[esp]
6fdb350d 8b44240c mov eax,dword ptr [esp+0Ch]
6fdb3511 8d400c lea eax,[eax+0Ch]
6fdb3514 8944240c mov dword ptr [esp+0Ch],eax
6fdb3518 8b842484020000 mov eax,dword ptr [esp+284h]
6fdb351f 8b08 mov ecx,dword ptr [eax]
6fdb3521 894c241c mov dword ptr [esp+1Ch],ecx
6fdb3525 52 push edx ;mc
6fdb3526 50 push eax ;arg\_ptr
6fdb3527 e82428f5ff call dynamorio!dynamorio\_earliest\_init\_takeover\_C (6fd05d50)
6fdb352c 8d642408 lea esp,[esp+8]
6fdb3530 61 popad
6fdb3531 9d popfd
6fdb3532 8da4245c020000 lea esp,[esp+25Ch]
6fdb3539 8d642408 lea esp,[esp+8]
6fdb353d c3 ret
```

此函数主要调用`dynamorio!dynamorio_earliest_init_takeover_C`，这之前的操作主要为了构建`priv_mcontext_t`结构。

![图片描述](upload/attach/202304/949068\_9FFMKVBAWVY7NCR.png)

## `dynamorio_earliest_init_takeover_C`

`priv_mcontext_t`结构保存了目标进程的上下文。对比刚切换子进程时的上下文，可以看到此时`EIP`为`RtlUserThreadStart`：

![图片描述](upload/attach/202304/949068\_9RP32UFMYWT3KU7.png)

`arg_ptr`为`remote_data`

![图片描述](upload/attach/202304/949068\_STETAG4Y32NKXCH.png)

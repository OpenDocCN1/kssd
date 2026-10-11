
# PspCidTable 杂谈

> 原文：[`www.pediy.com/kssd/pediy10/59680.html`](https://www.pediy.com/kssd/pediy10/59680.html)

今天再次看了`gz1X`牛的文章,回忆了许多知识. 哈哈. 玩了很长时间游戏,写点儿文字算是对自责的一种安慰![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

要说 R0 枚举隐藏进程, 只是对没有抹掉`PspCidTable`而言的.`gz1X`牛提示了 2 种方法:

引用: 1. 利用未导出的`ExEnumHandleTable`函数

2. 获取 `PHANDLE_TABLE_ENTRY` 等,然后`PsLookupProcessByProcessId`

偶觉得都不是很保险(假设暂且 `PspCidTable` 没有被抹掉), `ExEnumHandleTable`、`PsLookupProcessByProcessId`等都可以做手脚,简单的 `inline hook` 之类的.再说这些函数最终还是通过 `PHANDLE_TABLE`结构中的`TableCode`得到一些`HANDLE_TABLE_ENTRY`,到其中的 `Object` 里,取得相应的进程对象. 还不如自己来完成,嘿嘿. 安全感啊~![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

"第 8 个男人" 的 code 里面就做的很好,找到`PspCidTable`后自己搜索所有的进程对象就行了:

```
//----------------------------------------------------------------------

VOID

IsValidProcess ()

/*++

Author : 第 8 个男人

Leaner : sudami [xiao_rui_119@163.com]

Time   : 08/02/15

参数 : NULL

返回 : NULL

功能 :

  给出 PspCidTable 的地址,结合_EXHANDLE、HANDLE_TABLE、HANDLE_TABLE_ENTRY

  搜索到每个进程对象,纪录之.

  前提条件: 假设暂且 PspCidTable 没有被抹掉

--*/

{

ULONG PspCidTable;

ULONG TableCode;

ULONG table1,table2;

ULONG object,objectheader;

ULONG NextFreeTableEntry;

ULONG processtype,type;

ULONG flags;

ULONG i;

  PspCidTable = GetCidAddr(); // 搜索 PsLookupProcessByProcessId 函中的特征串即可

  processtype = GetProcessType();

if (PspCidTable == 0) {

return ;

  } else { //TableCode 的低 2 位决定句柄表的级数

    TableCode = *(PULONG) ( *(PULONG) PspCidTable );

if ( (TableCode & 3) == 0 ) { // 0 级

      table1 = TableCode;

      table2 = 0;

    } elseif ( (TableCode & 3 ) == 1 ) { // 1 级

      TableCode = TableCode & 0xfffffffc;

      table1    = *(PULONG)TableCode;

      table2    = *(PULONG)( TableCode + 4 );

    }

// 对 cid 从 0x0 到 0x4e1c 进行遍历

for (i = 0; i < 0x4e1c; i++) {

if (i <= 0x800) { // 第一张表中

if (MmIsAddressValid( (PULONG)(table1 + i*2) )) {

// HANDLE_TABLE_ENTRY 地址 + PID * 2 ---> 对象的地址

          object = *(PULONG)( table1 + i*2 );

if (MmIsAddressValid( (PULONG)(table1 + i*2 + NEXTFREETABLEENTRY) )) {

// 验证 HANDLE_TABLE_ENTRY 的合法性,这在 ExEnumHandleTable 函数的代码中也有

// 正常的_HANDLE_TABLE_ENTRY 中 NextFreeTableEntry 应该为 0

            NextFreeTableEntry = *(PULONG)(table1 + i*2 + NEXTFREETABLEENTRY);

if (NextFreeTableEntry == 0) {

// 去掉低 3 位掩码标志

                object = ((object | 0x80000000) & 0xfffffff8); // 转换为对象(体)指针

                  objectheader = (ULONG) \

                OBJECT_TO_OBJECT_HEADER(object); // 获取对象(头)指针

if (MmIsAddressValid( (PULONG)(objectheader + TYPE) )) {

                    type = *(PULONG)(objectheader + TYPE);

if (type == processtype) { // 是否为进程对象

                    flags = *(PULONG)( (ULONG)object + \

                    GetPlantformDependentInfo(OFFSET_EPROCESS_FLAGS) );

if ((flags&0xc) != 0xc)

                    RecordInfo( object ); //flags 显示进程没有退出

                }

              }

            }

          }

        }

      } else { // 第 2 张表

if (table2 != 0) {

// 步骤同上,只是 object 的获取为: HANDLE_TABLE_ENTRY 地址 + (PID- 0x800)*2

if (MmIsAddressValid( (PULONG)(table2 + (i - 0x800)*2) )) {

            object = *(PULONG)(table2 + (i - 0x800)*2);

if (MmIsAddressValid((PULONG)((table2+(i-0x800)*2)+NEXTFREETABLEENTRY))) {

              NextFreeTableEntry=*(PULONG)((table2+(i-0x800)*2)+NEXTFREETABLEENTRY);

if (NextFreeTableEntry==0x0) {

                  object = ((object | 0x80000000) & 0xfffffff8);

                    objectheader = (ULONG) OBJECT_TO_OBJECT_HEADER(object);

if (MmIsAddressValid( (PULONG)(objectheader + TYPE) )) {

                    type = *(PULONG)(objectheader + TYPE);

if(type == processtype) {

                      flags = *(PULONG) ((ULONG)object + \

                      GetPlantformDependentInfo(OFFSET_EPROCESS_FLAGS));

if ((flags&0xc) != 0xc)

                      RecordInfo(object);

                }

              }

            }

          }

        }

      }

    }

  }

}
```

当然,用`ExEnumHandleTable`也是不错的. Windbg 把它的实现看了一遍, 和 WRK 上的无任何出入[`XP SP2`], 实现起来也很方便![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")

R3 下就更简单了.一阵`OpenProcess`就把部分隐藏进程揪出来了:

```
#include <iostream.h>

#include <stdio.h>

#include <windows.h>

void main()

{

int a = 0;

for (int i=0;i<=65535;i+=4)  {

if(OpenProcess(PROCESS_QUERY_INFORMATION,FALSE,i)!= 0) {

      a++;

cout << "ProcessID: " << i << endl;

CloseHandle(&i);

    }

  }

cout << "Total Counts = " << a << endl;

}
```

不过俺看了下,好像有些进程没有显示出来,比如 `SVCHOST.exe`、`SMSS.EXE`...

嘿嘿,结合下面这些就更好搞了 [V 大语录]:

引用: killvxk 的驱动查进程：

1.`native api` 获得进程表 `a`

2.通过 `activelist` 获得进程表 `b`

3.通过 `pspCidTable` 获得进程表 `c`

4.通过 `handletablelisthead` 获得进程表 `d`

5.通过 `csrss` 的 `handletable` 用 2 种方法枚举获得进程表 `e` 和 `f`

6.通过扫描当前进程的 `handletable` 获得进程表 `g`

7.遍历表 `c` 的每一个进程的 `SessionProcessLinks` 获得进程表 `h`

8.遍历表 `c` 的每一个进程 `Vm.WorkingSetExpansionLinks` 获得进程表 `i`

9.通过 `Typelist` 分别取 `process` 和 `thread` 的表 `j` 和表 `k`

10.通过表 `k` 得到进程表 `l`

11.搜索内存中的 `threadobject` 和 `processobject` 得到进程表 `m`

12.通过 `Wait/Dispatch` 得到进程表 `n`

13.如果系统是 `Win2003` 以上遍历表 `c` 的每一个进程的 `MmProcessLinks` 得到表 `o`

14.综合上面的进程表得到表 `p`

15.对表 `p` 每一个进程做 `HandleTable`，`Vm.WorkXX`,`MmProcessXX`,`SessionProcessList` 扫描得到表 `q`

16.枚举 `HWNDHandle` 得到进程表 `r`

17.枚举 `JobObject` 得到表 `s`

18.综合得表 `t`,此时枚举结束~~

动态部分：

`KiReadyThread`

和 `KiSwapContext` 的钩子

还有 `KiService` 钩子

还有 `CreateProcessNotifyRoutine` 和 `CreateThreadNotifyRoutine`

`NtCreateThread` 钩子

动态维护一张表，静态枚举结束后综合两表~

其实,隐藏进程越来越难了,保护进程还让其稳定更加不易. 但俺们小菜还是需要掌握这些老掉的技术的,不断学习,不断进步![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

目前通过`job`杀进程 MS 很流行,内存清零也很厉害(就怕 attach 不上),插 APC,杀线程之流的....杀杀 IS 等不在话下.

不过一般搜索到未导出的`PspTerminateProcessByPointer`,再深入点儿把这个函数,甚至`PspExitThread`都自己实现一遍(用 Windbg 看了遍 `PspTerminateThreadByPointer`,发现和 WRK 上的一样.哈哈,没有任何出入 [XP SP2]), Kill 掉大部分进程是木有问题的,不过像 `KV2008` 这样猥亵的家伙在投递 APC 上下了手脚,就要先恢复了 `inline hook` 再杀了.

炉子牛的 R0 那个干掉`KV2008`的好像就是这样搞的.哈哈, 丢个驱动在这里,供有兴趣的 `IDA look` 之(应该无壳). R3 下搞 KV 炉子出了个录象,很神秘的样子,不过好像是成功了.哈哈

对了.炉子牛(又是炉子...)放了个 simple task, R3 下的简单管理器 [`VB`],---

检测隐藏进程的思路很科普:

一个`EnumProcess` or `ToolHelp32`来做对照, 一个`ZwQuerySsytemInformation`来走过场,一个从 0 到 65535 的遍历`OpenProcess`来达到 PspCidTable 的效果. 3 个途径来扫盲式的检测,方法是科普点儿.

结束进程部分很扫盲(原作者: EST 的`willy123`牛):

`ZwCreateJobObject-->ZwAssignProcessToJobObject-->ZwTerminateJobObject`

对付 IS 这样 inlie hook 了`NtOpenProcess`等的就用了一种很和谐的方法,FlowerCode 提供滴.不过是 VB 写的, 俺无聊把其转化为了 C 描述,方便些:

```
//--------------------------------------------------------------

// btw: 头文件的那些申明自己写咯~

// 调用前要有 SE_DEBUG 权限

HANDLE

SDM_OpenProcess (

DWORD dwDesiredAccess,

BOOL  bInhert,

DWORD ProcessId,

BOOL  bOpenIt,

LPDWORD aryPids

  )

/*++

原作者 : FlowerCode | 炉子 [VB]

转换者 : sudami [xiao_rui_119@163.com] [C]

Time   : 08/02/12

参数 :

  dwDesiredAccess - 希望以怎样的方式打开进程

  bInhert - 是否有继承权限

  ProcessId - PID

  bOpenIt - 是要打开进程，还是要保存所有进程 ID

  paryPids - 保存 PID 的数组

返回 :

  成功 - 指定的进程句柄

  失败 - 0

功能 :

  1. 复制来复制去,总之是要获得指定进程的句柄

  2. 用普通方法得到所有进程 ID

--*/

{

ULONG           cbBuffer = 0x1000; //先设定一个较小的缓冲空间

ULONG           uRetSzie;

ULONG           NumOfHandle = 0;

PCHAR           pBuffer = NULL;

PVOID           pOneEprocess = 0;

    CLIENT_ID          cid;

  NTSTATUS          st;

    OBJECT_ATTRIBUTES  oa;

    PROCESS_BASIC_INFORMATION        pbi;

    PSYSTEM_HANDLE_TABLE_ENTRY_INFO  h_info;

HANDLE hProcessToDup, hProcessCur, hProcessToRet;

    oa.Length = sizeof (OBJECT_ATTRIBUTES);

  oa.RootDirectory = 0;

  oa.ObjectName = 0;

  oa.Attributes = 0;

  oa.SecurityDescriptor = 0;
```



```c
  oa.SecurityQualityOfService = 0;  

  if (bInhert) {  
    oa.Attributes |= OBJ_INHERIT;  
  }  

  if (bOpenIt) {  
    // 看能否直接得到句柄  
    cid.UniqueProcess = (HANDLE) (ProcessId + 1);  
    cid.UniqueThread  = 0;  

    st = NtOpenProcess (&hProcessToRet, dwDesiredAccess, &oa, &cid);  
    if (NT_SUCCESS (st)) {  
      return hProcessToRet;  
    }  
    ::`MessageBox` (NULL, "NtOpenProcess 失败", "=。=！", MB_OK);  
  }  

  // 传递 16 号获得所有句柄,可能被 hook 过;不管了,走过场~  
  do {  
    st = ZwQuerySystemInformation( /*SystemHandleInformation*/16, \\  
           pBuffer, cbBuffer, &uRetSzie );  
    if (st == STATUS_INFO_LENGTH_MISMATCH) {  
      `free`( pBuffer);  
      cbBuffer *= 2;  
      pBuffer = (PCHAR) malloc (cbBuffer);  
    } elseif ( !NT_SUCCESS (st) ) {  
      `free`( pBuffer);  
      ::`MessageBox` (NULL, "ZwQuerySystemInformation 失败", ".", MB_OK);  
      return 0;  
    }  
  } while (st == STATUS_INFO_LENGTH_MISMATCH);  

  NumOfHandle = (ULONG) pBuffer;  
  h_info = (PSYSTEM_HANDLE_TABLE_ENTRY_INFO) ( (ULONG)pBuffer + 4 );  

  /*
  // 获得进程对象类型  
  pOneEprocess = h_info[0].Object;  
  g_pObjectTypeProcess = *(PULONG) ( (ULONG)pOneEprocess - OBJECT_HEADER_SIZE \\  
  + OBJECT_TYPE_OFFSET );  

  */

  for (ULONG i = 0; i < NumOfHandle; i++) {  
    aryPids[i] = h_info[i].UniqueProcessId;  
    if (bOpenIt) {  
      ::`MessageBox` (NULL, "333", "=。=！", MB_OK);  
      if ( 5 == h_info[i].ObjectTypeIndex) { // 是进程的句柄，打开它  
        cid.UniqueProcess = (HANDLE)h_info[i].UniqueProcessId;  
        st = NtOpenProcess (&hProcessToDup, PROCESS_DUP_HANDLE, &oa, &cid);  
        if (NT_SUCCESS (st)) { // 复制该句柄后赋予全部权限，以便调用 ZwQuery*时顺利进行  
          st = ZwDuplicateObject (hProcessToDup, (PHANDLE)h_info[i].HandleValue, (HANDLE)-1,\\  
                &hProcessCur, PROCESS_ALL_ACCESS, 0, DUPLICATE_SAME_ATTRIBUTES);  
          if (NT_SUCCESS (st)) { // 查看复制的句柄是否为我们想要的  
            st = ZwQueryInformationProcess (hProcessCur, ProcessBasicInformation,\\  
                  &pbi, sizeof(pbi), 0);  
            if (NT_SUCCESS (st)) {  
              if (ProcessId == pbi.UniqueProcessId ) {  
                // 若是想要的 PID,就把该句柄按照希望的访问权限复制，然后返回  
                st = ZwDuplicateObject (hProcessToDup, (PHANDLE)h_info [i].HandleValue,\\  
                    (HANDLE)-1, &hProcessToRet, dwDesiredAccess,\\  
                    OBJ_INHERIT, DUPLICATE_SAME_ATTRIBUTES);  
                ZwClose (hProcessCur);  
                ZwClose (hProcessToDup);  
                return hProcessToRet;  
              }  
            }      
          }  
          ZwClose (hProcessCur);  
        }  
        ZwClose (hProcessToDup);  
      }  
    }  
  }  

  return 0;  
}
```

顺便问下: 要抹掉`PspCidTable`中指定的进程容易,抹`CSRSS.exe`的也容易,什么挂`SwapContext`的,自己实现线程调度`rootkit.com`上的`PHIDE2 engine`也实现了(颇为复杂,引擎写的很漂亮),还有啥改`ETHREAD`,`EPROCESS`, `flag`等`DKOM`类的...好多好多.但是一个内存暴力搜索不就都出来了.这可怎么隐藏啊![](img/1e00a4c5ba6d273521879fcbd5363999.jpg "碌眼") ...  

---

### 参考资料:

1. [基于 pspCidTable 的进程检测技术](http://hi.baidu.com/gz1x/blog/item/d99aeefa4d1c92ddb48f31b9.html)
2. [句柄啊，3 层表啊，ExpLookupHandleTableEntry 啊](http://hi.baidu.com/sudami/blog/item/47b2ac504302f7581038c206.html)
3. [PsLookupProcessByProcessId 执行流程](http://hi.baidu.com/sudami/blog/item/7773b71fe4545867f624e4f3.html)
4. [PspTerminateThreadByPointer](http://hi.baidu.com/sudami/blog/item/ea1e9107f5c78cc97b894713.html)
5. `WRK`,`ReactOS`

### 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 炉子作品\_\_killkv2008.rar  

 |

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `ProcMgr.rar`  

 |

| ![](img/0f24970173caf34cf302ee22e88648bc.jpg) || `ExEnumHandleTable.txt`  

 |

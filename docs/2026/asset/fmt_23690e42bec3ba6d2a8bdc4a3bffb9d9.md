
```
push  edi
push  11h
pop   ecx
push  1000h       ; 一个页表大小
push  [ebp+pProcess] ; _ERPOCESS指针
xor   eax, eax
mov   edi, esi
rep stosd         ; 分配一页内存
call  _ExpAllocateTablePagedPoolNoZero@8 ; 分配一页内存池,作为第一级句柄表
cmp   eax, ebx    ; 判断是否分配成功
jnz   short AllOC_PAGE_SUCCESS ; 判断DoInit参数是否为FALSE
push  ebx         ; TagToFree
push  esi         ; P
call  _ExFreePoolWithTag@8 ; 释放内存分配的内存池
cmp   [ebp+DoInit], bl ; 判断DoInit参数是否为FALSE
mov   [esi+_HANDLE_TABLE.TableCode], eax ; 将分配的页表基地址赋值给
;HANDLT_TABLE第一项TableCode,建立一级表的映射关系 
```

初始句柄表,设置空闲句柄链表:

```
mov   dword ptr [eax+_HANDLE_TABLE_ENTRY.NextFreeTableEntry], 0FFFFFFFEh 
; 句柄页表的第一个句柄项作为审计用,
; NextFreeTableEntry设置为EX_ADDITIONAL_INFO_SIGNATURE标志
mov   [eax+_HANDLE_TABLE_ENTRY.___u0.Value], ebx ; 设置第一项Value为0
mov   edx, 800h
jz    short DOINIT_FALSE
push  8
pop   ecx         ; ecx = 8
push  4
add   eax, 8      ; 指向第二项HANDLE_TABLE_ENTRY指针
pop   edi         ; 记录空闲项链表,当前为第二项HANDLE_TABLE_ENTRY值为4

END_LOOP:     
; 设置当前项的下一个空闲句柄索引
mov   [eax+_HANDLE_TABLE_ENTRY.___u1.NextFreeTableEntry], ecx   mov   [eax+_HANDLE_TABLE_ENTRY.___u0.Value], ebx ; 设置句柄值为0
add   ecx, edi    ; sizeof(HANDLE_TABLE_ENTRY) = 8,步长为8
add   eax, 8      ; 指向下个HANDLE_TABLE_ENTRY项
cmp   ecx, edx    ; 判断是否到页表倒数第二项
jb    short END_LOOP ; 设置当前项的下一个空闲句柄索引
mov   [eax], ebx
; 页表的最后一项NextFreeTableEntry设置为0，即无下一个空闲句柄
mov   [eax+_HANDLE_TABLE_ENTRY.___u1.NextFreeTableEntry], ebx   mov   [esi+_HANDLE_TABLE.FirstFree], edi   ;设置当前项的下一个空闲项
```

## 建立进程与句柄表的映射关系

```
mov   eax, [ebp+pProcess]
;初始化HANDLE_TABLE的QuotaProcess为传入的EPROCESS指针
mov   [esi+_HANDLE_TABLE.QuotaProcess], eax 
mov   eax, large fs:124h   ; 获取当前线程指针
;设置NextHandleNeedingPool句柄表扩展的起始页句柄索,
; NextHandleNeedingPool记录的是以页为单位
mov [esi+_HANDLE_TABLE.NextHandleNeedingPool],edx
;获取到当前进程EPROCESS指针
mov   eax, [eax+_KTHREAD.___u6.ApcState.Process]
mov   eax, [eax+_EPROCESS.UniqueProcessId] ; 获取进程ID
mov   [esi+_HANDLE_TABLE.UniqueProcessId], eax ; 填充HADLE_TABLE的UniqueProcessId域
xor   eax, eax
mov   [esi+_HANDLE_TABLE.___u12.Flags], ebx ; 设置标记为0 
```

下图是调试时初始化的句柄表的结果:

![](https://bbs.kanxue.com/upload/attach/201006/162380_dkm1kdss329brx8.jpg)

大家可以自己调试一下

## 4.1 查看3层表

`TestThread`是我编写的一个小程序,为了让大家熟悉句柄表结构,让大家见识下3层句柄表.

运行`TestThread`,还是继续等待,泡壶茶,验证奇迹的时刻就要来到.

`kd>` `!Process 0 0`命令查找到断下来的进程信息:

![](https://bbs.kanxue.com/upload/attach/201006/162380_nxbpdvmcnwtjbsu.jpg)

句柄计数:1050601>511\*1024 = 523264,现在应该是3层表结构,下面验证一下:

![](https://bbs.kanxue.com/upload/attach/201006/162380_50tzc0j0fg6igda.jpg)

`TableCode`最低2位为10,则是三层表:

第一层表结构:

![](https://bbs.kanxue.com/upload/attach/201006/162380_vw3ua2emde7omw5.jpg)

第二层表结构:

![](https://bbs.kanxue.com/upload/attach/201006/162380_2j2r8rgzobr7o8q.jpg)

第三层表结构

![](https://bbs.kanxue.com/upload/attach/201006/162380_ms8gadvpisxenyt.jpg)

如果有的看不懂逆向代码的,可以说一声,我把注释的源码发出来

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [进程句柄表与创建句柄表.doc](javascript:void(0);)
  （407.00kb，227次下载）
- [1.JPG](javascript:void(0);)
  （27.13kb，799次下载）
- [2.JPG](javascript:void(0);)
  （12.71kb，801次下载）
- [初始化进程句柄表.JPG](javascript:void(0);)
  （119.55kb，810次下载）
- [分配句柄表.JPG](javascript:void(0);)
  （50.95kb，803次下载）
- [进程与对象.JPG](javascript:void(0);)
  （19.77kb，801次下载）
- [进程与资源.JPG](javascript:void(0);)
  （29.77kb，807次下载）
- [三层句柄表结构.JPG](javascript:void(0);)
  （22.06kb，802次下载）
- [一层句柄表.JPG](javascript:void(0);)
  （7.40kb，802次下载）
- [3.1.JPG](javascript:void(0);)
  （23.44kb，797次下载）
- [3.2.JPG](javascript:void(0);)
  （78.72kb，800次下载）
- [3.3.JPG](javascript:void(0);)
  （68.02kb，789次下载）
- [3.4.JPG](javascript:void(0);)
  （68.40kb，788次下载）
- [3.5.JPG](javascript:void(0);)
  （62.38kb，790次下载）
- [TestThread.rar](javascript:void(0);)
  （231.20kb，253次下载）

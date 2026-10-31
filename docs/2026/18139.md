
# [原创]发个微软的classpnp.sys的0day,可用来执行任意内核代码

首先说明，这个漏洞触发需要管理员权限，用处不是很大，不过做做无驱进ring0,过过主防应该还是有点用的

————————————————————————————————————————

`classpnp.sys`的`ServiceTransferRequest`存在一个安全缺陷，可使应用程序执行任意内核代码,已验证的环境`windows xp sp3`, `ntfs`文件驱动

函数原型`NTSTATUS ServiceTransferRequest(PDEVICE_OBJECT Fdo,PIRP Irp);`

在函数中直接使用了参数`irp`的`MdlAddress`成员，没有对其进行有效性检查，通过某种特征的函数访问路径可使`IRP`的`MdlAddress`为0而导致`BSOD`,函数头逆向如下

`CLASSPNP!ServiceTransferRequest`:

```
f7562bfd 8bff            mov     edi,edi
f7562bff 55              push    ebp
f7562c00 8bec            mov     ebp,esp
f7562c02 83ec18          sub     esp,18h
f7562c05 8b4508          mov     eax,dword ptr [ebp+8]
f7562c08 8b4028          mov     eax,dword ptr [eax+28h]
f7562c0b 53              push    ebx
f7562c0c 8b98b4020000    mov     ebx,dword ptr [eax+2B4h]
f7562c12 56              push    esi
f7562c13 8b750c          mov     esi,dword ptr [ebp+0Ch]//irp获取参数irp
f7562c16 8b4e04          mov     ecx,dword ptr [esi+4]//irp.mdladdress得到irp的mdladdres成员
f7562c19 8b4660          mov     eax,dword ptr [esi+60h]
f7562c1c 8b5004          mov     edx,dword ptr [eax+4]
f7562c1f 57              push    edi
f7562c20 8b7918          mov     edi,dword ptr [ecx+18h]//mdl.byteoffset访问18h会导致bsod
f7562c23 037910          add     edi,dword ptr [ecx+10h]//mdl.startVa
```

通过某种特殊的访问可导致上述的`IRP`中的`mdladdres`为0导致蓝屏，重现步骤

1、获取`system`进程中`pagefile`文件的句柄

2、通过此句柄来读取`pagefile`文件

具体重现步骤请见附件中的`poc`文件夹，`bsod`时的函数栈如下：

`STACK_TEXT`:

```
b2a3a980 f7562dcd 82b2a9c0 82642008 82642008 CLASSPNP!ServiceTransferRequest+0x23
b2a3a9a4 804e23c9 82b2a9c0 00000000 826b2030 CLASSPNP!ClassReadWrite+0xff
b2a3a9b4 f77aa921 826b7c70 8264212c b2a3a9f0 nt!IopfCallDriver+0x31
b2a3a9c4 804e23c9 82b2a798 82642008 82642150 PartMgr!PmReadWrite+0x95
b2a3a9d4 f74a41c6 826b53a0 82b9fa58 82642008 nt!IopfCallDriver+0x31
b2a3a9f0 804e23c9 826b7bb8 82642008 826b2f38 ftdisk!FtDiskReadWrite+0x194
b2a3aa00 f754251a 826b77e8 826b77e8 b2a3aae8 nt!IopfCallDriver+0x31
b2a3aa10 804e23c9 826b77e8 82642008 bbfca000 VolSnap!VolSnapRead+0x26
b2a3aa20 f737aa19 829bd608 825ca008 82727008 nt!IopfCallDriver+0x31
b2a3aae8 f73807e2 82727008 82642008 825ca008 Ntfs!NtfsPagingFileIo+0x1b2
b2a3abc4 f737a00a 82727008 82642008 00000001 Ntfs!NtfsCommonRead+0x2bd
b2a3ac64 804e23c9 826b52c0 82642008 826b7400 Ntfs!NtfsFsdRead+0x22d
b2a3ac74 f741b459 b2a3aca0 804e23c9 826b5c88 nt!IopfCallDriver+0x31
b2a3ac7c 804e23c9 826b5c88 82642008 80702410 sr!SrPassThrough+0x31
b2a3ac8c 805716e2 82642174 82642008 82895580 nt!IopfCallDriver+0x31
b2a3aca0 8057fb27 826b5c88 82642008 82895580 nt!IopSynchronousServiceTail+0x70
b2a3ad38 804de98f 00000080 00000084 00000000 nt!NtReadFile+0x55d
b2a3ad38 7c92e4f4 00000080 00000084 00000000 nt!KiFastCallEntry+0xfc
0012ddc0 7c92d9bc 7c80199d 00000080 00000084 ntdll!KiFastSystemCallRet
0012ddc4 7c80199d 00000080 00000084 00000000 ntdll!ZwReadFile+0xc
0012de2c 0042f3ba 00000080 0012df18 00002000 kernel32!ReadFile+0x10d
0012ff54 0042ffc2 00000001 009a2fb8 009a3058 poc!wmain+0xba
0012ffb8 0042fd7d 0012fff0 7c817067 80000001 poc!__tmainCRTStartup+0x232
0012ffc0 7c817067 80000001 0007ddb4 7ffd6000 poc!wmainCRTStartup+0xd
0012fff0 00000000 0042c5aa 00000000 78746341 kernel32!BaseProcessStart+0x23
```

通过此漏洞在应用层执行任意内核代码的演示请见附件中的文件夹`exploit`,请在`ide`磁盘上实验，`scsi`实现要复杂些，没有给出

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [classpnpCN.rar](javascript:void(0);)

  （171.34kb，161次下载）

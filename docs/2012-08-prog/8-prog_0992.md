
# 分页机制 打造自己的变速齿轮

> 原文：[`www.pediy.com/kssd/pediy12/142326.html`](https://www.pediy.com/kssd/pediy12/142326.html)

传统变速齿轮通过 `HOOK GetTickCount` 时间函数 之后修改返回值获得效果。

这样做的好处是方便,不好是容易被一些反的游戏检测出来.或者游戏自己仿写一个 `GetTickCount` 也不难.

因为分析 `GetTickCount` 的动作。其实是访问了进程内存地址 `$7FFE0000`。

进程内存地址 `$7FFE0000` 上是一个不断更新的数值。其更新是由系统来完成。系统中 `$FFDF0000` 地址正是跟进程上映射到相同的物理地址。

那么我们可以在系统中再创建一个空间，作为一个假的时间记录空间，然后由系统更新 `$FFDF0000` 的时候顺便更新这个假的记录空间，当然这个是按照我们的意思来更新（即时间变快），然后将这要改变速度的目标进程的内存地址 `$7FFE0000` 映射改为我们这个假的空间。那么无论是 `GetTickCount` 函数获得的都是只按我们意愿改变的时间。

原理说完了。来看下代码：

首先。我们在驱动入口时，此时位于系统上下文，我们应该做些什么。

### 1. 获得系统更新时间函数的地址

代码:

```
 RTlInitAnsiString(@g_KeUpdateSystemTime_Name_Ansi,'KeUpdateSystemTime');
 RtlAnsiStringToUnicodeString(@g_KeUpdateSystemTime_Name_Unicode,@g_KeUpdateSystemTime_Name_Ansi,true);   //字符
 g_KeUpdateSystemTime_Address:=MmGetSystemRoutineAddress(@g_KeUpdateSystemTime_Name_Unicode);
 g_KeUpdateSystemTime_Address_C:=cardinal(g_KeUpdateSystemTime_Address);
 RtlFreeUnicodeString(@g_KeUpdateSystemTime_Name_Unicode);
```

### 2. 因为我们要 HOOK 这个函数（为的是系统更新时间时一并替我们更新假的空间上数据），所以要看下这个函数的版本是否适合（我用的是 XP SP3）,这个叫硬编码吧?我使用了....没办法..技术的确不高...

代码:

```
 if g_KeUpdateSystemTime_Address<>nil then
 begin
  g_pmdlSystemCall:=MmCreateMdl(nil,g_KeUpdateSystemTime_Address,$6D);
  if g_pmdlSystemCall<>nil then
  begin
   MmBuildMdlForNonPagedPool(g_pmdlSystemCall);
   g_pmdlSystemCall^.MdlFlags:=g_pmdlSystemCall^.MdlFlags or MDL_MAPPED_TO_SYSTEM_VA;
   MappedSystemCallTable:=MmMapLockedPages(g_pmdlSystemCall,KernelMode);
   Byte_KeUpdateSystemTime:=g_KeUpdateSystemTime_Address;
   if Byte_KeUpdateSystemTime^=$B9 then
   begin
    DbgPrint('版本匹配成功 1!'#13#10);
    Cardinal_KeUpdateSystemTime:=PTR(g_KeUpdateSystemTime_Address_C+1);
    if Cardinal_KeUpdateSystemTime^=$FFDF0000 then
    begin
     DbgPrint('版本匹配成功 2!'#13#10);
     SysBBOK:=true;
    end;
   end;
  end;
 end;
```

### 3. 改写时间更新函数的代码，一个跳转跳到我们的自己的代码的地址上去。。

代码:

```
  HST1Next:=PTR(g_KeUpdateSystemTime_Address_C+5);
  HST1To:=cardinal(@HST1)-g_KeUpdateSystemTime_Address_C-5;
  Cardinal_KeUpdateSystemTime:=PTR(g_KeUpdateSystemTime_Address_C+1);
  Byte_KeUpdateSystemTime:=g_KeUpdateSystemTime_Address;
  Cardinal_KeUpdateSystemTime^:=HST1To;
  Byte_KeUpdateSystemTime^:=$E9;
```

### 4. 我们自己的代码，看看做了什么。

代码:

```
procedure HST1;
asm
 push eax
 push ebx
 push ecx
 push edx
 call HST1EX    //调用顺便更新假空间的函数
 pop edx
 pop ecx
 pop ebx
 pop eax
 mov ecx,$FFDF0000   //恢复我们改写的内容,在系统更新时间函数上我们的跳转覆盖了这个指令
 jmp HST1Next         //跳回去下一个指令
end;
```

### 5. 而 HST1EX（调用顺便更新假空间的函数）所做的：

代码:

```
procedure HST1EX;
label L;
begin
 if HST1_OPEN=1 then
 begin
  HST1_GT_Temp1:=PTR($FFDF0004);
  HST1_GT1_INGG:=HST1_GT_Temp1^;
  HST1_GT_Temp1:=PTR($FFDF0000);                 //取系统上真实的时间.
  HST1_GT1:=HST1_GT_Temp1^;
  HST1_GT_Temp2:=PTR($FFDF0008);
  HST1_GT2:=HST1_GT_Temp2^;
  HST1_GT_Temp3:=PTR($FFDF0014);
  HST1_GT3:=HST1_GT_Temp3^;
  
  if HST1_GT_JiaShu=0 then
  begin
   HST1_GT1_Save:=HST1_GT1;
   HST1_GT2_Save:=HST1_GT2;
   HST1_GT3_Save:=HST1_GT3;
  end;

//加快的时间=真实系统时间+(真实系统时间-原记录系统时间)\*要改变的速度倍数
  HST1_GT1_ING:=HST1_GT1+(HST1_GT1-HST1_GT1_Save)\*(HST1_GT_JiaShu);

//  HST1_GT2_ING:=HST1_GT2+(HST1_GT2-HST1_GT2_Save)\*(HST1_GT_JiaShu);
  HST1_GT_CC1:=HST1_GT_JiaShu;
  HST1_GT_CC2:=HST1_GT2-HST1_GT2_Save;
  //HST1_GT2_ING:=HST1_GT_CC1\*HST1_GT_CC2;
  llmul(HST1_GT_CC1,HST1_GT_CC2,HST1_GT2_ING);
  HST1_GT2_ING:=HST1_GT2+HST1_GT2_ING;

  HST1_GT_TempC2:=@HST1_GT2_ING;
  inc(HST1_GT_TempC2);
  HST1_GT2_INGG:=HST1_GT_TempC2^;
  
//  HST1_GT3_ING:=HST1_GT3+(HST1_GT3-HST1_GT3_Save)\*(HST1_GT_JiaShu);
  HST1_GT_CC1:=HST1_GT_JiaShu;
  HST1_GT_CC2:=HST1_GT3-HST1_GT3_Save;
  //HST1_GT3_ING:=HST1_GT_CC1\*HST1_GT_CC2;
  llmul(HST1_GT_CC1,HST1_GT_CC2,HST1_GT3_ING);
  HST1_GT3_ING:=HST1_GT3+HST1_GT3_ING;

  HST1_GT_TempC3:=@HST1_GT3_ING;
  inc(HST1_GT_TempC3);
  HST1_GT3_INGG:=HST1_GT_TempC3^;


  HST1_GT_Temp1:=g_pSharedMemory;
  HST1_GT_Temp1^:=HST1_GT1_ING;
  Inc(HST1_GT_Temp1);
  HST1_GT_Temp1^:=HST1_GT1_INGG;

  HST1_GT_Temp2:=PTR(cardinal(g_pSharedMemory)+$8);
  HST1_GT_Temp2^:=HST1_GT2_ING;

  HST1_GT_Temp1:=PTR(cardinal(g_pSharedMemory)+$10);
  HST1_GT_Temp1^:=HST1_GT2_INGG;

  HST1_GT_Temp3:=PTR(cardinal(g_pSharedMemory)+$14);
  HST1_GT_Temp3^:=HST1_GT3_ING;

  HST1_GT_Temp1:=PTR(cardinal(g_pSharedMemory)+$1C);
  HST1_GT_Temp1^:=HST1_GT3_INGG;

  g_pSharedMemory_Temp:=PTR(cardinal(g_pSharedMemory)+$20);
  HST1_Memory_Temp:=PTR($FFDF0020);
  HST1_N:=0;
L:
  g_pSharedMemory_Temp^:=HST1_Memory_Temp^;
  inc(g_pSharedMemory_Temp);       //除了更新时间地址,还得把后面空间上的内容一并复制上去.
  inc(HST1_Memory_Temp);
  HST1_N:=HST1_N+1;
  if HST1_N<1016 then
  begin
   goto L;
  end;
 end;
end; 
```

好了,现在我们拥有了一个和 `$FFDF0000` (一个页面)上面内容一模一样的页面,并且上面的时间记录数据是可以按我们的意思改变的.

然后我们该去改变目标进程了.找到关于内存映射到物理地址的表.

### 1. 首先.我们要知道 分页机制开启 CR0 中最高位 PG 位控制分页管理机制

代码:

```
    asm
     mov eax,cr0
     mov GetMyCR0,eax
    end;
```

### 2. 如果 GetMyCR0 最高位为 1,那么我们得知道 开启 PAE 情况

代码:

```
    asm
     mov eax,cr4
     mov GetMyCR4,eax
    end;
```

### 3. 如果 GetMyCR4 第 5 位上为 0.那么就为未开启 PAE,否则开启 PAE

### 4. 对于两种情况的处理,我们这里还先得获得目标进程的 CR3

代码:

```
    if PsLookupProcessByProcessId(PID,GetThePeProcess)=STATUS_SUCCESS then
    begin
     ObfDereferenceObject(GetThePeProcess);
     GetThePeProcessP:=GetThePeProcess;
     GETPIDCR3:=GetThePeProcessP^.Pcb.DirectoryTableBase;
    end;
```

### 5. 未开启 PAE 情况

代码:

```
 PDEBA:=uCR3;
 dwPDEIndex:=GetCardinalBIT(VirtualAddr,22,10); //取线性地址的高 2 位作为选取页目录指针表项的索引
 dwToPTE:=PDEBA+dwPDEIndex\*4;   //计算指针表项地址
 //读出地址数据
 CLow:=ReadPhyMem(dwToPTE);

 if GetCardinalBIT(CLow,7,1)=1 then  //使用的是 4MB 的页
 begin
  exit;
 end;

 CI:=GetCardinalBIT(CLow,12,20);
 CI:=CI shl 12;

 PTEBA:=CI;
 dwPTEIndex:=GetCardinalBIT(VirtualAddr,12,10);
 dwToP:=PTEBA+dwPTEIndex\*4;   //计算指针表项地址

 result:=dwToP;
```

### 6. 开启 PAE 情况

代码:

```
 PDPTEBA:=uCR3;
 dwPDPTEIndex:=GetCardinalBIT(VirtualAddr,30,2); //取线性地址的高 2 位作为选取页目录指针表项的索引
 dwToPDE:=PDPTEBA+dwPDPTEIndex\*8;   //计算指针表项地址
 //读出地址数据
 CLow:=ReadPhyMem(dwToPDE);
 CHigh:=ReadPhyMem(dwToPDE+4);
 CI:=GetCardinalBIT(CLow,12,20)+GetCardinalBIT(CHigh,0,4) shl 20;
 CI:=CI shl 12;

 PDEBA:=CI;
 dwPDEIndex:=GetCardinalBIT(VirtualAddr,21,9);
 dwToPTE:=PDEBA+dwPDEIndex\*8;   //计算指针表项地址

 //读出地址数据
 CLow:=ReadPhyMem(dwToPTE);
 CHigh:=ReadPhyMem(dwToPTE+4);

 if GetCardinalBIT(CLow,7,1)=1 then  //使用的是 2MB 的页
 begin
  exit;
 end;

 CI:=GetCardinalBIT(CLow,12,20)+GetCardinalBIT(CHigh,0,4) shl 20;
 CI:=CI shl 12;

 PTEBA:=CI;
 dwPTEIndex:=GetCardinalBIT(VirtualAddr,12,9);
 dwToP:=PTEBA+dwPTEIndex\*8;   //计算指针表项地址

 result:=dwToP;
```

得到表记录的地址之后,剩下的事情就是将记录内容(原指向的物理地址),改为我们假空间的物理地址即可..

### 未开启

代码:

```
    CLow:=ReadPhyMem(Addr);
    CI:=GetCardinalBIT(CLow,12,20);
    CI:=CI shl 12;
    S1:=PNAME+';'+inttostr(PID)+';'+inttostr(Addr)+';'+inttostr(CLow)+';'+''+';'+inttostr(CI)+';';
    CLow:=MYPLYMEM+GetCardinalBIT(CLow,0,12);
    WritePhyMem(Addr,CLow);
    ListBox2.Items.Add(S1);
```

### 开启

代码:

```
    CLow:=ReadPhyMem(Addr);
    CHigh:=ReadPhyMem(Addr+4);
    CI:=GetCardinalBIT(CLow,12,20)+(GetCardinalBIT(CHigh,0,4) shl 20);
    CI:=CI shl 12;
    S1:=PNAME+';'+inttostr(PID)+';'+inttostr(Addr)+';'+inttostr(CLow)+';'+inttostr(CHigh)+';'+inttostr(CI)+';';
    CI:=MYPLYMEM;
    CLow:=GetCardinalBIT(CLow,0,12)+(GetCardinalBIT(CI,12,20) shl 12);
    CHigh:=(GetCardinalBIT(CHigh,4,28) shl 4)+GetCardinalBIT(CI,32,5);
```

最后,因为这个工具是很久之前写的了.写得比较乱,所以现在我自己也不能完全理解了!!只能从其中摘要一点发出来.全部详细请自行分析源代码.

最后上工程源代码:

编译环境:程序:`Delphi 6` 驱动:`Delphi6 + KmdKit4D`

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `BXCL.rar`
|

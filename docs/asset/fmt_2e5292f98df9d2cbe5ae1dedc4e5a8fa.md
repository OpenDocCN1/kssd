
```c
//======================================获取 pspcidtable 地址 CODE1=======================
//利用 PsLookupXX 函数搜索特征码 0x35ff 和 0xe8 获取
ULONG GetAddrFromProcessId()
{   
  UNICODE_STRING pslookup;
  PUCHAR addr;               //单字节指针
  PUCHAR p;                  //单字节指针
  ULONG q;                   //四字节
  RtlInitUnicodeString(&pslookup,L"PsLookupProcessByProcessId");
  addr=(PUCHAR)MmGetSystemRoutineAddress(&pslookup);
  KdPrint(("[PsLookupProcessByProcessId] addr:0x%x\\n",addr));
  for(p=addr;p<addr+PAGE_SIZE;p++)
  {
    if((*(PUSHORT)p==0x35ff)&&(*(p+6)==0xe8))
    {   
      q=*(PULONG)(p+2);
      KdPrint(("[GetAddrFromProcessId] pspcidtable:0x%x\\n",q));
      return q;
      break;
    }
  }
  return 0;
}
//======================================获取 pspcidtable 地址 CODE2=======================

//利用 KPCR 取得
ULONG GetAddrFromKpcr()
{
  ULONG pspcidtable;
  //#define kpcr   0xffdff000
  //kpcr+0x37 是 KdVersionBlock
    //KdVersionBlock+0x80 是 pspcidtable 指针
  pspcidtable=*((PULONG)((*(PULONG)(kpcr+0x34)) + (ULONG)(0x80)));
  KdPrint(("[GetAddrFromKpcr] pspcidtable:0x%x\\n",pspcidtable));
  return pspcidtable;
  
}
//====================================获取 pspciatable 地址 CODE3=========================
//KdEnableDebugger->KdInitSystem->KdDebuggerDataBlock->KDDEBUGGER_DATA32->PspCidTable

//======================================================================================
//====================================遍历 pspcidtable CODE1=========================
//利用导出的 ExEnumHandleTable（ntoskrnl.exe 导出函数），可以直接操作，不需要根据 GMM 自己定位地址

//================================================================================
//====================================自己遍历 pspcidtable 表=========================
//=============通过当前进程获取进程对象指针(PsGetCurrentProcess 函数获取 object 指针转换成 object_header 里面记录对象类型)===============
typedef struct _OBJECT_HEADER
{
  union
  {
    struct
    {
      LONG PointerCount;
      LONG HandleCount;
    };
    LIST_ENTRY Entry;
  };
  POBJECT_TYPE Type;
  UCHAR NameInfoOffset;
  UCHAR HandleInfoOffset;
  UCHAR QuotaInfoOffset;
  UCHAR Flags;
  union
  {
    //POBJECT_CREATE_INFORMATION ObjectCreateInfo;
    PVOID QuotaBlockCharged;
  };
  PSECURITY_DESCRIPTOR SecurityDescriptor;
  QUAD Body;
} OBJECT_HEADER, *POBJECT_HEADER;
ULONG  GetProcessType()
{
  ULONG  type;
  ULONG  objecttoeprocess;
  objecttoeprocess=(ULONG)PsGetCurrentProcess();
  //object 转换成 object_header 完全可以 object-0x18
  objecttoeprocess=(ULONG)OBJECT_TO_OBJECT_HEADER(objecttoeprocess);
  type=*(PULONG)(objecttoeprocess+TYPE);
  KdPrint(("[GetProcessType] type:0x%x\\n",type));
  return type;
}
//===================================采用单链表记录进程 Eprocess 地址================================================
void RecordProcess(ULONG address) //address   
{   
  PROCESSINFO *r;
  if(head==NULL)
  {
    head=(PROCESSINFO *)ExAllocatePoolWithTag(NonPagedPool,sizeof(PROCESSINFO),MEM_TAG);//分配头指针
    if(head==NULL)
    {
      KdPrint(("[RecoardProcess] Aloocate error\\n"));
    }
    //分配内存，用完必须释放，否则内存泄漏
    head->addr=0x0;
  }
  if(head->addr==0x0)
  {
    head->addr=address;
    KdPrint(("[RecordProcess] head->addr:0x%x",head->addr));
    p=head;
  }
  else
  {
    r=(PROCESSINFO *)ExAllocatePoolWithTag(NonPagedPool,sizeof(PROCESSINFO),MEM_TAG);
    if(r==NULL)
    {
      KdPrint(("[RecoardProcess] Aloocate error\\n"));
    }
    p->next=r;
    p=r;
    r->addr=address;
    KdPrint(("[RecoardProcess] r->addr:0x%x\\n",r->addr));
    r->next=NULL;
  }
}
//==================================根据 PID 找对象指针=====================================================
void GetPointerToObject(ULONG table,ULONG pid)//函数返回的 object 即指向 EPROCESS 的指针
{    
    
    ULONG object,objectheader;
    ULONG NextFreeTableEntry;
    ULONG processtype,type;
    ULONG flags;
    processtype=GetProcessType();//调用函数获取进程类型
    if(MmIsAddressValid((PULONG)(table+pid*2)))
    {   
        if(MmIsAddressValid((PULONG)(table+ pid*2 +NEXTFREETABLEENTRY)))
        {
            
            NextFreeTableEntry=*(PULONG)(table + pid*2 + NEXTFREETABLEENTRY);
            if(NextFreeTableEntry==0)//正常的 handle_table_entry 中 NextFreeTableEntry 为 0
            {
                object=*(PULONG)(table+pid*2);
                object=((object | 0x80000000)& 0xfffffff8);//转换为对象指针
                KdPrint(("[GetPointerToObject] object:0x%x\\n",object));    //函数要记录的进程 ERROCESS 地址
                objectheader=(ULONG)OBJECT_TO_OBJECT_HEADER(object);//获取对象头指针
                KdPrint(("[GetPointerToObject] objectheader:0x%x\\n",objectheader));
                if(MmIsAddressValid((PULONG)(objectheader+TYPE)))
                {   
                    type=*(PULONG)(objectheader+TYPE);
                    if(type==processtype)//表明是进程对象
                    {   
                        flags=*(PULONG)(object+FLAGS);//EPROCESS 中 Flags 偏移量，指明了进程的死活
                        //KdPrint(("[GetPointerToObject]) flags:0x%x\\n",flags)
                        if((flags&0xc)!=0xc)//死进程的 flags 最后一位为 C
                        {
                            RecordProcess(object);
                      num=num+1;  //进程数量
                        }       
                    }
                }
            }
        }
        
    }
    
    
}

//==================================遍历 pspcidtable 列举进程信息============================================
void ListProcess()
{
  ULONG pspcidtable;
  ULONG tablecode;
  ULONG table1,table2,table3,table4,table5;  //2 层表正常情况下足够存放进程了，所以 3 层表就不考虑
  
    ULONG cid;
  ULONG NextHandleNeedingPool;
  
  pspcidtable=GetAddrFromKpcr();  //取 pspcidtable 地址
  KdPrint(("[ListProcess]pspcidtable:0x%x\\n",pspcidtable));
  if(MmIsAddressValid((PULONG)pspcidtable))//地址是否有效
  {
    tablecode=*(PULONG)(*(PULONG)pspcidtable);//取 tablecode 地址
    NextHandleNeedingPool=*(PULONG)(*(PULONG)pspcidtable+0x038);//NextHandleNeedingPool 值
    KdPrint(("[ListProcess] tablecode:0x%x\\n",tablecode));
    KdPrint(("[ListProcess] NextHandleNeedingPool:0x%x\\n",NextHandleNeedingPool));  //PID 最大值
    if((tablecode & 0x00000003)==0)//1 层表存放的就是 handle_table 的基址
    {
      table1=tablecode;
      KdPrint(("[ListProcess] table1:0x%x\\n",table1));
      table2=0;
    }
    else if((tablecode & 0x00000003)==1)//2 层表，一级表存放的是指向 2 级表的指针
    {    
      tablecode=tablecode&0xfffffffe;  //低二位清零
            table1=*(PULONG)tablecode;    //1 个表存放 0x800=2048 大的 PID，不够大   
      table2=*(PULONG)(tablecode+4);//2 个表存放 0x1000=4096 大的 PID，不够大
            table3=*(PULONG)(tablecode+8);//3 个表存放 0x1800=6144 大的 PID，不够大
      table4=*(PULONG)(tablecode+12);//4 个表可以存放 0x2000=8192 大的 PID,还不够大啊
            table5=*(PULONG)(tablecode+16);//5 个表可以存放 0x2800=10240 的的 PID，足够了吧
      KdPrint(("[ListProcess] table1:0x%x\\n",table1));
            KdPrint(("[ListProcess] table2:0x%x\\n",table2));
      KdPrint(("[ListProcess] table3:0x%x\\n",table3));
      KdPrint(("[ListProcess] table4:0x%x\\n",table4));
            KdPrint(("[ListProcess] table5:0x%x\\n",table5));
    }
        //遍历
     
    for(cid=0;cid<NextHandleNeedingPool;cid=cid+4)//要加 4
    {
      if((table1!=0)&&(cid<=0x800))//在第一个表中
      {
                GetPointerToObject(table1,cid);
      }
      if((table2!=0)&&(0x800<cid && cid<=0x1000))//在第二个表中
      {
        cid=(ULONG)(cid-0x800);
        //KdPrint(("[ListProcess] cid 0x%x\\n",cid));
        GetPointerToObject(table2,cid);
                cid=(ULONG)(cid+0x800);
                //KdPrint(("[ListProcess] cid 0x%x\\n",cid));
      }
      if((table3!=0)&&(0x1000<cid && cid<=0x1800))//在第三个表中
      {
        cid=(ULONG)(cid-0x1000);
        GetPointerToObject(table3,cid);
        cid=(ULONG)(cid+0x1000);
        
      }
      if((table4!=0)&&(0x1800<cid && cid<=0x2000))//在第四个表中
      {
        cid=(ULONG)(cid-0x1800);
        GetPointerToObject(table4,cid);
        cid=(ULONG)(cid+0x1800);
      }
      if((table5!=0)&&(0x2000<cid && cid<=0x2800))//在第五个表中
      {
        cid=(ULONG)(cid-0x2000);
        GetPointerToObject(table5,cid);
        cid=(ULONG)(cid+0x2000);
      }
    }
    
  }
  
}
//
//取进程全路径====================================================================
//原理 Eprocess->sectionobject(0x138)->Segment(0x014)->ControlAera(0x000)->FilePointer(0x024)->(FileObject->FileName,FileObject->DeviceObject)
void GetProcessPath(ULONG eprocess,CHAR ProcessPath[256])
{
  ULONG object;
  PFILE_OBJECT FilePointer;
  UNICODE_STRING path;  //路径
  UNICODE_STRING name;  //盘符
  ANSI_STRING  string;
  path.Length=0;
    path.MaximumLength=256;
  path.Buffer=(PWCHAR)ExAllocatePoolWithTag(NonPagedPool,256,MEM_TAG);     //必须释放
  if(MmIsAddressValid((PULONG)(eprocess+0x138)))//Eprocess->sectionobject(0x138)
  {
    object=*(PULONG)(eprocess+0x138);
        KdPrint(("[GetProcessFileName] sectionobject :0x%x\\n",object));
    if(MmIsAddressValid((PULONG)((ULONG)object+0x014)))
    {
      object=*(PULONG)((ULONG)object+0x014);
      KdPrint(("[GetProcessFileName] Segment :0x%x\\n",object));
      if(MmIsAddressValid((PULONG)((ULONG)object+0x0)))
      {
        object=*(PULONG)((ULONG_PTR)object+0x0);
        KdPrint(("[GetProcessFileName] ControlAera :0x%x\\n",object));
        if(MmIsAddressValid((PULONG)((ULONG)object+0x024)))
        {
          object=*(PULONG)((ULONG)object+0x024);
          KdPrint(("[GetProcessFileName] FilePointer :0x%x\\n",object));
        }
        else
          return ;
      }
      else
        return ;
    }
    else
      return ;
  }
  else
    return ;
    FilePointer=(PFILE_OBJECT)object;
    //KdPrint(("[GetProcessFileName] FilePointer :%wZ\\n",&FilePointer->FileName));
  ObReferenceObjectByPointer((PVOID)FilePointer,0,NULL,KernelMode);//引用计数+1，操作对象
  RtlVolumeDeviceToDosName(FilePointer->DeviceObject,&name); //获取盘符名
  //KdPrint(("[GetProcessFileName] FilePointer :%wZ\\n",&name));
    RtlCopyUnicodeString(&path,&name);//盘符连接
  RtlAppendUnicodeStringToString(&path,&FilePointer->FileName);//路径连接
  //KdPrint(("[GetProcessFileName] FilePointer :%wZ\\n",&path));
  ObDereferenceObject(FilePointer);         //关闭对象引用
  //需要转换成 ANSI_STRING，然后在转换成 char 输出给 ring3
  RtlUnicodeStringToAnsiString(&string,&path,TRUE);    //释放内存
  if(string.Length >= 256 ) //保证以\0 结尾
  { 
    memcpy(ProcessPath, string.Buffer, 256); 
    *(ProcessPath + 255) = 0; 
  } 
  else 
  { 
    memcpy(ProcessPath, string.Buffer, string.Length); 
    ProcessPath[string.Length] = 0; 
  }
  ExFreePool(path.Buffer); //释放
  RtlFreeAnsiString(&string);//释放
}
//====================================进程信息==========================================================
void  GetProcessInformation()//获取进程名，PID，路径
{
  ListProcess();       //调用函数
  KdPrint(("[DispatchIoctl] num:%d\\n",num));//打印出进程数量
  for(p=head;MmIsAddressValid(p);p=p->next)     //写实现功能，获取进程 PID，进程名，图标、全路径
  {    
    //KdPrint(("===============================================\\n"));
    KdPrint(("[GetProcessInformation] EPROCESS:0x%x\\n",p->addr));
    //EPROCESS 获取进程名
    //XP 下 0x084 偏移存放的进程 PID
    //XP 下 0x174 偏移存放的进程 ProcessName
    p->pid=*(int *)(p->addr+CID);
    KdPrint(("[GetProcessInformation] PID:%d\\n",p->pid));          //进程 PID
    KdPrint(("===============================================\\n"));
    memcpy(p->name,(PUCHAR)(p->addr+NAME),16);
    KdPrint(("[GetProcessInformation] ProcessName:%s\\n",p->name));  //进程名    
    //获取全路径
    GetProcessPath(p->addr,p->Path);
    KdPrint(("[GetProcessInformation] ProcessPath:%s\\n",p->Path)); //进程全路径
  }
}
//==================================释放链表函数=========================================================
void FreeList()
{
    //释放链表,释放内存
  PROCESSINFO *q;
    p=head;
  q=p->next;
  while(q!=NULL)
  {
    KdPrint(("[DispathIoctl] p->addr:0x%x\\n",p->addr));
    ExFreePool(p);
    p=q;
    q=p->next;
    n=n+1;
  }
  KdPrint(("[DispathIoctl] p->addr:0x%x\\n",p->addr));
  ExFreePool(p);
  head=NULL;        //不知道 ExFreePool 释放后 head 竟然不为 NULL.p 和 head 都是全局变量
  n=n+1;
  KdPrint(("[DispatchIoctl] n:%d\\n",n));//打印出释放进程数量
  n=0;
}
```

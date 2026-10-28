
# [原创]注册表文件解析完整解决方案

UESTC 鹿剑

开始是看大米的那个增删改方案，不过，由于好多GUI处理干扰，头很大，最后到网上找到了下面这个库，大米那个其实就是基于这个改写的,最近要用到这个，所以好好看了看，做了良好的封装，支持几乎所有操作：

枚举子键、枚举键值，修改、删除、增加键值、子键，关键是接口良好，移植使用相当方便，就算不懂注册表格式的程序员移植也很简单。

注册表格式虽然已经可以找到相对完整资料，但是自己来解析的话难度还是有难度，现今比较完整的资料都是英文，这个对于英文不太好的同学来说是个不小的挑战，另外要读懂那些资料然后自己写出完整代码来也不是一件容易的事，最简单的做法就是在前人的代码上来进行二次开发。网上比较完整的资料就是这个了：

`NTREG - Window registry file reader / writer library`

作者是 `Petter Nordahl-Hagen`.

下载地址可以使用谷歌搜索引擎来获取，输入关键字`Ntreg.h`之后前几页都是，请不要使用百度，然后找不到时问我要下载地址，相信外事问谷歌，内事问百度这句话你还是知道的。下载回来之后一般直接编译成静态库就可以了，附件里有我编译好的静态库。原作者的代码是使用运行时C写的，所以在使用时请使用`extern "C"`包含头文件。废话不多说，接下来正式编程，首先遇到的问题是访问被系统独占的文件，XP下可以枚举系统句柄表，`DuplicateHandle`句柄过来，原因就是xp下可以用复制句柄权限打开system进程，到了Windows7下就完全不行了，用复制句柄权限无法打开system进程（我很意外，为什么资源监视器还是能看到system进程打开的文件呢？自己逆向水平太差了，没本事去分析资源监视器，还望高手指点一二^\_^）,所以为了通用，我写了一个驱动来实现句柄权限修改，都到了驱动，可以访问注册表文件的方法很多了，直接使用驱动读写文件等等，我之所以选择修改句柄权限的方法是因为方法稳定，比较简单，首先获取进程句柄表，遍历句柄表，找到目标句柄，修改权限，为了通用（其实是我也真的不知道到底那个权限域和SDK里面的权限对应关系），用户态传入两个句柄，目标句柄会获取到源句柄相同的访问权限，这样就不需要关心在各个系统下SDK里面的权限和句柄对象权限的对应关系了，我也没有能力像别人一样实现句柄表的手工解析，资料倒是很多，读起来头大，我又比较懒，所以使用系统提供的`ExEnumHandleTable`函数了，一句足以搞定，手动解析还要分层次，麻烦。驱动代码大致如下:

```
bool EnumHandleCallBack(PHANDLE_TABLE_ENTRY Entry,HANDLE handle,ENUMHANDLE_PARAMETER* Param)
{
	KdPrint(("函数被调用! 句柄：%08x\n",handle));
	if(Param->AccessMask==0)
	{
		if(handle==Param->SorHadnle)
		{
			KdPrint(("获取源权限成功！"));
			Param->AccessMask=Entry->GrantedAccess;
			return true;
		}
	}
	else
	{
		if(handle==Param->DesHandle)
		{
			KdPrint(("修改成功!\n"));
			Entry->GrantedAccess=Param->AccessMask;
			return true;
		}
	}
	return false;
}

NTSTATUS ModifyRight(FILE_RIGHT_MODIFY* Data)
{
	NTSTATUS  Status=STATUS_UNSUCCESSFUL;
	KdPrint(("输入句柄：Des %08x---Sour %08x",Data->DesHandle,Data->SourceHandle));
	ENUMHANDLE_PARAMETER * Param=(ENUMHANDLE_PARAMETER *)ExAllocatePool(NonPagedPool,sizeof(ENUMHANDLE_PARAMETER));
	if(Param>0)
	{
		Param->AccessMask=0;
		Param->DesHandle=Data->DesHandle;
		Param->SorHadnle=Data->SourceHandle;
		KdPrint(("%08x\n",PsGetCurrentProcess()));	
		PVOID TableAddr=(PVOID)GetHandleTableFromProcessXp(PsGetCurrentProcess());
		if(ExEnumHandleTable(TableAddr,
			(EX_ENUMERATE_HANDLE_ROUTINE)EnumHandleCallBack,Param,NULL))
		{
			KdPrint(("Enum Success!\n"));
			if(ExEnumHandleTable((PVOID)GetHandleTableFromProcessXp(PsGetCurrentProcess()),
				(EX_ENUMERATE_HANDLE_ROUTINE)EnumHandleCallBack,Param,NULL))
			{
				Status=STATUS_SUCCESS;
			}
		}
		ExFreePool(Param);
	}
	return Status;
}

NTSTATUS DisModifyRight(PDEVICE_OBJECT pDevice,PIRP pIrp)
{
	PIO_STACK_LOCATION  Stack=IoGetCurrentIrpStackLocation(pIrp);
	pIrp->IoStatus.Information=0;
	NTSTATUS Status;
	if(Stack->Parameters.DeviceIoControl.InputBufferLength<4)
	{
		Status=STATUS_INFO_LENGTH_MISMATCH;
	}
	else
	{
		Status=ModifyRight((FILE_RIGHT_MODIFY*)pIrp->AssociatedIrp.SystemBuffer);
	}
	pIrp->IoStatus.Status=Status;
	IofCompleteRequest(pIrp,IO_NO_INCREMENT);
	return Status;
}
```

移植时，只需要注意一个地方`GetHandleTableFromProcessXp` 这个宏我是这样定义的，这个只是在Xp下使用

`#define GetHandleTableFromProcessXp(a) (*(PULONG)((char*)a+0xc4))`

当然了，上面这小段代码可以用来更改任何句柄的权限。只需要在自己的驱动力加进去就好了，连分发函数都写好了^-^.

好了，现在任务完成一大半了，我对注册表函数的操作封装为一个类。首相使用传统注册表操作函数读取`SYSTEM\CurrentControlSet\Control\hivelist`下面的数据，将键名作为我这个注册表的根。

![](https://bbs.kanxue.com/upload/attach/201006/243689_bus8f84rqh88efl.jpg)

以后所有的操作都是基于键名的完整路径来说的。

比如你要操作`HKEY_LOCAL_MACHINE\SOFTWARE\Adobe`这个路径，在我这个类里面，你应该传进来的完整路径是：`\REGISTRY\MACHINE\SOFTWARE\Adobe`,注意这里的`\REGISTRY\MACHINE\SOFTWARE`就是`SYSTEM\CurrentControlSet\Control\hivelist`里面的`SOFTWARE`对应的键名（上图中划线部分），我说了，这个会作为根键，之后各个函数介绍如下，封装的函数如下：

1. 枚举子键

`REGSTATUS HiveAnysBase::EnumSubKey(char *FullPath, PVOID Buffer, ULONG BufferSize);`

`FullPath` --- eg: `\REGISTRY\MACHINE\SOFTWARE\Adobe`

`Buffer` – 返回数据的缓冲区，

`BufferSize` — 缓冲区大小

返回值 `REG_NOT_FIND` 给的路径没找到

`REG_BUFFER_TOO_SMALL` 缓冲区太小

`REG_SUCCESS` 查询成功

`Buffer`里面的数据结构`SUB_KEY_INFO`，`Count`指明了`PSUBKEY_ENTRY`的数目

```
typedef struct tagSUBKEY_BUFFER
{
        char Name[MAX_PATH];
}SUB_KEY_ENTRY,*PSUBKEY_ENTRY;

typedef struct tag_SUBKEY_INFO/*EnumSubKey返回的数据结构*/
{
        ULONG Count;
        SUB_KEY_ENTRY Entrys[1];
}SUB_KEY_INFO,*PSUB_KEY_INFO;
```

Eg:

```
PSUB_KEY_INFO pInfo =malloc(0x1000);
If(EnumSubKey("\\REGISTRY\\MACHINE\\SOFTWARE\\Adobe",pInfo,0x1000)==REG_SUCCESS)
{
    For(ULONG i=0;i<pInfo->Count'i++)
    {
        Ptinf("%s\n",pInfo->Entrys[i].Name);
    }
}
```

2. 枚举键值

`REGSTATUS HiveAnysBase::EnumKeyValue(char *FullPath, PVOID Buffer, ULONG BufferSize);`

参数同上面是一样的，返回值和1一样。

缓冲区数据结构如下

```
typedef struct tagKeyValue
{
        ULONG Type;// reg type
        ULONG DataLen; //if REG_DWORD is value key else length of DataBuffer
        char  Name[MAX_PATH];//value name
        unsigned char * DataBuffer;//key value data
}KEY_VALUE_ENTRY,*PKEY_VALUE_ENTRY;

typedef struct tag_KEY_VALUE_INFO/*EnumValue返回的数据结构*/
{
        ULONG Count;Entrys数据的大小
        KEY_VALUE_ENTRY Entrys[1];
}KEY_VALUE_INFO,*PKEY_VALUE_INFO;
```

特别请注意，如果数据类型是`REG_DWORD`，则其值放在`DataLen`域传回，否则数据在`DataBuffer`域，在使用这个域之后需要你自己释放这个`DataBuffer`占用的内存，请使用

```
void FreeBuffer(void *p)
{
        VirtualFree(p,0,MEM_RELEASE);
}
```

函数来释放。

3. 修改键值名字

`REGSTATUS HiveAnysBase::SetKeyValueName(char *FullPath, char *OldName, char *NewName);`

`FuPath`参数同上，`OldName`是现在键值的名字，`NewName`就是要修改成的新名字

4. 修改键值

`REGSTATUS HiveAnysBase::SetKeyValueData(char *FullPath, char *Name, BYTE *Data, ULONG DataSize);`

`FullPath`参数同上，

`Name`是键值的名字，

`Data`是数据缓冲区，

`DataSize` 缓冲区大小

5. 删除子键

`REGSTATUS HiveAnysBase::DeleteSubKey(char *FullPath);`

子键的完整路径，比如要删除`HKEY_LOCAL_MACHINE\SOFTWARE\Adobe\Photoshop`子键就传入`\REGISTRY\MACHINE\SOFTWARE\Adobe\Photoshop`就好了

6. 删除键值

`REGSTATUS HiveAnysBase::DeleteValue(char *FullPath, char *Name);`

`FullPath` — 子键路径

`Name` --- 键值名字

7. 增加子键

`REGSTATUS HiveAnysBase::AddSubKey(char *FullPath, char *Name);`

`FullPath` — 要增加子键的路径

`Name` — 子键名字

Eg：在`HKEY_LOCAL_MACHINE\SOFTWARE\Adobe\Photoshop`增加一个`test`的子键

调用为

`AddSubKey("\\REGISTRY\\MACHINE\\SOFTWARE\\Adobe\\Photoshop", "test");`

8. 增加键值

`REGSTATUS HiveAnysBase::AddValue(char *FullPath, char *Name, BYTE *Data, ULONG DataSize, ULONG Type);`

`FullPath` – 子键路径

`Name` — 键值名字

`Data` — 键值数据

`DataSize` — 数据长度

`Type` — 键值类型

例：在`HKEY_LOCAL_MACHINE\SOFTWARE\Adobe\Photoshop`下增加一个键值，名字为`Lujian` 的`DWORD` 值为`1`的数据

`ULONG Data=1;`

`AddValue("\\REGISTRY\\MACHINE\\SOFTWARE\\Adobe\\Photoshop", "lujian", &Data, sizeof(DWORD), REG_DWORD);`

运用上面这个类，就算是你对注册表文件格式一无所知，也可以实现离线解析注册表了，我在这个类的基础上，又派生了一个类，`CRegEditEx`，这类在上面那个类的基础之上特别提供了对`CTreeCtrl`控件支持，可参看源代码。

效果图：

![](https://bbs.kanxue.com/upload/attach/201006/243689_4ilzxx6u4h0w897.jpg)

## 移植指南

1. 首先在驱动里面加入上面修改句柄权限的代码，修改`RegSuport.cpp`里面的

```
BOOL GetRightToAccessFile(HANDLE hSor,HANDLE hDes)
{
        /*
        hSor 源句柄
        hDes 目标句柄，
        目标句柄会获得和源句柄相同的权限
        */
        ASSERT(hSor!=0);
        ASSERT(hDes!=0);
        FILE_RIGHT_MODIFY Data={0};
        Data.SourceHandle=hSor;
        Data.DesHandle=hDes;
        return theApp.m_hDriver.ControDevice(IOCTL_MODIFY_FILE_RIGHT,&Data,sizeof(FILE_RIGHT_MODIFY),0,0);
}
```

将最后这行换成给你的驱动发送控制码来修改权限的实现代码

2. 我这个代码是在大工程下的一小部分代码 ，代码使用`UNICODE`编码，如果要在`Ascii`下使用，修改有关字符处理的部分。

3. 两个内存处理函数：

```
void * AllocateBuffer(DWORD Size)
{
        void *p=VirtualAlloc(NULL,Size,MEM_COMMIT,PAGE_EXECUTE_READWRITE);
        if(p!=NULL)
                RtlZeroMemory(p,Size);
        return p;
}

void FreeBuffer(void *p)
{
        VirtualFree(p,0,MEM_RELEASE);
}
```

4. 在Windows7下和vista下，驱动里面修改获取程序句柄表的那个宏

5. 没有解决的问题，中文键名乱码（中文是`Unicde`码，但是英文是`char`，所以没找到好的处理）

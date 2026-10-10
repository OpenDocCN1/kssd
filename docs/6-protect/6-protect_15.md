

# 软件注册之获得硬件码各种方法

> 原文：[`www.pediy.com/kssd/pediy10/78504.html`](https://www.pediy.com/kssd/pediy10/78504.html)

在软件的保护注册中，特别是实现一机一码中，获得硬件编码是一个很重要的工作  
  1）MAC 地址  
   
   
           //对每块网卡，通过网卡编号，获得 MAC 地址  
           //网卡编号一般从 0 开始，但是 2000 中并不一定是连续分配  
           for (int i=0 ;i<lenum.length;i++)  
{  
//发送一个 NCBRESET 命令，进行初始化  
memset(&ncb,0,sizeof(ncb));  
ncb.ncb\_command = NCBRESET ;  
ncb.ncb\_lana\_num = lenum.lana\[i\];//MAC NUM  
uRetCode = Netbios(&ncb);  
  
//发送 NCBASTAT 命令获得网卡信息  
memset(&ncb,0,sizeof(ncb));  
ncb.ncb\_command = NCBASTAT;  
ncb.ncb\_lana\_num = lenum.lana\[i\];  
strcpy((char \*)ncb.ncb\_callname,"\* ");  
ncb.ncb\_buffer = (unsigned char \*)&Adapter;  
ncb.ncb\_length = sizeof(Adapter);  
uRetCode = Netbios(&ncb)\[color=#008000\];  
  2）硬盘号  
     
  逻辑硬盘序列号  
  GetVolumeInformation(“c:\\\\”,VolumeNameBuffer,256,&VolumeSerialNumber,&MaximumComponentLength,&FileSystemFlags,FileSystemNameBuffer,256);  
     
  但是这个是一个后天信息，而且有可能作弊  
           然后我们想从硬盘中直接提取信息，但是这个在 windows 下是很不方便的，因为 windows 不允许直接 I/O 操作，同时 windows 也不提供相应的 api 给我们，怎么办呢？  
  

   
  
static unsigned int WaitHardDiskIde()  
{  
    BYTE xx;  
  
Waiting:  
    \_asm  
    {  
        mov dx,0x1f7  
        in al,dx  
        cmp al,0x80  
        jb Endwaiting  
        jmp Waiting  
  
    }  
EndWaiting:  
        \_asm  
        {  
            mov xx,al  
        }  
  
        return (xx);  
}  
  
void \_\_declspec(naked)InterruptProcess(void)  
{  
    int xx;  
    int i;  
    WORD temp;  
  
    //store  
    \_asm  
    {  
        push eax  
        push ebx  
        push ecx  
        push edx  
        push esi  
    }  
    WaitHardDiskIde();//等待硬盘空闲  
    \_asm  
    {  
        mov dx,0x1f6  
            mov al,0xa0  
            out dx,al  
    }  
    xx = WaitHardDiskIde();  
    if ((xx&0x58)!=0x58)  
    {  
        goto EndIntrupt;  
    }  
    for (i=0;i<256;i++)  
    {  
        \_asm  
        {  
            mov dx,0x1f0  
                in ax,dx  
                mov temp,ax  
        }  
        serial\[i\] = temp ;  
    }  
EndIntrupt:  
    //恢复寄存器  
    \_asm  
    {  
        pop esi  
            pop edx  
            pop ecx  
            pop ebx  
            pop eax  
            iretd  
    }  
  
}  
       
  3）Cpu 编号  
  这个是破解一个软件的时候发现的，结果真的是有实际用途的  
  \_asm  
{  
mov eax,1  
cupid  
mov ps,eax  
mov mi,ebx  
mov ff,edx  
}
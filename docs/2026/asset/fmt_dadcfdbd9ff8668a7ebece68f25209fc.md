
```c
//本程序研究动态TLS，动态TLS是通过调用一组API实现，在生成的PE文件中没有.tls段。

//可执行文件运行后，没新建一个线程，都会为这个线程创建一个数组TLS Slot，用于存放动态TLS数据。该数组就是由_TEB的TlsSlots指出。

//TlsSetValue和TlsGetValue其实也就是访问的这个数组，当维数超过64时，由于TlsSlots不够，所以用到了TlsExpansionSlots

#include <stdio.h> 

#include <windows.h> 

#include <WinNT.h>



//通过windgb查看_TEB得到的我的系统(winXP+SP3)中的_TEB的实现

struct STEB

{

    NT_TIB NtTib;

    PVOID EnvironmentPointer;

    //中间若干数据，与此处研究无关，故不展开，只标记偏移。下面的Reserved2,Reserved3也是同理

    BYTE Reserved1[12]; 

    PVOID ThreadLocalStoragePointer;    //指向存放静态TLS数据的地址的指针的地址

    BYTE Reserved2[3552];

    PVOID TlsSlots[64];                    //指向存放动态TLS数据的TLS Slot数组

    BYTE Reserved3[132];

    PVOID TlsExpansionSlots;             //当索引大于63时，TlsSlots数组存不下了，就会新分配内存来存放，并且将指针记录在这里

    //后面还有若干数据，与此处研究无关，故省略

};



//设定启动线程数

#define THREADCOUNT 5

int baseorder=0;

//TLS Slot索引

DWORD dwTlsIndex1,dwTlsIndex2; 



//为了查看内存数据时正常，采用临界区

CRITICAL_SECTION gDisplayTLS_CS;



//内存中查看动态TLS数据

VOID TlsMemFunc(VOID) 

{ 

    EnterCriticalSection( &gDisplayTLS_CS );



    printf("=============================原始动态TLS数据=============================\n"); 

    printf("dwThreadID=%d；threadinfo=%s\n",(DWORD)TlsGetValue(dwTlsIndex1),(char*)TlsGetValue(dwTlsIndex2)); 

    printf("=========================================================================\n"); 



    PNT_TIB pTIB;

    STEB * pTEB;

    __asm

    {

        mov     EAX, FS:[18h]

        mov     [pTIB], EAX

    } 

    pTEB = (STEB *)pTIB;

    printf("=======================通过内存偏移查看动态TLS数据=======================\n"); 



    DWORD dwTlsIndex1Value,dwTlsIndex2Value;

    if (dwTlsIndex1 < 64)

    {

        dwTlsIndex1Value = (DWORD)pTEB->TlsSlots[dwTlsIndex1];

    }

    else

    {

        DWORD * pES = (DWORD*)(pTEB->TlsExpansionSlots);

        pES += dwTlsIndex1-64;

        dwTlsIndex1Value = *pES;

    }

    if (dwTlsIndex2 < 64)

    {

        dwTlsIndex2Value = (DWORD)pTEB->TlsSlots[dwTlsIndex2];

    }

    else

    {

        DWORD * pES = (DWORD*)(pTEB->TlsExpansionSlots);

        pES += dwTlsIndex2-64;

        dwTlsIndex2Value = (*pES);

    }

    

    printf("dwThreadID=%d；threadinfo=%s\n",dwTlsIndex1Value,(char*)dwTlsIndex2Value);     

    printf("=========================================================================\n\n"); 



    LeaveCriticalSection( &gDisplayTLS_CS );

} 





DWORD WINAPI ThreadFunc(VOID) 

{ 

    TlsSetValue(dwTlsIndex1,(LPVOID)GetCurrentThreadId());

    char info[100]={0};

    sprintf(info,"My threadID is %d,My runorder is %d!",GetCurrentThreadId(),++baseorder);

    TlsSetValue(dwTlsIndex2,info);



    TlsMemFunc(); 

    return 0; 

} 



DWORD main(VOID) 

{ 

    DWORD IDThread; 

    HANDLE hThread[THREADCOUNT]; 

    

    InitializeCriticalSection( &gDisplayTLS_CS );



    dwTlsIndex1 = TlsAlloc();

    //故意使索引超过64，而测试扩展slot

    while(dwTlsIndex1 < 63)

        dwTlsIndex1 = TlsAlloc();



    dwTlsIndex2 = TlsAlloc();

    if (dwTlsIndex1 == TLS_OUT_OF_INDEXES || dwTlsIndex2 == TLS_OUT_OF_INDEXES) 

    {

        fprintf(stderr, "TlsAlloc failed\n"); 

        return -1;

    }

    printf("dwTlsIndex1=%d：用于存放线程ID；dwTlsIndex2=%d：用于存放指向线程信息的指针。\n", dwTlsIndex1, dwTlsIndex2); 

    

    // Create multiple threads. 

    for (int i = 0; i < THREADCOUNT; i++) 

    { 

        hThread[i] = CreateThread(NULL, // default security attributes 

            0,                           // use default stack size 

            (LPTHREAD_START_ROUTINE) ThreadFunc, // thread function 

            NULL,                    // no thread function argument 

            0,                       // use default creation flags 

            &IDThread);              // returns thread identifier 



        // Check the return value for success. 

        if (hThread[i] == NULL) 

            fprintf(stderr, "CreateThread %d error\n", i); 

    } 



    for (int i = 0; i < THREADCOUNT; i++) 

        WaitForSingleObject(hThread[i], INFINITE); 



    TlsFree(dwTlsIndex1);

    TlsFree(dwTlsIndex2);



    DeleteCriticalSection( &gDisplayTLS_CS );

    return 0; 

}
```

程序运行结果如下：

![](https://bbs.kanxue.com/upload/attach/201207/246127_acek5bxpfgiobcw.jpg)

同样可以看出，通过`TlsGetValue`访问动态TLS变量和访问`TEB`中相关内存得到的是一样的，由此我们就更加深入的理解了上一篇中讲的动态TLS的原理，已经底层的Tls slots数组的实现情况。抽丝剥茧，一目了然！

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [dynamicTLS.jpg](javascript:void(0);)

  （201.67kb，20次下载）

- [staticTLS.jpg](javascript:void(0);)

  （198.32kb，21次下载）

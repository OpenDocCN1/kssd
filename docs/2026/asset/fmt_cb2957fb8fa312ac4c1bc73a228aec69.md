
```c
                        if (g_nValidPages<0x10h)

                                g_nValidPages++;

                }

                InterlockedExchange(&g_nLock,0);// 解锁

                return 1;

                // 6.找到的nPageBase移到头

                //   但前面找到的结果也可能被其它线程改变位置甚至推出数组,但无论如何这个nPageBase合法可不再验证

        Finded:

                if (nFind <= 0)// 相当于if(nFind == 0)

                        return 1;// nPageBase已经在头,可直接返回

                if(InterlockedExchange(&g_nLock,1)!=0)// 写线程锁

                        return 1;// 其它线程已经进入,则不能再进入

                if(g_rgValidPages[nFind] != nPageBase)

                {// 再次对找到的pos进行比较,因为其它线程可能又修改了这个元素的值

                        nFind = g_nValidPages-1;

                        if (nFind>=0)

                        {

                                while(nFind>=0)

                                {// LOOP3

                                        if (g_rgValidPages[nFind]==nPageBase)

                                                break;

                                        nFind--;

                                }

                                if (nFind>=0)

                                        goto End1;

                        }

                        // 没找到,新增加

                        if(g_nValidPages < 0x10)

                                g_nValidPages++;

                        nFind = g_nValidPages-1;

                        goto end2;

                }

                else

                        goto end2;

        end1:

                if (nFind != 0)

                {

        end2:

                        if (nFind >= 0)

                        {

                                for(int j = 0;j<=nFind;j++)

                                {// LOOP6:g_rgValidPages中找到的nPageBase移到头或新nPageBase插入头,其余每个元素向后推

                                        int temp=g_rgValidPages[j];

                                        g_rgValidPages[j]=nPageBase;

                                        nPageBase=temp;

                                }

                        }

                }

                InterlockedExchange(&g_nLock,0);// 解线程锁

                return 1;

        }

        void _global_unwind2(_EH3_EXCEPTION_REGISTRATION* pEh3Exce)

        {// 对调用RtlUnwind的封装

                RtlUnwind(pEh3Exce,offset exit,0,0);

        exit:

                return;

        }

        int _UnwindHandler(EXCEPTION_RECORD* pExceRec,_EH3_EXCEPTION_REGISTRATION* pEh3Exce,

                                           void*,_EH3_EXCEPTION_REGISTRATION** ppEh3Exce)

        {

                if(pExceRec->ExceptionFlags && 6 == 0)

                        return 1;// ExceptionContinueSearch

                *ppEh3Exce = pEh3Exce;

                return 3;// ExceptionCollidedUnwind

        }

        // NLG == "non-local-goto"

        _NLGDestination g_NLGDestination;// 此全局变量我至始至终未见到任何其它EXE处或者DLL使用,通知是何意义暂无知!!!!!!!

        void _NLG_Notify1(int x)// --------------------------CallSettingFrame调用

        {// g_NLGDestination全局分配变量:0x19930520

                g_NLGDestination.dwInCode = EBP+8;// Try函数中第一个参数???

                g_NLGDestination.HandlerFunc = EAX;// 调用前通过EAX传入

                g_NLGDestination.TryEBP = EBP;// Try中的EBP???

        }

        void _NLG_Notify(int x)// --------------------------_except_handler3,_local_unwind2调用

        {// 传入参数未用

                // g_NLGDestination.unKnown全局分配变量:0x19930520

                g_NLGDestination.dwInCode = EBP+8;// Try函数中第一个参数,如:FunB(i,j)!i,crtMain!ebp+8

                g_NLGDestination.HandlerFunc = EAX;// 调用前通过EAX传入:pScopeTable.HandlerFunc

                g_NLGDestination.TryEBP = EBP;// Try中的EBP

        }

        // nToLevel:展开到此为止(不含nToLevel),举例:在goto中nToLevel由目标所在try决定,因为系统规定跳入TRY是不合法的,因此

        // goto不能跳入其它TRY(嵌套的上层TRY是合法的,很明显平级层是不合法的)层,这时编译是通不过的.

        void _local_unwind2(_EH3_EXCEPTION_REGISTRATION* pEh3Exce,int nToLevel)

        {// 用ESP,未用EBP,之前的EBP传入内调的_NLG_Notify,__try中的goto将用此函数展开得到其handler

                _EH3_EXCEPTION_REGISTRATION  eh3Unwind;

                eh3Unwind.TryLevel = (int)pEh3Exce;

                eh3Unwind.pScopeTable = (_SCOPETABLE_ENTRY*)-2;

                eh3Unwind.ExceptionHandler = (EXCE_HANDLER)_UnwindHandler;

                eh3Unwind.pPrev = (_EH3_EXCEPTION_REGISTRATION*)FS[0];

                FS[0]=ESP;

                int nLevel;//

                while(1)

                {

                        _SCOPETABLE_ENTRY *pScope=pEh3Exce->pScopeTable;

                        ESI = pEh3Exce->TryLevel;

                        if (ESI == -1)

                                break;

                        if (ESI == nToLevel)

                                break;

                        nLevel=pScope[ESI].ParentLevel;

                        pEh3Exce->TryLevel=nLevel;

                        if (pScope[ESI].FilterFunc != NULL) // 有FilterFunc则是exception handler

                                continue;

                        EAX=pScope[ESI].HandlerFunc;                // 无FilterFunc则是termination handler

                        _NLG_Notify(0x101);                        // 无返回值

                        CALL(EAX);                                        // 这就是展开所求得的Handler,可以是termination handler,exception Handler

                }

                FS[0]=(DWORD)eh3Unwind.pPrev;

        }

        // _except_handler3执行两次原因:

        // 举例:EH3_List如 FS[0]->E1->E2->E3->E4,在E4中发生异常,依次搜索E1,E2,E3,最后在E4中找到能处理此异常的filter handler,这样E4,E3,E2,E1的_except_handler3均执行了一次,这是第一次.

        //     第二次:在执行E4的exception handler前要先调用全局展开(_global_unwind2)以运行E1,E2,E3的termination handler

        // 在全局展开(_global_unwind2)中将为EXCEPTION_RECORD.ExceptionFlags增加标志_EH_UNWINDING(2),再依次调用它们的

        // _except_handler3,这就是E1,E2,E3的_except_handler3的第二次调用,但E4只有一次.

        int _except_handler3(EXCEPTION_RECORD* pExceRec,_EH3_EXCEPTION_REGISTRATION* pEh3Exce,

                                                 CONTEXT* pContextRecord/*,void * DispatcherContext*/)

        {

                if ((pExceRec->ExceptionFlags & 6) == 0)

                {// 无_EH_UNWINDING and _EH_EXIT_UNWIND

                        EXCEPTION_POINTERS ePointer;

                        ePointer.ExceptionRecord = pExceRec;

                        ePointer.ContextRecord = pContextRecord;

                        // pEh3Exce=&EhRecord

                        // lea     EAX, [EBP+var_ExcePointers]

                        // mov     [ebx-4], EAX ;EBX=pEh3Exce

                        *((DWORD*)pEh3Exce-1)=(DWORD)&ePointer;        // 给_XcptFilter的参数pExceptPointers赋值

                        if (ValidateEH3RN(pEh3Exce))

                        {

                                _SCOPETABLE_ENTRY* pScopeTable=pEh3Exce->pScopeTable;

                                int i = pEh3Exce->TryLevel;

                                while(1)

                                {

                                        if (i==-1)

                                                return 1;// handler拒绝处理这个异常

                                        EAX=pScopeTable[i].FilterFunc;

                                        if(EAX!=NULL)

                                        {// 属于exception handler

                                                EBP=(DWORD)pEh3Exce+0x10;// 恢复原TRY基址pEh3Exce=&crtMain!EhRecord,sizeof(EhRecord)=0x10

                                                CALL(EAX);// 没有参数,ePointer已传过去

                                                if(EAX != 0)

                                                {

                                                        if ((int)EAX < 0)

                                                                return 0;                                // 如果在filter中返回值<0,此返回后会引起代码为
```

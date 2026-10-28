
```c
/*
函数说明:
获取进程IAT列表

输入参数:
INode **iNode    :  IAT表项结构体二级指针
PNode *pNode     :  进程结构体指针
unsigned int pid :  进程PID

输出参数:
*/
int GetIAT(INode **iNode, PNode *pNode, unsigned int pid)
{
    unsigned char buff[1025] = {0}; //用于临时保存读取的buff
    unsigned char nameAddrBuff[513] = {0}; //IAT表项函数名地址列表
    unsigned char addrBuff[513] = {0}; //IAT表项函数地址列表
    char dllName[NAMESIZE] = {0}; //IAT表项所属dll名
    unsigned char nameBuff[NAMESIZE] = {0}; //IAT表项函数名

    PNode *bkPNode = pNode; //初始化进程结构体链表操作指针
    INode *bkINode = NULL; //定义IAT表项结构体操作指针
    INode *newINode = NULL; //定义新的IAT表项结构体指针

    HANDLE handle = NULL; //初始化进程句柄

    LPCVOID addr = 0; //地址指针
    int offset = 0; //保存PE结构偏移
    LPDWORD readBuffCount = 0; //保存ReadProcessMemory实际读取的字节数
    int flag = 0; //函数调用标记
    int error = 0; //函数调用出错代码
    int order = 0; //函数在列表中的序号
    int IATaddr = 0; //IAT表的地址

    int descriptorBaseAddr = 0; //IMAGE_IMPORT_DESCRIPTOR结构体首地址
    int dllNameAddr = 0; //dll名地址
    int funcNameAddr = 0; //函数名列表地址
    int funcAddrAddr = 0; //函数地址列表地址
    int funcName = 0; //函数名地址

    int i = 0; //循环计数
    int j = 0; //循环变量

    InitINode(iNode); //初始化IAT表项结构体链表头指针
    bkINode = *iNode; //初始化IAT表项结构体链表操作指针

    if (NULL == bkPNode) //如果进程链表为空,则出错退出
    {
        return -1;
    }

    for (;;) //循环遍历进程结构体中与所给进程PID相符进程结构体
    {
        if (pid == bkPNode->pe32.th32ProcessID)
        {
            break;
        }
        else
        {
            if (NULL == bkPNode->next)
            {
                return -1;
            }
            else
            {
                bkPNode = bkPNode->next;
            }
        }
    }

    if (EnableDebugPriv(SE_DEBUG_NAME)) //获取进程调试权限
    {
        printf("Add Privilege error\n");

        return -1;
    }

    handle=OpenProcess(PROCESS_ALL_ACCESS,1,pid); //获取进程句柄
    if (handle == INVALID_HANDLE_VALUE)
    {
        return -1;
    }

    addr = bkPNode->me32.modBaseAddr; //获取进程加载基址
    flag = ReadProcessMemory(handle, addr, buff, 512, readBuffCount); // 读取进程前512字节信息
    offset = buff[60] + buff[61] * 256 + buff[62] * 256 * 256 + buff[63] * 256 * 256 * 256;
    offset = offset + 0x18;
    offset = offset + 0x60;
    offset = offset + 0x8;
    IATaddr = buff[offset] + buff[offset+1] *256 + buff[offset+2] * 256* 256 + buff[offset+3] * 256 * 256 * 256;
    addr = bkPNode->me32.modBaseAddr + IATaddr; //根据PE文件结构获取进程IAT表地址

    flag = ReadProcessMemory(handle, addr, buff, 1024, readBuffCount); //读取进程IAT表所在内存的1024字节信息

    descriptorBaseAddr = 0;
    for (order = 0;;)
    {
        //根据IMAGE_INPORT_DESCRIPTOR结构,获取相应dll名地址,函数名地址列表首地址,函数地址列表首地址
        funcNameAddr = buff[descriptorBaseAddr+0] + buff[descriptorBaseAddr+1] *256 + buff[descriptorBaseAddr+2] * 256* 256 + buff[descriptorBaseAddr+3] * 256 * 256 * 256;
        dllNameAddr = buff[descriptorBaseAddr+12] + buff[descriptorBaseAddr+13] *256 + buff[descriptorBaseAddr+14] * 256* 256 + buff[descriptorBaseAddr+15] * 256 * 256 * 256;
        funcAddrAddr = buff[descriptorBaseAddr+16] + buff[descriptorBaseAddr+17] *256 + buff[descriptorBaseAddr+18] * 256* 256 + buff[descriptorBaseAddr+19] * 256 * 256 * 256;

        //读取函数名地址列表
        flag = ReadProcessMemory(handle, bkPNode->me32.modBaseAddr+funcNameAddr, nameAddrBuff, 512, readBuffCount);
        if (0 == flag)
        {
            error = GetLastError();
            printf("Read funcNameAddr failed!!\nError : %d\n",error);
            return -1;
        }

        //读取函数地址列表
        flag = ReadProcessMemory(handle, bkPNode->me32.modBaseAddr+funcAddrAddr, addrBuff, 512, readBuffCount);
        if (0 == flag)
        {
            error = GetLastError();
            printf("Read funcAddrAddr failed!!\nError : %d\n",error);
            return -1;
        }

        //读取dll文件名
        flag = ReadProcessMemory(handle, bkPNode->me32.modBaseAddr+dllNameAddr, nameBuff, NAMESIZE-1, readBuffCount);
        if (0 == flag)
        {
            error = GetLastError();
            printf("Read funcName failed!!\nError : %d\n",error);
            return -1;
        }
        for (j = 0; j < NAMESIZE-1; j++)
        {
            if (0 == nameBuff[j])
            {
                break;
            }
            else
            {
                dllName[j] = nameBuff[j];
            }
        }
        dllName[j] = 0;

        for (i = 0;;) //循环获取IAT表项
        {
            bkINode->order = order; //函数序号
            order++;

            strcpy(bkINode->dllname,dllName); //函数所属dll名

            bkINode->addrOfAddr = funcAddrAddr + i; //函数地址所在内存地址

            //获取函数名所在内存首地址
            funcName = nameAddrBuff[i] + nameAddrBuff[i+1]*256 + nameAddrBuff[i+2]*256*256 + nameAddrBuff[i+3]*256*256*256;
            if (0x80000000 == (0x80000000&funcName)) //如果函数名所在地址最高位为1,则说明是以序号导入的
            {
                sprintf(bkINode->name,"Oridinal : %#0 8X",0x7fffffff&funcName);
                bkINode->address = funcName; //这种导入方式我不知道地址是多少
            }
            else
            {
                //读取函数名
                flag = ReadProcessMemory(handle, bkPNode->me32.modBaseAddr+funcName, nameBuff, NAMESIZE-1, readBuffCount);
                if (0 == flag)
                {
                    error = GetLastError();
                    printf("Read funcName failed!!\nError : %d\n",error);
                    return -1;
                }

                //获得函数名
                for (j = 0; j < NAMESIZE-1; j++)
                {
                    if (0 == nameBuff[j+2])
                    {
                        break;
                    }
                    else
                    {
                        bkINode->name[j] = nameBuff[j+2];
                    }
                }
                bkINode->name[j] = 0;

                //获取函数在内存中的地址
                bkINode->address = addrBuff[i] + addrBuff[i+1]*256 + addrBuff[i+2]*256*256 + addrBuff[i+3]*256*256*256;
            }

            i = i + 4; //如果下个函数名地址为0,则说明这个dll的导入函数结束了
            if (0 == nameAddrBuff[i] && 0 == nameAddrBuff[i+1] && 0 == nameAddrBuff[i+2] && 0 == nameAddrBuff[i+3])
            {
                break;
            }
            if (512 == i) //如果函数名地址列表超过512字节,则重新获取函数名地址列表和函数地址列表
            {
                i = 0;
                funcNameAddr += 512; //指针向前移51字节
                funcAddrAddr += 512;
                flag = ReadProcessMemory(handle, bkPNode->me32.modBaseAddr+funcNameAddr, nameAddrBuff, 512, readBuffCount);
                if (0 == flag)
                {
                    error = GetLastError();
                    printf("Read funcNameAddr failed!!\nError : %d\n",error);
                    return -1;
                }

                funcName = nameAddrBuff[0] + nameAddrBuff[1] *256 + nameAddrBuff[2] * 256* 256 + nameAddrBuff[3] * 256 * 256 * 256;
                flag = ReadProcessMemory(handle, bkPNode->me32.modBaseAddr+funcAddrAddr, addrBuff, 512, readBuffCount);
                if (0 == flag)
                {
                    error = GetLastError();
                    printf("Read funcAddrAddr failed!!\nError : %d\n",error);
                    return -1;
                }
            }
            InitINode(&newINode);
            bkINode->next = newINode;
            bkINode = newINode;
            newINode = NULL;
        }

        descriptorBaseAddr += 20; //如果下一个IMAGE_IMPORT_DESCRIPTOR结构体为空,则退出
        if (0 == buff[descriptorBaseAddr] && 0 == buff[descriptorBaseAddr+1] && 0 == buff[descriptorBaseAddr+2] && 0 == buff[descriptorBaseAddr+3])
        {
            break;
        }
        InitINode(&newINode);
        bkINode->next = newINode;
        bkINode = newINode;
        newINode = NULL;
    }

    CloseHandle(handle);
    return 0;
}

/*
函数说明:
hooking 某个IAT表中的函数

输入参数:
INode **iNode    :  IAT表项结构体二级指针
PNode *pNode     :  进程结构体指针
int order        :  函数序号
unsigned int pid :  进程PID

输出参数:
*/
int IATHook(INode *iNode, PNode *pNode, int order, unsigned int pid)
{
    char addr[5] = {0}; //保存四字节地址信息

    INode *bkINode = iNode; //初始化IAT表项结构体链表操作指针
    HANDLE hProcess; //进程句柄
    DWORD dwHasWrite; //实际读取的字节数
    LPVOID lpRemoteBuf; //新申请的内存空间指针
    int temp = 0; //临时变量

    //数据
    char data[] = "\x74\x65\x73\x74\x00\xCC\xCC\xCC"
                  "\xD7\xE9\xB3\xA4\x20\x3A\x20\xBA"
                  "\xCE\xC4\xDC\xB1\xF3\x20\x32\x30"
                  "\x31\x33\x33\x30\x32\x35\x33\x30"
                  "\x30\x32\x30\x0A\xD7\xE9\xD4\xB1"
                  "\x20\x3A\x20\xCD\xF5\x20\x20\xEC"
                  "\xB3\x20\x32\x30\x31\x33\x33\x30"
                  "\x32\x35\x33\x30\x30\x30\x35\x0A"
                  "\x20\x20\x20\x20\x20\x20\x20\xB5"
                  "\xCB\xB9\xE3\xF6\xCE\x20\x32\x30"
                  "\x31\x33\x33\x30\x32\x35\x33\x30"
                  "\x30\x31\x34\x0A\x20\x20\x20\x20"
                  "\x20\x20\x20\xB9\xA8\xD3\xF1\xB7"
                  "\xEF\x20\x32\x30\x31\x33\x33\x30"
                  "\x32\x35\x33\x30\x30\x32\x31\x00";

    //shellcode
    char shellcode[] =
        "\x9C\x50\x51\x52\x53\x55\x56\x57"
        "\x6A\x00\x68\x00\x10\x40\x00\x68"
        "\x00\x10\x40\x00\x6A\x00\xB8\xEA"
        "\x07\xD5\x77\xFF\xD0\x5F\x5E\x5D"
        "\x5B\x5A\x59\x58\x9D\xB8\xEA\x07"
        "\xD5\x7C\xFF\xE0";

    //循环遍历IAT表项结构体链表,寻找与所给函数序号相同的IAT表项结构体
    if (NULL == iNode)
    {
        return -1;
    }
    for (;;)
    {
        if (NULL == iNode->next)
        {
            if (iNode->order == order)
            {
                break;
            }
            else
            {
                return -1;
            }
        }
        else
        {
            if (iNode->order == order)
            {
                break;
            }
            else
            {
                iNode = iNode->next;
            }
        }
    }

    //循环遍历IAT表项结构体链表,寻找MessageBoxA的IAT表项结构体
    for (;;)
    {
        if (NULL == bkINode->next)
        {
            if (0 == strcmp(bkINode->name,"MessageBoxA"))
            {
                break;
            }
            else
            {
                return -1;
            }
        }
        else
        {
            if (0 == strcmp(bkINode->name,"MessageBoxA"))
            {
                break;
            }
            else
            {
                bkINode = bkINode->next;
            }
        }
    }

    //循环遍历进程结构体链表,寻找与所给函数所属进程PID相同的进程结构体
    if (NULL == pNode)
    {
        return -1;
    }
    for (;;)
    {
        if (pid == pNode->pe32.th32ProcessID)
        {
            break;
        }
        else
        {
            if (NULL == pNode->next)
            {
                return -1;
            }
            else
            {
                pNode = pNode->next;
            }
        }
    }

    if (EnableDebugPriv(SE_DEBUG_NAME)) //获取调试权限
    {
        fprintf(stderr,"Add Privilege error\n");

        return -1;
    }

    hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid); //获取进程句柄
    if(hProcess == NULL)
    {
        fprintf(stderr,"\n获取进程句柄错误%d",GetLastError());
        return -1;
    }

    //申请120字节的数据空间,并写入我们需要的数据
    lpRemoteBuf = VirtualAllocEx(hProcess, NULL, 120, MEM_COMMIT, PAGE_READWRITE);
    if(WriteProcessMemory(hProcess, lpRemoteBuf, data, 120, &dwHasWrite))
    {
        if(dwHasWrite != 120)
        {
            VirtualFreeEx(hProcess,lpRemoteBuf,120,MEM_COMMIT);
            CloseHandle(hProcess);
            return -1;
        }

    }else
    {
        printf("\n写入远程进程内存空间出错%d。",GetLastError());
        CloseHandle(hProcess);
        return -1;
    }

    temp = (int)lpRemoteBuf; //数据所在首地址
    addr[0] = temp&0xff;
    addr[1] = temp>>8&0xff;
    addr[2] = temp>>16&0xff;
    addr[3] = temp>>24&0xff;

    shellcode[11] = addr[0]; //"test" 的地址
    shellcode[12] = addr[1];
    shellcode[13] = addr[2];
    shellcode[14] = addr[3];

    shellcode[16] = addr[0]+8;//"所要显示的字符串首地址"
    shellcode[17] = addr[1];
    shellcode[18] = addr[2];
    shellcode[19] = addr[3];

    temp = (int)bkINode->address; //MessageBoxA的地址
    addr[0] = temp&0xff;
    addr[1] = temp>>8&0xff;
    addr[2] = temp>>16&0xff;
    addr[3] = temp>>24&0xff;
    shellcode[23] = addr[0];
    shellcode[24] = addr[1];
    shellcode[25] = addr[2];
    shellcode[26] = addr[3];

    temp = (int)iNode->address; //原函数的地址,用于jmp回原来的函数
    addr[0] = temp&0xff;
    addr[1] = temp>>8&0xff;
    addr[2] = temp>>16&0xff;
    addr[3] = temp>>24&0xff;
    shellcode[38] = addr[0];
    shellcode[39] = addr[1];
    shellcode[40] = addr[2];
    shellcode[41] = addr[3];

    //申请44字节的可读可写可执行的shellcode空间,并写入shellcode
    lpRemoteBuf = VirtualAllocEx(hProcess, NULL, 44, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
    if(WriteProcessMemory(hProcess, lpRemoteBuf, shellcode, 44, &dwHasWrite))
    {
        if(dwHasWrite != 44)
        {
            VirtualFreeEx(hProcess,lpRemoteBuf,44,MEM_COMMIT);
            CloseHandle(hProcess);
            return -1;
        }

    }else
    {
        printf("\n写入远程进程内存空间出错%d。",GetLastError());
        CloseHandle(hProcess);
        return -1;
    }

    temp = (int)lpRemoteBuf; //获取shellcode的首地址,并替换IAT表中相应的函数地址
    addr[0] = temp&0xff;
    addr[1] = temp>>8&0xff;
    addr[2] = temp>>16&0xff;
    addr[3] = temp>>24&0xff;
    if(WriteProcessMemory(hProcess, pNode->me32.modBaseAddr+iNode->addrOfAddr, addr, 4, &dwHasWrite))
    {
        return 0;
    }
    else
    {
        printf("\n写入远程进程内存空间出错%d。",GetLastError());
    }

    CloseHandle(hProcess);
    return -1;
}

/*
函数说明:
inline hooking 某个IAT表中的函数

输入参数:
INode **iNode    :  IAT表项结构体二级指针
PNode *pNode     :  进程结构体指针
int order        :  函数序号
unsigned int pid :  进程PID

输出参数:
*/
int InlineHook(INode *iNode, PNode *pNode, int order, unsigned int pid)
{
    char addr[5] = {0}; //用于保存4字节的地址
    char buff[6] = {0}; //用于保存jmp xxx指令和所要hook的函数起始五个字节

    INode *bkINode = iNode; //初始化IAT表项结构体链表操作指针
    HANDLE hProcess; //进程句柄
    DWORD dwHasWrite; //实际写入的字节数
    LPVOID lpRemoteBuf; //申请的内存首地址
    int temp = 0; //临时变量

    //数据
    char data[] = "\x74\x65\x73\x74\x00\xCC\xCC\xCC"
                  "\xD7\xE9\xB3\xA4\x20\x3A\x20\xBA"
                  "\xCE\xC4\xDC\xB1\xF3\x20\x32\x30"
                  "\x31\x33\x33\x30\x32\x35\x33\x30"
                  "\x30\x32\x30\x0A\xD7\xE9\xD4\xB1"
                  "\x20\x3A\x20\xCD\xF5\x20\x20\xEC"
                  "\xB3\x20\x32\x30\x31\x33\x33\x30"
                  "\x32\x35\x33\x30\x30\x30\x35\x0A"
                  "\x20\x20\x20\x20\x20\x20\x20\xB5"
                  "\xCB\xB9\xE3\xF6\xCE\x20\x32\x30"
                  "\x31\x33\x33\x30\x32\x35\x33\x30"
                  "\x30\x31\x34\x0A\x20\x20\x20\x20"
                  "\x20\x20\x20\xB9\xA8\xD3\xF1\xB7"
                  "\xEF\x20\x32\x30\x31\x33\x33\x30"
                  "\x32\x35\x33\x30\x30\x32\x31\x00";

    //shellcode
    char shellcode[] =
        "\x9C\x50\x51\x52\x53\x55\x56\x57"
        "\x6A\x00\x68\x00\x10\x40\x00\x68"
        "\x00\x10\x40\x00\x6A\x00\xB8\xEA"
        "\x07\xD5\x77\xFF\xD0\x5F\x5E\x5D"
        "\x5B\x5A\x59\x58\x9D\x8b\xff\x55\x8b\xec"
        //shellco中有所要hooking的函数前五个字节了
        "\xe9\x90\x90\x90\x90"; //所以后面jmp 回到的是函数的第六个字节

    if (NULL == iNode) //如果IAT表项链表为空,则退出
    {
        return -1;
    }
    for (;;)
    {
        if (NULL == iNode->next)
        {
            if (iNode->order == order)
            {
                break;
            }
            else
            {
                return -1;
            }
        }
        else
        {
            if (iNode->order == order)
            {
                break;
            }
            else
            {
                iNode = iNode->next;
            }
        }
    }

    //获取MessageBoxA的IAT表项结构体
    for (;;)
    {
        if (NULL == bkINode->next)
        {
            if (0 == strcmp(bkINode->name,"MessageBoxA"))
            {
                break;
            }
            else
            {
                return -1;
            }
        }
        else
        {
            if (0 == strcmp(bkINode->name,"MessageBoxA"))
            {
                break;
            }
            else
            {
                bkINode = bkINode->next;
            }
        }
    }

    //获取所要hook的函数所属进程结构体
    if (NULL == pNode)
    {
        return -1;
    }
    for (;;)
    {
        if (pid == pNode->pe32.th32ProcessID)
        {
            break;
        }
        else
        {
            if (NULL == pNode->next)
            {
                return -1;
            }
            else
            {
                pNode = pNode->next;
            }
        }
    }

    //获取调试权限
    if (EnableDebugPriv(SE_DEBUG_NAME))
    {
        fprintf(stderr,"Add Privilege error\n");

        return -1;
    }

    //获取进程句柄
    hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);
    if(hProcess == NULL)
    {
        fprintf(stderr,"\n获取进程句柄错误%d",GetLastError());
        return -1;
    }

    //读取所要hook的函数前五个字节
    if(ReadProcessMemory(hProcess, iNode->address, buff, 5, &dwHasWrite))
    {
        if(dwHasWrite != 5)
        {
            CloseHandle(hProcess);
            return -1;
        }

    }else
    {
        printf("\n读取远程进程内存空间出错%d。",GetLastError());
        CloseHandle(hProcess);
        return -1;
    }

    //如果函数前五个字节不是 mov edi,edi push ebp mov ebp,esp则退出inline hooking
    if (0 != strcmp(buff,"\x8b\xff\x55\x8b\xec"))
    {
        return -1;
    }

    //申请120字节的数据空间
    lpRemoteBuf = VirtualAllocEx(hProcess, NULL, 120, MEM_COMMIT, PAGE_READWRITE);
    if(WriteProcessMemory(hProcess, lpRemoteBuf, data, 120, &dwHasWrite))
    {
        if(dwHasWrite != 120)
        {
            VirtualFreeEx(hProcess,lpRemoteBuf,120,MEM_COMMIT);
            CloseHandle(hProcess);
            return -1;
        }

    }else
    {
        printf("\n写入远程进程内存空间出错%d。",GetLastError());
        CloseHandle(hProcess);
        return -1;
    }

    temp = (int)lpRemoteBuf; //获取数据在内存中的首地址
    addr[0] = temp&0xff;
    addr[1] = temp>>8&0xff;
    addr[2] = temp>>16&0xff;
    addr[3] = temp>>24&0xff;

    shellcode[11] = addr[0]; //"test"的首地址
    shellcode[12] = addr[1];
    shellcode[13] = addr[2];
    shellcode[14] = addr[3];

    shellcode[16] = addr[0]+8; //所要显示的字符串首地址
    shellcode[17] = addr[1];
    shellcode[18] = addr[2];
    shellcode[19] = addr[3];

    temp = (int)bkINode->address; //MessageBoxA的地址
    addr[0] = temp&0xff;
    addr[1] = temp>>8&0xff;
    addr[2] = temp>>16&0xff;
    addr[3] = temp>>24&0xff;
    shellcode[23] = addr[0];
    shellcode[24] = addr[1];
    shellcode[25] = addr[2];
    shellcode[26] = addr[3];

    //先写入42字节的shellcode
    lpRemoteBuf = VirtualAllocEx(hProcess, NULL, 42, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
    if(WriteProcessMemory(hProcess, lpRemoteBuf, shellcode, 42, &dwHasWrite))
    {
        if(dwHasWrite != 42)
        {
            VirtualFreeEx(hProcess,lpRemoteBuf,42,MEM_COMMIT);
            CloseHandle(hProcess);
            return -1;
        }

    }else
    {
        printf("\n写入远程进程内存空间出错%d。",GetLastError());
        CloseHandle(hProcess);
        return -1;
    }

    temp = (int)lpRemoteBuf; //获得shellcode的首地址
    temp = temp - iNode->address -5; //计算jmp到shellcode的偏移
    buff[0] = 0xe9;
    buff[1] = temp&0xff;
    buff[2] = temp>>8&0xff;
    buff[3] = temp>>16&0xff;
    buff[4] = temp>>24&0xff; //得到jmp xxx的二进制数据并写入函数的其实五个字节
    if(!WriteProcessMemory(hProcess, iNode->address, buff, 5, &dwHasWrite))
    {
        printf("\n写入远程进程内存空间出错%d。",GetLastError());
    }

    temp = (int)lpRemoteBuf; //获取shellcode的地址
    temp = temp+47; //得到shellcode中jmp xx的下条指令的地址
    temp = iNode->address - temp+5; //得到jmp回原来函数第六个字节的起始地址
    buff[0] = 0xe9;
    buff[1] = temp&0xff;
    buff[2] = temp>>8&0xff;
    buff[3] = temp>>16&0xff;
    buff[4] = temp>>24&0xff;
    temp = (int)lpRemoteBuf+42; //得到jmp xxx在shellcode中的地址,并写入shellcode最后五个字节
    if(WriteProcessMemory(hProcess,temp , buff, 5, &dwHasWrite))
    {
        return 0;
    }
    else
    {
        printf("\n写入远程进程内存空间出错%d。",GetLastError());
    }

    CloseHandle(hProcess);
    return -1;
}
```

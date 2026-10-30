
# [原创]2题第4题。。我的方法。。

```c
//标志减7就是函数头

BYTE DecodeFlag[16] = {0x8B, 0xFA, 0x83, 0xC9, 0xFF, 0x33, 0xC0, 0x33, 0xF6, 0xF2, 0xAE, 0xF7, 0xD1, 0x49, 0x8B, 0xE9};

//搜索数据
BYTE* SearchData(BYTE* dataall,int alllen,BYTE* data,int len)
{
        for(int i = 0; i < alllen; i++)
        {
                if( !memcmp(dataall+i,data,len) )
                        return dataall+i;
        }
        return NULL;
}

HANDLE handle;
DWORD VirtualBaseAddr;
DWORD FuncAddr;
BYTE reg_al;
BYTE reg_bl;
BYTE reg_cl;
BYTE reg_dl;

//list<t_disasm> dalist;
CLink dalist;

void Write(char* data)
{
        SetFilePointer(handle,0,NULL,FILE_END);
        int filelen = strlen(data);
        data[filelen++] = '\r';
        data[filelen++] = '\n';
        DWORD dw;
        WriteFile(handle,data,filelen,&dw,NULL);
        dw = GetLastError();
}

static char espdata[0x1000000];
static DWORD espdataaddr;
static DWORD tmpesp;
static DWORD codeaddr;
static DWORD endesp;//ESP尾步
static BYTE gotoretcode[5] = {0xE9,0x00,0x00,0x00,0x00};
static DWORD dwoffset = 0;

char *codedata = new char[102400];

//执行代码
void ExecuteCode(BYTE* code,int len)
{
        memset(espdata,0,0x1000000);
        static DWORD dw;
        VirtualProtect(espdata,0x1000000,PAGE_READWRITE | PAGE_EXECUTE,&dw);
        espdataaddr = (DWORD)espdata + 0x0500000;
        codeaddr = (DWORD)code;
        //__asm _emit 0xCC
        _asm
        {
                pushad
                mov eax,_Exec_End
                sub eax,codeaddr
                sub eax,len
                sub eax,5
                mov dwoffset,eax
                popad
        }
        *(DWORD*)&gotoretcode[1] = dwoffset;
        memcpy(code+len,gotoretcode,5);
        _asm
        {
                pushad
                mov tmpesp,esp
                mov esp,espdataaddr
                mov al,reg_al
                mov bl,reg_bl
                mov cl,reg_cl
                mov dl,reg_dl
                jmp codeaddr
_Exec_End:
                mov endesp,esp
                mov esp,tmpesp
                popad
        }
        static char* data;
        data = (char*)*(DWORD*)endesp;
        Write(data);
}

void InsetList(t_disasm *da)
{
        //dalist.push_back(*da);
        t_disasm* t_da = new t_disasm;
        if( !t_da )
        {
                MsgBox("错误!");
                return;
        }
        memcpy(t_da,da,sizeof(t_disasm));
        dalist.AddTail(t_da);
}

void ClearList()
{
        dalist.RemoveAll();
}

//识别代码
void IdentifyCode(t_disasm *da)
{
        if( !strcmp(da->vm_name,"VMOV_REG08_IMM32") )
        {
                switch(da->reg[0])
                {
                case REG_EAX:
                        reg_al = da->immconst;
                        break;
                case REG_EBX:
                        reg_bl = da->immconst;
                case REG_ECX:
                        reg_bl = da->immconst;
                case REG_EDX:
                        reg_dl = da->immconst;
                }
        }
        if( !strcmp(da->vm_name,"VXOR_REG08_REG08") )
        {
                switch(da->reg[0])
                {
                case REG_EAX:
                        reg_al = 0;
                        break;
                case REG_EBX:
                        reg_bl = 0;
                case REG_ECX:
                        reg_bl = 0;
                case REG_EDX:
                        reg_dl = 0;
                }
        }
        if( da->cmdtype == C_CAL )//如果是CALL
        {
                if( da->jmpconst == FuncAddr )//如果是调用解密函数
                {
                        memset(codedata,0,102400);
                        char* pcode = codedata;
                        //list<t_disasm>::iterator itr = dalist.begin();
                        //for(itr = dalist.begin(); itr != dalist.end(); ++itr)
                        //{
                        //        t_disasm t_da = *itr;
                        //        memcpy(pcode,t_da.hexcode,t_da.codelen);
                        //        pcode += t_da.codelen;
                        //}
                        T_Link* link = dalist.GetLink();
                        while(link)
                        {
                                t_disasm* t_da = (t_disasm*)link->data;
                                memcpy(pcode,t_da->hexcode,t_da->codelen);
                                pcode += t_da->codelen;
                                link = dalist.GetNext();
                        }
                        char calltext[16];
                        t_asmmodel t_asm;
                        sprintf(calltext,"call %08X",(DWORD)Uncode);
                        char errtext[255];
                        int len = Assemble(calltext,(ulong)pcode,&t_asm,0,0,errtext);
                        memcpy(pcode,t_asm.code,len);pcode += len;
                        ExecuteCode((BYTE*)codedata,pcode-codedata);
                        ClearList();
                }
                else//否则清除链表
                {
                        ClearList();
                }
        }
        else if( da->cmdtype == C_RET )//遇到返回清除
        {
                ClearList();
        }
        else//否则添加到链表
        {
                InsetList(da);
        }
}

int _tmain(int argc, _TCHAR* argv[])
{
        CPEStructure PEStructure;
        if( argc < 2 )
        {
                delete[] codedata;codedata = NULL;
                printf("请输入分析参数如 - VParse.exe StrDiggerMe.exe\n");
                system("pause");
                return 0;
        }
        if( !PEStructure.OpenFileName(argv[1]) )
        {
                delete[] codedata;codedata = NULL;
                printf("请返回");
                system("pause");
                return 0;
        }
        handle = CreateFile("out.txt",GENERIC_READ | GENERIC_WRITE, 0, NULL, OPEN_ALWAYS, FILE_ATTRIBUTE_NORMAL,0);
        if( handle == INVALID_HANDLE_VALUE )
        {
                DWORD dw = GetLastError();
                MsgBox("打开文件失败!");
                return 0;
        }
        //得到入口点所在索引
        int OEPidx = PEStructure.FindSectionIndex(PEStructure.image_nt_headers.OptionalHeader.AddressOfEntryPoint);
        BYTE* pcode = SearchData((BYTE*)PEStructure.image_section[OEPidx],PEStructure.image_section_header[OEPidx].SizeOfRawData,(BYTE*)DecodeFlag,16);
        if(!pcode)
        {
                delete[] codedata;codedata = NULL;
                printf("没找到解码标志");
                system("pause");
                return 0;
        }
        pcode-=7;
        VirtualBaseAddr = PEStructure.image_nt_headers.OptionalHeader.ImageBase + PEStructure.image_section_header[OEPidx].VirtualAddress;
        FuncAddr = VirtualBaseAddr + (DWORD)(pcode - (BYTE*)PEStructure.image_section[OEPidx]);
        int   Segment_Len = PEStructure.image_section_header[OEPidx].SizeOfRawData;//段长度
        char* paddr = PEStructure.image_section[OEPidx];
        int   Segment_End = VirtualBaseAddr + Segment_Len;
        DWORD StepVirtualAddr = VirtualBaseAddr;
        while( StepVirtualAddr < Segment_End )
        {
                t_disasm da;
                int len = Disasm(paddr,16,StepVirtualAddr,&da,DISASM_CODE);
                IdentifyCode(&da);
                StepVirtualAddr += len;
                paddr += len;
        }
        delete[] codedata;codedata = NULL;
        CloseHandle(handle);
        return 0;
}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

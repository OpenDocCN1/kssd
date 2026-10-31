
# WIN7 任务门进入1环

## 环境
`win7 x86 sp1` 专业版
`vs 2008`

## windbg修改
任务门: `eq 80b99500 0000e500`00480000`

任务段: `eq 80b99000 + (0x48&0xf8)0000e940`90300068`

`cs`: `eq 80b99000 + (0x61 & f8) 00cfbb00`0000ffff`

`ds`: `eq 80b99000 + (0x69 & f8) 00cfb300`0000ffff`

`fs`: `eq  80b99000 + (0x79 & f8) 0040b300`00000fff`  //不需要赋值  但是有些人需要赋值  我的没赋值也可以

## 注意
千万不要`int3`嘿嘿嘿

修复`cr3`的代码是别人给我的,  `win 7`  `tss`切换之后 上一次的`tss`中不会保存`cr3`或者保存的是别的人的

如果`cr3 = 0`回到`vm`必然会崩溃,  如果保存的是别人的 返回之后 必然提示`c0000005`成功修复成当前程序的`cr3`即可

## 代码

```c
#include <STDIO.H>

#include <WINDOWS.H>

typedef struct _KTSS
{
	USHORT Backlink;                                                        //0x0
	USHORT Reserved0;                                                       //0x2
	ULONG Esp0;                                                             //0x4
	USHORT Ss0;                                                             //0x8
	USHORT Reserved1;                                                       //0xa
	ULONG Esp1;                                                             //0x4
	USHORT Ss1; 
	USHORT Reserved111; 
	ULONG Esp2;                                                             //0x4
	USHORT Ss2;		
	USHORT Reserved1111;													//0xc
	ULONG CR3;                                                              //0x1c
	ULONG Eip;                                                              //0x20
	ULONG EFlags;                                                           //0x24
	ULONG Eax;                                                              //0x28
	ULONG Ecx;                                                              //0x2c
	ULONG Edx;                                                              //0x30
	ULONG Ebx;                                                              //0x34
	ULONG Esp;                                                              //0x38
	ULONG Ebp;                                                              //0x3c
	ULONG Esi;                                                              //0x40
	ULONG Edi;                                                              //0x44
	USHORT Es;                                                              //0x48
	USHORT Reserved2;                                                       //0x4a
	USHORT Cs;                                                              //0x4c
	USHORT Reserved3;                                                       //0x4e
	USHORT Ss;                                                              //0x50
	USHORT Reserved4;                                                       //0x52
	USHORT Ds;                                                              //0x54
	USHORT Reserved5;                                                       //0x56
	USHORT Fs;                                                              //0x58
	USHORT Reserved6;                                                       //0x5a
	USHORT Gs;                                                              //0x5c
	USHORT Reserved7;                                                       //0x5e
	USHORT LDT;                                                             //0x60
	USHORT Reserved8;                                                       //0x62
	USHORT Flags;                                                           //0x64
	USHORT IoMapBase;                                                       //0x66                                             //0x208c
}KTSS,*PKTSS; 

char esp3[0x2000] = {0};
char esp0[0x2000] = {0};

KTSS tss = {0};

char g_sgdt[6]={0};
DWORD h_gdt = 0;
DWORD l_gdt = 0;
char base[0x4]={0};
DWORD dwCr3 = 0;

__declspec(naked) Func()
{
	__asm{
		//int 3;

		sgdt g_sgdt
			mov eax,dword ptr [g_sgdt+2]
		mov ecx,[eax+0x28]
		mov eax,[eax+0x28+4]
		mov h_gdt,eax
			mov l_gdt,ecx
			mov eax,h_gdt
			shr eax,24
			and eax,0x000000ff
			mov [base+3],al
			mov eax,h_gdt        
			and eax,0x000000ff        
			mov [base+2],al
			mov eax,l_gdt
			shr eax,16
			and eax,0x0000ffff
			mov [base+1],ah
			mov [base],al
			mov eax,dword ptr [base]
		mov ecx,dwCr3
			mov [eax+0x1c],ecx    

		pushfd;
		pop eax;
		or eax,0x4000;
		push eax;
		popfd;	

		iretd;
	}
}

int main(void)
{
	
	char buf[6] = {0,0,0,0,0x48,0};
	
	memset(&tss,0,sizeof(KTSS));
	memset(&esp3,0xcc,sizeof(esp3));
	memset(&esp0,0xcc,sizeof(esp0));

	printf("TssAdr: %x\n",&tss);
	printf("sizeof TSS struct: %x\n",sizeof(KTSS));

	printf("input Cr3: ");
	scanf("%x",&dwCr3);
	tss.CR3 = (ULONG)dwCr3;

	tss.Cs = 0x61;
	tss.Ss = 0x69;
	tss.Ds = 0x23;
	tss.Es = 0x23;
	//tss.Fs = 0x79;
	tss.Ss1 = 0x69;

	tss.Eip = (ULONG)Func;
	tss.Esp = (ULONG)(esp3 + 0x2000 - 0x8);
	tss.Esp1 = (ULONG)(esp0 + 0x2000 - 0x8);

	printf("funcAddr: %x\n",Func);
	printf("eip: %x\n",tss.Eip);
	printf("esp0: %x\n",tss.Esp0);
	printf("esp3: %x\n",tss.Esp);
	printf("CR3: %x\n",tss.CR3);

	system("pause");

	__asm{
		int 0x20;
	}

	system("pause");
	return 0;
}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2020-10-27 12:55 被清风qfccc编辑，原因：漏掉了细节补充一下

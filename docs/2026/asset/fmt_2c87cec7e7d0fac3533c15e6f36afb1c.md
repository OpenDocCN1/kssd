
```cpp
*ctx.vEsp = num1 ^ num2;
return;
}

void vNot(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    *ctx.vEsp = ~(*ctx.vEsp);
    return;
}

void vRet(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    ctx.state = 0xff;
    return;
}

void vRead(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    int* addr = (int*)(*ctx.vEsp);
    *ctx.vEsp = *addr;
    return;
}

void vPopReg32(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    unsigned char reg = *ctx.opcode++;
    ((int*)(&ctx))[reg / 4] = *ctx.vEsp;
    ctx.vEsp++;
    return;
}

void vPushReg32(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    unsigned char reg = *ctx.opcode++;
    ctx.vEsp--;
    *(ctx.vEsp) = ((unsigned int*)(&ctx))[reg / 4];
    return;
}

void vPushImm32(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    unsigned int Imm32 = *(unsigned int*)ctx.opcode;
    ctx.opcode += 4;
    ctx.vEsp--;
    *ctx.vEsp = Imm32;
}

void vAdd(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    unsigned int num1 = *ctx.vEsp;
    ctx.vEsp++;
    unsigned int num2 = *ctx.vEsp;
    *ctx.vEsp = num1 + num2;
    return;
}

void vSub(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    unsigned int num1 = *ctx.vEsp;
    ctx.vEsp++;
    unsigned int num2 = *ctx.vEsp;
    *ctx.vEsp = num1 - num2;
    return;
}

void vDiv(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    unsigned int num1 = *ctx.vEsp;
    ctx.vEsp++;
    unsigned int num2 = *ctx.vEsp;
    *ctx.vEsp = num1 / num2;
    return;
}

void vMul(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    unsigned int num1 = *ctx.vEsp;
    ctx.vEsp++;
    unsigned int num2 = *ctx.vEsp;
    *ctx.vEsp = num1 * num2;
    return;
}

void vRSH(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    unsigned int Imm32 = *(unsigned int*)ctx.opcode;
    ctx.opcode += 4;
    *ctx.vEsp >>= Imm32;
}

void vLSH(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    unsigned int Imm32 = *(unsigned int*)ctx.opcode;
    ctx.opcode += 4;
    *ctx.vEsp <<= Imm32;
}

void vAnd(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    unsigned int num1 = *ctx.vEsp;
    ctx.vEsp++;
    unsigned int num2 = *ctx.vEsp;
    *ctx.vEsp = num1 & num2;
    return;
}

void vOr(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    unsigned int num1 = *ctx.vEsp;
    ctx.vEsp++;
    unsigned int num2 = *ctx.vEsp;
    *ctx.vEsp = num1 | num2;
    return;
}

void vXor(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    unsigned int num1 = *ctx.vEsp;
    ctx.vEsp++;
    unsigned int num2 = *ctx.vEsp;
    *ctx.vEsp = num1 ^ num2;
    return;
}

void vNot(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    *ctx.vEsp = ~(*ctx.vEsp);
    return;
}

void vRet(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    ctx.state = 0xff;
    return;
}

void vRead(VMContext& ctx)
{
    unsigned char op = *ctx.opcode++;
    int* addr = (int*)(*ctx.vEsp);
    *ctx.vEsp = *addr;
    return;
}

int VmEntry(unsigned char* opcode)
{
    VMContext ctx{0};
    //VmInit(&ctx, opcode);
    unsigned int* caller_ebp;
    __asm {
        mov eax, [ebp] // 获取add函数的ebp
        mov caller_ebp, eax
    }

    VmInit(&ctx, opcode);
    // 手动设置正确的ebp
    ctx.ebp = caller_ebp;
    while (true)
    {
        if (ctx.state == 0xff)
            break;
        unsigned char op = DecrypyOpcode(GetOpcode(ctx));
        g_handle[op](ctx);
    }

    return ctx.eax;
}

int VmEntry(unsigned char* opcode)
{
    VMContext ctx{0};
    //VmInit(&ctx, opcode);
    unsigned int* caller_ebp;
    __asm {
        mov eax, [ebp] // 获取add函数的ebp
        mov caller_ebp, eax
    }

    VmInit(&ctx, opcode);
    // 手动设置正确的ebp
    ctx.ebp = caller_ebp;
    while (true)
    {
        if (ctx.state == 0xff)
            break;
        unsigned char op = DecrypyOpcode(GetOpcode(ctx));
        g_handle[op](ctx);
    }

    return ctx.eax;
}

// 加法字节码
unsigned char g_opAdd[] =
{
    // 获取参数1 (a)：ebp + 0x08
    0x78, 0x10, // vPush ebp
    0x7B, 0x08, 0x00, 0x00, 0x00, // vPush 0x08
    0x7A, // vAdd (计算ebp + 0x08)
    0x77, // vRead (读取参数1)
    // 获取参数2 (b)：ebp + 0x0C
    0x78, 0x10, // vPush ebp
    0x7B, 0x0C, 0x00, 0x00, 0x00, // vPush 0x0C
    0x7A, // vAdd (计算ebp + 0x0C)
    0x77, // vRead (读取参数2)
    // 执行加法
    0x7A, // vAdd (参数1 + 参数2)
    // 返回结果
    0x79, 0x00, // vPop eax (结果保存到eax)
    0x74                           // vRet
};

/* 加法调用如下 */
__declspec(naked) int add(int a, int b)
{
    __asm {
        // 建立栈帧
        push ebp
        mov ebp, esp
        // 保存寄存器
        push eax
        push ebx
        push ecx
        push edx
        // 调用VM
        push offset g_opAdd
        call VmEntry
        add esp, 4
        // 恢复寄存器 (eax是返回值，不恢复)
        pop edx
        pop ecx
        pop ebx
        add esp, 4  // 跳过原eax
        // 清理栈帧
        mov esp, ebp
        pop ebp
        ret
    }
}

// 加法字节码
unsigned char g_opAdd[] =
{
    // 获取参数1 (a)：ebp + 0x08
    0x78, 0x10, // vPush ebp
    0x7B, 0x08, 0x00, 0x00, 0x00, // vPush 0x08
    0x7A, // vAdd (计算ebp + 0x08)
    0x77, // vRead (读取参数1)
    // 获取参数2 (b)：ebp + 0x0C
    0x78, 0x10, // vPush ebp
    0x7B, 0x0C, 0x00, 0x00, 0x00, // vPush 0x0C
    0x7A, // vAdd (计算ebp + 0x0C)
    0x77, // vRead (读取参数2)
    // 执行加法
    0x7A, // vAdd (参数1 + 参数2)
    // 返回结果
    0x79, 0x00, // vPop eax (结果保存到eax)
    0x74                           // vRet
};

/* 加法调用如下 */
__declspec(naked) int add(int a, int b)
{
    __asm {
        // 建立栈帧
        push ebp
        mov ebp, esp
        // 保存寄存器
        push eax
        push ebx
        push ecx
        push edx
        // 调用VM
        push offset g_opAdd
        call VmEntry
        add esp, 4
        // 恢复寄存器 (eax是返回值，不恢复)
        pop edx
        pop ecx
        pop ebx
        add esp, 4  // 跳过原eax
        // 清理栈帧
        mov esp, ebp
        pop ebp
        ret
    }
}

#include <stdio.h>
#include <malloc.h>
#include <time.h>
#include <random>

#define STACK_SIZE 0x1000

void* g_orgStack = nullptr;

/* x86堆栈环境 */
typedef struct _VMContext
{
    /*
    eax 00
    ebx 04
    ecx 08
    edx 0c
    ebp 10
    esp 14
    edi 18
    esi 1c
    eflags 20
    */
    unsigned int eax;
    unsigned int ebx;
    unsigned int ecx;
    unsigned int edx;
    unsigned int* ebp;
    unsigned int* esp;
    unsigned int* vEsp;
    unsigned int edi;
    unsigned int esi;
    unsigned int eflags;
    unsigned char* opcode;
    int state;
} VMContext, *PVMContext;

/* HANDLE */
typedef void (*HANDLE)(VMContext& ctx);

/*

*
00 (参数指示位) 000000 Handle idx
应该存在一个表用于随机映射handle
00 vPop Reg32 弹出到寄存器
01 vPush Reg32 压入寄存器
02 vPush Imm32 压入立即数
03 vAdd 加法
04 vSub 减法
05 vDiv 除法
```

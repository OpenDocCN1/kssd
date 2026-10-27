
#### （一）反汇编代码比对与分析

给出用`加花的程序`的关键函数部分的反汇编代码。

```
//实现求n^{m}
#include<stdio.h>
#include<windows.h>
int a = 0;
void __declspec(naked)__cdecl cnuF(int* a)//裸函数，开辟和释放堆栈由我们自己写。
{
    //55 8b ec 83
    __asm
    {
        /*保留栈底*/
        push ebp
        /*开辟栈空间*/
        mov ebp, esp
        sub esp, 0x40  //0x40是缓冲区大小

        /*保留现场（寄存器状态）*/
        push ebx
        push esi
        push edi

        /*缓冲区写入数据*/
        mov eax, 0xCCCCCCCC  //0xCCCC在gb2312中是'烫'字
        mov ecx, 0x10        //cx为下面填'烫'操作计数
        lea edi, dword ptr ds : [ebp - 0x40]
        rep stos dword ptr es : [edi] //用烫填充
    }

    /*执行的操作*/
    *a = 1;
    MessageBox(NULL, "加花指令8", "YQC", 0);
    //MessageBox(NULL, "未加花指令", "YQC", 0);
    int n, m, result, i;
    printf("请输入n和m:\n");
    scanf_s("%d%d", &n, &m);
    /*花指令*/
    _asm
    {
        call LABEL9;
        _emit 0xE8; //垃圾指令
        _emit 0x01;
        _emit 0x00;
        _emit 0x00;
        _emit 0x00;

        LABEL9:
        push eax;
        push ebx;
        lea  eax, dword ptr ds : [ebp - 0x0]
        add dword ptr ss : [eax - 0x50], 26;

        pop eax;
        pop ebx;
        pop eax;
        jmp eax;
        __emit 0xE8;
        _emit 0x03;
        _emit 0x00;
        _emit 0x00;
        _emit 0x00;
        mov eax, dword ptr ss : [esp - 8];

    }
    for (result = 1, i = 0; i < m; i++)
        result *= n;
    printf("结果为%d", result);

    __asm
    {
        /*恢复现场*/
        pop edi
        pop esi
        pop ebx

        /*释放栈空间*/
        mov esp, ebp
        pop ebp
        ret
    }

}
int main()
{
    cnuF(&a);
    return 0;
}
```

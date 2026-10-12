
# 发一个从导出表查找 api 然后调用的例子

> 原文：[`www.pediy.com/kssd/pediy12/128533.html`](https://www.pediy.com/kssd/pediy12/128533.html)

嗯嗯这些其实都是很久很久以前的技术了,发到这里来也只是怀旧一下吧,嘿嘿  

下面的代码就是我学习这些技术的时候写的,本来是用 `C` 写的,写完了觉得 `C` 不够过瘾,于是用汇编重写了一个,大牛不要看了吧,很基础的东西....  

代码:

```
#define nth 0x3c
#define exp 0x78
#define funname 0x20
#define funaddr 0x1c
#define expfunnum 0x18

#define kernelbase 0x00
#define gpa 0x04
#define expb 0x08
#define expnum 0x0c
#define expfunname 0x10
#define expfunaddr 0x14
#define user32h 0x18
#define msgboxf 0x1c
#define loadlib 0x20
//void dcores();

void __declspec(naked) dcores()
{
  __asm
  {
    jmp start;
user32table:
    __emit 'u';
    __emit 's';
    __emit 'e';
    __emit 'r';
    __emit '3';
    __emit '2';
    __emit '.';
    __emit 'd';
    __emit 'l';
    __emit 'l';
    __emit 0;
msgboxtable:
    __emit 'M';
    __emit 'e';
    __emit 's';
    __emit 's';
    __emit 'a';
    __emit 'g';
    __emit 'e';
    __emit 'B';
    __emit 'o';
    __emit 'x';
    __emit 'A';
    __emit 0;
loadlibrarytable:
    __emit 'L';
    __emit 'o';
    __emit 'a';
    __emit 'd';
    __emit 'L';
    __emit 'i';
    __emit 'b';
    __emit 'r';
    __emit 'a';
    __emit 'r';
    __emit 'y';
    __emit 'A';
    __emit 0;
start:
    push ebp;
    sub esp, kernelbase+gpa+expb+expnum+expfunname+expfunaddr+user32h+msgboxf;

    mov eax, fs:[0];
re1:
    mov ebx, [eax+4];
    cmp dword ptr [eax], 0xFFFFFFFF;
    je ok;
    mov eax, [eax];
    jmp re1;
ok:
    cmp dword ptr [ebx], 0x00905a4d;
    je ok1;
    dec ebx;
    jmp ok;
ok1:
    mov eax, ebx;
    add eax, [ebx+nth];
    cmp dword ptr [eax], 0x00004550;
    jne end;
    mov [esp+kernelbase], ebx;
    mov ebp, ebx;
    add ebx, [eax+exp];
    mov [esp+expb], ebx;
    mov eax, [ebx+expfunnum];
    mov [esp+expnum], eax;
    mov ecx, eax;
    mov eax, [ebx+funname];
    add eax, ebp;
    mov [esp+expfunname], eax;
    mov eax, [ebx+funaddr];
    add eax, ebp;
    mov [esp+expfunaddr], eax;
loopfindname:
    mov eax, [esp+expfunname];
    mov eax, [eax];
    add eax, ebp;
    cmp dword ptr [eax], 'PteG';
    jne notf;
    cmp dword ptr [eax+4], 'Acor';
    jne notf;
    cmp dword ptr [eax+8], 'erdd';
    jne notf;
    cmp word ptr [eax+12], 'ss';
    jne notf;
    mov eax, [esp+expfunaddr];
    mov eax, [eax];
    add eax, ebp;
    jmp end;
notf:
    add dword ptr [esp+expfunname], 4;
    add dword ptr [esp+expfunaddr], 4;
    loop loopfindname;


end:
    mov [esp+gpa], eax;
    mov eax, loadlibrarytable;
    push eax;
    mov ebx, [esp+kernelbase+4];
    push ebx;
    mov eax, [esp+gpa+4+4];
    call eax;
    mov [esp+loadlib], eax;
    push user32table;
    mov ebx, [esp+loadlib+4];
    call ebx;
    mov [esp+user32h], eax;
    mov ecx, [esp+gpa];
    mov ebx, msgboxtable;
    push ebx;
    push eax;
    call ecx;
    mov [esp+msgboxf], eax;
    push 0;
    push msgboxtable;
    push msgboxtable;
    push 0;
    call eax;
    add esp, kernelbase+gpa+expb+expnum+expfunname+expfunaddr+user32h+msgboxf;
    pop ebp;
    ret;
  }

}
```

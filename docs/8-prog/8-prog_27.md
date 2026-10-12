
# VC++中委托的简单实现(源代码)

> 原文：[`www.pediy.com/kssd/pediy11/104950.html`](https://www.pediy.com/kssd/pediy11/104950.html)

在 `C#` 中的委托(deletegate)极大地增强了语言的功能,以下代码是在 `VC++` 中的简单实现.欢迎交流.  

参考资料:http://blog.csdn.net/chenjiazhou/relatedarticles/470323.aspx  

代码:

```cpp
// vc++ compile: cl /nologo /MD DelegateTest.cpp

#include <stdio.h>
#pragma runtime_checks( "s", off )    // 由于使用内联汇编,当启用堆栈帧运行时错误检查会产生错误

#pragma pack(push, 4)
struct Delegate
{
public:
    Delegate(){ pObj = 0; pFunc = 0; }
   
    void Bind(void (*pFunction)(void*)){ Bind(0, pFunction); }
    void Bind(void *pobj, void *pF){ pObj = pobj; pFunc = pF; }

    int operator==(Delegate& d) const
    { return (pObj == d.pObj) && (pFunc == d.pFunc); }

public:
    void Invoke(void*);

protected:
    void* pObj;
    void* pFunc;
};
#pragma pack(pop)

__declspec(naked) 
void Delegate::Invoke(void* p)  // key code
{
    __asm
    {
        mov eax, ecx
        mov ecx, dword ptr [eax+0]  ; pObj
        jmp dword ptr [eax+4]       ; pFunc
    }
}

inline Delegate CreateDelegate(void (*pFunction)(void*))
{ Delegate d; d.Bind(0, pFunction); return d; }

#define DELEGATE(ClassName)  \
inline Delegate CreateDelegate(ClassName *pobj, void (ClassName::*pFunction)(void*)) \
{ union { void* pF; void (ClassName::*pTmp)(void*); } u; u.pTmp = pFunction; Delegate d; d.Bind(pobj, u.pF); return d; }\

class MulticastDelegate
{
private:
    struct DelegateNode
    {
        DelegateNode(){ hander.Bind(0,0); pNext = 0; }
        ~DelegateNode(){ hander.Bind(0,0); pNext = 0; }
     
        Delegate hander;
        DelegateNode* pNext;
    };

public:
    MulticastDelegate()
    {
        pHead = 0;
    }
    ~MulticastDelegate()
    {
        DelegateNode* p = pHead;
        while(p != 0)
        {
            DelegateNode* pnext = p->pNext;
            delete p;
            p = pnext;
        }
        pHead = 0;
    }

    void AddDelegate(Delegate d)
    {
        DelegateNode* pNew = new DelegateNode;
        pNew->hander = d;
        pNew->pNext = pHead;
        pHead = pNew;
    }
    void operator+=(Delegate d)
    { AddDelegate(d); }

    void RemoveDelegate(Delegate d)
    {
        if(pHead == 0)
        return;

        if(pHead->hander == d)
        {
        DelegateNode* p = pHead->pNext;
        delete pHead;
        pHead = p;
        return;
        }

        DelegateNode *pLast, *pCur;
        pLast = pHead;
        pCur = pLast->pNext;
        while(pCur != 0)
        {
            if(pCur->hander == d)
            {
                DelegateNode* tmp = pCur ->pNext;
                delete pCur;
                pLast->pNext = tmp;
                break;
            }   
            else
            {
                pLast = pCur;
                pCur = pCur->pNext;
            }
        }
    }
    void operator-=(Delegate d)
    { RemoveDelegate(d); }

    void Invoke(void* param)
    {
        DelegateNode* p = pHead;
        while(p != 0)
        {
            p->hander.Invoke(param);
            p = p->pNext;
        }
    }

private:
    DelegateNode *pHead;
};

class A
{
public:
    void Func1(void* p){ printf("A::Func1(%d)\n", *(int*)p); }
};
DELEGATE(A)

class B
{
public:
    void Func1(void* p){ printf("B::Func1(%d)\n", *(int*)p); }
};
DELEGATE(B)

void Demo(void* p){ printf("Demo(%d)\n", *(int*)p); }

int main(int argc, char *argv[])
{
    A a;
    B b;
    int param = 0;

    printf("Test single delegate...\n");

    Delegate x[3];
    x[0] = CreateDelegate(&a, &A::Func1);
    param++;
    x[0].Invoke(&param);
    x[1] = CreateDelegate(&b, &B::Func1);
    param++;
    x[1].Invoke(&param);
    x[2].Bind(Demo);
    param++;
    x[2].Invoke(&param);

    printf("\nTest multicast delegate...\n");
    MulticastDelegate md;
    md += CreateDelegate(&a, &A::Func1);
    md += CreateDelegate(&b, &B::Func1);
    md += CreateDelegate(Demo);
    md.Invoke(&param);

    printf("\nTest remove delegate...\n");

    md -= CreateDelegate(&b, &B::Func1);
    md.Invoke(&param);

    printf("\nTest OK!\n");
    return 0;
}

/* output:
R:\>cl /nologo /MD Delegate.cpp
Delegate.cpp

R:\>delegate
Test single delegate...
A::Func1(1)
B::Func1(2)
Demo(3)

Test multicast delegate...
Demo(3)
B::Func1(3)
A::Func1(3)

Test remove delegate...
Demo(3)
A::Func1(3)

Test OK!
*/
```


# 久其新某集成账务核算软件

> 原文：[`www.pediy.com/kssd/pediy05/pediy50411.htm`](https://www.pediy.com/kssd/pediy05/pediy50411.htm)

/\*某账务系统软件破解：

很久没破东西了，这次因朋友强烈要求，破一个试试，不想太简单了，一次成功。

调试器载入，下断点，追到关键处：

```
CODE:005DFE6E                call    sub_0_437078

CODE:005DFE73                mov     eax, [ebp+var_40] ; 标志 1

CODE:005DFE76                mov     ecx, [ebp+var_C] ; 标志 2

CODE:005DFE79                pop     edx                 ; 标志 3

CODE:005DFE7A                call    sub_0_4DB960        ; 注册码计算过程

CODE:005DFE7F                cmp     eax, [ebp+var_4] ; 注册码伪码对比

CODE:005DFE82                jnz     loc_0_5DFF8E        ; 跳转到错误处
```

`eax`是注册码。。。。不说什么了。。。。

delphi写的，子过程经过分析都改了名字，冗余代码太多，实在无趣，于是分析一下算法并写了一个小注册机，enjoy it！

```
E:004DB960 sub_0_4DB960    proc near            ; CODE XREF: sub_0_4DBC44+52p

CODE:004DB960

CODE:004DB960                push    ebp

CODE:004DB961                mov     ebp, esp

CODE:004DB963                add     esp, 0FFFFFFC4h

CODE:004DB966                push    ebx

CODE:004DB967                push    esi

CODE:004DB968                push    edi

。。。。。。。。。。。。。。。。

CODE:004DB99E                xor     eax, eax

CODE:004DB9A0                push    ebp

CODE:004DB9A1                push    offset exception_handler0 ; 异常处理

CODE:004DB9A1                                                ;

CODE:004DB9A6                push    dword ptr fs:[eax]

CODE:004DB9A9                mov     fs:[eax], esp

CODE:004DB9AC                mov     [ebp+var_10], 99813721h

CODE:004DB9B3                mov     dl, 1

。。。。。。。。。。。。。。。。。

CODE:004DBA04                mov     eax, [ebp+var_4]

CODE:004DBA07                call    get_string_len      ; 得到用户名长度，处理用户名

CODE:004DBA0C                test    eax, eax

CODE:004DBA0E                jle     short loc_0_4DBA7A

CODE:004DBA10                mov     [ebp+var_20], eax

CODE:004DBA13                mov     ebx, 1

CODE:004DBA18

CODE:004DBA18 loc_0_4DBA18:               ; CODE XREF: sub_0_4DB960+118j

CODE:004DBA18                lea     edx, [ebx+3]

CODE:004DBA1B                and     edx, 8000001Fh

CODE:004DBA21                jns     short loc_0_4DBA28 ; 机器特征码,初始值为注册窗之值

CODE:004DBA23                dec     edx

CODE:004DBA24                or      edx, 0FFFFFFE0h

CODE:004DBA27                inc     edx

CODE:004DBA28

CODE:004DBA28 loc_0_4DBA28:               ; CODE XREF: sub_0_4DB960+C1j

CODE:004DBA28                mov     eax, [ebp+var_18] ; 机器特征码,初始值为注册窗之值

CODE:004DBA2B                call    rol_eax_edx_bit   ; 伪码 rol eax，edx

CODE:004DBA30                mov     esi, eax

CODE:004DBA32                mov     eax, [ebp+var_4]

CODE:004DBA35                mov     al, [eax+ebx-1]

CODE:004DBA39                mov     dl, 3

CODE:004DBA3B                call    rol_al_dl_bit

CODE:004DBA40                mov     edi, eax

CODE:004DBA42                and     edi, 0FFh

CODE:004DBA48                mov     eax, [ebp+var_4]

CODE:004DBA4B                xor     ecx, ecx

CODE:004DBA4D                mov     cl, [eax+ebx-1]

CODE:004DBA51                and     ecx, 7

CODE:004DBA54                add     ecx, 4

CODE:004DBA57                shl     edi, cl

CODE:004DBA59                mov     eax, [ebp+var_4]

CODE:004DBA5C                mov     al, [eax+ebx-1]

CODE:004DBA60                mov     dl, 5

CODE:004DBA62                call    rol_al_dl_bit

CODE:004DBA67                and     eax, 0FFh

CODE:004DBA6C                imul    edi, eax

CODE:004DBA6F                add     esi, edi

CODE:004DBA71                add     [ebp+var_10], esi

CODE:004DBA74                inc     ebx

CODE:004DBA75                dec     [ebp+var_20]

CODE:004DBA78                jnz     short loc_0_4DBA18

CODE:004DBA7A

CODE:004DBA7A loc_0_4DBA7A:               ; CODE XREF: sub_0_4DB960+AEj

CODE:004DBA7A                mov     eax, [ebp+var_8]

CODE:004DBA7D                call    get_string_len    ; 大致同上，处理公司名

CODE:004DBA82                test    eax, eax

CODE:004DBA84                jle     short loc_0_4DBAF0

CODE:004DBA86                mov     [ebp+var_20], eax

CODE:004DBA89                mov     ebx, 1

CODE:004DBA8E

CODE:004DBA8E loc_0_4DBA8E:               ; CODE XREF: sub_0_4DB960+18Ej

CODE:004DBA8E                lea     edx, [ebx+8]

CODE:004DBA91                and     edx, 8000001Fh

CODE:004DBA97                jns     short loc_0_4DBA9E

CODE:004DBA99                dec     edx

CODE:004DBA9A                or      edx, 0FFFFFFE0h

CODE:004DBA9D                inc     edx

CODE:004DBA9E

CODE:004DBA9E loc_0_4DBA9E:               ; CODE XREF: sub_0_4DB960+137j

CODE:004DBA9E                mov     eax, [ebp+var_18]

CODE:004DBAA1                call    rol_eax_edx_bit

CODE:004DBAA6                mov     esi, eax

CODE:004DBAA8                mov     eax, [ebp+var_8]

CODE:004DBAAB                mov     al, [eax+ebx-1]

CODE:004DBAAF                mov     dl, 1

CODE:004DBAB1                call    rol_al_dl_bit

CODE:004DBAB6                mov     edi, eax

CODE:004DBAB8                and     edi, 0FFh

CODE:004DBABE                mov     eax, [ebp+var_8]

CODE:004DBAC1                xor     ecx, ecx

CODE:004DBAC3                mov     cl, [eax+ebx-1]

CODE:004DBAC7                and     ecx, 7

CODE:004DBACA                add     ecx, 5

CODE:004DBACD                shl     edi, cl

CODE:004DBACF                mov     eax, [ebp+var_8]

CODE:004DBAD2                mov     al, [eax+ebx-1]

CODE:004DBAD6                mov     dl, 4

CODE:004DBAD8                call    rol_al_dl_bit

CODE:004DBADD                and     eax, 0FFh

CODE:004DBAE2                imul    edi, eax

CODE:004DBAE5                add     esi, edi

CODE:004DBAE7                add     [ebp+var_10], esi

CODE:004DBAEA                inc     ebx

CODE:004DBAEB                dec     [ebp+var_20]

CODE:004DBAEE                jnz     short loc_0_4DBA8E

CODE:004DBAF0

CODE:004DBAF0 loc_0_4DBAF0:               ; CODE XREF: sub_0_4DB960+124j

CODE:004DBAF0                cmp     [ebp+var_C], 0

CODE:004DBAF4                jz      loc_0_4DBBE6

CODE:004DBAFA                mov     ecx, [ebp+var_14]

CODE:004DBAFD                mov     dl, 3Bh       ; ";"分隔符，分离子串

CODE:004DBAFF                mov     eax, [ebp+var_C] ; 对应注册模块进行同样计算，不过略有变化，算法基本一致

CODE:004DBB02                call    sub_0_4DB394

CODE:004DBB07                mov     eax, [ebp+var_14]

CODE:004DBB0A                mov     edx, [eax]

CODE:004DBB0C                call    dword ptr [edx+14h]

CODE:004DBB0F                dec     eax

CODE:004DBB10                test    eax, eax

CODE:004DBB12                jl      loc_0_4DBBE6

CODE:004DBB18                inc     eax

CODE:004DBB19                mov     [ebp+var_20], eax

CODE:004DBB1C                mov     [ebp+var_1C], 0

CODE:004DBB23

CODE:004DBB23 loc_0_4DBB23:               ; CODE XREF: sub_0_4DB960+280j

CODE:004DBB23                lea     ecx, [ebp+var_30]

CODE:004DBB26                mov     edx, [ebp+var_1C]

CODE:004DBB29                mov     eax, [ebp+var_14]

CODE:004DBB2C                mov     ebx, [eax]

CODE:004DBB2E                call    dword ptr [ebx+0Ch]

CODE:004DBB31                mov     eax, [ebp+var_30]

CODE:004DBB34                call    get_string_len

CODE:004DBB39                test    eax, eax
```



注册机附上：

```cpp
#include <windows.h>
#include <iostream>
#include <string>
#include <vector>

using namespace std;

#define rol32(x,y) (((x)<<(y))|((x)>>(32-(y))))
#define rol8(x,y) (((x)<<(y))|((x)>>(8-(y))))

UINT calc_comm(UINT machine_ID, string &str, int x0, int x1, int x2, int x3);
void strip_blank(string &);
void copy_clip(int);

void __cdecl main(int argc, char *argv[])
{
    cout << "JQ financial soft V 4.0.1.24 keygen\n"
         << "if you need it enjoy it!\n"
         << "the way of Hume,2k3\n\n";

    string user, company;
    UINT machine_ID, regcode = 0;

    cout << "Please input the following infos:" << endl;
    cout << "\nUser name:\t\t";
    cin >> user;
    cout << "\nCompany name:\t\t";
    cin >> company;
    cout << "\nmachine ID(del \"-\"):\t";
    cin >> hex >> machine_ID;
    //should be xxxxxxxx format.

    strip_blank(user);
    strip_blank(company);

    //magic number
    regcode = 0x99813721;

    //first step process user name
    regcode += calc_comm(machine_ID, user, 3, 3, 4, 5);

    //second step process company name
    regcode += calc_comm(machine_ID, company, 8, 1, 5, 4);

    //step3 to register all modules....stupid vendor of the software!
    string ia[] = {"ZW", "GZ", "GDZC", "XJLL"};
    vector<string> svec(ia, ia + 4);

    for (int ix = 0; ix < svec.size(); ix++)
    {
        regcode += calc_comm(machine_ID, svec[ix], 0x16, 5, 6, 3);
    }

    regcode += machine_ID;
    cout << "\nThe registration code is:" << hex << HIWORD(regcode) << "-" << LOWORD(regcode) << endl << endl;

    copy_clip(regcode);
    cin.get();
}

//calculate regcode according to somesyntax x0,x1,x2,x3
UINT calc_comm(UINT machine_ID, string &str, int x0, int x1, int x2, int x3)
{
    UINT result = 0;
    int iy = 0;
    char ch;

    for (int ix = 0; ix < str.size(); ix++)
    {
        iy = ix + 1 + x0;
        ch = str[ix];
        //就是这么简单的算法
        result += rol32(machine_ID, iy) +
                  ((rol8(ch, x1) & 0xFF) << ((ch & 7) + x2)) *
                  (rol8(ch, x3) & 0xFF);
    }

    return result;
}

//to strim all blanks in the file
void strip_blank(string &str)
{
    string::size_type pos = 0;
    while ((pos = str.find_first_of(" ", pos)) != string::npos)
        str.erase(pos);
}

void copy_clip(int regcode)
{
    HGLOBAL hG = GlobalAlloc(GMEM_DDESHARE, 256);
    LPVOID pM = GlobalLock(hG);
    wsprintf((char *)pM, "%04X-%04X", HIWORD(regcode), LOWORD(regcode));
    GlobalUnlock(hG);

    OpenClipboard(NULL);
    EmptyClipboard();
    SetClipboardData(CF_TEXT, hG);
    CloseClipboard();
}

//the way of Hume Feb, 2K3   4dd466d0e1e0cc123afbe504da2462ca
```

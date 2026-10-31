
# [原创]一个速度不是很快但是逻辑很简单的内存特征码搜索

```cpp
// ConsoleApplication8.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。

// 很多头文件不需要，你自己看着用吧 我是复制我之前的

#include "pch.h"

#include <Windows.h>

#include <windowsx.h>

#include <process.h>

#include <direct.h>

#include <tchar.h>

#include <stdio.h>

#include <algorithm>

#include <math.h>

#include <Psapi.h>

#include <Tlhelp32.h>

#include <iostream>

#include <vector>

#include <string>

#include <map>

#include <ctime>

#include <thread>

/// 字符转字节

int ctoh(char hex)

{

    if (hex >= '0' && hex <= '9') return hex - '0';

    if (hex >= 'A' && hex <= 'F') return hex - 'A' + 10;

    if (hex >= 'a' && hex <= 'f') return hex - 'a' + 10;

    return 0;

}

/// 十六进制字符串转字节数组

std::vector<BYTE> HexStringToBytes(std::string str)

{

    std::vector<BYTE> Bytes;

    for (SIZE\_T i = 0; i < str.size(); i++)

    {

        if (str[i] == ' ')

        {

            continue;

        }

        if (str[i] == '?')

        {

            Bytes.insert(Bytes.end(), '?');

            // 兼容OD模式的特征码

            if (str[i+1] == '?')

                i++;

        }

        else if (str[i] == '\*') {

            Bytes.insert(Bytes.end(), '\*');

        }

        else {

            Bytes.insert(Bytes.end(), 0x10 \* ctoh(str[i]) + ctoh(str[i + 1]));

            i++;

        }

    }

    return Bytes;

}

// 获取进程模块句柄

HMODULE GetProcessModuleHandle(DWORD ProcessId, CONST TCHAR\* ModuleName)

{

    HMODULE hMods[1024];

    DWORD cbNeeded;

    TCHAR szModName[MAX\_PATH];

    HANDLE ProcessHandle = ::OpenProcess(PROCESS\_ALL\_ACCESS, FALSE, ProcessId);

    if (EnumProcessModulesEx(ProcessHandle, hMods, sizeof(hMods), &cbNeeded, 0x1))

    {

        for (size\_t i = 0; i < (cbNeeded / sizeof(HMODULE)); i++)

        {

            GetModuleFileNameEx(ProcessHandle, hMods[i], szModName, sizeof(szModName) / sizeof(TCHAR));

            if (\_tcscmp(szModName, ModuleName)) {

                return hMods[i];

            }

        }

    }

    CloseHandle(ProcessHandle);

    return NULL;

}

// 读字节数组

std::vector<BYTE> ReadBytes(HANDLE ProcessHandle, DWORD\_PTR Address, SIZE\_T Size)

{

    std::vector<BYTE> Bytes = {};

    Bytes.resize(Size);

    BYTE\* Buffer = new BYTE[Size];

    // HANDLE ProcessHandle = ::OpenProcess(PROCESS\_ALL\_ACCESS, FALSE, ProcessId);

    ReadProcessMemory(ProcessHandle, (LPCVOID)Address, Buffer, Size, NULL);

    for (size\_t i = 0; i < Size; i++)

    {

        Bytes[i] = Buffer[i];

    }

    delete[]Buffer;

    // CloseHandle(ProcessHandle);

    return Bytes;

}

// 用来保存模块内存数据

static std::map<CONST TCHAR\*, std::map<DWORD\_PTR, std::vector<BYTE>>> g\_ModuleMemorys;

//256 表示 0x00~0xff 特征码长度不要超过 256

static int maxByteSize = 256;

static int shift[256] = { 0 };

long Sunday(std::vector<byte> T, std::vector<byte> P)

{

    int n = T.size();

    int m = P.size();

    // 默认值，移动m+1位 256 表示 0x00~0xff

    for (int i = 0; i < maxByteSize; i++) {

        shift[i] = m + 1;

    }

    // 模式串P中每个字母出现的最后的下标

    // 所对应的主串参与匹配的最末位字符的下一位字符移动到该位，所需要的移动位数

    for (int i = 0; i < m; i++) {

        shift[P[i]] = m - i;

    }

    // 模式串开始位置在主串的哪里

    int s = 0;

    // 模式串已经匹配到的位置

    int j;

    while (s <= n - m) {

        j = 0;

        while (P[j] == '\*' || P[j] == '?' || T[s + j] == P[j]) {

            j++;

            // 匹配成功

            if (j >= m) {

                return s;

            }

        }

        // 找到主串中当前跟模式串匹配的最末字符的下一个字符

        // 在模式串中出现最后的位置

        // 所需要从(模式串末尾+1)移动到该位置的步数

        s += shift[T[s + m]];

    }

    return -1;

}

DWORD\_PTR FindFeatureCode(DWORD ProcessId, CONST TCHAR\* ModuleName, std::string FeatureCodeString)

{

    DWORD\_PTR Address = 0;

    DWORD\_PTR BeginAddress = 0;

    DWORD\_PTR EndAddress = 0;

    HANDLE ProcessHandle = NULL;

    MODULEINFO ModuleInfo;

    MEMORY\_BASIC\_INFORMATION mbi;

    std::map<DWORD\_PTR, std::vector<BYTE>> Memorys;

    std::map<DWORD\_PTR, std::vector<BYTE>>::iterator it;

    std::vector<BYTE> FeatureCode = HexStringToBytes(FeatureCodeString);

    // 这里获取模块内存信息

    if (g\_ModuleMemorys.find(ModuleName) == g\_ModuleMemorys.end())

    {

        //printf("%s\n", "123456");

        ProcessHandle = ::OpenProcess(PROCESS\_ALL\_ACCESS, FALSE, ProcessId);

        GetModuleInformation(ProcessHandle, GetProcessModuleHandle(ProcessId, ModuleName), &ModuleInfo, sizeof(ModuleInfo));

        BeginAddress = (DWORD\_PTR)ModuleInfo.lpBaseOfDll;

        EndAddress = (DWORD\_PTR)ModuleInfo.lpBaseOfDll + ModuleInfo.SizeOfImage;

        while (BeginAddress <= EndAddress)

        {

            ::VirtualQueryEx(ProcessHandle, (LPCVOID)BeginAddress, &mbi, sizeof(mbi));

            if (mbi.Protect != 1 && mbi.Protect != 16 && mbi.Protect != 512 && mbi.RegionSize > 1)

            {

                g\_ModuleMemorys[ModuleName].insert(std::pair<DWORD\_PTR, std::vector<BYTE>>(BeginAddress, ReadBytes(ProcessHandle, BeginAddress, mbi.RegionSize)));

            }

            BeginAddress += mbi.RegionSize;

        }

        CloseHandle(ProcessHandle);

    }

    Memorys = g\_ModuleMemorys[ModuleName];

    it = Memorys.begin();

    while (it != Memorys.end())

    {

        long offset = Sunday(it->second, FeatureCode);

        if (offset >= 0)

        {

            Address = it->first + offset;

            goto \_exit;

        }

        it++;

    }

    // 暴力匹配

    //SIZE\_T i = 0;

    //SIZE\_T j = 0;

    //// 这个就是匹配的逻辑了

    //while (it != Memorys.end())

    //{

    //    i = it->second.size();

    //    while (i--) {

    //        j = FeatureCode.size();

    //        while (j--) {

    //            if (j == 0)

    //            {

    //                Address = it->first + i;

    //                goto \_exit;

    //            }

    //            else if (FeatureCode[j] != it->second[i + j] && FeatureCode[j] != '?' && FeatureCode[j] != '\*') {

    //                break;

    //            }

    //        }

    //    }

    //    it++;

    //}

\_exit:

    return Address;

}

int main(int argc, char const\* argv[])

{

    clock\_t startTime, endTime;

    startTime = clock();//计时开始

    // 特征码字符串格式随意  这里是拿网易的有道词典测试的

    ULONG address = 0;

    address = FindFeatureCode(13232, L"YoudaoDict.exe", "3E A0 ?? ?? ?? ?? 3E AC 3E B0 ?? B4 ?? B8 ?? ?? ?? ?? 3E C4 3E C8 3E CC ?? D0 3E D4 ?? D8 3E DC 3E E0 ?? E4 ?? E8 ?? ?? ?? ??  ");

    printf("address %x\n", address);

    endTime = clock();//计时结束

    printf("运行时间: %f/s\n", (double)(endTime - startTime) / CLOCKS\_PER\_SEC);

    system("pause");

    //MSG msg;

    //while (GetMessage(&msg, NULL, 0, 0))

    //{

    //    //翻译消息

    //    TranslateMessage(&msg);

    //    //分发消息

    //    DispatchMessage(&msg);

    //}

    //return 0;

}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2020-5-8 10:58

被aas102400编辑

，原因：

[#其他内容](https://bbs.kanxue.com/forum-4-1-10.htm)

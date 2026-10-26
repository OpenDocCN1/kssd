
```
//Injector.exe

#include <iostream>

#include <windows.h>

using namespace std;

//要加载的dll路径
// 最好改为相对路径（相对于连连看程序的
// WCHAR szDllPath[] = L"C:\\Users\\15pb-win7\\Desktop\\MFCGamePlugin.dll";

WCHAR szDllPath[] = L"../../MFCGamePlugin.dll";

int main()
{
    //1.要注入，需要dll文件
    //2.找到要注入的进程PID
    DWORD dwPid = 0;
    //HWND hwnd = FindWindow(NULL, L"new 1 - Notepad++");
    //GetWindowThreadProcessId(hwnd, &dwPid);
    printf("please input PID>> ");
    scanf_s("%d", &dwPid);

    //3.打开进程，获取进程句柄
    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwPid);

    //4.在目标进程中申请空间
    LPVOID pBuff = VirtualAllocEx(
        hProcess,
        0,
        sizeof(szDllPath),
        MEM_RESERVE | MEM_COMMIT,
        PAGE_EXECUTE_READWRITE
    );

    //5.将路径写入到目标进程中
    DWORD dwSize;
    WriteProcessMemory(
        hProcess,
        pBuff,             //在指申请的地址上
        szDllPath,         //写入的内容
        sizeof(szDllPath), //写入大小
        &dwSize
    );

    //6.使用关键函数加载目标dll
    // 利用远程创建线程函数，实现目标进程加载dll
    // 远程线程执行函数直接指向LoadLibaray函数，同时参数指向dll路径，完美实现加载dll
    HANDLE hThread = CreateRemoteThread(
        hProcess,
        NULL,
        NULL,
        (LPTHREAD_START_ROUTINE)LoadLibrary, //线程执行地址指向LoadLibrary
        pBuff,                                //线程的附加参数dll路径
        NULL, NULL
    );

    //7 释放句柄
    CloseHandle(hProcess);
    CloseHandle(hThread);
}
```

5. `MFCGamePlugin.cpp` 关键代码

```
if (Msg == WM_DATA1)
{
    OutputDebugString(L"无限指南针");

    //0041DE4D | .  8B86 9404000 > MOV EAX, DWORD PTR DS : [ESI + 0x494]
    //0041DE53 | .  8D8E 9404000 > LEA ECX, DWORD PTR DS : [ESI + 0x494]
    //0041DE59 | .  52           PUSH EDX
    //0041DE5A | .  53           PUSH EBX
    //0041DE5B | .  53           PUSH EBX
    //0041DE5C | .FF50 28      CALL DWORD PTR DS : [EAX + 0x28];  使用指南针道具
    _asm
    {
        mov ecx, 0x45DEBC
        mov ecx, [ecx]
        LEA ECX, DWORD PTR DS : [ecx + 0x494]
        PUSH 0xF0 // 若炸弹，则F4
        PUSH 0
        PUSH 0
        mov eax, 0x0041E691
        call eax
    }

    return DefWindowProc(hWnd, Msg, wParam, lParam);
}
```

## 2.4 实现单次消除

1. 要想消除，就需要获得可以消除的两个点，由上知，指南针 `call` 中最里面的那个 `0041E76C`，就可以提供两个点（且可以消除）。
2. 在手动找两个可消除的两个点，消除过程中，必然会访问连连看数组（将相应位置置为 0），在数组处下内存写入断点，会断下来 `0040FF5F`，然后删除内存断点，`F2` 下断，通过栈回溯，不断找“消除”时会调用的 `call`。
3. （注意，写入哪个会在哪个断下，以每个字节为单位，并非写入数组中任意一个位置，都会断下，所以，根据点击的那两个将要消除的点，在数组内存所在处相应位置下断）
4. 从外到里，找到如下 `call`：`0041B4B7` - `0041AB34` - `0041C6C3`。
5. 就像指南针那个一样，必然不止一个，由内而外/由外而内依次检查每一个 `call`，看其做了什么工作，检查其参数都是干嘛的（看 `push` 了谁，代码或堆栈中看），看看哪个传入了点的坐标（要想消除，就需要这两个点）。
6. `0041B4B7`：内部 `retn 0x1c`（`0x1c` = 28，即 7 个参数），从栈顶依次找 7 个参数，就参数 2 靠谱点，是一个地址（其他都是数，一看就不是点坐标），数据窗口中跟随，确实是两个点坐标，再看游戏窗口中点击的那两个待消除的点，确实也符合，但是是一个地址中保存了两个点，而非理想中的一个参数对应一个点，先记下，继续往后找（尽量往里找，找更满足条件的）。
7. `0041AB34`：enter 进入 `call` 的内部，在最后 `retn 0x18`（`0x18` = 24，即 6 个参数），参数1-0、参数2-连连看数组地址、参数3-点1坐标、参数4-点2坐标、参数5-同上一样，有那两个点的坐标，暂记做坐标点数组、参数6-数值2，这么一看，这个 `call` 相当靠谱。
8. 最里层那个 `call` 4 个参数，不太靠谱，故从里往外，倒数第二个即为目标 `call`，`0041AB34`，通过其来构造汇编代码，实现程序外调用。
9. 难点及重点：如何构造 `call` 这个函数相应的 6 个参数？通过程序中汇编代码来构造，在 `call` 之前，第一个 `push` 处下断，看每一个参数的值都是怎么来的（追本溯源）。
10. 参数2/5比较难找：参数2 = `12BB50`，参数5 = `1A5DE18`，看这俩值怎么构造，是这么 `x+y=` 的。
11. 注意一点：`call` 单次消除用到获取两点坐标功能，而后者又是在 `call` 指南针功能中调用的，就像注释中所说的：

    ```
    lea ecx, DWORD PTR DS : [ecx + 0x494]// 要加上此，原程序中，此函数是在call指南针内部call的

    mov ecx, DWORD PTR DS : [ecx + 0x19F0]// 即在前面的基础上调用的，因此ecx...
    ```

12. 注意：不仅要构造模拟参数，还有注意各个寄存器的值（用不到的就不管），如 `ecx=0012A1F4`，在基址 `45DEBC` 中存储，它是好找的。
13. 因此，对于参数2和参数5，二者的值，可以在 `ecx` 的基础上加上某个数得到：参数2 + `40`，参数5 + `4`。要特别注意此思路，不管他为什么要加上此数，只要构造出这个值就行。
14. （这样就看出，这个 `ecx` 的值特别重要，作为一个基础，而那个基址中存储的这个 `ecx`，可见，找到合适的基址尤其重要，是构造汇编代码的重中之重，特别注意基址的寻找，有了基址，一切都好办（哪怕同参数2/5一样，强行 `+x` 构造出某个值，只要我能构造出程序当时运行的环境就行））

## 2.5 实现秒杀

1. 就是单次消除功能的循环，设置一个停止条件即可，点坐标的 `x/y == 0`。
2. 关键代码

```
// MFCGamePlugin.cpp

// 循环消除中，判断是否停止

if (pt1.x == 0 && pt1.y == 0)
{
    return -1;
}

//CMyDlg.cpp

void CMyDlg::OnBnClickedButton3()
{
    // TODO: 在此添加控件通知处理程序代码

    CMFCGamePluginApp* pApp = (CMFCGamePluginApp*)AfxGetApp();

    // 循环消除
    for (int i = 0; i < 100; i++)
    {
        int nRet = ::SendMessage(pApp->m_hWnd, WM_DATA2, 0, 0);
        if (nRet == -1)
            break;
    }
}
```

## 2.6 另一个思路来单消/秒杀

1. 不断测试，当点击两个炸弹成功消除后，会出现炸弹道具。
2. 同理，找到相应的 `call`，观察参数，发现同指南针相比，就是 `F0` 换成了 `F4`，二者就是一样的思路来的。
3. 炸弹一次就相当于单次消除，加上循环便是秒杀（此时循环没有加停止条件，仅限制了循环次数，无伤大雅，有那个意思就行）。
4. 版本1的单次消除和秒杀：获取两个点 + 手动消除，本思路借用炸弹道具，明显简单多了，也提了个醒，逆向时，要举一反三，通过指南针道具的调用，联想其他工具，程序员一般都是按照同一个思路来做的，无非换个参数（时间有限，其他道具也是这个道理，学到思路即可）。
5. 关键代码

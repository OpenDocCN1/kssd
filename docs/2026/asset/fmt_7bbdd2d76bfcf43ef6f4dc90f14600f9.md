
## 在 Windows 上，没有[`s2e.so`](https://bbs.kanxue.com/elink@a5aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6e0x3V1g2Q4x3V1k6Y4N6h3g2K6N6q4)9J5k6s2c8G2L8$3I4K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3I4A6L8Y4g2^5i4K6u0r3M7K6u0W2i4K6u0W2M7$3!0Q4x3V1k6K6x3X3g2Q4x3X3g2U0) 的 dll 版本。因此，我们需要换种方式来将符号化的数据注入到恶意软件中。我们可以写个 S2E 插件来实现，但有点复杂。相反，我们 通过 hook Windows API 来注入DLL，并且插入符号化的数据。

## Hook Windows API

有许多不同的技术可以实现 hook Windows API，我们会使用现有的办法而非再去发明一种新的。当我刚开始这项工作时，我想重用[Cuckoo Sandbox](https://bbs.kanxue.com/elink@d9cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6h3y4C8L8$3!0K6j5h3&6V1j5X3!0^5i4K6u0W2L8%4u0Y4i4K6u0r3)的 [Monitor](https://bbs.kanxue.com/elink@fa7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4#2j5$3E0G2L8#2)9J5k6r3#2G2L8X3W2@1L8%4u0Q4x3X3g2J5k6h3q4V1N6r3S2W2k6r3!0U0M7#2)9J5k6h3W2G2i4K6u0r3k6h3&6Q4x3V1k6D9j5i4c8W2M7%4c8Q4x3V1j5`.) 来 实现对API 的hook(它就是为了分析恶意软件而设计的)。然而，后来我们决定用 [EasyHook](https://bbs.kanxue.com/elink@207K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2j5i4y4&6K9r3!0G2K9#2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3) 的办法，主要是因为它在开始阶段需要更少的工作。

在开始深入研究代码前，先看下我们下面要做的：

- `malware-inject`：启动其他程序 (e.g. 恶意软件) ，将DLL 注入 新启动进程的地址空间；
- `malware-hook` ： 通过 `malware-inject` 注入其他进程地址空间的 DLL ，这个 DLL 是 hook Windows API 的关键函数，给我们注入符号化的数据提供了一种机制。

现在让我们深入研究一些代码！

在 Visual Studio 中 打开 `$S2EDIR/source/s2e/guest/windows/s2e.sln` ，创建两个项目：

- `malware-inject`： Win32 控制台应用程序；
- `malware-hook`：Win32 DLL。

这两个项目都需要 EasyHook 的本地包， 通过 [Nuget](https://bbs.kanxue.com/elink@703K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2F1N6h3N6W2N6q4)9J5k6h3!0J5k6#2)9J5c8R3`.`.)  安装。 注意在Github 上的 `malware-hook` 项目分为 `GetLocalTime-hook` 和 `wannacry-hook` 两个项目 (我们的研究实例)。

## `malware-inject`

`malware-inject` 基于 EasyHook 的样例程序 [injector](https://bbs.kanxue.com/elink@022K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2j5i4y4&6K9r3!0G2K9#2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3N6s2g2@1L8%4u0A6j5h3I4K6i4K6u0r3L8X3q4@1K9i4k6W2M7X3g2E0L8%4c8W2K9r3!0G2K9#2)9J5k6h3S2@1L8h3H3`.) 。在这里，我们使用 `RhCreateAndInject` 而非`RhInjectLibrary` (将 DLL 注入已经运行的程序)。 我们使用这个函数启动处于挂起状态的应用程序，注入DLL，然后恢复挂起进程。`malware-inject`还会在返回之前等待注入的进程完成，这非常有用，因为它可以防止当`malware-inject` 进程退出时 S2E 终止状态。

创建 `inject.c` 文件，把下面的代码加进去：

```
#include <stdio.h>

#include <string.h>



#include <Shlwapi.h>

#include <Windows.h>



#include <easyhook.h>



// We must add this header file to support writing to S2E's logs. s2e.h resides

// in the libcommon project, so the libcommon project must be added as a

// dependency to the malware-inject project

#define USER_APP

#include <s2e/s2e.h>



#define S2E_MSG_LEN 512

#define MAX_PATH_LEN 256



static INT s2eVersion = 0;



static void Message(LPCSTR fmt, ...) {

    CHAR message[S2E_MSG_LEN];

    va_list args;



    va_start(args, fmt);

    vsnprintf(message, S2E_MSG_LEN, fmt, args);

    va_end(args);



    if (s2eVersion) {

        S2EMessageFmt("[malware-inject] %s", message);

    } else {

        printf("[malware-inject] %s", message);

    }

}



static void GetFullPath(LPCWSTR path, PWCHAR fullPath) {

    if (!path) {

        Message("Path has not been provided\n");

        exit(1);

    }



    if (!PathFileExistsW(path)) {

        Message("Invalid path %S has been provided\n", path);

        exit(1);

    }



    if (!GetFullPathNameW(path, MAX_PATH_LEN, fullPath, NULL)) {

        Message("Unable to get full path of %S\n", path);

        exit(1);

    }

}



int main() {

    INT argc;

    LPWSTR *argv = CommandLineToArgvW(GetCommandLineW(), &argc);



    if (argc < 5) {

        printf("Usage: %S [options..]\n"

            "   --dll <dll>       Path to DLL to inject into the application\n"

            "   --app <target>    Path to application to start\n"

            "   --timeout <time>  Timeout value in milliseconds "

            "(infinite if not provided)\n", argv[0]);

        exit(1);

    }



    // Used by the Message function to decide where to write output to

    s2eVersion = S2EGetVersion();



    LPWSTR dllPath = NULL;

    WCHAR fullDllPath[MAX_PATH_LEN];



    LPWSTR appPath = NULL;

    WCHAR fullAppPath[MAX_PATH_LEN];



    DWORD timeout = INFINITE;



    for (int i = 1; i < argc; ++i) {

        if (wcscmp(argv[i], L"--dll") == 0) {

            dllPath = argv[++i];

            continue;

        }



        if (wcscmp(argv[i], L"--app") == 0) {

            appPath = argv[++i];

            continue;

        }



        if (wcscmp(argv[i], L"--timeout") == 0) {

            timeout = wcstoul(argv[++i], NULL, 10);

            continue;

        }



        Message("Unsupported argument: %s\n", argv[i]);

        exit(1);

    }



    // Check that the given paths are valid

    GetFullPath(dllPath, fullDllPath);

    GetFullPath(appPath, fullAppPath);



    // Start the target application (in a suspended state) and inject the given

    // DLL

    ULONG pid;

    NTSTATUS result = RhCreateAndInject(appPath, L"", CREATE_SUSPENDED,

        EASYHOOK_INJECT_DEFAULT,

#if defined(_M_IX86)

        dllPath, NULL,

#elif defined(_M_X64)

        NULL, dllPath,

#else

        #error "Platform not supported"

#endif

        NULL, 0, &pid);



    if (FAILED(result)) {

        Message("RhCreateAndInject failed: %S\n", RtlGetLastErrorString());

        exit(1);

    }



    Message("Successfully injected %S into %S (PID=0x%x)\n", fullDllPath,

        fullAppPath, pid);



    DWORD exitCode = 1;



    // Get a handle to the newly-created process and wait for it to terminate.

    // Once the process has terminated, get its return code and return that as

    // our return code

    HANDLE hProcess = OpenProcess(SYNCHRONIZE | PROCESS_QUERY_INFORMATION,

        FALSE, pid);

    if (hProcess) {

        WaitForSingleObject(hProcess, timeout);

        GetExitCodeProcess(hProcess, &exitCode);

        CloseHandle(hProcess);

    } else {

        Message("Unable to open process 0x%x: 0x%X\n", pid, GetLastError());

    }



    return exitCode;

}
```

当然，恶意软件完全可能会监视 API 的 hook（毕竟我们正在处理恶意软件），尽管这是一个重要问题，但在这里不做过多探讨。

既然我们已经编写了可以将 DLL 注入恶意软件的工具，那么让我们把注意力转向这个 DLL 实际要做的事情。

## `malware-hook`

同样 ，我们将基于 EasyHook 的样例 程序 [BeepHook](https://bbs.kanxue.com/elink@200K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2j5i4y4&6K9r3!0G2K9#2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3N6s2g2@1L8%4u0A6j5h3I4K6i4K6u0r3L8X3q4@1K9i4k6W2M7X3g2E0L8%4c8W2K9r3!0G2K9#2)9J5k6h3S2@1L8h3H3`.) 来 实现`malware-hook` ，下面是我们 hook 的 DLL 的框架， 也就是 `malware-hook.cpp` 的内容：

```
#include <Windows.h>

#include <strsafe.h>



#include <easyhook.h>



#define USER_APP

extern "C" {

#include <s2e/s2e.h>

}



#define S2E_MSG_LEN 512



static INT s2eVersion = 0;



static void Message(LPCSTR fmt, ...) {

    CHAR message[S2E_MSG_LEN];

    va_list args;



    va_start(args, fmt);

    vsnprintf(message, S2E_MSG_LEN, fmt, args);

    va_end(args);



    if (s2eVersion) {

        S2EMessageFmt("[0x%x|malware-hook] %s", GetCurrentProcessId(),

            message);

    } else {

        printf("[0x%x|malware-hook] %s", GetCurrentProcessId(), message);

    }

}



// EasyHook will be looking for this export to support DLL injection. If not

// found then DLL injection will fail

extern "C" void __declspec(dllexport) __stdcall NativeInjectionEntryPoint(REMOTE_ENTRY_INFO *);



void __stdcall NativeInjectionEntryPoint(REMOTE_ENTRY_INFO *inRemoteInfo) {

    // Unused

    (void*) inRemoteInfo;



    // Used by the Message function to decide where to write output to

    s2eVersion = S2EGetVersion();



    // TODO initialize hooks



    // The process was started in a suspended state. Wake it up...

    RhWakeUpProcess();

}
```


```
            DebugEvent->u.LoadDll.dwDebugInfoFileOffset =
                WaitStateChange->StateInfo.LoadDll.DebugInfoFileOffset;

            DebugEvent->u.LoadDll.nDebugInfoSize =
                WaitStateChange->StateInfo.LoadDll.DebugInfoSize;

            /* Open the thread */
            InitializeObjectAttributes(&ObjectAttributes, NULL, 0, NULL, NULL);

            Status = NtOpenThread(&ThreadHandle,
                                 THREAD_QUERY_INFORMATION,
                                 &ObjectAttributes,
                                 &WaitStateChange->AppClientId);

            if (NT_SUCCESS(Status))
            {
                /* Query thread information */
                Status = NtQueryInformationThread(ThreadHandle,
                                                  ThreadBasicInformation,
                                                  &ThreadBasicInfo,
                                                  sizeof(ThreadBasicInfo),
                                                  NULL);

                NtClose(ThreadHandle);
            }

            /* Check if we got thread information */
            if (NT_SUCCESS(Status))
            {
                /* Save the image name from the TIB */
                DebugEvent->u.LoadDll.lpImageName =
                    &((PTEB)ThreadBasicInfo.TebBaseAddress)->
                    Tib.ArbitraryUserPointer;
            }
            else
            {
                /* Otherwise, no name */
                DebugEvent->u.LoadDll.lpImageName = NULL;
            }

            /* It's Unicode */
            DebugEvent->u.LoadDll.fUnicode = TRUE;

            break;

        /* DLL Unload */
        case DbgUnloadDllStateChange:

            /* Set Win32 code and DLL Base */
            DebugEvent->dwDebugEventCode = UNLOAD_DLL_DEBUG_EVENT;

            DebugEvent->u.UnloadDll.lpBaseOfDll =
                WaitStateChange->StateInfo.UnloadDll.BaseAddress;

            break;

        /* Anything else, fail */
        default: return STATUS_UNSUCCESSFUL;
    }

    /* Return success */
    return STATUS_SUCCESS;
}
```

Let's take a look at the interesting fixups. First of all, the lack of a `TEB` pointer is easily fixed by calling `NtQueryInformationThread` with the `ThreadBasicInformation` type, which returns, among other things, a pointer to the `TEB`, which is then saved in the Win32 structure. As for Debug Strings, the API analyzes the exception code and looks for `DBG_PRINTEXCEPTION_C`, which has a specific exception record that is parsed and converted into a debug string output.

我们来看看有趣的修正。首先，很容易地修正了缺少指向`TEB`结构的指针，通过调用`NtQueryInformationThread`函数并传递`ThreadBasicInformation`类型参数，在它返回的内容之间，有一个指向`TEB`的指针，然后它被保存在Win32结构中。至于调试字符串，这个API分析了异常代码并寻找`DBG_PRINTEXCEPTION_C`类型，如果匹配，其中有一个特定的异常记录，被解析并转换为一个调试字符串输出。

So far so good, but perhaps the nastiest hack is present in the code for DLL loading. Because a loaded DLL doesn't have a structure like `EPROCESS` or `ETHREAD` in kernel memory, but in `ntdll`'s private `Ldr` structures, the only thing that identifies it is a `Section Object` in memory for its memory mapped file. When the kernel gets a request to create a section for an executable memory mapped file, it saves the name of the file in a field inside the `TEB` (or `TIB`, rather) called `ArbitraryUserPointer`.

到目前为止还挺顺利的，但也许在代码中目前最难理解的就是DLL加载了。因为一个加载的DLL在内核内存中没有一个像`EPROCESS`或`ETHREAD`的结构，但是在`ntdll`私有的`Ldr`结构里，唯一确定在内存中内存映射文件的就是一个区段对象（`Section Object`）。当内核接收到一个请求，为一个可执行的内存映射文件创建一个区段，在`TEB`（或者`TIB`）结构中，称为`ArbitraryUserPointer`的字段，保存这个文件的名称。

This function then knows that a string is located there, and sets it as the pointer for the debug event's `lpImageName` member. This hack has been in NT every since the first builds, and as far as I know, it's still there in Vista. Could it be that hard to solve?

这个函数已经知道字符串位于那里，并将其设置成指针，赋值给调试事件的`lpImageName`成员。自从NT的第一个版本，这个方法就已经存在，据我所知，它仍然存在于Vista中。可能是难以解决的问题？

Once again, we come to an end in our discussion, since there isn't much left in `ntdll` that deals with the Debug Object. Here's an overview of what was discussed in this part of the series:

- `DbgUi` provides a level of separation between the kernel and Win32 or other subsystems. It's written as a fully independent class, even having accessor and mutator methods instead of exposing its handles.
- The handle to a thread's Debug Object is stored in the second field of the `DbgSsReserved` array in the `TEB`.
- `DbgUi` allows a thread to have a single `DebugObject`, but using the native system calls allows you to do as many as you want.
- Most `DbgUi` APIs are simple wrappers around the `NtXxxDebugObject` system calls, and use the `TEB` handle to communicate.
- `DbgUi` is responsible for breaking into the attached process, not the kernel.
- `DbgUi` uses its own structure for debug events, which the kernel understands. In some ways, this structure provides more information about some events (such as the subsystem and whether this was a single step or a breakpoint exception), but in others, some information is missing (such as a pointer to the thread's `TEB` or a separate debug string structure).
- The `TIB` (located inside the `TEB`)'s `ArbitraryPointer` member contains the name of the loaded DLL during a Debug Event.

再次，要结束我们的讨论了，因为没有遗留太多在`ntdll`中关于调试对象（Debug Object）的处理。接下来是对这篇文章所讨论的做个总结：

1. `DbgUi`在内核和Win32或者其他子系统之间提供一层分离。它被设计成为一个完全独立的类，有访问器和突变方法来代替暴露它的具体处理。
2. 一个线程的调试对象句柄存储在`TEB`结构中`DbgSsReserved`数组的第二个字段里。
3. `DbgUi`允许一个线程拥有一个单一的调试对象，但是使用本地系统调用允许你做很多你想要的。
4. 多数`DbgUi` API都只是简单地对`NtXxxDebugObject`系统调用进行包装，且使用`TEB`来处理通信。
5. `DbgUi`负责中断被附加调试的进程，而不是内核负责中断。
6. `DbgUi`使用它自己的结构，针对内核能够理解的调试事件。在某些方面，这种结构提供了关于一些事件的更多的信息（比如子系统，这是否是一个单步或者断点异常），但在其他情况下，缺少某些信息（比如一个指向线程`TEB`结构的指针或者一个单独的调试字符串结构）。
7. 在一个调试事件期间，`TIB`（位于`TEB`中）的`ArbitraryPointer`成员包含了加载的DLL的名称。

系列一Windows用户模式调试内幕的word附件：

 [Windows用户模式调试内幕.rar](https://bbs.kanxue.com/attach-download-89836.htm)

系列二Windows本地（Native）调试内幕的word附件：

 [Windows本地（Native）调试内幕.rar](https://bbs.kanxue.com/attach-download-89835.htm)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [Windows本地（Native）调试内幕.rar](javascript:void(0);)  
  （35.52kb，53次下载）
- [Windows用户模式调试内幕.rar](javascript:void(0);)  
  （43.76kb，54次下载）

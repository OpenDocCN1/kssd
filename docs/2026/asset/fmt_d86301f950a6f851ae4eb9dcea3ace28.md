
```text
01005588   .  E8 31510000   call <jmp.&MFC42u.#5568>

0100558D   >  6A 01         push 1

0100558F   .  8D45 B0       lea eax,dword ptr ss:[ebp-50]

01005592   .  50            push eax

01005593   .  6A FF         push -1

01005595   .  FF75 EC       push dword ptr ss:[ebp-14]

01005598   .  FFB5 3CFFFFFF push dword ptr ss:[ebp-C4]

0100559E   .  FFD7          call edi
```

---

```text
    int buf[13] = {0}; //积分数组为13个元素
    int buf2[13] = {0};

    HWND wnd1;
    HANDLE ID;
    DWORD Pid;
    int i, fen;
    unsigned long len;
    long *m_addr, *m_addr2; //Microsoft 网上红心大战的关键数组地址

    m_addr = (long *)0x100d41c;
    m_addr2 = (long *)0x100D4BC;

    ::memset(buf, 0, 13);
    ::memset(buf2, 0, 13);

    if ((wnd1 = ::FindWindow(NULL, "Microsoft 网上红心大战")) == NULL)
    {
        AfxMessageBox("Microsoft 网上红心大战未运行!");
        return;
    }

    ::GetWindowThreadProcessId(wnd1, &Pid);
    ID = ::OpenProcess(PROCESS_ALL_ACCESS, false, Pid);

    if (ID)
    {
        ::ReadProcessMemory(ID, m_addr, buf, 52, &len); //将Microsoft 网上红心大战的数据读入buf数组中
        ::ReadProcessMemory(ID, m_addr2, buf2, 52, &len);

        for (i = 0; i < 13; i++) //
        {
            buf[i] = 0; //数组的值代表分数
        }

        for (i = 0; i < 13; i++) //
        {
            buf2[i] += 100; //数组的值代表分数
        }

        //::DrawText( const CString& str, LPRECT lpRect, DT_CENTER    );
        ::WriteProcessMemory(ID, m_addr, buf, 52, &len); //将处理过的值再次写入Microsoft 网上红心大战中。
        ::WriteProcessMemory(ID, m_addr2, buf2, 52, &len); //将处理过的值再次写入Microsoft 网上红心大战中。对方一定死翘翘了。

        ::CloseHandle(ID);
        ::BringWindowToTop(wnd1);
    }
```

![](http://bbs.pediy.com/upload/2005/4/image/1.jpg_404.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

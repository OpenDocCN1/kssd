
```c
static int initWebs()

{

    struct hostent    *hp;

    struct in_addr    intaddr;

    char            host[128], dir[128], webdir[128];

    char            *cp;

    char_t            wbuf[128];



/*

 *    Initialize the socket subsystem

 */

    socketOpen();



#ifdef USER_MANAGEMENT_SUPPORT

/*

 *    Initialize the User Management database

 */

    umOpen();

    umRestore(T("umconfig.txt"));

#endif



/*

 *    Define the local Ip address, host name, default home page and the 

 *    root web directory.

 */

    if (gethostname(host, sizeof(host)) < 0) {

        error(E_L, E_LOG, T("Can't get hostname"));

        return -1;

    }

    if ((hp = gethostbyname(host)) == NULL) {

        error(E_L, E_LOG, T("Can't get host address"));

        return -1;

    }

    memcpy((char *) &intaddr, (char *) hp->h_addr_list[0],

        (size_t) hp->h_length);







/*

 *    Set ../web as the root web. Modify this to suit your needs

 */

    getcwd(dir, sizeof(dir)); 

    if ((cp = strrchr(dir, '/'))) {

        *cp = '\0';

    }

    sprintf(webdir, "%s/%s", dir, rootWeb);



/*

 *    Configure the web server options before opening the web server

 */

    websSetDefaultDir(webdir);

    cp = inet_ntoa(intaddr);

    ascToUni(wbuf, cp, min(strlen(cp) + 1, sizeof(wbuf)));

    websSetIpaddr(wbuf);

    ascToUni(wbuf, host, min(strlen(host) + 1, sizeof(wbuf)));

    websSetHost(wbuf);



/*

 *    Configure the web server options before opening the web server

 */

    websSetDefaultPage(T("default.asp"));

    websSetPassword(password);



/* 

 *    Open the web server on the given port. If that port is taken, try

 *    the next sequential port for up to "retries" attempts.

 */

    websOpenServer(port, retries);



/*

 *     First create the URL handlers. Note: handlers are called in sorted order

 *    with the longest path handler examined first. Here we define the security 

 *    handler, forms handler and the default web page handler.

 */

    websUrlHandlerDefine(T(""), NULL, 0, websSecurityHandler, 

        WEBS_HANDLER_FIRST);

    websUrlHandlerDefine(T("/goform"), NULL, 0, websFormHandler, 0);//自定义的路径，可执行特定功能

    websUrlHandlerDefine(T("/cgi-bin"), NULL, 0, websCgiHandler, 0);//执行cgi程序或脚本

    websUrlHandlerDefine(T(""), NULL, 0, websDefaultHandler, 

        WEBS_HANDLER_LAST); //普通地请求web文件



/*

 *    Now define two test procedures. Replace these with your application

 *    relevant ASP script procedures and form functions.

 */

    websAspDefine(T("aspTest"), aspTest);

    websFormDefine(T("formTest"), formTest);



/*

 *    Create the Form handlers for the User Management pages

 */

#ifdef USER_MANAGEMENT_SUPPORT

    formDefineUserMgmt();

#endif



/*

 *    Create a handler for the default home page

 */

    websUrlHandlerDefine(T("/"), NULL, 0, websHomePageHandler, 0); 

    return 0;

}
```

解析URL路径第一层，如果是`goform`就用`websFormHandler`处理，如果是`cgi-bin`就用`websCgiHandler`处理，如果是其他，就是`websDefaultHandler`处理。

对于cgi，编译好源码，把直接把`busybox`放到`/cgi-bin`，然后就“命令注入”了一把。

在实际应用中，如果提取解开了固件，对于cgi程序（也有lua php等），都可以找找命令注入。

可以看到下面的源码，其实最后是起了一个程序。

```
/*  函数websCgiHandler中最后运行的

 *    Now launch the process.  If not successful, do the cleanup of resources.

 *    If successful, the cleanup will be done after the process completes.

 */

    if ((pHandle = websLaunchCgiProc(cgiPath, argp, envp, stdIn, stdOut)) 

        == -1) {

        websError(wp, 200, T("failed to spawn CGI task"));

        for (ep = envp; *ep != NULL; ep++) {

            bfreeSafe(B_L, *ep);

        }

        bfreeSafe(B_L, cgiPath);

        bfreeSafe(B_L, argp);

        bfreeSafe(B_L, envp);

        bfreeSafe(B_L, stdOut);

    } else {

/*
```

```
int websLaunchCgiProc(char_t *cgiPath, char_t **argp, char_t **envp, 

                      char_t *stdIn, char_t *stdOut)

{

    PROCESS_INFORMATION    procinfo;        /*  Information about created proc   */

    DWORD                dwCreateFlags;

    char                *fulldir;

    BOOL                bReturn;

    int                    i, nLen;



/*

 *    Replace directory delimiters with Windows-friendly delimiters

 */

    nLen = gstrlen(cgiPath);

    for (i = 0; i < nLen; i++) {

        if (cgiPath[i] == '/') {

            cgiPath[i] = '\\';

        }

    }



    fulldir = NULL;

    dwCreateFlags = CREATE_NEW_CONSOLE;



/*

 *    CreateProcess returns errors sometimes, even when the process was    

 *    started correctly.  The cause is not evident.  For now: we detect    

 *    an error by checking the value of procinfo.hProcess after the call.  

 */

    procinfo.hThread = NULL;

    bReturn = CreateProcess(//看到这里就明白了

        cgiPath,            /*  Name of executable module        */

        NULL,                /*  Command line string              */

        NULL,                /*  Process security attributes      */

        NULL,                /*  Thread security attributes       */

        0,                    /*  Handle inheritance flag          */

        dwCreateFlags,        /*  Creation flags                   */

        NULL,                /*  New environment block            */

        NULL,                /*  Current directory name           */

        NULL,                /*  STARTUPINFO                      */

        &procinfo);            /*  PROCESS_INFORMATION              */



    if (bReturn == 0) {

        DWORD dw;

        dw = GetLastError();

        return -1;

    } else {

        CloseHandle(procinfo.hThread);

    }

    return (int) procinfo.dwProcessId;

}
```

除了cgi，还有一些其他的自定义的路径，都可以尝试去挖掘漏洞，如之前dlink `/hnap`就爆出漏洞。`361K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3L8%4u0@1K9h3N6#2j5i4u0V1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6U0P5h3y4D9L8%4m8W2k6r3W2S2i4K6u0r3K9i4m8K6i4K6u0r3y4o6l9%4y4K6t1`。

从这里，可以明白两点，一是可以从固件解开得出的`webs`程序中找到有哪些路径可以访问（找未授权访问），二是这些路径对应的函数（用来分析漏洞，完成利用）

以上是我在挖掘ruckus漏洞时总结的一些东西，和大家分享下，大家最好结合源码自己分析，如果你也对路由器漏洞挖掘感兴趣。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [`webs`](javascript:void(0);)（344.43kb，17次下载）

- [`ws031202_ubuntu14.0.tar.gz`](javascript:void(0);)（1.28MB，14次下载）

- [`2825_4.2.0.0.132_Main.Bl7`](javascript:void(0);)（3.61MB，15次下载）

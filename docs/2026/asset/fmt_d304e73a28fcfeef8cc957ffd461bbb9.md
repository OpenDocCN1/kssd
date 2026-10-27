
测试环境为XP SP3，由于系统中安装的软件不尽相同，以上结果可能部分与你的不一致。

## 总结一下：

1. 在NT获取控制权时，`BOOT`类型的驱动已经被加载到内存中（由`ntldr`加载），但是并没有初始化

2. `BOOT`类型驱动初始化和`SYSTEM`类型驱动加载，是由`IoInitSystem`中完成的。此时已经到了内核初始化后期，不过这部分是内核初始化时最耗时间的。

3. 服务和`AUTO`驱动在用户登录之前启动和加载。

4. 每个会话对应一个`win32`子系统。关于会话与`win32`子系统之间的关系，请参见：[白话windows之 会话、工作站、桌面、窗口之间的关系](http://bbs.pediy.com/showthread.php?t=173271)

5. 关键进程创建顺序和父子关系如下：

- `Idle`
- `system`
  - `smss.exe`
    - `autochk.exe`
    - `win32k.sys`
    - `csrss.exe`
    - `winLogin.exe`
      - `services.exe`
      - `lsass.exe`
      - `logonui.exe`
      - `userinit.exe`
        - `explorer.exe`

非第一次用户登录时：

- `smss.exe`
  - `win32k.sys`
  - `csrss.exe`
  - `winlogin.exe`
    - `logonui.exe`
    - `userinit.exe`
      - `explorer.exe`

欢迎各位拍砖~![](https://bbs.kanxue.com/view/img/face/28.gif)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

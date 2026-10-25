
# Bochs+Windbg 调试 VT 代码

> 原文：[`www.pediy.com/kssd/pediy12/139031.html`](https://www.pediy.com/kssd/pediy12/139031.html)

准备工具:  
`Windbg 6.12`  
`bochs-2.4.6-msvc-src` (`http://sourceforge.net/projects/bochs/files/bochs/2.4.6/`)  
`vmware`  
`Securable` (检查是否开启 `VT` 的)  
`VC 2008`

`Intel vt` 技术十分诱人，但是如何调试 `vt` 程序却是让人头疼, 我们经常使用的 `vmware` 貌似不支持 `cpu vt` 技术的模拟, 貌似只有 `bochs` 支持 `cpu vt` 技术的模拟。

看了下 `qemu` 的代码发现其 `feature` 里有支持 `VT` 特性的, 不过试了它提供的几个 `cpu`, 发现只有 `qemu64` (`cpu` 名称) 支持 `VT`.

步骤:

1. 修改 `config.h` 中的

   ```c
   #define BX_SUPPORT_VMX 0
   ```

   改成

   ```c
   #define BX_SUPPORT_VMX 1
   或者
   #define BX_SUPPORT_VMX 2
   ```

2. 编译. 你会发现 `bx_cpu_c::init_VMCS()`, `vmcs_field_offset()` 等 `link error!`

   解决方法:

   将 `cpu\vmcs.cc` 加进工程就可以了

3. 到现在 `bochs` 已经编译出来了, 接下来就是安装系统. `bochs` 太慢, 使用 `vmware` 装把.

   具体步骤参考三寸法师的文章 (`http://bbs.pediy.com/showthread.php?p=964315`).

   装完后用 `Securable` 检测下看是否支持 `vt`. 再加上调试模式(修改 `boot.ini`... 不多说, 不知道的可以搜一下 `vmware+debug` 调试)

   发现绿色的 `YES`, 好了支持了, 不过还有问题, 见下文.

4. `bochs` 支持串口, 用 `Windbg` 连接 `bochs` 进行源码调试吧. 在 `bochs` 的配置文件中加一句

   ```ini
   com1: enabled=1, mode=pipe_server, dev=”\\.\pipe\bochs”
   ```

   然后运行虚拟机, 会发现 `bochs` 会一直在等待, 这时打开 `windbg` 连接 `\\.\pipe\bochs` . 嗯, `bochs` 开始运行了. 选择调试模式, 好吧. `windbg` 直接退了. 有木有. 有木有. 如果是 `windbg 6.09` 你就发现他不会退出而是没啥反应.

5. 看了下 `bochs` 中串口的实现, 是同步的(难道是因为这个退出的?), 翻了下 `virtualbox` 的串口的代码发现确实异步, 开了读写线程。好吧，看到网上有用 `bochs` 调试 `wrk` 的一篇文章 `<wrk 源码分析之 WinDbg+Bochs 调试>`, 里面给出了段代码(注: 这个代码我改了下发现这段代码在想串口写数据的时候锁死了, 修改及其原版代码见附件) 用这个试试.

   将其代码加到项目文件中. 然后在 `serial.h` 中的 `bx_serial_t` 定义中添加一变量(红色字体部分):

   ```c
   int io_mode;
     int tty_id;
     SOCKET socket_id;
     FILE *output;
   #ifdef WIN32
     HANDLE pipe;
     void* p_cls_our;
   #endif

   #if USE_RAW_SERIAL
     serial_raw* raw;
   #endif
   ```

   然后在 `serial.cc` 文件中的 `bx_serial_c::init(void)` 添加(红色部分) 这里我们添加了一个模式 "pipe" 具体设置见后面:

   ```c
   略…
   if (server) {
     pipe = CreateNamedPipe( dev,
       PIPE_ACCESS_DUPLEX | FILE_FLAG_FIRST_PIPE_INSTANCE,
       PIPE_TYPE_BYTE | PIPE_READMODE_BYTE | PIPE_WAIT,
       1, 4096, 4096, 0, NULL);
     if (pipe == INVALID_HANDLE_VALUE)
       BX_PANIC(("com%d: CreateNamedPipe(%s) failed", i+1, dev));
     BX_INFO(("com%d: waiting for client to connect to %s", i+1, dev));
     if (!ConnectNamedPipe(pipe, NULL) && GetLastError() != ERROR_PIPE_CONNECTED)
     {
       CloseHandle(pipe);
       pipe = INVALID_HANDLE_VALUE;
       BX_PANIC(("com%d: ConnectNamedPipe(%s) failed", i+1, dev));
     }
     }
     // client mode
     else {

      if ( strcmp(mode, "pipe") == 0 ){
       BX_INFO(("!!!!!!into our code!!!!!!!"));
       serial_pipe* our_serial = new serial_pipe(dev);
       BX_SER_THIS s[i].p_cls_our = our_serial;
      }else{
       pipe = CreateFile( dev,
         GENERIC_READ | GENERIC_WRITE,
         0, NULL, OPEN_EXISTING, 0, NULL);

       if (pipe == INVALID_HANDLE_VALUE)
         BX_INFO(("com%d: failed to open pipe %s", i+1, dev));
      }
     }
   略…
   ```

   在 `serial.cc` 文件中 `void bx_serial_c::rx_timer(void)` 函数添加(红色部分):

   ```c
   略…
   #ifdef WIN32
           DWORD avail = 0;
           if (BX_SER_THIS s[port].pipe &&
               PeekNamedPipe(BX_SER_THIS s[port].pipe, NULL, 0, NULL, &avail, NULL) &&
               avail > 0) {
             ReadFile(BX_SER_THIS s[port].pipe, &chbuf, 1, &avail, NULL);
             data_ready = 1;
           }
           if (BX_SER_THIS s[port].p_cls_our){
               chbuf = ((serial_pipe*)(BX_SER_THIS s[port].p_cls_our))->receive();
               data_ready = 1;
           }
   #endif
   略…
   ```

   再在 `serial.cc` 文件中 `void bx_serial_c::tx_timer(void)` 函数添加(红色部分):

   ```c
   略…
   #ifdef WIN32
           if (BX_SER_THIS s[port].pipe) {
             DWORD written;
             WriteFile(BX_SER_THIS s[port].pipe, (bx_ptr_t)& BX_SER_THIS s[port].tsrbuffer, 1, &written, NULL);
           }

           if (BX_SER_THIS s[port].p_cls_our) {
               serial_pipe* p_serial = (serial_pipe*)(BX_SER_THIS s[port].p_cls_our);
               p_serial->transmit(BX_SER_THIS s[port].tsrbuffer);
           }
   #endif
   略…
   ```

   再在头部添加新添文件的头文件:

   ```c
   #if USE_PIPE_SERIAL
   #include "serial_pipe.h"
   #endif
   ```

   好吧还得在 `config.h` 中添加:

   ```c
   #define USE_PIPE_SERIAL 1
   ```

6. 大功告成, 编译. 好! 成功.

7. 修改 `bochs` 配置文件:

   ```ini
   com1: enabled=1, mode=pipe, dev="pipe\bochs"
   ```

8. 发现 `windbg` 连上了, 并且可以源码调试了. 如图:

   ![](img/c918ba4d07e927cd28402de39c916a2f.jpg)

   这是啥? `newbluepill`???

9. 到这就完了嘛? `no`.. `windbg` 输入 `g` `enter`.. 草.. 蓝了, 断在 `vmx_on` 指令上.. 不支持?

   发现 `bochs` 后台输出了句: `#GP: VMXON is not allowed !`. 抬头对天大喊一声, 草!!. 明明是支持的呀!

   跟了下发现是:

   `!(BX_CPU_THIS_PTR msr.ia32_feature_ctrl & BX_IA32_FEATURE_CONTROL_LOCK_BIT)`

   引起的. 查找了下 `BX_IA32_FEATURE_CONTROL_LOCK_BIT` 的引用. 发现:

   ```c
   /* enable VMX, should be done in BIOS instead */
   BX_CPU_THIS_PTR msr.ia32_feature_ctrl =
   /*BX_IA32_FEATURE_CONTROL_LOCK_BIT|*/ BX_IA32_FEATURE_CONTROL_VMX_ENABLE_BIT;
   ```

   哦. 知道了原来在 `bios` 里设置. 可是 `bochs` 那丑陋的 `bios` 根本就不能设置, 只能设置从哪启动, 一不做二不休, 干脆把注释去掉:

   ```c
   /* enable VMX, should be done in BIOS instead */
   BX_CPU_THIS_PTR msr.ia32_feature_ctrl =
   BX_IA32_FEATURE_CONTROL_LOCK_BIT | BX_IA32_FEATURE_CONTROL_VMX_ENABLE_BIT;
   ```

   再跑 `ok` 了. 可以运行了. 如图:

   ![](img/7251c406abc87dd17b50b92b78b6f867.jpg)

   ![](img/2eb8941277740c8ab3a2ded3fc5ecc3f.jpg)

   这又是啥??

   `PS`: `bochs` 的时钟频率很快, 所以其串口速度也非常快. 比 `VirtualKD` 都看. 至于你你信不信, 反正我是信了.

修改过的代码:

`bochs-pipe-serial.rar`

`doc` 文档:

`Bochs 调试 VT 代码.doc`

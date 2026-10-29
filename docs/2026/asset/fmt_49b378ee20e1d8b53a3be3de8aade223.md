
# [原创] Frida 常用检测点及其原理——一把梭方案

## Frida 常见反调试

> 查看哪个 `so` 在检测 Frida。

```javascript
function hook_dlopen() {
  Interceptor.attach(Module.findExportByName(null, "android_dlopen_ext"), {
    onEnter: function (args) {
      var pathptr = args[0];
      if (pathptr !== undefined && pathptr != null) {
        var path = ptr(pathptr).readCString();
        console.log("load " + path);
      }
    }
  });
}
```

- `so` 的加载流程：`linker` 会先对 `so` 进行加载与链接，然后调用 `so` 的 `.init_proc` 函数，接着调用 `.init_array` 中的函数，最后才是 `JNI_OnLoad` 函数，所以需要先确定检测点大概在哪个函数中。

> 管他检测什么，直接一把梭。

```javascript
function replace_str() {
  var pt_strstr = Module.findExportByName("libc.so", "strstr");
  var pt_strcmp = Module.findExportByName("libc.so", "strcmp");

  Interceptor.attach(pt_strstr, {
    onEnter: function (args) {
      var str1 = args[0].readCString();
      var str2 = args[1].readCString();

      if (
        str2.indexOf("REJECT") !== -1 ||
        str2.indexOf("tmp") !== -1 ||
        str2.indexOf("frida") !== -1 ||
        str2.indexOf("gum-js-loop") !== -1 ||
        str2.indexOf("gmain") !== -1 ||
        str2.indexOf("linjector") !== -1
      ) {
        console.log("strstr-->", str1, str2);
        this.hook = true;
      }
    },
    onLeave: function (retval) {
      if (this.hook) {
        retval.replace(0);
      }
    }
  });

  Interceptor.attach(pt_strcmp, {
    onEnter: function (args) {
      var str1 = args[0].readCString();
      var str2 = args[1].readCString();

      if (
        str2.indexOf("REJECT") !== -1 ||
        str2.indexOf("tmp") !== -1 ||
        str2.indexOf("frida") !== -1 ||
        str2.indexOf("gum-js-loop") !== -1 ||
        str2.indexOf("gmain") !== -1 ||
        str2.indexOf("linjector") !== -1
      ) {
        // console.log("strcmp-->", str1, str2);
        this.hook = true;
      }
    },
    onLeave: function (retval) {
      if (this.hook) {
        retval.replace(0);
      }
    }
  });
}

replace_str();
```

- 测试 `xhs`、`bilibili` 都可以过。

## 1. 检测文件名、端口名和双进程保护

- 检测文件名，可通过改名绕过。
- 检测端口名 `27042`，可通过改端口绕过。
- 双进程保护，可通过 `spawn` 启动。

## 2. 检测 D-Bus

- 检测的原因是因为安卓系统中，D-Bus 通信并不常见，而且不同于在传统 Linux 系统中广泛使用，安卓系统使用 Binder 机制来实现进程间通信（IPC），而不是使用 D-Bus。
- D-Bus 是一种进程间通信（IPC）和远程过程调用（RPC）机制，最初是为 Linux 开发的，目的是用一个统一的协议替代现有的和竞争的 IPC 解决方案。

```c
遍历连接手机所有端口发送 D-Bus 消息，如果返回 "REJECT" 这个特征则认为存在 frida-server。
内存中存在 frida rpc 字符串，认为有 frida-server。

/*
 * Mini-portscan to detect frida-server on any local port.
 */
for (i = 0; i <= 65535; i++) {
  sock = socket(AF_INET, SOCK_STREAM, 0);
  sa.sin_port = htons(i);

  if (connect(sock, (struct sockaddr *)&sa, sizeof sa) != -1) {
    __android_log_print(
      ANDROID_LOG_VERBOSE,
      APPNAME,
      "FRIDA DETECTION [1]: Open Port: %d",
      i
    );

    memset(res, 0, 7);

    // send a D-Bus AUTH message. Expected answer is “REJECT"
    send(sock, "\x00", 1, NULL);
    send(sock, "AUTH\r\n", 6, NULL);
    usleep(100);

    if (ret = recv(sock, res, 6, MSG_DONTWAIT) != -1) {
      if (strcmp(res, "REJECT") == 0) {
        /* Frida server detected. Do something… */
      }
    }
  }

  close(sock);
}
```

- 检测 D-Bus 可以通过 hook 系统库函数，比如 `strstr`、`strcmp` 等等。

```javascript
function replace_str() {
  var pt_strstr = Module.findExportByName("libc.so", "strstr");
  var pt_strcmp = Module.findExportByName("libc.so", "strcmp");

  Interceptor.attach(pt_strstr, {
    onEnter: function (args) {
      var str1 = args[0].readCString();
      var str2 = args[1].readCString();

      if (str2.indexOf("REJECT") !== -1) {
        // console.log("strcmp-->", str1, str2);
        this.hook = true;
      }
    },
    onLeave: function (retval) {
      if (this.hook) {
        retval.replace(0);
      }
    }
  });

  Interceptor.attach(pt_strcmp, {
    onEnter: function (args) {
      var str1 = args[0].readCString();
      var str2 = args[1].readCString();

      if (str2.indexOf("REJECT") !== -1) {
        // console.log("strcmp-->", str1, str2);
        this.hook = true;
      }
    },
    onLeave: function (retval) {
      if (this.hook) {
        retval.replace(0);
      }
    }
  });
}

replace_str();
```

## 3. 检测 `/proc/pid/maps` 依赖文件

- `maps` 文件中存储的是 APP 运行时加载的依赖。
- 当启动 Frida 后，在 `maps` 文件中就会存在 `frida-agent-64.so`、`frida-agent-32.so` 文件。

```c
char line[512];
FILE *fp;

fp = fopen("/proc/self/maps", "r");
if (fp) {
  while (fgets(line, 512, fp)) {
    if (strstr(line, "frida")) {
      /* Evil library is loaded. Do something… */
    }
  }

  fclose(fp);
} else {
  /* Error opening /proc/self/maps. If this happens, something is off. */
}
```

### 绕过

```javascript
function replace_str_maps() {
  var pt_strstr = Module.findExportByName("libc.so", "strstr");
  var pt_strcmp = Module.findExportByName("libc.so", "strcmp");

  Interceptor.attach(pt_strstr, {
    onEnter: function (args) {
      var str1 = args[0].readCString();
      var str2 = args[1].readCString();

      if (
        str2.indexOf("REJECT") !== -1 ||
        str2.indexOf("frida") !== -1
      ) {
        this.hook = true;
      }
    },
    onLeave: function (retval) {
      if (this.hook) {
        retval.replace(0);
      }
    }
  });

  Interceptor.attach(pt_strcmp, {
    onEnter: function (args) {
      var str1 = args[0].readCString();
      var str2 = args[1].readCString();

      if (
        str2.indexOf("REJECT") !== -1 ||
        str2.indexOf("frida") !== -1
      ) {
        this.hook = true;
      }
    },
    onLeave: function (retval) {
      if (this.hook) {
        retval.replace(0);
      }
    }
  });
}

replace_str();
```

## 4. 检测 `/proc/pid/task` 下的线程和 `fd` 目录下的打开文件

- 在 `/proc/pid/task` 目录下，可以通过查看不同的线程子目录，来获取进程中每个线程的运行时信息。这些信息包括线程的状态、线程的寄存器内容、线程占用的 CPU 时间、线程的堆栈信息等。通过这些信息，可以实时观察和监控进程中每个线程的运行状态，帮助进行调试、性能优化和问题排查等工作。
- `/proc/pid/fd` 目录的作用在于提供一种方便的方式来查看进程的文件描述符信息，这对于调试和监控进程非常有用。通过查看文件描述符信息，可以了解进程打开了哪些文件、网络连接等，帮助开发者和系统管理员进行问题排查和分析工作。
- 打开 Frida 调试后，`task` 目录下会多出几个线程，检测点在于查看这些多出来的线程是否和 Frida 调试相关。
- 在某些 APP 中会去读取 `/proc/stask/线程ID/status` 文件，如果是运行 Frida 产生的，则进行反调试。例如：`gmain`、`gdbus`、`gum-js-loop`、`pool-frida` 等。

1. `gmain`：Frida 使用 Glib 库，其中的主事件循环被称为 `GMainLoop`。在 Frida 中，`gmain` 表示 `GMainLoop` 的线程。
2. `gdbus`：`GDBus` 是 Glib 提供的一个用于 D-Bus 通信的库。在 Frida 中，`gdbus` 表示 `GDBus` 相关的线程。
3. `gum-js-loop`：Gum 是 Frida 的运行时引擎，用于执行注入的 JavaScript 代码。`gum-js-loop` 表示 Gum 引擎执行 JavaScript 代码的线程。
4. `pool-frida`：Frida 中的某些功能可能会使用线程池来处理任务，`pool-frida` 表示 Frida 中的线程池。
5. `linjector` 是一种用于 Android 设备的开源工具，它允许用户在运行时向 Android 应用程序注入动态链接库（DLL）文件。通过注入 DLL 文件，用户可以修改应用程序的行为、调试应用程序、监视函数调用等，这在逆向工程、安全研究和动态分析中是非常有用的。

### 绕过

```javascript
function replace_str() {
  var pt_strstr = Module.findExportByName("libc.so", "strstr");
  var pt_strcmp = Module.findExportByName("libc.so", "strcmp");

  Interceptor.attach(pt_strstr, {
    onEnter: function (args) {
      var str1 = args[0].readCString();
      var str2 = args[1].readCString();

      if (
        str2.indexOf("tmp") !== -1 ||
        str2.indexOf("frida") !== -1 ||
        str2.indexOf("gum-js-loop") !== -1 ||
        str2.indexOf("gmain") !== -1 ||
        str2.indexOf("gdbus") !== -1 ||
        str2.indexOf("pool-frida") !== -1 ||
        str2.indexOf("linjector") !== -1
      ) {
        // console.log("strcmp-->", str1, str2);
        this.hook = true;
      }
    },
    onLeave: function (retval) {
      if (this.hook) {
        retval.replace(0);
      }
    }
  });

  Interceptor.attach(pt_strcmp, {
    onEnter: function (args) {
      var str1 = args[0].readCString();
      var str2 = args[1].readCString();

      if (
        str2.indexOf("tmp") !== -1 ||
        str2.indexOf("frida") !== -1 ||
        str2.indexOf("gum-js-loop") !== -1 ||
        str2.indexOf("gmain") !== -1 ||
        str2.indexOf("gdbus") !== -1 ||
        str2.indexOf("pool-frida") !== -1 ||
        str2.indexOf("linjector") !== -1
      ) {
        // console.log("strcmp-->", str1, str2);
        this.hook = true;
      }
    },
    onLeave: function (retval) {
      if (this.hook) {
        retval.replace(0);
      }
    }
  });
}

replace_str();
```

## 5. `frida-server` 启动

- 当 `frida-server` 启动时，在 `/data/local/tmp/` 目录下会生成一个文件夹，并且会生成一些带有 Frida 特征的内容。
- 重新编译去特征。
- hook。

## 6. 直接调用 `openat` 的 `syscall` 检测

直接在 `text` 节表中搜索 `frida-gadget.so`、`frida-agent.so` 字符串，避免通过 hook `libc` 进行 anti-anti。

### 注入方式改为

1. `frida-inject`
2. `ptrace` 注入，以配置文件的形式注入。
3. 编译 ROM 注入。

## 7. 从 inline hook 角度检测 Frida

- <https://bbs.kanxue.com/thread-269862.htm>：未学习。

<mark class="encrypted">233K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0f1#2y4K6M7I4x3K6l9I4y4R3`.`.</mark>

## 8. hook `open` 函数查看检测 SO 文件的位置

```javascript
function hook_open() {
  var pth = Module.findExportByName(null, "open");

  Interceptor.attach(ptr(pth), {
    onEnter: function (args) {
      this.filename = args[0];
      console.log("", this.filename.readCString());

      if (this.filename.readCString().indexOf(".so") !== -1) {
        args[0] = ptr(0);
      }
    },
    onLeave: function (retval) {
      return retval;
    }
  });
}

setImmediate(hook_open);
```

替换一个正常的。

## Frida 常见反调试

> 查看哪个 `so` 在检测 Frida。

|  |  |
| --- | --- |

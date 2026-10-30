
`func (c *linuxContainer) newInitProcess(p *Process, cmd *exec.Cmd, messageSockPair, logFilePair filePair) (*initProcess, error) {`
`cmd.Env = append(cmd.Env, "_LIBCONTAINER_INITTYPE="+string(initStandard))`
`nsMaps := make(map[configs.NamespaceType]string)`
`for _, ns := range c.config.Namespaces {`
`if ns.Path != "" {`
`nsMaps[ns.Type] = ns.Path`
`}`
`}`
`_, sharePidns := nsMaps[configs.NEWPID]`
`data, err := c.bootstrapData(c.config.Namespaces.CloneFlags(), nsMaps, initStandard)`
`if err != nil {`
`return nil, err`
`}`

`if c.shouldSendMountSources() {`
`// Elements on this slice will be paired with mounts (see StartInitialization()`
`// and prepareRootfs()). This slice MUST have the same size as c.config.Mounts.`
`mountFds := make([]int, len(c.config.Mounts))`
`for i, m := range c.config.Mounts {`
`if !m.IsBind() {`
`// Non bind-mounts do not use an fd.`
`mountFds[i] = -1`
`continue`
`}`

`// The fd passed here will not be used: nsexec.c will overwrite it with dup3(). We just need`
`// to allocate a fd so that we know the number to pass in the environment variable. The fd`
`// must not be closed before cmd.Start(), so we reuse messageSockPair.child because the`
`// lifecycle of that fd is already taken care of.`
`cmd.ExtraFiles = append(cmd.ExtraFiles, messageSockPair.child)`
`mountFds[i] = stdioFdCount + len(cmd.ExtraFiles) - 1`
`}`

`mountFdsJson, err := json.Marshal(mountFds)`
`if err != nil {`
`return nil, fmt.Errorf("Error creating _LIBCONTAINER_MOUNT_FDS: %w", err)`
`}`

`cmd.Env = append(cmd.Env,`
`"_LIBCONTAINER_MOUNT_FDS="+string(mountFdsJson),`
`)`
`}`

`init := &initProcess{`
`cmd: cmd,`
`messageSockPair: messageSockPair,`
`logFilePair: logFilePair,`
`manager: c.cgroupManager,`
`intelRdtManager: c.intelRdtManager,`
`config: c.newInitConfig(p),`
`container: c,`
`process: p,`
`bootstrapData: data,`
`sharePidns: sharePidns,`
`}`
`c.initProcess = init`
`return init, nil`
`}`
在该函数中首先设置 `standard` 环境变量，然后从 `config.json` 里面读取需要新建的 `namespaces`，并将这些数据进行存储，然后创建 `initProcess` 结构体，中间的 `shouldSendMountSources` 不用特别关心，它其实是为了挂载一些目录所设置的。到此为止，`parentProcess` 结构体就基本设置完成了。

在 `start` 方法中接下来调用了 `parentProcess` 的 `start()` 函数，这里其实是 `initProcess` 结构体实现的 `start` 函数。在该 `start` 函数中会启动之前设置的 `/proc/self/exe` 进程，参数为 `init`，然后给父进程设置了 `cgroup`，之后通过 `sock` 把信息传输给子进程，这里最关键的其实是启动了 `runC init` 这样一个子进程，因为创建的容器可能具备新的 `namespaces`，因此，通过子进程执行 `runC init` 的时候可以很方便的通过 `setns()` 完成命名空间的切换，同时 `setns` 其实是不运行在多线程条件下使用的，但是 `go runtime` 就是多线程的，因此必须在 `go runtime` 之前设置命名空间，因此使用 `cgo` 在 `go runtime` 启动之前使用 `c` 代码设置命名空间。

在 `cgo` 中，首先利用环境变量拿到了 `pipe`，然后以 `netlink msg` 的格式读取父进程发送的 `config` 配置信息，接着同样执行了创建 `sock` 组的操作，这是为了使得它和孙进程之间可以相互通信，接着以状态机的形式用 `clone` 创建出符合 `config.json` 中设置的命名空间的进程，然后本来的子进程就 `exit(0)` 销毁。

接着回到 `create` 中，在执行 `init` 进程之后对其进行了 `cgroup` 的限制，这也方便在接下来的过程中防止子进程通过 `cgroup` 进行逃逸，接着父进程发送 `bootstrapData` 数据到 `init` 进程，之后 `create` 拿到 `init` 创建的子进程的 `pid`，然后通过 `pipe` 管拿到子进程打开的 `fd` 进行保存，在进行一系列的设置之后通过 `sendConfig` 发送 `config.json` 中的要执行的进程的信息，接下来就是容器初始化和执行 `config.json` 中设置的进程了，具体的过程可以参考 `standard_init_linux.go` 中 `linuxStandardInit` 的 `Init` 函数，到此为止一个容器的大致启动过程就基本分析结束了。

> 参考链接：
>
> 84fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3N6E0k6h3&6@1k6X3q4#2L8s2c8Q4x3X3g2U0L8$3#2Q4x3V1k6S2i4K6u0r3x3e0p5&6x3o6l9H3x3o6l9I4y4K6f1%4y4U0x3I4y4q4)9J5x3$3W2@1k6h3#2Q4x3X3b7I4
>
> 2f3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2M7r3g2F1j5$3!0F1N6r3q4A6L8X3g2J5M7#2)9J5c8Y4u0#2L8X3x3

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

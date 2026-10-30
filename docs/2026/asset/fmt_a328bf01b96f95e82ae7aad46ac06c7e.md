
`Factory` 接口定义如下：

```go
type Factory interface {
    // Creates a new container with the given id and starts the initial process inside it.
    // id must be a string containing only letters, digits and underscores and must contain
    // between 1 and 1024 characters, inclusive.
    //
    // The id must not already be in use by an existing container. Containers created using
    // a factory with the same path (and filesystem) must have distinct ids.
    //
    // Returns the new container with a running process.
    //
    // On error, any partially created container parts are cleaned up (the operation is atomic).
    Create(id string, config *configs.Config) (Container, error)

    // Load takes an ID for an existing container and returns the container information
    // from the state. This presents a read only view of the container.
    Load(id string) (Container, error)

    // StartInitialization is an internal API to libcontainer used during the reexec of the container.
    StartInitialization() error

    // Type returns info string about factory type (e.g. lxc, libcontainer...)
    Type() string
}
```

其中也有对应 Linux 平台的一个实现：

```go
// LinuxFactory implements the default factory interface for linux based systems.
type LinuxFactory struct {
    // Root directory for the factory to store state.
    Root string

    // InitPath is the path for calling the init responsibilities for spawning
    // a container.
    InitPath string

    // InitArgs are arguments for calling the init responsibilities for spawning
    // a container.
    InitArgs []string

    // CriuPath is the path to the criu binary used for checkpoint and restore of
    // containers.
    CriuPath string

    // New{u,g}idmapPath is the path to the binaries used for mapping with
    // rootless containers.
    NewuidmapPath string
    NewgidmapPath string

    // Validator provides validation to container configurations.
    Validator validate.Validator

    // NewIntelRdtManager returns an initialized Intel RDT manager for a single container.
    NewIntelRdtManager func(config *configs.Config, id string, path string) intelrdt.Manager
}
```

Linux Factory 中的 `create` 的具体实现其实就是创建一个 `LinuxContainer`（这正和我们之前所说的 Linux 下的 container 接口相对应）：

```go
type linuxContainer struct {
    id                   string
    root                 string
    config *configs.Config
    cgroupManager        cgroups.Manager
    intelRdtManager      intelrdt.Manager
    initPath             string
    initArgs             []string
    initProcess          parentProcess
    initProcessStartTime uint64
    criuPath             string
    newuidmapPath        string
    newgidmapPath        string
    m                    sync.Mutex
    criuVersion int
    state                containerState
    created              time.Time
    fifo *os.File
}
```

```go
func createContainer(context *cli.Context, id string, spec *specs.Spec) (libcontainer.Container, error) {
    rootlessCg, err := shouldUseRootlessCgroupManager(context)
    if err != nil {
        return nil, err
    }
    config, err := specconv.CreateLibcontainerConfig(&specconv.CreateOpts{
        CgroupName: id,
        UseSystemdCgroup: context.GlobalBool("systemd-cgroup"),
        NoPivotRoot:      context.Bool("no-pivot"),
        NoNewKeyring:     context.Bool("no-new-keyring"),
        Spec:             spec,
        RootlessEUID:     os.Geteuid() != 0,
        RootlessCgroups:  rootlessCg,
    })
    if err != nil {
        return nil, err
    }

    factory, err := loadFactory(context)
    if err != nil {
        return nil, err
    }
    return factory.Create(id, config)
}
```

可以看到，首先加载配置 `config`，然后使用 `loadFactory` 创建相关的 `LinuxFactory`，最终调用了 `factory.Create(id, config)`，然后由 `factory.Create(id, config)` 返回一个 `LinuxContainer`。其中 `LoadFactory` 十分关键，他在最后调用了 `libcontainer.New()` 函数来返回 `LinuxContainer`，在该 `New` 函数里面其设置了 `InitPath`（`InitPath` 非常重要）：

```go
// New returns a linux based container factory based in the root directory and
// configures the factory with the provided option funcs.
func New(root string, options ...func(*LinuxFactory) error) (Factory, error) {
    if root != "" {
        if err := os.MkdirAll(root, 0o700); err != nil {
            return nil, err
        }
    }
    l := &LinuxFactory{
        Root:      root,
        InitPath: "/proc/self/exe",
        InitArgs:  []string{os.Args[0], "init"},
        Validator: validate.New(),
        CriuPath: "criu",
    }

    for _, opt := range options {
        if opt == nil {
            continue
        }
        if err := opt(l); err != nil {
            return nil, err
        }
    }
    return l, nil
}
```

在 `LinuxFactory` 的 `Create` 过程中 `InitPath` 和 `InitArgs` 被传递给 `linuxContainer`。在知道是如何创建出一个 `linuxContainer` 之后，我们把目光返回到 `startContainer`，该函数最后生成了 `runner` 结构体，然后调用了其 `run` 方法，参数为 `spec.Process`，这里的 `spec.Process` 其实就是当初 `config.json` 里面的进程信息。

在 `run` 方法中，一方面通过 `newProcess` 以 `config.json` 为模板创建了 `libcontainer.Process` 结构体，与进程相关的 `limt` 和 `Capabilities` 等设置都在此时完成，另一方面主要根据 `action` 做了三种操作：

```go
switch r.action {
case CT_ACT_CREATE:
    err = r.container.Start(process)
case CT_ACT_RESTORE:
    err = r.container.Restore(process, r.criuOpts)
case CT_ACT_RUN:
    err = r.container.Run(process)
default:
    panic("Unknown action")
}
```

`Process` 结构体，其中大部分的内容都来自 `config.json` 文件：

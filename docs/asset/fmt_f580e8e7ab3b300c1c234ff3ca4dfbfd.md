
### 01004545
```
01004540   |.  C745 F0 06000000  mov dword ptr ss:[ebp-10],6               ; |
01004547   |.  897D D4           mov dword ptr ss:[ebp-2C],edi             ; |
0100454A   |.  897D DC           mov dword ptr ss:[ebp-24],edi             ; |
0100454D   |.  897D E0           mov dword ptr ss:[ebp-20],edi             ; |
01004550   |.  FF15 D0110001     call dword ptr ds:[<&USER32.RegisterClassExW>] ; \RegisterClassExW
```

我想这个结构在 windows 编程中是很重要的一部分，那个 `WNDCLASSEx` 结构看看 msdn 就更清楚了。

### 消息处理过程
```
01003421   \.  C2 0800           retn 8

01003424       CC                int3-------------------将从这里开始消息处理，只是一个简单的 call
01003425       CC                int3
01003426       CC                int3
01003427       CC                int3
01003428       CC                int3
01003429       8BFF              mov edi,edi-------------消息处理过程是从这里开始的

0100342B   /.  55                push ebp
0100342C   |.  8BEC              mov ebp,esp
0100342E   |.  51                push ecx
0100342F   |.  51                push ecx
01003430   |.  56                push esi
01003431   |.  8B75 0C           mov esi,dword ptr ss:[ebp+C]
01003434   |.  83FE 1C           cmp esi,1C                           ;  Switch (cases 2..8001)
```

利用 5 个 `int3` 和 `mov edi,edi` 共 7 个字节，改写一个 `call` 指令。

#### 关于 `mov edi,edi`
> `MOV EDI,EDI` is indeed a 2-byte no-op that is there to enable hot-patching. It enables the application of a hot-fix to a function without a need for a reboot, or even a restart of a running application. Instead, at runtime, the 2-byte NOP is replaced by a short jump to a long jump instruction that jumps to the hot-fix function. A 2-byte instruction is required so that when patching the instruction pointer will not point in a middle of an instruction.

### 修改步骤
`call dword ptr [1013018]`（共六个，后面不一个 `nop`）

然后修改 `01004539` 处的赋值 `mov dword ptr ss:[ebp-28],NOTEPAD.01003429`（改成 `01003424`）

用 `LordPE` 查看 `17521` 的 `plug.dll`

发现只导入一个 `Plug` 含数

现在在要改的版本加入/调用接口：`00024032`（以你的机子而定）

以 `peansen` 的：记事本功能增加方案即可

现在这个记事本真正的好用多了/现在正与楼主谈给记事本加个自动隐藏功能呢/


# CUDA C++ Programming Guide：[de5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8Y4k6A6k6r3W2S2i4K6u0W2j5$3!0E0i4K6u0r3j5%4g2V1j5g2)9J5c8X3y4#2k6r3q4Q4x3X3c8U0i4K6u0V1M7s2u0G2k6%4u0S2L8h3#2A6L8X3N6Q4x3X3c8Y4N6h3W2V1k6g2)9J5c8X3W2F1k6r3g2^5i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@1b4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8Y4k6A6k6r3W2S2i4K6u0W2j5$3!0E0i4K6u0r3j5%4g2V1j5g2)9J5c8X3y4#2k6r3q4Q4x3X3c8U0i4K6u0V1M7s2u0G2k6%4u0S2L8h3#2A6L8X3N6Q4x3X3c8Y4N6h3W2V1k6g2)9J5c8X3W2F1k6r3g2^5i4K6u0W2K9s2c8E0L8l9`.`.)

## 使用 GPU 的好处

图形处理单元 `GPU` 在类似的价格和功率范围内提供比 `CPU` 更高的指令吞吐量和内存带宽。许多应用程序利用这些更高的功能在 `GPU` 上比在 `CPU` 上运行得更快。其他计算设备，如 `FPGA`，也非常节能，但提供的编程灵活性远低于 `GPU`。

`GPU` 和 `CPU` 之间存在这种功能差异，因为它们在设计时考虑了不同的目标。虽然 `CPU` 旨在以尽可能快的速度执行一系列操作（称为线程）并且可以并行执行几十个这样的线程，但 `GPU` 旨在擅长并行执行数千个线程（分摊较慢的单线程性能以实现更大的吞吐量）。

`GPU` 专门用于高度并行计算，因此设计为更多晶体管专用于数据处理，而不是数据缓存和流量控制。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/921830_NH98E8M24KHAUM6.png)

将更多的晶体管用于数据处理，例如浮点计算，有利于高度并行计算；`GPU` 可以通过计算隐藏内存访问延迟，而不是依赖大型数据缓存和复杂的流量控制来避免长时间的内存访问延迟，这两者在晶体管方面都是昂贵的。

通常，应用程序混合了并行部分和顺序部分，因此系统设计时混合使用 `GPU` 和 `CPU` 以最大限度地提高整体性能。具有高度并行性的应用程序可以利用 `GPU` 的这种大规模并行特性来实现比 `CPU` 更高的性能。

## Sample_code — add 2 numbers

This sample code adds 2 numbers together with a `GPU`:

1. Define a `kernel` (a function to run on a `GPU`).
2. Allocate & initialize the host data.
3. Allocate & initialize the device data.
4. Invoke a kernel in the `GPU`.
5. Copy kernel output to the host.
6. Cleanup.

### Define a kernel

使用关键字 `__global__` 来定义 `kernel`。内核是在 `GPU` 而不是 `CPU` 上运行的函数。

该 `kernel` 将 2 个数字 `a` 和 `b` 相加并将结果存储在 `c` 中。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7 | `// Kernel definition` `// Run on GPU` `// Adding 2 numbers and store the result in c` `__global__ void add(int *a, int *b, int *c)` `{` `*c = *a + *b;` `}` |

### Allocate & initialize host data

在宿主机中，为内核调用分配输入输出参数，并初始化所有输入参数。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7 | `int main(void) {` `// Allocate & initialize host data` `- run on the host` `int a, b, c;` `// host copies of a, b, c` `a = 2;` `b = 7;` `...` `}` |

### Allocate and copy host data to the device

`CUDA` 应用程序通过调用 `CUDA Runtime` 来管理设备空间内存。这包括设备内存分配和释放以及主机和设备内存之间的数据传输。

我们在设备中分配空间，以便我们可以将内核（`a` 和 `b`）的输入从主机复制到设备。

我们还分配空间以便稍后将结果从设备复制到主机。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15 | `int main(void) {` `...` `int *d_a, *d_b, *d_c;` `// device copies of a, b, c` `// Allocate space for device copies of a, b, c` `cudaMalloc((void **)&d_a, size);` `cudaMalloc((void **)&d_b, size);` `cudaMalloc((void **)&d_c, size);` `// Copy a & b from the host to the device` `cudaMemcpy(d_a, &a, size, cudaMemcpyHostToDevice);` `cudaMemcpy(d_b, &b, size, cudaMemcpyHostToDevice);` `...` `}` |

### Invoke the kernel

Invoke the kernel `add` with parameters for `a,b,c`.

|  |  |
| --- | --- |
| 1  2  3  4  5  6 | `int main(void) {` `...` `// Launch add() kernel on GPU with parameters (d_a, d_b, d_c)` `add<<<1,1>>>(d_a, d_b, d_c);` `...` `}` |

为了提供数据并行性，多线程 `CUDA` 应用程序被划分为彼此独立 (通常并发) 执行的线程块。

`add` 的每个并行调用都称为一个 `block`，每个 `block` 都有多个线程。这些线程块可以在 `GPU` 中任何可用的流式多处理器 `SM` 上进行调度。

在我们的简单示例中，因为我们只添加了一对数字 `<<<1,1>>>`，所以我们只需要包含 `1` 个线程的 `1` 个 `block`。`<<<numBlocks,threadsPerBlock>>>`

{ 与常规 `C` 函数调用相比，kernel 可以由 `M` 个 `CUDA` 线程并行执行 `N` 次 `<<<N, M>>>`。在当前的 `GPU` 上，一个线程 `block` 最多可包含 `1024` 个线程 }.

### Copy kernel output to the host

将 `add` 的结果从设备复制到主机

|  |  |
| --- | --- |
| 1  2 | `// Copy result back to the host` `cudaMemcpy(&c, d_c, size, cudaMemcpyDeviceToHost);` |

### Clean up

清理内存

|  |  |
| --- | --- |
| 1  2 | `// Cleanup` `cudaFree(d_a); cudaFree(d_b); cudaFree(d_c);` |

### complete source code

## CUDA logical model

- **thread:** 一个 `CUDA` 的并行程序 `kernel` 会被以许多个 `thread` 来执行。
- **block:** 数个 `thread` 会被组成一个 `block`，同一个 `block` 中的 `thread` 可以同步，也可以通过 `shared memory` 进行通信。
- **grid:** 多个 `block` 则会再构成 `grid`。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/921830_HHNENACHDRX9UVV.png)

- `add<<<4,4>>>(d_a, d_b, d_c);`

![图片描述](https://bbs.kanxue.com/upload/attach/202302/921830_ZBGX6A45HKC8SVC.png)

- 在第二个示例中，我们有 `6` 个 `Block`，每个 `Block` 有 `12` 个 `thread`

![图片描述](https://bbs.kanxue.com/upload/attach/202302/921830_24EPSXECRBYQPYM.png)

## GPU physical model

**SP(Streaming Processor):** 流处理器， 是 `GPU` 最基本的处理单元，在 `fermi` 架构开始被叫做 **CUDA core**。

**SM(Streaming MultiProcessor):** 一个 `SM` 由多个 `CUDA core` 组成，每个 `SM` 根据 `GPU` 架构不同有不同数量的 `CUDA core`，`Pascal` 架构中一个 `SM` 有 `128` 个 `CUDA core`。`SM` 还包括特殊运算单元 `SFU`，共享内存 `shared memory`，寄存器文件 `Register File` 和调度器 `Warp Scheduler` 等。`register` 和 `shared memory` 是稀缺资源，这些有限的资源就使每个 `SM` 中 `active warps` 有非常严格的限制，也就限制了并行能力。

- A SM in the Fermi architecture

![图片描述](https://bbs.kanxue.com/upload/attach/202302/921830_BND75SUCFW83PJB.png)

`GPU` 由许多 `SM` 组成，具有可供所有 `SM` 访问的全局内存 和 本地内存。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/921830_7HJ53XWX66BBBX8.png)

每个 `SM` 包含多个核心，它们共享一个共享内存以及一个自己的本地内存。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/921830_RUQXBUK4FMC7UJP.png)

- Here is the architect for GeoForce 8800 with 16 SMs each with 8 cores (Streaming processing SP).

![图片描述](https://bbs.kanxue.com/upload/attach/202302/921830_EHM795U7K9X7EQR.png)

## Execution model

### Device level

当主机上的 `CUDA` 应用程序调用 `kernel grid` 时，`grid` 的 `block` 被枚举，全局工作分配引擎将它们分配给具有可用执行能力的 `SM`。同一 `block` 的线程总是在同一个 `SM` 上运行。

多个线程块和一个线程块中的多个线程可以在一个 `SM` 上并发执行。当该线程块终止时，新块将在腾出的多处理器上启动。

`grid` 中的所有线程都执行相同的 `kernel`。`GPU` 可以同时处理来自同一应用程序的多个 `kernel`。

`Pascal GP100` 最多可以处理 `32` 个线程块和每个 `SM 2048` 个线程。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/921830_JCYYN2URK5BYE4V.png)

在这里，我们有一个由 `8` 个块组成的 `CUDA` 应用程序。它可以在具有 `2` 个 `SM` 或 `4` 个 `SM` 的 `GPU` 上执行。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/921830_NKJSPZNH22ETXCP.png)

notice：整个设备一次只能处理一个应用程序，应用程序之间的切换速度很慢。

### SM level

- **Wrap**

`SM` 采用的 `SIMT(Single-Instruction, Multiple-Thread，单指令多线程)` 架构，**`warp`(线程束)是最基本的执行单元**，一个 `warp` 一般包含 `32` 个并行 `thread`，这些 `thread` **以不同数据资源执行相同的指令。**

当一个 `kernel` 被执行时，**`grid` 中的线程块被分配到 `SM`** 上，**一个线程块的 `thread` 只能在一个 `SM` 上调度，`SM` 一般可以调度多个线程块**，大量的 `thread` 可能被分到不同的 `SM` 上。每个 `thread` 拥有它自己的程序计数器和状态寄存器，并且用该 `thread` 自己的数据执行指令，这就是所谓的 `Single Instruction Multiple Thread(SIMT)`。

**一个 `CUDA core` 可以执行一个 `thread`，一个 `SM` 的 `CUDA core`s 会分成几个 `warp`（即 `CUDA core` 在 `SM` 中分组)，由 `warp scheduler` 负责调度。**

尽管 `warp` 中的线程从同一程序地址，但可能具有不同的行为，比如分支结构，因为 `GPU` 规定 `warp` 中所有线程在同一周期执行相同的指令，`warp` 发散会导致性能下降。**一个 `SM` 同时并发的 `warp` 是有限的(`active wrap is limited`)**，因为资源限制，`SM` 要为每个线程块分配共享内存，而也要为每个线程束中的线程分配独立的寄存器，所以 `SM` 的配置会影响其所支持的线程块和 `warp` 并发数量。

**每个 `block` 的 `warp` 数量可以由下面的公式计算获得：**

![图片描述](https://bbs.kanxue.com/upload/attach/202302/921830_DKA7CP9VXQQQAYS.png)

**被分到一个 `warp` 中的线程必然属于同一个 `block`，如果 `block` 所含线程数目不是 `warp` 大小的整数倍，那么多出的那些 `thread` 所在的 `warp` 中，会剩余一些 `inactive` 的 `thread`，也就是说，即使凑不够 `warp` 整数倍的 `thread`，硬件也会为 `warp` 凑足，只不过那些 `thread` 是 `inactive` 状态，需要注意的是，即使这部分 `thread` 是 `inactive` 的，也会消耗 `SM` 资源。**

**由于 `warp` 的大小一般为 `32`，所以 `block` 所含的 `thread` 的大小一般要设置为 `32` 的倍数。**

在下图中，我们有 `2` 个调度单元。每个运行不同的 `wrap`。在每个 `warp` 中，它运行相同的指令。当 `warp` 中的线程等待上一条指令完成时，`warp scheduler` 将选择另一个 `warp` 来执行。来自不同块或不同 `kernel` 的两个 `warp` 可以同时执行。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/921830_YQBNJA9P6K365E6.png)

## sample_code — Branch divergence

`warp` 一次执行一条通用指令。每个 `cuda core (SP)` 为 `warp` 中的每个线程运行相同的指令。

执行一个分支，如：

|  |  |
| --- | --- |
| 1  2  3  4 | `if (a[index]==0)` `a[index]++;` `else` `a[index]--;` |

`SM` 跳过受分支条件影响的 `cuda core` 的执行：

![图片描述](https://bbs.kanxue.com/upload/attach/202302/921830_8MMCQQBBJJ7S2YJ.png)

因此，当 `warp` 的所有 `32` 个线程都分支到相同的执行路径时，就会实现最高效率。如果 `warp` 的线程通过数据相关的条件分支发散，则 `warp` 连续执行每个分支路径，禁用不在该路径上的线程，当所有路径完成时，线程会聚回到相同的执行路径。

为了最大化吞吐量，`warp` 中的所有线程都应遵循相同的控制流。可以重写程序，使 `warp` 分支中的线程转到相同的代码：

|  |  |
| --- | --- |
| 1  2  3  4 | `if (a[index]<range)` `...` `// More likely, threads with a warp will branch the same way.` `else` `...` |

优于

|  |  |
| --- | --- |
| 1  2  3  4 | `if (a[index]%2==0)` `...` `else` `...` |

`for` 循环展开是另一种避免分支的技术：

|  |  |
| --- | --- |
| 1  2 | `for (int i=0; i<4; i++)` `c[i] += a[i];` |

|  |  |
| --- | --- |

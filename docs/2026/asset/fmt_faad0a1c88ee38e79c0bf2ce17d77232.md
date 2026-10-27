
## Static Shared Memory

```c
__global__ void staticReverse(int *d, int n)
{
__shared__ int s[64];//静态共享内存, 64个int
int t = threadIdx.x;//线程索引
int tr = n-t-1;//反转后的线程索引
s[t] = d[t];
__syncthreads();//同步所有线程, 保证s[t]已经被赋值
d[t] = s[tr];
}
int main(void)
{
const int n = 64;
int a[n], r[n], d[n];
for (int i = 0; i < n; i++) {//给a赋值为[0, n-1], 给r赋值为[n-1, 0], 给d赋值为0
a[i] = i;
r[i] = n-i-1;
d[i] = 0;
}
int *d_d;
cudaMalloc(&d_d, n * sizeof(int));//为a分配设备内存
// run version with static shared memory
cudaMemcpy(d_d, a, n*sizeof(int), cudaMemcpyHostToDevice);//将a从主机内存复制到设备内存
staticReverse<<<1,n>>>(d_d, n);//以1个block, 64个thread的方式运行staticReverse函数
cudaMemcpy(d, d_d, n*sizeof(int), cudaMemcpyDeviceToHost);//将d从设备内存复制到主机内存
for (int i = 0; i < n; i++) //检查结果是否正确
if (d[i] != r[i]) printf("Error: d[%d]!=r[%d] (%d, %d)n", i, i, d[i], r[i]);
}
```

`__syncthreads()` 是轻量级的并且是 block 级同步屏障。

`__syncthreads()` 确保所有线程在继续之前已完成。

## Dynamic Shared Memory

```c
__global__ void dynamicReverse(int *d, int n)
{
// Dynamic shared memory
extern __shared__ int s[];
int t = threadIdx.x;
int tr = n-t-1;
s[t] = d[t];
__syncthreads();
d[t] = s[tr];
}
int main(void)
{
const int n = 64;
int a[n], r[n], d[n];
for (int i = 0; i < n; i++) {//初始化a,r,d数组
a[i] = i;
r[i] = n-i-1;
d[i] = 0;
}
int *d_d;
cudaMalloc(&d_d, n * sizeof(int)); //为a在设备上分配内存
// run dynamic shared memory version
cudaMemcpy(d_d, a, n*sizeof(int), cudaMemcpyHostToDevice);//将a从主机内存复制到设备内存
dynamicReverse<<<1,n,n*sizeof(int)>>>(d_d, n);//以1个线程块，每个线程块有n个线程，共享内存大小为n*sizeof(int)的动态共享内存版本
cudaMemcpy(d, d_d, n * sizeof(int), cudaMemcpyDeviceToHost);
for (int i = 0; i < n; i++)
if (d[i] != r[i]) printf("Error: d[%d]!=r[%d] (%d, %d)n", i, i, d[i], r[i]);
}
```

## Constant memory

SM 积极地缓存常量内存，从而导致较短的延迟。

```c
__constant__ float M[10];
...
cudaMemcpyToSymbol(...);
```

## sample_code — Reasonable definition of blocks and threads

CUDA 使用块和线程来提供数据并行性。CUDA 创建多个块，每个块有多个线程。每个线程调用同一个 kernel 来处理一段数据。

![图片描述](upload/attach/202302/921830_XRKCMHHZ9XY3H39.png)

在这里，我们的目的是将 1024x1024 个数字相加，为了保持内核几乎保持不变，为了添加所有数字，我们创建了 4096 个块，每个块有 256 个线程。

1024×1024=4096×256×1

每个执行 kernel 的线程都有一个唯一的 Block ID 和线程 ID，可以通过内置的 `blockIdx.x` 和 `threadIdx.x` 变量在 kernel 中访问。我们使用这个索引来定位我们要在内核中添加的数字对。

![图片描述](upload/attach/202302/921830_K7SGVNR8RNK7KVN.png)

```cpp
#define N (1024*1024) //threads的数量,也就是我们要执行多少次addition
#define THREADS_PER_BLOCK 256
__global__ void add(int *a, int *b, int *c)
{
// blockIdx.x is the index of the block.
// Each block has blockDim.x threads.
// threadIdx.x is the index of the thread.
// Each thread can perform 1 addition.
// a[index] & b[index] are the 2 numbers to add in the current thread.
int index = blockIdx.x * blockDim.x + threadIdx.x;
c[index] = a[index] + b[index];
}
int main(void) {
int *a, *b, *c;
int size = N * sizeof(int);
// Alloc space for host copies of a, b, c and setup input values
a = (int *)malloc(size); random_ints(a, N);//随机生成N个整数
b = (int *)malloc(size); random_ints(b, N);//随机生成N个整数
c = (int *)malloc(size);
int *d_a, *d_b, *d_c;
// Alloc space for device copies of a, b, c
cudaMalloc((void **)&d_a, size);
cudaMalloc((void **)&d_b, size);
cudaMalloc((void **)&d_c, size);
// Copy inputs to device
cudaMemcpy(d_a, a, size, cudaMemcpyHostToDevice);
cudaMemcpy(d_b, b, size, cudaMemcpyHostToDevice);
// Launch add() kernel on GPU
add<<<N/THREADS_PER_BLOCK,THREADS_PER_BLOCK>>>(d_a, d_b, d_c);//以N/THREADS_PER_BLOCK个block，每个block有THREADS_PER_BLOCK个threads来执行addition
// Copy result back to host
cudaMemcpy(c, d_c, size, cudaMemcpyDeviceToHost);
// Cleanup
free(a); free(b); free(c);
cudaFree(d_a); cudaFree(d_b); cudaFree(d_c);
return 0;
}
```

### Threads & shared memory

为什么我们有 block 时还需要线程？CUDA 线程可以访问多个具有不同性能的内存空间。每个线程都有自己的本地内存。每个线程块都具有对该块的所有线程可见的共享内存，并且与该块具有相同的生命周期。所有线程都可以访问相同的全局内存。共享内存的数据访问比全局内存快。首先将数据从主机复制到 GPU 中的全局内存。块中的所有线程都在同一个多处理器上运行。因此，为了减少内存延迟，我们可以将块所需的所有数据从全局内存复制到共享内存。

Use `__shared__` to declare a variable using the shared memory:

```cpp
__global__ void add(int *a, int *b, int *c)
{
__shared__ int temp[1000];
}
```

共享内存可以加快性能，尤其是当我们需要频繁访问数据时。在这里，我们创建了一个新的内核模板，将其所有相邻数据添加到一个半径内。

![图片描述](upload/attach/202302/921830_QZSZ7RC3MAWVU2G.png)

我们将块中需要的所有数据读取到共享内存中。半径为 7，块索引为 512 到 1023，我们需要读取 505(512-7) 到 1030(1023+7) 的数据。

```cpp
#define RADIUS 7
#define BLOCK_SIZE 512
__global__ void stencil(int *in, int *out)
{
__shared__ int temp[BLOCK_SIZE + 2 * RADIUS];
int gindex = threadIdx.x + blockIdx.x * blockDim.x;
int lindex = threadIdx.x + RADIUS;
// Read input elements into shared memory
temp[lindex] = in[gindex];
// At both end of a block, the sliding window moves beyond the block boundary.
// E.g, for thread id = 512, we wiil read in[505] and in[1030] into temp.
if (threadIdx.x < RADIUS) {
temp[lindex - RADIUS] = in[gindex - RADIUS];
temp[lindex + BLOCK_SIZE] = in[gindex + BLOCK_SIZE];
}
// Apply the stencil
int result = 0;
for (int offset = -RADIUS ; offset <= RADIUS ; offset++)
result += temp[lindex + offset];
// Store the result
out[gindex] = result;
}
```

### Thread synchronization

上一节中的代码有一个致命的数据竞争问题。在访问数据之前，数据不会存储在共享内存中。例如，要计算线程 20 的结果，我们需要访问对应于 `in[13]` 到 `in[27]` 的 `temp`。

```cpp
for (int offset = -RADIUS ; offset <= RADIUS ; offset++)
result += temp[lindex + offset]; // Data race problem here.
```

然而，线程 27 负责使用 `in[27]` 加载 `temp`。由于线程是在不保证顺序的情况下并行执行的，因此我们可以在线程 27 将 `in[27]` 存储到 `temp` 之前计算线程 20 的结果。

```cpp
if (threadIdx.x < RADIUS) {
temp[lindex - RADIUS] = in[gindex - RADIUS];
temp[lindex + BLOCK_SIZE] = in[gindex + BLOCK_SIZE];
}
```

所以，和其他多线程编程一样，CUDA 提供了线程同步方法 `__syncthreads()` 来解决这个数据竞争问题。所有线程都将被阻塞在 `__syncthreads()` 处，直到同一块中的所有线程都到达同一点。

```cpp
__global__ void stencil_1d(int *in, int *out) {
__shared__ int temp[BLOCK_SIZE + 2 * RADIUS];
int gindex = threadIdx.x + blockIdx.x * blockDim.x;
int lindex = threadIdx.x + RADIUS;
// Read input elements into shared memory
temp[lindex] = in[gindex];
// At both end of a block, the sliding window moves beyond the block boundary.
if (threadIdx.x < RADIUS) {
temp[lindex - RADIUS] = in[gindex - RADIUS];
temp[lindex + BLOCK_SIZE] = in[gindex + BLOCK_SIZE];
}
// Synchronize (ensure all the threads will be completed before continue)
__syncthreads();
// Apply the stencil
int result = 0;
for (int offset = -RADIUS ; offset <= RADIUS ; offset++)
result += temp[lindex + offset];
// Store the result
out[gindex] = result;
}
```

Other synchronization methods:

| Call | Behavior |
| --- | --- |
| `cudaMemcpy()` | 阻塞 CPU 直到复制完成。复制在所有前面的 CUDA 调用完成时开始 |
| `cudaMemcpyAsync()` | 异步，不阻塞 CPU |
| `cudaDeviceSynchronize()` | 阻塞 CPU，直到所有前面的 CUDA 调用都完成 |

## Thread hierarchy（线程层次结构）

在前面的示例中，线程索引 `threadIdx.x` 是一维的。为了更方便地访问多维矩阵，CUDA 还支持多维线程索引。

`threadIdx` 是一个 3 分量向量，因此可以使用一维、二维或三维线程索引来标识线程，形成一维、二维或三维线程块。这提供了一种自然的方式来调用域中元素的计算，例如向量、矩阵或体积。

线程的索引和它的线程 ID 以一种直接的方式相互关联：

对于一维块，它们是相同的；对于大小为 (`Dx`, `Dy`) 的二维块，索引为 (`x`, `y`) 的线程的线程 ID 为 (`x + y Dx`)；对于大小为 (`Dx`, `Dy`, `Dz`) 的三维块，索引为 (`x`, `y`, `z`) 的线程的线程 ID 为 (`x + y Dx + z Dx Dy`)。

以下代码将两个二维矩阵与 1 个 NxN 线程的线程块相加。`threadIdx.x` 和 `threadIdx.y` 表示二维索引，便于二维矩阵访问。

```cpp
__global__ void MatAdd(float A[N][N], float B[N][N],
float C[N][N])
{
// the blockIdx and treadIdx is now 2-dimensional.
int i = blockIdx.x * blockDim.x + threadIdx.x;
int j = blockIdx.y * blockDim.y + threadIdx.y;
C[i][j] = A[i][j] + B[i][j];
}
int main()
{
...
// Kernel invocation with one block of N * N * 1 threads
dim3 threadsPerBlock(N, N);//二维threads
MatAdd<<<1, threadsPerBlock>>>(A, B, C);
...
}
```

CUDA 支持类型为 `dim3` 的一维、二维或三维线程索引。

块可能不会与输入数据边界完全对齐。我们添加一个 `if` 循环以避免线程超出输入数据边界。例如，在最后一个块中，我们可能没有足够的数据来配置线程数量。


```
static boolean detect\_threadCpuTimeNanos(){
long start = Debug.threadCpuTimeNanos();
for(int i=0; i<1000000; ++i)
continue;
long stop = Debug.threadCpuTimeNanos();
if(stop - start < 10000000) {
return false;
}
else {
return true;
}
}
```

#### 处理与 JDWP 相关的数据结构

在 Dalvik 中，可以通过 `DvmGlobals` 结构访问全局虚拟机状态，全局变量 `gDvm` 有一个指向这个结构的指针。`DvmGlobals` 包含各种变量和指针，这些变量和指针对于 JDWP 调试非常重要，可以被篡改。

```
struct DvmGlobals {
/\*
\* Some options that could be worth tampering with :)
\*/
bool jdwpAllowed; // debugging allowed for this process?
bool jdwpConfigured; // has debugging info been provided?
JdwpTransportType jdwpTransport;
bool jdwpServer;
char\* jdwpHost;
int jdwpPort;
bool jdwpSuspend;
Thread\* threadList;
bool nativeDebuggerActive;
bool debuggerConnected; /\* debugger or DDMS is connected \*/
bool debuggerActive; /\* debugger is making requests \*/
JdwpState\* jdwpState;
};
```

例如，<a href="elink@3e1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0M7X3q4\*7P5h3E0A6k6o6V1#2i4K6u0r3b7X3q4U0K9%4g2H3i4K6u0V1e0h3!0T1K9h3I4W2i4K6u0V1f1$3g2U0N6i4u0A6N6s2W2Q4x3X3c8d9k6i4m8G2M7Y4c8Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3b7h3&6V1M7X3!0A6k6q4u0q4L8V1c8W2k6X3g2F1M7$3g2K6x3U0l9I4x3K6l9#2i4K6u0W2M7r3c8X3">将 `gDvm.methDalvikDdmcServer\_dispatch` 函数指针设置为 `NULL` 会使 JDWP 线程崩溃</a>：

```
JNIEXPORT jboolean JNICALL Java\_poc\_c\_crashOnInit ( JNIEnv\* env , jobject ) {
gDvm.methDalvikDdmcServer\_dispatch = NULL;
}
```

即使 `gDvm` 变量不可用，也可以通过使用 ART 中的类似技术禁用调试。ART运行时将与 jdwp 相关的类的一些虚函数表导出为全局符号（在 C++ 中，虚函数表是包含指向类方法的指针的表）。这包括类 `JdwpSocketState` 和 `JdwpAdbState` 的虚函数表，它们分别通过网络套接字和 ADB 处理 JDWP 连接。你可以<a href="elink@b70K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3j5h3&6@1j5h3N6W2M7r3!0A6L8Y4c8Q4x3X3g2K6k6#2)9J5c8X3u0D9L8$3N6Q4x3V1j5^5z5q4)9J5k6r3q4F1N6r3W2Q4x3X3c8V1k6h3u0#2k6$3N6A6L8X3N6Q4x3X3c8X3N6h3&6Q4x3X3c8%4K9i4c8Z5i4K6u0V1j5h3&6V1M7X3!0A6k6q4)9J5k6r3q4J5N6l9`.`.">通过重写关联虚函数表中的方法指针来操纵调试运行时的行为</a>。

重写方法指针的一种方法是用 `JdwpAdbState:: Shutdown` 的地址重写函数 `jdwpAdbState:: processIncoming` 的地址，这将导致调试器立即断开连接。

```
#include <jni.h>
#include <string>
#include <android/log.h>
#include <dlfcn.h>
#include <sys/mman.h>
#include <jdwp/jdwp.h>
#define log(FMT, ...) \_\_android\_log\_print(ANDROID\_LOG\_VERBOSE, "JDWPFun", FMT, ##\_\_VA\_ARGS\_\_)
// Vtable structure. Just to make messing around with it more intuitive
struct VT\_JdwpAdbState {
unsigned long x;
unsigned long y;
void \* JdwpSocketState\_destructor;
void \* \_JdwpSocketState\_destructor;
void \* Accept;
void \* showmanyc;
void \* ShutDown;
void \* ProcessIncoming;
};
extern "C"
JNIEXPORT void JNICALL Java\_sg\_vantagepoint\_jdwptest\_MainActivity\_JDWPfun(
JNIEnv \*env,
jobject /\* this \*/) {
void\* lib = dlopen("libart.so", RTLD\_NOW);
if (lib == NULL) {
log("Error loading libart.so");
dlerror();
}else{
struct VT\_JdwpAdbState \*vtable = ( struct VT\_JdwpAdbState \*)dlsym(lib, "\_ZTVN3art4JDWP12JdwpAdbStateE");
if (vtable == 0) {
log("Couldn't resolve symbol '\_ZTVN3art4JDWP12JdwpAdbStateE'.\n");
}else {
log("Vtable for JdwpAdbState at: %08x\n", vtable);
// Let the fun begin!
unsigned long pagesize = sysconf(\_SC\_PAGE\_SIZE);
unsigned long page = (unsigned long)vtable & ~(pagesize-1);
mprotect((void \*)page, pagesize, PROT\_READ | PROT\_WRITE);
vtable->ProcessIncoming = vtable->ShutDown;
// Reset permissions & flush cache
mprotect((void \*)page, pagesize, PROT\_READ);
}
}
}
```

#### 传统的反调试

在 Linux 上，`ptrace` 系统调用用于观察和控制进程（tracee）的执行，并检查和更改该进程的内存和寄存器。`ptrace` 是在原生代码中实现系统调用跟踪和断点调试的主要方式。大多数 JDWP 反调试技巧（对于基于计时器的检查来说可能是安全的）不会捕获基于 `ptrace` 的经典调试器，因此，许多 Android 反调试技巧包括 `ptrace`，通常利用了一次只能连接到进程的一个调试器这一事实。

#### 检查 TracerPid

当你调试应用程序并在原生代码上设置断点时，Android Studio 将把所需的文件复制到目标设备并启动 `lldb-server`，它将使用 `ptrace` 附加到进程。从现在开始，如果你检查被调试进程的<a href="elink@63dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0j5h3^5%4i4K6u0W2L8%4u0Y4i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6E0j5h3&6Q4x3X3c8H3j5h3N6W2M7#2)9J5c8X3#2S2L8U0g2Q4x3V1k6H3M7X3!0U0i4K6u0W2y4g2)9J5k6h3S2@1L8h3H3`.">状态文件</a>（`/proc//status` 或 `/proc/self/status`），你将看到“TracerPid”字段的值与0不同，这是调试的标志。

>记住，这只适用于原生代码。如果你正在调试只使用 Java/ kotlin 的应用程序，那么“TracerPid”字段的值应该是0。

这种技术通常应用于 C 写的 JNI 原生库中，如 <a href="elink@339K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4M7r3g2J5k6Y4c8G2L8$3I4K6i4K6u0r3k6%4m8W2M7X3k6@1L8$3!0D9M7#2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6K6M7X3y4Q4x3V1k6Z5k6h3q4H3i4K6u0V1j5$3S2W2j5$3E0W2M7W2)9J5k6h3y4U0i4K6t1K6e0o6p5I4x3R3`.`.">Google 的 gperftools（Google 性能工具）堆检查器实现的 `IsDebuggerAttached` </a>所示。但是，如果你希望将此检查作为 Java/Kotlin 代码的一部分，你可以参考 <a href="elink@15cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6s2u0S2P5Y4A6W2M7X3g2Q4x3V1k6S2L8Y4c8A6i4K6u0V1k6h3#2#2L8r3q4@1L8%4u0Q4x3V1j5`.">Tim Strazzere 的反模拟器项目</a>中的 `hasTracerPid` 方法的 Java 实现。

当你尝试自己实现这种方法时，可以使用 ADB 手动检查 TracerPid 的值。下面的清单使用谷歌的 NDK 示例应用 <a href="elink@662K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0r3L8X3c8C8i4K6u0V1M7$3q4E0M7r3I4W2M7#2)9J5c8Y4c8J5k6h3g2Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0V1L8h3E0Q4x3V1k6Z5k6h3I4D9L8#2)9J5k6r3A6F1K9b7`.`.">hello-jni (com.example.hellojni)</a> 在附加 Android Studio 的调试器后执行检查：

```
$ adb shell ps -A | grep com.example.hellojni
u0\_a271 11657 573 4302108 50600 ptrace\_stop 0 t com.example.hellojni
$ adb shell cat /proc/11657/status | grep -e "^TracerPid:" | sed "s/^TracerPid:\t//"
TracerPid: 11839
$ adb shell ps -A | grep 11839
u0\_a271 11839 11837 14024 4548 poll\_schedule\_timeout 0 S lldb-server
```

你可以看到 com.example.hellojni（PID = 11657）的状态文件如何包含11839的 TracerPID，我们可以将其标识为 lldb-server 进程。

#### 使用 Fork 和 petrace

你可以通过类似于以下简单示例代码，将子进程附加到父进程作为调试器来防止进程调试：

```
void fork\_and\_attach()
{
int pid = fork();
if (pid == 0)
{
int ppid = getppid();
if (ptrace(PTRACE\_ATTACH, ppid, NULL, NULL) == 0)
{
waitpid(ppid, NULL, 0);
/\* Continue the parent process \*/
ptrace(PTRACE\_CONT, NULL, NULL);
}
}
}
```

附加了子进程后，进一步附加到父进程的尝试将会失败。我们可以通过将代码编译成 JNI 函数并将其打包到我们在设备上运行的应用程序中来验证这一点。

```
root@android:/ # ps | grep -i anti
u0\_a151 18190 201 1535844 54908 ffffffff b6e0f124 S sg.vantagepoint.antidebug
u0\_a151 18224 18190 1495180 35824 c019a3ac b6e0ee5c S sg.vantagepoint.antidebug
```

使用 gdbserver 试图附加到父进程会失败并有一个错误：

```
root@android:/ # ps | grep -i anti
u0\_a151 18190 201 1535844 54908 ffffffff b6e0f124 S sg.vantagepoint.antidebug
u0\_a151 18224 18190 1495180 35824 c019a3ac b6e0ee5c S sg.vantagepoint.antidebug
```

但是，你可以通过杀死子进程并释放父进程而不被 trace 来轻松地绕过这个失败。因此，你通常会发现更复杂的方案，包括多个进程和线程以及某种形式的监视来阻止篡改。常用的方法包括

* fork 多个进程，相互跟踪

* 跟踪运行过程以确保子进程活着

* 监视 `/proc` 文件系统中的值，例如 `/proc/pid/status` 中的 TracerPID。

让我们看看对上述方法的一个简单改进。在初始 `fork` 之后，我们在父进程中启动一个额外的线程，该线程持续监视子进程的状态。根据应用程序是在 debug 模式还是 release 模式（由清单文件中的 `android:debuggable` 标志表示），子进程应该做以下事情之一：

* 在 release 模式中：对 `ptrace` 的调用失败，子进程立即崩溃并出现段错误（退出代码11）。

* 在 debug 模式下：对 `ptrace` 的调用有效，并且子进程应该一直运行，因此，对 `waitpid(child\_pid)` 的调用应该永远不会返回。 如果返回了，则说明有些可疑，将杀死整个流程组。

下面是使用 JNI 函数实现的完整的改进代码：

```
#include <jni.h>
#include <unistd.h>
#include <sys/ptrace.h>
#include <sys/wait.h>
#include <pthread.h>
static int child\_pid;
void \*monitor\_pid() {
int status;
waitpid(child\_pid, &status, 0);
/\* Child status should never change. \*/
\_exit(0); // Commit seppuku
}
void anti\_debug() {
child\_pid = fork();
if (child\_pid == 0)
{
int ppid = getppid();
int status;
if (ptrace(PTRACE\_ATTACH, ppid, NULL, NULL) == 0)
{
waitpid(ppid, &status, 0);
ptrace(PTRACE\_CONT, ppid, NULL, NULL);
while (waitpid(ppid, &status, 0)) {
if (WIFSTOPPED(status)) {
ptrace(PTRACE\_CONT, ppid, NULL, NULL);
} else {
// Process has exited
\_exit(0);
}
}
}
} else {
pthread\_t t;
/\* Start the monitoring thread \*/
pthread\_create(&t, NULL, monitor\_pid, (void \*)NULL);
}
}
JNIEXPORT void JNICALL
Java\_sg\_vantagepoint\_antidebug\_MainActivity\_antidebug(JNIEnv \*env, jobject instance) {
anti\_debug();
}
```

同样，我们把它打包到一个 Android 应用程序中，看看它是否能工作。与前面一样，当我们运行应用程序的调试版本时，会出现两个进程。

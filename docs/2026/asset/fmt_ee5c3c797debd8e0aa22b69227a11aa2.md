
1）`IoCopyCurrentIrpStackLocationToNext`覆盖了下一层IO栈，不是等于覆盖了下一层的运行上下文？

——下一层IO栈没有任何内容，需要上一层分发函数自己构造。`IoCallDriver`已经帮我们填好了`Device`，我们只用考虑有没有回调函数或者需要修改的字段，来选择`Skip`或者`Copy`就好。

2）为什么要`IoSkipCurrentIrpStackLocation`，而不让栈帧的指针直接由`IoCallDriver`指向下一层？

——因为下一层没有任何内容，完全需要自己来构造。

## 4. 参考内容

### 4.1 WRK IoCallDriver

```
NTSTATUS

FORCEINLINE

IopfCallDriver(

    IN PDEVICE_OBJECT DeviceObject,

    IN OUT PIRP Irp

    )

/*++

Routine Description:

    This routine is invoked to pass an I/O Request Packet (IRP) to another

    driver at its dispatch routine.

Arguments:

    DeviceObject - Pointer to device object to which the IRP should be passed.

    Irp - Pointer to IRP for request.

Return Value:

    Return status from driver's dispatch routine.

--*/

{

    PIO_STACK_LOCATION irpSp;

    PDRIVER_OBJECT driverObject;

    NTSTATUS status;

    //

    // Ensure that this is really an I/O Request Packet.

    //

    ASSERT( Irp->Type == IO_TYPE_IRP );

    //

    // Update the IRP stack to point to the next location.

    //

    Irp->CurrentLocation--;

    if (Irp->CurrentLocation <= 0) {

        KiBugCheck3( NO_MORE_IRP_STACK_LOCATIONS, (ULONG_PTR) Irp, 0, 0 );

    }

    irpSp = IoGetNextIrpStackLocation( Irp );

    Irp->Tail.Overlay.CurrentStackLocation = irpSp;

    //

    // Save a pointer to the device object for this request so that it can

    // be used later in completion.

    //

    irpSp->DeviceObject = DeviceObject;

    //

    // Invoke the driver at its dispatch routine entry point.

    //

    driverObject = DeviceObject->DriverObject;

    //

    // Prevent the driver from unloading.

    //

    status = driverObject->MajorFunction[irpSp->MajorFunction]( DeviceObject,

                                                              Irp );

    return status;

}
```

### 4.2 其他参考文献

[1] AntBean [原创]驱动入门：从WRK看IRP 理论篇 , <https://bbs.pediy.com/thread-130876.htm>

[2] [385K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3c8#2M7$3g2J5i4K6u0W2N6s2g2Q4x3X3c8U0K9r3g2E0L8X3W2@1P5W2)9J5k6h3c8W2i4K6u0r3i4K6N6q4K9r3g2Z5j5g2)9J5c8X3!0F1k6i4W2Q4y4h3k6%4k6r3#2Q4x3V1k6U0K9o6l9#2k6g2)9J5k6h3S2@1L8b7`.`.](https://bbs.kanxue.com/elink@a15K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3c8#2M7$3g2J5i4K6u0W2N6s2g2Q4x3X3c8U0K9r3g2E0L8X3W2@1P5W2)9J5k6h3c8W2i4K6u0r3i4K6N6q4K9r3g2Z5j5g2)9J5c8X3!0F1k6i4W2Q4y4h3k6%4k6r3#2Q4x3V1k6U0K9o6l9#2k6g2)9J5k6h3S2@1L8b7`.`.)

[3] [1d7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3#2E0N6h3&6A6N6s2W2Q4x3X3g2G2M7%4u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1K9i4y4U0N6i4y4K6K9h3!0F1i4K6u0r3x3U0t1H3y4U0R3#2i4K6u0r3K9h3!0K6K9$3W2H3j5%4g2J5M7X3g2F1N6r3W2J5M7s2y4@1j5h3y4C8L8r3!0U0j5i4c8A6L8$3&6Q4x3X3c8S2L8X3c8Q4x3X3c8A6L8$3y4G2M7s2W2U0N6i4u0J5k6h3&6@1K9i4u0H3M7%4c8S2j5$3E0D9L8$3y4S2N6r3W2G2L8Y4c8G2L8X3g2^5N6l9`.`.](https://bbs.kanxue.com/elink@d6eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3#2E0N6h3&6A6N6s2W2Q4x3X3g2G2M7%4u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1K9i4y4U0N6i4y4K6K9h3!0F1i4K6u0r3x3U0t1H3y4U0R3#2i4K6u0r3K9h3!0K6K9$3W2H3j5%4g2J5M7X3g2F1N6r3W2J5M7s2y4@1j5h3y4C8L8r3!0U0j5i4c8A6L8$3&6Q4x3X3c8S2L8X3c8Q4x3X3c8A6L8$3y4G2M7s2W2U0N6i4u0J5k6h3&6@1K9i4u0H3M7%4c8S2j5$3E0D9L8$3y4S2N6r3W2G2L8Y4c8G2L8X3g2^5N6l9`.`.)

[4] Windows内核原理与实现 潘爱民

风扫春残雪@52pojie/NONAME剑人@pediy

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2020-7-17 18:36

被NONAME剑人编辑

，原因：

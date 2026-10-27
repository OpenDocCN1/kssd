
# [原创]修改`System.dll`绕过.net的License

前两天同学给我发了个.net编写的cpu优化的程序，由于注册机制无法正常使用，他寻思看我能不能有什么解决办法。以前搞过一些简单的crackme，但对.net程序的机制还很陌生。破解过程当中遇到了很多问题，发个帖子叙述一下具体情况，当中一些遗留问题也希望能得到达人们的指点~![](https://bbs.kanxue.com/view/img/face/16.gif)

![](https://bbs.kanxue.com/upload/attach/201103/208207_irrtqpwvgz2tmex.jpg)

解压后就这些文件，`DLLInject`是主程序，`HID`是机器码文件生成用的，那个dll文件至今不知道做什么用的（猜测可能是优化CPU时用到）。直接运行`DLLInject`会弹出需要注册的对话框。

![](https://bbs.kanxue.com/upload/attach/201103/208207_p7jb9iwr56e39vh.jpg)

这种对话框乍看来只有许可证文件存在程序才能正常运行，点击OK程序就退出了。后来才查到这样的验证机制是.net本身提供的`System.ComponentModel.LicenseXXX`。.net许可证机制提供了框架，具体的许可证校验算法肯定还是作者自己实现的，以配合硬件机器码完成验证。

![](https://bbs.kanxue.com/upload/attach/201103/208207_e82vixk4zvbrwpr.jpg)

用`Reflector`无法正常载入，看来文件还被特意修改过啊，解决方法：62cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6V1M7X3g2S2L8i4A6Y4K9W2)9J5c8X3u0D9L8$3N6Q4x3V1k6A6N6r3g2E0i4K6u0r3x3$3x3H3j5X3x3I4z5r3f1J5y4o6x3#2z5e0u0X3x3o6f1I4x3$3b7&6x3U0S2S2i4K6u0W2K9s2c8E0L8q4!0q4x3#2)9^5x3q4)9^5x3W2!0q4y4q4!0n7z5q4)9^5c8q4!0q4z5q4!0n7c8W2)9^5y4#2!0q4z5g2!0m8z5g2!0m8b7#2!0q4y4q4!0n7z5q4)9^5b7g2!0q4y4g2)9^5c8W2)9^5z5q4!0q4y4g2)9^5y4#2!0n7b7g2!0q4y4#2)9^5c8g2!0n7x3q4!0q4y4q4!0n7b7g2)9^5y4W2!0q4y4W2)9&6y4W2!0n7x3q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4z5g2)9&6y4q4)9&6z5g2!0q4z5q4!0m8c8W2!0m8c8W2!0q4c8W2!0n7b7#2)9&6b7g2u0W2k6X3I4W2j5%4c8G2M7W2)9J5y4X3&6T1M7%4m8Q4x3@1u0q4M7Y4u0G2M7W2)9K6b7f1#2G2k6s2g2D9k6g2)9J5y4X3&6T1M7%4m8Q4x3@1u0Q4x3U0N6Q4c8e0u0Q4z5o6m8Q4b7e0k6Q4x3U0N6Q4x3U0k6F1j5Y4y4H3i4K6y4n7j5$3!0F1N6r3q4A6L8Y4y4Q4x3U0k6F1j5Y4y4H3i4K6y4n7P5X3g2J5L8#2)9J5y4X3&6T1M7%4m8Q4x3@1u0G2M7W2)9J5y4X3&6T1M7%4m8Q4x3@1u0E0N6h3I4@1K9i4m8D9k6g2)9J5y4X3&6T1M7%4m8Q4x3@1u0E0L8$3c8#2L8r3g2Q4x3U0k6F1j5Y4y4H3i4K6y4n7k6r3g2X3K9h3&6A6N6r3W2G2L8Y4y4Q4c8e0y4Q4z5o6m8Q4z5o6u0Q4c8e0S2Q4b7e0N6Q4b7e0y4Q4c8e0g2Q4z5o6k6Q4b7U0y4Q4c8e0k6Q4z5e0k6Q4b7U0W2Q4c8e0k6Q4b7U0y4Q4z5e0g2Q4c8f1k6Q4b7V1y4Q4z5f1q4Z5N6s2c8H3i4K6y4m8i4K6u0r3i4K6u0r3K9r3W2Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8X3c8J5k6h3q4E0P5X3N6B7i4K6u0r3j5X3I4G2k6#2)9J5c8X3W2@1k6h3#2Q4x3V1j5#2j5$3b7#2k6U0p5@1y4e0k6X3k6h3c8U0x3K6R3^5j5U0u0T1y4$3c8U0z5r3q4Q4x3X3g2Z5N6r3#2D9i4@1f1K6i4K6R3H3i4K6R3J5i4@1f1&6i4K6R3K6i4@1u0p5i4@1f1$3i4K6V1^5i4@1q4r3i4@1f1#2i4K6V1H3i4K6S2o6i4@1f1@1i4@1t1^5i4K6R3H3i4@1f1@1i4@1t1^5i4@1q4m8i4@1f1@1i4@1u0m8i4@1u0m8i4@1f1#2i4K6R3$3i4K6V1&6i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1$3i4K6V1$3i4K6R3%4i4@1f1%4i4@1q4n7i4@1p5H3i4@1f1K6i4K6R3H3i4K6R3J5i4@1f1^5i4@1u0r3i4K6V1&6i4@1f1#2i4K6W2n7i4K6W2q4f1X3g2X3L8r3g2U0N6r3!0J5i4@1f1^5i4@1u0p5i4@1u0p5i4@1f1#2i4K6R3#2i4@1p5#2i4@1f1^5i4K6V1&6i4@1u0p5i4@1f1%4i4K6R3@1i4@1t1$3i4@1f1^5i4@1u0r3i4K6V1^5i4@1f1$3i4K6V1^5i4@1q4r3i4@1f1$3i4K6S2r3i4K6V1H3i4@1f1%4i4@1p5@1i4@1u0m8i4@1f1$3i4K6V1#2i4@1t1H3i4@1f1%4i4@1u0n7i4K6R3@1i4@1f1^5i4@1t1$3i4K6S2m8i4@1f1%4i4K6V1#2i4K6S2o6i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1&6i4K6V1@1i4K6V1&6i4@1f1^5i4@1q4r3i4@1q4r3i4@1g2r3i4@1u0o6i4K6R3^5i4@1f1%4i4K6W2o6i4K6S2n7i4@1f1$3i4K6W2p5i4@1p5#2i4@1f1^5i4@1u0r3i4K6V1^5i4@1f1$3i4K6W2o6i4K6R3&6i4@1f1#2i4K6R3#2i4@1t1$3i4@1f1@1i4@1u0n7i4K6V1$3i4@1f1^5i4@1p5J5i4@1q4n7i4@1f1@1i4@1u0r3i4@1q4q4i4@1f1$3i4K6V1@1i4@1t1&6i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1#2i4@1t1H3i4K6S2r3i4@1f1#2i4K6W2o6i4@1t1H3i4@1f1$3i4K6V1$3i4@1t1&6i4@1g2r3i4@1u0o6i4K6R3&6i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4K6V1H3i4K6S2p5i4@1f1%4i4@1p5%4i4@1t1H3i4@1f1@1i4@1t1&6i4K6W2r3i4@1f1&6i4K6R3K6i4@1u0p5i4@1f1^5i4@1p5J5i4@1q4n7i4@1f1$3i4@1t1%4i4@1t1%4i4@1f1$3i4@1t1%4i4K6R3$3i4@1f1^5i4@1u0r3i4K6R3%4i4@1g2r3i4@1u0o6i4K6R3^5i4@1f1#2i4K6S2r3i4@1p5$3i4@1f1#2i4@1p5@1i4K6V1$3i4@1f1%4i4K6V1@1i4@1p5^5f1%4c8J5L8$3&6Y4e0X3q4E0k6g2u0W2L8h3!0$3k6g2!0q4y4W2!0m8x3q4!0n7z5g2!0q4y4W2)9&6b7#2!0m8b7#2!0q4y4q4!0n7z5q4)9^5c8q4!0q4z5q4)9^5x3#2!0n7c8q4!0q4y4W2!0m8c8q4!0m8x3#2!0q4y4g2!0n7z5q4!0n7z5q4!0q4z5q4!0n7c8q4!0n7c8q4!0q4y4g2)9^5y4g2!0m8y4g2!0q4z5q4!0m8c8W2!0m8y4g2!0q4y4#2!0m8z5q4)9^5b7W2!0q4y4g2!0n7b7g2)9^5c8W2!0q4c8W2!0n7b7#2)9^5z5g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4q4!0n7c8q4)9^5y4W2!0q4z5q4!0n7y4g2!0n7y4#2!0q4y4#2!0m8x3q4)9^5x3g2!0q4y4g2)9^5c8W2!0m8c8W2!0q4y4q4!0n7b7W2!0m8y4g2!0q4y4#2)9&6b7#2)9^5b7W2!0q4y4q4!0n7z5q4!0m8b7g2!0q4y4g2!0m8y4q4!0m8y4#2!0q4z5q4)9^5y4#2!0n7y4q4!0q4y4#2!0n7b7W2)9&6x3#2!0q4y4W2)9&6c8g2)9^5y4q4!0q4y4q4!0n7b7g2)9^5y4W2!0q4c8W2!0n7b7#2)9^5b7$3g2F1N6s2u0&6M7r3!0A6L8Y4c8Q4c8e0g2Q4z5o6N6Q4b7V1c8Q4c8e0k6Q4z5e0g2Q4b7U0m8Q4c8f1k6Q4b7V1y4Q4z5o6S2e0z5h3!0s2c8e0c8r3e0f1c8Q4c8f1k6Q4b7V1y4Q4z5o6W2Q4c8e0S2Q4b7V1k6Q4z5e0S2Q4c8e0k6Q4z5e0S2Q4b7f1k6Q4c8e0k6Q4z5e0N6Q4b7e0m8Q4c8e0k6Q4b7U0y4Q4z5e0g2Q4c8e0k6Q4z5e0S2Q4b7V1g2Q4c8e0N6Q4b7e0c8Q4b7V1q4Q4c8e0y4Q4z5o6m8Q4z5o6t1`.

![](https://bbs.kanxue.com/upload/attach/201103/208207_weff5chxie35y78.jpg)

其他很多函数内容都是形如下式，都是一些看起来毫无作用的垃圾信息。估计是干扰函数或者是运行时动态修复吧。

![](https://bbs.kanxue.com/upload/attach/201103/208207_7i2kgz974ur0ych.jpg)

用`ILDASM`把代码导成`il`形式看入口函数终于能正常查看了，也是为数不多的能看到一些语句的函数（其他函数都是返回个`null`值就没了）。

```
.method assembly static void  S9oGE4FMD(string[] A_0) cil managed noinlining

{

  .entrypoint

  .custom instance void [mscorlib]System.STAThreadAttribute::.ctor() = ( 01 00 00 00 ) 

  .custom instance void [System]System.ComponentModel.EditorBrowsableAttribute::.ctor(valuetype [System]System.ComponentModel.EditorBrowsableState) = ( 01 00 02 00 00 00 00 00 ) 

  .custom instance void [mscorlib]System.Diagnostics.DebuggerHiddenAttribute::.ctor() = ( 01 00 00 00 ) 

  // Code size       50 (0x32)

  .maxstack  3

  IL_0000:  br         IL_0007

  IL_0005:  pop

  IL_0006:  ldc.i4.0

  .try

  {

    IL_0007:  call       bool [Microsoft.VisualBasic]Microsoft.VisualBasic.ApplicationServices.WindowsFormsApplicationBase::get_UseCompatibleTextRendering()

    IL_000c:  call       void [System.Windows.Forms]System.Windows.Forms.Application::SetCompatibleTextRenderingDefault(bool)

    IL_0011:  ldtoken    EIOtYCk427B2j46XCZ.aC4npiTU1ygUa7KS3q

    IL_0016:  call       class [mscorlib]System.Type [mscorlib]System.Type::GetTypeFromHandle(valuetype [mscorlib]System.RuntimeTypeHandle)

    IL_001b:  call       void [System]System.ComponentModel.LicenseManager::Validate(class [mscorlib]System.Type)

    IL_0020:  leave      IL_0026

  }  // end .try

  finally

  {

    IL_0025:  endfinally

  }  // end handler

  IL_0026:  call       class EIOtYCk427B2j46XCZ.aC4npiTU1ygUa7KS3q usJHEvC4iTtNdZDPye.bhVRye8yqVI1niw4o6::vaCk4npiU()

  IL_002b:  ldarg.0

  IL_002c:  callvirt   instance void [Microsoft.VisualBasic]Microsoft.VisualBasic.ApplicationServices.WindowsFormsApplicationBase::Run(string[])

  IL_0031:  ret

} // end of method aC4npiTU1ygUa7KS3q::S9oGE4FMD
```

`Validate`应该就是许可证验证函数了吧，不过网上说这种.net许可证机制好像不能直接跳过去，因为后面会用其中的一些信息数据，这里一直没有弄明白，哪位了解这种许可证机制的给解释一下能不能直接跳过吧![](https://bbs.kanxue.com/view/img/face/50.gif)。用`PEBrowsePro`跟入`Validate`并没有直接跳到`System.dll`空间，这点倒是和纯粹PE文件的函数调用大不相同啊，并且执行中总是提示各种执行异常。

无奈还是直接把`validate`函数去掉试试吧，不过得直接在二进制文件上改（`ILDASM`导出的`xx.il`文件用`ILASM`虽然编译成功但执行时会弹出错误）。用`IDA`打开`exe`文件，找到两个函数的调用字节码的文件偏移，再用`ultraedit`打开`exe`填写`0x00`（`nop`）覆盖对应位置。不过执行后还是会弹出许可证那个窗口，看来程序在运行中还多次调用了`Validate`函数，静态找不到其他的调用迹象，动态又不太顺利。

用`DotNet_Id`的查了一下壳，天啊![](https://bbs.kanxue.com/view/img/face/33.gif)，原来是加过壳啊，难怪分析起来这么费劲呢。

![](https://bbs.kanxue.com/upload/attach/201103/208207_s8kysqp1bkb15f9.jpg)

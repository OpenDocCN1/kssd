
```
BOOL CheckVirtualPC()  

{  

    bool rc = TRUE;  

    __try  

    {  

        __asm  

        {  

            push ebx  

            mov ebx, 0  

            mov eax, 1  

            __emit 0fh  

            __emit 3fh  

            __emit 07h  

            __emit 0bh  

            test ebx, ebx  

            setz[rc]  

            pop ebx  

        }  

    }  

    __except(IslnsideVPC_exceptionFilter(GetExceptionInformation()))  

    {  

        rc = FALSE;  

    }  

    return rc;  

}
```

## 三、基于社会工程学的技巧

### 1.检测电脑中常用软件的使用情况

名为`Intelligent Software Solutions Inc.doc`的恶意软件样本文件使用了下面的反虚拟机技巧，SHA256值为`048fc07fb94a74990d2d2b8e92c099f3f986af185c32d74c857b07f7fcce7f8e`。

![](https://bbs.kanxue.com/upload/attach/201804/734571_CAYQBZX6ZX34KUR.png)

`RecentFiles`对象表示系统最近打开过的历史文档。通常，安装了word程序的用户可能会打开超过2个或更多数量的文档。然而，当该恶意软件植入到新创建的虚拟机和word环境中后，总是状况不断，不能正常运行。每次测试时手动打开一两次，总是出现程序异常。即使是保存了虚拟机镜像状态，重启调试分析后，恶意程序仍然不能正常执行。从`DKTxHE`函数功能可以看出，恶意软件以`RecentFiles`数量来判断是否身处VM环境中，如果在VM环境中，它将不会执行任何恶意行为。之后，随意创建了3个不同名称的word文档，逐一打开并关闭，让历史文档数量为3，最终成功运行并检测到了恶意软件。

### 2.探测杀毒软件公司相关的IP地址

同样是上面的恶意软件，它在另一个子程序中使用了下面的反虚拟机技巧。首先，它通过向远程地址`2dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0j5i4S2E0K9h3&6V1i4K6u0W2j5$3!0E0i4K6u0r3k6$3g2G2K9i4m8Q4x3V1k6$3x3W2)9J5k6e0q4Q4x3V1k6U0K9i4c8&6i4K6u0r3L8h3g2Q4c8e0g2Q4z5p5k6Q4z5e0q4Q4c8e0g2Q4z5o6N6Q4b7V1q4Q4c8e0k6Q4z5f1k6Q4z5e0m8Q4c8e0N6Q4b7e0N6Q4z5p5c8Q4c8e0S2Q4b7f1g2Q4b7e0c8Q4c8e0S2Q4b7f1k6Q4z5o6q4Q4c8e0S2Q4b7f1k6Q4b7U0N6Q4c8e0k6Q4b7U0q4Q4z5o6u0Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0c8Q4b7U0W2Q4z5p5u0Q4c8e0g2Q4z5e0m8Q4z5p5g2Q4c8e0S2Q4b7f1g2Q4b7V1g2Q4c8e0N6Q4b7V1c8Q4b7f1g2Q4c8e0S2Q4b7f1k6Q4b7U0N6Q4c8e0k6Q4b7U0q4Q4z5o6u0Q4c8e0c8Q4b7V1k6Q4b7e0q4Q4c8e0k6Q4z5o6q4Q4b7f1k6Q4c8e0c8Q4b7U0S2Q4b7f1c8Q4c8e0N6Q4z5f1q4Q4z5o6c8t1g2q4c8b7`Refer属性和`User-Agent`值，访问链接`026K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0j5i4S2E0K9h3&6V1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3V1k6D9L8$3y4S2N6r3g2Q4x3X3c8E0P5g2)9J5k6r3W2H3i4K6u0V1j5h3c8V1M7X3g2K6M7#2!0q4y4q4!0n7b7W2!0m8y4g2!0q4y4W2!0m8c8q4!0m8y4q4!0q4z5q4)9^5c8g2!0n7y4#2!0q4y4g2)9^5c8W2)9&6y4W2!0q4y4g2!0m8c8g2!0n7c8W2!0q4y4q4!0n7z5q4!0n7b7W2!0q4y4#2!0n7x3#2!0n7b7W2!0q4y4#2!0n7b7W2)9&6c8W2!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4#2!0n7b7W2)9^5y4q4!0q4y4#2!0n7b7W2)9^5y4#2!0q4y4W2)9&6b7#2!0n7b7g2!0q4y4W2)9&6c8g2)9^5y4q4!0q4y4#2!0m8c8q4)9^5z5g2!0q4y4q4!0n7c8W2!0m8x3g2!0q4y4W2)9^5x3g2!0m8c8W2!0q4x3#2)9^5x3q4)9^5x3V1W2b7i4@1f1@1i4@1u0r3i4@1p5I4i4@1f1$3i4K6R3I4i4@1q4r3i4@1f1%4i4K6W2m8i4K6R3@1L8%4u0Y4j5h3&6A6P5X3q4@1K9h3!0F1i4@1f1#2i4@1q4p5i4K6V1%4i4@1f1$3i4@1q4q4i4@1t1#2i4@1f1$3i4K6V1^5i4@1u0q4i4@1f1%4i4@1p5@1i4@1u0m8i4@1f1@1i4@1t1^5i4@1u0m8i4@1f1%4i4@1u0q4i4K6S2q4i4@1f1#2i4K6W2n7i4@1u0p5b7$3!0E0j5$3q4K6N6q4!0q4y4g2!0m8c8g2!0n7c8q4!0q4y4g2!0n7z5q4!0m8y4W2!0q4y4#2!0n7c8q4)9&6x3g2!0q4y4#2!0n7b7W2)9&6b7#2!0q4y4q4!0n7c8g2)9&6b7W2!0q4y4g2!0n7b7g2)9&6y4q4!0q4y4g2)9&6y4g2)9^5y4W2!0q4x3#2)9^5x3q4)9^5x3W2!0q4y4W2)9^5x3g2!0n7y4W2!0q4y4W2)9^5y4q4)9^5c8W2!0q4z5q4!0n7c8q4!0m8c8W2!0q4y4q4!0n7b7W2!0n7y4W2!0q4y4g2)9^5c8W2)9&6x3g2!0q4y4g2)9^5y4#2!0n7b7g2!0q4z5q4!0m8c8g2!0n7c8W2!0q4z5g2)9&6y4#2!0m8c8g2!0q4z5q4!0m8c8W2!0n7y4#2!0q4y4W2!0n7x3g2)9^5x3W2!0q4y4g2)9&6x3q4)9^5c8g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4z5q4)9^5c8g2!0n7y4#2!0q4y4g2)9^5c8W2)9&6y4W2!0q4y4g2)9^5z5q4!0n7x3q4!0q4y4g2!0m8c8g2!0n7c8W2!0q4y4q4!0n7z5q4!0n7b7W2!0q4y4#2!0n7x3#2!0n7b7W2!0q4y4#2!0n7b7W2)9&6c8W2!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4#2)9&6b7W2!0n7z5q4!0q4y4g2)9^5y4g2!0n7x3#2!0q4y4q4!0n7c8W2!0m8x3g2!0q4y4W2)9^5x3g2!0m8c8W2!0q4y4g2!0n7x3q4)9^5y4W2!0q4y4g2!0m8c8q4)9&6z5q4!0q4y4g2)9^5x3W2!0m8z5q4!0q4y4q4!0n7b7g2)9^5c8g2!0q4y4W2)9&6c8W2)9&6x3q4!0q4y4q4!0n7z5q4!0m8b7g2!0q4y4W2)9&6y4g2!0n7x3q4!0q4y4#2!0n7b7W2)9^5y4q4!0q4y4q4!0n7z5q4!0m8c8q4!0q4x3#2)9^5x3q4)9^5x3W2!0q4y4g2!0m8y4W2)9^5x3W2!0q4y4W2)9&6c8g2)9&6b7#2!0q4z5q4)9^5c8g2!0n7y4#2!0q4y4g2)9^5c8W2)9&6y4W2!0q4y4g2)9^5z5q4!0n7x3q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4#2!0n7b7W2)9^5y4q4!0q4y4#2!0n7b7W2)9^5y4#2!0q4y4W2)9&6b7#2!0n7b7g2!0q4y4W2)9&6c8g2)9^5y4q4!0q4y4g2)9&6x3q4)9^5c8q4!0q4y4#2!0m8y4#2!0n7x3q4!0q4y4q4!0n7z5q4)9^5c8f1A6e0e0@1&6Q4c8e0k6Q4z5e0k6Q4z5o6N6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0c8Q4b7U0S2Q4b7f1c8Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0c8Q4b7V1u0Q4b7V1u0Q4c8e0c8Q4b7V1c8Q4z5e0g2Q4c8e0k6Q4z5f1y4Q4b7V1q4Q4c8e0k6Q4z5f1g2Q4z5o6c8Q4c8e0g2Q4b7f1c8Q4z5e0N6Q4c8e0N6Q4b7f1y4Q4b7e0k6Q4c8e0c8Q4b7U0S2Q4b7U0u0Q4c8e0g2Q4z5p5y4Q4b7U0W2Q4c8e0W2Q4z5o6g2Q4z5p5c8Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0k6Q4z5o6q4Q4b7U0k6Q4c8e0k6Q4z5o6c8Q4z5p5k6Q4c8e0S2Q4b7V1c8Q4b7f1k6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0g2Q4b7U0m8Q4z5o6k6Q4c8e0g2Q4z5p5k6Q4z5e0q4Q4c8e0N6Q4z5e0c8Q4z5f1k6Q4c8e0g2Q4b7V1y4Q4z5o6u0Q4c8e0g2Q4b7U0S2Q4b7U0S2Q4c8e0g2Q4b7U0W2Q4b7U0k6Q4c8e0g2Q4z5o6q4Q4z5f1y4Q4c8e0k6Q4b7f1c8Q4b7e0u0Q4c8e0S2Q4b7V1k6Q4z5e0m8Q4c8e0S2Q4b7e0q4Q4z5p5y4Q4c8e0y4Q4z5o6m8Q4z5o6u0Q4c8e0g2Q4b7V1c8Q4z5e0y4Q4c8e0N6Q4z5o6c8Q4b7U0k6Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0g2Q4z5o6S2Q4z5e0N6Q4c8e0S2Q4b7e0q4Q4b7e0S2Q4c8e0c8Q4b7U0S2Q4b7f1c8Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0k6Q4z5f1y4Q4b7V1q4Q4c8e0k6Q4z5f1g2Q4z5o6c8Q4c8e0g2Q4z5e0m8Q4z5p5c8Q4c8e0N6Q4b7e0N6Q4b7U0m8Q4c8e0g2Q4z5f1y4Q4b7e0S2Q4c8e0c8Q4b7V1u0Q4b7e0y4Q4c8e0N6Q4b7e0m8Q4z5o6q4Q4c8e0c8Q4b7U0S2Q4b7f1c8Q4c8e0k6Q4z5e0S2Q4b7f1k6Q4c8e0N6Q4b7V1u0Q4z5p5k6Q4c8e0S2Q4b7V1k6Q4z5o6N6Q4c8e0k6Q4b7U0N6Q4b7U0N6Q4c8e0k6Q4b7U0N6Q4z5o6k6Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0y4Q4z5o6m8Q4z5o6t1`.

![](https://bbs.kanxue.com/upload/attach/201804/734571_JCBCMSAXS9KJCRU.png)

## 四、虚拟机逃逸

VMware等软件中或多或少都存在一些安全漏洞，可以利用这些漏洞使宿主操作系统崩溃或者是在宿主操作系统中运行代码。当主机系统被感染后，一些公开可用的工具可以用来对VMware等软件进行攻击。

## 五、总结

当遇到的恶意代码似乎不能运行时，在使用调试或反汇编恶意代码搜索其反虚拟机探测代码之前，应该考虑使用一个卸载了VMware Tools的虚拟机。VMware中有一些未文档化的功能可以帮助减轻反虚拟机技术的探测。将下面的代码放到VMware的`.vmx`文件中，以减轻虚拟机被探测的可能。

```
isolation.tools.getPtrLocation.disable = "TRUE" 

isolation.tools.setPtrLocation.disable = "TRUE" 

isolation.tools.setVersion.disable = "TRUE" 

isolation.tools.getVersion.disable = "TRUE" 

monitor_control.disable_directexec = "TRUE" 

monitor_control.disable_chksimd = "TRUE" 

monitor_control.disable_ntreloc = "TRUE" 

monitor_control.disable_selfmod = "TRUE" 

monitor_control.disable_reloc = "TRUE" 

monitor_control.disable_btinout = "TRUE" 

monitor_control.disable_btmemspace = "TRUE" 

monitor_control.disable_btpriv = "TRUE" 

monitor_control.disable_btseg = "TRUE"
```

参数`directexec`可以使用户模式下的代码被模拟执行而不是直接在硬件上运行，因此它可以挫败一些反虚拟机技术。前四条设置被VMware后门命令使用，它们的作用是使得运行在Guest系统中的VMware Tools不能获取宿主系统的信息。这些设置会禁用VMware Tools的一些有用功能，并可能对虚拟机性能有严重负面影响。所以，仅当其他技术无效时再添加这些选项。当然，也可以将恶意代码在其他虚拟环境或者物理主机上运行。同反调试技术一样，要想发现恶意代码中的反虚拟机技术需要在长期调试过程中积累更多经验。例如，看到一个代码在一个条件跳转处过早终止，这可能就是反虚拟机技术造成的结果。一如既往地警惕这种类型的问题，然后查看其之前的代码，来确定它到底执行了什么操作。和反调试技术一样，通过修改条件跳转指令或者使用NOP指令覆盖来绕过相关探测。最后让我们总结一下提到的内容。腾讯2016游戏安全技术竞赛有一道题，大概意思就是给一个exe，要求编写一个`Tencent2016C.dll`，并导出多个接口函数`CheckVirtualPCX`、`CheckVMWareX`、`CheckVirtualBoxX`。X为1-100之间的数字。函数功能是检测自己是否处于相应的虚拟机中，是返回`TRUE`，否则返回`FALSE`。函数的原型都是`typedef BOOL (WINAPI* Type_CheckXXXXXX)();`。编译好dll之后，放在`Tencent2016C.exe`的同目录，运行`Tencent2016C.exe`，点击检测按钮，在物理机中运行时函数接口输出为0，在VMware虚拟机、VirtualBox虚拟机和VirtualPC虚拟机中运行时，相关的接口输出1。我们把提到的知识综合一下完成这道题目。

解题的参考代码和题目相关信息：`f67K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5L8%4g2B7K9h3&6Y4P5h3V1J5x3K6y4Q4x3V1k6@1k6i4y4@1i4K6u0V1N6X3W2J5N6s2g2S2L8q4)9J5k6r3#2S2j5$3S2A6L8X3g2Q4x3V1j5`.

参考资料：

1.《恶意代码分析实战》第17章反虚拟机技术(本文的主体框架)

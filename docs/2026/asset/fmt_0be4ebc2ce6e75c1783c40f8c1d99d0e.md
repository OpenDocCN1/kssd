
顺便提一句，`ntoskrnl.exe`提供了许多基本的字符串函数，如`strcmp`、`strcpy`和`strlen`等。

```
 memset(@szModuleName, 0, sizeof(szModuleName));

              strcpy(@szModuleName, pModuleName);

              _snprintf(@buffer, sizeof(buffer),

                        'SystemModules: Found %s base: %08X size: %08X',

                        @szModuleName,

                        (PSYSTEM_MODULE_INFORMATION(pTemp))^.Base,

                        (PSYSTEM_MODULE_INFORMATION(pTemp))^._Size);

              strcat(pMessage, @buffer);

            end;
```

假如上文提及的模块被找到，我们就用`_snprintf`（内核提供）函数格式化字符串，字符串中包含了模块名、基地址和尺寸，然后将字符串加到`buffer`去。

```
 if pMessage[0] <> #0 then

          begin

             DbgPrint(pMessage);

          end else

          begin

             DbgPrint('SystemModules: Found neither ntoskrnl nor ntice');

          end;
```

这里很容易看懂，由于我们前面把字符串都置了零，这里很容易判断我们是否找到了什么东西。

```
 ExFreePool(pMessage);

          DbgPrint('SystemModules: Memory at address %08X released', pMessage);

        end;

      end;

      ExFreePool(p);
```

释放在系统内存池里分配的内存。

```
result := STATUS_DEVICE_CONFIGURATION_ERROR;
```

返回失败代码，这样强制系统将驱动程序从内存中卸载。

现在你清楚了吧，使用系统内存池比使用用户模式的堆简单多了。唯一需要注意的问题就是正确地定义内存池类型。

用户模式下的`ntdll.dll`提供了许多`ZwXxx`系列函数，他们是进入内核模式的大门。注意：他们的参数的数量和含义都是一样的。你可以省不少事儿了吧！

由于内核的错误会导致系统瘫痪，所以你可以在用户模式下调试，然后小小地改动（如果需要）后拷贝到你的驱动程序。例如：`ntdll.dll`的`ZwQuerySystemInformation` 调用返回同样的信息。使用这个技巧你就不用总是重新启动你的机器了。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`KmdKit4D_patch0119.rar`](javascript:void(0);)

  （974.45kb，585次下载）

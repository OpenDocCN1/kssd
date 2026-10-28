
Net Trace 名：`L"SYSMON TRACE"`; 或者使用系统的`L"NT Kernel Logger"`;

方法参考微软官方实例：`0bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6V1k6i4y4C8N6r3!0H3i4K6u0r3k6i4c8%4i4K6u0r3j5$3!0F1k6X3W2Y4N6i4u0A6L8X3N6Q4x3X3c8S2L8X3c8Q4x3X3c8K6N6r3q4J5N6r3W2F1k6#2)9J5k6s2c8Z5k6g2)9J5k6r3&6@1i4K6u0V1K9$3g2J5L8X3g2D9i4K6u0V1L8r3!0Y4k6$3g2J5i4K6u0V1M7$3g2K6M7$3W2G2L8R3`.

![](https://bbs.kanxue.com/upload/attach/201809/36653_SRR5SQS5H27JRF7.jpg)

事件回调`EventCallBack()`接受数据

![](https://bbs.kanxue.com/upload/attach/201809/36653_76TBDUVKW8MYWKC.jpg)

在解析数据时使用的是 WMI Mof 的方法来解析

![](https://bbs.kanxue.com/upload/attach/201809/36653_WCNVS3J2A6BBDVP.jpg)

![](https://bbs.kanxue.com/upload/attach/201809/36653_JG4ZQX8PK6EZVKF.jpg)

可以参考微软的官方例子：

`d06K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6V1k6i4y4C8N6r3!0H3i4K6u0r3M7X3g2@1M7X3W2W2N6X3W2F1k6#2)9J5k6r3g2$3k6h3&6@1i4K6u0V1k6r3q4@1j5g2)9J5k6s2g2K6K9h3&6Y4i4K6u0V1L8h3!0X3`

### 第三步：接受上报数据写入 windows 的 Application 日志

在第二步中我们可以看到通过 `ReportEventWriteEvent` 函数上报信息，在 `ReportEventWriteEvent` 函数里分两种情况系统 API 上报

![](https://bbs.kanxue.com/upload/attach/201809/36653_QDQ7AST34XAB3B8.jpg)

通过 `ReportEvent` 或者 `EventWrite` 两个其中一个 API 上报，而上报的事件 IDD 类都是前面我们看到的 Sysmon 自己注册到系统的里 `<provider name="Microsoft-Windows-Sysmon" guid="{5770385F-C22A-43E0-BF4C-06F5698FFBD9}" symbol="SYSMON_PROVIDER" resourceFileName="%filename%" messageFileName="%filename%">`

`<events>` 的 Microsoft-Windows-Sysmon 事件代理，这个会生成到 windows 日志的 Application 项目下，具体会使用哪个 API 是根据 windows 的版本来选择的

![](https://bbs.kanxue.com/upload/attach/201809/36653_4P2W5VKS5ZCAEYH.jpg)

这里可以看到如果操作系统主版本，如果是 vista 之前的操作系统使用 `ReportEvent`，如果是 vista 以及以上操作系统则使用 `EventWrite` 函数。

Sysmon 记录上报了数据源通过注册的 WMIEvent 的 wmi 数据持久化过滤事件去过滤不会被记录的事件，我们下面看它如何实现的。

![](https://bbs.kanxue.com/upload/attach/201809/36653_G4498DAWEZ7N8ZW.jpg)

在之前的服务启动入口有一个函数 `RegisterWmiEvent`，该函数就是注册过滤 WmiEvent 的函数，我们继续往下看

![](https://bbs.kanxue.com/upload/attach/201809/36653_ZXGNKTH2T98QFU3.jpg)

函数开头会创建实例 `IDD_WebCImv`，class Id: `IID_IWbemLocator` `GUID2`

`<0dc12a687h,0737fh,011cfh,088h,04dh,000h,0aah,000h,04bh,02eh,024h>`

链接 `ROOT\\Subscription` 的服务

接着创建 Stub 插口

![](https://bbs.kanxue.com/upload/attach/201809/36653_3WC54ME6WJNBMP3.jpg)

`g_WMIListenerEvent` 接口类型是 `IWbemObjectSink`, 其定义如下

```
MIDL_INTERFACE("7c857801-7381-11cf-884d-00aa004b2e24")
IWbemObjectSink : public IUnknown
{
public:
    virtual HRESULT STDMETHODCALLTYPE Indicate(
        /* [in] */ long lObjectCount,
        /* [size_is][in] */ __RPC__in_ecount_full(lObjectCount) IWbemClassObject **apObjArray) = 0;
    virtual HRESULT STDMETHODCALLTYPE SetStatus(
        /* [in] */ long lFlags,
        /* [in] */ HRESULT hResult,
        /* [unique][in] */ __RPC__in_opt BSTR strParam,
        /* [unique][in] */ __RPC__in_opt IWbemClassObject *pObjParam) = 0;
};
```

![](https://bbs.kanxue.com/upload/attach/201809/36653_UT5GAYHQDHU2CVH.jpg)

然后执行

`"SELECT * FROM __InstanceOperationEvent WITHIN  5  WHERE TargetInstance ISA '__EventConsumer' OR Tar"`

`"getInstance ISA '__EventFilter' OR TargetInstance ISA '__FilterToConsumerBinding'"`

```
g_WmiSubscriptProxy->lpVtbl->ExecNotificationQueryAsync(
       g_WmiSubscriptProxy,
       strQueryLanguage,
       strQuery,
       128,
       0,
       (IWbemObjectSink *)g_pIWebmObjectSink);
```

去设置 WMI Event 的过滤事件，操作类型是所有操作 InstanceOperationEvent，设置三种事件 `EventConsumer'`、`EventFilter'`、`FilterToConsumerBinding'`，查询时间是 5 秒一次，这样就注册了。

下面我们看 `g_WMIListenerEvent` 结构

![](https://bbs.kanxue.com/upload/attach/201809/36653_QXJ5CKPNCPWCW6K.jpg)

过滤事件就是在 `Indicate` 函数中实现，会通过 `IWbemClassObject**` 数组的形式输入，函数内会枚举数据，如果是要过滤的数据则循环枚举否则中断枚举。

![](https://bbs.kanxue.com/upload/attach/201809/36653_FZ5UA6K4D8Y3MTD.jpg)

至此第一篇对 sysmon 的 ring3 的大致原理流程我们分析完毕，通过对它分析，学习它的实现过程，可以自己完成实现一个 sysmon（还有驱动部分第二篇讲解），当然也可以绕过 sysmon 的监控，这就需要读者自己去研究与发现，第二篇我会讲解驱动部分的分析。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

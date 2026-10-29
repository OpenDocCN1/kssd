
动态污点分析则是在程序运行中过程，通过跟踪变量、内存及寄存器的值，依据执行流程跟踪污点数据的传播，最后达到跟踪攻击路径与获取漏洞信息的目的，以TaintDroid为例，需要对Android 系统进行定制化修改以便对 App 运行过程中的污点数据进行跟踪，但由于Android 设备或模拟器性能瓶颈问题，动态污点分析无法获取 App 所有可能的可执行路径。

以往污点分析常用于Android 恶意 App 分析，信息泄露检测等问题，现在越来越多的厂商也应用于App 漏洞检测上（如阿里聚安全使用的基于TaintDroid方案）。相比传统的App 漏洞检测，污点分析可以跟踪污点数据的传播过程，确定漏洞是否在实际环境中可能被触发，检测能力更加强大。目前也有不少开源的Android 污点分析方案与工具如：TaintDroid、FlowDroid、amadroid 及 CodeInspect，在此之上也发展出一些针对 Android App 漏洞检测方面的工具，如腾讯科恩实验室 Flanker 之前开源的 JAADS（[https://github.com/manfiS/droidfuzzer](https://github.com/manfiS/droidfuzzer)）/MFFA。

![](https://bbs.kanxue.com/upload/attach/201609/639384_ce813e85a6a79ab0.png)

图4 Flowdroid运行流程

### 1.3.3 Android App 通用脱壳

App 二次打包，破解等问题的泛滥催生了 App 加固产业的发展，两者技术在也攻防中不断发展和进化，目前国内主流的加固方案有梆梆，爱加密，百度加固，360加固，阿里聚安全，腾讯御安全/乐固，通付盾，NAGA 等，对于在线漏洞检测平台来说，如果没有通用的自动化脱壳方案就意味无法对应用市场中的很多 App 代码进行静态分析，甚至由于应用被加固无法运行于模拟器中或特定测试设备中，影响动态分析结果。

目前针对 dex 文件加固主流的脱壳方法有静态脱壳、内存 dump、Dalvik 虚拟机插桩，第一种方法需要针对厂商加密方案不断更新，基本不具备通用性，后面两者的开源代表作有：ZjDroid 与 DexHunter。

![](https://bbs.kanxue.com/upload/attach/201609/639384_f3dec2fda54af49d.png)

图5 笔者整理的国内部分APP加固方案

### 1.3.4 移动 App UI 自动化遍历

App UI 自动化遍历常在 App 开发测试中用于检测 App 性能与兼容性，目前在 App 漏洞检测领域用的比较少（主要由于效率较低）。一般主流的 App 漏洞检测平台都包含动态分析，主要是安装 App 后自动运行 App 并监测 App 行为，通常使用的是Android Monkey test 脚本或其它工具随机点击 APP 界面。

实际上，为了更深入的检测 App 敏感信息泄露与后端 WEB 接口漏洞，仅靠随机点击 App 界面进行动态分析是不够的（比如现在大部分 App 功能需要注册登录后才能使用），如果能更好的模拟正常用户使用 App 的过程，则可以扩展监测 Logcat 日志，本地文件存储，网络通信等数据审计能力。

目前 APP UI 测试框架按原理可分为黑盒与白盒两种，白盒测试需要在 App 开发时添加测试组件并调用，需要 App 完整源码，黑盒测试一般提取 App 界面的 UI 元素并根据算法进行遍历，无需 App 源码。笔者推荐的黑盒 APP UI 测试框架与工具主要有 AndroidViewClient 及国人基于 Appium 开发的 appCrawler。

![](https://bbs.kanxue.com/upload/attach/201609/639384_50346c85e5819c7f.png)

图6 appCrawler生成的APP测试报告

## 二、Android App 漏洞检测

### 2.1 国内 Android App 在线漏洞检测平台

- 腾讯金刚审计系统：免费、无限制
- 腾讯御安全
- 阿里聚安全
- 360显微镜

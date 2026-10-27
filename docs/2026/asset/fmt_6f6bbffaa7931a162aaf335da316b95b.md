
# 六、如何绕过主动防御，行为查杀

1、白加黑，利用杀软可执行文件EXE白名单，有些白名单内的EXE存在DLL劫持漏洞，可以编写黑dll，插入shellcode实现劫持，达到表面免杀和行为查杀。

2、至于如何查找白名单EXE，我已经写好了Python脚本，一键查找，需要的私信我。交流加我微信（BASE64）：`Mzk2MjM2ODA2`

# 七、实现效果：

只要进程和文件名称包含`$77`，即在被注入的进程中达到隐藏。这是R77 hook内被人的进程的系统函数。

注入R77前：

![](https://bbs.kanxue.com/upload/attach/202305/523507_E56Z2GQ93MP8GE6.jpg)

注入之后：进程和文件消失。并且无法查杀R77 DLL的主文件。

![](https://bbs.kanxue.com/upload/attach/202305/523507_VCRQ59EMBMEU3A4.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`PELoader.asm`](javascript:void(0);)

  （18.31kb，125次下载）

- [`远程注入R77.zip`](javascript:void(0);)

  （397.18kb，152次下载）

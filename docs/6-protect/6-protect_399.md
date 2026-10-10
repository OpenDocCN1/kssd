

# 修复 SafeEngine 针对 OllyICE 的溢出补丁

> 原文：[`www.pediy.com/kssd/pediy11/105379.html`](https://www.pediy.com/kssd/pediy11/105379.html)

知道啥叫出师未捷身先死么？。。。  
今天看见了 SafeEngine，好吧，我承认我很菜，我只是下载下来占硬盘玩的。。。  
  
OD 一载入就跳出，又没有 TLS，所以就分析了一下，原来是溢出了，补丁如下：  
  
{再}好吧，我承认我的英语也很烂，如果补丁说明里面有语法错误请见谅。。。  
  
受 SafeEngine 保护的文件会在输入表的第一个 DLL 和函数的名称后面加上很多的 0D0Ah，这样就导致了 Ollydbg 在格式化生成"DLL.FuncionName"(比如 Kernel32.GetProcAddress)的时候产生了溢出。打上这个补丁就可以解决这个问题。具体使用的方法是在调用格式化函数的时候为其添加了一个参数检查，看见了 0D0Ah 就改成 0。不过万一这些被 SafeEnging 看见了咋办……

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || ODBG\_patcher.rar  
 |
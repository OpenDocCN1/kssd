
60022e6a e811000000 `call` `___!__tmainCRTStartup (60022e80)`  

`___!__security_init_cookie:`  

600312f0 8bff `mov edi,edi`  

600312f2 55 `push ebp`  

600312f3 8bec `mov ebp,esp`  

600312f5 83ec18 `sub esp,18h`  

600312f8 c745f800000000 `mov dword ptr [ebp-8],0`  

600312ff c745fc00000000 `mov dword ptr [ebp-4],0`  

60031306 813d38d108604ee640bb `cmp dword ptr [___!__security_cookie (6008d138)],0BB40E64Eh`  

...........  

里面会引用许多 `security cookie` 的全局变量，而这些变量没有在重定向表里，所以如果加载的 地址不是 PE 头种的 `ImageBase`，就会出现访问错误，导致崩溃。  

解决办法：  

不要加载到 PE 头的 `ImageBase` 以外的地址。那如果跟我们的 `EXE` 地址冲突怎么办？ 解决办法就是我们编译的模块选一个加载的基地址。  

这里我选的是 `0x5FFF0000`，这个地址就可以避免跟要 `RUN` 的 `EXE` 模块的基地址冲突了，另外为了能在 `WIN` `7` 跑，还需要 `Disable Image Randomization (/DYNAMICBASE:NO)`。 为了避免 `DEP` 而导致出错，所以分配的内存的属性一律改为  

`PAGE_EXECUTE_READWRITE`

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `RunPE.rar`  
|

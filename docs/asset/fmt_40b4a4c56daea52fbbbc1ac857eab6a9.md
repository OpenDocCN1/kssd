
> `SOFTWARE\Microsoft\Windows\CurrentVersion\ShellServiceObjectDelayLoad`  
>  
> 下面带有 `AceExt32.dll` 和 `ZipExt32.dll` 的请删了  
>  
> `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\SM_GameDro`  
>  
> `HKEY_CLASSES_ROOT\CLSID\{35CEC8A3-2BE6-11D2-8773-92E220524150}`  
> `HKEY_CLASSES_ROOT\CLSID\{35CEC8A3-2BE6-11D2-8773-92E220524140}`  
>  
> 3. 关闭自动运行  
>  
> 开始 -> 运行 -> `Gpedit.msc` -> 计算机配置 -> 管理模块 -> 系统 -> 关闭自动播放 -> 已启动 -> 所有驱动器 -> 确定 OK~  
>  
> 4. 插入移动存储器，鼠标右键打开，删除里面的病毒程序：  
> - `X:\RECYCLER\UcHelp.exe`  
> - `X:\RECYCLER\desktop.ini`  
> - `X:\autorun.inf`  
>  
> 重启电脑应该就没有事了！  
>  
> BTW：当然你不愿意进安全模式，那么强行卸载 Explorer 中那两个刀 `AceExt32.dll`、`ZipExt32.dll`，再去删除那些文件，清理那些注册表也是可以的。  
>  
> \--------------------------------------------------------------------------------  
> 【经验总结】  
> 1. 第一次分析病毒程序，感觉怕怕，为此我还装了影子系统  
> 2. 感谢 恶灵骑士 MJ0011 介绍 `sysret.sys` 的工作机理  
> 3. 感谢 xyzreg 大虾提供的强奸注册表 `Pass HIPS-RD` 方法  
> 4. 好了，我可以去吐血了~  
>  
> \--------------------------------------------------------------------------------  
> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
>  
> 2007 年 06 月 03 日 上午 10:20:04  
>  
> `4dd466d0e1e0cc123afbe504da2462ca`

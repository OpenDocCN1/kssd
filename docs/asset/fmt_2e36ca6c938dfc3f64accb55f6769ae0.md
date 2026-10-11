
```
Result := WriteFile(hDriver, Query, SizeOf(TDriverQuery), Bytes, nil);  

end else Result := TrueDumpProcess(dwProcessId, pStartAddr,  

                              dwcBytes, pDumpedBytes);  

end;  
```

--------------------------------------------------------------------------------

eXtreme dumper 可以分为以下部分：

- 两种 dump 的方法（`Dll-Injection` 法和驱动法）
- 获取句柄的隐式方法（`OpenProcessEx`）
- 防止 PE Tools 受其它进程影响（`Protect PE Tools`）
- 对内核模块的 dump（`Enable kernel modules dumping`）

简言之，这个 plugin 无疑是很有用的。

--------------------------------------------------------------------------------

我想提醒的是，写破解程序这类文章的作者们请不要进行违反俄联邦法规的活动。本文内容只能用于教学目的，对任何由此引起的非法行为，本作者不负有任何责任。（对使用本译文进行违反中华人民共和国法规的行为，译者亦不负任何责任）

`[C] Ms-Rem`

greatdong 译

http://greatdong.blog.edu.cn

`4dd466d0e1e0cc123afbe504da2462ca`

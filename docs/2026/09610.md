
# [原创]Lua5.3多线程版本（Windows 32）

先贴一下运行效果：

![](https://bbs.kanxue.com/upload/attach/201503/580622_nnusgt66b3bwnsa.jpg)

（看文章下面已经有新的版本了！！！）

早就看到 Lua 可以支持多线程的帖子，但也没见人发布一个多线程的 Lua 解释器。于是自己根据前人的经验编译了一个。本来想发在 Lua 中国开发者这个论坛上，不过看上面没大有讨论 Lua 技术的，基本都是游戏之类的，而且我的密码也忘了，多次找回密码都没反应。所以来到了看雪，我的家。让小伙伴们多了解一下 Lua 这个小巧而又强大的脚本语言。

修改方法参考的这个帖子：330K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4F1j5X3I4G2k6%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6\*7K9r3q4F1k6$3c8G2L8X3N6K6K9r3g2F1k6#2)9J5c8Y4m8Q4x3V1j5K6y4U0M7&6x3o6t1@1i4K6u0W2K9s2c8E0L8l9`.`.

源代码来自于 Lua 官方网站：9d6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3I4#2j5g2)9J5k6h3!0J5k6#2)9J5c8R3`.`.

我修改后的代码放在了 Github 上：b6aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9N6i4A6Z5L8r3!0F1i4K6u0r3e0s2g2S2y4e0y4y4g2l9`.`.

感兴趣的一起来和我交流，给我提建议也好。

## 多线程的 Lua 解释器: （下面有新的！！！！）

## 更新

### 2015-3-28

新版本的运行截图：

![](https://bbs.kanxue.com/upload/attach/201503/580622_h5hubormu7gq6mz.jpg)

这次主要将 `newthread` 和 `startthread` 这两个全局函数去掉，改为加入了 `systhread` 这个库。

测试代码：

```lua
local function ff1(n)

    for i = 1, n do

        print('Thread 1 --> \t'..i)

    end

end

local function ff2(n)

    for i = 1, n do

        print('Thread 2 --> \t'..i)

    end

end

local function ff3(n)

    for i = 1, n do

        print('Thread 3 --> \t'+i)--Error code

    end

end

local function ff4(n)

    for i = 1, n do

        print('Thread 4 --> \t'..i)

        --systhread.sleep(5)

    end

end

local st1 = systhread.create(ff1, 4)

local st2 = systhread.create(ff2, 6)

local st3 = systhread.create(ff3, 8)

st1:resume()

st2:resume()

st3:resume()

st1:wait() print('Thread 1 exitcode : '..st1:exitcode())

st2:wait() print('Thread 2 exitcode : '..st2:exitcode())

st3:wait() print('Thread 3 exitcode : '..st3:exitcode())

print ''

print 'Create st4 ...'

local st4 = systhread.create(ff4, 10)

print 'Resume st4 ...'

st4:resume()

systhread.sleep(5)

print 'Suspend st4 ...'

st4:suspend()

print 'Resume st4 ...'

st4:resume()
```

新的多线程 Lua 解释器：[lua.zip](https://bbs.kanxue.com/attach-download-96898.htm)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- `LuaMT53运行效果.jpg`（57.14kb，31次下载）
- `jjj.jpg`（131.47kb，10次下载）
- `lua.zip`（237.24kb，155次下载）

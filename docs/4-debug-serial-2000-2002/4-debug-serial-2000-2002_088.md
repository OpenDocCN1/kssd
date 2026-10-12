
# Opera 4.

> 原文：[`www.pediy.com/kssd/pediy02/forum134.htm`](https://www.pediy.com/kssd/pediy02/forum134.htm)

## 1. 用户询问

dr0 兄：

Hi! 很高兴认识你，我是 `dREAMtHEATER`，有一事向你请教。

你能告诉我 Opera 把安装时间存在什么地方吗？我从 3.0 开始破解这个程序的，但始终没有找到正确的答案，这种情况我很少遇到，一般存在注册表或 windows 目录下，但它好象不是，望赐教。

`dREAMtHEATER`

## 2. 安装时间存储位置

`Opera 4.0 Final` 把时间记录在如下两个地方：

### 1. `WIN.INI`

```
[Compatibility Driver]
SAPDRV  = 0x00002d      // 这个是 4.0 Final 的
SAPDRVQ = 0x00003d      // 下面这两个是旧版本的
SAPDRVX = 0x00001c
```

### 2. `OUsr402.dat`

在该文件的偏移地址 `0x00000004` 处，占 4 个字节（即一个长整数），是和四个固定字节异或后的值。整个文件的解密都是采用异或的方式。

旧版本的文件名与此稍有不同，如 `OUsr401.dat` 等。注册码也放在这个文件中。

这个文件长度固定为 `0x4BC`，即 1212 字节。

将文件 `OUsr402.dat` 删除，并将 `win.ini` 中的相应部分删除就可以再用 30 天。

## 3. 时间判断逻辑

```
:00417272  call 005101A5                //调用 GetLocalTime()取当前时间
:00417277  sub eax, dword ptr [ebx+04]  //减去安装时间
:0041727A  push eax
:0041727B  call 0050E648
:00417280  add esp, 00000018
:00417283  cmp eax, 00015180            //0x15180 = 3600 秒每小时 * 24 小时 = 1 天
:00417288  jl 00417298
:0041728A  inc [ebx+08]
:0041728D  push 00000000
:0041728F  call 005101A5
:00417294  pop ecx
:00417295  mov dword ptr [ebx+04], eax
:00417298  push 0000001F
:0041729A  mov ecx, ebx
:0041729C  call 00416E58
:004172A1  mov ecx, dword ptr [ebx+08]
:004172A4  cmp ecx, eax
:004172A6  jle 004172AA
:004172A8  mov eax, ecx
:004172AA  cmp eax, 0000001E            //30 天过期没有
:004172AD  jle 004172B2
:004172AF  push 0000001E
:004172B1  pop eax
:004172B2  push eax
:004172B3  mov ecx, ebx
:004172B5  mov dword ptr [ebx+08], eax
:004172B8  call 00416DE8
:004172BD  pop edi
:004172BE  pop esi
:004172BF  pop ebx
:004172C0  leave
:004172C1  ret
```

`4dd466d0e1e0cc123afbe504da2462ca`

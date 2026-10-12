
# netants1.0-1.24

> 原文：[`www.pediy.com/kssd/pediy03/forum1114.htm`](https://www.pediy.com/kssd/pediy03/forum1114.htm)

`netants1.0-1.24` 一直不变的去广告方法

### 1.23 去广告

* Possible Reference to String Resource ID=00001: "Invalid retry delay"

|

```
:0041BE36 6A01                   push 00000001
:0041BE38 2B45F0                 sub eax, dword ptr [ebp-10]
:0041BE3B 6A42                   push 00000042                  //push 0
:0041BE3D 50                     push eax
:0041BE3E FF75F4                 push [ebp-0C]
:0041BE41 FF75F0                 push [ebp-10]
:0041BE44 FF35A8AB4400           push dword ptr [0044ABA8]
:0041BE4A FFD7                   call edi
:0041BE4C 8345F443               add dword ptr [ebp-0C], 00000043    //nop
:0041BE50 391D78AB4400           cmp dword ptr [0044AB78], ebx
:0041BE56 0F84F5000000           je 0041BF51
```

### 1.24 去广告

* Possible Reference to String Resource ID=00001: "Invalid retry delay"

|

```
:0041C531 6A01                   push 00000001
:0041C533 2B45F0                 sub eax, dword ptr [ebp-10]
:0041C536 6A42                   push 00000042                  //push 0
:0041C538 50                     push eax
:0041C539 FF75F4                 push [ebp-0C]
:0041C53C FF75F0                 push [ebp-10]
:0041C53F FF3588E94400           push dword ptr [0044E988]
:0041C545 FFD7                   call edi
:0041C547 8345F443               add dword ptr [ebp-0C], 00000043    //nop
:0041C54B 391D58E94400           cmp dword ptr [0044E958], ebx
:0041C551 0F84F5000000           je 0041C64C
```

### 清后门

(先用 `optout0.999999` 把垃圾清掉 `http://grc.com/optout.htm` 作者不再更新它了,我放在 `zombieys.126.com` 了)

#### 1.23

```
40223e call 40bc19  ->nop
4024ee jne  4024fd  ->jmp 4024fd
40261c jne  40262b  ->jmp 40262b
40261a cmp  eax,esi ->mov al,1
```

#### 1.24

```
402544 call 402322 ->nop
4025eb jnz 4025fa  ->jmp 4025fa
402719 jnz 402728  ->jmp 402728
402717 cmp eax,esi ->mov al,01
```

改这些的主要目的是把缺少文件的提示去掉

### 其他补丁:

1. `FTP` 站点无法登录的补丁程序: `fix_defaultpassword.reg`
2. 去掉窗口背景图的补丁程序: `nowindowbk.reg`
3. 去掉程序开启 `Splash` 的补丁程序: `netants_nosplash.reg`

#### 1. `fix_defaultpassword.reg`

```
REGEDIT4
[HKEY_CURRENT_USER\Software\NetAnts\Configure]
"DefaultPassword"=""
```

#### 2. `nowindowbk.reg`

```
REGEDIT4
[HKEY_CURRENT_USER\Software\NetAnts\Layout]
"NoWindowBK"=dword:00000001
```

#### 3. `netants_nosplash.reg`

```
REGEDIT4
[HKEY_CURRENT_USER\Software\NetAnts\Layout]
"NoSplash"="1"
```

这是我的全部家当了，全套下载去 `zombieys.cn.hongnet.com/cnetants.rar`

`zombieys[CCG]`

`2001.7.3`  `4dd466d0e1e0cc123afbe504da2462ca`

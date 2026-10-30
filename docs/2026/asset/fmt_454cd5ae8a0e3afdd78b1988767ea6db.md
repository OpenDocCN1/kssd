
```c
enum ucount_type {
    UCOUNT_USER_NAMESPACES,
    UCOUNT_PID_NAMESPACES,
    UCOUNT_UTS_NAMESPACES,
    UCOUNT_IPC_NAMESPACES,
    UCOUNT_NET_NAMESPACES,
    UCOUNT_MNT_NAMESPACES,
    UCOUNT_CGROUP_NAMESPACES,
    UCOUNT_TIME_NAMESPACES,
#ifdef CONFIG_INOTIFY_USER
    UCOUNT_INOTIFY_INSTANCES,
    UCOUNT_INOTIFY_WATCHES,
#endif
#ifdef CONFIG_FANOTIFY
    UCOUNT_FANOTIFY_GROUPS,
    UCOUNT_FANOTIFY_MARKS,
#endif
    UCOUNT_RLIMIT_NPROC,
    UCOUNT_RLIMIT_MSGQUEUE,
    UCOUNT_RLIMIT_SIGPENDING,
    UCOUNT_RLIMIT_MEMLOCK,
    UCOUNT_COUNTS,
};//in user_namespace.h
```

但是去看了 `ucounts` 结构体的定义发现里面就定义了一个 `user_namespace` 的指针和一个链表，队列，以及标识了一个 `uid`。这个 `ucounts` 可能只是一个用于做某些标记的东西，我们暂且不管把先。

后面有一个 `current_chrooted` 函数，我们同样看看它的定义：

```c
bool current_chrooted(void)
{
    /* Does the current process have a non-standard root */
    struct path ns_root;
    struct path fs_root;
    bool chrooted;
    /* Find the namespace root */
    ns_root.mnt = &current->nsproxy->mnt_ns->root->mnt;
    ns_root.dentry = ns_root.mnt->mnt_root;
    path_get(&ns_root);
    while (d_mountpoint(ns_root.dentry) && follow_down_one(&ns_root))
        ;
    get_fs_root(current->fs, &fs_root);
    chrooted = !path_equal(&fs_root, &ns_root);
    path_put(&fs_root);
    path_put(&ns_root);
    return chrooted;
}
```

根据注释以及关键的语句 `chrooted = !path_equal(&fs_root, &ns_root);` 我们大概也能猜测出来，它应该就是判断 `namespace` 的根目录是否于文件系统一致，一致才允许你创建这个 `namespace`。

#### part2

![15](upload/attach/202209/919002_HK2GUHYD9F63YKZ.jpg)

然后在这里需要判断一下属主和数组是否映映射到了父 `namespace` 上。

后面的话基本上和我们复现的漏洞无关了，我们这么来了解了一下构成形式，`namespace` 确实是树状图形式，而且下面我们很清楚地能看到 `ns->level=parent_ns->level+1`。

### 权限设置

这里我们来查看对应版本的代码，<a href="elink@d5aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6$3y4q4)9J5k6e0p5@1i4K6u0W2x3U0V1I4i4K6u0r3M7$3!0#2M7X3y4W2i4K6u0r3k6Y4y4Q4x3V1k6^5j5i4c8@1M7W2)9J5k6h3x3`.">链接贴上</a>。

先来看到`416`行，对`setxattr`函数进行分析，这里解释一下`setxattr`的一个名字由来（自己意淫的，非官方说法，经供参考），`set`就是设置， `x`其实它可以代表`extended`扩展的，`attr`就是属性了，连起来就是**设置扩展属性**，这里的扩展属性就是指`capability`。其实我感觉吧， `x`好像能表示一切`ex`开头的单词，比如我们经常见到的三个权限位，用`x`标识`execute`。

```c
/*
 * Extended attribute SET operations
 */
static long
setxattr(struct dentry *d, const char __user *name, const void __user *value,
         size_t size, int flags)
{
    int error;
    void *kvalue = NULL;
    char kname[XATTR_NAME_MAX + 1];

    if (flags & ~(XATTR_CREATE|XATTR_REPLACE))
        return -EINVAL;
    error = strncpy_from_user(kname, name, sizeof(kname));
    if (error == 0 || error == sizeof(kname))
        error = -ERANGE;
    if (error < 0)
        return error;
    if (size) {
        if (size > XATTR_SIZE_MAX)
            return -E2BIG;
        kvalue = kvmalloc(size, GFP_KERNEL);
        if (!kvalue)
            return -ENOMEM;
        if (copy_from_user(kvalue, value, size)) {
            error = -EFAULT;
            goto out;
        }
        if ((strcmp(kname, XATTR_NAME_POSIX_ACL_ACCESS) == 0) ||
            (strcmp(kname, XATTR_NAME_POSIX_ACL_DEFAULT) == 0))
            posix_acl_fix_xattr_from_user(kvalue, size);
        else if (strcmp(kname, XATTR_NAME_CAPS) == 0) {
            error = cap_convert_nscap(d, &kvalue, size);
            if (error < 0)
                goto out;
            size = error;
        }
    }
    error = vfs_setxattr(d, kname, kvalue, size, flags);
out:
    kvfree(kvalue);
    return error;
}
```

乍一看逻辑有点小复杂，主要是很多的宏定义和很多没见过的函数，也不太能够望文生义，于是我找到了Linux手册对于`setxattr`的说明：

```
$man 2 setxattr
```

> setxattr() sets the value of the extended attribute identified by name and associated with the given path in the filesystem. The size argument specifies the size (in bytes) of value; a zero-length value is permitted.

貌似介绍的也比较笼统，还是靠自己试试吧。

#### part1

`flags & ~(XATTR_CREATE|XATTR_REPLACE)`，这其实是很常见的掩码写法，差不多意思就是`flag`标志只在`create`和`replace`位上设置，如果设置了其它位则退出。

`strncpy_from_user(kname, name, sizeof(kname))`对传入的`name`参数进行拷贝，拷贝到了`kname`也就是内核栈当中。第一个判断应该是判断空字符串和防止溢出，因为如果`sizeof(kname)`字节都被占满了那么这个字符串还会跟下面连续的字符串相连，造成一些错误。

这里出现了我的知识盲区，这里也来解释一下，在内核里面，看见全大写字母的变量基本都不是变量，都是宏定义。而我实在不知道字符串常量有直接拼接的做法：

```c
#include<stdio.h>
#define s1 "123"
#define s2 "456"
#define s3 s1 s2
int main(){
    puts(s3);
}
/*output:
123456
*/
```

我还以为是宏定义的特殊写法呢，这里mark一下。

这里给出这些宏定义的最终结果

```c
#define XATTR_NAME_POSIX_ACL_ACCESS "system.posix_acl_access"
#define XATTR_NAME_POSIX_ACL_DEFAULT "system.posix_acl_default"
#define XATTR_NAME_CAPS "security.capability"
```

那么第一个`if`我们`duck`不必关心，我们主要关心第二个跟`capability`相关的分支。

我们具体逻辑也不进一步分析了，我们就看看这个函数给的注释：

`This function will then take care to map the inode according to @mnt_userns before checking permissions.`

我们也不难看出来，在检查权限之前就是会对文件系统和`user namespace `进行映射，这个函数叫`cap_convert_nscap`，那其实就是对`capability`的`userns`进行一个映射了（应该是这个意思。

就是可能，它会在不同的`namespace`上嘛，比如这个文件夹是其中一个`user namespace`创建的，不可能我换一个`namespace`去检测权限也是相同的手法，肯定是要把权限映射一下的，映射到同一个`namespace`上才能进行权限检查。

经过一系列检查之后，走到了`vfs_setxattr`，也就是虚拟文件系统的扩展属性设置。

```C
int
vfs_setxattr(struct dentry *dentry, const char *name, const void *value,
             size_t size, int flags)
{
    struct inode *inode = dentry->d_inode;
    int error;

    error = xattr_permission(inode, name, MAY_WRITE);
    if (error)
        return error;

    inode_lock(inode);
    error = security_inode_setxattr(dentry, name, value, size, flags);
    if (error)
        goto out;
    error = __vfs_setxattr_noperm(dentry, name, value, size, flags);
out:
    inode_unlock(inode);
    return error;
}
```

第一条不深入挖下去了，就是判断有没有写的权限，然后上锁，防止发生竞争，然后进行`security_inode_setxattr`函数进行进一步的权限校验，最后执行`__vfs_setxattr_noperm`函数，它的后缀`noperm`就是还没有进行权限检查的`__vfs_setxattr`与我们之前说的分析思路是一致的。在这个函数里面有一个大`if`判断文件是否有权限，最终调用一个`__vfs_setxattr`去真实设置`xattr`。

因此我们可以发现，在调用设置文件扩展属性时候，会有一系列的检查，比如你是否是`root`，你对文件操作是否有权限之类的，因为即使你是`root`也得看看那个文件系统的权限是否归你所有，有可能是其它`user_namespace`的用户创建的，那么你有可能也是没有权限的，这个地方是不会出现越权行为的。

然后我们看到`overlayfs`的设置文件扩展属性。

```C
int ovl_xattr_set(struct dentry *dentry, struct inode *inode, const char *name,
                  const void *value, size_t size, int flags)
{
    int err;
    struct dentry *upperdentry = ovl_i_dentry_upper(inode);
    struct dentry *realdentry = upperdentry ?: ovl_dentry_lower(dentry);
    const struct cred *old_cred;

    err = ovl_want_write(dentry);
    if (err)
        goto out;

    if (!value && !upperdentry) {
        err = vfs_getxattr(realdentry, name, NULL, 0);
        if (err < 0)
            goto out_drop_write;
    }

    if (!upperdentry) {
        err = ovl_copy_up(dentry);
        if (err)
            goto out_drop_write;
        realdentry = ovl_dentry_upper(dentry);
    }

    old_cred = ovl_override_creds(dentry->d_sb);
    if (value)
        err = vfs_setxattr(realdentry, name, value, size, flags);
    else {
        WARN_ON(flags != XATTR_REPLACE);
        err = vfs_removexattr(realdentry, name);
    }
    revert_creds(old_cred);

out_drop_write:
    ovl_drop_write(dentry);
out:
    return err;
}///fs/overlayfs/inode.c
```

其它的我们不看，我们解释比较容易理解的。

```c
if (!upperdentry) {
    err = ovl_copy_up(dentry);
    if (err)
        goto out_drop_write;
    realdentry = ovl_dentry_upper(dentry);
}
```

这个地方其实就是我们说的，如果文件在`lower`当中，那么拷贝一份到`upper`当中去，然后把新的文件节点指向`upper`。

```c
if (value)
    err = vfs_setxattr(realdentry, name, value, size, flags);
else {
    WARN_ON(flags != XATTR_REPLACE);
    err = vfs_removexattr(realdentry, name);
}
```

然后直接调用`vfs_setxattr`函数了，我们知道在`vfs_setxattr`之前有一个入口，也就是`setxattr`这个地方会有一个调用，调用`cap_convert_nscap`函数去检查`user namespace`是否一致。而这里直接调用`vfs_setxattr`这个函数就绕过了`namespace`的检查。

所以我们之前的利用步骤就是先创建了一个`namespace`，然后挂载了一个`overlayfs`，在`merge`文件夹中是我们创建的`fs namespace`，因此我们创建的`root`用户对这个`fs namespace`有设置`capability`的操作权限，这个其实没有问题，因为我即使运行了这个`a.out`也不会有真正的`root`权限，有的只是我们创建的这个`user namespace`的`root`权限，而这个权限实际是`init`的`user`创建的，因此实际操作还是获得不了真实的`root`，但是问题就是`overlayfs`的这个特性：我们修改了`merge`中的`a.out`会反向修改之前在`upper`中的`a.out`，因此我们给它`setuid`的权限导致了`upper/a.out`也有`setuid`的权限，而`upper/a.out`是在实际的`init user namespace`创建的，因此它有了`init user namespace`的`setuid`权限。我们运行`upper/a.out`直接获取真实`root`权限。

## 修复方案

这个其实我个人认为应该是`overlayfs`的问题，在修改的时候应该检查`user namespace`才对，但是它修改了`xattr.c`中的`vfs_setxattr`函数，这个函数重新用了一个`cap_convert_nscap`函数检查`namespace`。

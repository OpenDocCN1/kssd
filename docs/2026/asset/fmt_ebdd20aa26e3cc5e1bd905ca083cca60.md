
``ae3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3N6s2u0Q4x3X3c8&6N6h3c8S2K9g2)9J5k6h3S2S2N6r3g2F1j5h3u0D9L8$3N6Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8Y4c8J5P5g2)9J5c8U0t1H3x3U0m8Q4x3V1j5H3x3W2)9J5c8U0l9$3i4K6u0r3x3e0x3H3y4e0f1I4i4K6t1K6M7s2N6F1i4K6u0V1y4o6V1I4M7s2c8K6i4K6u0V1g2r3S2A6L8X3E0Q4x3X3c8@1N6$3W2U0k6g2)9J5k6r3u0W2k6X3!0J5k6g2)9J5k6s2y4H3k6h3q4C8K9h3&6Y4i4K6u0V1L8$3&6U0k6b7.`.``

## 三、trick_to_trick

这道题目给了两处任意地址写的机会，并且提供了`libc`的地址，`libc`环境是2.29，2.29的`vtable`是具有写权限的。

``b6cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1k6h3q4E0M7X3!0U0K9$3g2@1K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1j5J5x3o6t1H3i4K6u0r3x3o6u0Q4x3V1j5H3y4g2)9J5c8W2m8%4L8W2)9J5k6p5S2S2j5$3E0f1e0g2)9J5k6o6t1H3x3U0m8Q4x3X3c8f1M7X3W2H3i4K6u0V1g2r3!0Q4x3X3c8f1M7X3W2U0K9#2)9J5c8W2!0q4z5q4!0n7c8W2)9&6z5g2!0q4y4q4!0n7z5q4!0m8b7g2!0q4y4g2)9^5c8q4)9&6b7g2!0q4y4g2!0m8c8g2!0m8x3W2!0q4z5q4!0m8c8g2!0n7x3q4!0q4y4g2!0n7c8q4)9&6y4g2!0q4y4q4!0n7b7g2)9^5y4W2!0q4z5q4!0m8c8W2!0m8y4W2!0q4y4#2!0n7b7W2)9^5y4W2!0q4y4#2)9&6b7g2)9^5y4q4!0q4z5q4!0m8y4#2!0m8x3#2!0q4z5g2!0m8x3W2)9&6z5q4!0q4z5q4!0n7c8W2)9^5y4#2!0q4y4#2!0m8z5q4)9^5b7W2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4#2)9&6b7#2)9^5b7W2!0q4y4q4!0n7b7g2)9^5y4W2!0q4y4q4!0n7z5g2)9^5b7W2!0q4y4g2)9&6x3q4)9^5c8g2!0q4y4g2)9^5c8W2)9&6y4#2!0q4y4#2)9&6b7W2)9^5b7g2!0q4y4g2!0n7c8g2)9^5z5q4!0q4y4g2!0m8y4q4)9&6b7g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4z5q4)9^5x3q4)9^5x3#2!0q4y4W2)9&6c8W2!0m8y4g2!0q4y4#2)9^5x3W2!0n7z5g2!0q4y4q4!0n7z5q4!0n7b7W2!0q4z5q4!0m8y4W2)9^5x3g2!0q4y4W2)9&6z5q4!0m8c8X3k6U0L8r3!0K6k6g2)9J5z5q4)9J5z5g2!0q4y4g2)9^5y4#2!0n7c8q4!0q4y4W2)9&6y4g2!0n7x3q4!0q4z5q4!0m8y4#2!0m8y4W2!0q4y4g2)9^5c8W2)9&6x3g2!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4#2!0m8z5q4)9^5b7W2!0q4y4g2!0n7b7g2)9^5c8W2!0q4y4W2!0n7y4g2)9^5x3g2!0q4y4g2)9^5b7g2!0m8b7W2!0q4y4W2)9^5b7#2)9^5x3g2!0q4x3#2)9^5x3q4)9^5x3R3.`.``

本题目是可以修改`vtable`的值，利用`fclose()`控制程序流主要是如下源码

```
int

_IO_new_fclose (FILE *fp)

{

  int status;



  CHECK_FILE(fp, EOF);



#if SHLIB_COMPAT (libc, GLIBC_2_0, GLIBC_2_1)

  /* We desperately try to help programs which are using streams in a

     strange way and mix old and new functions.  Detect old streams

     here.  */

  if (_IO_vtable_offset (fp) != 0)

    return _IO_old_fclose (fp);

#endif



  /* First unlink the stream.  */

  if (fp->_flags & _IO_IS_FILEBUF)

    _IO_un_link ((struct _IO_FILE_plus *) fp);



  _IO_acquire_lock (fp);

  if (fp->_flags & _IO_IS_FILEBUF)

    status = _IO_file_close_it (fp);

  else

    status = fp->_flags & _IO_ERR_SEEN ? -1 : 0;

  _IO_release_lock (fp);

  _IO_FINISH (fp);  -------------------------------------------------------此处开始劫持程序流

  if (fp->_mode > 0)

    {

      /* This stream has a wide orientation.  This means we have to free

     the conversion functions.  */

      struct _IO_codecvt *cc = fp->_codecvt;



      __libc_lock_lock (__gconv_lock);

      __gconv_release_step (cc->__cd_in.__cd.__steps);

      __gconv_release_step (cc->__cd_out.__cd.__steps);

      __libc_lock_unlock (__gconv_lock);

    }

  else

    {

      if (_IO_have_backup (fp))

    _IO_free_backup_area (fp);

    }

  if (fp != _IO_stdin && fp != _IO_stdout && fp != _IO_stderr)

    {

      fp->_flags = 0;

      free(fp);

    }



  return status;

}
```

当调用`setbuf`关闭缓冲区之后，再次输入的时候将不会在`heap`区域分配空间当做缓冲区，其中`IO_buf_base`指向的是缓冲区起点，`IO_buf_end`指向的是缓冲区终点，关闭缓冲区之后，`IO_buf_base`指向`_IO_2_1_stdin_->shortbuf`，位于`stdin`内部，保存`\n`或者`\0`，`IO_buf_end=IO_buf_base+1`。

我们修改`_IO_2_1_stdin_->IO_buf_end`，使其增大之后，便可以控制此缓冲区内部的内容，即输入时的内容都会先存储在当前的缓冲区内（`libc`空间）。

我最初还有一个疑问是，`scanf("%llx %llx"，&v4,&v5)`正常来说是输入两个64位十六进制整数，在第二次输入时输入了长度很长的`payload`，不按照正常输入可以输入进去么？ 经过程序测试的结果是，他会将输入的所有字符串先缓存在缓冲区内，然后按照格式解析给`v4`,`v5`，所以我们的`payload`是可以正常覆盖到`libc`上的。

解题思路：

1. 修改`_IO_2_1_stdin->_IO_buf_end`，增大其值导致可以覆盖`libc`空间内容
2. 修改`_IO_2_1_stdout->vtable`值，并且伪造`fake_table->__finish`函数地址为`setcontext+0x35`，同时布置好orw的ROP链
3. `fclose(stdout)`触发ROP链，进行orw获得shell

## 四、网鼎杯--VMpwn

之所以介绍这道题目，是因为这道题目最后的get shell方式也是通过`fclose`函数，本题目不可以在`libc`段写，但是可以再`bss`段写，我们可以在`bss`段伪造`stderr`，然后利用`fclose(stderr)`获得shell。

由于是伪造`stderr`，所以不同于上题目中利用`_IO_FINISH (fp);`来控制程序流，我们控制程序流的位置要稍微靠前一下,否则还要保证之前的功能可以正常，劫持程序流位置如下：

```
int

_IO_new_fclose (FILE *fp)

{

  int status;



  CHECK_FILE(fp, EOF);



#if SHLIB_COMPAT (libc, GLIBC_2_0, GLIBC_2_1)

  /* We desperately try to help programs which are using streams in a

     strange way and mix old and new functions.  Detect old streams

     here.  */

  if (_IO_vtable_offset (fp) != 0)

    return _IO_old_fclose (fp);

#endif



  /* First unlink the stream.  */

  if (fp->_flags & _IO_IS_FILEBUF)

    _IO_un_link ((struct _IO_FILE_plus *) fp);



  _IO_acquire_lock (fp);

  if (fp->_flags & _IO_IS_FILEBUF)

    status = _IO_file_close_it (fp); -------------------------------------------------------此处开始劫持程序流

  else

    status = fp->_flags & _IO_ERR_SEEN ? -1 : 0;

  _IO_release_lock (fp);

  _IO_FINISH (fp); 

  if (fp->_mode > 0)

    {

      /* This stream has a wide orientation.  This means we have to free

     the conversion functions.  */

      struct _IO_codecvt *cc = fp->_codecvt;



      __libc_lock_lock (__gconv_lock);

      __gconv_release_step (cc->__cd_in.__cd.__steps);

      __gconv_release_step (cc->__cd_out.__cd.__steps);

      __libc_lock_unlock (__gconv_lock);

    }



  else

    {

      if (_IO_have_backup (fp))

    _IO_free_backup_area (fp);

    }

  if (fp != _IO_stdin && fp != _IO_stdout && fp != _IO_stderr)

    {

      fp->_flags = 0;

      free(fp);

    }



  return status;

}
```

贴一下`exp`，感兴趣的可以自己去调试一下

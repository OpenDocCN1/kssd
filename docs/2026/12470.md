
# Socket网络模式下获取客户端磁盘文件并上传下载

服务端是用 Delphi 做的界面和写的代码，客户端只是响应，用的是 C++ 写客户端。

## 下载文件

由于我的代码采用的是数据流和消息异步，贴码的话太多，这里我就用口述+重要代码截图的方式。

1. 建立连接后，先获取客户端的磁盘信息。这里我们给客户端发送一个命令，比如 `PKT_DISK` 等类似消息；在客户端接受消息，判断接收到的是相等命令后，在客户端执行相关函数或操作。

   这里调用函数 `GetLogicalDriveStrings` 获取电脑磁盘信息，然后将获取到的磁盘信息 `send` 给服务端。

   ![](https://bbs.kanxue.com/upload/attach/202105/922647_ENK542A96ZXR9CF.jpg)

   ![](https://bbs.kanxue.com/upload/attach/202105/922647_GNSPUZYZQVHF344.jpg)

2. 在服务端 `recv` 到客户的电脑磁盘信息后，添加到界面，客户端的磁盘就已枚举好。

   ![](https://bbs.kanxue.com/upload/attach/202105/922647_8K9YNMUJF3A5SZM.jpg)

3. 获取磁盘下的目录，同样也是 `send` 指定命令和想要下载的路径给客户端，客户端 `recv` 到相对应命令之后，响应相关代码，使用函数 `FindFirstFile`、`FindNextFile` 查找相关路径的文件。需要定义一个 `WIN32_FIND_DATA` 结构，返回给此结构，判断结构的 `dwFileAttributes` 属性不为“`.`”和“`..`”，循环查找此路径文件，然后一一将文件名、属性、时间、大小发送给服务端，最后调用 `FindClose` 关闭打开的句柄。

   ![](https://bbs.kanxue.com/upload/attach/202105/922647_ENVQQ4A4G2WQWWZ.jpg)

4. 服务端收到文件名等信息后，可枚举出此目录下的所有文件。

   ![](https://bbs.kanxue.com/upload/attach/202105/922647_A22DMRKHSAFGQXS.jpg)

5. 有了文件名之后，我们就可以执行下载命令了。给客户端发送下载命令，将想要下载的路径名发给客户端，客户端收到消息之后，使用函数 `CreateFile` 打开收到的文件路径，判断是不是一个可以正常打开的文件。打开成功后，使用函数 `GetFileTime`、`GetFileAttributes`、`GetFileSizeEx` 获取文件的时间、属性、大小等，获取成功后 `send` 给服务端。

   ![](https://bbs.kanxue.com/upload/attach/202105/922647_PBFBK3W528KM7S3.jpg)

6. 当客户端确定可以打开后，服务端成功接收到消息，读取客户端发送的文件属性、大小、时间，使用函数 `CreateFile` 来创建文件，以预接收要下载的数据。创建成功后，给客户端发送消息，告诉客户端：“你可以给我发送这个需要下载的文件的数据了”。

   ![](https://bbs.kanxue.com/upload/attach/202105/922647_H974X5JMDNXJ3HU.jpg)

7. 客户端调用函数 `ReadFile` 读取刚才打开的文件，循环地发送给服务端，直到读取的字节数为 0，则调用函数 `CloseHandle` 关闭句柄，避免泄露句柄。

   ![](https://bbs.kanxue.com/upload/attach/202105/922647_EYGA44Z639WK2EJ.jpg)

8. 服务端则循环 `recv` 客户端的数据，利用函数 `WriteFile` 将收到的数据写入刚才创建的文件中，直到收到的消息为结束，则退出循环。接收完毕后，使用函数 `SetFileAttributes`、`SetFileTime` 设置文件的属性和时间，使用 `CloseHandle` 关闭句柄。

至此，下载一个文件成功。

![](https://bbs.kanxue.com/upload/attach/202105/922647_CDE9AWJEQ7JPYFU.jpg)

## 上传文件

1. 服务端使用函数 `CreateFile` 打开想要上传的文件，利用函数 `GetFileTime`、`GetFileAttributes`、`GetFileSizeEx` 获取文件的时间、属性、大小，将文件的信息发送给客户端，等待接收客户端是否可以创建文件的命令。

   ![](https://bbs.kanxue.com/upload/attach/202105/922647_5S6KWQMB7R2F4MJ.jpg)

   ![](https://bbs.kanxue.com/upload/attach/202105/922647_YE6WQDXBPVHXJJ6.jpg)

2. 客户端接收到消息后，使用函数 `CreateFile` 创建文件，创建成功后，发送消息给服务端：“创建文件成功，可以给我发送需要上传的数据了”。

   ![](https://bbs.kanxue.com/upload/attach/202105/922647_XM84FGWQ68QE642.jpg)

3. 当服务端收到客户端创建文件成功的消息后，使用函数 `ReadFile` 循环读取刚才打开的文件，一一发送数据给客户端，直到读取文件的字节数为 0 时，则发送一个完成的命令给客户端，并使用函数 `CloseHandle` 关闭句柄。

   ![](https://bbs.kanxue.com/upload/attach/202105/922647_6YYW554E67HEFYS.jpg)

4. 客户端开启循环模式，一直接收服务端发送的数据，然后利用函数 `WriteFile` 给刚才创建成功的文件写入数据，直到收到的命令是完成，则给服务端再 `send` 一条命令，告诉服务端接收上传的数据成功，使用函数 `SetFileTime` 设置文件时间，最后一步使用函数 `CloseHandle`、`SetFileAttributes` 来关闭句柄和设置文件属性。

   ![](https://bbs.kanxue.com/upload/attach/202105/922647_T4RCEHWX64BCAFP.jpg)

5. 服务端这边再执行一次 `recv`，收到成功后，且没有错误的情况下，至此上传文件完成。

   ![](https://bbs.kanxue.com/upload/attach/202105/922647_RUYGASQE9JV8XUG.jpg)

## 总结

服务端和客户端的单个文件的上传下载就分享到这里。

- 如果想要下载/上传多个文件，则在两端增加一个变量，用于记录需要下载的个数，循环下载/上传文件。
- 如果想要下载一个文件夹，我采用的方式是比较简单的递归算法：枚举文件夹内的所有文件，如果遇到文件夹则再次递归，最多再调用创建文件夹的函数，就可以上传/下载文件夹了。

欢迎大家多多探讨，多多指导，多多交流。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-5-26 22:09 被 PE_Hacker 编辑，原因：

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)
[#开发技巧](https://bbs.kanxue.com/forum-41-1-135.htm)

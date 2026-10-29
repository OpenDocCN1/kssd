
## MD5 文件验证

| 参数与日志 |
| --- |
| `onEnter main_md5File arg0: 0x86844940 arg1: 0x3b arg2: 0x304f5f42 arg3: 0x4c747a48 arg4: 0xe6f73709`<br>`onEnter main_md5File arg0: /data/app/tv.iytqy.cvhaca-gmGCryB_O0HztLC7rqkhRQ==/base.apk`<br>`onLeave main_md5File`<br>`678c7f3bf3584a2079295d8834928146` |

- 写了个 Python 代码进行验证，发现结果是对的：

```python
import hashlib

filename = '50ash_5.7.1_230305_3.apk'
hasher = hashlib.md5()

with open(filename, 'rb') as f:
    buf = f.read()
    hasher.update(buf)

md5hash = hasher.hexdigest()
print(md5hash)

# [Running] python -u "c:\Users\Administrator\Desktop\50度灰\getMD5File.py"
# 678c7f3bf3584a2079295d8834928146
```

### `encoding_json_Marshal`

- 这里就是把我们之前的字符串再次序列化：

```c
v68 = encoding_json_Marshal((int)&map_string_interface_, *(_DWORD *)v95); // 转json数据
v18 = v68;
main_oldEncrypt(v28, v19, v18, v17, v93, v80, v81, v92, v87, v88, v81, 0); // 开始加密

// 所以这里我们直接 hook main_oldEncrypt 就可以得到加密前的明文了
```

### `main_oldEncrypt`

- hook 代码

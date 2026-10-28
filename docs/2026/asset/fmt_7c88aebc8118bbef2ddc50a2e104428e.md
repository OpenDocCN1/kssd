
```c
output( "public: %s", pszName );
}

comentFlags = 0;        // Don't know what this is yet.

if ( m_commentClass != 0xA0 )   // Bail out if not an OMF extension
    return;

if ( 0x1 != *p )                // Bail out if not an IMPDEF
    return;

comentFlags |= OMF_COMENT_IMPDEF;           // It's an IMPDEF

p += 2;                 // Blast past the subtype and ordinal fields

// Copy the "internal" name to a buffer, then advance past it
memcpy( szName, p+1, *p );
szName[*p] = 0;
p += (*p + 1);          // Advance past internal name

// Copy the module name out of the record, then advance past it.
memcpy( szModuleName, p+1, *p );
szModuleName[*p] = 0;

output( "public: %s.%s", szModuleName, szName );
}
```

```c
OBJ_FILE_TYPE DumpIntelOMFFile( PBYTE pFileBase )
{
    POMF_RECORD pBaseRec = (POMF_RECORD)pFileBase;
    DWORD comentFlags;
    // Check to see if it's a library, and if so, grab the page align size
    BOOL fLib = (BOOL)(OMF_LIBHDR == pBaseRec->m_type);
    unsigned cbAlign = fLib ? (pBaseRec->m_length + 3): 0;  // Don't ask...
    BOOL fContinue = TRUE;

    while ( fContinue )
    {
        // output( "Record %02X at: %08X\n", pBaseRec->m_type,
        //      (PBYTE)pBaseRec - pFileBase );
        BOOL fPageAlign = FALSE;        // Round up record to next paragraph
        switch( pBaseRec->m_type )
        {
            case OMF_EXTDEF:
                ((POMF_EXTDEF_RECORD)pBaseRec)->Display();
                break;
            case OMF_PUBDEF:
            case OMF_PUBD32:
                ((POMF_PUBDEF_RECORD)pBaseRec)->Display();
                break;
            case OMF_COMENT:
                ((POMF_COMENT_RECORD)pBaseRec)->Display( comentFlags );
                break;
            case OMF_MODEND:
            case OMF_MODE32:
                if ( fLib )
                    fPageAlign = TRUE;
                else
                    fContinue = FALSE;  // Not a .LIB.  We're done dumping
                break;
            case OMF_LIBEND:
                fContinue = FALSE;
                break;
            default:
                if ( (pBaseRec->m_type < OMF_THEADR) || // Check for bogus
                     (pBaseRec->m_type > OMF_LIBEND) )  // OMF records
                    fContinue = FALSE;
                break;
        }

        // Point to the next record
        pBaseRec = (POMF_RECORD)((DWORD)pBaseRec + pBaseRec->m_length + 3);
        // If necessary, adjust up to the next paragraph (.LIB files need
        // this when you encounter a MODEND record).
        if ( fPageAlign )
        {
            // Write-only code to align to the next page-sized boundary
            DWORD temp = (DWORD)pBaseRec + (cbAlign-1);
            pBaseRec = (POMF_RECORD)(temp & ~(cbAlign-1));
        }
    }

    if ( comentFlags & OMF_COMENT_IMPDEF )
        return OBJ_OMF_IMPLIB;
    else if ( fLib )
        return OBJ_OMF_LIB;
    else
        return OBJ_OMF_OBJ;
}
```

我查找公共符号与外部符号的方法并没有什么。对于OMF格式的文件，我只是简单地扫描所有的记录，但是只显示在`PUBDEF`，`PUBD32`和`EXTDEF`记录中的名字。对于COFF格式的OBJ和LIB文件，我使用了每个OBJ文件都包含的符号表。当符号表中出现重复信息时，我就跳过那些记录。但读取外部符号时，不得不使用其它方法。

当创建`OBJHELP.MS`时指定`HARDCORE=1`选项可以产生`DUMPOBJ`而不生成`OBJHELP`。也就是说，“`NMAKE HARDCORE=1 OBJHELP.MS`”将创建`DUMPOBJ.EXE`，这是一个命令行程序。它仅有一个命令行参数（就是你想要显示的OBJ或LIB文件的名字），并把结果输出到标准输出设备。这有利于把输出重定向到一个文件。

`DUMPOBJ.EXE d:\mstools\lib\comctl32.lib > myfile`

它的输出像下面这个样子：

```
DUMPOBJ - Matt Pietrek 1996, for Microsoft Systems Journal
public: __IMPORT_DESCRIPTOR_COMCTL32
extern: __NULL_IMPORT_DESCRIPTOR
extern: COMCTL32_NULL_THUNK_DATA
public: __NULL_IMPORT_DESCRIPTOR
public: COMCTL32_NULL_THUNK_DATA
public: _AddMRUData@12
public: __imp__AddMRUData@12
extern: __IMPORT_DESCRIPTOR_COMCTL32
<输出的其余部分省略>
```

译者：SmartTech　电子信箱：`zhzhtst@163.com`

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [Unresolved External Symbol.pdf](javascript:void(0);)

  （206.56kb，238次下载）

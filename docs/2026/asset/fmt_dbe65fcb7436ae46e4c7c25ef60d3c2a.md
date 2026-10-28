
# [翻译]Android资源文件结构分析，包括AndroidManifest，layout等

## 一些说明

Android资源文件头，每一个都是按照这种格式定义的。  
`uint16_t`，表示是16位即两个字节。  
`uint32_t`，表示是32位即4个字节。  
我觉得翻译多来，不如读原文。后面会有例子解析。  
`axmlprinter`是根据这个格式进行编写的，不是很全面。

## 例子

1. 对于给的这个`test.xml`文件，这是一个`AndroidManifest.xml`文件，
   对照`ResChunk_header`结构。前8个字节依次是`03 00 08 00 18 06 00 00`。
   类型为`0003`，为`RES_XML_TYPE`，该块长度为`0008`个字节，该块所拥有的整体字节，
   `00 00 06 18`，对比该`test.xml`文件，恰好。

2. 往下看，`01 00 1C 00 14 03 00 00 1C 00 00 00 00 00 00 00 00 00 00 00 8C 00 00 00 00 00 00 00` 为第2个块，
   类型为`1`，`RES_STRING_POOL_TYPE`，是一个字符串池。对比`ResStringPool_header`结构体，
   该结构体大小正好为28个字节。
   `1C`即描述该块的大小字节数为28个字节，整体块大小为`0314`。`1C`为字符串的个数。
   下面是style的个数，为0，flag标志为0，`8C`为字符串开始偏移位置，后面那四个字节为style的偏移，也是0。

3. 其他的结构体，分析类似。
4. 根据后面的分析，可以建立好资源映射等。读者可自己分析，以加强印象。
5. 该文件在Android的源代码中。位置在4.4版本中是`frameworks/base/include/androidfw/ResourceTypes.h`。
6. 那个`text.xml.txt`把后缀名字修改一下，就是一个`AndroidManifest.xml`文件了。是，资源格式的。

```cpp
/**
*******************************************************************************
*  Base Types
*
*  These are standard types that are shared between multiple specific
*  resource types.
*
*******************************************************************************
*/

/**
* Header that appears at the front of every data chunk in a resource.
*/
struct ResChunk_header
{
    // Type identifier for this chunk.  The meaning of this value depends
    // on the containing chunk.
    uint16_t type;

    // Size of the chunk header (in bytes).  Adding this value to
    // the address of the chunk allows you to find its associated data
    // (if any).
    uint16_t headerSize;

    // Total size of this chunk (in bytes).  This is the chunkSize plus
    // the size of any data associated with the chunk.  Adding this value
    // to the chunk allows you to completely skip its contents (including
    // any child chunks).  If this value is the same as chunkSize, there is
    // no data associated with the chunk.
    uint32_t size;
};

enum {
    RES_NULL_TYPE               = 0x0000,
    RES_STRING_POOL_TYPE        = 0x0001,
    RES_TABLE_TYPE              = 0x0002,
    RES_XML_TYPE                = 0x0003,

    // Chunk types in RES_XML_TYPE
    RES_XML_FIRST_CHUNK_TYPE    = 0x0100,
    RES_XML_START_NAMESPACE_TYPE= 0x0100,
    RES_XML_END_NAMESPACE_TYPE  = 0x0101,
    RES_XML_START_ELEMENT_TYPE  = 0x0102,
    RES_XML_END_ELEMENT_TYPE    = 0x0103,
    RES_XML_CDATA_TYPE          = 0x0104,
    RES_XML_LAST_CHUNK_TYPE     = 0x017f,

    // This contains a uint32_t array mapping strings in the string
    // pool back to resource identifiers.  It is optional.
    RES_XML_RESOURCE_MAP_TYPE   = 0x0180,

    // Chunk types in RES_TABLE_TYPE
    RES_TABLE_PACKAGE_TYPE      = 0x0200,
    RES_TABLE_TYPE_TYPE         = 0x0201,
    RES_TABLE_TYPE_SPEC_TYPE    = 0x0202
};

/**
* Representation of a value in a resource, supplying type
* information.
*/
struct Res_value
{
    // Number of bytes in this structure.
    uint16_t size;

    // Always set to 0.
    uint8_t res0;

    // Type of the data value.
    enum {
        // Contains no data.
        TYPE_NULL = 0x00,

        // The 'data' holds a ResTable_ref, a reference to another resource
        // table entry.
        TYPE_REFERENCE = 0x01,

        // The 'data' holds an attribute resource identifier.
        TYPE_ATTRIBUTE = 0x02,

        // The 'data' holds an index into the containing resource table's
        // global value string pool.
        TYPE_STRING = 0x03,

        // The 'data' holds a single-precision floating point number.
        TYPE_FLOAT = 0x04,

        // The 'data' holds a complex number encoding a dimension value,
        // such as "100in".
        TYPE_DIMENSION = 0x05,

        // The 'data' holds a complex number encoding a fraction of a
        // container.
        TYPE_FRACTION = 0x06,

        // Beginning of integer flavors...
        TYPE_FIRST_INT = 0x10,

        // The 'data' is a raw integer value of the form n..n.
        TYPE_INT_DEC = 0x10,

        // The 'data' is a raw integer value of the form 0xn..n.
        TYPE_INT_HEX = 0x11,

        // The 'data' is either 0 or 1, for input "false" or "true" respectively.
        TYPE_INT_BOOLEAN = 0x12,

        // Beginning of color integer flavors...
        TYPE_FIRST_COLOR_INT = 0x1c,

        // The 'data' is a raw integer value of the form #aarrggbb.
        TYPE_INT_COLOR_ARGB8 = 0x1c,

        // The 'data' is a raw integer value of the form #rrggbb.
        TYPE_INT_COLOR_RGB8 = 0x1d,

        // The 'data' is a raw integer value of the form #argb.
        TYPE_INT_COLOR_ARGB4 = 0x1e,

        // The 'data' is a raw integer value of the form #rgb.
        TYPE_INT_COLOR_RGB4 = 0x1f,

        // ...end of integer flavors.
        TYPE_LAST_COLOR_INT = 0x1f,

        // ...end of integer flavors.
        TYPE_LAST_INT = 0x1f
    };

    uint8_t dataType;

    // Structure of complex data values (TYPE_UNIT and TYPE_FRACTION)
    enum {
        // Where the unit type information is.  This gives us 16 possible
        // types, as defined below.
        COMPLEX_UNIT_SHIFT = 0,
        COMPLEX_UNIT_MASK = 0xf,

        // TYPE_DIMENSION: Value is raw pixels.
        COMPLEX_UNIT_PX = 0,

        // TYPE_DIMENSION: Value is Device Independent Pixels.
        COMPLEX_UNIT_DIP = 1,

        // TYPE_DIMENSION: Value is a Scaled device independent Pixels.
        COMPLEX_UNIT_SP = 2,

        // TYPE_DIMENSION: Value is in points.
        COMPLEX_UNIT_PT = 3,

        // TYPE_DIMENSION: Value is in inches.
        COMPLEX_UNIT_IN = 4,

        // TYPE_DIMENSION: Value is in millimeters.
        COMPLEX_UNIT_MM = 5,

        // TYPE_FRACTION: A basic fraction of the overall size.
        COMPLEX_UNIT_FRACTION = 0,

        // TYPE_FRACTION: A fraction of the parent size.
        COMPLEX_UNIT_FRACTION_PARENT = 1,

        // Where the radix information is, telling where the decimal place
        // appears in the mantissa.  This give us 4 possible fixed point
        // representations as defined below.
        COMPLEX_RADIX_SHIFT = 4,
        COMPLEX_RADIX_MASK = 0x3,

        // The mantissa is an integral number -- i.e., 0xnnnnnn.0
        COMPLEX_RADIX_23p0 = 0,

        // The mantissa magnitude is 16 bits -- i.e, 0xnnnn.nn
        COMPLEX_RADIX_16p7 = 1,

        // The mantissa magnitude is 8 bits -- i.e, 0xnn.nnnn
        COMPLEX_RADIX_8p15 = 2,

        // The mantissa magnitude is 0 bits -- i.e, 0x0.nnnnnn
        COMPLEX_RADIX_0p23 = 3,

        // Where the actual value is.  This gives us 23 bits of
        // precision.  The top bit is the sign.
        COMPLEX_MANTISSA_SHIFT = 8,
        COMPLEX_MANTISSA_MASK = 0xffffff
    };

    // The data for this item, as interpreted according to dataType.
    uint32_t data;

    void copyFrom_dtoh(const Res_value& src);
};

/**
*  This is a reference to a unique entry (a ResTable_entry structure)
*  in a resource table.  The value is structured as: 0xpptteeee,
*  where pp is the package index, tt is the type index in that
*  package, and eeee is the entry index in that type.  The package
*  and type values start at 1 for the first item, to help catch cases
*  where they have not been supplied.
*/
struct ResTable_ref
{
    uint32_t ident;
};

/**
* Reference to a string in a string pool.
*/
struct ResStringPool_ref
{
    // Index into the string pool table (uint32_t-offset from the indices
    // immediately after ResStringPool_header) at which to find the location
    // of the string data in the pool.
    uint32_t index;
};
```

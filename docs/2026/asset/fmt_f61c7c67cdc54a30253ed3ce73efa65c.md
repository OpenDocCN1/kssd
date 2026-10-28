
# String Pool

A set of strings that can be referenced by others through a `ResStringPool_ref`.

## `ResStringPool_header`

Definition for a pool of strings. The data of this chunk is an array of `uint32_t` providing indices into the pool, relative to `stringsStart`. At `stringsStart` are all of the UTF-16 strings concatenated together; each starts with a `uint16_t` of the string's length and each ends with a `0x0000` terminator. If a string is > 32767 characters, the high bit of the length is set meaning to take those 15 bits as a high word and it will be followed by another `uint16_t` containing the low word.

If `styleCount` is not zero, then immediately following the array of `uint32_t` indices into the string table is another array of indices into a style table starting at `stylesStart`. Each entry in the style table is an array of `ResStringPool_span` structures.

```
struct ResStringPool_header
{
    struct ResChunk_header header;

    // Number of strings in this pool (number of uint32_t indices that follow
    // in the data).
    uint32_t stringCount;

    // Number of style span arrays in the pool (number of uint32_t indices
    // follow the string indices).
    uint32_t styleCount;

    // Flags.
    enum {
        // If set, the string index is sorted by the string values (based
        // on strcmp16()).
        SORTED_FLAG = 1<<0,

        // String pool is encoded in UTF-8
        UTF8_FLAG = 1<<8
    };
    uint32_t flags;

    // Index from header of the string data.
    uint32_t stringsStart;

    // Index from header of the style data.
    uint32_t stylesStart;
};
```

## `ResStringPool_span`

This structure defines a span of style information associated with a string in the pool.

```
struct ResStringPool_span
{
    enum {
        END = 0xFFFFFFFF
    };

    // This is the name of the span -- that is, the name of the XML
    // tag that defined it.  The special value END (0xFFFFFFFF) indicates
    // the end of an array of spans.
    ResStringPool_ref name;

    // The range of characters in the string that this span applies to.
    uint32_t firstChar, lastChar;
};
```

# XML Tree

Binary representation of an XML document. This is designed to express everything in an XML document, in a form that is much easier to parse on the device.

## `ResXMLTree_header`

XML tree header. This appears at the front of an XML tree, describing its content. It is followed by a flat array of `ResXMLTree_node` structures; the hierarchy of the XML document is described by the occurrence of `RES_XML_START_ELEMENT_TYPE` and corresponding `RES_XML_END_ELEMENT_TYPE` nodes in the array.

```
struct ResXMLTree_header
{
    struct ResChunk_header header;
};
```

## `ResXMLTree_node`

Basic XML tree node. A single item in the XML document. Extended info about the node can be found after `header.headerSize`.

```
struct ResXMLTree_node
{
    struct ResChunk_header header;

    // Line number in original source file at which this element appeared.
    uint32_t lineNumber;

    // Optional XML comment that was associated with this element; -1 if none.
    struct ResStringPool_ref comment;
};
```

## `ResXMLTree_cdataExt`

Extended XML tree node for CDATA tags -- includes the CDATA string. Appears `header.headerSize` bytes after a `ResXMLTree_node`.

```
struct ResXMLTree_cdataExt
{
    // The raw CDATA character data.
    struct ResStringPool_ref data;

    // The typed value of the character data if this is a CDATA node.
    struct Res_value typedData;
};
```

## `ResXMLTree_namespaceExt`

Extended XML tree node for namespace start/end nodes. Appears `header.headerSize` bytes after a `ResXMLTree_node`.

```
struct ResXMLTree_namespaceExt
{
    // The prefix of the namespace.
    struct ResStringPool_ref prefix;

    // The URI of the namespace.
    struct ResStringPool_ref uri;
};
```

## `ResXMLTree_endElementExt`

Extended XML tree node for element start/end nodes. Appears `header.headerSize` bytes after a `ResXMLTree_node`.

```
struct ResXMLTree_endElementExt
{
    // String of the full namespace of this element.
    struct ResStringPool_ref ns;

    // String name of this node if it is an ELEMENT; the raw
    // character data if this is a CDATA node.
    struct ResStringPool_ref name;
};
```

## `ResXMLTree_attrExt`

Extended XML tree node for start tags -- includes attribute information. Appears `header.headerSize` bytes after a `ResXMLTree_node`.

```
struct ResXMLTree_attrExt
{
    // String of the full namespace of this element.
    struct ResStringPool_ref ns;

    // String name of this node if it is an ELEMENT; the raw
    // character data if this is a CDATA node.
    struct ResStringPool_ref name;

    // Byte offset from the start of this structure where the attributes start.
    uint16_t attributeStart;

    // Size of the ResXMLTree_attribute structures that follow.
    uint16_t attributeSize;

    // Number of attributes associated with an ELEMENT.  These are
    // available as an array of ResXMLTree_attribute structures
    // immediately following this node.
    uint16_t attributeCount;

    // Index (1-based) of the "id" attribute. 0 if none.
    uint16_t idIndex;

    // Index (1-based) of the "class" attribute. 0 if none.
    uint16_t classIndex;

    // Index (1-based) of the "style" attribute. 0 if none.
    uint16_t styleIndex;
};
```

## `ResXMLTree_attribute`

```
struct ResXMLTree_attribute
{
    // Namespace of this attribute.
    struct ResStringPool_ref ns;

    // Name of this attribute.
    struct ResStringPool_ref name;

    // The original raw string value of this attribute.
    struct ResStringPool_ref rawValue;

    // Processed typed value of this attribute.
    struct Res_value typedValue;
};
```

# RESOURCE TABLE

Header for a resource table. Its data contains a series of additional chunks:

- A `ResStringPool_header` containing all table values. This string pool contains all of the string values in the entire resource table (not the names of entries or type identifiers however).
- One or more `ResTable_package` chunks.

Specific entries within a resource table can be uniquely identified with a single integer as defined by the `ResTable_ref` structure.

## `ResTable_header`

```
struct ResTable_header
{
    struct ResChunk_header header;

    // The number of ResTable_package structures.
    uint32_t packageCount;
};
```

## `ResTable_package`

A collection of resource data types within a package. Followed by one or more `ResTable_type` and `ResTable_typeSpec` structures containing the entry values for each resource type.

```
struct ResTable_package
{
    struct ResChunk_header header;

    // If this is a base package, its ID.  Package IDs start
    // at 1 (corresponding to the value of the package bits in a
    // resource identifier).  0 means this is not a base package.
    uint32_t id;

    // Actual name of this package, \0-terminated.
    char16_t name[128];

    // Offset to a ResStringPool_header defining the resource
```

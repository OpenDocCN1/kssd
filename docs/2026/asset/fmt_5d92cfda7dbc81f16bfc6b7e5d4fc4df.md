
### `FILE_DIRECTORY_FILE` 0x00000001

The file being created or opened is a directory file. With this flag, the `CreateDisposition` field MUST be set to `FILE_CREATE`, `FILE_OPEN_IF`, or `FILE_OPEN`. With this flag, only the following `CreateOptions` values are valid: `FILE_WRITE_THROUGH`, `FILE_OPEN_FOR_BACKUP_INTENT`, `FILE_DELETE_ON_CLOSE`, and `FILE_OPEN_REPARSE_POINT`. If the file being created or opened already exists and is not a directory file and `FILE_CREATE` is specified in the `CreateDisposition` field, then the server MUST fail the request with `STATUS_OBJECT_NAME_COLLISION`. If the file being created or opened already exists and is not a directory file and `FILE_CREATE` is not specified in the `CreateDisposition` field, then the server MUST fail the request with `STATUS_NOT_A_DIRECTORY`. The server MUST fail an invalid `CreateDisposition` field or an invalid combination of `CreateOptions` flags with `STATUS_INVALID_PARAMETER`.

### `FILE_WRITE_THROUGH` 0x00000002

The server performs file write-through; file data is written to the underlying storage before completing the write operation on this open.

### `FILE_SEQUENTIAL_ONLY` 0x00000004

This indicates that the application intends to read or write at sequential offsets using this handle, so the server SHOULD optimize for sequential access. However, the server MUST accept any access pattern. This flag value is incompatible with the `FILE_RANDOM_ACCESS` value.

### `FILE_NO_INTERMEDIATE_BUFFERING` 0x00000008

File buffering is not performed on this open; file data is not retained in memory upon writing it to, or reading it from, the underlying storage.

### `FILE_SYNCHRONOUS_IO_ALERT` 0x00000010

This bit SHOULD be set to 0 and MUST be ignored by the server.

### `FILE_SYNCHRONOUS_IO_NONALERT` 0x00000020

This bit SHOULD be set to 0 and MUST be ignored by the server.

### `FILE_NON_DIRECTORY_FILE` 0x00000040

If the name of the file being created or opened matches with an existing directory file, the server MUST fail the request with `STATUS_FILE_IS_A_DIRECTORY`. This flag MUST NOT be used with `FILE_DIRECTORY_FILE` or the server MUST fail the request with `STATUS_INVALID_PARAMETER`.

### `FILE_COMPLETE_IF_OPLOCKED` 0x00000100

This bit SHOULD be set to 0 and MUST be ignored by the server.

### `FILE_NO_EA_KNOWLEDGE` 0x00000200

The caller does not understand how to handle extended attributes. If the request includes an `SMB2_CREATE_EA_BUFFER` create context, then the server MUST fail this request with `STATUS_ACCESS_DENIED`. If extended attributes with the `FILE_NEED_EA` flag (see [MS-FSCC] section `2.4.15`) set are associated with the file being opened, then the server MUST fail this request with `STATUS_ACCESS_DENIED`.

### `FILE_RANDOM_ACCESS` 0x00000800

This indicates that the application intends to read or write at random offsets using this handle, so the server SHOULD optimize for random access. However, the server MUST accept any access pattern. This flag value is incompatible with the `FILE_SEQUENTIAL_ONLY` value. If both `FILE_RANDOM_ACCESS` and `FILE_SEQUENTIAL_ONLY` are set, then `FILE_SEQUENTIAL_ONLY` is ignored.

### `FILE_DELETE_ON_CLOSE` 0x00001000

The file MUST be automatically deleted when the last open request on this file is closed. When this option is set, the `DesiredAccess` field MUST include the DELETE flag. This option is often used for temporary files.

### `FILE_OPEN_BY_FILE_ID` 0x00002000

This bit SHOULD be set to 0 and the server MUST fail the request with a `STATUS_NOT_SUPPORTED` error if this bit is set.

### `FILE_OPEN_FOR_BACKUP_INTENT` 0x00004000

The file is being opened for backup intent. That is, it is being opened or created for the purposes of either a backup or a restore operation. The server can check to ensure that the caller is capable of overriding whatever security checks have been placed on the file to allow a backup or restore operation to occur. The server can check for access rights to the file before checking the `DesiredAccess` field.

### `FILE_NO_COMPRESSION` 0x00008000

The file cannot be compressed. This bit is ignored when `FILE_DIRECTORY_FILE` is set in `CreateOptions`.

### `FILE_OPEN_REMOTE_INSTANCE` 0x00000400

This bit SHOULD be set to 0 and MUST be ignored by the server.

### `FILE_OPEN_REQUIRING_OPLOCK` 0x00010000

This bit SHOULD be set to 0 and MUST be ignored by the server.

### `FILE_DISALLOW_EXCLUSIVE` 0x00020000

This bit SHOULD be set to 0 and MUST be ignored by the server.

### `FILE_RESERVE_OPFILTER` 0x00100000

This bit SHOULD be set to 0 and the server MUST fail the request with a `STATUS_NOT_SUPPORTED` error if this bit is set.

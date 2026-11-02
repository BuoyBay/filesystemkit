> FileSystemPaths and all FSP-prefixed symbols are unsupported implementation details. Use CanonicalPath and AbsolutePath from FileSystemKit for the supported exception contract.

---
title: "filesystemkit typed API"
summary: "Source-derived public signatures, properties, parameters, defaults, and return values."
category: "Reference"
order: 21
types: ["FSPNativePathException", "FileEntry", "FileInspectionException", "FileTimestamp", "FileUpdateException"]
dependencies: []
symbols: ["FSPNativePathException", "FileEntry", "FileInspectionException", "FileTimestamp", "FileUpdateException"]
related: ["api.md", "getting-started.md", "index.md"]
---

# filesystemkit typed API

Generated from the package source declarations. Platform guards are shown where present; inherited members are documented on the base type. See the [API contracts](api.md) for behavior and limitations.

## FileInspectionException

`Public Class FileInspectionException Inherits Exception`

Native error code and operation/entryPath are preserved; failure is never an empty listing.

### Constructor

`Public Sub Constructor(operation As String, entryPath As String, code As Integer)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `operation` | `String` | `ByVal` | Required |
| `entryPath` | `String` | `ByVal` | Required |
| `code` | `Integer` | `ByVal` | Required |

Returns: no value.

## FileEntry

`Public Class FileEntry`

Immutable no-follow metadata. LinkTarget is the stored target, not a resolved entryPath.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Path` | `String` | Yes | No | — | — |
| `Kind` | `String` | Yes | No | — | — |
| `Size` | `Integer` | Yes | No | — | — |
| `LinkTarget` | `String` | Yes | No | — | — |
| `ReparseTag` | `Integer` | Yes | No | — | — |
| `IsLink` | `Boolean` | Yes | No | — | — |

### InspectFileEntry

`Public Function InspectFileEntry(entryPath As String, maxLinkBytes As Integer = 65536) As FileEntry`

Inspect the final entryPath component itself, including broken symbolic links.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `entryPath` | `String` | `ByVal` | Required |
| `maxLinkBytes` | `Integer` | `ByVal` | `65536` |

#### Returns

| Type |
| --- |
| `FileEntry` |

### ReadDirectoryEntries

`Public Function ReadDirectoryEntries(entryPath As String, maxEntries As Integer = 100000, maxLinkBytes As Integer = 65536) As Array Of FileEntry`

Nonrecursive listing; reject a linked root, preserve child links and report failures.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `entryPath` | `String` | `ByVal` | Required |
| `maxEntries` | `Integer` | `ByVal` | `100000` |
| `maxLinkBytes` | `Integer` | `ByVal` | `65536` |

#### Returns

| Type |
| --- |
| `Array Of FileEntry` |

### SetFileTimes

`Public Sub SetFileTimes(entryPath As String, accessTime As FileTimestamp, modificationTime As FileTimestamp, followFinalLink As Boolean = False)`

Update either/both timestamps; Nil preserves the existing field. Default updates the final link itself.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `entryPath` | `String` | `ByVal` | Required |
| `accessTime` | `FileTimestamp` | `ByVal` | Required |
| `modificationTime` | `FileTimestamp` | `ByVal` | Required |
| `followFinalLink` | `Boolean` | `ByVal` | `False` |

Returns: no value.

### SetFileCreationTime

`Public Sub SetFileCreationTime(entryPath As String, creationTime As FileTimestamp, followFinalLink As Boolean = False)`

Windows-only creation-time update. Other targets explicitly reject without touching the path.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `entryPath` | `String` | `ByVal` | Required |
| `creationTime` | `FileTimestamp` | `ByVal` | Required |
| `followFinalLink` | `Boolean` | `ByVal` | `False` |

Returns: no value.

### CanonicalPath

`Public Function CanonicalPath(entryPath As String) As String`

Resolve links in an existing path. Failure preserves the native error code.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `entryPath` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `String` |

### AbsolutePath

`Public Function AbsolutePath(entryPath As String) As String`

Make a lexical absolute path without checking existence or resolving links.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `entryPath` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `String` |

## FileTimestamp

`Public Class FileTimestamp`

Exact UTC instant; seconds are floor seconds since 1970, nanoseconds in 0..999999999.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `UnixSeconds` | `Integer` | Yes | No | — | — |
| `Nanoseconds` | `Integer` | Yes | No | — | — |

### Constructor

`Public Sub Constructor(unixSeconds As Integer, nanoseconds As Integer = 0)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `unixSeconds` | `Integer` | `ByVal` | Required |
| `nanoseconds` | `Integer` | `ByVal` | `0` |

Returns: no value.

### FromDateTime

`Public Shared Function FromDateTime(instant As DateTime) As FileTimestamp`

DateTime stores milliseconds; negative timestamps use floor division.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `instant` | `DateTime` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `FileTimestamp` |

## FileUpdateException

`Public Class FileUpdateException Inherits Exception`

Native update failure; ErrorCode preserves errno or GetLastError.

### Constructor

`Public Sub Constructor(operation As String, entryPath As String, code As Integer)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `operation` | `String` | `ByVal` | Required |
| `entryPath` | `String` | `ByVal` | Required |
| `code` | `Integer` | `ByVal` | Required |

Returns: no value.

## FSPNativePathException

`Public Class FSPNativePathException Inherits Exception`

### Constructor

`Public Sub Constructor(operation As String, entryPath As String, code As Integer)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `operation` | `String` | `ByVal` | Required |
| `entryPath` | `String` | `ByVal` | Required |
| `code` | `Integer` | `ByVal` | Required |

Returns: no value.

### FSPCanonicalPath

`Public Function FSPCanonicalPath(entryPath As String) As String`

Resolve an existing path through links; native failures preserve errno/GetLastError.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `entryPath` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `String` |

### FSPAbsolutePath

`Public Function FSPAbsolutePath(entryPath As String) As String`

Lexical absolute entryPath only: no existence check and no link resolution.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `entryPath` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `String` |

---
title: "FileSystemKit API"
summary: "No-follow entries, precise timestamp updates, native errors and directory budgets."
category: "Reference"
order: 20
keywords: ["filesystem", "symlink", "limits", "errors"]
symbols: ["InspectFileEntry", "ReadDirectoryEntries", "FileEntry", "FileInspectionException", "FileTimestamp", "FileUpdateException", "SetFileTimes", "SetFileCreationTime"]
related: ["index.md", "getting-started.md", "typed-api.md"]
---

# FileSystemKit 0.1.0

Link-aware filesystem inspection on macOS, Windows and Linux. Copy
`packages/filesystemkit-0.1.0.buipkg` into a consumer's packages directory, then
`Import filesystemkit`. System APIs are called directly; no extra native library
is required.

| API | Behavior |
| --- | --- |
| `InspectFileEntry(entryPath, maxLinkBytes = 65536)` | Inspect the final component itself, including broken links |
| `ReadDirectoryEntries(entryPath, maxEntries = 100000, maxLinkBytes = 65536)` | Fresh nonrecursive array; reject a linked final root component |
| `FileEntry.Path` | Supplied/constructed path with trailing separators removed except roots |
| `Kind` | file, directory, symlink, junction, reparse, or other |
| `Size` | Native metadata size; file content length only for regular files |
| `IsLink` | True for symlinks and Windows junctions |
| `LinkTarget` | Stored target text, not resolved or opened |
| `ReparseTag` | Raw Windows reparse tag; zero for POSIX/non-reparse entries |

FileEntry properties are read-only. Broken links and loops remain link entries;
directory links are not traversed. Listings exclude `.`/`..`, preserve native
enumeration order and never recurse. Unknown Windows reparse types retain their
tag and kind reparse with an empty target. Symlinks/junctions return their print
name, falling back to the substitute name. Handles and buffers are released before
an operation returns or throws.

## Errors and limits

Missing paths, permission errors, vanished children and enumeration failures throw
FileInspectionException with the original native ErrorCode and operation/path in
Message. Empty arrays mean successfully listed empty directories. Invalid
arguments and exceeded budgets throw InvalidArgumentException. Non-directory or
linked roots are rejected; POSIX open failures retain their native error codes.
No partial result is returned after failure.

maxEntries is nonnegative; zero permits empty directories only. maxLinkBytes is
1..1048576 and bounds the UTF-8 byte length of a target. Invalid POSIX UTF-8 or
Windows UTF-16 is rejected during text conversion. This String API cannot represent
arbitrary non-text POSIX names and does not silently replace bytes.

## Paths and concurrency

Trailing separators are removed first, so `link/` cannot dereference the final
link. Intermediate components follow OS resolution, including symlinks and
explicit `.`/`..`. This is an inspection facility, not a sandbox validator or
an atomic filesystem snapshot.

POSIX listing opens the final root with O_DIRECTORY/O_NOFOLLOW, owns that directory
descriptor, and reads metadata/targets relative to it using no-follow operations.
Renaming a parent cannot redirect those child lookups; individual children can
still change between reads. Windows explicitly opens the root with directory-list
access, including for empty directories, before path-based enumeration
and reads reparse data with OPEN_REPARSE_POINT handles. Concurrent replacements can
race those path lookups. Long Windows paths require native extended path syntax
or an app configured for OS long-path policy.

## Build and verify

```sh
/usr/bin/buoy pkg seal . -o packages/filesystemkit-0.1.0.buipkg
```

The verifier checks six targets, compiles a host C-header ABI check and executes
a fresh sealed consumer. Linux tests compare Python lstat/readlink for Unicode
names, regular files, directories, FIFOs and relative/absolute/broken/looping links.
They cover linked roots with trailing separators, budgets, missing paths,
permission errors and descriptor cleanup after repeated successful/failing calls.
Windows fixture creation requires symlink privilege. The inspection APIs (before
timestamp updates) also pass native execution on macOS ARM64 and real Windows 11 x86_64
lab, including junction/reparse/ACL and handle cleanup tests. Reproduce with
`tests/standard_library_macos.py` and `tests/standard_library_windows.py`. Optional --link-target requires a
complete SDK/runtime. BUOY, BPM_PACKAGER and CC select toolchain commands.

Linux uses optional libc bindings where Buoy's stubs omit symbols; macOS uses
typed dynamic bindings and the documented 64-bit-inode symbols for the same
limitation. The six supported targets are 64-bit desktop ABIs; field offsets and
buffer assumptions are checked against headers by tests/filesystemkit_abi.c.
Independent implementation; no declaresub code is copied.


## Explicit timestamp updates

`New FileTimestamp(unixSeconds, nanoseconds = 0)` represents an exact UTC
instant. Seconds are signed **floor** seconds since 1970-01-01T00:00:00Z;
nanoseconds are 0..999999999. Thus one nanosecond before the epoch is
`FileTimestamp(-1, 999999999)`. Read-only `UnixSeconds` and `Nanoseconds`
retain these values. Input seconds must lie within UTC years 1..9999;
the target filesystem may have a smaller range.
`FileTimestamp.FromDateTime(instant)` converts its exact Unix milliseconds,
independently of its presentation timezone. No floating-point conversion occurs.

- `SetFileTimes(entryPath, accessTime, modificationTime, followFinalLink = False)`
  updates access and/or modification time. Either timestamp may be Nil to
  preserve that field; both Nil reject. POSIX uses `utimensat`/`UTIME_OMIT`;
  Windows uses `SetFileTime` with null pointers for preserved fields.
- `SetFileCreationTime(entryPath, creationTime, followFinalLink = False)` updates
  creation time **on Windows only**. macOS/Linux throw
  `UnsupportedOperationException` before any native update. Nil rejects.
- Native update failures throw `FileUpdateException` with errno/GetLastError
  in `ErrorCode`. Validation errors throw `InvalidArgumentException`.

Updates do not create missing files. By default, the final symlink/reparse point
itself is updated, including a broken POSIX symlink. Set `followFinalLink` True
only when you intend to update its target. Trailing separators are stripped as
for inspection, so adding `/` cannot silently change this policy. Intermediate
components follow normal OS resolution; concurrent replacement can redirect a
path-based lookup. This is not an atomic file-identity or sandbox guarantee.

Windows uses a write-attributes handle with read/write/delete sharing and supports
directories. Its accepted timestamp range starts strictly after
1601-01-01T00:00:00Z (zero FILETIME means preserve), and nanoseconds must
be a multiple of 100; finer input rejects rather than silently rounds in the
package. POSIX passes nanoseconds directly. **A successful native call means the
filesystem accepted the request, not that it retained the requested range or
precision.** Filesystems may round or clamp dates, and later reads may change
access time under the filesystem's atime policy. Windows may defer timestamp
updates until handles are closed, and filesystem-specific UTC/local storage
rules still apply. No API is provided here for ctime (metadata-change time),
POSIX creation time, immutable/BSD flags, or a portable Locked property.

Maintainers run the private validation suite before release. See the repository README for public build instructions.

Native contracts: POSIX utimensat,
Windows SetFileTime.


[Typed API](typed-api.md) · [Getting started](getting-started.md)


## Absolute and resolved paths

`AbsolutePath(entryPath As String) As String` makes a lexical absolute path. It
normalizes separators and dot components without checking whether the target
exists and without resolving symbolic links. `CanonicalPath(entryPath As String)
As String` resolves an existing path through links; missing targets, dangling links
and loops fail with `FileInspectionException` and the native `ErrorCode`.

Both calls accept relative paths against the process working directory. Windows
canonical results may use extended-length paths such as `\\?\` paths. These APIs
are snapshots, not a sandbox or an atomic guarantee against path replacement.
Buoy 0.72 can truncate or silently drop embedded NUL before these calls receive
the String (BUI-003); the library cannot recover the original input.

Linux x64 and Windows x64 sealed consumers passed repeated native fixtures.
macOS is compiler-checked only; its SDK-selected `realpath$DARWIN_EXTSN` binding
still requires native missing-path, dangling-link, loop and allocation tests.
ARM64 execution, Windows denied ACLs and canonical UNC shares remain unverified.

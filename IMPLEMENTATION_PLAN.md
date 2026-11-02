# FileSystemKit timestamp updates

Extend the existing package with `FileTimestamp`, `FileUpdateException`,
`SetFileTimes(path, accessTime, modificationTime, followFinalLink = False)` and
`SetFileCreationTime(path, creationTime, followFinalLink = False)`.

`FileTimestamp` is an immutable pair of signed floor Unix seconds and
nonnegative nanoseconds. Validate UTC years 1–9999 and nanoseconds 0–999999999.
Provide exact DateTime millisecond conversion, including negative values.
Nil preserves an individual access/modification field; both Nil reject.
Strip trailing separators consistently with existing inspection APIs.

Use POSIX `utimensat` with `UTIME_OMIT` for atomic field preservation and
`AT_SYMLINK_NOFOLLOW` by default. Linux binds the optional libc symbol;
macOS uses the package's existing typed dlsym strategy. Use two 64-bit timespec
pairs, checked against native headers.

Use Windows `CreateFileW(FILE_WRITE_ATTRIBUTES)` and `SetFileTime`, preserving
unspecified fields through null pointers. Open reparse points by default;
follow-final-link is explicit. Windows rejects before 1601 and sub-100 ns
precision. Creation time is Windows-only; other targets reject explicitly.
BSD flags/immutable bits and a general Locked API remain outside this contract.

Risks: filesystem precision/range may truncate/clamp even successful native
calls; operating-system access policies vary; intermediate symlinks are still
resolved and this is not a sandbox or an atomic path-identity guarantee.
macOS/Windows are compiler-checked, not claimed runtime-verified in this pass.

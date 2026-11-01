---
title: "FileSystemKit"
summary: "Link-aware filesystem inspection and precise timestamp updates with explicit errors."
category: "Overview"
order: 0
keywords: ["filesystem", "symlink", "directory", "reparse"]
symbols: ["InspectFileEntry", "ReadDirectoryEntries", "FileTimestamp", "FileUpdateException", "SetFileTimes", "SetFileCreationTime"]
related: ["getting-started.md", "api.md", "typed-api.md"]
---

# FileSystemKit

`FileTimestamp` and `SetFileTimes` add precise access/modification updates with
explicit final-symlink handling. `SetFileCreationTime` is Windows-only. See the
[API contracts](api.md) for precision, preserved fields and native-validation scope.

Import filesystemkit for immutable no-follow metadata and immediate directory
listings. Broken links, loops and Windows junctions are preserved. Native failures
throw exceptions instead of returning empty results. Handles and buffers are
scoped to each operation.

Listings are nonrecursive, have entry/target budgets and return no partial array
on failure. Concurrent changes can race inspection; intermediate symlinks resolve
normally. This is not an atomic snapshot or containment security check.

See the [API reference](api.md).

# Inspect filesystem entries and update timestamps explicitly

Link-aware filesystem inspection on macOS, Windows and Linux. Copy
`packages/filesystemkit-0.1.0.buipkg` into a consumer's packages directory, then
`Import filesystemkit`. System APIs are called directly; no extra native library
is required.

FileEntry properties are read-only. Broken links and loops remain link entries;
directory links are not traversed. Listings exclude `.`/`..`, preserve native
enumeration order and never recurse. Unknown Windows reparse types retain their
tag and kind reparse with an empty target. Symlinks/junctions return their print
name, falling back to the substitute name. Handles and buffers are released before
an operation returns or throws.

## Install

From a Buoy consumer project with BPM installed:

```sh
bpm add filesystemkit --git https://github.com/BuoyBay/filesystemkit.git --rev refs/heads/main
```

BPM records the resolved commit in its lockfile. Keep that lockfile with the
consumer project for reproducible installs.

## API entrypoints

- `Import filesystemkit`
- `InspectFileEntry(entryPath, maxLinkBytes = 65536)`
- `ReadDirectoryEntries(entryPath, maxEntries = 100000, maxLinkBytes = 65536)`
- `Kind`
- `Size`

## Build from source

With the native `build-packages` driver from [devtools](https://github.com/BuoyBay/devtools)
on your `PATH`, run from this checkout:

```sh
build-packages --root "$PWD" --compile-consumer filesystemkit
```

The driver checks source targets and compiles an import consumer. See the
[API guide](docs/API.md) for behavior, limits and any host prerequisites.

## Local Buoy setup

On your own machine, install Buoy and run:

```sh
buoy ai init --both
```


## Contributing

[Contributing](CONTRIBUTING.md) · [License](LICENSE)

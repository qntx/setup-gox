# setup-gox

GitHub Action to set up [gox](https://github.com/qntx/gox) cross-compilation environment.

## Usage

```yaml
- uses: qntx/setup-gox@main
  with:
    go-version: "stable"      # optional, default: stable
    gox-version: "latest"     # optional, default: latest
    zig-version: ""           # optional, pre-install specific Zig version
```

## Example Workflows

### Basic Cross-Compilation

```yaml
name: Build

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: qntx/setup-gox@main

      - name: Build for Linux
        run: gox build --os linux --arch amd64 -o myapp

      - name: Build for Windows
        run: gox build --os windows --arch amd64 -o myapp.exe
```

### Matrix Build

```yaml
name: Cross-Compile

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        target:
          - { os: linux, arch: amd64 }
          - { os: linux, arch: arm64 }
          - { os: windows, arch: amd64 }
          - { os: darwin, arch: amd64 }

    steps:
      - uses: actions/checkout@v4

      - uses: qntx/setup-gox@main

      - name: Build
        run: gox build --os ${{ matrix.target.os }} --arch ${{ matrix.target.arch }} -o app-${{ matrix.target.os }}-${{ matrix.target.arch }}

      - uses: actions/upload-artifact@v4
        with:
          name: app-${{ matrix.target.os }}-${{ matrix.target.arch }}
          path: app-${{ matrix.target.os }}-${{ matrix.target.arch }}*
```

### With Specific Versions

```yaml
- uses: qntx/setup-gox@main
  with:
    go-version: "1.25"
    gox-version: "v0.6.0"
    zig-version: "0.15.1"
```

## Inputs

| Input | Description | Default |
| :--- | :--- | :--- |
| `go-version` | Go version to install | `stable` |
| `gox-version` | gox version (`latest` or specific tag) | `latest` |
| `zig-version` | Pre-install Zig version (optional) | `` |

## Outputs

| Output | Description |
| :--- | :--- |
| `gox-path` | Path to installed gox binary |

## Notes

- Zig is auto-downloaded by gox on first use if not pre-installed
- Pre-installing Zig via `zig-version` can speed up builds when running multiple gox commands
- The action caches Go modules via `actions/setup-go`

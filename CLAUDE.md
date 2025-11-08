# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Important: Git Commit Guidelines

When creating git commits in this repository, DO NOT add Claude Code credits to commit messages. Create clean commit messages without attribution lines like:
- "Generated with [Claude Code](https://claude.com/claude-code)"
- "Co-Authored-By: Claude <noreply@anthropic.com>"

## Project Overview

This is an unofficial binary distribution of CTP (Comprehensive Transaction Platform) SDK for CMake projects. The repository provides a CMake interface to link against CTP's proprietary trading APIs without manual configuration.

**Important:** CTP SDK is proprietary software from SFIT. This repository only provides convenient distribution - users must ensure they have proper licensing.

## Repository Structure

```
versions/
└── <version>/              # e.g., 6.7.10/
    ├── linux64/            # Linux 64-bit binaries
    ├── win32/              # Windows 32-bit binaries
    └── win64/              # Windows 64-bit binaries
        ├── include/        # Headers (same across platforms)
        │   ├── ThostFtdcMdApi.h      # Market data API
        │   ├── ThostFtdcTraderApi.h  # Trading API
        │   └── ...
        └── lib/            # Shared libraries
            ├── thostmduserapi_se.*   # Market data library
            ├── thosttraderapi_se.*   # Trading library
            └── error.{dtd,xml}
```

## CMake Configuration

### Key Variables

- `CTP_VERSION`: SDK version to use (default: 6.7.10, validated against `CTP_AVAILABLE_VERSIONS`)
- `CTP_BASE_DIR`: Override path to SDK directory (default: `${CMAKE_CURRENT_SOURCE_DIR}/versions/${CTP_VERSION}/${CTP_PLATFORM}`)

### Platform Detection

The CMakeLists.txt automatically detects platform based on `CMAKE_SIZEOF_VOID_P`:
- `win32` (Windows 32-bit): `CMAKE_SIZEOF_VOID_P EQUAL 4`
- `win64` (Windows 64-bit): `CMAKE_SIZEOF_VOID_P EQUAL 8`
- `linux64` (Linux 64-bit): default for non-Windows/Apple
- **macOS is NOT supported** - CMake will error with "CTP does not support macOS"

### Imported Targets

The CMakeLists.txt creates two IMPORTED SHARED library targets:
- `CTP::MdApi`: Market data API (`thostmduserapi_se`)
- `CTP::TraderApi`: Trading API (`thosttraderapi_se`)

Both targets provide:
- `IMPORTED_LOCATION`: Path to the .dll/.so file
- `IMPORTED_IMPLIB` (Windows only): Path to .lib import library
- `INTERFACE_INCLUDE_DIRECTORIES`: Include paths for headers

## Usage Patterns

### Method 1: FetchContent (Recommended)

```cmake
include(FetchContent)
set(CTP_VERSION "6.7.10")  # Optional: defaults to 6.7.10
FetchContent_Declare(
    ctp
    GIT_REPOSITORY https://github.com/euyuil/ctp.git
    GIT_TAG main
)
FetchContent_MakeAvailable(ctp)
target_link_libraries(your_app PRIVATE CTP::MdApi CTP::TraderApi)
```

### Method 2: Git Submodule

```bash
git submodule add https://github.com/euyuil/ctp.git third_party/ctp
```

```cmake
set(CTP_VERSION "6.7.10")  # Optional
add_subdirectory(third_party/ctp)
target_link_libraries(your_app PRIVATE CTP::MdApi CTP::TraderApi)
```

### Version Specification

Users can specify version via:
1. `set(CTP_VERSION "x.x.x")` in CMakeLists.txt before including ctp
2. Command line: `cmake -S . -B build -DCTP_VERSION="x.x.x"`
3. Overriding `CTP_BASE_DIR`: `cmake -S . -B build -DCTP_BASE_DIR="/path/to/ctp"`

## Common Issues

### CMake Fails with "CTP does not support macOS"
- CTP SDK does not provide macOS binaries. Users must develop on Linux or Windows.

### "CTP base directory not found"
- The default path `${CMAKE_CURRENT_SOURCE_DIR}/versions/${CTP_VERSION}/${CTP_PLATFORM}` doesn't exist
- Solution: Set `-DCTP_BASE_DIR` to correct path
- For Visual Studio: Ensure correct architecture with `-A Win32` or `-A x64`

### "CTP version X not available"
- Version requested not in `CTP_AVAILABLE_VERSIONS` list
- Check and update `CTP_AVAILABLE_VERSIONS` in CMakeLists.txt if adding new versions

## Adding New CTP Versions

When adding a new CTP SDK version:

1. Create directory structure: `versions/<version>/{linux64,win32,win64}/{include,lib}`
2. Copy headers to `include/` (same across all platforms)
3. Copy platform-specific binaries to respective `lib/` directories
4. Update `CTP_AVAILABLE_VERSIONS` in CMakeLists.txt
5. Update README.md "Versions" section
6. Test with both 32-bit and 64-bit Windows targets

Ensure consistent naming:
- Windows: `thostmduserapi_se.dll`, `thostmduserapi_se.lib`, `thosttraderapi_se.dll`, `thosttraderapi_se.lib`
- Linux: `thostmduserapi_se.so`, `thosttraderapi_se.so`

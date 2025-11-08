# CTP Binary Distribution

Unofficial binary distribution of CTP (Comprehensive Transaction Platform) SDK for CMake projects.

## Usage

### Method 1: FetchContent (Recommended)

```cmake
include(FetchContent)

# Optional: Specify CTP version (defaults to 6.7.10)
set(CTP_VERSION "6.7.10")

FetchContent_Declare(
    ctp
    GIT_REPOSITORY https://github.com/euyuil/ctp.git
    GIT_TAG main
)
FetchContent_MakeAvailable(ctp)

target_link_libraries(your_app PRIVATE CTP::MdApi CTP::TraderApi)
```

**Specify version via command line:**
```bash
cmake -S . -B build -DCTP_VERSION="6.7.10"
```

### Method 2: Git Submodule

```bash
git submodule add https://github.com/euyuil/ctp.git third_party/ctp
```

```cmake
# Optional: Specify CTP version (defaults to 6.7.10)
set(CTP_VERSION "6.7.10")

add_subdirectory(third_party/ctp)
target_link_libraries(your_app PRIVATE CTP::MdApi CTP::TraderApi)
```

**Specify version via command line:**
```bash
cmake -S . -B build -DCTP_VERSION="6.7.10"
```

### Platform & SDK location

- Windows 32-bit is supported. On Windows CMake will detect 32/64-bit targets automatically and set the CTP platform to `win32` or `win64` based on the CMake pointer size.
- Override the SDK location when configuring CMake, for example:

  ```powershell
  cmake -S . -B build -DCTP_BASE_DIR="D:/path/to/ctp/versions/6.7.10/win32"
  ```

**Notes**:

- For Visual Studio, use `-A Win32` / `-A x64` to select the generator architecture explicitly.
- If CMake cannot find the specified `CTP_BASE_DIR`, configuration will fail with an explanatory message.

## Versions

- v6.7.10 (2025-04-22)

## License

CTP SDK is proprietary software. This repository only provides convenient distribution.
Please ensure you have proper license from SFIT before using in production.

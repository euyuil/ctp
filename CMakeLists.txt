cmake_minimum_required(VERSION 3.15)
project(CTP VERSION 6.7.10 LANGUAGES CXX)

# 设置版本（可以作为参数传入）
set(CTP_VERSION "6.7.10" CACHE STRING "CTP SDK version")

# 用分号分隔的可用版本列表
set(CTP_AVAILABLE_VERSIONS "6.7.10")

if(NOT CTP_VERSION IN_LIST CTP_AVAILABLE_VERSIONS)
    message(FATAL_ERROR
        "CTP version ${CTP_VERSION} not available. "
        "Available versions: ${CTP_AVAILABLE_VERSIONS}")
endif()

# 检测平台
if(APPLE)
    message(FATAL_ERROR "CTP does not support macOS")
elseif(WIN32)
    # On Windows, detect whether we're targeting 32-bit or 64-bit by
    # checking the pointer size. CMake defines CMAKE_SIZEOF_VOID_P as 8 for
    # 64-bit and 4 for 32-bit builds.
    if(CMAKE_SIZEOF_VOID_P EQUAL 8)
        set(CTP_PLATFORM "win64")
    elseif(CMAKE_SIZEOF_VOID_P EQUAL 4)
        set(CTP_PLATFORM "win32")
    else()
        message(FATAL_ERROR "Unknown pointer size ${CMAKE_SIZEOF_VOID_P} — cannot determine Windows platform (win32/win64)")
    endif()
else()
    set(CTP_PLATFORM "linux64")
endif()

## Allow overriding the SDK base directory via cache (useful in CI or if SDK is outside repo)
set(CTP_BASE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/versions/${CTP_VERSION}/${CTP_PLATFORM}" CACHE PATH "CTP SDK base directory (e.g. D:/path/to/ctp/versions/${CTP_VERSION}/${CTP_PLATFORM})")

# Basic sanity check: ensure base dir exists
if(NOT EXISTS "${CTP_BASE_DIR}")
    message(FATAL_ERROR "CTP base directory not found: ${CTP_BASE_DIR}\nPlease set -DCTP_BASE_DIR to point to your unpacked CTP SDK for version ${CTP_VERSION} and platform ${CTP_PLATFORM}.")
endif()

# 创建 INTERFACE library (header-only style)
add_library(CTP::MdApi SHARED IMPORTED GLOBAL)
add_library(CTP::TraderApi SHARED IMPORTED GLOBAL)

# 设置 MdApi 和 TraderApi
if(WIN32)
    # On Windows with MSVC, the linker needs the import library (.lib).
    # File names on Windows (no 'lib' prefix)
    set_target_properties(CTP::MdApi PROPERTIES
        IMPORTED_LOCATION "${CTP_BASE_DIR}/lib/thostmduserapi_se.dll"
        IMPORTED_IMPLIB "${CTP_BASE_DIR}/lib/thostmduserapi_se.lib"
        INTERFACE_INCLUDE_DIRECTORIES "${CTP_BASE_DIR}/include"
    )

    set_target_properties(CTP::TraderApi PROPERTIES
        IMPORTED_LOCATION "${CTP_BASE_DIR}/lib/thosttraderapi_se.dll"
        IMPORTED_IMPLIB "${CTP_BASE_DIR}/lib/thosttraderapi_se.lib"
        INTERFACE_INCLUDE_DIRECTORIES "${CTP_BASE_DIR}/include"
    )
else()
    set_target_properties(CTP::MdApi PROPERTIES
        IMPORTED_LOCATION "${CTP_BASE_DIR}/lib/thostmduserapi_se.so"
        INTERFACE_INCLUDE_DIRECTORIES "${CTP_BASE_DIR}/include"
    )

    set_target_properties(CTP::TraderApi PROPERTIES
        IMPORTED_LOCATION "${CTP_BASE_DIR}/lib/thosttraderapi_se.so"
        INTERFACE_INCLUDE_DIRECTORIES "${CTP_BASE_DIR}/include"
    )
endif()

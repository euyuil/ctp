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
if(WIN32)
    set(CTP_PLATFORM "win64")
    set(LIB_SUFFIX ".dll")
elseif(APPLE)
    message(FATAL_ERROR "CTP does not support macOS")
else()
    set(CTP_PLATFORM "linux64")
    set(LIB_SUFFIX ".so")
endif()

set(CTP_BASE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/versions/${CTP_VERSION}/${CTP_PLATFORM}")

# 检查文件是否存在
if(NOT EXISTS "${CTP_BASE_DIR}")
    message(FATAL_ERROR "CTP ${CTP_VERSION} for ${CTP_PLATFORM} not found")
endif()

# 创建 INTERFACE library (header-only style)
add_library(CTP::MdApi SHARED IMPORTED GLOBAL)
add_library(CTP::TraderApi SHARED IMPORTED GLOBAL)

# 设置 MdApi
set_target_properties(CTP::MdApi PROPERTIES
    IMPORTED_LOCATION "${CTP_BASE_DIR}/lib/libthostmduserapi_se${LIB_SUFFIX}"
    INTERFACE_INCLUDE_DIRECTORIES "${CTP_BASE_DIR}/include"
)

# 设置 TraderApi
set_target_properties(CTP::TraderApi PROPERTIES
    IMPORTED_LOCATION "${CTP_BASE_DIR}/lib/libthosttraderapi_se${LIB_SUFFIX}"
    INTERFACE_INCLUDE_DIRECTORIES "${CTP_BASE_DIR}/include"
)

# 安装配置（如果需要install）
install(TARGETS CTP::MdApi CTP::TraderApi
    EXPORT CTPTargets
)

# Export配置
install(EXPORT CTPTargets
    FILE CTPTargets.cmake
    NAMESPACE CTP::
    DESTINATION lib/cmake/CTP
)

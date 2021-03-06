# Configurations
cmake_minimum_required(VERSION 3.14.1)
get_filename_component(CMAKE_SOURCE_REALPATH "${CMAKE_SOURCE_DIR}/../" REALPATH)

set(PROJECT_CONFIG_NAME "Sample" CACHE STRING "Project name")
set(PROJECT_CONFIG_VERSION "1.0.0" CACHE STRING "")
set(PROJECT_CONFIG_CPP_STANDARD "11" CACHE STRING "11")
set(PROJECT_CONFIG_ARCH "" CACHE STRING "Architecture")
set(PROJECT_CONFIG_GROUP "" CACHE STRING "Group")

set(PROJECT_ROOT_PATH ${CMAKE_SOURCE_REALPATH})
set(PROJECT_BUILD_DIR ${PROJECT_ROOT_PATH}/out/build-${PROJECT_CONFIG_GROUP} CACHE INTERNAL "")
set(PROJECT_CONAN_DIR ${PROJECT_ROOT_PATH}/out/conan-${PROJECT_CONFIG_GROUP} CACHE INTERNAL "")
set(PROJECT_CODE_DIR ${PROJECT_ROOT_PATH}/code CACHE INTERNAL "")

# Platform configurations
set(DEVELOPMENT_TEAM_ID "" CACHE STRING "Development Team ID")
set(CODE_SIGN_IDENTITY "" CACHE STRING "Sign Identity")
set(FRAMEWORK_NAME "${PROJECT_CONFIG_NAME}" CACHE STRING "Framework Name")
set(FRAMEWORK_BUNDLE_IDENTIFIER "com.prsolucoes.sample" CACHE STRING "Bundle Identifier")
set(DEVICE_FAMILY "1,2" CACHE STRING "Device Family")
set(CMAKE_OSX_ARCHITECTURES "${PROJECT_CONFIG_ARCH}")

message(STATUS "ARCH: ${PROJECT_CONFIG_ARCH}")

# Project
project(${PROJECT_CONFIG_NAME})

# Conan
include(${PROJECT_CONAN_DIR}/conanbuildinfo.cmake)
conan_basic_setup()

# Merge lists
set(
    TARGET_HEADER_FILES     
    ${PROJECT_CODE_DIR}/include/lib.hpp
)

set(
    TARGET_SOURCE_FILES     
    ${PROJECT_CODE_DIR}/src/lib.cpp
)

set(
    PROJECT_SOURCE_FILES
    ${PROJECT_SOURCE_FILES}
    ${TARGET_SOURCE_FILES}
    ${TARGET_HEADER_FILES}
    CACHE INTERNAL ""
)

set(
    PROJECT_HEADER_FILES
    ${TARGET_HEADER_FILES}
    CACHE INTERNAL ""
)

# Framework links
set(PROJECT_FRAMEWORK_LINKS
    ${PROJECT_FRAMEWORK_LINKS}
	"-framework Foundation"
	CACHE INTERNAL ""
)

# C flags
set(PROJECT_CMAKE_C_FLAGS "${PROJECT_CMAKE_C_FLAGS} -fobjc-arc -fobjc-abi-version=2" CACHE INTERNAL "")

# CXX flags
set(PROJECT_CMAKE_CXX_FLAGS "${PROJECT_CMAKE_CXX_FLAGS} -fobjc-arc -fvisibility-inlines-hidden -fobjc-abi-version=2 -stdlib=libc++" CACHE INTERNAL "")

# Flags
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${PROJECT_CMAKE_C_FLAGS}" CACHE INTERNAL "")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${PROJECT_CMAKE_CXX_FLAGS}" CACHE INTERNAL "")

set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_DEBUG} -O0 -g")
set(CMAKE_CXX_FLAGS_MINSIZEREL "${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_MINSIZEREL} -Os")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_RELWITHDEBINFO} -O2 -g")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_RELEASE} -O3")

# Project
include_directories(${PROJECT_CODE_DIR}/include)
add_library(${PROJECT_CONFIG_NAME} SHARED ${PROJECT_SOURCE_FILES})

target_link_libraries(${PROJECT_CONFIG_NAME} "${PROJECT_FRAMEWORK_LINKS}")
target_link_libraries(${PROJECT_CONFIG_NAME} "${CONAN_LIBS}")

set_target_properties(${PROJECT_CONFIG_NAME} PROPERTIES
    CXX_STANDARD "${PROJECT_CONFIG_CPP_STANDARD}"
    CXX_STANDARD_REQUIRED YES
    CXX_EXTENSIONS NO
    CXX_VISIBILITY_PRESET default
    PUBLIC_HEADER "${PROJECT_HEADER_FILES}"
    FRAMEWORK TRUE
    FRAMEWORK_VERSION A
    MACOSX_FRAMEWORK_IDENTIFIER "${FRAMEWORK_BUNDLE_IDENTIFIER}"
    MACOSX_FRAMEWORK_BUNDLE_VERSION "${PROJECT_CONFIG_VERSION}"
    MACOSX_FRAMEWORK_SHORT_VERSION_STRING "${PROJECT_CONFIG_VERSION}"
    XCODE_ATTRIBUTE_IPHONEOS_DEPLOYMENT_TARGET "${CMAKE_OSX_DEPLOYMENT_TARGET}"
    XCODE_ATTRIBUTE_CODE_SIGN_IDENTITY "${CODE_SIGN_IDENTITY}"
    XCODE_ATTRIBUTE_CODE_SIGNING_REQUIRED NO
    XCODE_ATTRIBUTE_DEVELOPMENT_TEAM "${DEVELOPMENT_TEAM_ID}"
    XCODE_ATTRIBUTE_TARGETED_DEVICE_FAMILY "${DEVICE_FAMILY}"
    XCODE_ATTRIBUTE_ONLY_ACTIVE_ARCH YES
    XCODE_ATTRIBUTE_SKIP_INSTALL YES
    XCODE_ATTRIBUTE_INSTALL_PATH "@loader_path/Frameworks"
    XCODE_ATTRIBUTE_ENABLE_BITCODE YES

    # Is "current version" in semantic format in Mach-O binary file
    VERSION "${PROJECT_CONFIG_VERSION}"
    # Is "compatibility version" in semantic format in Mach-O binary file
    SOVERSION "${PROJECT_CONFIG_VERSION}"
)

# Adapt install directory to allow distributing dylibs/frameworks in user's frameworks/application bundle
set_target_properties(${PROJECT_CONFIG_NAME} PROPERTIES INSTALL_NAME_DIR "@rpath")

if(NOT CMAKE_SKIP_BUILD_RPATH)
    set_target_properties(${PROJECT_CONFIG_NAME} PROPERTIES BUILD_WITH_INSTALL_NAME_DIR TRUE)
endif()

target_compile_options(${PROJECT_CONFIG_NAME} PUBLIC "${PROJECT_COMPILE_OPTIONS}")
target_compile_definitions(${PROJECT_CONFIG_NAME} PRIVATE PROJECT_CONFIG_VERSION="${PROJECT_CONFIG_VERSION}")

# Copyright (c) 2017 The Khronos Group Inc.
#
# SPDX-License-Identifier: Apache-2.0
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# Author:
#

file(GLOB LOCAL_HEADERS "*.h")
file(GLOB LOCAL_SOURCE "*.cpp")
file(GLOB VULKAN_SHADERS "vulkan_shaders/*.glsl")

# For including compiled shaders
include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(${PROJECT_BINARY_DIR}/include)

if (ANDROID)
#    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -Werror -Wall -Wno-unused-function -Wno-unused-parameter")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99 -Werror -Wall -Wno-unused-function -Wno-unused-parameter")
    add_library(openxr SHARED IMPORTED)
    set_target_properties(openxr PROPERTIES IMPORTED_LOCATION
            ${OPENXR_PATH}/${ANDROID_ABI}/libopenxr.so)

    add_library(hello_xr SHARED
            ${ANDROID_NDK}/sources/android/native_app_glue/android_native_app_glue.c
            ${LOCAL_SOURCE}
            ${LOCAL_HEADERS}
            ${VULKAN_SHADERS})
else()
    add_executable(hello_xr
        ${LOCAL_SOURCE}
        ${LOCAL_HEADERS}
        ${VULKAN_SHADERS})
endif()
set_target_properties(hello_xr PROPERTIES FOLDER ${SAMPLES_FOLDER})

source_group("Headers" FILES ${LOCAL_HEADERS})
source_group("Shaders" FILES ${VULKAN_SHADERS})

compile_glsl(run_hello_xr_glsl_compiles ${VULKAN_SHADERS})

add_dependencies(hello_xr
    generate_openxr_header
    run_hello_xr_glsl_compiles
)

target_include_directories(hello_xr
    PRIVATE
    ${PROJECT_SOURCE_DIR}/src
    ${PROJECT_SOURCE_DIR}/src/common

    # for OpenXR headers
    ${PROJECT_SOURCE_DIR}/include
    ${PROJECT_BINARY_DIR}/include

    # for helper headers
    ${PROJECT_SOURCE_DIR}/external/include
)
if(ANDROID)
    target_include_directories(hello_xr
        PRIVATE
        ${ANDROID_NDK}/sources/android/native_app_glue
    )
endif()

if(VulkanHeaders_INCLUDE_DIRS)
    target_include_directories(hello_xr
        PRIVATE
        ${VulkanHeaders_INCLUDE_DIRS}
    )
endif()
if(ANDROID)
    target_link_libraries(hello_xr
            openxr
            android
            log
            EGL
            GLESv3
            )
else()
    target_link_libraries(hello_xr openxr_loader)
endif()
if(TARGET openxr-gfxwrapper)
    target_link_libraries(hello_xr openxr-gfxwrapper)
endif()
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    if(MSVC)
        target_compile_definitions(hello_xr PRIVATE _CRT_SECURE_NO_WARNINGS)
        target_compile_options(hello_xr PRIVATE /Zc:wchar_t /Zc:forScope /W4 /WX)
    endif()
    target_link_libraries(hello_xr ole32)
    if(D3D_D3D11_FOUND AND D3D_DIRECTXCOLORS_INCLUDE_DIR)
        target_link_libraries(hello_xr ${D3D_D3D11_LIBRARIES})
        target_include_directories(hello_xr PRIVATE ${D3D_D3D11_INCLUDE_DIR} ${D3D_DIRECTXCOLORS_INCLUDE_DIR})
    endif()
    if(D3D_D3D12_FOUND AND D3D_DIRECTXCOLORS_INCLUDE_DIR)
        target_link_libraries(hello_xr ${D3D_D3D12_LIBRARIES})
        target_include_directories(hello_xr PRIVATE ${D3D_D3D12_INCLUDE_DIR} ${D3D_DIRECTXCOLORS_INCLUDE_DIR})
    endif()
    if(NOT D3D_DIRECTXCOLORS_INCLUDE_DIR)
        target_compile_definitions(hello_xr PRIVATE -DMISSING_DIRECTX_COLORS)
    endif()
endif()

if(Vulkan_LIBRARY)
    target_link_libraries(hello_xr ${Vulkan_LIBRARY})
endif()

if (NOT ANDROID)
install(TARGETS hello_xr
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
endif()

if(NOT WIN32)
    install(FILES hello_xr.1 DESTINATION ${CMAKE_INSTALL_MANDIR}/man1/ COMPONENT ManPages)
endif()

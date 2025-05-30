# ~~~
# Copyright (c) 2014-2024 Valve Corporation
# Copyright (c) 2014-2024 LunarG, Inc.
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
# ~~~

if (ANDROID)
    add_library(vk_extension_layer_tests MODULE)
else()
    add_executable(vk_extension_layer_tests)

    target_compile_definitions(vk_extension_layer_tests PRIVATE
        SHADER_OBJECT_BINARY_PATH="$<TARGET_FILE_DIR:VkLayer_khronos_shader_object>"
    )

    target_sources(vk_extension_layer_tests PRIVATE
        shader_object_tests.cpp
    )
endif()

# Needed for VK_KHR_PORTABILITY_SUBSET_EXTENSION_NAME
target_compile_definitions(vk_extension_layer_tests PRIVATE VK_ENABLE_BETA_EXTENSIONS)

lunarg_target_compiler_configurations(vk_extension_layer_tests ${BUILD_WERROR})

target_sources(vk_extension_layer_tests PRIVATE
    extension_layer_tests.cpp
    synchronization2_tests.cpp
    decompression_tests.cpp
    vkrenderframework.cpp
    vktestbinding.cpp
    vktestframework.cpp
    test_environment.cpp
)

set_target_properties(vk_extension_layer_tests PROPERTIES VS_DEBUGGER_ENVIRONMENT "VK_LAYER_PATH=$<TARGET_FILE_DIR:VkLayer_khronos_synchronization2>")

add_dependencies(vk_extension_layer_tests VkLayer_khronos_synchronization2 VkLayer_khronos_shader_object VkLayer_khronos_memory_decompression)

target_include_directories(vk_extension_layer_tests PRIVATE .)

find_package(SPIRV-Headers REQUIRED CONFIG QUIET)
target_link_libraries(vk_extension_layer_tests PRIVATE SPIRV-Headers::SPIRV-Headers)

find_package(SPIRV-Tools-opt REQUIRED CONFIG QUIET)
target_link_libraries(vk_extension_layer_tests PRIVATE SPIRV-Tools-opt)

find_package(SPIRV-Tools REQUIRED CONFIG QUIET)

# See https://github.com/KhronosGroup/SPIRV-Tools/issues/3909 for background on this.
# The targets available from SPIRV-Tools change depending on how SPIRV_TOOLS_BUILD_STATIC is set.
# Try to handle all possible combinations so that we work with externally built packages.
if (TARGET SPIRV-Tools)
    target_link_libraries(vk_extension_layer_tests PRIVATE SPIRV-Tools)
elseif(TARGET SPIRV-Tools-static)
    target_link_libraries(vk_extension_layer_tests PRIVATE SPIRV-Tools-static)
elseif(TARGET SPIRV-Tools-shared)
    target_link_libraries(vk_extension_layer_tests PRIVATE SPIRV-Tools-shared)
else()
    message(FATAL_ERROR "Cannot determine SPIRV-Tools target name")
endif()

find_package(GTest REQUIRED CONFIG QUIET)

find_package(glslang REQUIRED CONFIG QUIET)

find_package(volk QUIET CONFIG)

target_link_libraries(vk_extension_layer_tests PRIVATE
    VkExtLayer_utils
    Vulkan::LayerSettings
    Vulkan::Headers
    GTest::gtest
    GTest::gtest_main
    SPIRV-Headers::SPIRV-Headers
    glslang::SPIRV
    glslang::SPVRemapper
    ${CMAKE_DL_LIBS}
    volk::volk_headers
    $<TARGET_NAME_IF_EXISTS:PkgConfig::XCB>
    $<TARGET_NAME_IF_EXISTS:PkgConfig::X11>
    $<TARGET_NAME_IF_EXISTS:PkgConfig::WAYLAND_CLIENT>
)

# Needed for volk to work correctly
target_compile_definitions(vk_extension_layer_tests PRIVATE VK_NO_PROTOTYPES)

if (ANDROID)
    add_subdirectory(android)
    return()
endif()

include(GoogleTest)
gtest_discover_tests(vk_extension_layer_tests DISCOVERY_TIMEOUT 100)

install(TARGETS vk_extension_layer_tests)

# Copyright (c) Meta Platforms, Inc. and affiliates.
# All rights reserved.
#
# This source code is licensed under the license found in the
# LICENSE file in the root directory of this source tree.

cmake_minimum_required(VERSION 3.19)

include(${CMAKE_CURRENT_SOURCE_DIR}/../../Utils.cmake)

if(TORCHAO_BUILD_ATEN_OPS)
    find_package(Torch REQUIRED)
    add_library(torchao_ops_embedding_xbit_aten OBJECT
        op_embedding_xbit_aten.cpp
    )
    target_link_torchao_parallel_backend(torchao_ops_embedding_xbit_aten "${TORCHAO_PARALLEL_BACKEND}")
    if (TORCHAO_BUILD_CPU_AARCH64)
        target_link_libraries(torchao_ops_embedding_xbit_aten PRIVATE torchao_kernels_aarch64)
    endif()
    target_include_directories(torchao_ops_embedding_xbit_aten PRIVATE "${TORCH_INCLUDE_DIRS}")
    target_link_libraries(torchao_ops_embedding_xbit_aten PRIVATE "${TORCH_LIBRARIES}")
    target_compile_definitions(torchao_ops_embedding_xbit_aten PRIVATE USE_ATEN=1)
endif()

if(TORCHAO_BUILD_EXECUTORCH_OPS)
    # ExecuTorch package is not required, but EXECUTORCH_INCLUDE_DIRS and EXECUTORCH_LIBRARIES must
    # be defined and EXECUTORCH_LIBRARIES must include the following libraries installed by ExecuTorch:
    # libexecutorch.a
    # libextension_threadpool.a
    # libcpuinfo.a
    # libpthreadpool.a
    if(NOT DEFINED EXECUTORCH_INCLUDE_DIRS AND NOT DEFINED EXECUTORCH_LIBRARIES)
        message(WARNING "EXECUTORCH_INCLUDE_DIRS and EXECUTORCH_LIBRARIES are not defined. Looking for ExecuTorch.")
        find_package(ExecuTorch HINTS ${CMAKE_PREFIX_PATH}/executorch/share/cmake)
    endif()
    add_library(torchao_ops_embedding_xbit_executorch OBJECT
        op_embedding_xbit_executorch.cpp
    )
    target_link_torchao_parallel_backend(torchao_ops_embedding_xbit_executorch executorch)
    target_include_directories(torchao_ops_embedding_xbit_executorch PRIVATE "${EXECUTORCH_INCLUDE_DIRS}")
    target_compile_definitions(torchao_ops_embedding_xbit_executorch PRIVATE USE_EXECUTORCH=1)
    target_link_libraries(torchao_ops_embedding_xbit_executorch PRIVATE "${EXECUTORCH_LIBRARIES}")
    if (TORCHAO_BUILD_CPU_AARCH64)
        target_link_libraries(torchao_ops_embedding_xbit_executorch PRIVATE torchao_kernels_aarch64)
    endif()
endif()

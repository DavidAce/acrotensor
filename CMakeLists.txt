cmake_minimum_required(VERSION 3.12)

message(STATUS
        "------------------------------------------------------------\n"
        "--\n"
        "--                  Configuring Acrotensor\n"
        "--\n"
        "-- ------------------------------------------------------------"
        )

project(acrotensor
        VERSION 0.0.1
        DESCRIPTION "Acrotensor is a C++/CUDA library for computing arbitrary tensor contractions both on CPUs and GPUs"
        HOMEPAGE_URL "https://github.com/LLNL/acrotensor")

option(ACROTENSOR_ENABLE_CUDA   "Enable CUDA support"                          OFF)
option(ACROTENSOR_IS_SUBPROJECT "Enable use of Acrotensor as CMake subproject" OFF)

# Used when acrotensor is included as subproject (e.g., as Git submodule/subtree) in the source
# tree of a project that uses it. Users may set the option ACROTENSOR_IS_SUBPROJECT
# before add_subdirectory(acrotensor)
if (NOT ACROTENSOR_IS_SUBPROJECT)
    if ("^${CMAKE_SOURCE_DIR}$" STREQUAL "^${PROJECT_SOURCE_DIR}$")
        set(ACROTENSOR_IS_SUBPROJECT OFF)
    else ()
        message(STATUS "Detected acrotensor as subproject")
        set(ACROTENSOR_IS_SUBPROJECT ON)
    endif ()
endif ()


# Set default install directory for acrotensor and its dependencies
# Append directory <libname> to CMAKE_INSTALL_PREFIX
# Useful if you want to have separate directories for each libs
# and to easily delete them individually
if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT AND NOT ACROTENSOR_IS_SUBPROJECT)
    message(STATUS "Setting default install location: CMAKE_INSTALL_PREFIX --> ${CMAKE_BINARY_DIR}/install")
    set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/install" CACHE PATH "..." FORCE)
endif ()

message(STATUS "CMAKE_INSTALL_PREFIX: ${CMAKE_INSTALL_PREFIX}")

# list of source files
list(APPEND acrotensor_src exec/CPUInterpretedExecutor.cpp exec/CudaExecutor.cpp exec/KernelExecutor.cpp)
list(APPEND acrotensor_src kernel/DimensionedKernel.cpp kernel/DimensionedMultiKernel.cpp kernel/TensorEngine.cpp kernel/TensorKernel.cpp)
list(APPEND acrotensor_src ops/CudaGPUOps.cpp ops/NativeCPUOps.cpp)
list(APPEND acrotensor_src tensor/IndexMapping.cpp tensor/IndexVector.cpp tensor/SliceTensor.cpp tensor/Tensor.cpp)
list(APPEND acrotensor_src util/CudaUtil.cpp)

# this is the "object library" target: compiles the sources only once
add_library(acrotensor_obj OBJECT ${acrotensor_src})
# shared libraries need PIC
set_target_properties(acrotensor_obj PROPERTIES POSITION_INDEPENDENT_CODE ON)

# Include the directories
#target_include_directories(acrotensor_obj PRIVATE PUBLIC ./ exec kernel ops tensor util)
target_include_directories(acrotensor_obj PRIVATE $<BUILD_INTERFACE: exec kernel ops tensor util>)

# shared and static libraries built from the same object files
add_library(acrotensor_shared SHARED $<TARGET_OBJECTS:acrotensor_obj>)
add_library(acrotensor_static STATIC $<TARGET_OBJECTS:acrotensor_obj>)
# ... and they include the same directories once installed
target_include_directories(acrotensor_shared SYSTEM INTERFACE $<INSTALL_INTERFACE:include>)
target_include_directories(acrotensor_static SYSTEM INTERFACE $<INSTALL_INTERFACE:include>)



# Enable cuda support
if(ACROTENSOR_ENABLE_CUDA)
    enable_language(CUDA)
    target_compile_definitions(acrotensor_obj PUBLIC ACRO_HAVE_CUDA)
    # Set compatibility for sm_60 as in the default makefile
    target_compile_options(acrotensor_obj PRIVATE $<$<COMPILE_LANGUAGE:CUDA>:SHELL:-gencode arch=compute_60,code=sm_60>)
    set_source_files_properties(${acrotensor_src} PROPERTIES LANGUAGE CUDA)
    set_target_properties(acrotensor_obj    PROPERTIES CUDA_ARCHITECTURES 75)
    set_target_properties(acrotensor_shared PROPERTIES CUDA_ARCHITECTURES 75)
    set_target_properties(acrotensor_static PROPERTIES CUDA_ARCHITECTURES 75)
endif()



# Rename the output library names for compatibility
set_target_properties(acrotensor_shared PROPERTIES OUTPUT_NAME "acrotensor")
set_target_properties(acrotensor_static PROPERTIES OUTPUT_NAME "acrotensor")



# Install acrotensor
if(ACROTENSOR_IS_SUBPROJECT)
    add_library(acrotensor::acrotensor_shared ALIAS acrotensor_shared)
    add_library(acrotensor::acrotensor_static ALIAS acrotensor_static)
else()

    # Install library
    # Read about this share path here https://cmake.org/cmake/help/v3.12/command/find_package.html
    list(APPEND acrotensor_hdr exec/CPUInterpretedExecutor.hpp exec/CudaExecutor.hpp exec/Executor.hpp exec/KernelExecutor.hpp )
    list(APPEND acrotensor_hdr kernel/DimensionedKernel.hpp kernel/DimensionedMultiKernel.hpp kernel/TensorKernel.hpp kernel/TensorEngine.hpp )
    list(APPEND acrotensor_hdr ops/CudaGPUOps.hpp ops/NativeCPUOps.hpp ops/Ops.hpp ops/NonContractionOps.hpp)
    list(APPEND acrotensor_hdr tensor/Tensor.hpp tensor/IndexMapping.hpp tensor/IndexVector.hpp tensor/SliceTensor.hpp)
    list(APPEND acrotensor_hdr util/CudaUtil.hpp util/Error.hpp util/StringUtil.hpp util/Util.hpp)

    install(FILES AcroTensor.hpp    DESTINATION include/acrotensor COMPONENT headers)
    install(FILES ${acrotensor_hdr} DESTINATION include/acrotensor COMPONENT headers)
    install(FILES ${acrotensor_hdr} DESTINATION include/acrotensor COMPONENT headers)
    install(FILES ${acrotensor_hdr} DESTINATION include/acrotensor COMPONENT headers)
    install(FILES ${acrotensor_hdr} DESTINATION include/acrotensor COMPONENT headers)
    install(FILES ${acrotensor_hdr} DESTINATION include/acrotensor COMPONENT headers)

    install(TARGETS acrotensor_shared acrotensor_static EXPORT acrotensorTargets)

    #Export the targets to a script
    install(EXPORT
            acrotensorTargets
            NAMESPACE acrotensor::
            DESTINATION share/acrotensor/cmake)


    #Create a ConfigVersion.cmake file
    include(CMakePackageConfigHelpers)

    configure_package_config_file(
            ${CMAKE_CURRENT_SOURCE_DIR}/cmake/acrotensorConfig.cmake.in
            ${CMAKE_CURRENT_BINARY_DIR}/acrotensorConfig.cmake
            INSTALL_DESTINATION share/acrotensor/cmake
    )

    write_basic_package_version_file(
            ${CMAKE_BINARY_DIR}/acrotensorConfigVersion.cmake
            VERSION ${PROJECT_VERSION}
            COMPATIBILITY AnyNewerVersion
    )

    ##Install the config, configversion and custom find modules
    install(FILES
            ${CMAKE_CURRENT_BINARY_DIR}/acrotensorConfig.cmake
            ${CMAKE_CURRENT_BINARY_DIR}/acrotensorConfigVersion.cmake
            DESTINATION share/acrotensor/cmake
            COMPONENT config
            )

    # Uninstall target
    if(NOT TARGET uninstall)
        configure_file(
                ${CMAKE_CURRENT_SOURCE_DIR}/cmake/acrotensorUninstall.cmake.in
                ${CMAKE_CURRENT_BINARY_DIR}/acrotensorUninstall.cmake
                IMMEDIATE @ONLY)

        add_custom_target(uninstall
                COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/acrotensorUninstall.cmake)
    endif()
endif ()



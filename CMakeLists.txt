cmake_minimum_required (VERSION 2.8.6)

# If the user specifies -DCMAKE_BUILD_TYPE on the command line, take their
# definition and dump it in the cache along with proper documentation, otherwise
# set MAKE_BUILD_TYPE to Debug prior to calling PROJECT()
if (DEFINED CMAKE_BUILD_TYPE)
    set (CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE} CACHE STRING "Choose the type of build, options are: None(CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Debug Release RelWithDebInfo MinSizeRel.")
else()
    set (CMAKE_BUILD_TYPE "Debug" CACHE STRING "Choose the type of build, options are: None(CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Debug Release RelWithDebInfo MinSizeRel.")
endif()

if (COMMAND cmake_policy)
    cmake_policy (SET CMP0003 NEW)
    if (CMAKE_VERSION VERSION_GREATER 2.8.12 OR CMAKE_VERSION VERSION_EQUAL 2.8.12)
        # INTERFACE_LINK_LIBRARIES defines the link interface
        cmake_policy (SET CMP0022 NEW)
    endif ()
    if (CMAKE_VERSION VERSION_GREATER 3.0.0 OR CMAKE_VERSION VERSION_EQUAL 3.0.0)
        # Disallow use of the LOCATION target property - therefore we set to OLD as we still need it
        cmake_policy (SET CMP0026 OLD)
        # MACOSX_RPATH is enabled by default
        cmake_policy (SET CMP0042 NEW)
    endif ()
endif ()

###############################################################################
# Project name
###############################################################################

project ("PonyCraft")

# Make macs happy
set (CMAKE_MACOSX_RPATH OFF)

# determine platform - this is used to replace @PLATFORM@ in the various
# export.h.in files.
if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    set (PLATFORM "LINUX")
elseif (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    set (PLATFORM "MACOSX")
elseif (${CMAKE_SYSTEM_NAME} MATCHES "Windows")
    set (PLATFORM "WINDOWS")
endif ()

# Create vcproj.user file for Visual Studio to set debug working directory
# correctly.
# (Thanks David Avedissian)
function(create_vcproj_userfile TARGETNAME)
  if(MSVC)
    configure_file(
      ${CMAKE_SOURCE_DIR}/cmake/templates/VisualStudioUserFile.vcproj.user.in
      ${CMAKE_CURRENT_BINARY_DIR}/${TARGETNAME}.vcproj.user
      @ONLY)
    configure_file(
      ${CMAKE_SOURCE_DIR}/cmake/templates/VisualStudioUserFile.vcxproj.user.in
      ${CMAKE_CURRENT_BINARY_DIR}/${TARGETNAME}.vcxproj.user
      @ONLY)
  endif()
endfunction()

###############################################################################
# set output locations
###############################################################################

set (CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
if (WIN32)
    set (CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
else ()
    set (CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
endif ()

foreach( OUTPUTCONFIG ${CMAKE_CONFIGURATION_TYPES} )
    string( TOUPPER ${OUTPUTCONFIG} OUTPUTCONFIG )
    set( CMAKE_RUNTIME_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${CMAKE_RUNTIME_OUTPUT_DIRECTORY} )
    set( CMAKE_LIBRARY_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${CMAKE_LIBRARY_OUTPUT_DIRECTORY} )
    set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${CMAKE_ARCHIVE_OUTPUT_DIRECTORY} )
endforeach( OUTPUTCONFIG CMAKE_CONFIGURATION_TYPES )

###############################################################################
# compiler definitions and flags
###############################################################################

if(${PLATFORM} MATCHES "LINUX")
    option (ENABLE_PROFILING "Compiles with -pg on linux" OFF)
endif ()

if (${CMAKE_C_COMPILER_ID} STREQUAL "GNU")
    set (DEFINE_CMD -D)
    add_definitions (-std=c++11)
    if (ENABLE_PROFILING)
        add_definitions(-pg)
    endif ()
elseif (${CMAKE_C_COMPILER_ID} STREQUAL "Clang")
    set (DEFINE_CMD -D)
    add_definitions (-std=c++11)
    if (ENABLE_PROFILING)
        add_definitions(-pg)
    endif ()
elseif (${CMAKE_C_COMPILER_ID} STREQUAL "Intel")
elseif (${CMAKE_C_COMPILER_ID} STREQUAL "MSVC")
    set (DEFINE_CMD /D)
    add_definitions (/D _CRT_SECURE_NO_WARNINGS)
    if (CMAKE_BUILD_TYPE MATCHES Debug)
        STRING(REGEX REPLACE "/MDd" "" CMAKE_C_FLAGS_DEBUG ${CMAKE_C_FLAGS_DEBUG})
        add_definitions (/MTd)
    elseif (CMAKE_BUILD_TYPE MATCHES Release)
        STRING(REGEX REPLACE "/MD" "" CMAKE_C_FLAGS_RELEASE ${CMAKE_C_FLAGS_RELEASE})
        add_definitions (/MT)
    endif ()
endif ()

if (CMAKE_BUILD_TYPE MATCHES Debug)
    add_definitions(
        ${DEFINE_CMD}_DEBUG
        ${DEFINE_CMD}DEBUG
    )
endif ()
if (CMAKE_BUILD_TYPE MATCHES Release)
    add_definitions(${DEFINE_CMD}NDEBUG)
endif ()

###############################################################################
# set where the export.h template file is
###############################################################################

set (EXPORT_H_TEMPLATE "${CMAKE_SOURCE_DIR}/common/include/export.hpp.in")

###############################################################################
# Build Urho3D
###############################################################################

set (CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/Urho3D/CMake/Modules/")
set (CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH} "${CMAKE_SOURCE_DIR}/urho3d-dist/")

file (MAKE_DIRECTORY "${CMAKE_SOURCE_DIR}/urho3d-build")

execute_process (
    COMMAND cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=${CMAKE_SOURCE_DIR}/urho3d-dist ../Urho3D
    WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/urho3d-build")

if (${PLATFORM} MATCHES "LINUX" OR ${PLATFORM} MATCHES "MACOSX")
    execute_process (
        COMMAND make install
        WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/urho3d-build")

    # create symlink "lib" so find_package finds urho3D
    include (CheckTypeSize)
    check_type_size (void* SIZEOF_VOID_PTR)
    if (SIZEOF_VOID_PTR EQUAL 8)
        execute_process (COMMAND cmake -E create_symlink "lib64" "lib" WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/urho3d-dist")
    else ()
        execute_process (COMMAND cmake -E create_symlink "lib32" "lib" WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/urho3d-dist")
    endif ()
endif ()


###############################################################################
# Dependency settings
###############################################################################

option (BUILD_TESTS "Whether or not to build unit tests (note: requires C++)" OFF)

message (STATUS "------------------------------------------------------------")
message (STATUS "Global settings")
message (STATUS " + Build type: ${CMAKE_BUILD_TYPE}")
message (STATUS " + Platform is: ${PLATFORM}")
message (STATUS " + Unit Tests: ${BUILD_TESTS}")
message (STATUS "------------------------------------------------------------")

add_subdirectory ("ponycraft")

if (BUILD_TESTS)
    add_subdirectory ("tests")
endif ()

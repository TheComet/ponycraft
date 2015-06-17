###############################################################################
# compiler flags for this project
###############################################################################

if (${CMAKE_C_COMPILER_ID} STREQUAL "GNU")
    add_definitions (-W -Wall -Wextra -pedantic -Wno-unused-parameter)
elseif (${CMAKE_C_COMPILER_ID} STREQUAL "Clang")
    add_definitions (-W -Wall -Wextra -pedantic -Wno-unused-parameter)
elseif (${CMAKE_C_COMPILER_ID} STREQUAL "Intel")
elseif (${CMAKE_C_COMPILER_ID} STREQUAL "MSVC")
endif ()

###############################################################################
# source files and runtime definition
###############################################################################

file (GLOB ponycraft_HEADERS "include/ponycraft/*.hpp")
file (GLOB ponycraft_SOURCES "src/*.cpp")

set (ponycraft_HEADERS ${ponycraft_HEADERS}
    "include/ponycraft/config.hpp.in"
    "${EXPORT_H_TEMPLATE}"
)

include_directories ("include")

add_executable (ponycraft
    ${ponycraft_HEADERS}
    ${ponycraft_SOURCES}
)

if(${PLATFORM} MATCHES "LINUX" OR ${PLATFORM} MATCHES "MACOSX")
    set (MATH_LIB m)
endif ()

target_link_libraries (ponycraft
    ${MATH_LIB}
)

# generates the correct project files for visual studio, setting the correct
# debug working directory and other stuff
create_vcproj_userfile (ponycraft)

###############################################################################
# generate config.h and export.h
###############################################################################

set (PROJECT_NAME "PONYCRAFT")

configure_file ("${EXPORT_H_TEMPLATE}"
                "include/ponycraft/export.hpp")
configure_file ("include/ponycraft/config.hpp.in"
                "include/ponycraft/config.hpp")

include_directories ("${CMAKE_CURRENT_BINARY_DIR}/include")

###############################################################################
# Find Urho3D library
###############################################################################

find_package (Urho3D REQUIRED)
include_directories (${URHO3D_INCLUDE_DIRS})
target_link_libraries (ponycraft ${URHO3D_LIBRARIES})

###############################################################################
# install targets
###############################################################################

install (
    TARGETS
        ponycraft
    DESTINATION
        "bin"
)

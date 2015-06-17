project (ponycraft_tests)

add_subdirectory ("gmock")

include_directories ("gmock/include")
include_directories ("gmock/gtest/include")
include_directories ("include")

file (GLOB tests_SOURCES "src/*.cpp")

# death tests in debug mode
option (ENABLE_DEATH_TESTS "Whether or not to build death tests" ON)
if (ENABLE_DEATH_TESTS)
	if (CMAKE_BUILD_TYPE MATCHES "Debug")
		file (GLOB tests_SOURCES_UTIL ${tests_SOURCES_UTIL} "src/death/util/*.cpp")
		file (GLOB tests_SOURCES_FRAMEWORK ${tests_SOURCES_FRAMEWORK} "src/death/framework/*.cpp")
	endif ()
endif ()

add_definitions (${DEFINE_CMD}TESTING)

message (STATUS "------------------------------------------------------------")
message (STATUS "Settings for tests")
message (STATUS " + Death tests: ${ENABLE_DEATH_TESTS}")
message (STATUS "------------------------------------------------------------")

add_executable (ponycraft_tests
    ${tests_SOURCES}
)

target_link_libraries (ponycraft_tests
    gmock_main
)

# generates the correct project files for visual studio, setting the correct
# debug working directory and other stuff
create_vcproj_userfile (ponycraft_tests)

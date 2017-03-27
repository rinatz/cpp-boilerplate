cmake_minimum_required(VERSION 2.8)

project(example)

#
# Set version: ${MAJOR}.${MINOR}.${PATCH}-${PRE_RELEASE}+${BUILD_TAG}
#
set(PROJECT_VERSION_MAJOR 1)
set(PROJECT_VERSION_MINOR 0)
set(PROJECT_VERSION_PATCH 0)
set(PROJECT_VERSION_PRE_RELEASE "")
set(PROJECT_VERSION_BUILD_TAG "$ENV{BUILD_TAG}")

#
# Set macros
#
add_definitions(
)

#
# Set compile options
#
add_compile_options(
    -std=c++0x
    -Wall
    -Wextra
    -Werror
    -Wno-error=missing-field-initializers
    -pthread
)

#
# Set libraries or linker flags
#
set(
    LIBRARIES
    -pthread
)

#------------------------------------------------------------------------------
# Following settings need not to be modified usually.
#------------------------------------------------------------------------------

configure_file(version.h.in version.h)

#
# libs
#
include_directories(
    ${PROJECT_BINARY_DIR}
    ${PROJECT_SOURCE_DIR}/libs/include
)

link_directories(
    ${PROJECT_SOURCE_DIR}/libs/lib
)

#
# project
#
file(
    GLOB_RECURSE
    SOURCES
    ${PROJECT_SOURCE_DIR}/src/${PROJECT_NAME}/*.cc
)

add_library(${PROJECT_NAME} STATIC ${SOURCES})

target_include_directories(
    ${PROJECT_NAME}
    PRIVATE
    ${PROJECT_SOURCE_DIR}/src/${PROJECT_NAME}/headers
)

#
# test
#
file(
    GLOB_RECURSE
    TEST_SOURCES
    ${PROJECT_SOURCE_DIR}/src/${PROJECT_NAME}Test/*.cc
)

add_executable(${PROJECT_NAME}Test ${TEST_SOURCES})

target_include_directories(
    ${PROJECT_NAME}Test
    PRIVATE
    ${PROJECT_SOURCE_DIR}/src/${PROJECT_NAME}/headers
    ${PROJECT_SOURCE_DIR}/src/${PROJECT_NAME}Test/headers
)

target_link_libraries(
    ${PROJECT_NAME}Test
    ${PROJECT_NAME}
    ${LIBRARIES}
    libgmock.a
)

enable_testing()

add_test(
    NAME ${PROJECT_NAME}Test
    COMMAND ${PROJECT_NAME}Test --gtest_output=xml
)

#
# main
#
file(
    GLOB_RECURSE
    MAIN_SOURCES
    ${PROJECT_SOURCE_DIR}/src/main/*.cc
)

add_executable(main ${MAIN_SOURCES})

target_include_directories(
    main
    PRIVATE
    ${PROJECT_SOURCE_DIR}/src/${PROJECT_NAME}/headers
)

target_link_libraries(
    main
    ${PROJECT_NAME}
    ${LIBRARIES}
)

set_target_properties(
    main
    PROPERTIES
    RUNTIME_OUTPUT_NAME
    ${PROJECT_NAME}
)

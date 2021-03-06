### Project ###

cmake_minimum_required(VERSION 3.17)
project(DotMatrix VERSION 0.0.0.1 LANGUAGES CXX C)
set(PROJECT_DISPLAY_NAME "Dot Matrix")

# Options
option(DOT_MATRIX_WITH_AUDIO "Enable audio" ON)
option(DOT_MATRIX_WITH_BOOTSTRAP "Enable bootstrap" OFF)
option(DOT_MATRIX_WITH_DEBUGGER "Enable debugger" ON)
option(DOT_MATRIX_WITH_UI "Enable UI" ON)

### Targets ###

set(COMMON_PROJECT_NAME "${PROJECT_NAME}Common")
add_library(${COMMON_PROJECT_NAME} INTERFACE)
target_compile_features(${COMMON_PROJECT_NAME} INTERFACE cxx_std_17)
target_compile_definitions(${COMMON_PROJECT_NAME} INTERFACE DM_DEBUG=$<CONFIG:Debug>)
target_compile_definitions(${COMMON_PROJECT_NAME} INTERFACE DM_PROJECT_NAME="${PROJECT_NAME}")
target_compile_definitions(${COMMON_PROJECT_NAME} INTERFACE DM_PROJECT_DISPLAY_NAME="${PROJECT_DISPLAY_NAME}")
target_compile_definitions(${COMMON_PROJECT_NAME} INTERFACE DM_VERSION_STRING="${PROJECT_VERSION}")

set(EXECUTABLE_COMMON_PROJECT_NAME "${PROJECT_NAME}ExecutableCommon")
add_library(${EXECUTABLE_COMMON_PROJECT_NAME} INTERFACE)
target_link_libraries(${EXECUTABLE_COMMON_PROJECT_NAME} INTERFACE ${COMMON_PROJECT_NAME})
target_compile_definitions(${EXECUTABLE_COMMON_PROJECT_NAME} INTERFACE DM_WITH_BOOTSTRAP=$<BOOL:${DOT_MATRIX_WITH_BOOTSTRAP}>)
target_compile_definitions(${EXECUTABLE_COMMON_PROJECT_NAME} INTERFACE DM_WITH_DEBUGGER=$<BOOL:${DOT_MATRIX_WITH_DEBUGGER}>)
target_compile_definitions(${EXECUTABLE_COMMON_PROJECT_NAME} INTERFACE DM_WITH_AUDIO=$<BOOL:${DOT_MATRIX_WITH_AUDIO}>)
target_compile_definitions(${EXECUTABLE_COMMON_PROJECT_NAME} INTERFACE DM_WITH_UI=$<BOOL:${DOT_MATRIX_WITH_UI}>)
find_package(Threads REQUIRED)
target_link_libraries(${EXECUTABLE_COMMON_PROJECT_NAME} INTERFACE Threads::Threads)

add_executable(${PROJECT_NAME} "")
target_link_libraries(${PROJECT_NAME} PRIVATE ${EXECUTABLE_COMMON_PROJECT_NAME})

set(RETRO_PROJECT_NAME "${PROJECT_NAME}Retro")
add_library(${RETRO_PROJECT_NAME} SHARED "")
target_link_libraries(${RETRO_PROJECT_NAME} PRIVATE ${COMMON_PROJECT_NAME})
target_compile_definitions(${RETRO_PROJECT_NAME} PRIVATE DM_WITH_BOOTSTRAP=0)
target_compile_definitions(${RETRO_PROJECT_NAME} PRIVATE DM_WITH_DEBUGGER=0)
target_compile_definitions(${RETRO_PROJECT_NAME} PRIVATE DM_WITH_AUDIO=0)
target_compile_definitions(${RETRO_PROJECT_NAME} PRIVATE DM_WITH_UI=0)

set(TEST_PROJECT_NAME "${PROJECT_NAME}Test")
add_executable(${TEST_PROJECT_NAME} "")
target_link_libraries(${TEST_PROJECT_NAME} PRIVATE ${EXECUTABLE_COMMON_PROJECT_NAME})

include(TestBigEndian)
TEST_BIG_ENDIAN(DOT_MATRIX_IS_BIG_ENDIAN)
target_compile_definitions(${COMMON_PROJECT_NAME} INTERFACE DM_IS_BIG_ENDIAN=$<BOOL:${DOT_MATRIX_IS_BIG_ENDIAN}>)

if(WIN32)
   target_compile_definitions(${COMMON_PROJECT_NAME} INTERFACE NOMINMAX)
endif()

set_property(DIRECTORY ${CMAKE_PROJECT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${PROJECT_NAME})

### Source ###

# Source files
include(Source.cmake)

# Libraries
include(Libraries.cmake)

# Resources
set(RES_DIR "${PROJECT_SOURCE_DIR}/Resources")
set(BIN_RES_DIR "${PROJECT_BINARY_DIR}/Resources")
include(Resources.cmake)

### Install ###

include(Install.cmake)

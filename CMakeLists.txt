cmake_minimum_required(VERSION 3.15)
# set the project name
set(PROJECT_NAME Main)
project(${PROJECT_NAME})

# add the executable
add_executable(${PROJECT_NAME})

set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD 11)

# Set compile flags.
target_compile_options(${PROJECT_NAME} PRIVATE
        # Clang
        $<$<OR:$<CXX_COMPILER_ID:Clang>,$<CXX_COMPILER_ID:AppleClang>>:
        -Weverything -fcolor-diagnostics
        # Disable specific warnings.
        -Wno-c++98-compat -Wno-c++98-compat-pedantic -Wno-padded
        -Wno-deprecated-declarations -Wno-exit-time-destructors
        -Wno-switch-enum -Wno-weak-vtables -Wno-global-constructors>
        # GCC
        $<$<CXX_COMPILER_ID:GNU>:-Wall -Wextra -Wpedantic -fdiagnostics-color=always>
        # Visual Studio
        $<$<CXX_COMPILER_ID:MSVC>:/W4>
        # Enable the clang sanitizer.
        $<$<AND:$<CONFIG:Debug>,$<CXX_COMPILER_ID:Clang>,$<PLATFORM_ID:${SANITIZER_OS}>>:${SANITIZER_FLAGS}>
        )

# Link against the clang sanitizer.
target_link_options(${PROJECT_NAME} PRIVATE
        $<$<AND:$<CONFIG:Debug>,$<CXX_COMPILER_ID:Clang>,$<PLATFORM_ID:${SANITIZER_OS}>>:${SANITIZER_FLAGS}>
        )

if (LINUX)
    target_link_libraries(${PROJECT_NAME} PRIVATE -lm)
endif()

# --- DEPENDENCIES ---

Include(FetchContent)
FetchContent_Declare(
        Catch2
        GIT_REPOSITORY https://github.com/catchorg/Catch2.git
        GIT_TAG        v2.13.1)
FetchContent_MakeAvailable(Catch2)
target_link_libraries(${PROJECT_NAME} Catch2::Catch2)

# add Catch cmake module
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_BINARY_DIR}/_deps/catch2-src/contrib)

include(CTest)
include(Catch)
catch_discover_tests(${PROJECT_NAME})

if(${CMAKE_BUILD_TYPE} STREQUAL Debug)
    #option(CATCH_BUILD_TESTING "" OFF)
    #option(BUILD_TESTING "" OFF)
endif()

add_subdirectory(src)

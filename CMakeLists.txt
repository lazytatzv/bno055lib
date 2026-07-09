cmake_minimum_required(VERSION 3.10)
project(bno055lib VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Source files
set(SOURCES
    src/bno055.cpp
)

# Header files
set(HEADERS
    include/bno055lib/bno055.hpp
)

# Add library
add_library(bno055lib SHARED ${SOURCES} ${HEADERS})

# Include directories
target_include_directories(bno055lib
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
)

# Compiler flags
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_options(bno055lib PRIVATE -Wall -Wextra -Wpedantic)
endif()

# Clang-Tidy static analysis integration
option(ENABLE_CLANG_TIDY "Enable clang-tidy static analysis" ON)
if(ENABLE_CLANG_TIDY)
    find_program(CLANG_TIDY_EXE NAMES "clang-tidy")
    if(CLANG_TIDY_EXE)
        set(CMAKE_CXX_CLANG_TIDY "${CLANG_TIDY_EXE}")
        message(STATUS "clang-tidy enabled: ${CLANG_TIDY_EXE}")
    else()
        message(WARNING "clang-tidy not found, static analysis disabled.")
    endif()
endif()

# Dependencies
find_package(Threads REQUIRED)
target_link_libraries(bno055lib PRIVATE Threads::Threads)

# Example executables
add_executable(calibrate_imu examples/calibrate.cpp)
target_link_libraries(calibrate_imu PRIVATE bno055lib)

add_executable(read_data_noexcept examples/read_data_noexcept.cpp)
target_link_libraries(read_data_noexcept PRIVATE bno055lib)

add_executable(read_all_data examples/read_all_data.cpp)
target_link_libraries(read_all_data PRIVATE bno055lib)

# Installation definitions
include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

# Install targets
install(TARGETS bno055lib
    EXPORT bno055libTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(TARGETS calibrate_imu read_data_noexcept read_all_data
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

# Install headers
install(DIRECTORY include/bno055lib
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Export targets
install(EXPORT bno055libTargets
    FILE bno055libTargets.cmake
    NAMESPACE bno055lib::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/bno055lib
)

# Configure config file
configure_package_config_file(
    cmake/bno055libConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/bno055libConfig.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/bno055lib
)

# Write version file
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/bno055libConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

# Install config files
install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/bno055libConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/bno055libConfigVersion.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/bno055lib
)

# Testing
option(BUILD_TESTING "Build unit tests" ON)
if(BUILD_TESTING)
    include(FetchContent)
    FetchContent_Declare(
      googletest
      GIT_REPOSITORY https://github.com/google/googletest.git
      GIT_TAG        release-1.12.1
    )
    set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
    FetchContent_MakeAvailable(googletest)

    enable_testing()

    add_executable(bno055lib_tests
        tests/test_bno055.cpp
    )
    target_link_libraries(bno055lib_tests
        PRIVATE
            bno055lib
            gtest_main
    )
    include(GoogleTest)
    gtest_discover_tests(bno055lib_tests)
endif()

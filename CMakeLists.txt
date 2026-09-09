cmake_minimum_required(VERSION 3.21)
project(client_arm64 LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ---------------------------------------------------------------------------
# Put YOUR own source files here. This template does not include, embed,
# or reference any third-party/proprietary binaries.
# ---------------------------------------------------------------------------
file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS "src/*.cpp" "src/*.c")

if(NOT SOURCES)
    message(WARNING "No sources found in src/. Add your own .c/.cpp files.")
endif()

add_library(client SHARED ${SOURCES})

target_include_directories(client PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
)

set_target_properties(client PROPERTIES
    OUTPUT_NAME "client_arm64"
    VERSION 1.0.0
)

# iOS-specific settings (applied automatically when using the ios.toolchain.cmake
# toolchain file referenced in the CI workflow)
if(PLATFORM MATCHES "OS64" OR PLATFORM MATCHES "SIMULATOR64")
    set_target_properties(client PROPERTIES
        FRAMEWORK FALSE
        MACOSX_RPATH TRUE
    )
endif()

install(TARGETS client
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
)
install(DIRECTORY include/ DESTINATION include)

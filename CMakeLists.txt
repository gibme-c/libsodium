cmake_minimum_required(VERSION 3.11)

if(POLICY CMP0077)
    cmake_policy(SET CMP0077 NEW)
endif()

project("sodium" LANGUAGES C)

option(SODIUM_ENABLE_BLOCKING_RANDOM "Enable this switch only if /dev/urandom is totally broken on the target platform" OFF)

file(GLOB_RECURSE SODIUM_SRC ${PROJECT_SOURCE_DIR}/src/*.c)

if(MSVC)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /wd4197 /wd4244")
endif()

add_library(${PROJECT_NAME} ${SODIUM_SRC})
set_target_properties(${PROJECT_NAME} PROPERTIES C_STANDARD 99)
target_include_directories(${PROJECT_NAME}
    PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src/libsodium/include
    PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src/libsodium/include/sodium
)
target_compile_definitions(${PROJECT_NAME}
    PUBLIC $<$<NOT:$<BOOL:${BUILD_SHARED_LIBS}>>:SODIUM_STATIC>
    PRIVATE CONFIGURED
    $<$<BOOL:${BUILD_SHARED_LIBS}>:SODIUM_DLL_EXPORT>
    $<$<BOOL:${SODIUM_ENABLE_BLOCKING_RANDOM}>:USE_BLOCKING_RANDOM>
    $<$<C_COMPILER_ID:MSVC>:_CRT_SECURE_NO_WARNINGS>
)

# Variables that need to be exported to version.h.in
set(VERSION 1.0.21)
set(SODIUM_LIBRARY_VERSION_MAJOR 28)
set(SODIUM_LIBRARY_VERSION_MINOR 4)
configure_file(
    src/libsodium/include/sodium/version.h.in
    ${CMAKE_CURRENT_SOURCE_DIR}/src/libsodium/include/sodium/version.h
)

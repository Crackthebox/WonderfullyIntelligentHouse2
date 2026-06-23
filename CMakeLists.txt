cmake_minimum_required(VERSION 3.26)

set(MAIN_PROJECT_NAME "oop")
set(MAIN_EXECUTABLE_NAME "${MAIN_PROJECT_NAME}")

project(${MAIN_PROJECT_NAME})

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

include(cmake/Options.cmake)
include(cmake/CompilerFlags.cmake)
include(cmake/CopyHelper.cmake)

###############################################################################
# external dependencies cu FetchContent (Auto-Download SFML pentru CI/GitHub)
###############################################################################
include(FetchContent)

FetchContent_Declare(
    SFML
    GIT_REPOSITORY https://github.com/SFML/SFML.git
    GIT_TAG        2.6.1
)

# Dezactivăm modulele inutile înainte de a face pachetul disponibil, pentru a accelera compilarea
set(SFML_BUILD_AUDIO FALSE CACHE BOOL "" FORCE)
set(SFML_BUILD_NETWORK FALSE CACHE BOOL "" FORCE)
set(SFML_BUILD_WINDOW TRUE CACHE BOOL "" FORCE)
set(SFML_BUILD_GRAPHICS TRUE CACHE BOOL "" FORCE)

# 🔧 SEPARARE FIX MACOS: Clang vs GCC
if(APPLE)
    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        # Pentru Clang (Apple Clang/LLVM), includerea nativă a headerului este suficientă
        add_compile_options(-include dispatch/dispatch.h)
    else()
        # Pentru GCC pe Mac (g++-13), ocolim lipsa suportului de Apple Blocks (^) prin void*
        add_compile_definitions(_DARWIN_C_SOURCE "dispatch_block_t=void*")
    endif()
endif()

FetchContent_MakeAvailable(SFML)

# Aplicare specifică pe modulul sfml-window ca să fim siguri că setările ajung în subproiect
if(APPLE AND TARGET sfml-window)
    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        target_compile_options(sfml-window PRIVATE -include dispatch/dispatch.h)
    else()
        target_compile_definitions(sfml-window PRIVATE _DARWIN_C_SOURCE "dispatch_block_t=void*")
    endif()
endif()
###############################################################################

add_executable(${MAIN_EXECUTABLE_NAME}
    main.cpp
)

# Sursele tale .cpp
target_sources(${MAIN_EXECUTABLE_NAME}
    PRIVATE
        src/patterns.cpp
        src/devices.cpp
        src/room_house.cpp
)

set_compiler_flags(RUN_SANITIZERS ${USE_ASAN} TARGET_NAMES ${MAIN_EXECUTABLE_NAME})

# Include folderul principal "include" pentru headere
target_include_directories(${MAIN_EXECUTABLE_NAME} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
)
target_include_directories(${MAIN_EXECUTABLE_NAME} SYSTEM PRIVATE
    ext
)

# Legăm modulele SFML la executabilul tău
target_link_libraries(${MAIN_EXECUTABLE_NAME} PRIVATE sfml-graphics sfml-window sfml-system)
###############################################################################

install(TARGETS ${MAIN_EXECUTABLE_NAME} DESTINATION ${DESTINATION_DIR})
if(APPLE)
    install(FILES launcher.command DESTINATION ${DESTINATION_DIR})
endif()

copy_files(FILES tastatura.txt config_casa.txt DIRECTORY assets COPY_TO_DESTINATION TARGET_NAME ${MAIN_EXECUTABLE_NAME})
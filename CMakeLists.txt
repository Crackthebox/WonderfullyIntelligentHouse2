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

# 🔧 REPARAȚIE PENTRU MAC OS: 
# Rezolvă eroarea "dispatch_block_t has not been declared" din SDK-ul macOS 14
if(APPLE)
    add_compile_options(-include dispatch/dispatch.h)
endif()

FetchContent_Declare(
    SFML
    GIT_REPOSITORY https://github.com/SFML/SFML.git
    GIT_TAG        2.6.1
)

FetchContent_GetProperties(SFML)
if(NOT sfml_POPULATED)
    FetchContent_Populate(SFML)
    
    # Dezactivăm modulele inutile pentru a accelera compilarea
    set(SFML_BUILD_AUDIO FALSE CACHE BOOL "" FORCE)
    set(SFML_BUILD_NETWORK FALSE CACHE BOOL "" FORCE)
    set(SFML_BUILD_WINDOW TRUE CACHE BOOL "" FORCE)
    set(SFML_BUILD_GRAPHICS TRUE CACHE BOOL "" FORCE)
    
    add_subdirectory(${sfml_SOURCE_DIR} ${sfml_BINARY_DIR})
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
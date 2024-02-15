cmake_minimum_required(VERSION 3.21)

# make "install" write to the build directory
set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_BINARY_DIR})

# set output dirs to the build directory
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/$<CONFIGURATION>)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/$<CONFIGURATION>)

PROJECT(HelloCube)

string(REPLACE "-Wl,--gc-sections" "" CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS}")
string(REPLACE "-Wl,--no-undefined" "" CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS}")

add_subdirectory("RavEngine" EXCLUDE_FROM_ALL) # configure the engine library

# configure your executable like normal
file(GLOB SOURCES "*.cpp" "*.hpp" "*.h")
if (ANDROID)
   # The SDL java code is hardcoded to load libmain.so on android, so we need to change EXECUTABLE_NAME
   # it must also explicitly be a shared library
   set(EXECUTABLE_NAME main)
   add_library("${EXECUTABLE_NAME}" SHARED)
else()
   set(EXECUTABLE_NAME "${PROJECT_NAME}")
   add_executable("${EXECUTABLE_NAME}")
endif()
target_sources(${EXECUTABLE_NAME} PRIVATE ${SOURCES})

target_link_libraries("${EXECUTABLE_NAME}" PUBLIC "RavEngine" )  # also adds header includes
target_compile_features("${EXECUTABLE_NAME}" PRIVATE cxx_std_20)  # require C++20

#target_link_options("${EXECUTABLE_NAME}" PUBLIC "-Wl,--undefined=SDL_main")

# inform engine about your different assets
file(GLOB objects "objects/*.obj" "objects/*.fbx")
file(GLOB textures "textures/*")
file(GLOB shaders "shaders/*.cmake")
file(GLOB fonts "fonts/*.ttf")
file(GLOB sounds "sounds/*.ogg")
file(GLOB uis "${sample_dir}/ui/*.rml" "${sample_dir}/uis/*.rcss")
pack_resources(TARGET "${EXECUTABLE_NAME}"
   OBJECTS ${objects}
   SHADERS ${shaders}
   TEXTURES ${textures}
   UIS ${uis}
   FONTS ${fonts}
   SOUNDS ${sounds}
)

if (ANDROID)
   set_target_properties(${EXECUTABLE_NAME} PROPERTIES
           DEBUG_POSTFIX ""   # SDL android is hardcoded to load "libmain.so" with no "d" postfix
   )
   #target_link_options(${EXECUTABLE_NAME} PUBLIC "-Wl,--no-gc-sections" "-Wl,--undefined=ignore")
endif()

# fixup macOS / iOS / tvOS bundle
# this is important if you use third party libraries that are linked dynamically. 
if(APPLE)
INSTALL(CODE 
   "include(BundleUtilities)
   fixup_bundle(\"${CMAKE_INSTALL_PREFIX}/$<CONFIGURATION>/${EXECUTABLE_NAME}.app\" \"\" \"\")
   " 
   COMPONENT Runtime
)
endif()


message(
        CMAKE_SHARED_LINKER_FLAGS = ${CMAKE_SHARED_LINKER_FLAGS}
        CMAKE_SHARED_LINKER_FLAGS_RELEASE = ${CMAKE_SHARED_LINKER_FLAGS_RELEASE}
        CMAKE_SHARED_LINKER_FLAGS_DEBUG = ${CMAKE_SHARED_LINKER_FLAGS_DEBUG}
)
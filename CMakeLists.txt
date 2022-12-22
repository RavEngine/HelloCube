cmake_minimum_required(VERSION 3.23)

# make "install" write to the build directory
set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_BINARY_DIR})

# set output dirs to the build directory
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/$<CONFIGURATION>)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/$<CONFIGURATION>)

PROJECT(HelloCube)

add_subdirectory("RavEngine") # configure the engine library

# configure your executable like normal
file(GLOB SOURCES "*.cpp" "*.hpp" "*.h")
add_executable("${PROJECT_NAME}" ${SOURCES})
target_link_libraries("${PROJECT_NAME}" PUBLIC "RavEngine" )  # also adds header includes
target_compile_features("${PROJECT_NAME}" PRIVATE cxx_std_20)  # require C++20

# inform engine about your different assets
file(GLOB objects "objects/*.obj" "objects/*.fbx")
file(GLOB textures "textures/*")
file(GLOB shaders "shaders/*.cmake")
file(GLOB fonts "fonts/*.ttf")
file(GLOB sounds "sounds/*.ogg")
file(GLOB uis "${sample_dir}/ui/*.rml" "${sample_dir}/uis/*.rcss")
pack_resources(TARGET "${PROJECT_NAME}" 
   OBJECTS ${objects}
   SHADERS ${shaders}
   TEXTURES ${textures}
   UIS ${uis}
   FONTS ${fonts}
   SOUNDS ${sounds}
)

# fixup macOS / iOS / tvOS bundle
# this is important if you use third party libraries that are linked dynamically. 
if(APPLE)
INSTALL(CODE 
   "include(BundleUtilities)
   fixup_bundle(\"${CMAKE_INSTALL_PREFIX}/$<CONFIGURATION>/${PROJECT_NAME}.app\" \"\" \"\")
   " 
   COMPONENT Runtime
)
endif()
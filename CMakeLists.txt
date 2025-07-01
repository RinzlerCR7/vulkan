cmake_minimum_required(VERSION 3.22)

# Set the project name
project(vulkancourseapp VERSION 1.0.0 LANGUAGES C CXX)

# Set the C++ standard
set(CMAKE_CXX_STANDARD 17)

find_package(Vulkan REQUIRED)

# Set the GLSLC compiler (you can also set this from the command line)
set(GLSLC "/usr/bin/glslc")
# set(VK_RENDERER "VulkanRenderer")
# set(VK_APP "VulkanCourseApp")

# Find all .vert & .frag files in shaders directory
file(GLOB_RECURSE VERT_SOURCES "${CMAKE_SOURCE_DIR}/shaders/*.vert")
file(GLOB_RECURSE FRAG_SOURCES "${CMAKE_SOURCE_DIR}/shaders/*.frag")

# Function to generate .spv files from shader sources
function(add_shader_targets OUT_VAR)
    set(SPV_FILES "")
    foreach(SHADER_FILE ${ARGN})
        get_filename_component(FILE_NAME ${SHADER_FILE} NAME)
        set(SPV_FILE "${CMAKE_BINARY_DIR}/shaders/${FILE_NAME}spv")
        add_custom_command(
            OUTPUT ${SPV_FILE}
            COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_BINARY_DIR}/shaders"
            COMMAND ${GLSLC} ${SHADER_FILE} -o {SPV_FILE}
            DEPENDS ${SHADER_FILE}
            COMMENT "Compiling shader ${SHADER_FILE} to ${SPV_FILE}"
            VERBATIM
        )
        list(APPEND SPV_FILES ${SPV_FILE})
    endforeach()
    set(${OUT_VAR} "${SPV_FILES}" PARENT_SCOPE)
endfunction()

# Generate .spv targets for vert & frag shaders
add_shader_targets(VERT_OBJ_FILES ${VERT_SOURCES})
add_shader_targets(FRAG_OBJ_FILES ${FRAG_SOURCES})

# File all .cpp & .hpp files for the main target
file(GLOB SRC_FILES "*.cpp")
file(GLOB HDR_FILES "*.hpp")

# Add additional include directories
# include_directories(/usr/include)
# include_directories(${CMAKE_SOURCE_DIR}/externals)
# include_directories(${CMAKE_SOURCE_DIR}/shaders)
# include_directories(/home/rin/Programs/vulkan/1.4.304.1/x86_64/include)
include_directories(${CMAKE_SOURCE_DIR})

# Copy the .spv file to the build directory.
file(COPY ${CMAKE_SOURCE_DIR}/shaders DESTINATION ${CMAKE_BINARY_DIR})

# Add additional library directories
# link_directories(/usr/lib64)
# link_directories(/home/rin/Programs/vulkan/1.4.304.1/x86_64/lib)
# link_directories(${CMAKE_SOURCE_DIR}/lib) --> Just for reference
# link_directories(${CMAKE_SOURCE_DIR})

# Add the libraries
# add_library(VulkanRenderer STATIC "Source Files/VulkanRenderer.cpp")
# add_library(firstapp STATIC "first_app.hpp")

# Add executable
# add_executable(vulkancourseapp "Source Files/main.cpp")
# add_executable(vulkancourseapp main.cpp lve_window.cpp lve_pipeline.cpp first_app.cpp lve_device.cpp lve_swap_chain.cpp)
add_executable(vulkancourseapp ${SRC_FILES} ${HDR_FILES})

# Link sources to the executable
# target_sources(vulkancourseapp PUBLIC first_app.hpp)

# Link libraries to the executable
# target_link_libraries(vulkancourseapp PUBLIC libglfw.so.3)
# target_link_libraries(VulkanRenderer PUBLIC libglfw.so.3)
# target_link_libraries(vulkancourseapp PUBLIC VulkanRenderer)
target_link_libraries(vulkancourseapp PUBLIC libglfw.so.3 Vulkan::Vulkan)
# target_link_libraries(vulkancourseapp PRIVATE first_app.hpp)

# Add subdirectories
# add_subdirectory(src)
# add_subdirectory(app)

# install(TARGETS ${VK_APP} RUNTIME DESTINATION bin)

# Make sure the executable depends on the generated .spv files
add_custom_target(shaders ALL DEPENDS ${VERT_OBJ_FILES} ${FRAG_OBJ_FILES})
add_dependencies(vulkancourseapp shaders)

# Optionally, set CFLAGS & LDFLAGS
# set_target_properties(vulkancourseapp PROPERTIES COMPILE_FLAGS "${CFLAGS}" LINK_FLAGS "${LDFLAGS}")

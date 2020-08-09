cmake_minimum_required(VERSION 3.1)
project(SPH)

set(SPH_SRCS src/ogl_utils.c
             src/dividers_gl.c
             src/particles_gl.c
             src/mover_gl.c
             src/font_gl.c
             src/lodepng.c
             src/exit_menu_gl.c
             src/rectangle_gl.c
             src/renderer.c
             src/image_gl.c
             src/cursor_gl.c
             src/background_gl.c
             src/controls.c
             src/geometry.c
             src/hash.c
             src/communication.c
             src/fluid.c
             src/liquid_gl.c)

file(READ /proc/cpuinfo cpuinfo)
string(FIND "${cpuinfo}" "Raspberry Pi" find_res)
if(NOT ${find_res} EQUAL -1)
	set(BUILD_PLATFORM "Pi")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	set(BUILD_PLATFORM "Mac")
else()
	set(BUILD_PLATFORM "Linux")
endif()

set(OpenGL_GL_PREFERENCE GLVND)

find_package(MPI REQUIRED)
find_package(Freetype REQUIRED)
find_package(GLEW REQUIRED)
find_package(OpenGL REQUIRED)

add_executable(sph ${SPH_SRCS})

target_compile_options    (sph PRIVATE ${MPI_COMPILE_FLAGS})
target_include_directories(sph PRIVATE ${MPI_INCLUDE_PATH} ${FREETYPE_INCLUDE_DIRS})
target_link_options       (sph PRIVATE ${MPI_LINK_FLAGS})
target_link_libraries     (sph PRIVATE ${MPI_LIBRARIES} ${FREETYPE_LIBRARIES} GLEW::GLEW m)

if(${BUILD_PLATFORM} STREQUAL "Pi")
	target_compile_options    (sph PRIVATE -mfloat-abi=hard -mfpu=vfp)
	target_compile_definitions(sph PRIVATE RASPI)
	target_include_directories(sph PRIVATE /opt/vc/include /opt/vc/include/interface/vcos/pthreads /opt/vc/include/interface/vmcs_host/linux ./blink1)
	target_link_libraries     (sph PRIVATE -L/opt/vc/lib/ -lGLESv2 -lEGL -lopenmaxil -lbcm_host -lvcos -lvchiq_arm -lpthread -lrt)
	target_sources            (sph PRIVATE src/egl_utils.c)
elseif(${BUILD_PLATFORM} STREQUAL "Mac")
	find_package(glfw3 3.3 REQUIRED)
	target_compile_definitions(sph PRIVATE GLFW EXIT_SIMPLE)
	target_link_libraries     (sph PRIVATE glfw OpenGL::GL -framework Cocoa -framework IOkit -framework CoreVideo)
	target_sources            (sph PRIVATE src/glfw_utils.c)
elseif(${BUILD_PLATFORM} STREQUAL "Linux")
	find_package(glfw3 3.3 REQUIRED)
	target_compile_definitions(sph PRIVATE GLFW)
	target_link_libraries     (sph PRIVATE glfw OpenGL::GL)
	target_sources            (sph PRIVATE src/glfw_utils.c)
endif()

# TODO Enable previous makefile options for Raspberry Pi
# light: -DLIGHT
# blink1: make in blink1 dir and link library, -DBLINK1
# leap: Implies blink1? -DLEAP_MOTION_ENABLED

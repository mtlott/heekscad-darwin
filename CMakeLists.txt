project(heekscad-darwin)

cmake_minimum_required(VERSION 2.8.12)

add_definitions ( -Wall )

set(CMAKE_CXX_STANDARD 11)


#set additionnal CMake module path
set( CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")

find_package( wxWidgets REQUIRED COMPONENTS base core gl )

set( CASCADE_LIBS TKernel )
find_package(OpenCASCADE REQUIRED COMPONENTS ${CASCADE_LIBS})

add_definitions ( -Wall )

#find OCE or OpenCASCADE
set( CASCADE_LIBS TKernel )
if( UNIX )
  set( OCE_DIR "/usr/local/share/cmake/" )
else()
  set( OCE_DIR "c:/OCE-0.3.0/share/cmake" )
endif()
#find_package ( OCE )
if( ${OCE_FOUND} )
  add_definitions ( -DHAVE_CONFIG_H )
  set( OpenCASCADE_LIBRARIES ${CASCADE_LIBS} )
  set( OpenCASCADE_INCLUDE_DIR ${OCE_INCLUDE_DIRS} )
else() #look for OpenCASCADE
  find_package ( OpenCASCADE REQUIRED COMPONENTS ${CASCADE_LIBS} )
endif()


#find libarea
#find_package( libarea REQUIRED )
#find_package( HeeksCAD REQUIRED )
include_directories(
    ${CMAKE_SOURCE_DIR}/submodules/libarea
    ${CMAKE_SOURCE_DIR}/submodules/heekscad/src
    ${CMAKE_SOURCE_DIR}/submodules/heekscad/tinyxml
)
link_directories(
    ${CMAKE_SOURCE_DIR}/submodules/libarea
    ${CMAKE_SOURCE_DIR}/submodules/heekscad/src
    ${CMAKE_SOURCE_DIR}/submodules/heekscad/tinyxml
)

if( APPLE )
  # Apple includes two different openGL implementations, for Carbon and X11.  We want X11
  set( OSX_INCLUDE_DIRS /usr/X11R6/include )
  #set( OSX_LIBS -L/usr/X11R6/lib -lGL -lX11 -lGLU -lglut )
endif( APPLE )

include(${wxWidgets_USE_FILE})

include_directories ( SYSTEM
  ${OpenCASCADE_INCLUDE_DIR}
)

include_directories (
  ${CMAKE_SOURCE_DIR}
  ${wxWidgets_INCLUDE_DIRS}
  ${OSX_INCLUDE_DIRS}
)

# CMake summary
message (STATUS "CMAKE_BUILD_TYPE: ${CMAKE_BUILD_TYPE}")
message (STATUS "CMAKE_INSTALL_PREFIX: ${CMAKE_INSTALL_PREFIX}")

set(CPACK_PACKAGE_VERSION_MAJOR 1)
set(CPACK_PACKAGE_VERSION_MINOR 0)
set(CPACK_PACKAGE_VERSION_PATCH 0)

if(DEFINED LIB_INSTALL_DIR)
    set(LIB_INSTALL_DIR ${LIB_INSTALL_DIR})
else(DEFINED LIB_INSTALL_DIR)
    set(LIB_INSTALL_DIR ${CMAKE_INSTALL_PREFIX}/lib)
endif(DEFINED LIB_INSTALL_DIR)
message(STATUS "LIB_INSTALL_DIR: " ${LIB_INSTALL_DIR})

#---------- put subdirs down here so that the package version vars above are visible to them -----------

add_subdirectory( submodules/libarea )
add_subdirectory( submodules/heekscad/interface )
add_subdirectory( submodules/heekscad/tinyxml )
add_subdirectory( submodules/heekscad/translations )
add_subdirectory( submodules/heekscad/src )  #needs libraries from other subdirs, so it goes last
add_subdirectory( submodules/heekscnc/src )

add_custom_target(heekscad-darwin
    DEPENDS libarea 
            interface
            tinyxml
            translations
            src
            heekscnc
)

if(POLICY CMP0048)
    cmake_policy(SET CMP0048 NEW)
endif()
CMAKE_MINIMUM_REQUIRED(VERSION 3.1)
PROJECT(libiconv VERSION 1.15.0)

if (NOT MSVC) # Everything only for msvc
    RETURN()
endif()
    
#
# Use Conan.io
#
if(USE_CONAN_IO)
    include(conanbuildinfo.cmake)
    conan_basic_setup()
    conan_define_targets()
endif()

if(NOT LIBICONV_PROJECT_DIR)
  set(LIBICONV_PROJECT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/libiconv-1.15)
endif()

SET(LIBICONV_VERSION "1.15.0")
SET(LIBICONV_SOVERSION "1")

set(ICONV_CONST)
include(cmake/config.cmake)
message("==========> ICONV_CONST:${ICONV_CONST}")
if( BUILD_SHARED_LIBS )
   set(DLL_VARIABLE "__declspec (dllexport)")
endif()

CONFIGURE_FILE( ${CMAKE_CURRENT_SOURCE_DIR}/cmake/config.h.cmakein ${CMAKE_CURRENT_BINARY_DIR}/config.h )
CONFIGURE_FILE( ${LIBICONV_PROJECT_DIR}/include/iconv.h.build.in ${CMAKE_CURRENT_BINARY_DIR}/include/iconv.h @ONLY)
CONFIGURE_FILE( ${LIBICONV_PROJECT_DIR}/libcharset/include/libcharset.h.in ${CMAKE_CURRENT_BINARY_DIR}/libcharset/include/libcharset.h)
CONFIGURE_FILE( ${LIBICONV_PROJECT_DIR}/libcharset/include/localcharset.h.in ${CMAKE_CURRENT_BINARY_DIR}/libcharset/include/localcharset.h)

if( BUILD_SHARED_LIBS )
   set(DLL_VARIABLE "__declspec (dllimport)")
endif()
message("@@@@@@@@@@@@ ICONV_CONST:${ICONV_CONST}")
CONFIGURE_FILE( ${LIBICONV_PROJECT_DIR}/include/iconv.h.in package/iconv.h @ONLY)

# Definitions
if (BUILD_SHARED_LIBS)
    add_definitions(-DLIBICONV_SHARED_BUILD)
endif()
add_definitions(-DBUILDING_LIBCHARSET
                -DICONV_CONST)

IF (WIN32)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
    add_definitions(-D_WIN32_NATIVE)
ENDIF (WIN32)

IF (MSVC)
    add_definitions(/wd4018)
    add_definitions(/wd4273)
    add_definitions(/wd4090)
    add_definitions(/wd4013)
    add_definitions(/wd4244)
    add_definitions(/wd4311)
    add_definitions(/wd4267)
ENDIF (MSVC)

add_definitions(-DICONV_NO_LIBDIR)
add_definitions(-DLIBDIR)
 
# Includes
include_directories(${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR}/include ${CMAKE_CURRENT_SOURCE_DIR}/include ${LIBICONV_PROJECT_DIR}/srclib ${LIBICONV_PROJECT_DIR}/lib)
include_directories(${CMAKE_CURRENT_BINARY_DIR}/libcharset/include ${LIBICONV_PROJECT_DIR}/libcharset/include ${LIBICONV_PROJECT_DIR}/libcharset/lib)  

IF (WIN32)
    include_directories(${LIBICONV_PROJECT_DIR}/msvc)
ENDIF (WIN32)

include_directories(${CMAKE_CURRENT_BINARY_DIR}/include)

# Build
add_library(iconv ${LIBICONV_PROJECT_DIR}/lib/iconv.c 
                  ${LIBICONV_PROJECT_DIR}/libcharset/lib/localcharset.c
                  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/libiconv.def)
set_target_properties(iconv PROPERTIES VERSION ${LIBICONV_VERSION} SOVERSION ${LIBICONV_SOVERSION})
#set_target_properties(iconv PROPERTIES INTERFACE_INCLUDE_DIRECTORIES "${CMAKE_CURRENT_BINARY_DIR}/include")

target_include_directories(iconv
    PUBLIC
        $<INSTALL_INTERFACE:include>
    )
    
if (NOT BUILD_SHARED_LIBS)
    set_target_properties(iconv PROPERTIES INTERFACE_COMPILE_DEFINITIONS "LIBICONV_STATIC")
endif()

# Install
install(FILES 
	${CMAKE_CURRENT_BINARY_DIR}/package/iconv.h
	DESTINATION include
	)

install(TARGETS iconv
    EXPORT Targets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    INCLUDES DESTINATION include
    )


#
# Export
#
include(CMakePackageConfigHelpers)

write_basic_package_version_file("${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
    )

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/LibraryConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    @ONLY
    )

install(
    FILES
        "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    DESTINATION
        . # Root Directory
    )

install(
    FILES
        "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    DESTINATION
        share/cmake/${PROJECT_NAME}/
    )

install(EXPORT Targets
    FILE ${PROJECT_NAME}Targets.cmake
    DESTINATION share/cmake/${PROJECT_NAME}
    )

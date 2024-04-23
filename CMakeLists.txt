project(mbe)
cmake_minimum_required(VERSION 2.6)

option(BUILD_SHARED_LIBS "Build shared libraries" ON)
option(BUILD_STATIC_LIBS "Build static libraries" ON)

if(MSVC)
    # needed for M_PI macro
    add_definitions(-D_USE_MATH_DEFINES)
endif()

FILE(GLOB SRCS *.c)

include_directories("${PROJECT_SOURCE_DIR}")

set(TARGETS "")
if(BUILD_STATIC_LIBS)
    add_library(mbe-static STATIC ${SRCS})
    if(NOT WIN32)
        target_link_libraries(mbe-static m)
    endif()
    list(APPEND TARGETS "mbe-static")
endif()
if(BUILD_SHARED_LIBS)
    add_library(mbe-shared SHARED ${SRCS})
    if(NOT WIN32)
        target_link_libraries(mbe-shared m)
    endif()
    list(APPEND TARGETS "mbe-shared")
endif()

include(GNUInstallDirs)

set_target_properties(${TARGETS}
                      PROPERTIES
                      OUTPUT_NAME mbe
                      PREFIX lib
                      VERSION 1.3
                      SOVERSION 1
                      INSTALL_NAME_DIR ${CMAKE_INSTALL_FULL_LIBDIR}
                      PUBLIC_HEADER "mbelib.h")

configure_file("${CMAKE_CURRENT_SOURCE_DIR}/libmbe.pc.in"
               "${CMAKE_CURRENT_BINARY_DIR}/libmbe.pc"
               @ONLY)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/libmbe.pc"
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")

install(TARGETS ${TARGETS}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# uninstall target
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)

add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)

option(DISABLE_TEST "Disable building of test framework." OFF)

if (NOT DISABLE_TEST)
    enable_testing()
    add_subdirectory(test)
endif()

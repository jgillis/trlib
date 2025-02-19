include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

find_package(BLAS REQUIRED)
find_package(LAPACK REQUIRED)

configure_file("${CMAKE_CURRENT_SOURCE_DIR}/trlib_private.h.in" trlib_private.h)

configure_file("${CMAKE_CURRENT_SOURCE_DIR}/trlib.pc.in"
  "${PROJECT_BINARY_DIR}/trlib.pc"
  @ONLY)

add_library(trlib SHARED
  ${PROJECT_SOURCE_DIR}/src/trlib_krylov.c
  ${PROJECT_SOURCE_DIR}/src/trlib_tri_factor.c
  ${PROJECT_SOURCE_DIR}/src/trlib_leftmost.c
  ${PROJECT_SOURCE_DIR}/src/trlib_eigen_inverse.c
  ${PROJECT_SOURCE_DIR}/src/trlib_quadratic_zero.c)

set_target_properties(trlib
  PROPERTIES
  VERSION ${PROJECT_VERSION})

# trlib_private.h
target_include_directories(trlib
  PRIVATE
  "${CMAKE_CURRENT_BINARY_DIR}")

target_include_directories(trlib
  PUBLIC
  $<BUILD_INTERFACE:${PROJECT_BINARY_DIR}/include>
  $<INSTALL_INTERFACE:include>)

target_link_libraries(trlib ${LAPACK_LIBRARIES} ${BLAS_LIBRARIES} m)
set_property(TARGET trlib PROPERTY C_STANDARD 11)

add_library("trlib::trlib" ALIAS trlib)

if(APPLE)
  set_property(TARGET trlib PROPERTY INSTALL_NAME_DIR "${CMAKE_INSTALL_PREFIX}/lib")
endif(APPLE)

install(TARGETS trlib
  EXPORT trlib-config
  DESTINATION lib)

if(WIN32)
    set(CMAKE_INSTALL_CONFIG "${CMAKE_INSTALL_PREFIX}/cmake")
else()
    set(CMAKE_INSTALL_CONFIG "${CMAKE_INSTALL_LIBDIR}/cmake/trlib")
endif()

install(EXPORT trlib-config
  NAMESPACE "trlib::"
  DESTINATION "${CMAKE_INSTALL_CONFIG}")

install(FILES "${PROJECT_BINARY_DIR}/trlib.pc"
  DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/pkgconfig")

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/trlib_private.h"
  DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/trlib")

# upon request collect all files needed for compilation in folder sources
install ( FILES ${PROJECT_SOURCE_DIR}/src/trlib_krylov.c
                ${PROJECT_SOURCE_DIR}/src/trlib_tri_factor.c
                ${PROJECT_SOURCE_DIR}/src/trlib_leftmost.c
                ${PROJECT_SOURCE_DIR}/src/trlib_eigen_inverse.c
                ${PROJECT_SOURCE_DIR}/src/trlib_quadratic_zero.c
                ${PROJECT_BINARY_DIR}/src/trlib_private.h
                DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/trlib/src" )

write_basic_package_version_file(
  "${PROJECT_BINARY_DIR}/trlib-config-version.cmake"
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY ExactVersion
)

install(
  FILES "${PROJECT_BINARY_DIR}/trlib-config-version.cmake"
  DESTINATION "${CMAKE_INSTALL_CONFIG}"
)

#find_package(HDF5)
#find_package(SuiteSparse OPTIONAL_COMPONENTS CHOLMOD)
#if(HDF5_FOUND AND SuiteSparse_FOUND)
#    configure_file("${CMAKE_CURRENT_SOURCE_DIR}/trlib_driver.c.in" "${CMAKE_CURRENT_BINARY_DIR}/trlib_driver.c")
#    if(APPLE)
#       set (ARGP_STATIC_LIBRARY "/opt/local/lib/libargp.a")
#    else(APPLE)
#       set (ARGP_STATIC_LIBRARY "")
#    endif(APPLE)
#    add_executable(driver ${CMAKE_CURRENT_BINARY_DIR}/trlib_driver.c)
#    set_property(TARGET driver PROPERTY C_STANDARD 11)
#    target_include_directories(driver PRIVATE ${HDF5_INCLUDE_DIRS} ${SuiteSparse_INCLUDE_DIRS})
#    #message(${SuiteSparse_LIBRARIES})
#    target_link_libraries(driver ${ARGP_STATIC_LIBRARY} trlib ${HDF5_C_LIBRARIES} ${SuiteSparse_LIBRARIES})
#endif(HDF5_FOUND AND SuiteSparse_FOUND)

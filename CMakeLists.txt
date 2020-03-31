cmake_minimum_required(VERSION 3.4)
project(libssh2lv-nilrt-ipk VERSION 0.3 LANGUAGES C)
set(PROJECT_URL "https://github.com/fieldrndservices/libssh2lv-nilrt-ipk")
set(PROJECT_DESCRIPTION "A CMake Super Build to create libssh2lv IPK files for the NI Linux RT operating system")

# CMake extensions
include(ExternalProject)

# libssh2 configuration and build
set(LIBSSH2LV "libssh2lv")
set(LIBSSH2LV_VERSION "0.2.1")
ExternalProject_Add(${LIBSSH2LV}
  PREFIX ${LIBSSH2LV}
  URL file:///${CMAKE_CURRENT_SOURCE_DIR}/${LIBSSH2LV}-${LIBSSH2LV_VERSION}.tar.gz
  CMAKE_ARGS -DBUILD_DEPS=OFF
  BUILD_IN_SOURCE NO
  TEST_COMMAND ""
  INSTALL_COMMAND ""
  )

ExternalProject_Get_Property(${LIBSSH2LV} BINARY_DIR)
set(LIBSSH2LV_BINARY_DIR ${BINARY_DIR})
ExternalProject_Get_Property(${LIBSSH2LV} SOURCE_DIR)
set(LIBSSH2LV_SOURCE_DIR ${SOURCE_DIR})
set(LIBSSH2LV_INCLUDE_DIR ${LIBSSH2LV_SOURCE_DIR}/src)
set(LIBSSH2LV_ARCHIVE_DIR ${LIBSSH2LV_BINARY_DIR}/lib)
set(LIBSSH2LV_LIBRARY_DIR ${LIBSSH2LV_BINARY_DIR}/bin)

# Package-wide variables
if(CMAKE_SYSTEM_PROCESSOR MATCHES "^arm")
  set(PACKAGE_ARCH "cortexa9-vfpv3")
else()
  set(PACKAGE_ARCH "core2-64")
endif()
set(PACKAGE_FILE_NAME "${LIBSSH2LV}_${LIBSSH2LV_VERSION}-r${PROJECT_VERSION}_${PACKAGE_ARCH}")

# CPack configuration
set(CPACK_CMAKE_GENERATOR "Unix Makefiles")
set(CPACK_GENERATOR "DEB")
set(CPACK_PACKAGE_VENDOR "Field R&D Services, LLC")
set(CPACK_PACKAGE_FILE_NAME ${PACKAGE_FILE_NAME})
set(CPACK_DEBIAN_PACKAGE_NAME ${LIBSSH2LV})
set(CPACK_DEBIAN_PACKAGE_VERSION "${LIBSSH2LV_VERSION}-r${PROJECT_VERSION}")
set(CPACK_DEBIAN_PACKAGE_RELEASE "r${PROJECT_VERSION}")
set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE ${PACKAGE_ARCH})
set(CPACK_DEBIAN_PACKAGE_DESCRIPTION "LabVIEW-friendly C library for the libssh2")
set(CPACK_DEBIAN_PACKAGE_SECTION "libs")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Christopher R. Field <chris@fieldrndservices.com>")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libssh2 (>= 1.8.2")

# CPack uses the `install` command to determine the contents of the package.
install(FILES
  "${LIBSSH2LV_INCLUDE_DIR}/lv-libssh2.h"
  DESTINATION
  "/usr/include"
  RENAME
  "${LIBSSH2LV}.h"
)

install(FILES
  "${LIBSSH2LV_LIBRARY_DIR}/${LIBSSH2LV}.so.1.0.0"
  PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
  DESTINATION
  "/usr/lib"
)

install(FILES
  "${LIBSSH2LV_LIBRARY_DIR}/${LIBSSH2LV}.so.1"
  "${LIBSSH2LV_LIBRARY_DIR}/${LIBSSH2LV}.so"
  DESTINATION
  "/usr/lib"
)

include(CPack)

# The `include(CPack)` command will create the `package` and `package_source`
# build targets, which will build a deb file if invoked as `cmake --build .
# --target package`. For CMake 3.9 or earlier, it is not possible to change the
# file extension from deb to ipk with the Cpack DEB generator. Newer versions of
# CMake/CPack have the `CPACK_DEBIAN_FILE_NAME` variable that can be set with
# either the `.deb` or `.ipk` file extension.
#
# Because of a bug, or missing feature, in CMake and CPack, it is not possible
# for a custom target or command to depend on the `package` or `package_source`
# build targets created by the `include(CPack)` command. Instead, a custom build
# target is added that first builds the `package` target then renames the deb.
# This is essentially a hack to workaround two problems: older versions of
# CMake/CPack and inability to depend on built-in targets.
add_custom_target(ipk
  COMMAND "${CMAKE_COMMAND}" --build . --target package
  COMMAND mv ${PACKAGE_FILE_NAME}.deb ${PACKAGE_FILE_NAME}.ipk
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
  COMMENT "Building IPK file"
)

cmake_minimum_required(VERSION 3.0.2 FATAL_ERROR)

project(libsysstat)

set(MAJOR_VERSION 0)
set(MINOR_VERSION 3)
set(PATCH_VERSION 2)
set(SYSSTAT_VERSION "${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_VERSION}")

set(LXQTBT_MINIMUM_VERSION "0.1.0")

# additional cmake files
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

find_package(Qt5Core REQUIRED QUIET)
find_package(lxqt-build-tools ${LXQTBT_MINIMUM_VERSION} REQUIRED)

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)             # Standard directories for installation
include(LXQtCreatePkgConfigFile)
include(LXQtCreatePortableHeaders)
include(LXQtCompilerSettings NO_POLICY_SCOPE)

set(CMAKE_AUTOMOC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(SYSSTAT_LIBRARY_NAME "sysstat-qt5")
set(HEADER_NAMESPACE "SysStat")

set(SYSSTAT_PUBLIC_HDRS
    version.h
    sysstat_global.h
    basestat.h
    cpustat.h
    memstat.h
    netstat.h
)

set(SYSSTAT_PUBLIC_CLASSES
    Version
    Global
    BaseStat
    CpuStat
    MemStat
    NetStat
)

set(SYSSTAT_PRIVATE_HDRS
    version_p.h
    basestat_p.h
    cpustat_p.h
    memstat_p.h
    netstat_p.h
)

set(SYSSTAT_SRCS
    version.cpp
    basestat.cpp
    cpustat.cpp
    memstat.cpp
    netstat.cpp
)

set(SYSSTAT_QM_FILES "")

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()


set(APP_SHARE_DIR "${CMAKE_INSTALL_DATAROOTDIR}/${SYSSTAT_LIBRARY_NAME}")

add_library(${SYSSTAT_LIBRARY_NAME}
    SHARED
        ${SYSSTAT_PUBLIC_HDRS}
        ${SYSSTAT_PRIVATE_HDRS}
        ${SYSSTAT_SRCS}
        ${SYSSTAT_QM_FILES}
)

target_link_libraries(${SYSSTAT_LIBRARY_NAME} Qt5::Core)

set_target_properties(${SYSSTAT_LIBRARY_NAME} PROPERTIES
    VERSION ${SYSSTAT_VERSION}
    SOVERSION ${MAJOR_VERSION}
)

target_compile_definitions(${SYSSTAT_LIBRARY_NAME}
    PRIVATE "MAJOR_VERSION=${MAJOR_VERSION}"
    PRIVATE "MINOR_VERSION=${MINOR_VERSION}"
    PRIVATE "PATCH_VERSION=${PATCH_VERSION}"
    PRIVATE "SYSSTAT_LIBRARY"
)

target_include_directories(${SYSSTAT_LIBRARY_NAME}
    INTERFACE "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${SYSSTAT_LIBRARY_NAME}>"
)

target_include_directories(${SYSSTAT_LIBRARY_NAME}
    INTERFACE "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/include/${SYSSTAT_LIBRARY_NAME}>"
)

lxqt_create_portable_headers(SYSSTAT_NAMESPACE_HEADERS
    PATH_PREFIX ".."
    HEADER_NAMES ${SYSSTAT_PUBLIC_CLASSES}
)

foreach(h ${SYSSTAT_NAMESPACE_HEADERS})
    get_filename_component(bh ${h} NAME)
    configure_file("${h}" "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/include/${SYSSTAT_LIBRARY_NAME}/${HEADER_NAMESPACE}/${bh}" COPYONLY)
endforeach()

foreach(h ${SYSSTAT_PUBLIC_HDRS})
    get_filename_component(bh ${h} NAME)
    configure_file("${h}" "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/include/${SYSSTAT_LIBRARY_NAME}/${bh}" COPYONLY)
endforeach()

install(TARGETS
    ${SYSSTAT_LIBRARY_NAME}
    DESTINATION "${CMAKE_INSTALL_LIBDIR}" EXPORT ${SYSSTAT_LIBRARY_NAME}-targets
    COMPONENT Runtime
)

install(FILES
    ${SYSSTAT_PUBLIC_HDRS}
    DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/${SYSSTAT_LIBRARY_NAME}"
    COMPONENT Devel
)

install(FILES
    ${SYSSTAT_NAMESPACE_HEADERS}
    DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/${SYSSTAT_LIBRARY_NAME}/${HEADER_NAMESPACE}"
    COMPONENT Devel
)

#install(FILES ${SYSSTAT_QM_FILES} DESTINATION "${APP_SHARE_DIR}")

export(TARGETS
    ${SYSSTAT_LIBRARY_NAME}
    FILE "${CMAKE_BINARY_DIR}/${SYSSTAT_LIBRARY_NAME}-targets.cmake"
)

lxqt_create_pkgconfig_file(
    PACKAGE_NAME ${SYSSTAT_LIBRARY_NAME}
    DESCRIPTIVE_NAME ${SYSSTAT_LIBRARY_NAME}
    DESCRIPTION "SysStat, a Qt-based interface to system statistics"
    INCLUDEDIRS ${SYSSTAT_LIBRARY_NAME}
    LIBS ${SYSSTAT_LIBRARY_NAME}
    REQUIRES "Qt5Core"
    VERSION ${SYSSTAT_VERSION}
    INSTALL
)


# cmake config file
set(SYSSTAT_INCLUDE_DIR "${CMAKE_INSTALL_PREFIX}/include/${SYSSTAT_LIBRARY_NAME}")
configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/sysstat-config.cmake.in"
    "${CMAKE_BINARY_DIR}/${SYSSTAT_LIBRARY_NAME}-config.cmake"
    INSTALL_DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/cmake/${SYSSTAT_LIBRARY_NAME}"
)

# cmake version config file
write_basic_package_version_file(
    "${CMAKE_BINARY_DIR}/${SYSSTAT_LIBRARY_NAME}-config-version.cmake"
    VERSION ${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_VERSION}
    COMPATIBILITY AnyNewerVersion
)

install(FILES
    "${CMAKE_BINARY_DIR}/${SYSSTAT_LIBRARY_NAME}-config.cmake"
    "${CMAKE_BINARY_DIR}/${SYSSTAT_LIBRARY_NAME}-config-version.cmake"
    DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/cmake/${SYSSTAT_LIBRARY_NAME}"
    COMPONENT Devel
)

install(EXPORT
    ${SYSSTAT_LIBRARY_NAME}-targets
    DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/cmake/${SYSSTAT_LIBRARY_NAME}"
    COMPONENT Devel
)

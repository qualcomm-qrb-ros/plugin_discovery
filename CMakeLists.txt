# Copyright (c) Qualcomm Technologies, Inc. and/or its subsidiaries.
# SPDX-License-Identifier: BSD-3-Clause-Clear
cmake_minimum_required(VERSION 3.16)
project(plugin-discovery-platform-config VERSION 0.1.0 LANGUAGES NONE)

# Every <domain>/<machine>-<os>-<domain>.config.yaml is a candidate that platform-detect may link
# into /etc/plugin_discovery/platform.d/. One package carries all domains; each domain's directory
# belongs to that domain's team (see .github/CODEOWNERS).
#
#   cmake -S . -B build && cd build && cpack -G DEB
file(GLOB domain_dirs LIST_DIRECTORIES true RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} CONFIGURE_DEPENDS */)
set(candidates "")
foreach(dir IN LISTS domain_dirs)
  if(NOT IS_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/${dir} OR dir MATCHES "^(\\.|packaging|yocto|build)")
    continue()
  endif()
  file(GLOB files RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}/${dir} ${CMAKE_CURRENT_SOURCE_DIR}/${dir}/*)
  foreach(f IN LISTS files)
    # The file name says machine, OS and domain; the domain must be the directory it lives in.
    if(NOT f MATCHES "^([A-Za-z0-9]+)-(Ubuntu|QCLinux)-([a-z0-9_]+)\\.config\\.yaml$")
      message(FATAL_ERROR "${dir}/${f}: name must be <machine>-<Ubuntu|QCLinux>-<domain>.config.yaml")
    endif()
    if(NOT CMAKE_MATCH_3 STREQUAL dir)
      message(FATAL_ERROR "${dir}/${f}: the domain in the name (${CMAKE_MATCH_3}) is not its directory (${dir})")
    endif()
    file(READ ${CMAKE_CURRENT_SOURCE_DIR}/${dir}/${f} content)
    if(NOT content MATCHES "\n  ${dir}:")
      message(FATAL_ERROR "${dir}/${f}: no 'domains: ${dir}:' block inside")
    endif()
    list(APPEND candidates ${CMAKE_CURRENT_SOURCE_DIR}/${dir}/${f})
  endforeach()
endforeach()
if(NOT candidates)
  message(FATAL_ERROR "no candidate found")
endif()
install(FILES ${candidates} DESTINATION share/plugin_discovery/platforms.d)

set(CPACK_GENERATOR DEB)
set(CPACK_PACKAGE_NAME plugin-discovery-platform-config)
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_PACKAGE_CONTACT "Chalia <chalia@qti.qualcomm.com>")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "plugin_discovery platform configuration of all machines and domains")
set(CPACK_PACKAGING_INSTALL_PREFIX /usr)
set(CPACK_DEBIAN_FILE_NAME DEB-DEFAULT)
set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE all)
set(CPACK_DEBIAN_PACKAGE_SECTION libs)
set(CPACK_DEBIAN_PACKAGE_DEPENDS "plugin-discovery (>= 0.3.0)")
set(CPACK_DEBIAN_PACKAGE_CONTROL_EXTRA
  ${CMAKE_CURRENT_SOURCE_DIR}/packaging/postinst ${CMAKE_CURRENT_SOURCE_DIR}/packaging/postrm)
include(CPack)

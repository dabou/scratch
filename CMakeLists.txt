# Check http://elementaryos.org/docs/developer-guide/cmake for documentation

cmake_minimum_required (VERSION 2.8)
cmake_policy (VERSION 2.8)
project (io.elementary.code)
enable_testing ()
include (GNUInstallDirs)
set (DATADIR "${CMAKE_INSTALL_FULL_DATAROOTDIR}/${CMAKE_PROJECT_NAME}")
set (SCRIPTDIR "${CMAKE_INSTALL_FULL_DATAROOTDIR}/${CMAKE_PROJECT_NAME}/scripts")
set (PKGDATADIR "${DATADIR}/${CMAKE_PROJECT_NAME}")
set (GETTEXT_PACKAGE ${CMAKE_PROJECT_NAME})
set (RELEASE_NAME "The elementary Text Editor.")
set (EXEC_NAME ${CMAKE_PROJECT_NAME})
set (VERSION "2.4.1")
set (VERSION_INFO "Release")
set (LIBNAME "scratchcore")
set (LIBVERSION "0")
set (SOVERSION 0)
set (PLUGINDIR "${CMAKE_INSTALL_FULL_LIBDIR}/${CMAKE_PROJECT_NAME}/plugins")
set (PREFIX ${CMAKE_INSTALL_PREFIX})
set (DOLLAR "$")
list (APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

# Add 'make dist' command for creating release tarball
set (CPACK_PACKAGE_VERSION ${VERSION})
set (CPACK_SOURCE_GENERATOR "TGZ")
set (CPACK_SOURCE_PACKAGE_FILE_NAME "${CMAKE_PROJECT_NAME}-${CPACK_PACKAGE_VERSION}")
set (CPACK_SOURCE_IGNORE_FILES "/build/;/.bzr/;/.bzrignore;~$;${CPACK_SOURCE_IGNORE_FILES}")

include (CPack)
add_custom_target (dist COMMAND ${CMAKE_MAKE_PROGRAM} package_source)

# Some configuration
configure_file (${CMAKE_SOURCE_DIR}/src/config.vala.cmake ${CMAKE_SOURCE_DIR}/src/config.vala)
configure_file (${CMAKE_SOURCE_DIR}/src/${LIBNAME}.pc.cmake ${CMAKE_BINARY_DIR}/src/${LIBNAME}.pc)

# Some definitions
add_definitions(-DGETTEXT_PACKAGE=\"${GETTEXT_PACKAGE}\")

# Vala
find_package (Vala REQUIRED)
include (ValaVersion)
ensure_vala_version ("0.22" MINIMUM)
include (ValaPrecompile)

add_definitions(-DVALA_VERSION="${VALA_SHORTVER}")
add_definitions(-w) # Disable gcc warnings

option (USE_ZEITGEIST "Use Zeitgeist integration" ON)

if (USE_ZEITGEIST)
  message ("-- Zeitgeist integration enabled")
  set (ZEITGEIST_DEPS zeitgeist-2.0)
  set (ZEITGEIST_OPTIONS --define=HAVE_ZEITGEIST)
else ()
  message ("-- Zeitgeist integration disabled")
endif ()

# Dependencies
set (SCRATCH_DEPS
  libpeas-1.0
  libpeas-gtk-1.0
  gtksourceview-3.0
  gee-0.8
  gobject-2.0
  glib-2.0
  gio-2.0
  gtk+-3.0
  granite>=0.3.0
  ${ZEITGEIST_DEPS})

find_package (PkgConfig)
pkg_check_modules (GTK+ REQUIRED
  gtk+-3.0>=3.10)

pkg_check_modules (LIBSOURCE REQUIRED
  libpeas-gtk-1.0
  gtksourceview-3.0>=3.10
  gthread-2.0)

pkg_check_modules (DEPS REQUIRED ${SCRATCH_DEPS})

pkg_check_modules (GTKSOURCEVIEW_3_18 QUIET gtksourceview-3.0>=3.18)
if (GTKSOURCEVIEW_3_18_FOUND)
  set(GTKSOURCEVIEW_FLAGS "--define=GTKSOURCEVIEW_3_18")
  message ("-- Found gtksourceview >=3.18 GtkSourceMap available")
endif ()

set(NORMAL_CFLAGS ${DEPS_CFLAGS} ${LIBSOURCE_CFLAGS} ${GCONF_CFLAGS})
set(NORMAL_LINK_DIRS ${DEPS_LIBRARY_DIRS} ${LIBSOURCE_LIBRARY_DIRS} ${GCONF_LIBRARY_DIRS})
set(NORMAL_LIBRARIES ${DEPS_LIBRARIES} ${LIBSOURCE_LIBRARIES} ${GCONF_LIBRARIES})

add_definitions (${DEPS_CFLAGS} ${LIBSOURCE_CFLAGS} ${GCONF_CFLAGS})
link_libraries (${DEPS_LIBRARIES} ${LIBSOURCE_LIBRARIES} ${GCONF_LIBRARIES})
link_directories (${DEPS_LIBRARY_DIRS} ${LIBSOURCE_LIBRARY_DIRS} ${GCONF_LIBRARY_DIRS})


# Some include paths
include_directories(${CMAKE_BINARY_DIR}/scratchcore)
include_directories(${CMAKE_BINARY_DIR}/src)

# Subdirectories
add_subdirectory (src)
add_subdirectory (plugins)
add_subdirectory (schemas)
add_subdirectory (po)

# Data
configure_file_translation(${CMAKE_SOURCE_DIR}/data/org.pantheon.scratch.desktop.in ${CMAKE_BINARY_DIR}/data/org.pantheon.scratch.desktop ${CMAKE_SOURCE_DIR}/po/)
configure_file_translation(${CMAKE_SOURCE_DIR}/data/io.elementary.code.appdata.xml.in ${CMAKE_BINARY_DIR}/data/io.elementary.code.appdata.xml ${CMAKE_SOURCE_DIR}/po/)

install (FILES ${CMAKE_BINARY_DIR}/data/org.pantheon.scratch.desktop DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/applications)
install (FILES ${CMAKE_BINARY_DIR}/data/io.elementary.code.appdata.xml DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/metainfo)

install (FILES ${CMAKE_SOURCE_DIR}/data/icons/16/io.elementary.code.svg DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/16x16/apps)
install (FILES ${CMAKE_SOURCE_DIR}/data/icons/16/io.elementary.code.svg DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/16x16@2/apps)
install (FILES ${CMAKE_SOURCE_DIR}/data/icons/24/io.elementary.code.svg DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/24x24/apps)
install (FILES ${CMAKE_SOURCE_DIR}/data/icons/24/io.elementary.code.svg DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/24x24@2/apps)
install (FILES ${CMAKE_SOURCE_DIR}/data/icons/32/io.elementary.code.svg DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/32x32/apps)
install (FILES ${CMAKE_SOURCE_DIR}/data/icons/32/io.elementary.code.svg DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/32x32@2/apps)
install (FILES ${CMAKE_SOURCE_DIR}/data/icons/48/io.elementary.code.svg DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/48x48/apps)
install (FILES ${CMAKE_SOURCE_DIR}/data/icons/48/io.elementary.code.svg DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/48x48@2/apps)
install (FILES ${CMAKE_SOURCE_DIR}/data/icons/64/io.elementary.code.svg DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/64x64/apps)
install (FILES ${CMAKE_SOURCE_DIR}/data/icons/64/io.elementary.code.svg DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/64x64@2/apps)
install (FILES ${CMAKE_SOURCE_DIR}/data/icons/128/io.elementary.code.svg DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/128x128/apps)
install (FILES ${CMAKE_SOURCE_DIR}/data/icons/128/io.elementary.code.svg DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/128x128@2/apps)

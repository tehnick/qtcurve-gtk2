project(QtCurve-Gtk2)
cmake_minimum_required(VERSION 2.4.0)

set(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/README")
set(CPACK_GENERATOR "DEB;RPM;TGZ")
set(DEBIAN_PACKAGE_DESCRIPTION "QtCurve for Gtk2")
set(DEBIAN_PACKAGE_SECTION "gtk")
set(CPACK_SOURCE_GENERATOR "TBZ2")
set(CPACK_PACKAGE_VERSION_MAJOR "0")
set(CPACK_PACKAGE_VERSION_MINOR "55")
set(CPACK_PACKAGE_VERSION_PATCH "0")
set(CPACK_PACKAGE_CONTACT "Craig Drummond <Craig.Drummond@lycos.co.uk>")
set(QTCURVE_VERSION "${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}")
set(QTCURVE_VERSION_FULL "${QTCURVE_VERSION}.${CPACK_PACKAGE_VERSION_PATCH}")
set(CPACK_SOURCE_PACKAGE_FILE_NAME "${CMAKE_PROJECT_NAME}-${QTCURVE_VERSION_FULL}")
set(CPACK_PACKAGE_FILE_NAME "${CMAKE_PROJECT_NAME}-${QTCURVE_VERSION_FULL}")
include(CPack)

include(UsePkgConfig)
pkgconfig(gtk+-2.0 GTK_INCLUDE_DIR GTK_LIB_DIR GTK_LINK_FLAGS GTK_CFLAGS)

if (GTK_INCLUDE_DIR)
    # Try to determine KDE3's icon prefix...
    find_program(KDE3CONFIG_EXECUTABLE NAMES kde-config
                 PATHS $ENV{KDEDIR}/bin /opt/kde3/bin /opt/kde/bin NO_DEFAULT_PATH)

    find_program(KDE3CONFIG_EXECUTABLE kde-config)

    if(KDE3CONFIG_EXECUTABLE)
        exec_program(${KDE3CONFIG_EXECUTABLE}
                    ARGS --expandvars --install icon
                    OUTPUT_VARIABLE KDE3_ICONS_PREFIX)
        exec_program(${KDE3CONFIG_EXECUTABLE}
                     ARGS --prefix
                     OUTPUT_VARIABLE KDE3PREFIX)
    endif(KDE3CONFIG_EXECUTABLE)

    # Try to determine KDE4's icon prefix...
    file(TO_CMAKE_PATH "$ENV{KDEDIRS}" _KDEDIRS)

    find_program(KDE4CONFIG_EXECUTABLE NAMES kde4-config
                 PATHS ${CMAKE_INSTALL_PREFIX}/bin ${_KDEDIRS} /opt/kde4/bin NO_DEFAULT_PATH )

    if(NOT KDE4CONFIG_EXECUTABLE)
        find_program(KDE4CONFIG_EXECUTABLE NAMES kde4-config )
    endif(NOT KDE4CONFIG_EXECUTABLE)

    if(KDE4CONFIG_EXECUTABLE)
        exec_program(${KDE4_CONFIG_EXECUTABLE}
                    ARGS --expandvars --install icon
                    OUTPUT_VARIABLE KDE4_ICONS_PREFIX)
        exec_program(${KDE4_CONFIG_EXECUTABLE}
                     ARGS --prefix
                     OUTPUT_VARIABLE KDE4PREFIX)
    endif(KDE4CONFIG_EXECUTABLE)

    exec_program(${PKGCONFIG_EXECUTABLE}
                 ARGS --variable=gtk_binary_version gtk+-2.0
                 OUTPUT_VARIABLE GTK_VERSION)
    exec_program(${PKGCONFIG_EXECUTABLE}
                 ARGS --variable=prefix gtk+-2.0
                 OUTPUT_VARIABLE GTK_PREFIX)
    set(GTK_THEME_DIR ${GTK_PREFIX}/share/themes/QtCurve)
    set(CMAKE_INSTALL_PREFIX ${GTK_PREFIX})

    if(NOT KDE3PREFIX)
        if(KDE4PREFIX)
            set(KDE3PREFIX ${KDE4PREFIX})
        else(KDE4PREFIX)
            set(KDE3PREFIX /usr)
        endif(KDE4PREFIX)
    endif(NOT KDE3PREFIX)

    if(NOT KDE4PREFIX)
        if(KDE3PREFIX)
            set(KDE4PREFIX ${KDE3PREFIX})
        else(KDE3PREFIX)
            set(KDE4PREFIX /usr)
        endif(KDE3PREFIX)
    endif(NOT KDE4PREFIX)

    if(NOT KDE3_ICONS_PREFIX)
        if(KDE4_ICONS_PREFIX)
            set(KDE3_ICONS_PREFIX ${KDE4_ICONS_PREFIX})
        else(KDE4_ICONS_PREFIX)
            set(KDE3_ICONS_PREFIX ${KDE3PREFIX}/share/icons)
        endif(KDE4_ICONS_PREFIX)
    endif(NOT KDE3_ICONS_PREFIX)

    if(NOT KDE4_ICONS_PREFIX)
        if(KDE3_ICONS_PREFIX)
            set(KDE4_ICONS_PREFIX ${KDE3_ICONS_PREFIX})
        else(KDE3_ICONS_PREFIX)
            set(KDE4_ICONS_PREFIX ${KDE4PREFIX}/share/icons)
        endif(KDE3_ICONS_PREFIX)
    endif(NOT KDE4_ICONS_PREFIX)

    configure_file(config.h.cmake ${CMAKE_BINARY_DIR}/config.h)
    add_subdirectory(style)
    add_subdirectory(mozilla)
else (GTK_INCLUDE_DIR)
    MESSAGE(FATAL_ERROR "Could not locate Gtk2 include files")
endif (GTK_INCLUDE_DIR)

kdeconnect_add_plugin(kdeconnect_mousepad SOURCES mousepadplugin.cpp abstractremoteinput.cpp)

if(UNIX AND NOT APPLE)
    find_package(KF5 ${KF5_MIN_VERSION} QUIET OPTIONAL_COMPONENTS Wayland)

    find_package(LibFakeKey QUIET)
    set_package_properties(LibFakeKey PROPERTIES DESCRIPTION "fake key events"
                        URL "https://www.yoctoproject.org/tools-resources/projects/matchbox"
                        TYPE OPTIONAL
                        PURPOSE "Needed for the remote mousepad plugin"
                        )
    if (LibFakeKey_FOUND)
        find_package(Qt5 ${QT_MIN_VERSION} REQUIRED COMPONENTS X11Extras)
        find_package(XTest REQUIRED)
        find_package(X11 REQUIRED)
        target_include_directories(kdeconnect_mousepad PRIVATE ${XTEST_INCLUDE_DIRS} ${X11_INCLUDE_DIR} ${LibFakeKey_INCLUDE_DIRS})
    endif()
endif()

set(HAVE_WINDOWS ${WIN32})
set(HAVE_X11 ${LibFakeKey_FOUND})
set(HAVE_WAYLAND ${KF5Wayland_FOUND})
set(HAVE_MACOS ${APPLE})
configure_file(config-mousepad.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config-mousepad.h )

target_link_libraries(kdeconnect_mousepad kdeconnectcore Qt5::Gui KF5::I18n)

if (HAVE_WINDOWS)
    target_sources(kdeconnect_mousepad PUBLIC windowsremoteinput.cpp)
endif()

if(HAVE_WAYLAND)
    target_sources(kdeconnect_mousepad PUBLIC waylandremoteinput.cpp)
    target_link_libraries(kdeconnect_mousepad KF5::WaylandClient)
endif()

if(HAVE_X11)
    target_sources(kdeconnect_mousepad PUBLIC x11remoteinput.cpp)
    target_link_libraries(kdeconnect_mousepad Qt5::X11Extras ${X11_LIBRARIES} ${XTEST_LIBRARIES} ${LibFakeKey_LIBRARIES})
endif()

if (HAVE_MACOS)
    target_sources(kdeconnect_mousepad PUBLIC macosremoteinput.mm)
    target_link_libraries(kdeconnect_mousepad "-framework CoreGraphics" "-framework ApplicationServices" "-framework Cocoa")
endif()

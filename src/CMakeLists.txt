# in version 3.6 IMPORTED_TARGET has been added to pkg_*_modules
cmake_minimum_required(VERSION 3.6)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(Qt5 REQUIRED COMPONENTS Core Widgets)

find_package(PkgConfig)
pkg_check_modules(glib REQUIRED glib-2.0>=2.40 IMPORTED_TARGET)


# disable Qt debug messages except for debug builds
if (NOT CMAKE_BUILD_TYPE STREQUAL "Debug")
    message(STATUS "CMake build type is ${CMAKE_BUILD_TYPE}, hence disabling debug messages in AppImageLauncher")
    add_definitions(-DQT_NO_DEBUG_OUTPUT)
    add_definitions(-DQT_NO_INFO_OUTPUT)
    add_definitions(-DQT_NO_WARNING_OUTPUT)
endif()


# utility library
add_library(shared STATIC shared.h shared.cpp trashbin.cpp trashbin.h)
target_link_libraries(shared PUBLIC PkgConfig::glib Qt5::Core Qt5::Widgets xdg-basedir libappimage)
target_compile_definitions(shared
    PRIVATE -DCMAKE_INSTALL_PREFIX="${CMAKE_INSTALL_PREFIX}"
)

add_library(translationmanager translationmanager.cpp translationmanager.h)
target_link_libraries(translationmanager PUBLIC Qt5::Core Qt5::Widgets)
target_compile_definitions(translationmanager PRIVATE -DCMAKE_PROJECT_SOURCE_DIR="${PROJECT_SOURCE_DIR}")


# main AppImageLauncher application
add_executable(AppImageLauncher main.cpp)
target_link_libraries(AppImageLauncher shared translationmanager Qt5::Core Qt5::Widgets PkgConfig::glib libappimage)
# set binary runtime rpath to make sure the libappimage.so built and installed by this project is going to be used
# by the installed binaries (be it the .deb, the AppImage, or whatever)
# in order to make the whole install tree relocatable, a relative path is used
set_target_properties(AppImageLauncher PROPERTIES INSTALL_RPATH "\$ORIGIN/../lib/appimagelauncher")

# add dependency on runtime to make sure it's built
add_dependencies(AppImageLauncher runtime)

install(
    TARGETS
    AppImageLauncher
    RUNTIME DESTINATION bin COMPONENT APPIMAGELAUNCHER
    LIBRARY DESTINATION lib COMPONENT APPIMAGELAUNCHER
)


# AppImage removal helper
add_executable(remove remove_main.cpp)
target_link_libraries(remove shared translationmanager Qt5::Core Qt5::Widgets libappimage)
# see AppImageLauncher for a description
set_target_properties(remove PROPERTIES INSTALL_RPATH "\$ORIGIN")

install(
    TARGETS remove
    RUNTIME DESTINATION lib/appimagelauncher COMPONENT APPIMAGELAUNCHER
)


# AppImage update helper
add_executable(update update_main.cpp)
target_link_libraries(update shared translationmanager Qt5::Core Qt5::Widgets libappimage libappimageupdate-qt)
# see AppImageLauncher for a description
set_target_properties(update PROPERTIES INSTALL_RPATH "\$ORIGIN")

install(
    TARGETS update
    RUNTIME DESTINATION lib/appimagelauncher COMPONENT APPIMAGELAUNCHER
)

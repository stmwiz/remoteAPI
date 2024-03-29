cmake_minimum_required(VERSION 3.14)

project(test_remote_ui VERSION 0.1 LANGUAGES CXX)
set(TAG_NAME test_remote_ui)
include(AddLib)
include(FileRecursive)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)


foreach(MOD Core Gui Multimedia MultimediaWidgets Widgets)
    find_package(QT NAMES Qt6 COMPONENTS ${MOD})
    find_package(Qt${QT_VERSION_MAJOR} COMPONENTS ${MOD})
endforeach()

#find_package(Qt6 COMPONENTS Multimedia)
#find_package(Qt6 COMPONENTS MultimediaWidgets)

list(APPEND _INCLUDES ${CMAKE_SOURCE_DIR}/src)
list(APPEND _INCLUDES ${CMAKE_SOURCE_DIR}/src/webrtc_moudle)
file(GLOB_RECURSE _SOURCES ${CMAKE_SOURCE_DIR}/src/*.cpp)

set(PROJECT_SOURCES
    main.cpp
    dialog.cpp
    dialog.hpp
    dialog.ui
    ${_SOURCES}
    ${META_SOURCES}
)

if(${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_executable(${TAG_NAME}
        MANUAL_FINALIZATION
        ${PROJECT_SOURCES}
    )
# Define target properties for Android with Qt 6 as:
#    set_property(TARGET ${TAG_NAME} APPEND PROPERTY QT_ANDROID_PACKAGE_SOURCE_DIR
#                 ${CMAKE_CURRENT_SOURCE_DIR}/android)
# For more information, see https://doc.qt.io/qt-6/qt-add-executable.html#target-creation
else()
    if(ANDROID)
        add_library(${TAG_NAME} SHARED
            ${PROJECT_SOURCES}
        )
    # Define properties for Android with Qt 5 after find_package() calls as:
    #    set(ANDROID_PACKAGE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/android")
else()
    add_executable(${TAG_NAME}
        ${PROJECT_SOURCES}
    )
endif()
endif()

target_link_libraries(${TAG_NAME} PRIVATE
    Qt${QT_VERSION_MAJOR}::Core
    Qt${QT_VERSION_MAJOR}::Gui
    Qt${QT_VERSION_MAJOR}::Multimedia
    Qt${QT_VERSION_MAJOR}::MultimediaWidgets
    Qt${QT_VERSION_MAJOR}::Widgets
)

set_target_properties(${TAG_NAME} PROPERTIES
    MACOSX_BUNDLE_GUI_IDENTIFIER my.test_remote.com
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    MACOSX_BUNDLE TRUE
    WIN32_EXECUTABLE TRUE
)

if(QT_VERSION_MAJOR EQUAL 6)
    qt_finalize_executable(${TAG_NAME})
endif()

target_include_directories(${TAG_NAME} PRIVATE
    ${META_INCLUDES}
    ${CMAKE_SOURCE_DIR}/include
    ${_INCLUDES}
)
#message("META_INCLUDES = ${META_INCLUDES}")

target_link_directories(${TAG_NAME} PRIVATE
    ${META_DIRECTORIES}
    ${CMAKE_CURRENT_BINARY_DIR})
#message("META_DIRECTORIES = ${META_DIRECTORIES}")


if(ANDROID)
    set(_ANDROID_LIBRARY_DIR                            ${CMAKE_CURRENT_BINARY_DIR}/android-build/libs/${CMAKE_ANDROID_ARCH_ABI})
    file(GLOB META_SO                                   ${CMAKE_BINARY_DIR}/meta_libs/*.so)
    file(COPY ${META_SO}       DESTINATION              ${_ANDROID_LIBRARY_DIR} FOLLOW_SYMLINK_CHAIN)
    message("file(COPY ${META_SO} DESTINATION           ${_ANDROID_LIBRARY_DIR} FOLLOW_SYMLINK_CHAIN)")
endif(ANDROID)

find_libs(log-lib log)
find_libs(android-lib android)
#find_libs(pthread-lib pthread)
#message(" findliblist =${findliblist}")


target_link_libraries(${TAG_NAME} PRIVATE
    ${META_LIBS}
    ${findliblist}
)
#message("META_LIBS = ${META_LIBS}")

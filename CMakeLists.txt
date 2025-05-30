#
# This file is part of KDDockWidgets.
#
# SPDX-FileCopyrightText: 2019-2023 Klarälvdalens Datakonsult AB, a KDAB Group company <info@kdab.com>
# Author: Sergio Martins <sergio.martins@kdab.com>
#
# SPDX-License-Identifier: GPL-2.0-only OR GPL-3.0-only
#
# Contact KDAB at <info@kdab.com> for commercial licensing options.
#

# Pass the following variables to cmake to control the build:
#
# -DKDDockWidgets_QT6=[true|false]
#  Build against Qt6 rather than Qt5
#  Default=false (Qt5 will be used even if Qt6 is available)
#
# -DKDDockWidgets_STATIC=[true|false]
#  Build static versions of the libraries
#  Default=false
#
# -DKDDockWidgets_TESTS=[true|false]
#  Build the test harness.
#  Currently ignored (except for Python bindings) unless KDDockWidgets_DEVELOPER_MODE=True.
#  Default=false
#
# -DKDDockWidgets_EXAMPLES=[true|false]
#  Build the examples.
#  Default=true
#
# -DKDDockWidgets_DOCS=[true|false]
#  Build the API documentation. Enables the 'docs' build target.
#  Default=false
#
# -DKDDockWidgets_QTQUICK=[true|false]
#  Build for QtQuick instead of QtWidgets.
#  Default=false
#
# -DKDDockWidgets_PYTHON_BINDINGS=[true|false]
#  Build/Generate python bindings.  Always false for Debug builds
#  (If your shiboken or pyside is installed in a non-standard locations
#  try passing the SHIBOKEN_CUSTOM_PREFIX and PYSIDE_CUSTOM_PREFIX variables.)
#  Default=false
#
# -DKDDockWidgets_PYTHON_BINDINGS_INSTALL_PREFIX=[path]
#  Set an alternative install path for Python bindings
#  Default=CMAKE_INSTALL_PREFIX

# ## DO NOT USE IF YOU ARE AN END-USER.  FOR THE DEVELOPERS ONLY!!
## Special CMake Options for Developers
#
# -DKDDockWidgets_DEVELOPER_MODE=[true|false]
#  Configure the build for a developer setup.
#  Enables some features that are not geared towards end-users.
#  Forces the test harness to be built.
#  Default=false
#
# -DKDDockWidgets_WERROR=[true|false]
#  Compile with the -Werror gcc/clang option (always true for developer-mode)
#  Default=false
#
# -DKDDockWidgets_LINTER=[true|false]
#  Build the layout linter.
#  Ignored unless KDDockWidgets_DEVELOPER_MODE=True
#  Default=true

cmake_minimum_required(VERSION 3.12)

# Allow using a non-KDAB install location.
set(KDAB_INSTALL
    True
    CACHE INTERNAL "Install to default KDAB Location"
)
if(DEFINED CMAKE_INSTALL_PREFIX)
    if(NOT "${CMAKE_INSTALL_PREFIX}" STREQUAL "")
        set(KDAB_INSTALL
            False
            CACHE INTERNAL "Install to non-KDAB Location"
        )
    endif()
endif()

project(
    KDDockWidgets
    DESCRIPTION "An advanced docking system for Qt"
    HOMEPAGE_URL "https://github.com/KDAB/KDDockWidgets"
    LANGUAGES CXX
)

set(${PROJECT_NAME}_VERSION_MAJOR 1)
set(${PROJECT_NAME}_VERSION_MINOR 7)
set(${PROJECT_NAME}_VERSION_PATCH 1)
set(${PROJECT_NAME}_VERSION
    ${${PROJECT_NAME}_VERSION_MAJOR}.${${PROJECT_NAME}_VERSION_MINOR}.${${PROJECT_NAME}_VERSION_PATCH}
)
set(PROJECT_VERSION ${${PROJECT_NAME}_VERSION}) #PROJECT_VERSION is needed by some ECM modules
set(${PROJECT_NAME}_SOVERSION "1.7")

include(FeatureSummary)

option(${PROJECT_NAME}_QT6 "Build against Qt 6" OFF)
option(${PROJECT_NAME}_DEVELOPER_MODE "Developer Mode" OFF)
option(${PROJECT_NAME}_PYTHON_BINDINGS "Build python bindings" OFF)
option(${PROJECT_NAME}_QTQUICK "Build for QtQuick instead of QtWidgets" OFF)
option(${PROJECT_NAME}_STATIC "Build statically" OFF)
option(${PROJECT_NAME}_TESTS "Build the tests" OFF)
option(${PROJECT_NAME}_EXAMPLES "Build the examples" ON)
option(${PROJECT_NAME}_DOCS "Build the API documentation" OFF)
option(${PROJECT_NAME}_WERROR "Use -Werror (will be true for developer-mode unconditionally)" OFF)
option(${PROJECT_NAME}_X11EXTRAS
       "Link with QtX11Extras to detect if the compositor supports transparency. Not applicable to non-Linux or Qt6."
       ON
)
option(${PROJECT_NAME}_XLib "On Linux, link against XLib, for a more robust window z-order detection." OFF)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake/ECM/modules")
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake/KDAB/modules")

# Set a default build type if none was specified
set(default_build_type "Release")
if(EXISTS "${CMAKE_SOURCE_DIR}/.git" OR ${PROJECT_NAME}_DEVELOPER_MODE)
    set(default_build_type "Debug")
endif()
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to ${default_build_type} as none was specified.")
    set(CMAKE_BUILD_TYPE
        "${default_build_type}"
        CACHE STRING "Choose the type of build." FORCE
    )
    # Set the possible values of build type for cmake-gui
    set_property(
        CACHE CMAKE_BUILD_TYPE
        PROPERTY STRINGS
                 "Debug"
                 "Release"
                 "MinSizeRel"
                 "RelWithDebInfo"
    )
endif()

if(${PROJECT_NAME}_XLib)
    add_definitions(-DKDDockWidgets_XLIB)
endif()

if(${PROJECT_NAME}_QT6)
    set(Qt_VERSION_MAJOR 6)
    set(QT_MIN_VERSION "6.2.0")
    set(${PROJECT_NAME}_LIBRARY_QTID "-qt6")
else()
    set(Qt_VERSION_MAJOR 5)
    set(QT_MIN_VERSION "5.15")
    set(${PROJECT_NAME}_LIBRARY_QTID "")
endif()
find_package(Qt${Qt_VERSION_MAJOR} ${QT_MIN_VERSION} NO_MODULE REQUIRED COMPONENTS Widgets Test)
include(KDQtInstallPaths) #to set QT_INSTALL_FOO variables

set(${PROJECT_NAME}_DEPS "widgets")
if(${PROJECT_NAME}_QTQUICK)
    find_package(Qt${Qt_VERSION_MAJOR} NO_MODULE REQUIRED COMPONENTS Quick QuickControls2)
    add_definitions(-DKDDOCKWIDGETS_QTQUICK)
    set(${PROJECT_NAME}_DEPS "${${PROJECT_NAME}_DEPS} quick quickcontrols2")
else()
    add_definitions(-DKDDOCKWIDGETS_QTWIDGETS)
endif()

if(WIN32 AND (Qt6Core_VERSION VERSION_EQUAL "6.5.3" OR Qt6Core_VERSION VERSION_EQUAL "6.6.1"))
    message(
        FATAL_ERROR
            "Qt 6.5.3 and 6.6.1 are not supported on Windows, as they suffer from a Qt regression: QTBUG-117704. "
            "Please either downgrade or upgrade to 6.6.2 or superior"
    )
endif()

if(NOT WIN32
   AND NOT APPLE
   AND NOT EMSCRIPTEN
   AND NOT ${PROJECT_NAME}_QT6
   AND ${PROJECT_NAME}_X11EXTRAS
)
    set(${PROJECT_NAME}_DEPS "${${PROJECT_NAME}_DEPS} x11extras")
endif()

#Always build the test harness in developer-mode
if(${PROJECT_NAME}_DEVELOPER_MODE)
    set(${PROJECT_NAME}_TESTS ON)
    set(${PROJECT_NAME}_WERROR ON)
    include(ECMEnableSanitizers)
endif()

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    set(IS_CLANG_BUILD TRUE)
else()
    set(IS_CLANG_BUILD FALSE)
endif()

if(${PROJECT_NAME}_QTQUICK)
    set(CMAKE_CXX_STANDARD 17)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

# Default to hidden visibility for symbols
set(CMAKE_C_VISIBILITY_PRESET hidden)
set(CMAKE_CXX_VISIBILITY_PRESET hidden)
set(CMAKE_VISIBILITY_INLINES_HIDDEN 1)

# Sets compiler flags for the specified target, taking platform into consideration.
macro(set_compiler_flags targetName)
    if(${PROJECT_NAME}_DEVELOPER_MODE)
        target_compile_definitions(
            ${targetName}
            PUBLIC DOCKS_DEVELOPER_MODE
            PRIVATE QT_FORCE_ASSERTS
        )

        if(NOT MSVC)
            target_compile_options(${targetName} PRIVATE -Wall -Wextra)
        endif()

        if(APPLE)
            target_compile_options(${targetName} PRIVATE -Wweak-vtables)
        endif()
    endif()

    # Enable -Werror
    if(${PROJECT_NAME}_WERROR AND (NOT MSVC OR IS_CLANG_BUILD)) # clang-cl accepts these too
        target_compile_options(${targetName} PRIVATE -Werror -Wundef -Wno-error=deprecated-declarations)
    endif()
endmacro()

if((CMAKE_CXX_COMPILER_ID STREQUAL "GNU" AND NOT APPLE)
   OR (CMAKE_CXX_COMPILER_ID MATCHES "Clang" AND NOT APPLE)
   OR (CMAKE_CXX_COMPILER_ID STREQUAL "Intel" AND NOT WIN32)
)
    # Linker warnings should be treated as errors
    set(CMAKE_SHARED_LINKER_FLAGS "-Wl,--fatal-warnings ${CMAKE_SHARED_LINKER_FLAGS}")
    set(CMAKE_MODULE_LINKER_FLAGS "-Wl,--fatal-warnings ${CMAKE_MODULE_LINKER_FLAGS}")

    string(TOUPPER "CMAKE_CXX_FLAGS_${CMAKE_BUILD_TYPE}" compileflags)
    if("${CMAKE_CXX_FLAGS} ${${compileflags}}" MATCHES "-fsanitize")
        set(sanitizers_enabled TRUE)
    else()
        set(sanitizers_enabled FALSE)
    endif()

    if(APPLE OR LINUX)
        # cannot enable this for clang + sanitizers
        if(NOT sanitizers_enabled OR NOT CMAKE_CXX_COMPILER_ID MATCHES "Clang")
            # Do not allow undefined symbols, even in non-symbolic shared libraries
            set(CMAKE_SHARED_LINKER_FLAGS "-Wl,--no-undefined ${CMAKE_SHARED_LINKER_FLAGS}")
            set(CMAKE_MODULE_LINKER_FLAGS "-Wl,--no-undefined ${CMAKE_MODULE_LINKER_FLAGS}")
        endif()
    endif()
endif()

if(${PROJECT_NAME}_STATIC)
    set(${PROJECT_NAME}_LIBRARY_MODE "STATIC")
else()
    set(${PROJECT_NAME}_LIBRARY_MODE "SHARED")
endif()

if(KDAB_INSTALL)
    if(UNIX)
        set(CMAKE_INSTALL_PREFIX
            "/usr/local/KDAB/${PROJECT_NAME}-${${PROJECT_NAME}_VERSION}"
            CACHE INTERNAL "Install to default KDAB Location"
        )
    elseif(WIN32)
        set(CMAKE_INSTALL_PREFIX
            "C:\\KDAB\\${PROJECT_NAME}-${${PROJECT_NAME}_VERSION}"
            CACHE INTERNAL "Install to default KDAB Location"
        )
    endif()
endif()

# setup default install locations
include(KDInstallLocation)

if(CMAKE_SOURCE_DIR STREQUAL PROJECT_SOURCE_DIR)
    set(${PROJECT_NAME}_IS_ROOT_PROJECT TRUE)

    message(STATUS "Building ${PROJECT_NAME} ${${PROJECT_NAME}_VERSION} in ${CMAKE_BUILD_TYPE} mode. "
                   "Installing to ${CMAKE_INSTALL_PREFIX}"
    )

    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/bin")
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/lib")
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/lib")

    install(FILES LICENSE.txt README.md DESTINATION ${INSTALL_DOC_DIR})
    install(DIRECTORY LICENSES DESTINATION ${INSTALL_DOC_DIR})

    # Generate .pri file for qmake users (except when using the VS generator)
    if(NOT CMAKE_CONFIGURATION_TYPES)
        if(Qt_VERSION_MAJOR EQUAL 5 OR (Qt_VERSION_MAJOR EQUAL 6 AND Qt6Core_VERSION VERSION_GREATER "6.2"))
            include(ECMGeneratePriFile)
            set(PROJECT_VERSION_STRING ${${PROJECT_NAME}_VERSION})
            ecm_generate_pri_file(
                BASE_NAME
                KDDockWidgets
                LIB_NAME
                kddockwidgets${${PROJECT_NAME}_LIBRARY_QTID}
                DEPS
                ${${PROJECT_NAME}_DEPS}
                FILENAME_VAR
                pri_filename
                INCLUDE_INSTALL_DIR
                ${CMAKE_INSTALL_INCLUDEDIR}
            )
            install(FILES ${pri_filename} DESTINATION ${ECM_MKSPECS_INSTALL_DIR})
        endif()
    endif()
else()
    #Always disable tests, examples, docs when used as a submodule
    set(${PROJECT_NAME}_IS_ROOT_PROJECT FALSE)
    set(${PROJECT_NAME}_TESTS FALSE)
    set(${PROJECT_NAME}_EXAMPLES FALSE)
    set(${PROJECT_NAME}_DOCS FALSE)
endif()

if(${PROJECT_NAME}_TESTS)
    enable_testing()
endif()

add_subdirectory(src)

if(${PROJECT_NAME}_PYTHON_BINDINGS)
    if(CMAKE_BUILD_TYPE MATCHES "^[Dd]eb" OR ${PROJECT_NAME}_STATIC)
        message(FATAL_ERROR "** Python Bindings are disabled in debug or static builds.")
    endif()
    if(CMAKE_UNITY_BUILD)
        message(FATAL_ERROR "** Python Bindings are disabled in Unity builds. " "Try again with CMAKE_UNITY_BUILD=OFF")
    endif()
endif()
if(${PROJECT_NAME}_PYTHON_BINDINGS)
    add_subdirectory(python)
endif()

if(${PROJECT_NAME}_EXAMPLES)
    if(${PROJECT_NAME}_QTQUICK)
        add_subdirectory(examples/qtquick)
    else()
        add_subdirectory(examples/dockwidgets)
        add_subdirectory(examples/minimal)
        add_subdirectory(examples/minimal-mdi)
        add_subdirectory(examples/mdi_with_docking)
        set_compiler_flags(kddockwidgets_example)
        set_compiler_flags(kddockwidgets_minimal_example)
        set_compiler_flags(kddockwidgets_mdi_with_docking_example)
    endif()
endif()

if(${PROJECT_NAME}_TESTS)
    if(${PROJECT_NAME}_DEVELOPER_MODE)
        add_subdirectory(tests)

        add_test(NAME tst_docks COMMAND tst_docks)

        if(NOT ${PROJECT_NAME}_QTQUICK)
            # tst_multisplitter depends on QWidget
            add_test(NAME tst_multisplitter COMMAND tst_multisplitter)
        endif()

    endif()
endif()

if(${PROJECT_NAME}_DOCS)
    add_subdirectory(docs) # needs to go last, in case there are build source files
endif()

if(${PROJECT_NAME}_IS_ROOT_PROJECT)
    # Add uninstall target (not for submodules since parent projects typically have uninstall too)
    include(ECMUninstallTarget)
endif()

# Deployment
if(WIN32)
    add_custom_target(
        createZipDemo
        COMMAND cmd /c ${CMAKE_CURRENT_SOURCE_DIR}\\deploy\\create-demo-win-zip.bat
                ${CMAKE_PROJECT_NAME}-Demo-${${PROJECT_NAME}_VERSION}
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        COMMENT "Target to generate the Zip demo installer for Windows"
    )
endif()

feature_summary(WHAT ALL INCLUDE_QUIET_PACKAGES FATAL_ON_MISSING_REQUIRED_PACKAGES)

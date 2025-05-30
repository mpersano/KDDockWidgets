#
# This file is part of KDDockWidgets.
#
# SPDX-FileCopyrightText: 2020-2023 Klarälvdalens Datakonsult AB, a KDAB Group company <info@kdab.com>
# Author: Renato Araujo Oliveira Filho <renato.araujo@kdab.com>
#
# SPDX-License-Identifier: GPL-2.0-only OR GPL-3.0-only
#
# Contact KDAB at <info@kdab.com> for commercial licensing options.
#
set(PYTHON_BINDING_NAMESPACE "PyKDDockWidgets")

# Just to fix warnings with --warn-uninitialized
if(NOT DEFINED SHIBOKEN_CUSTOM_PREFIX) # look for shiboken in a custom location
    set(SHIBOKEN_CUSTOM_PREFIX "")
endif()
if(NOT DEFINED PYSIDE_CUSTOM_PREFIX) # look for pyside in a custom location
    set(PYSIDE_CUSTOM_PREFIX "")
endif()

if(${PROJECT_NAME}_QT6)
    set(PYSIDE_MAJOR_VERSION "6")
    set(PYTHON_BINDING_NAMESPACE "${PYTHON_BINDING_NAMESPACE}Qt${PYSIDE_MAJOR_VERSION}")
    set(QtWidgets_VERSION ${Qt6Widgets_VERSION})
else()
    set(PYSIDE_MAJOR_VERSION "2")
    set(QtWidgets_VERSION ${Qt5Widgets_VERSION})
endif()

find_package(Python3 3.7 REQUIRED COMPONENTS Interpreter Development)
find_package(Shiboken${PYSIDE_MAJOR_VERSION} REQUIRED)
find_package(PySide${PYSIDE_MAJOR_VERSION} ${QtWidgets_VERSION} EXACT REQUIRED)

if(NOT ${PROJECT_NAME}_PYTHON_BINDINGS_INSTALL_PREFIX)
    # cmake-lint: disable=C0103
    set(${PROJECT_NAME}_PYTHON_BINDINGS_INSTALL_PREFIX
        ${INSTALL_LIBRARY_DIR}
        CACHE FILEPATH "Custom path to install python bindings."
    )
endif()
if(NOT WIN32)
    set(Python3_VERSION_MAJORMINOR "${Python3_VERSION_MAJOR}.${Python3_VERSION_MINOR}")
    set(BINDINGS_DIR "python${Python3_VERSION_MAJORMINOR}/site-packages/${PYTHON_BINDING_NAMESPACE}")
else()
    set(BINDINGS_DIR "site-packages/${PYTHON_BINDING_NAMESPACE}")
endif()
set(${PROJECT_NAME}_PYTHON_BINDINGS_INSTALL_PREFIX "${${PROJECT_NAME}_PYTHON_BINDINGS_INSTALL_PREFIX}/${BINDINGS_DIR}")

include(KDPySide${PYSIDE_MAJOR_VERSION}ModuleBuild)
add_subdirectory(PyKDDockWidgets)
if(${PROJECT_NAME}_TESTS)
    add_subdirectory(tests)
endif()

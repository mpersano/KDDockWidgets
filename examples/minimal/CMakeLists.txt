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

cmake_minimum_required(VERSION 3.7)
project(kddockwidgets_minimal_example)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_INCLUDE_CURRENT_DIRS ON)

if(NOT TARGET kddockwidgets)
    # This will look for Qt, do find_package yourself manually before
    # if you want to look for a specific Qt version for instance.
    find_package(KDDockWidgets REQUIRED)
endif()

set(RESOURCES_EXAMPLE_SRC ${CMAKE_CURRENT_SOURCE_DIR}/../dockwidgets/resources_example.qrc)

add_executable(
    kddockwidgets_minimal_example
    ${RESOURCES_EXAMPLE_SRC} ../dockwidgets/MyWidget.cpp main.cpp
)

target_link_libraries(
    kddockwidgets_minimal_example
    PRIVATE KDAB::kddockwidgets
)

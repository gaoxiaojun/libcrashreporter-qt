cmake_minimum_required(VERSION 3.0.2)
project(libcrashreporter-qt)
include(cmake/macroSetupQt5.cmake)
MacroSetupQt5()
add_subdirectory(3rdparty)
add_subdirectory(src)


cmake_minimum_required(VERSION 3.0.2)

project(libcrashreporter-qt)

set(LIBCRASHREPORTER_MAJOR 0)
set(LIBCRASHREPORTER_MINOR 0)
set(LIBCRASHREPORTER_MICRO 1)

set(LIBCRASHREPORTER_VERSION "${LIBCRASHREPORTER_MAJOR}.${LIBCRASHREPORTER_MINOR}.${LIBCRASHREPORTER_MICRO}")

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release")
endif(NOT CMAKE_BUILD_TYPE)

## Setup Qt5
include(cmake/macroSetupQt5.cmake)
MacroSetupQt5()

## Setup Install
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)

set(CMAKE_AUTOMOC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)

include_directories(
    src/breakpad
    src/libcrashreporter-gui
    src/libcrashreporter-handler
)

if(UNIX)
    list(APPEND LIB_SOURCES
        src/breakpad/client/minidump_file_writer.cc
        src/breakpad/common/convert_UTF.c
        src/breakpad/common/string_conversion.cc
    )

    if(APPLE)
        list(APPEND LIB_SOURCES
            src/breakpad/client/mac/crash_generation/crash_generation_client.cc
            src/breakpad/client/mac/crash_generation/crash_generation_server.cc
            src/breakpad/client/mac/handler/breakpad_nlist_64.cc
            src/breakpad/client/mac/handler/dynamic_images.cc
            src/breakpad/client/mac/handler/exception_handler.cc
            src/breakpad/client/mac/handler/minidump_generator.cc
            src/breakpad/client/mac/handler/protected_memory_allocator.cc
            src/breakpad/common/mac/file_id.cc
            src/breakpad/common/mac/macho_id.cc
            src/breakpad/common/mac/macho_reader.cc
            src/breakpad/common/mac/macho_utilities.cc
            src/breakpad/common/mac/macho_walker.cc
            src/breakpad/common/mac/string_utilities.cc
            src/breakpad/common/md5.cc
            src/breakpad/common/mac/dump_syms.mm
            src/breakpad/common/mac/MachIPC.mm
            src/breakpad/common/dwarf/dwarf2reader.cc
            src/breakpad/common/mac/bootstrap_compat.cc
            src/breakpad/common/dwarf_cfi_to_module.cc
        )
    else()
        list(APPEND LIB_SOURCES
            src/breakpad/client/linux/crash_generation/crash_generation_client.cc
            src/breakpad/client/linux/handler/exception_handler.cc
            src/breakpad/client/linux/handler/minidump_descriptor.cc
            src/breakpad/client/linux/log/log.cc
            src/breakpad/client/linux/minidump_writer/linux_dumper.cc
            src/breakpad/client/linux/minidump_writer/linux_ptrace_dumper.cc
            src/breakpad/client/linux/minidump_writer/minidump_writer.cc
            src/breakpad/common/linux/elfutils.cc
            src/breakpad/common/linux/file_id.cc
            src/breakpad/common/linux/guid_creator.cc
            src/breakpad/common/linux/linux_libc_support.cc
            src/breakpad/common/linux/memory_mapped_file.cc
            src/breakpad/common/linux/safe_readlink.cc
        )
    endif()
endif()

if(WIN32)
    add_definitions( -DUNICODE -fshort-wchar )
    list(APPEND LIB_SOURCES
        src/breakpad/client/windows/handler/exception_handler.cc
        src/breakpad/client/windows/crash_generation/crash_generation_client.cc
        src/breakpad/common/windows/guid_string.cc
    )
endif()

list(APPEND LIB_SOURCES
    src/libcrashreporter-gui/CrashReporter.cpp
    src/libcrashreporter-handler/Handler.cpp
    )

set(LIB_HEADERS src/libcrashreporter-gui/CrashReporter.h)
set(LIB_UI src/libcrashreporter-gui/CrashReporter.ui)

qt5_wrap_ui( LIB_UI_HEADERS ${LIB_UI} )

add_library(crashreporter STATIC
    ${LIB_SOURCES}
    ${LIB_HEADERS}
    ${LIB_UI_HEADERS}
)


target_link_libraries(crashreporter
    Qt5::Core
    Qt5::Widgets
    Qt5::Network
)

set(CRASHREPORTER_INCLUDE_DIRS ${CMAKE_INSTALL_PREFIX}/include CACHE INTERNAL "crashreporter_include" FORCE)
set(CRASHREPORTER_LIBRARIES crashreporter CACHE INTERNAL "crashreporter_library" FORCE)

install_files(${CMAKE_INSTALL_PREFIX}/include FILES src/libcrashreporter-gui/CrashReporter.h)
install(TARGETS crashreporter ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)


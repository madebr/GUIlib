cmake_minimum_required(VERSION 3.16)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

set(MAJOR_VERSION 2)
set(MINOR_VERSION 0)
set(MICRO_VERSION 0)
set(SDL_REQUIRED_VERSION 2.0.0)

set(DYLIB_COMPATIBILITY_VERSION "1")

include(PrivateSdlFunctions)
sdl_calculate_derived_version_variables()

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_BINARY_DIR)
    message(FATAL_ERROR "Prevented in-tree built. Please create a build directory outside of the SDL_net source code and call cmake from there")
endif()

project(GUIlib2
    LANGUAGES C CXX
    VERSION "${FULL_VERSION}"
)

set(GUILIB2_DEBUG_POSFIX "d"
    CACHE STRING "Name suffix for debug builds")
mark_as_advanced(GUILIB2_DEBUG_POSFIX)

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

option(CMAKE_POSITION_INDEPENDENT_CODE "Build static libraries with -fPIC" ON)
option(BUILD_SHARED_LIBS "Build the library as a shared library" ON)

option(GUILIB2_INSTALL "Enable GUILIB2 install target" ON)

option(GUILIB2_SAMPLES "Build samples" ON)

# Save BUILD_SHARED_LIBS variable
set(GUILIB2_BUILD_SHARED_LIBS ${BUILD_SHARED_LIBS})

if(GUILIB2_BUILD_SHARED_LIBS)
    set(guilib2_export_name GUIlib2)
    set(guilib2_install_name_infix shared)
    set(sdl2_target_name SDL2::SDL2)
else()
    set(guilib2_export_name GUIlib2-static)
    set(guilib2_install_name_infix static)
    set(sdl2_target_name SDL2::SDL2-static)
endif()

sdl_find_sdl2(${sdl2_target_name} ${SDL_REQUIRED_VERSION})

set(guilib2_headers
    GUI.h
    GUI_C.h
    GUI_area.h
    GUI_button.h
    GUI_font.h
    GUI_generic.h
    GUI_image.h
    GUI_loadimage.h
    GUI_menu.h
    GUI_output.h
    GUI_scroll.h
    GUI_scrollbar.h
    GUI_status.h
    GUI_termwin.h
    GUI_widget.h
    GUI_widgets.h
)

add_library(GUIlib2
    GUI.cpp
    GUI_C.cpp
    GUI_area.cpp
    GUI_button.cpp
    GUI_font.cpp
    GUI_generic.cpp
    GUI_image.cpp
    GUI_loadimage.cpp
    GUI_loadimage.h
    GUI_menu.cpp
    GUI_output.cpp
    GUI_output.h
    GUI_scrollbar.cpp
    GUI_termwin.cpp
    GUI_types.h
    GUI_widget.cpp
    cancel_img.h
    okay_img.h
    the_font.h
    the_checker.h
    ${guilib2_headers}
)
add_library(GUIlib2::${guilib2_export_name} ALIAS GUIlib2)
target_include_directories(GUIlib2 PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
    "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/GUI2>"
)
if(MSVC)
else()
    target_compile_options(GUIlib2 PRIVATE
        -Wall
    )
endif()
set_target_properties(GUIlib2
    PROPERTIES
        CXX_STANDARD 98
        CXX_EXTENSIONS OFF
        OUTPUT_NAME GUI
)
if(NOT GUILIB2_BUILD_SHARED_LIBS)
    if(MSVC OR (WATCOM AND (OS2 OR WIN32)))
        set_target_properties(GUIlib2
            PROPERTIES
                OUTPUT_NAME GUI-static
        )
    endif()
endif()

target_link_libraries(GUIlib2 PRIVATE $<BUILD_INTERFACE:${sdl2_target_name}>)

if(GUILIB2_INSTALL)
    if(MSVC)
        set(GUILIB2_INSTALL_EXPORTDIR "cmake")
    else()
        set(GUILIB2_INSTALL_EXPORTDIR "${CMAKE_INSTALL_INCLUDEDIR}/cmake/${PROJECT_NAME}")
    endif()

    install(TARGETS GUIlib2 EXPORT GUIlib2_exports
        ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    )
    install(FILES ${guilib2_headers} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/GUI")
    install(EXPORT GUIlib2_exports DESTINATION "${GUILIB2_INSTALL_EXPORTDIR}")
endif()

if(GUILIB2_SAMPLES)
    add_executable(okay okay.c)
    add_executable(hello hello.cpp)
    add_executable(hello_C hello_C.c)
    add_executable(keyboard keyboard.cpp)

    set(samples okay hello hello_C keyboard)
    foreach(sample IN LISTS samples)
        target_link_libraries(${sample} PRIVATE GUIlib2 ${sdl2_target_name})
        set_target_properties(${sample} PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}")
    endforeach()

    set(hello_images hello.bmp hello2.bmp scroll_up.bmp scroll_dn.bmp)

    foreach(hello hello hello_C)
        add_custom_command(TARGET ${hello} POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_if_different ${hello_images} "$<TARGET_FILE_DIR:${hello}>"
            WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
        )
    endforeach()
endif()

cmake_minimum_required(VERSION 3.18)
project(stb VERSION 1.0.0 LANGUAGES C)

# ---------- Header-only umbrella target ----------
add_library(stb INTERFACE)
add_library(stb::stb ALIAS stb)

target_include_directories(stb
        INTERFACE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)


# ---------- Optional: build stb_vorbis as a static lib ----------
option(STB_BUILD_VORBIS "Build stb_vorbis as a compiled library" OFF)

if(STB_BUILD_VORBIS)
    # vorbis .c can be at repo root or under src/
    set(_VORBIS_SRC "")
    foreach(_cand stb_vorbis.c src/stb_vorbis.c)
        if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/${_cand}")
            set(_VORBIS_SRC ${_cand})
            break()
        endif()
    endforeach()
    if(NOT _VORBIS_SRC)
        message(FATAL_ERROR "STB_BUILD_VORBIS=ON but stb_vorbis.c not found.")
    endif()

    add_library(stb_vorbis STATIC ${_VORBIS_SRC})
    add_library(stb::vorbis ALIAS stb_vorbis)

    target_include_directories(stb_vorbis
            PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
            $<INSTALL_INTERFACE:include>)

    target_compile_features(stb_vorbis PUBLIC c_std_99)
    # target_compile_definitions(stb_vorbis PUBLIC STB_VORBIS_NO_PUSHDATA_API=1)
endif()

# ---------- Install / package ----------
include(CMakePackageConfigHelpers)

set(_EXPORT_TARGETS stb)
if(TARGET stb_vorbis)
    list(APPEND _EXPORT_TARGETS stb_vorbis)
endif()

install(TARGETS ${_EXPORT_TARGETS}
        EXPORT stbTargets
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)

install(DIRECTORY include/ DESTINATION include)

write_basic_package_version_file(
        "${CMAKE_CURRENT_BINARY_DIR}/stbConfigVersion.cmake"
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY SameMajorVersion)

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/stbConfig.cmake"
        "@PACKAGE_INIT@\ninclude(\"\${CMAKE_CURRENT_LIST_DIR}/stbTargets.cmake\")\n")

install(EXPORT stbTargets
        NAMESPACE stb::
        FILE stbTargets.cmake
        DESTINATION lib/cmake/stb)

install(FILES
        "${CMAKE_CURRENT_BINARY_DIR}/stbConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/stbConfigVersion.cmake"
        DESTINATION lib/cmake/stb)

export(PACKAGE stb)

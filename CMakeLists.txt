cmake_minimum_required(VERSION 3.2)

# Build options
option(Android "Android" OFF)

option(Emscripten "Build as WASM" OFF)
option(VITA "Build for PS Vita" OFF)
option(Extern_Config "Do not embed config file" ON)
set(EMSCRIPTEN_PATH "CHANGE ME PLEASE!!!/emsdk")

project(OpenClaw)

#if(NOT CMAKE_BUILD_TYPE)
#  set(CMAKE_BUILD_TYPE Release)
#endif()

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2 -g")

# CMake compiled directories with dependencies
add_subdirectory(Box2D)
add_subdirectory(libwap)
add_subdirectory(./ThirdParty/Tinyxml)
if (WIN32)
    add_subdirectory(MidiProc)
endif (WIN32)

#if(Android)
#    add_subdirectory(./ThirdParty/SDL2-2.0.5)
#    include_directories(./ThirdParty/SDL2-2.0.5/include/)
#    link_directories(./ThirdParty/SDL2-2.0.5/libs/armeabi/)
#    add_subdirectory(./ThirdParty/SDL2_gfx-1.0.1)
#endif(Android)

# Directories with headers
if (VITA)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -idirafter ${CMAKE_CURRENT_SOURCE_DIR}/ThirdParty")
    include_directories(./Box2D/)
    include_directories(./libwap/)
    include_directories(./ThirdParty/Tinyxml/)
else()
    include_directories(./Box2D/)
    include_directories(./libwap/)
    include_directories(./ThirdParty/)
    include_directories(./ThirdParty/Tinyxml/)
endif()
if (Emscripten)
    include_directories("${EMSCRIPTEN_PATH}/upstream/emscripten/system/include/emscripten/")
endif (Emscripten)

# Directories with libs
link_directories(./libwap)
link_directories(./Box2D)
if (WIN32)
    link_directories(./ThirdParty/SDL2/lib/x86)
    link_directories(./Build_Release)
endif (WIN32)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ../Build_Release)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ../android/libs/armeabi-v7a)

# Cmake target definitions
if(Android)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DANDROID")
    add_library(openclaw SHARED "")
else (Android)
    add_executable(openclaw "")
endif(Android)

add_subdirectory(OpenClaw)

# Linker settings
list(APPEND TARGET_LIBS
    libwap
    tinyxml
    Box2D
)
if (Emscripten)
    if (Extern_Config)
        set(CONFIG_PRELOAD_FILE "")
    else (Extern_Config)
        set(CONFIG_PRELOAD_FILE "--preload-file ../Build_Release/config.xml@config.xml")
    endif (Extern_Config)
    set_target_properties(openclaw PROPERTIES LINK_FLAGS "-s FETCH -s WASM=1 -s BINARYEN_METHOD='native-wasm' -s EXPORTED_FUNCTIONS='[_main]' -s USE_SDL=2 -s USE_SDL_IMAGE=2 -s SDL2_IMAGE_FORMATS='[\"pcx\",\"png\",\"tga\"]' -s USE_SDL_TTF=2 -s USE_SDL_GFX=2 -s ASYNCIFY=1 -s TOTAL_MEMORY=268435456 ${CONFIG_PRELOAD_FILE} --preload-file ../Build_Release/CLAW.REZ@CLAW.REZ --preload-file ../Build_Release/ASSETS.ZIP@ASSETS.ZIP --preload-file ../Build_Release/console02.tga@console02.tga --preload-file ../Build_Release/clacon.ttf@clacon.ttf")
else (Emscripten)
    list(APPEND TARGET_LIBS
            SDL2
            SDL2_mixer
            SDL2_ttf
            SDL2_image
            SDL2_gfx
            )
    if (WIN32)
        list(APPEND TARGET_LIBS
                Rpcrt4
                )
    else (WIN32)
        # TODO: Is it really important???
        list(APPEND TARGET_LIBS
                stdc++
                m
                )
    endif (WIN32)
    if (Android)
        list(APPEND TARGET_LIBS
                GLESv2
                GLESv1_CM
                dl
                log
                android
                )
    endif (Android)
    if (VITA)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DVITA -Wno-psabi")
        set(TARGET_LIBS
                libwap
                tinyxml
                Box2D
                SDL2_mixer
                SDL2_ttf
                SDL2_image
                SDL2_gfx
                SDL2
                vitaGL
                vitashark
                mathneon
                vorbisfile
                vorbis
                ogg
                opusfile
                opus
                modplug
                xmp
                mpg123
                flac
                freetype
                harfbuzz
                webpmux
                webpdemux
                webp
                jpeg
                png
                bz2
                z
                m
                pthread
                SceAudioIn_stub
                SceMotion_stub
                SceHid_stub
                SceGxm_stub
                SceDisplay_stub
                SceCtrl_stub
                SceTouch_stub
                SceAudio_stub
                SceSysmodule_stub
                SceAppMgr_stub
                SceAppUtil_stub
                SceNet_stub
                SceNetCtl_stub
                ScePower_stub
                SceCommonDialog_stub
                SceShaccCg_stub
                SceShaccCgExt
                taihen_stub
                SceKernelDmacMgr_stub
                )
    endif (VITA)
endif (Emscripten)

target_link_libraries(openclaw ${TARGET_LIBS})

if (VITA)
    set_target_properties(openclaw PROPERTIES OUTPUT_NAME "openclaw_bin")
    include("${VITASDK}/share/vita.cmake" REQUIRED)
    vita_create_self(openclaw.self "../Build_Release/openclaw_bin")
    add_dependencies(openclaw_bin-velf openclaw)
    vita_create_vpk(openclaw.vpk "OPENCLAW1" openclaw.self
        VERSION "01.00"
        NAME "OpenClaw"
        FILE sce_sys/icon0.png sce_sys/icon0.png
        FILE sce_sys/livearea/contents/bg0.png sce_sys/livearea/contents/bg0.png
        FILE sce_sys/livearea/contents/startup.png sce_sys/livearea/contents/startup.png
        FILE sce_sys/livearea/contents/template.xml sce_sys/livearea/contents/template.xml
    )
endif (VITA)

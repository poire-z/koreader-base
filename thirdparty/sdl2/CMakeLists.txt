list(APPEND PATCH_FILES
    # Remove workarounds for standalone applications and add
    # a couple of actions to be used from the osx main menu.
    cocoa.patch
    # FIXME: koreader should really be using `libSDL2-2.0.so.0` instead.
    no_library_versioning.patch
)

list(APPEND CMAKE_ARGS
    -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}
    # Project options.
    -DSDL2_DISABLE_SDL2MAIN=ON
    -DSDL_AUDIO=OFF
    -DSDL_CMAKE_DEBUG_POSTFIX= # Remove 'd' suffix for the debug build library.
    -DSDL_SHARED=ON
    -DSDL_STATIC=OFF
    -DSDL_TEST=OFF
)

list(APPEND BUILD_CMD COMMAND ninja)

list(APPEND INSTALL_CMD COMMAND ${CMAKE_COMMAND} --install .)

append_shared_lib_install_commands(INSTALL_CMD SDL2)

external_project(
    DOWNLOAD URL a344eb827a03045c9b399e99af4af13d
    https://github.com/libsdl-org/SDL/releases/download/release-2.28.5/SDL2-2.28.5.tar.gz
    PATCH_FILES ${PATCH_FILES}
    CMAKE_ARGS ${CMAKE_ARGS}
    BUILD_COMMAND ${BUILD_CMD}
    INSTALL_COMMAND ${INSTALL_CMD}
)

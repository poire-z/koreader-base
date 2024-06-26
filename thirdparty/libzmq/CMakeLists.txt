list(APPEND PATCH_CMD COMMAND ${KO_PATCH}
    ${CMAKE_CURRENT_SOURCE_DIR}/android.patch
    ${CMAKE_CURRENT_SOURCE_DIR}/cmake_tweaks.patch
)

list(APPEND CMAKE_ARGS
    -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}
    # Project options.
    -DWITH_DOC=FALSE
    -DWITH_TWEETNACL=TRUE
    -DZMQ_BUILD_FRAMEWORK=FALSE
    -DZMQ_BUILD_TESTS=FALSE
)
if(LEGACY OR POCKETBOOK)
    list(APPEND CMAKE_ARGS -DENABLE_EVENTFD=FALSE)
endif()
if(NOT EMULATE_READER)
    list(APPEND CMAKE_ARGS
        -DZMQ_HAVE_SOCK_CLOEXEC_EXITCODE=1
        -DZMQ_HAVE_SOCK_CLOEXEC_EXITCODE__TRYRUN_OUTPUT=
        -DZMQ_HAVE_SO_KEEPALIVE_EXITCODE=1
        -DZMQ_HAVE_SO_KEEPALIVE_EXITCODE__TRYRUN_OUTPUT=
        -DZMQ_HAVE_TCP_KEEPCNT_EXITCODE=1
        -DZMQ_HAVE_TCP_KEEPCNT_EXITCODE__TRYRUN_OUTPUT=
        -DZMQ_HAVE_TCP_KEEPIDLE_EXITCODE=1
        -DZMQ_HAVE_TCP_KEEPIDLE_EXITCODE__TRYRUN_OUTPUT=
        -DZMQ_HAVE_TCP_KEEPINTVL_EXITCODE=1
        -DZMQ_HAVE_TCP_KEEPINTVL_EXITCODE__TRYRUN_OUTPUT=
    )
endif()

list(APPEND BUILD_CMD COMMAND ${KO_MAKE_PROGRAM})

list(APPEND INSTALL_CMD COMMAND ${CMAKE_COMMAND} --install .)

append_shared_lib_install_commands(INSTALL_CMD zmq VERSION 4)

ko_write_gitclone_script(
    GIT_CLONE_SCRIPT_FILENAME
    https://github.com/zeromq/libzmq
    883e95b22e0bffffa72312ea1fec76199afbe458
    ${SOURCE_DIR}
)

external_project(
    DOWNLOAD_COMMAND ${CMAKE_COMMAND} -P ${GIT_CLONE_SCRIPT_FILENAME}
    PATCH_COMMAND ${PATCH_CMD}
    CMAKE_ARGS ${CMAKE_ARGS}
    BUILD_COMMAND ${BUILD_CMD}
    INSTALL_COMMAND ${INSTALL_CMD}
)

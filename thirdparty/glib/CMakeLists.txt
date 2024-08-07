list(APPEND PATCH_FILES
    # We only care about `libglib-2.0.a`.
    glib-libglib-only.patch
    # Android makes Linux syscalls hard to get to... Fix pilfered from Python.
    glib-2.58.2-android-linux-syscalls.patch
)

list(APPEND PATCH_CMD COMMAND autoreconf -vif)

# Autotools needs some hints when x-compiling
if(CHOST)
    list(APPEND gen_cache_config glib_cv_stack_grows=no)
    list(APPEND gen_cache_config glib_cv_uscore=no)
    # In order to support legacy PocketBook 4.x firmware we should get rid of eventfd@GLIBC_2.7 and pipe2@GLIBC_2.9
    if(LEGACY OR POCKETBOOK)
        list(APPEND gen_cache_config glib_cv_eventfd=no)
        list(APPEND gen_cache_config ac_cv_func_pipe2=no)
        # As well as mkostemp@GLIBC_2.7
        list(APPEND gen_cache_config ac_cv_func_mkostemp=no)
    endif()
    if(ANDROID)
        # disable getpwuid, getgrgid and timegm for > android-12
        list(APPEND gen_cache_config ac_cv_func_posix_getpwuid_r=no)
        list(APPEND gen_cache_config ac_cv_func_posix_getgrgid_r=no)
        list(APPEND gen_cache_config ac_cv_func_getpwuid_r=no)
        list(APPEND gen_cache_config ac_cv_func_getgrgid_r=no)
        list(APPEND gen_cache_config ac_cv_func_timegm=no)
    endif()
endif()

list(APPEND CFG_CMD COMMAND sh -c "printf '%s\\n' \"$@\" >cache.conf" -- ${gen_cache_config})

if(ANDROID OR DARWIN)
    set(WITH_LIBICONV gnu)
else()
    set(WITH_LIBICONV no)
endif()

string(APPEND CPPFLAGS " -I${STAGING_DIR}/include")
string(APPEND LDFLAGS " -L${STAGING_DIR}/lib")

list(APPEND CFG_CMD COMMAND env)
append_autotools_vars(CFG_CMD)
list(APPEND CFG_CMD
    ${SOURCE_DIR}/configure --host=${CHOST} --cache-file=cache.conf --prefix=/
    --enable-static=true --enable-shared=false
    --disable-compile-warnings
    --disable-libelf
    --disable-selinux
    --enable-libmount=no
    --with-libiconv=${WITH_LIBICONV}
    --with-pcre=internal
    --with-threads=posix
)

list(APPEND BUILD_CMD COMMAND make -C glib)

list(APPEND INSTALL_CMD COMMAND make -C glib DESTDIR=${STAGING_DIR} install)

external_project(
    DOWNLOAD URL 8058c7bde846dcffe5fa453eca366d73
    https://download.gnome.org/sources/glib/2.58/glib-2.58.3.tar.xz
    PATCH_FILES ${PATCH_FILES}
    PATCH_COMMAND ${PATCH_CMD}
    CONFIGURE_COMMAND ${CFG_CMD}
    BUILD_COMMAND ${BUILD_CMD}
    INSTALL_COMMAND ${INSTALL_CMD}
)

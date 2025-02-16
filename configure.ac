AC_PREREQ(2.60)

m4_define(axa_major_version, 2)
m4_define(axa_minor_version, 2)
m4_define(axa_patchlevel_version, 1)
m4_define(axa_version,
          axa_major_version.axa_minor_version.axa_patchlevel_version)

AC_INIT([axa],
	[axa_version()],
	[https://github.com/farsightsec/axa/issues],
	[axa],
	[https://github.com/farsightsec/axa])
PACKAGE_DESCRIPTION="Farsight Security Advanced Exchange Access (AXA)"
AC_SUBST(PACKAGE_DESCRIPTION)
AC_CONFIG_SRCDIR([axa/axa.h])
AC_CONFIG_AUX_DIR([build-aux])
AM_INIT_AUTOMAKE([foreign 1.11 -Wall -Wno-portability silent-rules subdir-objects])
AC_PROG_CC_STDC
AC_USE_SYSTEM_EXTENSIONS
AC_SYS_LARGEFILE
AC_CONFIG_MACRO_DIR([m4])
AM_SILENT_RULES([yes])
LT_INIT

AXA_MAJOR_VERSION=axa_major_version()
AXA_MINOR_VERSION=axa_minor_version()
AXA_PATCHLEVEL_VERSION=axa_patchlevel_version()
AXA_VERSION=axa_version()

AC_SUBST(AXA_MAJOR_VERSION)
AC_SUBST(AXA_MINOR_VERSION)
AC_SUBST(AXA_PATCHLEVEL_VERSION)
AC_SUBST(AXA_VERSION)

my_CFLAGS=" \
-Wextra -Wall -Wbad-function-cast -Wcast-align -Wmissing-noreturn \
-Wnested-externs -Wpointer-arith -Wshadow -Wwrite-strings \
-Wdeclaration-after-statement -Wswitch-enum \
-Wunused \
-Wunreachable-code \
-Wuninitialized \
-Wno-unknown-pragmas \
-Wmissing-declarations -Wmissing-prototypes \
-Wsign-compare -Wchar-subscripts \
-Wstrict-prototypes -Wshadow \
-Wformat-security \
-Wno-unused-command-line-argument"
AC_SUBST([my_CFLAGS])

AC_CHECK_FUNCS([floor gettimeofday])

AC_CONFIG_HEADERS(config.h)

PKG_PROG_PKG_CONFIG
if test -n "$PKG_CONFIG"; then
    if $PKG_CONFIG --variable=pc_path pkg-config 2>/dev/null | grep -q /libdata/; then
        PKG_INSTALLDIR(['${prefix}/libdata/pkgconfig'])
    else
        PKG_INSTALLDIR
    fi
fi

LT_LIB_M

MY_CHECK_LIBPCAP

MY_CHECK_LIBPROTOBUF_C

save_LIBS="$LIBS"
AC_SEARCH_LIBS([strlcpy],
               [bsd],
               [strlcpy_LIBS="$LIBS"],
               [AC_MSG_ERROR([could not find libbsd, it can be downloaded here: http://libbsd.freedesktop.org/wiki/])]
              )
LIBS="$save_LIBS"
AC_SUBST(strlcpy_LIBS)

PKG_CHECK_MODULES([libedit], [libedit], [], [MY_CHECK_LIBEDIT])

save_CPPFLAGS="$CPPFLAGS"
CPPFLAGS="$libedit_CFLAGS"
AC_MSG_CHECKING(libedit has unicode) 
AC_CHECK_TYPES([el_rfunc_t],
		AC_DEFINE([LIBEDIT_IS_UNICODE], [1],
			[Define if libedit has unicode support.]),
		[], [[#include <histedit.h>]])
CPPFLAGS="$save_CPPFLAGS"

PKG_CHECK_MODULES([libnmsg], [libnmsg >= 0.9.1])

#AC_CHECK_FUNCS([nmsg_get_version])
#AC_CHECK_FUNCS([wdns_get_version])

PKG_CHECK_MODULES([libwdns], [libwdns])

PKG_CHECK_MODULES([libssl], [libssl])

PKG_CHECK_MODULES([libcrypto], [libcrypto])

AC_CHECK_HEADER([zlib.h], [], [ AC_MSG_ERROR([required header file not found]) ])
AC_CHECK_LIB([z], [deflate], [], [ AC_MSG_ERROR([required library not found]) ])
AC_CHECK_LIB([lmdb], [mdb_env_open], [], [ AC_MSG_ERROR([required library not found]) ])

AC_ARG_WITH([yajl], AS_HELP_STRING([--without-yajl], [Disable yajl support]))
if test "x$with_yajl" != "xno"; then
	PKG_CHECK_MODULES([yajl], [yajl >= 2.1.0])
	AC_DEFINE([HAVE_YAJL], [1], [Define to 1 if yajl support is enabled.])
	use_yajl="true"
else
	use_yajl="false"
fi
AM_CONDITIONAL([HAVE_YAJL], [test "$use_yajl" = "true"])

PKG_CHECK_MODULES([check], [check >= 0.9.10],
		[CHECK_MOD_MSG="yes"], [CHECK_MOD_MSG="no"])

AC_PATH_PROG([DOXYGEN], [doxygen])
AM_CONDITIONAL([HAVE_DOXYGEN], [test -n "$DOXYGEN"])
if test -n "$DOXYGEN"; then
        DOC_HTML_MSG="yes, doxygen available: $DOXYGEN"
        AC_MSG_CHECKING(doxygen version)
	installed_version=`$DOXYGEN --version 2>/dev/null`
	AS_VERSION_COMPARE($installed_version, "1.8.3",
			   DOXYGEN_version="oldish",
			   DOXYGEN_version="newish",
			   DOXYGEN_version="newish")
        AC_MSG_RESULT($DOXYGEN_version)
        AC_CONFIG_FILES([Doxyfile])
        DOXYGEN_INPUT="${srcdir}/axa ${srcdir}/README.md"
        AC_SUBST(DOXYGEN_INPUT)
        if test $DOXYGEN_version = "oldish"; then
            AC_MSG_WARN(old Doxygen version found API documentation may not render as intended consider upgrading to version 1.8.3 or newer)
        fi
else
        DOC_HTML_MSG="no, doxygen not available"
fi

save_CPPFLAGS="$CPPFLAGS"
CPPFLAGS="$libnmsg_CFLAGS $libprotobuf_c_CFLAGS"
AC_CHECK_HEADERS([nmsg/base/dnsqr.pb-c.h nmsg/base/encode.pb-c.h nmsg/base/packet.pb-c.h nmsg/sie/newdomain.pb-c.h],
                 [],
                 [AC_MSG_ERROR([required header file not found])])
CPPFLAGS="$save_CPPFLAGS"

AX_DEFINE_DIR([CONFDIR], [sysconfdir/axa], [config dir for axa])
AC_DEFINE_UNQUOTED(AXACONFDIR,"$CONFDIR",[config dir for axa])

AC_CONFIG_FILES([Makefile axa/libaxa.pc axa/version.h])

AC_CONFIG_FILES([sratool/sratool.man sratool/radtool.man sratool/options.man
		sratunnel/sratunnel.man sratunnel/radtunnel.man
                tools/axa_link_certs.man tools/axa_make_cert.man
		tools/axa_server_cert.man tools/axa_tsindextool.man])

AC_CONFIG_FILES([tools/axa_link_certs],
		[chmod +x tools/axa_link_certs])
AC_CONFIG_FILES([tools/axa_make_cert],
		[chmod +x tools/axa_make_cert])
AC_CONFIG_FILES([tools/axa_server_cert],
		[chmod +x tools/axa_server_cert])

AC_CONFIG_FILES([tests/test-tools-without-server.sh],
		[chmod +x tests/test-tools-without-server.sh])

AC_CONFIG_FILES([tests/test-config],
		[chmod 600 tests/test-config])

AC_CONFIG_FILES([tests/test-config-badperms],
		[chmod 640 tests/test-config-badperms])

AC_OUTPUT
AC_MSG_RESULT([
    $PACKAGE $VERSION

        compiler:               ${CC}
        cflags:                 ${CFLAGS}
        ldflags:                ${LDFLAGS}
        libs:                   ${LIBS}

        prefix:                 ${prefix}
        sysconfdir:             ${sysconfdir}
        libdir:                 ${libdir}
        includedir:             ${includedir}
        pkgconfigdir:           ${pkgconfigdir}

        building html docs:     ${DOC_HTML_MSG}

	yajl support:           ${use_yajl}
	make check available:   ${CHECK_MOD_MSG}
])

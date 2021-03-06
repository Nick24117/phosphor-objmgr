# Initialization
AC_PREREQ([2.69])
AC_INIT([phosphor-objmgr], [1.0], [https://github.com/openbmc/phosphor-objmgr/issues])
AC_CONFIG_HEADERS([libmapper/config.h])
AC_CONFIG_MACRO_DIR([m4])
AM_INIT_AUTOMAKE([subdir-objects -Wall -Werror foreign dist-xz])
AM_SILENT_RULES([yes])
AC_LANG([C++])

# Modify header template
AH_TOP([#pragma once])
AH_BOTTOM([#include "libmapper/workaround/systemd.h"])

# Checks for programs.
AC_PROG_CC
AC_PROG_CXX
AM_PROG_AR
AC_PROG_INSTALL
AC_PROG_MAKE_SET
AC_PROG_AWK
# Python
AM_PATH_PYTHON([2.7], [AC_SUBST([PYTHON], [echo "$PYTHON"])], [AC_MSG_ERROR([Could not find python-2.7 installed...python-2.7 is required])])
AM_CONDITIONAL([HAVE_PYTHON], [test "$PYTHON" != :])
AC_SUBST([HAVE_PYTHON])
AC_SUBST([PYTHONDIR], ${pythondir})

# Checks for libraries.
PKG_CHECK_MODULES([SDBUSPLUS], [sdbusplus],, [AC_MSG_ERROR([Could not find sdbusplus...openbmc/sdbusplus package required])])
PKG_CHECK_MODULES([TINYXML2], [tinyxml2],, AC_MSG_ERROR(["Requires tinyxml2."]))
PKG_CHECK_MODULES([SYSTEMD], [libsystemd >= 221])
PKG_CHECK_MODULES([PHOSPHOR_LOGGING], [phosphor-logging],, AC_MSG_ERROR(["Requires phosphor-logging."]))

# Checks for header files.
AC_CHECK_HEADER(systemd/sd-bus.h, ,[AC_MSG_ERROR([Could not find systemd/sd-bus.h...systemd development package required])])

# Checks for typedefs, structures, and compiler characteristics.
AX_CXX_COMPILE_STDCXX_17([noext])
AX_APPEND_COMPILE_FLAGS([-fpic -Wall -Werror], [CFLAGS CXXFLAGS])

# Checks for boost by the headers used.
AC_CHECK_HEADER(boost/algorithm/string/predicate.hpp, ,[AC_MSG_ERROR([Could not find boost/algorithm/string/predicate.hpp])])
AC_CHECK_HEADER(boost/container/flat_map.hpp, ,[AC_MSG_ERROR([Could not find boost/container/flat_map.hpp])])
AC_CHECK_HEADER(boost/container/flat_set.hpp, ,[AC_MSG_ERROR([Could not find boost/container/flat_set.hpp])])

# Checks for library functions.
LT_INIT([disable-static shared])

# Check/set gtest specific functions.
AX_PTHREAD([GTEST_CPPFLAGS="-DGTEST_HAS_PTHREAD=1"],[GTEST_CPPFLAGS="-DGTEST_HAS_PTHREAD=0"])
AC_SUBST(GTEST_CPPFLAGS)

AC_ARG_ENABLE([oe-sdk],`
    AS_HELP_STRING([--enable-oe-sdk], [Link testcases absolutely against OE SDK so they can be ran within it.])
)
AC_ARG_VAR(OECORE_TARGET_SYSROOT,
    [Path to the OE SDK SYSROOT])
AS_IF([test "x$enable_oe_sdk" == "xyes"],
    AS_IF([test "x$OECORE_TARGET_SYSROOT" == "x"],
          AC_MSG_ERROR([OECORE_TARGET_SYSROOT must be set with --enable-oe-sdk])
    )
    AC_MSG_NOTICE([Enabling OE-SDK at $OECORE_TARGET_SYSROOT])
    [
        testcase_flags="-Wl,-rpath,\${OECORE_TARGET_SYSROOT}/lib"
        testcase_flags="${testcase_flags} -Wl,-rpath,\${OECORE_TARGET_SYSROOT}/usr/lib"
        testcase_flags="${testcase_flags} -Wl,-dynamic-linker,`find \${OECORE_TARGET_SYSROOT}/lib/ld-*.so | sort -r -n | head -n1`"
    ]
    AC_SUBST([OESDK_TESTCASE_FLAGS], [$testcase_flags])
)

# Add defines to config header
AC_DEFINE([MAPPER_BUSNAME], ["xyz.openbmc_project.ObjectMapper"], [Object mapper bus name])
AC_DEFINE([MAPPER_INTERFACE], ["xyz.openbmc_project.ObjectMapper"], [Object mapper interface])
AC_DEFINE([MAPPER_PATH], ["/xyz/openbmc_project/object_mapper"], [Object mapper DBUS path])

# Workaround for unpatched systemd installs
AC_ARG_ENABLE([unpatched-systemd],
    AS_HELP_STRING([--enable-unpatched-systemd], [Enable unpatched systemd workarounds]))
AS_IF([test "x$enable_unpatched_systemd" == "xyes"],
    AC_MSG_NOTICE([Enabling unpatched systemd workarounds])
    [
        AC_DEFINE([IS_UNPATCHED_SYSTEMD], ,[Enable unpatched systemd workarounds])
    ]
)

# Create configured output
AC_CONFIG_FILES([Makefile setup.py fail-monitor/Makefile])
AC_OUTPUT

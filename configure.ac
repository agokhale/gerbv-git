dnl configuration script for gerbv
dnl
dnl  gEDA - GNU Electronic Design Automation
dnl  This file is part of gerbv.
dnl
dnl    Copyright (C) 2000-2001 Stefan Petersen (spe@stacken.kth.se)
dnl
dnl  $Id$
dnl
dnl  This program is free software; you can redistribute it and/or modify
dnl  it under the terms of the GNU General Public License as published by
dnl  the Free Software Foundation; either version 2 of the License, or
dnl  (at your option) any later version.
dnl
dnl  This program is distributed in the hope that it will be useful,
dnl  but WITHOUT ANY WARRANTY; without even the implied warranty of
dnl  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
dnl  GNU General Public License for more details.
dnl
dnl  You should have received a copy of the GNU General Public License
dnl  along with this program; if not, write to the Free Software
dnl  Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA 02111 USA


AC_INIT([gerbv], [2.0.0-beta])
AC_CONFIG_SRCDIR([src/gerbv.c])
AC_PREREQ([2.59])
AM_INIT_AUTOMAKE([1.9])

dnl Create a configuration header 
AM_CONFIG_HEADER([config.h])

dnl Initialize maintainer mode
AM_MAINTAINER_MODE

############################################################
#
# Checks for cygwin/mingw32
#

AC_CANONICAL_HOST


# if you want -mno-cygwin, then add it to MINGW_CFLAGS or CYGWIN_CFLAGS
# in your configure environment.  After all you may or may not
# want to always force -mno-cygwin on all users.
case $host_os in
	*cygwin*  )
		CFLAGS="$CFLAGS ${CYGWIN_CFLAGS}"
                CPPFLAGS="$CPPFLAGS ${CYGWIN_CPPFLAGS}"
		;;

	*mingw32* )
		CFLAGS="$CFLAGS ${MINGW_CFLAGS:--mms-bitfields -mwindows}"
		CPPFLAGS="$CPPFLAGS ${MINGW_CPPFLAGS:--mms-bitfields -mwindows}"
		;;
esac


#
#
############################################################

############################################################
# 
# Checks for our configure args
#

dnl --enable-debug
debug=0
AC_ARG_ENABLE(debug,
    [  --enable-debug          Enable fairly verbose debug output],
    [
        if test $enableval = "yes"; then
		debug=1
	fi
    ],
    [
	debug=0
    ])
AC_DEFINE_UNQUOTED(DEBUG, $debug, [Define to 1 to enable debugging code])

dnl --enable-exportpng
AC_ARG_ENABLE(exportpng,
    [  --disable-exportpng     Disable PNG export.],
    [
        if test $enableval = "yes"; then
		exportpng=yes
	  	AC_DEFINE(EXPORT_PNG,,[Support for export of PNG's])
	fi
    ],
    [
	exportpng=yes
  	AC_DEFINE(EXPORT_PNG,,[Support for export of PNG's])
    ])

dnl --enable-unit-mm : Set default unit for coordinates in status bar to mm
AC_ARG_ENABLE(unit-mm,
    [  --enable-unit-mm        Set default unit for coordinates in status bar to mm],
    [
        if test "$enableval" = "yes"; then
		default_unit="mm"
	fi
    ])
if test "$default_unit" = "mm"; then
	AC_DEFINE(GERBV_DEFAULT_UNIT, GERBV_MMS, [Default unit to display in statusbar])
else
	AC_DEFINE(GERBV_DEFAULT_UNIT, GERBV_MILS, [Default unit to display in statusbar])
fi

dnl --disable-cairo
use_cairo=yes
AC_ARG_ENABLE(cairo,
    [  --disable-cairo         Disable the use of cairo for rendering],
    [
        if test "$enableval" = "yes"; then
		use_cairo=yes
	else
		use_cairo=no
	fi
    ],
    [
	use_cairo=yes
    ])

AM_CONDITIONAL(USE_CAIRO, test x$use_cairo = xyes)
if test "$use_cairo" = "yes"; then
	RENDERER=cairo
else
	RENDERER=gdk
	AC_DEFINE(RENDER_USING_GDK, 1, [Define to use gdk instead of cairo for rendering])
fi

#
#
############################################################


############################################################
#
# Preliminary checks
#

dnl Build time sanity check... (?)
AM_SANITY_CHECK

dnl Checks for programs.
AC_PROG_CC
AC_PROG_MAKE_SET
AC_PROG_INSTALL

# if we have gcc then add -Wall
if test "x$GCC" = "xyes"; then
	if test -z "`echo "$CFLAGS" | grep "\-Wall" 2> /dev/null`" ; then
		CFLAGS="$CFLAGS -Wall"
	fi
fi

# Check for ImageMagick tools used by the testsuite
AC_PATH_PROG(ANIMATE, animate, notfound)
AC_PATH_PROG(COMPARE, compare, notfound)
AC_PATH_PROG(COMPOSITE, composite, notfound)
AC_PATH_PROG(CONVERT, convert, notfound)
AC_PATH_PROG(DISPLAY, display, notfound)
AC_PATH_PROG(MONTAGE, montage, notfound)
missing_magick=""
test "${ANIMATE}" != "notfound" || missing_magick="${missing_magick} animate"
test "${COMPARE}" != "notfound" || missing_magick="${missing_magick} compare"
test "${COMPOSITE}" != "notfound" || missing_magick="${missing_magick} composite"
test "${CONVERT}" != "notfound" || missing_magick="${missing_magick} convert"
test "${DISPLAY}" != "notfound" || missing_magick="${missing_magick} display"
test "${MONTAGE}" != "notfound" || missing_magick="${missing_magick} montage"

AC_MSG_CHECKING([if all ImageMagick tools needed for the testsuite were found])
if test "X${missing_magick}" != "X" ; then
	AC_MSG_RESULT([no.  The testsuite will be disabled because the following
tools from the ImageMagick suite were not found:
${missing_magick}
No loss in gerbv functionality should be experienced, you just will not
be able to run the regression testsuite.
])
	have_magick=no
else
	AC_MSG_RESULT([yes])
	have_magick=yes
fi
AM_CONDITIONAL(HAVE_MAGICK, test x$have_magick = xyes)

# Check for pkg-config
AC_PATH_PROG(PKG_CONFIG, pkg-config, no)
if test "$PKG_CONFIG" = "no"; then
	AC_MSG_ERROR([Cannot find pkg-config, make sure it is installed and in your PATH])
fi      

#
#
#
############################################################

############################################################
#
# Library checks
#

AC_CHECK_LIB(m, sin)

#
#
############################################################

############################################################
#
# GTK and cairo checks
#

 
if test "$use_cairo" = "yes"; then

	PKG_CHECK_MODULES(CAIRO, cairo >= 1.2.0, , [AC_MSG_ERROR([
*** Cairo > 1.2.0 required but not found.  Please install first or if
you wish to use the old GDK rendering (with possible loss of some 
functionality) then use the --disable-cairo argument to configure  ***
Please review the following errors:
$CAIRO_PKG_ERRORS])]
)

	CAIRO_VER=`$PKG_CONFIG cairo --modversion`
else
	CAIRO_VER="N/A"
fi

PKG_CHECK_MODULES(GTK, gtk+-2.0 >= 2.4.0, , [AC_MSG_ERROR([
*** GTK >= 2.4.0 is required but was not found.  Please review
the following errors:
$GTK_PKG_ERRORS])]
)

GTK_VER=`$PKG_CONFIG gtk+-2.0 --modversion`

#
#
############################################################


############################################################
#
# libpng checks
#

if test "$exportpng" = "yes" ; then
	AC_PATH_PROG([LIBPNG_CONFIG], [libpng-config], [notfound])
	if test "$LIBPNG_CONFIG" = "notfound" ; then
		AC_ERROR([You have requested png export but libpng-config could not be found.  You may
		build $PACKAGE without png support by using --disable-exportpng])
	fi

	AC_MSG_CHECKING([for libpng cflags])
	PNG_CFLAGS="`$LIBPNG_CONFIG --cflags`"
	AC_MSG_RESULT([$PNG_CFLAGS])

	AC_MSG_CHECKING([for libpng libs])
	PNG_LIBS="`$LIBPNG_CONFIG --ldflags`"
	AC_MSG_RESULT([$PNG_LIBS])
dnl	AC_CHECK_LIB($PNG_LIBS, png_write_info,, 
dnl	AC_MSG_ERROR(*** Must have libpng to get export-png to work))
else
	GDK_PIXBUF_CFLAGS=
	GDK_PIXBUF_LIBS=
	PNG_CFLAGS=
	PNG_LIBS=
fi

#
#
############################################################

############################################################
#
# ElectricFence
with_efence=no
AC_MSG_CHECKING([if ElectricFence debugging should be enabled])
AC_ARG_ENABLE([efence], 
[  --enable-efence         Link with ElectricFence for malloc debugging [default=no]],
[
if test "X$enable_efence" != "Xno" ; then
	AC_MSG_RESULT([yes])
	AC_CHECK_LIB(efence,main,,
	AC_MSG_ERROR([You have requested ElectricFence debugging but -lefence could not be found]))
		with_efence=yes
else    
	AC_MSG_RESULT([no])
fi      
],
[
AC_MSG_RESULT([no])
])

#
#
############################################################

AC_CHECK_HEADERS(unistd.h getopt.h string.h sys/mman.h sys/types.h sys/stat.h stdlib.h regex.h libgen.h)

AC_CHECK_FUNCS(getopt_long)


CFLAGS="$CFLAGS $GDK_PIXBUF_CFLAGS $PNG_CFLAGS $GTK_CFLAGS $CAIRO_CFLAGS $WIN32_CFLAGS"
LIBS="$LIBS $GDK_PIXBUF_LIBS $PNG_LIBS $GTK_LIBS $CAIRO_LIBS"


############################################################
#
# scheme configure stuff
#

AC_DEFINE([STANDALONE], [0],[Scheme interpreter not used standalone])
AC_DEFINE([USE_DL],[1],[Dynamic linking in Scheme interpreter])
AC_DEFINE([SUN_DL],[1],[DL Sun method])
AC_DEFINE([USE_MATH],[1],[Math in Scheme interpreter])
AC_DEFINE([USE_ASCII_NAMES],[1],[ASCII names in Scheme interpreter])
AC_DEFINE([USE_COLON_HOOKS],[1],[Colon Hooks in Scheme interpreter])
AC_DEFINE([USE_STRING_HOOKS],[1],[String Hooks in Scheme interpreter])
AC_DEFINE([USE_INTERFACE],[1],[Use extension interface of Scheme interpreter])

#
#
############################################################

AC_OUTPUT(	Makefile \
		desktop/Makefile \
		doc/PNG-print/Makefile \
		doc/eagle/Makefile \
		doc/html/Makefile \
		doc/Makefile \
		example/eaglecad1/Makefile \
		example/nollezappare/Makefile \
		example/numpres/Makefile \
		example/jj/Makefile \
		example/dan/Makefile \
		example/ekf2/Makefile \
		example/exposure/Makefile \
		example/am-test/Makefile \
		example/cslk/Makefile \
		example/orcad/Makefile \
		example/Mentor-BoardStation/Makefile \
		example/pick-and-place/Makefile \
		example/polarity/Makefile \
		example/thermal/Makefile \
		example/trailing/Makefile \
		example/Makefile \
		man/Makefile \
		scheme/Makefile \
		src/Makefile \
		test/Makefile \
		test/golden/Makefile \
		test/inputs/Makefile )
        
AC_MSG_RESULT([
** Configuration summary for $PACKAGE $VERSION:

   PREFIX:                   $PREFIX

   CPPFLAGS:                 $CPPFLAGS
   CFLAGS:                   $CFLAGS
   LDFLAGS:                  $LDFLAGS
   LIBS:                     $LIBS

   GTK Version:              $GTK_VER
   Cairo Version:            $CAIRO_VER

   Rendering engine:         $RENDERER

   Electric Fence Debugging: $with_efence

])

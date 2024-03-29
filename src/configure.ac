AC_INIT
AC_CONFIG_SRCDIR([src/rateup.c])
AC_PREREQ([2.69])

dnl Check for programs.
AC_PROG_CC
AC_PREFIX_DEFAULT( /opt/mrtg-2 )

AC_PROG_CPP
AC_PROG_MAKE_SET
AC_PROG_INSTALL
AC_PATH_PROG(PERL, perl, no, $PATH:/usr/bin:/usr/local/bin)
if test $PERL = no; then 
   echo
   echo "** No Perl found in the PATH. A recent copy of Perl"
   echo "   is required for mrtg to work. Check www.perl.com"
   echo 
   exit 1
fi

AC_PATH_PROGS(NROFF, groff nroff)

dnl Check for long long type (64 bit rateup)
AC_CHECK_HEADERS(inttypes.h)
AC_CHECK_TYPE(unsigned long long)
AC_CHECK_TYPE(long long)
AC_CHECK_FUNCS(strtoll)

# Taken from libIDL-0.8.2
AC_MSG_CHECKING([for printf long long format specifier])
AC_CACHE_VAL(mr_cv_long_long_format_specifier,[
    for format in ll l q I64; do
        AC_RUN_IFELSE([AC_LANG_SOURCE([[#include <stdio.h>  
            int main()
            {
                long long b, a = -0x3AFAFAFAFAFAFAFALL;
                char buffer[1000];
                sprintf (buffer, "%${format}u", a);
                sscanf (buffer, "%${format}u", &b);
                return b!=a;
            }
            ]])],[mr_cv_long_long_format_specifier="%${format}d"
            mr_cv_long_long_format="${format}d"
            break],[],[])
        done])
AC_MSG_RESULT($mr_cv_long_long_format_specifier)
AC_DEFINE_UNQUOTED(LLD, "$mr_cv_long_long_format_specifier",
    [How to print a long long])
AC_DEFINE_UNQUOTED(LLD_FORMAT, "$mr_cv_long_long_format",
    [long long format without % specifier])

AH_BOTTOM(
#ifndef HAVE_STRTOLL     
long long int strtoll(const char *str, char **ptr, int base)
{
        long long int ll;
        sscanf(str, LLD, &ll);
        return ll;
}
#endif
)

dnl Checks for libraries.
AC_CHECK_LIB(m, pow, [ MATHLIBS="-lm" ])

AC_ARG_WITH(gd,[  --with-gd=DIR           location of the gd lib/inc],
		[LDFLAGS="${LDFLAGS} -L${withval}"
		CPPFLAGS="${CPPFLAGS} -I${withval}"
		LD_RUN_PATH="${withval}${LD_RUN_PATH:+:}${LD_RUN_PATH}"])

AC_ARG_WITH(gd-lib,[  --with-gd-lib=DIR       location of the gd library],
		[LDFLAGS="${LDFLAGS} -L${withval}"
		LD_RUN_PATH="${withval}${LD_RUN_PATH:+:}${LD_RUN_PATH}"])

AC_ARG_WITH(gd-inc,[  --with-gd-inc=DIR       location of the gd include files],
		[CPPFLAGS="${CPPFLAGS} -I${withval}"])

AC_ARG_WITH(z,[  --with-z=DIR            location of zlib lib/inc],
		[LDFLAGS="${LDFLAGS} -L${withval}"
		CPPFLAGS="${CPPFLAGS} -I${withval}"
		LD_RUN_PATH="${withval}${LD_RUN_PATH:+:}${LD_RUN_PATH}"])

AC_ARG_WITH(z-lib,[  --with-z-lib=DIR        location of zlib library],
		[LDFLAGS="${LDFLAGS} -L${withval}"
		LD_RUN_PATH="${withval}${LD_RUN_PATH:+:}${LD_RUN_PATH}"])

AC_ARG_WITH(z-inc,[  --with-z-inc=DIR        location of the zlib include files],
		[CPPFLAGS="${CPPFLAGS} -I${withval}"])

AC_ARG_WITH(png,[  --with-png=DIR          location of png lib/inc],
		[LDFLAGS="${LDFLAGS} -L${withval}"
		CPPFLAGS="${CPPFLAGS} -I${withval}"
		LD_RUN_PATH="${withval}${LD_RUN_PATH:+:}${LD_RUN_PATH}"])

AC_ARG_WITH(png-lib,[  --with-png-lib=DIR      location of png library],
		[LDFLAGS="${LDFLAGS} -L${withval}"
		LD_RUN_PATH="${withval}${LD_RUN_PATH:+:}${LD_RUN_PATH}"])

AC_ARG_WITH(png-inc,[  --with-png-inc=DIR      location of the libpng include files],
		[CPPFLAGS="${CPPFLAGS} -I${withval}"])


LIBS="${LIBS} ${MATHLIBS}"

GDEXTRALIB=""
dnl which version of gd do we have here ?
AC_CHECK_LIB(gd,gdImageGif,[
  GDFORM_EXT=gif
  GLIBS="-lgd"
                           ],:,[-lgd ${MATHLIBS}])

AC_CHECK_LIB(gd,gdImagePng,[
   GDFORM_EXT=png
   GLIBS="-lgd -lpng -lz"
        ],:,[-lgd -lpng -lz ${MATHLIBS}])

AC_CHECK_LIB(gd,gdImagePng_jpg,[
   GDFORM_EXT=png
   GLIBS="-lgd -lpng -lz -ljpeg"
        ],:,[-lgd -lpng -lz -ljpeg ${MATHLIBS}])

AC_CHECK_LIB(gd,gdImagePng_jpg_ft,[
   GDFORM_EXT=png
   GLIBS="-lgd -lpng -lz -ljpeg -lfreetype"
        ],:,[-lgd -lpng -lz -ljpeg -lfreetype ${MATHLIBS}])

if test x$GDFORM_EXT = xgif; then
  CPPFLAGS="$CPPFLAGS -DGFORM_GD=gdImageGif"
fi

LIBS="${GLIBS} ${LIBS}" 

if test x$LD_RUN_PATH != x; then
	SET_LD_RUN_PATH="LD_RUN_PATH=${LD_RUN_PATH}"
fi

dnl Make sure the header is here
AC_CHECK_HEADER( gd.h,:,[ GDFORM_EXT="" ])

if test x$GDFORM_EXT = x; then
echo
	echo "** Ooops, one of many bad things happened:"
	echo ""
	echo "   a)  You don't have the GD library installed."
	echo "       Get it from http://www.boutell.com, compile it and"
	echo "       use either --with-gd-lib=DIR and --with-gd-inc=DIR to specify"
	echo "       its location. You might also have to use --with-z-inc,"
        echo "	     --with-z-lib and --with-png-inc, --with-png-lib for gd"
        echo "	     versions 1.6 and higher.  Check config.log for more"
	echo "       information on the problem."
	echo ""
	echo "   b)  You have the GD library installed, but not the gd.h"
	echo "       header file.  Download the source (see above) and use"
	echo "       --with-gd-inc=DIR to specify where the file can be found."
	echo ""
	echo "   c)  You have the library and the header file installed, but"
	echo "       you also have a shared GD library in the same directory. "
        echo "       Remove the shared library files and/or links (e.g. "
	echo "       libgd.so.2.0.0, libgd.so and libgd.so.2).  This is especially"
        echo "	     likely if you're using a recent (post 1.8.4) version of GD"
	echo "       and didn't configure it with --disable-shared."
	echo ""
	echo "   d)  You have gd library installed and also it's headers, but you are"
	echo "       missing libpng (and headers) or freetype (and headers)"
	echo "       (mrtg does not use freetype, but if your copy of gd is precompiled"
	echo "       against it, you have to install it ... "
	echo ""
        echo "   Consider following the instructions in doc/mrtg-unix-guide.txt"
	exit 1
fi

AC_MSG_CHECKING(the weather)
AC_MSG_RESULT((cached) it's fine)

dnl Does the compiler like -Wall and
if test "x$GCC" = "xyes"; then
  oCFLAGS=$CFLAGS
  CFLAGS="$CFLAGS -Wall -Wextra -Wpedantic -Wpointer-arith -Wcast-align -Wmissing-declarations -Wnested-externs -Winline"
  AC_CACHE_CHECK(if we can use GCC-specific compiler options, rd_cv_gcc_opt,
                [AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[]], [[return 0 ]])],[rd_cv_gcc_opt=yes],[rd_cv_gcc_opt=no ])
               ]
        )
  if test $rd_cv_gcc_opt = no; then
         CFLAGS=$oCFLAGS
  fi
fi

AC_SUBST(PERL)
AC_SUBST(RRD_PERL)
AC_SUBST(GDFORM_EXT)
AC_SUBST(SET_LD_RUN_PATH)
AC_SUBST(LIBS)
AC_SUBST(CFLAGS)
AC_CONFIG_HEADERS(config.h)
AC_CONFIG_FILES([Makefile])
AC_OUTPUT


echo $ECHO_N "ordering CD from http://tobi.oetiker.ch/wish $ECHO_C" 1>&6
sleep 1
echo $ECHO_N ".$ECHO_C" 1>&6
sleep 1
echo $ECHO_N ".$ECHO_C" 1>&6
sleep 1
echo $ECHO_N ".$ECHO_C" 1>&6
sleep 1
echo $ECHO_N ".$ECHO_C" 1>&6
sleep 1
AC_MSG_RESULT([ just kidding ;-)])
sleep 2
echo
echo "----------------------------------------------------------------"
echo "Config is DONE!"
echo
echo "Type 'make' to compile the software"
echo 
echo "       ... that wishlist mentioned above does really exist. So if"
echo "you feel like showing your appreciation for MRTG, this is the"
echo "place to go. I just love CDs and DVDs"
echo 
echo "                            -- Tobi Oetiker <tobi@oetiker.ch>"
echo "----------------------------------------------------------------"

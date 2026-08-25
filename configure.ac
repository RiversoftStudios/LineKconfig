#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

#---------------------------------------------------------------------------
# Global initialisation

#----------------------------------------
# Prepare autoconf
AC_PREREQ([2.67])
AC_INIT(
    [kconfig-frontends],
    [m4_esyscmd_s([./scripts/version.sh])],
    [yann.morin.1998@free.fr])
AC_CONFIG_SRCDIR([frontends/conf/conf.c])
# Use a config.h to avoid brazilions -DHAVE_FOO on compile lines
AC_CONFIG_HEADERS([scripts/.autostuff/config.h])
AC_CONFIG_AUX_DIR([scripts/.autostuff/scripts])
AC_CONFIG_MACRO_DIR([scripts/.autostuff/m4])

#----------------------------------------
# Prepare automake

AM_INIT_AUTOMAKE([foreign subdir-objects])

# For releases, enable silent rules, unless the user explicitly
# disables them.
# For the devel tree, do build with verbose output, unless user
# explicitly enables silent rules
AS_IF(
    [test "$(${srcdir}/scripts/version.sh --internal)" = "git"],
    [AM_SILENT_RULES],
    [AM_SILENT_RULES([yes])])

#----------------------------------------
# Prepare libtool
m4_ifdef([AM_PROG_AR], [AM_PROG_AR]) dnl Automake, we have a problem...
LT_PREREQ([2.2.6])
LT_INIT([disable-static])

#---------------------------------------------------------------------------
# Set misc options

# By default, do build with -Wall, unless the user asks not to
[wall_CFLAGS=-Wall]
AC_ARG_ENABLE(
    [wall],
    [AS_HELP_STRING(
        [--disable-wall],
        [build with -Wall (default=yes)])],
    [AS_CASE(
        ["$enableval"],
        [yes], [wall_CFLAGS=-Wall],
        [*],   [wall_CFLAGS=""])])
AC_SUBST([wall_CFLAGS],[${wall_CFLAGS}])

# For releases, do not build with -Werror, unless the user explicitly
# requests to build with -Werror.
# For the devel tree, do build with -Werror by default, unless user
# explicitly disables -Werror
AS_IF(
    [test "$(${srcdir}/scripts/version.sh --internal)" = "git"],
    [werror_CFLAGS=-Werror])
AC_ARG_ENABLE(
    [werror],
    [AS_HELP_STRING(
        [--enable-werror],
        [build with -Werror (default=no)])],
    [AS_CASE(
        ["$enableval"],
        [yes], [werror_CFLAGS=-Werror],
        [*],   [werror_CFLAGS=""])])
AC_SUBST([werror_CFLAGS],[${werror_CFLAGS}])

# Although there is a default (="linux") in the code, we do provide
# a default here, to get a consistent autostuff behavior
AC_ARG_ENABLE(
    [root-menu-prompt],
    [AS_HELP_STRING(
        [--enable-root-menu-prompt=PROMPT],
        [set the root-menu prompt (default=Configuration)])],
    [AS_CASE(
        ["$enableval"],
        [yes], [root_menu=Configuration],
        [no],  [root_menu=],
               [# Escape the $ signs, otherwise they would get munged by make
                root_menu="$( echo "$enableval" |sed -e 's/\$/\\$$/g;' )"])])
AC_SUBST([root_menu], [${root_menu=Configuration}])

AC_ARG_ENABLE(
    [config-prefix],
    [AS_HELP_STRING(
        [--enable-config-prefix=PREFIX],
        [the prefix to the config option (default=CONFIG_)])],
    [AS_CASE(
        ["$enableval"],
        [*" "*],[AC_MSG_ERROR([config prefix can not contain spaces: '$enableval'])],
        [yes],  [config_prefix=CONFIG_],
        [no],   [config_prefix=],
                [config_prefix=$enableval])])
AC_SUBST([config_prefix], [${config_prefix-CONFIG_}])

AC_ARG_ENABLE(
    [utils],
    [AS_HELP_STRING(
        [--disable-utils],
        [install utilities to manage .config files (default=yes)])])
AC_SUBST([enable_utils], [${enable_utils:-yes}])

AC_ARG_ENABLE(
    [L10n],
    [AS_HELP_STRING(
        [--disable-L10n],
        [enable localisation (L10n) (default=auto)])])
AC_SUBST([enable_L10n], [${enable_L10n:-yes}])

#----------------------------------------
# Options to selectively enable/disable frontends
# All are selected by default
AC_ARG_ENABLE(
    [conf],
    [AS_HELP_STRING(
        [--disable-conf],
        [conf, the stdin-based frontend (default=auto)])])
AC_SUBST([enable_conf], [${enable_conf:-auto}])

AC_ARG_ENABLE(
    [frontends],
    [AS_HELP_STRING(
        [--enable-frontends=list],
        [enables only the set of frontends in comma-separated 'list'
         (default: auto selection), takes precedence over all
         --enable-*conf, above])],
    [for f in conf; do
        AS_CASE(
            ["$enableval"],
            [yes],      [eval enable_$f=yes],
            ["$f"],     [eval enable_$f=yes],
            ["$f",*],   [eval enable_$f=yes],
            [*,"$f"],   [eval enable_$f=yes],
            [*,"$f",*], [eval enable_$f=yes],
                        [eval enable_$f=no])
     done])
AC_SUBST([enable_frontends])

#----------------------------------------
# What extra CFLAGS we will be using
AC_SUBST([kf_CFLAGS], ["$wall_CFLAGS $werror_CFLAGS"])

#----------------------------------------
# Dependencies that will be needed, depending on the frontends
AS_CASE(
    [*yes*],  [need_curses=yes],
    [*auto*], [need_curses=auto],
              [need_curses=no])
[need_panel_menu="$enable_nconf"]

#---------------------------------------------------------------------------
# Now check we have the required stuff

#----------------------------------------
# Some program checks
AC_PROG_CC
AM_PROG_CC_C_O
AC_C_INLINE
AC_PROG_MAKE_SET
# Look for `lex'. If it cannot be found, autoconf sets $LEX to ':'.
AC_PROG_LEX
AS_IF(
    [test "$LEX" = ":"],
    [AC_MSG_ERROR([can not find a lexer generator (such as lex or flex)])])
AC_SUBST([AM_LFLAGS], ["-L -P zconf"])
# Look for `yacc'. If it cannot be found, autoconf sets $YACC to 'yacc'.
AC_PROG_YACC
AS_IF(
    [test "$YACC" = "yacc"],
    [AC_CHECK_PROGS(
        [YACC_IN_PATH],
        [yacc])]
     AS_IF(
        [test -z "$YACC_IN_PATH"],
        [AC_MSG_ERROR([can not find a parser generator (such as yacc or bison)])]))
AC_SUBST([AM_YFLAGS], ["-t -l -p zconf"])

#----------------------------------------
# Special section to check for gperf.
AC_CHECK_PROGS(
    [GPERF],
    [gperf])
AS_IF(
    [test -z "$GPERF"],
    [AC_MSG_ERROR([can not find gperf])])

#----------------------------------------
# Check for gettext, for the kconfig frontends
[has_gettext="$enable_L10n"]
AS_IF(
    [test "$has_gettext" = "yes"],
    [AC_CHECK_HEADERS(
        [libintl.h],
        [ac_ct_gettext_hdr=$ac_header; break],
        [has_gettext=no])])
AS_IF(
    [test "$has_gettext" = "yes"],
    [AC_CHECK_DECL(
        [gettext],,
        [has_gettext=no],
        [#include <$ac_ct_gettext_hdr>])])
AS_IF(
    [test "$has_gettext" = "yes"],
    [LIBS_old="$LIBS"
     LIBS=
     AC_SEARCH_LIBS(
        [gettext],
        [intl],,
        [has_gettext=no])
    intl_LIBS="$LIBS"
    LIBS="$LIBS_old"])
AS_IF(
    [test "$has_gettext" = "no"],
    [intl_CPPFLAGS=-DKBUILD_NO_NLS])
AC_SUBST([intl_CPPFLAGS])
AC_SUBST([intl_LIBS])

#----------------------------------------
# Check for libpanel and libmenu, for the nconf frontend
AS_IF(
    [test "$need_panel_menu" = "yes" -o "$need_panel_menu" = "auto"],
    [AC_SUBST([ncurses_nconf_CPPFLAGS])
     AC_SUBST([ncurses_panel_menu_LIBS])
     AS_CASE(
        [$CURSES_LOC],
        [ncursesw/*],[ncurses_nconf_CPPFLAGS="-I/usr/include/ncursesw"],
        [ncurses/*],[ncurses_nconf_CPPFLAGS="-I/usr/include/ncurses"])
     LIBS_old="$LIBS"
     LIBS=
     AC_SEARCH_LIBS(
        [new_panel],
        [panelw panel],
        [ac_ct_panel_lib_found=yes; break],,
        [$ncurses_LIBS])
     AS_IF(
        [test -z "$ac_ct_panel_lib_found"],
        [AS_IF(
            [test "$need_panel_menu" = "yes"],
            [AC_MSG_ERROR([could not find libpanel library (frontend: nconf)])],
            [has_panel_menu=no])])
     AC_SEARCH_LIBS(
        [menu_init],
        [menuw menu],
        [ac_ct_menu_lib_found=yes; break],,
        [$ncurses_LIBS])
     AS_IF(
        [test -z "$ac_ct_panel_lib_found"],
        [AS_IF(
            [test "$need_panel_menu" = "yes"],
            [AC_MSG_ERROR([could not find libmenu library (frontend: nconf)])],
            [has_panel_menu=no])])
     ncurses_panel_menu_LIBS="$LIBS"
     LIBS=$LIBS_old])

AS_IF(
    [test "$has_panel_menu" = "no" ],
    [enable_nconf=no])

#----------------------------------------
# Per-frontends extra libraries
AC_ARG_VAR([conf_EXTRA_LIBS],  [Extra libraries to build the conf frontend] )

#---------------------------------------------------------------------------
# Now, we know what frontends to enable
AS_IF([test "$enable_conf"  = "auto"], [enable_conf=yes ])

#----------------------------------------
# Setup automake conditional build
AM_CONDITIONAL(
    [COND_conf],
    [test "$enable_conf" = "yes"])
AM_CONDITIONAL(
    [COND_utils],
    [test "$enable_utils" = "yes"])
AM_CONDITIONAL(
    [COND_utils_gettext],
    [test "$has_gettext" = "yes"])

#----------------------------------------
# Get the version to apply to the parser shared library
AC_SUBST(
    [KCONFIGPARSER_LIB_VERSION],
    [m4_esyscmd_s([./scripts/version.sh --plain])])

#----------------------------------------
# Pretty-print the configuration settings
[fe_list=]
AS_IF([test "$enable_conf"  = "yes"], [fe_list="$fe_list conf" ])
# The meta frontend is handled separatly, below, because we do not
# want it in the list of tools, kcfg_list.

[kcfg_list="$fe_list"]
AS_IF([test "$enable_utils" = "yes"], [kcfg_list="$kcfg_list diff merge tweak"])
AS_IF([test "$has_gettext" = "yes"],  [kcfg_list="$kcfg_list gettext"])
AC_SUBST([kcfg_list], [${kcfg_list}])

[lib_list=]
AS_IF(
    [test "$enable_shared" = "yes"],
    [lib_list="$lib_list shared (version: $KCONFIGPARSER_LIB_VERSION)"])
AS_IF(
    [test "$enable_static" = "yes"],
    [lib_list="$lib_list${lib_list:+,} static"])

#----------------------------------------
# Finalise
# All generated files are generated by a Makefile rule, except Makefile
# itself of course.
# There is no generic solution in automake to generate a file from its
# .in source, so we'd have to provide custom, hand-made rules, which is
# not nice. So, we handle libs/parser/kconfig-parser.pc here.
AC_CONFIG_FILES([
    Makefile
    libs/parser/kconfig-parser.pc
])
AC_OUTPUT

AC_MSG_NOTICE()
AC_MSG_NOTICE([Configured with:])
AC_MSG_NOTICE([- parser library     :$lib_list])
AC_MSG_NOTICE([  - root-menu prompt : $root_menu])
AC_MSG_NOTICE([  - config prefix    : $config_prefix])
AC_MSG_NOTICE([- frontends          :$fe_list])
AC_MSG_NOTICE([  - localised        : $has_gettext])
AC_MSG_NOTICE([- install utilities  : $enable_utils])
AC_MSG_NOTICE([- CFLAGS CXXFLAGS    : $wall_CFLAGS $werror_CFLAGS])

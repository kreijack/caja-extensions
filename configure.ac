AC_INIT([caja-extensions], 1.26.0, [https://github.com/mate-desktop/caja-extensions], [caja-extensions], [https://mate-desktop.org])

AM_INIT_AUTOMAKE([1.9 no-dist-gzip dist-xz tar-ustar check-news])
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])
AC_CONFIG_HEADERS(config.h)
AC_CONFIG_MACRO_DIR([m4])

AM_DISABLE_STATIC
AC_PROG_CC
AM_PROG_LIBTOOL
AC_PROG_INSTALL
AC_PROG_MAKE_SET

CAJA_REQUIRED=1.21.3
LIBMATE_DESKTOP_REQUIRED=1.17.0
GLIB_REQUIRED=2.50.0
GTK_REQUIRED=3.22.0

GLIB_GSETTINGS

GTK_DOC_CHECK(1.9)

MATE_COMMON_INIT
MATE_COMPILE_WARNINGS

# Check for pkgconfig first
AC_CHECK_PROG(HAVE_PKGCONFIG, pkg-config, yes, no)
AC_PATH_PROG(PKG_CONFIG, pkg-config, no)

# Give error and exit if we dont have pkgconfig
if test "x$HAVE_PKGCONFIG" = "xno"; then
  AC_MSG_ERROR([you need to have pkgconfig installed !])
fi

GLIB_GENMARSHAL=`$PKG_CONFIG --variable=glib_genmarshal glib-2.0`
AC_SUBST(GLIB_GENMARSHAL)

# GLIB_COMPILE_RESOURCES

AC_ARG_VAR([GLIB_COMPILE_RESOURCES],[glib-compile-resources bin])
AC_PATH_PROG([GLIB_COMPILE_RESOURCES],[glib-compile-resources],[])
if test -z "$GLIB_COMPILE_RESOURCES"; then
  AC_MSG_ERROR([glib-compile-resources not found])
fi

# XMLLINT

AC_ARG_VAR([XMLLINT],[xmllint bin])
AC_PATH_PROG([XMLLINT],[xmllint],[])
if test -z "$XMLLINT"; then
  AC_MSG_ERROR([xmllint not found])
fi

# Common requirements
PKG_CHECK_MODULES(CAJA,
                  libcaja-extension >= $CAJA_REQUIRED)
AC_SUBST(CAJA_CFLAGS)
AC_SUBST(CAJA_LIBS)

# av extension
AC_ARG_ENABLE([av],
        AS_HELP_STRING([--enable-totem-properties], [Enable AV plugin]),
        [enable_totem_properties=$enableval],
        [enable_totem_properties=yes])

if test "x$enable_totem_properties" = "xyes"; then
        PKG_CHECK_MODULES(TOTEM_PROPERTIES,
		gstreamer-tag-1.0
		gstreamer-pbutils-1.0
                gtk+-3.0  >= $GTK_REQUIRED
                glib-2.0  >= $GLIB_REQUIRED
                gio-2.0   >= $GLIB_REQUIRED)
        AC_SUBST(TOTEM_PROPERTIES_CFLAGS)
        AC_SUBST(TOTEM_PROPERTIES_LIBS)
fi

AM_CONDITIONAL(ENABLE_TOTEM_PROPERTIES, test x"$enable_totem_properties" = "xyes")

# image-converter extension
AC_ARG_ENABLE([image-converter],
        AS_HELP_STRING([--enable-image-converter], [Enable image-converter plugin]),
        [enable_image_converter=$enableval],
        [enable_image_converter=yes])

if test "x$enable_image-converter" = "xyes"; then
        PKG_CHECK_MODULES(IMAGE_CONVERTER,
                gtk+-3.0  >= $GTK_REQUIRED
                glib-2.0  >= $GLIB_REQUIRED
                gio-2.0   >= $GLIB_REQUIRED)
        AC_SUBST(IMAGE_CONVERTER_CFLAGS)
        AC_SUBST(IMAGE_CONVERTER_LIBS)
fi

AM_CONDITIONAL(ENABLE_IMAGE_CONVERTER, test x"$enable_image_converter" = "xyes")

# open-terminal extension
AC_ARG_ENABLE([open-terminal],
        AS_HELP_STRING([--enable-open-terminal], [Enable open-terminal plugin]),
        [enable_open_terminal=$enableval],
        [enable_open_terminal=yes])

if test "x$enable_open_terminal" = "xyes"; then
        PKG_CHECK_MODULES(OPENTERMINAL,
                glib-2.0           >= $GLIB_REQUIRED
                mate-desktop-2.0   >= $LIBMATE_DESKTOP_REQUIRED)
        AC_SUBST(OPENTERMINAL_CFLAGS)
        AC_SUBST(OPENTERMINAL_LIBS)
fi

AM_CONDITIONAL(ENABLE_OPEN_TERMINAL, test x"$enable_open_terminal" = "xyes")

# sendto extension
AC_ARG_ENABLE([sendto],
        AS_HELP_STRING([--enable-sendto], [Enable sendto plugin]),
        [enable_sendto=$enableval],
        [enable_sendto=yes])

if test "x$enable_sendto" = "xyes"; then
        GTHREAD_REQUIRED=2.6.0
        DBUS_REQUIRED=1.0
        DBUS_GLIB_REQUIRED=0.60
        GUPNP_REQUIRED=0.13
        PKG_CHECK_MODULES(SENDTO,
                glib-2.0    >= $GLIB_REQUIRED
                gthread-2.0 >= $GTHREAD_REQUIRED
                gmodule-2.0 >= $GLIB_REQUIRED
                gobject-2.0 >= $GLIB_REQUIRED
                gtk+-3.0 >= $GTK_REQUIRED)
        AC_SUBST(SENDTO_CFLAGS)
        AC_SUBST(SENDTO_LIBS)

    # The full list of plugins
    allowed_sendto_plugins="emailclient gajim caja-burn pidgin removable-devices upnp"

    sendto_plugin_error_or_ignore()
    {
        if test "${error_on_bad_sendto_plugin}" = "1" ; then
            AC_MSG_ERROR([$1])
        else
            AC_MSG_WARN([$1 (disabling sendto plugin)])
        fi
    }

    AC_ARG_WITH([sendto-plugins],
        [AS_HELP_STRING([--with-sendto-plugins],[Which caja-sendto plugins to compile (default: auto; "all", "autodetect" are valid)])],
        [], [with_sendto_plugins=autodetect])

    used_sendto_plugins=""
    used_sendto_plugins2=""

    if test "x${with_sendto_plugins}" = "xautodetect" ; then
        # Default to all plugins and just ignore them if they won't compile
        with_sendto_plugins="all"
        error_on_bad_sendto_plugin="0"
    else
        # Clean up the plugin list and error if a plugin won't compile
        with_sendto_plugins=`echo ,"${with_sendto_plugins}", | sed -e 's/[[ ,]][[ ,]]*/ /g' -e 's/,$//'`
        error_on_bad_sendto_plugin="1"
    fi

    # Iterate through the wanted plugins and weed out bad ones (or error)
    for wanted_sendto_plugin in ${with_sendto_plugins}; do
        for allowed_sendto_plugin in ${allowed_sendto_plugins}; do
            case ${wanted_sendto_plugin} in
		all)
                    # List all plugins as being used
                    used_sendto_plugins=${allowed_sendto_plugins}
                ;;
                autodetect)
                    # List all plugins as being used
                    used_sendto_plugins=${allowed_sendto_plugins}
                ;;
                ${allowed_sendto_plugin}) used_sendto_plugins="${used_sendto_plugins} ${wanted_sendto_plugin}";; # List the plugin
                *) ;;
            esac
        done
    done

    # Check for plugin-specific requirements and error if necessary
    for sendto_plugin in ${used_sendto_plugins}; do
        add_sendto_plugin="1"

        case ${sendto_plugin} in
            emailclient)
                PKG_CHECK_MODULES(EMAILCLIENT, gio-2.0,
                    enable_emailclient=yes, enable_emailclient=no)
                if test "${enable_emailclient}" != "yes" ; then
                    sendto_plugin_error_or_ignore "you need gio-2.0 to build the emailclient plugin"
                    add_sendto_plugin="0"
                fi
            ;;
            gajim)
                AC_ARG_WITH(sendto_gajim,
                    [AS_HELP_STRING(--with-sendto-gajim,[Path to Gajim install prefix])],
                    [GAJIM_PATH="$withval"],[GAJIM_PATH="$prefix"])

                GAJIM_SHARE_DIR="$GAJIM_PATH"/share/gajim

                PKG_CHECK_MODULES(DBUS, dbus-1 >= $DBUS_REQUIRED dbus-glib-1 >= $DBUS_GLIB_REQUIRED,
                    [AC_SUBST(DBUS_CFLAGS)
                    AC_SUBST(DBUS_LIBS)
                    enable_gajim=yes
                    AC_DEFINE_UNQUOTED(GAJIM_SHARE_DIR, "$GAJIM_SHARE_DIR", [path to gajim share dir])],
                    [enable_gajim=no])

                if test "${enable_gajim}" != "yes" ; then
                    sendto_plugin_error_or_ignore "you need dbus-glib installed to build the gajim plugin"
                    add_sendto_plugin="0"
                fi
            ;;
            caja-burn)
                PKG_CHECK_MODULES(GIO, gio-2.0,
                    enable_burn=yes, enable_burn=no)
                if test "${enable_burn}" != "yes" ; then
                    sendto_plugin_error_or_ignore "you need gio installed to build the caja-burn plugin"
                    add_sendto_plugin="0"
                fi
            ;;
            pidgin)
                PKG_CHECK_MODULES(PIDGIN, dbus-glib-1 >= $DBUS_GLIB_REQUIRED,
                    enable_pidgin=yes, enable_pidgin=no)
                if test "${enable_pidgin}" != "yes" ; then
                    sendto_plugin_error_or_ignore "you need dbus-glib installed to build the pidgin plugin"
                    add_sendto_plugin="0"
                fi
            ;;
            removable-devices)
                PKG_CHECK_MODULES(GIO, gio-2.0,
                    enable_removable=yes, enable_removable=no)
                if test "${enable_removable}" != "yes" ; then
                    sendto_plugin_error_or_ignore "you need gio-2.0 installed to build the removable-devices plugin"
                    add_sendto_plugin="0"
            fi
            ;;
            upnp)
                PKG_CHECK_MODULES(UPNP, gupnp-1.2 >= $GUPNP_REQUIRED,
                    enable_upnp=yes AC_DEFINE([HAS_GUPNP_VERSION_1_2], 1, ["Version is at least 1.2"]), enable_upnp=no)
                PKG_CHECK_MODULES(UPNP, gupnp-1.0 >= $GUPNP_REQUIRED,
                    enable_upnp=yes, enable_upnp=no)
                if test "${enable_upnp}" != "yes" ; then
                    sendto_plugin_error_or_ignore "you need gupnp installed to build the upnp plugin"
                    add_sendto_plugin="0"
                fi
            ;;
        esac

        # Add the specified plugin
        if test "${add_sendto_plugin}" = "1" ; then
            used_sendto_plugins2="${used_sendto_plugins2} ${sendto_plugin}"
        fi
    done

    ALL_SENDTO_PLUGINS=$allowed_sendto_plugins
    SENDTO_PLUGINS=$used_sendto_plugins2

    dnl Error out if no plugins will be built
    if test -z "${SENDTO_PLUGINS}" ; then
        AC_MSG_ERROR([No sendto plugins will be built, and caja-sendto requires at least one])
    fi

    AC_MSG_CHECKING([which sendto plugins to compile])
    AC_MSG_RESULT([$SENDTO_PLUGINS])

    AC_SUBST([ALL_SENDTO_PLUGINS])
    AC_SUBST([SENDTO_PLUGINS])

    dnl For the pidgin sendto plugin
    AM_CONDITIONAL(HAVE_SENDTO_PIDGIN, test "x$enable_pidgin" = "xyes")

    dnl For the GIO based plugins
    enable_nst_common=no
    if test "x$enable_burn" = "xyes" -o "x$enable_removable" = "xyes"; then
        enable_nst_common=yes
    fi
    AM_CONDITIONAL(HAVE_GIO, test "x$enable_nst_common" = "xyes")

    if test "x${SENDTO_PLUGINS}" != "x" ; then
        for allowed_sendto_plugin in ${ALL_SENDTO_PLUGINS}; do
            for sendto_plugin in ${SENDTO_PLUGINS}; do
                case ${allowed_sendto_plugin} in
                    ${sendto_plugin})
                        AC_MSG_NOTICE([** ${allowed_sendto_plugin} sendto plugin enabled])
                        continue 2
                    ;;
                    *);;
                esac
            done
            AC_MSG_NOTICE([ ${allowed_sendto_plugin} sendto plugin disabled])
        done
    else
        AC_MSG_NOTICE([ No caja-sendto plugins enabled])
    fi
fi
AM_CONDITIONAL(ENABLE_SENDTO, test x"$enable_sendto" = "xyes")

# share extension
AC_ARG_ENABLE([share],
        AS_HELP_STRING([--enable-share], [Enable share plugin]),
        [enable_share=$enableval],
        [enable_share=yes])

if test "x$enable_share" = "xyes"; then
        PKG_CHECK_MODULES(SHARE,
                glib-2.0 >= $GLIB_REQUIRED)
        AC_SUBST(SHARE_CFLAGS)
        AC_SUBST(SHARE_LIBS)
fi

AM_CONDITIONAL(ENABLE_SHARE, test x"$enable_share" = "xyes")

# gksu extension
AC_ARG_ENABLE([gksu],
        AS_HELP_STRING([--enable-gksu], [Enable gksu plugin]),
        [enable_gksu=$enableval],
        [enable_gksu=yes])

AM_CONDITIONAL(ENABLE_GKSU, test x"$enable_gksu" = "xyes")

# wallpaper extension
AC_ARG_ENABLE([wallpaper],
        AS_HELP_STRING([--enable-wallpaper], [Enable set as wallpaper plugin]),
        [enable_wallpaper=$enableval],
        [enable_wallpaper=yes])

AM_CONDITIONAL(ENABLE_WALLPAPER, test x"$enable_wallpaper" = "xyes")

# xattr-tags extension
AC_ARG_ENABLE([xattr-tags],
        AS_HELP_STRING([--enable-xattr-tags], [Enable xattr tags plugin]),
        [enable_xattr_tags=$enableval],
        [enable_xattr_tags=yes])

AM_CONDITIONAL(ENABLE_XATTR_TAGS, test x"$enable_xattr_tags" = "xyes")

# Get caja extensions directory
AC_ARG_WITH(cajadir,
    AS_HELP_STRING([--with-cajadir=DIR],[Installation path for Caja extension @<:@auto@:>@]),
    [ac_with_cajadir=$withval],
    [ac_with_cajadir=""])
if test "${ac_with_cajadir}" = ""; then
    ac_with_cajadir=`$PKG_CONFIG --variable=extensiondir libcaja-extension`
fi
AC_MSG_NOTICE([Installing caja plugins in ${ac_with_cajadir}])
AC_SUBST([CAJA_EXTENSION_DIR],[${ac_with_cajadir}])

# gettext
GETTEXT_PACKAGE=AC_PACKAGE_NAME
AC_SUBST(GETTEXT_PACKAGE)
AC_DEFINE_UNQUOTED(GETTEXT_PACKAGE, "$GETTEXT_PACKAGE", [gettext package])

AM_GNU_GETTEXT([external])
AM_GNU_GETTEXT_VERSION([0.19.8])
AM_CONDITIONAL([USE_NLS], [test "x${USE_NLS}" = "xyes"])

DISABLE_DEPRECATED_CPPFLAGS="-DG_DISABLE_DEPRECATED -DGDK_PIXBUF_DISABLE_DEPRECATED"
AC_SUBST(DISABLE_DEPRECATED_CPPFLAGS)

AC_CONFIG_FILES([
  Makefile
  audio-video-properties/Makefile
  audio-video-properties/libcaja-av.caja-extension.desktop.in
  sendto/org.mate.Caja.Sendto.gschema.xml
  open-terminal/org.mate.caja-open-terminal.gschema.xml
  gksu/libcaja-gksu.caja-extension.desktop.in
  gksu/Makefile
  image-converter/libcaja-image-converter.caja-extension.desktop.in
  image-converter/Makefile
  open-terminal/libcaja-open-terminal.caja-extension.desktop.in
  open-terminal/Makefile
  sendto/libcaja-sendto.caja-extension.desktop.in
  sendto/Makefile
  sendto/caja-sendto.pc
  sendto/docs/Makefile
  sendto/docs/version.xml
  sendto/plugins/Makefile
  sendto/plugins/emailclient/Makefile
  sendto/plugins/gajim/Makefile
  sendto/plugins/caja-burn/Makefile
  sendto/plugins/pidgin/Makefile
  sendto/plugins/removable-devices/Makefile
  sendto/plugins/upnp/Makefile
  share/libcaja-share.caja-extension.desktop.in
  share/Makefile
  wallpaper/libcaja-wallpaper.caja-extension.desktop.in
  wallpaper/Makefile
  xattr-tags/libcaja-xattr-tags.caja-extension.desktop.in
  xattr-tags/Makefile
  po/Makefile.in
])
AC_OUTPUT

echo "
Configure summary:

      ${PACKAGE_STRING}
      `echo $PACKAGE_STRING | sed "s/./=/g"`
      prefix:                     ${prefix}
      compiler:                   ${CC}
      cflags:                     ${CFLAGS}
      warning flags:              ${WARN_CFLAGS}
      caja-extension dir:         ${ac_with_cajadir}
      Native Language support:    ${USE_NLS}

      Plugins to be built
      `echo "Plugins to be built" | sed "s/./=/g"`
      Audio Video Properties:     $enable_totem_properties
      Image Converter:            $enable_image_converter
      Open Terminal:              $enable_open_terminal
      Sendto:                     $enable_sendto
      Share:                      $enable_share
      Gksu:                       $enable_gksu
      Wallpaper:                  $enable_wallpaper
      xattr Tags:                 $enable_xattr_tags
"

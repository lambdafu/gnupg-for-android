# Please install the following prerequisites (instructions for each follows):
# 	Android OS SDK: http://source.android.com/download
#
# Install and prepare the Android OS SDK ( http://source.android.com/download )
# on Debian or Ubuntu

### these modify the calling shell
# point pkg-config to the .pc files generated from these builds
export PKG_CONFIG_PATH=$(LOCAL)/lib/pkgconfig
# workaround for cross-compiling bug in autoconf
export ac_cv_func_malloc_0_nonnull=yes

CWD = $(shell pwd)
PROJECT_ROOT = $(CWD)/..
EXTERNAL_ROOT = $(PROJECT_ROOT)/external

# Android NDK setup
NDK_BASE ?=  /usr/local/android-ndk
NDK_PLATFORM_LEVEL ?= 9
NDK_SYSROOT=$(NDK_BASE)/platforms/android-$(NDK_PLATFORM_LEVEL)/arch-arm
NDK_UNAME=`uname -s | tr '[A-Z]' '[a-z]'`
NDK_TOOLCHAIN=$(NDK_BASE)/toolchains/arm-linux-androideabi-4.4.3/prebuilt/$(NDK_UNAME)-x86

# to use the real HOST tag, you need the latest libtool files:
# http://stackoverflow.com/questions/4594736/configure-does-not-recognize-androideabi
#HOST := arm-none-linux-gnueabi
HOST := arm-linux-androideabi

# install root for built files
DESTDIR = $(EXTERNAL_ROOT)
# TODO try adding the Android-style /data/app.name here
prefix = /data/data/info.guardianproject.gpg/app_opt
LOCAL := $(DESTDIR)$(prefix)

PATH := ${PATH}:$(NDK_TOOLCHAIN)/bin:$(LOCAL)/bin

CC := $(NDK_TOOLCHAIN)/bin/arm-linux-androideabi-gcc --sysroot=$(NDK_SYSROOT)
CXX := $(NDK_TOOLCHAIN)/bin/arm-linux-androideabi-g++
CPP := $(NDK_TOOLCHAIN)/bin/arm-linux-androideabi-cpp
LD := $(NDK_TOOLCHAIN)/bin/arm-linux-androideabi-ld
AR := $(NDK_TOOLCHAIN)/bin/arm-linux-androideabi-ar
RANLIB := $(NDK_TOOLCHAIN)/bin/arm-linux-androideabi-ranlib
STRIP := $(NDK_TOOLCHAIN)/bin/arm-linux-androideabi-strip \
	--strip-unneeded -R .note -R .comment

CFLAGS = -DANDROID -I$(LOCAL)/include
LDFLAGS = -L$(LOCAL)/lib

# build as small as possible, mostly useful for static binaries
CFLAGS += -fdata-sections -ffunction-sections -Os
LDFLAGS += -Wl,--gc-sections


.PHONY = clean distclean install-clean \
	libgpg-error-build libgpg-error-clean libgpg-error-install \
	libgcrypt-build libgcrypt-clean libgcrypt-install \
	libassuan-build libassuan-clean libassuan-install \
	libksba-build libksba-clean libksba-install \
	gnupg-build gnupg-clean \
	curl-build curl-clean curl-install \
	openldap-build openldap-clean openldap-install \
	npth-build npth-clean npth-install \
	pth-build pth-clean pth-install \
	libiconv-build libiconv-clean libiconv-install

all: $(LOCAL)/lib/libgcrypt.a


#------------------------------------------------------------------------------#
# libgpg-error

libgpg-error/config.sub: config.sub
	cp config.sub libgpg-error/config.sub

libgpg-error/config.guess: config.guess
	cp config.guess libgpg-error/config.guess

libgpg-error/configure: libgpg-error/configure.ac
	cd libgpg-error && ./autogen.sh

libgpg-error/Makefile: libgpg-error/config.sub libgpg-error/config.guess libgpg-error/configure
	cd libgpg-error && \
		./configure \
			CC="$(CC)" \
			AR=$(AR) \
			RANLIB=$(RANLIB) \
			CFLAGS="$(CFLAGS)" \
			LDFLAGS="$(LDFLAGS)" \
			--disable-languages \
			--enable-static \
			--host=$(HOST) \
			--prefix=$(prefix)


libgpg-error-build-stamp: libgpg-error/Makefile
	$(MAKE) -C libgpg-error
	touch libgpg-error-build-stamp

libgpg-error-build: libgpg-error-build-stamp

$(LOCAL)/lib/libgpg-error.a: libgpg-error-build
	$(MAKE) -C libgpg-error DESTDIR=$(DESTDIR) prefix=$(prefix) install

libgpg-error-install: $(LOCAL)/lib/libgpg-error.a

libgpg-error-clean:
	-rm -f libgpg-error-build-stamp


#------------------------------------------------------------------------------#
# libgcrypt

libgcrypt/config.sub: config.sub
	cp config.sub libgcrypt/config.sub

libgcrypt/config.guess: config.guess
	cp config.guess libgcrypt/config.guess

libgcrypt/configure: libgcrypt/configure.ac
	cd libgcrypt && ./autogen.sh

libgcrypt/Makefile: libgcrypt/config.sub libgcrypt/config.guess libgcrypt/configure
	cd libgcrypt && \
		CC="$(CC)" AR="$(AR)" RANLIB=$(RANLIB) CFLAGS="$(CFLAGS)" LDFLAGS="$(LDFLAGS)" \
			./configure \
				--enable-maintainer-mode \
				--enable-static \
				--host=$(HOST) \
				--with-gpg-error-prefix=$(LOCAL) \
				--prefix=$(prefix)

libgcrypt-build-stamp: $(LOCAL)/lib/libgpg-error.a libgcrypt/Makefile
	$(MAKE) -C libgcrypt
	touch libgcrypt-build-stamp

libgcrypt-build: libgcrypt-build-stamp

$(LOCAL)/lib/libgcrypt.a: libgcrypt-build
	$(MAKE) -C libgcrypt DESTDIR=$(DESTDIR) prefix=$(prefix) install
	ls -l $(LOCAL)/lib/libgcrypt.a

libgcrypt-clean:
	-rm -f libgcrypt-build-stamp


#------------------------------------------------------------------------------#
# libassuan

libassuan/config.sub: config.sub
	cp config.sub libassuan/config.sub

libassuan/config.guess: config.guess
	cp config.guess libassuan/config.guess

libassuan/configure: libassuan/configure.ac
	cd libassuan && ./autogen.sh

libassuan/Makefile: libassuan/config.sub libassuan/config.guess libassuan/configure
	cd libassuan && \
		CC="$(CC)" AR="$(AR)" RANLIB=$(RANLIB) CFLAGS="$(CFLAGS)" LDFLAGS="$(LDFLAGS)" \
			./configure \
				--enable-maintainer-mode \
				--enable-static \
				--host=$(HOST) \
				--with-gpg-error-prefix=$(LOCAL) \
				--prefix=$(prefix)

libassuan-build-stamp: $(LOCAL)/lib/libgpg-error.a libassuan/Makefile
	$(MAKE) -C libassuan
	touch libassuan-build-stamp

$(LOCAL)/lib/libassuan.a: libassuan-build-stamp
	$(MAKE) -C libassuan DESTDIR=$(DESTDIR) prefix=$(prefix) install
	ls -l $(LOCAL)/lib/libassuan.a

libassuan-install: $(LOCAL)/lib/libassuan.a

libassuan-clean:
	-rm -f libassuan-build-stamp


#------------------------------------------------------------------------------#
# npth

npth/config.sub: config.sub
	cp config.sub npth/config.sub

npth/config.guess: config.guess
	cp config.guess npth/config.guess

npth/configure: npth/configure.ac
	cd npth && ./autogen.sh

npth/Makefile: npth/config.sub npth/config.guess npth/configure
	cd npth && \
		CC="$(CC)" AR="$(AR)" RANLIB=$(RANLIB) CFLAGS="$(CFLAGS)" LDFLAGS="$(LDFLAGS)" \
			./configure \
				--enable-maintainer-mode \
				--enable-static \
				--host=$(HOST) \
				--with-gnu-ld \
				--prefix=$(prefix)

npth-build-stamp: $(LOCAL)/lib/libgpg-error.a npth/Makefile
	$(MAKE) -C npth
	touch npth-build-stamp

$(LOCAL)/lib/libnpth.a: npth-build-stamp
	$(MAKE) -C npth DESTDIR=$(DESTDIR) prefix=$(prefix) install
	ls -l $(LOCAL)/lib/libnpth.a

npth-install: $(LOCAL)/lib/libnpth.a

npth-clean:
	-rm -f npth-build-stamp


#------------------------------------------------------------------------------#
# libksba

libksba/config.sub: config.sub
	cp config.sub libksba/config.sub

libksba/config.guess: config.guess
	cp config.guess libksba/config.guess

libksba/configure: libksba/configure.ac
	cd libksba && ./autogen.sh

libksba-build: $(LOCAL)/lib/libgpg-error.a libksba/config.sub libksba/config.guess libksba/configure
	cd libksba && \
		CC="$(CC)" AR="$(AR)" RANLIB=$(RANLIB) CFLAGS="$(CFLAGS)" LDFLAGS="$(LDFLAGS)" \
			./configure \
				--enable-maintainer-mode \
				--enable-static \
				--host=$(HOST) \
				--with-gpg-error-prefix=$(LOCAL) \
				--prefix=$(prefix)
	$(MAKE) -C libksba

$(LOCAL)/lib/libksba.a: libksba-build
	$(MAKE) -C libksba DESTDIR=$(DESTDIR) prefix=$(prefix) install
	ls -l $(LOCAL)/lib/libksba.a

libksba-install: $(LOCAL)/lib/libksba.a

libksba-clean:
	-rm -f libksba-build-stamp


#------------------------------------------------------------------------------#
# openldap

openldap/configure:
	cd openldap && git checkout configure

# TODO try --with-tls=openssl or gnutls
openldap/Makefile: openldap/configure
	cd openldap && \
		CC="$(CC)" AR="$(AR)" RANLIB=$(RANLIB) CFLAGS="$(CFLAGS)" LDFLAGS="$(LDFLAGS)" \
			./configure \
				--enable-static \
				--host=$(HOST) \
				--prefix=$(prefix) \
				--without-tls \
				--without-threads \
				--without-yielding-select \
				--disable-slapd \
				--disable-syslog \
				--disable-bdb \
				--disable-hdb \
				--with-gnu-ld
# cross-compiling makes openldap's ./configure think memcmp() is broken
# http://zhuqy.wordpress.com/2010/04/22/cross-compile-openldap-error-undefined-reference-to-lutil_memcmp/
	sed --in-place 's|#define NEED_MEMCMP_REPLACEMENT 1|/* #undef NEED_MEMCMP_REPLACEMENT */|' openldap/include/portable.h
# I haven't found a reliable way to install/uninstall a patch from a Makefile,
# so just always try to apply it, and ignore it if it fails. Works fine unless
# the files being patched have changed, in which cause a partial application
# could happen unnoticed.
	-patch -N -p1 --reject-file=- < openldap-disable-sys_errlist.patch

openldap-build-stamp: openldap/Makefile
	$(MAKE) -C openldap
	touch openldap-build-stamp

openldap-build: openldap-build-stamp

$(LOCAL)/lib/libldap.a: openldap-build
	$(MAKE) -C openldap/include DESTDIR=$(DESTDIR) prefix=$(prefix) install
	$(MAKE) -C openldap/libraries DESTDIR=$(DESTDIR) prefix=$(prefix) install
	ls -l $(LOCAL)/lib/libldap.a

openldap-install: $(LOCAL)/lib/libldap.a

openldap-clean:
	-rm -f openldap-build-stamp


#------------------------------------------------------------------------------#
# curl

curl/config.sub: config.sub
	cp config.sub curl/config.sub

curl/config.guess: config.guess
	cp config.guess curl/config.guess

curl/configure: curl/configure.ac
	cd curl && ./buildconf

curl/Makefile: curl/config.sub curl/config.guess curl/configure
	cd curl && \
		CC="$(CC)" AR="$(AR)" RANLIB=$(RANLIB) CFLAGS="$(CFLAGS)" LDFLAGS="$(LDFLAGS)" \
			./configure \
				--enable-static \
				--host=$(HOST) \
				--prefix=$(prefix) \
				--with-ldap-lib=$(LOCAL) \
				--with-lber-lib=$(LOCAL) \
				--with-gnu-ld

curl-build-stamp: curl/Makefile
	$(MAKE) -C curl
	touch curl-build-stamp

curl-build: curl-build-stamp

$(LOCAL)/lib/libcurl.a: curl-build
	$(MAKE) -C curl DESTDIR=$(DESTDIR) prefix=$(prefix) install
	$(MAKE) -C curl DESTDIR=$(DESTDIR) prefix=$(prefix) install
	ls -l $(LOCAL)/lib/libcurl.a

curl-install: $(LOCAL)/lib/libcurl.a

curl-clean:
	-rm -f curl-build-stamp


#------------------------------------------------------------------------------#
# pth

pth-2.0.7.tar.gz.sig: pth-2.0.7.tar.gz
	wget http://ftp.gnu.org/gnu/pth/pth-2.0.7.tar.gz.sig
	gpg --recv-key  4E23E878D41A0A88EDFCFA5A6E744ACBA9C09E30
	gpg --verify pth-2.0.7.tar.gz.sig

pth-2.0.7.tar.gz:
	wget -c http://ftp.gnu.org/gnu/pth/pth-2.0.7.tar.gz
	sha1sum -c pth-2.0.7.tar.gz.sha1

pth-2.0.7: pth-2.0.7.tar.gz.sig
	tar xzf pth-2.0.7.tar.gz

pth-2.0.7/config.sub: pth-2.0.7 config.sub
	cp config.sub pth-2.0.7/config.sub

pth-2.0.7/config.guess: pth-2.0.7 config.guess
	cp config.guess pth-2.0.7/config.guess

pth-2.0.7/aclocal.m4: /usr/share/libtool/libltdl/aclocal.m4
	cp /usr/share/libtool/libltdl/aclocal.m4 pth-2.0.7/aclocal.m4
#	cd pth-2.0.7 && libtoolize --install --force
#	mkdir pth-2.0.7/m4
#	cp -a /usr/share/aclocal/*.m4 pth-2.0.7/m4

pth-2.0.7/Makefile: pth-2.0.7 pth-2.0.7/aclocal.m4 pth-2.0.7/config.guess pth-2.0.7/config.sub
	patch -N -p0 --reject-file=- < pth-fix-include.patch || true
	cd pth-2.0.7 && \
		CC="$(CC)" AR="$(AR)" RANLIB=$(RANLIB) CFLAGS="$(CFLAGS)" LDFLAGS="$(LDFLAGS)" \
			./configure \
				--prefix=$(prefix) \
				--host=$(HOST) \
				--enable-static \
				--disable-shared \
				--with-mctx-mth=sjlj \
				--with-mctx-dsp=ssjlj \
				--with-mctx-stk=sas

pth-build: pth-2.0.7/Makefile
	$(MAKE) -C pth-2.0.7

$(LOCAL)/lib/libpth.a: pth-build
	$(MAKE) -C pth-2.0.7 DESTDIR=$(DESTDIR) prefix=$(prefix) install

pth-install: $(LOCAL)/lib/libpth.a

pth-clean:
	-rm -rf pth-2.0.7
	-rm -f pth-2.0.7.tar.gz
	-rm -f pth-2.0.7.tar.gz.sig*


#------------------------------------------------------------------------------#
# adns

adns-1.4.tar.gz.sig: adns-1.4.tar.gz
	wget http://www.chiark.greenend.org.uk/~ian/adns/ftp/adns-1.4.tar.gz.sig
	gpg --verify adns-1.4.tar.gz.sig

adns-1.4.tar.gz:
	wget -c http://www.chiark.greenend.org.uk/~ian/adns/ftp/adns-1.4.tar.gz
	sha1sum -c adns-1.4.tar.gz.sha1

adns-1.4: adns-1.4.tar.gz.sig
	tar xzf adns-1.4.tar.gz

adns-1.4/config.sub: adns-1.4 config.sub
	cp config.sub adns-1.4/config.sub

adns-1.4/config.guess: adns-1.4 config.guess
	cp config.guess adns-1.4/config.guess

adns-1.4/Makefile: adns-1.4 adns-1.4/Makefile.in adns-1.4/config.guess adns-1.4/config.sub
	patch -N -p1 --reject-file=- < adns-destdir-install-fix.patch
	cd adns-1.4 && \
		CC="$(CC)" AR="$(AR)" RANLIB=$(RANLIB) CFLAGS="$(CFLAGS)" LDFLAGS="$(LDFLAGS)" \
			./configure \
				--prefix=$(prefix) \
				--host=$(HOST) \
				--disable-dynamic

adns-build: adns-1.4/Makefile
	$(MAKE) -C adns-1.4

$(LOCAL)/lib/libadns.a: adns-build
	$(MAKE) -C adns-1.4 DESTDIR=$(DESTDIR) prefix=$(prefix) install

adns-install: $(LOCAL)/lib/libadns.a

adns-clean:
	-rm -rf adns-1.4
	-rm -f adns-1.4.tar.gz
	-rm -f adns-1.4.tar.gz.sig*


#------------------------------------------------------------------------------#
# libiconv

libiconv-1.14.tar.gz.sig: libiconv-1.14.tar.gz
	wget -c http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.14.tar.gz.sig
	gpg --verify libiconv-1.14.tar.gz.sig

libiconv-1.14.tar.gz:
	wget -c http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.14.tar.gz
	sha1sum -c libiconv-1.14.tar.gz.sha1

libiconv-1.14: libiconv-1.14.tar.gz.sig
	tar xzf libiconv-1.14.tar.gz

libiconv-1.14/config.sub: libiconv-1.14 config.sub
	cp config.sub libiconv-1.14/config.sub

libiconv-1.14/config.guess: libiconv-1.14 config.guess
	cp config.guess libiconv-1.14/config.guess

libiconv-1.14/Makefile: libiconv-1.14 libiconv-1.14/Makefile.in libiconv-1.14/config.guess libiconv-1.14/config.sub
	cd libiconv-1.14 && \
		CFLAGS="$(CFLAGS) -mandroid" LDFLAGS="$(LDFLAGS)" \
			./configure \
				--prefix=$(prefix) \
				--host=arm-linux-eabi \
				--with-sysroot=$(NDK_SYSROOT) \
				--with-gnu-ld

libiconv-build-stamp: libiconv-1.14/Makefile
	$(MAKE) -C libiconv-1.14
	touch libiconv-build-stamp

libiconv-build: libiconv-build-stamp

$(LOCAL)/lib/libiconv.a: libiconv-build-stamp
	$(MAKE) -C libiconv-1.14 DESTDIR=$(DESTDIR) prefix=$(prefix) install

libiconv-install: $(LOCAL)/lib/libiconv.a

libiconv-clean:
	-rm -rf libiconv-1.14
	-rm -f libiconv-1.14.tar.gz
	-rm -f libiconv-1.14.tar.gz.sig*
	-rm -f libiconv-build-stamp


#------------------------------------------------------------------------------#
# gnupg

GNUPG_SOURCES := $(wildcard gnupg/include/*.h) $(wildcard gnupg/*/*.c)

gnupg/config.sub: config.sub
	cp config.sub gnupg/config.sub

gnupg/config.guess: config.guess
	cp config.guess gnupg/config.guess

gnupg/configure:
	cd gnupg && ./autogen.sh

gnupg/Makefile: gnupg/configure gnupg/config.sub gnupg/config.guess
	cd gnupg && \
		CC="$(CC)" AR="$(AR)" RANLIB=$(RANLIB) CFLAGS="$(CFLAGS)" LDFLAGS="$(LDFLAGS)" \
			./configure \
				--enable-maintainer-mode \
				--host=$(HOST) \
				--with-gpg-error-prefix=$(LOCAL) \
				--with-libgcrypt-prefix=$(LOCAL) \
				--with-libassuan-prefix=$(LOCAL) \
				--with-ksba-prefix=$(LOCAL) \
				--with-npth-prefix=$(LOCAL) \
				--with-libcurl=$(LOCAL) \
				--with-ldap=$(LOCAL) \
				--with-libiconv-prefix=$(LOCAL) \
				--enable-symcryptrun \
				--disable-scdaemon \
				--disable-g13 \
				--disable-tools \
				--disable-doc \
				--disable-ccid-driver \
				--prefix=$(prefix)

# adns builds for Android, but gnupg doesn't build for Android with adns enabled
#				--with-adns=$(LOCAL) \ $(LOCAL)/lib/libadns.a


gnupg-build: gnupg-build-stamp 
gnupg-build-stamp: $(LOCAL)/lib/libgpg-error.a $(LOCAL)/lib/libgcrypt.a $(LOCAL)/lib/libksba.a $(LOCAL)/lib/libassuan.a $(LOCAL)/lib/libldap.a $(LOCAL)/lib/libnpth.a gnupg/Makefile
	-patch -N -p1 --reject-file=- < gnupg-android-fixes.patch
	$(MAKE) -C gnupg
	touch gnupg-build-stamp

$(LOCAL)/bin/gpg2: gnupg-build $(GNUPG_SOURCES) gnupg/configure
	$(MAKE) -C gnupg DESTDIR=$(DESTDIR) prefix=$(prefix) install
	ls -l $(LOCAL)/bin/gpg2

gnupg-install: $(LOCAL)/bin/gpg2
	install -d $(LOCAL)/etc/gnupg
	install -d $(LOCAL)/var/run/gnupg
	install -d $(LOCAL)/var/cache/gnupg

gnupg-clean:
	-rm -f gnupg-build-stamp


#------------------------------------------------------------------------------#
# gpgme

gpgme/config.sub: config.sub
	cp config.sub gpgme/config.sub

gpgme/config.guess: config.guess
	cp config.guess gpgme/config.guess

gpgme/configure: gpgme/configure.ac
	cd gpgme && ./autogen.sh
# force these to copy until they are updated in the gpgme git upstream
	cp config.guess gpgme/config.guess
	cp config.sub gpgme/config.sub

gpgme/Makefile: gpgme/config.sub gpgme/config.guess gpgme/configure
# kludge to fix file location in the config scripts
	sed -i 's|^prefix=$(prefix)|prefix=$(LOCAL)|' $(LOCAL)/bin/*-config
	cd gpgme && \
		CC="$(CC)" AR="$(AR)" RANLIB=$(RANLIB) CFLAGS="$(CFLAGS)" LDFLAGS="$(LDFLAGS)" \
			./configure \
				--host=$(HOST) \
				--enable-maintainer-mode \
				--with-gnu-ld \
				--with-gpg-error-prefix=$(LOCAL) \
				--with-libassuan-prefix=$(LOCAL) \
				--with-gpg=$(prefix)/bin \
				--with-gpgsm=$(prefix)/bin \
				--with-gpgconf=$(prefix)/bin \
				--with-g13=$(prefix)/bin \
				--disable-glibtest \
				--disable-gpg-test \
				--disable-gpgsm-test \
				--disable-gpgconf-test \
				--disable-g13-test \
				--prefix=$(LOCAL)
	-patch -N -p1 --reject-file=- < gpgme-android-fixes.patch

gpgme-build-stamp: gpgme/Makefile
# it seems to die, but then recover
	$(MAKE) -C gpgme V=1 || $(MAKE) -C gpgme V=1
	touch gpgme-build-stamp

gpgme-build: gpgme-build-stamp

$(LOCAL)/lib/libgpgme.so: gpgme-build
# kludge because the install dies trying to install libgpgme.la
	$(MAKE) -C gpgme DESTDIR=$(DESTDIR) prefix=$(prefix) install \
		|| echo "IGNORING THE REST OF THE GPGME INSTALL"
	ls -l $(LOCAL)/lib/libgpgme.so

gpgme-install: gnupg-install $(LOCAL)/lib/libgpgme.so

gpgme-clean:
	-rm -f gpgme-build-stamp gpgme/Makefile gpg/configure
	-patch -N -R -p1 --reject-file=- < gpgme-android-fixes.patch


#------------------------------------------------------------------------------#
# ugly static linking hacks

# crazy ugly hack
gnupg-static: gpg2-static gpg-agent-static gpgv2-static dirmngr-static

gpg-agent-static:
	-rm -f gnupg/agent/gpg-agent
	cd gnupg/agent && $(CC) -DANDROID -I$(LOCAL)/include -Os -Wall -Wcast-align -Wshadow -Wstrict-prototypes -Wformat -Wno-format-y2k -Wformat-security -W -Wno-sign-compare -Wno-missing-field-initializers -Wdeclaration-after-statement -Wno-pointer-sign -Wpointer-arith  -L$(LOCAL)/lib -Wl,--rpath,$(prefix)/lib -o gpg-agent gpg_agent-gpg-agent.o gpg_agent-command.o gpg_agent-command-ssh.o gpg_agent-call-pinentry.o gpg_agent-cache.o gpg_agent-trans.o gpg_agent-findkey.o gpg_agent-pksign.o gpg_agent-pkdecrypt.o gpg_agent-genkey.o gpg_agent-protect.o gpg_agent-trustlist.o gpg_agent-divert-scd.o gpg_agent-cvt-openpgp.o gpg_agent-call-scd.o gpg_agent-learncard.o ../common/libcommonpth.a ../gl/libgnu.a  ../..$(prefix)/lib/libgcrypt.a ../..$(prefix)/lib/libgpg-error.a ../..$(prefix)/lib/libassuan.a ../..$(prefix)/lib/libnpth.a -ldl
	install gnupg/agent/gpg-agent $(LOCAL)/bin/gpg-agent-static
	$(STRIP)  $(LOCAL)/bin/gpg-agent-static

dirmngr-static:
	-rm -f gnupg/dirmngr/dirmngr gnupg/dirmngr/dirmngr-client
	cd gnupg/dirmngr && $(CC) -I$(LOCAL)/include -DANDROID -Os -Wall -Wcast-align -Wshadow -Wstrict-prototypes -Wformat -Wno-format-y2k -Wformat-security -W -Wno-sign-compare -Wno-missing-field-initializers -Wdeclaration-after-statement -Wno-pointer-sign -Wpointer-arith  -Wl,--rpath,$(prefix)/lib -o dirmngr dirmngr.o server.o crlcache.o crlfetch.o ldapserver.o certcache.o cdblib.o ldap.o misc.o ocsp.o validate.o  ks-action.o ks-engine-hkp.o ks-engine-http.o ks-engine-finger.o ks-engine-kdns.o ldap-wrapper.o  ../common/libcommonpth.a ../gl/libgnu.a  ../..$(prefix)/lib/libassuan.a ../..$(prefix)/lib/libgpg-error.a ../..$(prefix)/lib/libgcrypt.a ../..$(prefix)/lib/libksba.a ../..$(prefix)/lib/libnpth.a -ldl
	cd gnupg/dirmngr && $(CC) -I$(LOCAL)/include -DANDROID -O3 -Wall -Wcast-align -Wshadow -Wstrict-prototypes -Wformat -Wno-format-y2k -Wformat-security -W -Wno-sign-compare -Wno-missing-field-initializers -Wdeclaration-after-statement -Wno-pointer-sign -Wpointer-arith  -Wl,--rpath,$(prefix)/lib -o dirmngr-client dirmngr-client.o ../common/libcommon.a no-libgcrypt.o ../gl/libgnu.a ../..$(prefix)/lib/libassuan.a ../..$(prefix)/lib/libgpg-error.a
	install gnupg/dirmngr/dirmngr $(LOCAL)/bin/dirmngr-static
	$(STRIP)  $(LOCAL)/bin/dirmngr-static
	install gnupg/dirmngr/dirmngr-client $(LOCAL)/bin/dirmngr-client-static
	$(STRIP)  $(LOCAL)/bin/dirmngr-client-static

gpg2-static:
	-rm -f gnupg/g10/gpg2
	cd gnupg/g10 && $(CC) -DANDROID -I$(LOCAL)/include -Os -Wall -Wcast-align -Wshadow -Wstrict-prototypes -Wformat -Wno-format-y2k -Wformat-security -W -Wno-sign-compare -Wno-missing-field-initializers -Wdeclaration-after-statement -Wno-pointer-sign -Wpointer-arith  -Wl,--rpath,$(prefix)/lib -o gpg2 gpg.o server.o build-packet.o compress.o  free-packet.o getkey.o keydb.o keyring.o seskey.o kbnode.o mainproc.o armor.o mdfilter.o textfilter.o progress.o misc.o rmd160.o openfile.o keyid.o parse-packet.o cpr.o plaintext.o sig-check.o keylist.o pkglue.o ecdh.o pkclist.o skclist.o pubkey-enc.o passphrase.o decrypt.o decrypt-data.o cipher.o encrypt.o sign.o verify.o revoke.o keyedit.o dearmor.o import.o export.o trustdb.o tdbdump.o tdbio.o delkey.o keygen.o helptext.o keyserver.o call-dirmngr.o photoid.o call-agent.o card-util.o exec.o ../common/libcommon.a ../gl/libgnu.a ../common/libgpgrl.a  ../..$(prefix)/lib/libassuan.a  ../..$(prefix)/lib/libgpg-error.a  ../..$(prefix)/lib/libgcrypt.a  ../..$(prefix)/lib/libksba.a  ../..$(prefix)/lib/libldap.a  ../..$(prefix)/lib/liblber.a  -lz
	install gnupg/g10/gpg2 $(LOCAL)/bin/gpg2-static
	$(STRIP)  $(LOCAL)/bin/gpg2-static

gpgv2-static:
	-rm -f gnupg/g10/gpgv2
	cd gnupg/g10 && $(CC) -DANDROID -I$(LOCAL)/include -Os -Wall -Wcast-align -Wshadow -Wstrict-prototypes -Wformat -Wno-format-y2k -Wformat-security -W -Wno-sign-compare -Wno-missing-field-initializers -Wdeclaration-after-statement -Wno-pointer-sign -Wpointer-arith  -L$(LOCAL)/lib -Wl,--rpath,$(prefix)/lib -o gpgv2 gpgv.o build-packet.o compress.o  free-packet.o getkey.o keydb.o keyring.o seskey.o kbnode.o mainproc.o armor.o mdfilter.o textfilter.o progress.o misc.o rmd160.o openfile.o keyid.o parse-packet.o cpr.o plaintext.o sig-check.o keylist.o pkglue.o ecdh.o verify.o ../common/libcommon.a ../gl/libgnu.a ../common/libgpgrl.a ../..$(prefix)/lib/libgcrypt.a ../..$(prefix)/lib/libgpg-error.a ../..$(prefix)/lib/libassuan.a -lz
	install gnupg/g10/gpgv2 $(LOCAL)/bin/gpgv2-static
	$(STRIP)  $(LOCAL)/bin/gpgv2-static

#------------------------------------------------------------------------------#
# setup as assets for Android app

ASSETS := $(CWD)/../assets
android-assets:
	install -d $(ASSETS)
	-rm -rf $(ASSETS)/*
	cp -a $(LOCAL)/* $(ASSETS)
# remove all the stuff we don't need
	rm -f $(ASSETS)/bin/*-static
	rm -f $(ASSETS)/bin/gpgv2
	rm -f $(ASSETS)/lib/*.a $(ASSETS)/lib/*.la
# remove lib symlinks since Android AssetManager copies them as files
	rm -rf $(ASSETS)/lib/*.so
	for f in $(ASSETS)/lib/*.so.*; do \
		echo $$f; \
		test ! -L $$f || \
			(rm $$f && mv $$f.[0-9]* $$f); \
	done
	rm -rf $(ASSETS)/include
	rm -rf $(ASSETS)/share/man $(ASSETS)/share/info $(ASSETS)/share/doc
	rmdir $(ASSETS)/sbin


#------------------------------------------------------------------------------#
# clean

install-clean:
	rm -rf -- $(LOCAL)

clean: install-clean gnupg-clean curl-clean openldap-clean libksba-clean libassuan-clean npth-clean libgcrypt-clean libgpg-error-clean pth-clean adns-clean libiconv-clean
	-$(MAKE) -C gnupg clean
	-$(MAKE) -C curl clean
	-$(MAKE) -C openldap clean
	-$(MAKE) -C libksba clean
	-$(MAKE) -C libassuan clean
	-$(MAKE) -C libgcrypt clean
	-$(MAKE) -C libgpg-error clean

distclean: clean install-clean
	-$(MAKE) -C gnupg distclean
	-$(MAKE) -C curl distclean
	-rm -f curl/configure # their distclean fails to rm this
	-$(MAKE) -C openldap distclean
	-$(MAKE) -C libksba distclean
	-$(MAKE) -C npth distclean
	-$(MAKE) -C libassuan distclean
	-$(MAKE) -C libgcrypt distclean
	-$(MAKE) -C libgpg-error distclean

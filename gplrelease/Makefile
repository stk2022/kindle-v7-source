# Makefile that builds a cross compiler against an existing system.
#
# This version fetches the Maverick libc and headers and builds a
# compiler against that.  To build for a different system,
# replace the sysroot.stamp rule with one that installs the headers
# and libc for your target.
#
# Tested on Ubuntu Maverick.  It should work on any Linux
# operating that has 'dpkg' including Fedora.  Note that .deb files are
# 'ar' archives so you could extract them manually.
#
# Copyright (c) 2010 Linaro
# Contributors:
#   Michael Hope - initial implementation
#

# Basics:

# Target machine triplet
TARGET ?= arm-linux-gnueabi

CURDIR := $(shell pwd)

# Where to install resulting tarball. Does not have to be writable.
PREFIX = $(HOME)/opt/cross-gcc-linaro

# Configure flags to pass to GCC.  Sets the default system that GCC
# builds for.  Note that the calling convention must be the same as
# the pre-built sysroot
GCCFLAGS = --with-arch=armv7-a --with-tune=cortex-a9 --with-float=softfp --with-fpu=neon --enable-threads=posix --disable-libmudflap --disable-libssp --disable-libstdcxx-pch --enable-languages=c,c++ --enable-shared --with-system-zlib --disable-nls --without-included-gettext --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --enable-symvers=gnu --enable-__cxa_atexit --disable-libgomp --disable-multilib --disable-multiarch

# Sysroot:

# Where the headers and libc will end up.  Check out the GCC
# --with-build-sysroot configure option
SYSROOT = $(PREFIX)/$(TARGET)
BUILD_SYSROOT = $(CURDIR)/build/sysroot

# Sources:

# The binutils version to build
BINUTILSSRC = src/binutils-2.24.tar.gz

# The Linaro GCC version to build.
GCCSRC = src/gcc-linaro-4.8-2014.04.tar.gz

# Options:

# Build in parallel
PARALLEL =

# Directory to put all stamp files
stamp = stamp/

-include local.mk

all: tarball

# Make the final binary tarball
tarball: binutils gcc
	strip $(BUILD_SYSROOT)$(SYSROOT)/bin/*
	echo Install into $(PREFIX) > $(BUILD_SYSROOT)$(PREFIX)/INSTALL
	tar czf cross-$(TARGET)-$(notdir $(GCCSRC)) --numeric-owner -C $(dir $(BUILD_SYSROOT)$(PREFIX)) .

$(stamp)init-dirs:
	install -d src $(stamp) build
	touch $@

# Binutils
binutils: $(stamp)binutils-install

# Convenience target to fetch the binutils source
binutils-src: $(BINUTILSSRC)

# Copy the binutils tarball to src dir
$(BINUTILSSRC): $(stamp)init-dirs
	cp $(@F) $@ 

# Extract binutils source
$(stamp)binutils-extract: $(BINUTILSSRC)
	tar xf $< -C src
	touch $@

# Configure
$(stamp)binutils-configure: $(stamp)binutils-extract
	install -d build/binutils
	cd build/binutils && ../../src/binutils*/configure \
	--prefix=$(PREFIX) \
	--target=$(TARGET) \
	--disable-multiarch \
	--disable-multilib \
	--disable-nls \
	--disable-werror \
	--enable-lto \
	--with-sysroot=$(SYSROOT) \
	--with-build-sysroot=$(BUILD_SYSROOT)
	touch $@

# Build and install
$(stamp)binutils-install: $(stamp)binutils-configure
	make -C build/binutils $(PARALLEL) 
	make -C build/binutils $(PARALLEL) install DESTDIR=$(BUILD_SYSROOT)
	touch $@

# Fetch and extract the sysroot.  Unfortunately the packages include
# absolute links in sysroot/usr/lib.  Fix these up to point to the
# sysroot instead.
$(stamp)sysroot-extract: $(stamp)init-dirs
	cp sysroot_eglibc-2.19.tar.gz src
	tar -x -z -C $(BUILD_SYSROOT)$(SYSROOT) -f sysroot_eglibc-2.19.tar.gz
	touch $@

# Build GCC
gcc: $(stamp)gcc-install

# Convenience target to fetch the GCC source
gcc-src: $(GCCSRC)

# Copy source tar to src
$(GCCSRC): $(stamp)init-dirs
	cp $(@F) $@ 

# Extract source
$(stamp)gcc-extract: $(GCCSRC) $(stamp)init-dirs
	tar xf $< -C src
	touch $@

# Configure
$(stamp)gcc-configure: $(stamp)binutils-install $(stamp)sysroot-extract $(stamp)gcc-extract
	install -d build/gcc 
	export PATH=$(BUILD_SYSROOT)$(SYSROOT)/../bin:$(PATH); \
	cd build/gcc && ../../src/gcc-*/configure \
	--enable-languages=c,c++ \
	--prefix=$(PREFIX) \
	--target=$(TARGET) \
	--with-sysroot=$(SYSROOT) \
	--with-build-sysroot=$(BUILD_SYSROOT)$(SYSROOT) \
	$(GCCFLAGS)
	touch $@

# Build and install
$(stamp)gcc-install: $(stamp)gcc-configure
	export PATH=$(BUILD_SYSROOT)$(SYSROOT)/../bin:$(PATH); \
	make -C build/gcc $(PARALLEL); \
	make -C build/gcc $(PARALLEL) install DESTDIR=$(BUILD_SYSROOT)
	touch $@

# Remove all temporary files
clean:
	rm -rf $(stamp) build src

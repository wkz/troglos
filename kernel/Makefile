# Glue for cross compiling the Linux kernel
#
# Copyright (c) 2014-2016  Joachim Nilsson <troglobit@gmail.com>
#
# Permission to use, copy, modify, and/or distribute this software for
# any purpose with or without fee is hereby granted, provided that the
# above copyright notice and this permission notice appear in all copies.
#
# THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
# WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
# MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
# ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
# WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
# ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF OR
# IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
.PHONY: all defconfig oldconfig saveconfig headers_install modules_install clean distclean

-include $(ROOTDIR)/.config

PKGVER         ?= $(shell echo $(CONFIG_LINUX_VERSION)$(CONFIG_LINUX_RC) | sed 's/"//g')
PKGBASEVER     := $(shell echo $(PKGVER) | sed 's/\([0-9]*\.[0-9]*\).*/\1/')
PKG            := linux-$(PKGVER)
PKGNAME	       := kernel
PKGSUFFIX      := tar.xz
PKGTARGETS     := kernel headers_install modules modules_install
PKGPATCHES     := patches/$(PKGBASEVER)/*.patch patches/$(PKGBASEVER)/series
ifdef CONFIG_LINUX_RC
RC_PATH         = testing/
endif
PKGURL          = http://ftp.kernel.org/pub/linux/kernel/v4.x/$(RC_PATH)$(PKGTAR)
CONFIG          = config-$(PKGBASEVER)
ZIMAGE          = $(PKG)/arch/$(ARCH)/boot/zImage

include $(ROOTDIR)/pkg.mk

$(PKG)/.config::
	@if [ ! -s $(PKG)/.config ]; then					\
		echo "  INSTALL $@ from $(CONFIG)" | tee -a $(BUILDLOG);	\
		if [ -f $(CONFIG) ]; then					\
			cp $(CONFIG) $@;					\
			$(MAKE) V=1 -C $(PKG) oldconfig;			\
		else								\
			$(MAKE) V=1 -C $(PKG) versatile_defconfig $(REDIRECT);	\
		fi;								\
		echo "  PREPARE $(PKG)" | tee -a $(BUILDLOG);			\
		$(MAKE) V=1 -C $(PKG) prepare scripts $(REDIRECT);		\
	else									\
		echo "  NO-OP   $(PKG)/.config already installed." | tee -a $(BUILDLOG); \
	fi

$(ZIMAGE): $(PKG)/.config
	@echo "  BUILD   $(PKG)" | tee -a $(BUILDLOG)
	+@$(MAKE) V=1 -C $(PKG) zImage $(REDIRECT)

defconfig:
	@$(MAKE) V=1 -C $(PKG) versatile_defconfig $(REDIRECT)

oldconfig:: $(PKG)/.config
	@$(MAKE) V=1 -C $(PKG) oldconfig

saveconfig:
	@echo -n "  SAVING  "
	@cp -v $(PKG)/.config $(CONFIG)

menuconfig:
	@echo "  CONFIG  $(PKG)" | tee -a $(BUILDLOG)
	@$(MAKE) -C $(PKG) menuconfig

dtbinst: $(ZIMAGE)
	INSTALL_DTBS_PATH=$(IMAGEDIR) $(MAKE) V=1 -C $(PKG) dtbs dtbs_install $(REDIRECT)

kernel: $(ZIMAGE)
	@cp -v $(ZIMAGE) $(IMAGEDIR) $(REDIRECT)

modules: kernel
	@echo "  MODULES $(PKG)" | tee -a $(BUILDLOG)
	+@$(MAKE) V=1 -C $(PKG) modules $(REDIRECT)

headers_install: kernel
	@echo "  HEADERS kernel/$(PKG)" | tee -a $(BUILDLOG)
	+$(MAKE) V=1 -C $(PKG) headers_install ARCH=$(ARCH) INSTALL_HDR_PATH=$(STAGING) $(REDIRECT)

#modules:
#	rm -rf modules
#	mkdir -p modules/lib/modules/$(KERNELRELEASE)
#	sh -ec "cd $(KERNEL_MODULES);find * -depth -print0 | cpio -0pdm $(PWD)/modules/lib/modules/$(KERNELRELEASE)"
#	find modules/lib/modules/$(KERNELRELEASE) -name '*.ko' | xargs $(CROSS_COMPILE)strip -g

modules_install: modules
	@echo "  MODINST kernel/$(PKG)" | tee -a $(BUILDLOG)
	@. $(PKG)/.config;						\
	if [ "$$CONFIG_MODULES" = "y" ]; then				\
		if [ ! -d $(STAGING)/lib/modules ]; then		\
			mkdir -p $(STAGING)/lib/modules;		\
		fi;							\
		$(MAKE) V=1 -C $(PKG) INSTALL_MOD_PATH=$(STAGING)	\
				  INSTALL_MOD_STRIP=--strip-all		\
				  modules_install $(REDIRECT);		\
	fi

distclean::
	-@$(RM) -rf linux-*

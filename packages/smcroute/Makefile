# Glue for cross compiling SMCRoute

PKGVER     := 2.1.0
PKG        := smcroute-$(PKGVER)
PKGSUFFIX  := tar.xz
PKGURL      = $(TROGLOHUB)/$(PKGNAME)/releases/download/$(PKGVER)/$(PKGTAR)
PKGPATCHES :=
PKGCFG     := --host=$(CROSS_TARGET) --prefix= --disable-ipv6
PKGINSTALL := install-exec

include $(ROOTDIR)/pkg.mk

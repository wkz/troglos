# Glue for cross compiling libite (LITE)

PKGVER     := 1.8.2
PKG        := libite-$(PKGVER)
PKGSUFFIX  := tar.xz
PKGURL      = $(TROGLOHUB)/$(PKGNAME)/releases/download/v$(PKGVER)/$(PKGTAR)
PKGCFG     := --host=$(CROSS_TARGET) --prefix=

include $(ROOTDIR)/pkg.mk

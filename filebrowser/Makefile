#
# Copyright (C) 2020 Project OpenWrt
# This is free software, licensed under the GNU General Public License v3.
# See /LICENSE.txt for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=filebrowser
PKG_VERSION:=2.6.2
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_VERSION:=be8683f
PKG_SOURCE_URL:=https://github.com/filebrowser/filebrowser
PKG_MIRROR_HASH:=skip

PKG_LICENSE:=Apache-2.0
PKG_LICENSE_FILES:=LICENSE.txt
PKG_MAINTAINER:=CN_SZTL <cnsztl@project-openwrt.eu.org>

PKG_BUILD_DEPENDS:=golang/host node/host
PKG_BUILD_PARALLEL:=1
PKG_USE_MIPS16:=0

PKG_CONFIG_DEPENDS:= \
	CONFIG_FILEBROWSER_COMPRESS_GOPROXY \
	CONFIG_FILEBROWSER_COMPRESS_UPX

GO_PKG:=github.com/filebrowser/filebrowser
GO_PKG_LDFLAGS:=-s -w
GO_PKG_LDFLAGS_X:= \
	github.com/filebrowser/filebrowser/v2/version.CommitSHA=$(PKG_SOURCE_VERSION) \
	github.com/filebrowser/filebrowser/v2/version.Version=$(PKG_VERSION)

include $(INCLUDE_DIR)/package.mk
include $(TOPDIR)/feeds/packages/lang/golang/golang-package.mk

define Package/filebrowser
  SECTION:=utils
  CATEGORY:=Utilities
  TITLE:=Web File Browser which can be used as a middleware or standalone app.
  URL:=https://github.com/filebrowser/filebrowser
  DEPENDS:=$(GO_ARCH_DEPENDS)
endef

define Package/filebrowser/description
  filebrowser provides a file managing interface within a specified directory
  and it can be used to upload, delete, preview, rename and edit your files.
  It allows the creation of multiple users and each user can have its own directory.
  It can be used as a standalone app or as a middleware.
endef

define Package/filebrowser/config
config FILEBROWSER_COMPRESS_GOPROXY
	bool "Compiling with GOPROXY proxy"
	default n

config FILEBROWSER_COMPRESS_UPX
	bool "Compress executable files with UPX"
	default y
endef

ifeq ($(CONFIG_FILEBROWSER_COMPRESS_GOPROXY),y)
export GO111MODULE=on
export GOPROXY=https://goproxy.cn
endif

define Build/Compile
( \
  cd $(PKG_BUILD_DIR) ; \
  rm -rf frontend/dist ; \
  rm -f http/rice-box.go ; \
  cd $(PKG_BUILD_DIR)/frontend ; \
  npm ci; \
  npm run lint ; \
  npm run build ; \
  GOOS=$$$$(go env GOOS) GOARCH=$$$$(go env GOARCH) GO111MODULE=off go get -v github.com/GeertJohan/go.rice/rice/... ; \
  cd $(PKG_BUILD_DIR)/http ; \
  "$$$$(go env GOPATH)/bin/rice" embed-go ; \
  $(call GoPackage/Build/Compile) ; \
)
ifeq ($(CONFIG_FILEBROWSER_COMPRESS_UPX),y)
	$(STAGING_DIR_HOST)/bin/upx --lzma --best $(GO_PKG_BUILD_BIN_DIR)/filebrowser
endif
endef

define Package/filebrowser/install
	$(call GoPackage/Package/Install/Bin,$(PKG_INSTALL_DIR))
	$(INSTALL_DIR) $(1)/etc/config/ $(1)/etc/init.d/ $(1)/usr/bin/
	$(INSTALL_CONF) $(CURDIR)/files/filebrowser.config $(1)/etc/config/filebrowser
	$(INSTALL_BIN) $(CURDIR)/files/filebrowser.init $(1)/etc/init.d/filebrowser
	$(INSTALL_BIN) $(GO_PKG_BUILD_BIN_DIR)/filebrowser $(1)/usr/bin/filebrowser
endef

$(eval $(call GoBinPackage,filebrowser))
$(eval $(call BuildPackage,filebrowser))

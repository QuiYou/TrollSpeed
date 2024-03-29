ARCHS := arm64  # arm64e
TARGET := iphone:clang:latest:14.0
INSTALL_TARGET_PROCESSES := TrollSpeed
ENT_PLIST := $(PWD)/ent.plist
LAUNCHD_PLIST := $(PWD)/layout/Library/LaunchDaemons/ch.xxtou.hudapp.plist

THEOS := /opt/theos

include $(THEOS)/makefiles/common.mk

GIT_TAG_SHORT := $(shell git describe --tags --always --abbrev=0)
APPLICATION_NAME := TrollSpeed

TrollSpeed_USE_MODULES := 0
TrollSpeed_FILES += $(wildcard *.mm *.m)
TrollSpeed_FILES += $(wildcard *.swift)
TrollSpeed_CFLAGS += -fobjc-arc
TrollSpeed_CFLAGS += -Iinclude
TrollSpeed_CFLAGS += -include hud-prefix.pch
TrollSpeed_CCFLAGS += -DNOTIFY_LAUNCHED_HUD=\"ch.xxtou.notification.hud.launched\"
TrollSpeed_CCFLAGS += -DNOTIFY_DISMISSAL_HUD=\"ch.xxtou.notification.hud.dismissal\"
TrollSpeed_CCFLAGS += -DNOTIFY_RELOAD_HUD=\"ch.xxtou.notification.hud.reload\"
TrollSpeed_CCFLAGS += -DNOTIFY_RELOAD_APP=\"ch.xxtou.notification.app.reload\"
ifeq ($(SPAWN_AS_ROOT),1)
TrollSpeed_CCFLAGS += -DSPAWN_AS_ROOT
endif
MainApplication.mm_CCFLAGS += -std=c++14
TrollSpeed_FRAMEWORKS += CoreGraphics QuartzCore UIKit
TrollSpeed_PRIVATE_FRAMEWORKS += BackBoardServices CoreServices GraphicsServices IOKit SpringBoardServices
ifeq ($(TARGET_CODESIGN),ldid)
TrollSpeed_CODESIGN_FLAGS += -Sent.plist
else
TrollSpeed_CODESIGN_FLAGS += --entitlements ent.plist $(TARGET_CODESIGN_FLAGS)
endif

include $(THEOS_MAKE_PATH)/application.mk

ifeq ($(SPAWN_AS_ROOT),1)
ifneq ($(FINALPACKAGE),1)
SUBPROJECTS += memory_pressure
include $(THEOS_MAKE_PATH)/aggregate.mk
endif
endif

before-all::
	$(ECHO_NOTHING)[ ! -z $(SPAWN_AS_ROOT) ] && defaults write $(ENT_PLIST) com.apple.private.persona-mgmt -bool true || defaults delete $(ENT_PLIST) com.apple.private.persona-mgmt || true$(ECHO_END)
	$(ECHO_NOTHING)plutil -convert xml1 $(ENT_PLIST)$(ECHO_END)
	$(ECHO_NOTHING)defaults write $(LAUNCHD_PLIST) ProgramArguments -array "$(THEOS_PACKAGE_INSTALL_PREFIX)/Applications/TrollSpeed.app/TrollSpeed" "-hud" || true$(ECHO_END)
	$(ECHO_NOTHING)plutil -convert xml1 $(LAUNCHD_PLIST)$(ECHO_END)

before-package::
	$(ECHO_NOTHING)mv -f $(THEOS_STAGING_DIR)/usr/local/bin/memory_pressure $(THEOS_STAGING_DIR)/Applications/TrollSpeed.app || true$(ECHO_END)
	$(ECHO_NOTHING)rmdir $(THEOS_STAGING_DIR)/usr/local/bin $(THEOS_STAGING_DIR)/usr/local $(THEOS_STAGING_DIR)/usr || true$(ECHO_END)

after-package::
	$(ECHO_NOTHING)mkdir -p packages $(THEOS_STAGING_DIR)/Payload$(ECHO_END)
	$(ECHO_NOTHING)cp -rp $(THEOS_STAGING_DIR)$(THEOS_PACKAGE_INSTALL_PREFIX)/Applications/TrollSpeed.app $(THEOS_STAGING_DIR)/Payload$(ECHO_END)
	$(ECHO_NOTHING)cd $(THEOS_STAGING_DIR); zip -qr TrollSpeed_${GIT_TAG_SHORT}.tipa Payload; cd -;$(ECHO_END)
	$(ECHO_NOTHING)mv $(THEOS_STAGING_DIR)/TrollSpeed_${GIT_TAG_SHORT}.tipa packages/TrollSpeed_${GIT_TAG_SHORT}.tipa$(ECHO_END)

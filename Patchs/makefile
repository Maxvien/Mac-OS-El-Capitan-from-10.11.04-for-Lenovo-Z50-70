# makefile
#
#
# Created by RehabMan, modified Zaid-Rahawi
#

BUILDDIR=./build
RESOURCES=./Resources_CX20751
HDAINJECT=AppleHDA_CX20751.kext
HDALAYOUT=layout28

IASLFLAGS=-ve
IASL=iasl

VERSION_ERA=$(shell ./print_version.sh)
ifeq "$(VERSION_ERA)" "10.10-"
INSTDIR=/System/Library/Extensions
else
INSTDIR=/Library/Extensions
endif
SLE=/System/Library/Extensions

# for now only build SSDT-HACK.aml, not patched set
.PHONY: all
all: $(BUILDDIR)/SSDT-HACK.aml $(HDAINJECT)

$(BUILDDIR)/SSDT-HACK.aml: ./SSDT-HACK.dsl
	$(IASL) $(IASLFLAGS) -p $@ $<

.PHONY: clean
clean:
	rm -f $(BUILDDIR)/*.dsl $(BUILDDIR)/*.aml

# Clover Install
.PHONY: install
install: $(BUILDDIR)/SSDT-HACK.aml
	$(eval EFIDIR:=$(shell sudo ./mount_efi.sh /))
	cp $(BUILDDIR)/SSDT-HACK.aml $(EFIDIR)/EFI/CLOVER/ACPI/patched/SSDT-HACK.aml

$(HDAINJECT): $(RESOURCES)/ahhcd.plist $(RESOURCES)/layout/Platforms.xml.zlib $(RESOURCES)/layout/$(HDALAYOUT).xml.zlib ./patch_hda.sh
	./patch_hda.sh
	touch $@

$(RESOURCES)/layout/Platforms.xml.zlib: $(RESOURCES)/layout/Platforms.plist $(SLE)/AppleHDA.kext/Contents/Resources/Platforms.xml.zlib
	./tools/zlib inflate $(SLE)/AppleHDA.kext/Contents/Resources/Platforms.xml.zlib >/tmp/rm_Platforms.plist
	/usr/libexec/plistbuddy -c "Delete ':PathMaps'" /tmp/rm_Platforms.plist
	/usr/libexec/plistbuddy -c "Merge $(RESOURCES)/layout/Platforms.plist" /tmp/rm_Platforms.plist
	./tools/zlib deflate /tmp/rm_Platforms.plist >$@

$(RESOURCES)/layout/$(HDALAYOUT).xml.zlib: $(RESOURCES)/layout/$(HDALAYOUT).plist
	./tools/zlib deflate $< >$@

.PHONY: update_kernelcache
update_kernelcache:
	sudo touch $(SLE)
	sudo kextcache -update-volume /

.PHONY: install_hda
install_hda:
	sudo rm -Rf $(INSTDIR)/$(HDAINJECT)
	sudo cp -R ./$(HDAINJECT) $(INSTDIR)
	if [ "`which tag`" != "" ]; then sudo tag -a Blue $(INSTDIR)/$(HDAINJECT); fi
	make update_kernelcache


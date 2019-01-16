MKFILES_DIR=$(abspath $(dir $(lastword $(MAKEFILE_LIST))))
PROJECT_DIR=$(abspath $(MKFILES_DIR)/..)
PACKAGES_DIR=$(PROJECT_DIR)/packages
BUILD_DIR=$(PROJECT_DIR)/build
ETC_DIR=$(PROJECT_DIR)/etc

EDAPACK_BUILD_URL=https://github.com/EDAPack/edapack-build

include $(PROJECT_DIR)/etc/package.info
name:=$(name)
version:=$(version)

ifeq (true,$(VERBOSE))
Q=
WGET=wget
UNTARGZ=tar xvzf
else
Q=@
WGET=wget -q
UNTARGZ=tar xzf
endif

uname_o:=$(shell uname -o)

ifeq (Linux,$(uname_o))
platform=linux_x86_64
else
ifeq (GNU/Linux,$(uname_o))
platform=linux_x86_64
else
platform=unknown
endif
endif

platform:=any

VUNIT_VERSION=4.0.8
VUNIT_DIR=vunit-$(VUNIT_VERSION)
VUNIT_TGZ=$(VUNIT_DIR).tar.gz
VUNIT_URL=https://github.com/VUnit/vunit/archive/v$(VUNIT_VERSION).tar.gz

COLORAMA_VERSION=0.4.1
COLORAMA_DIR=colorama-$(COLORAMA_VERSION)
COLORAMA_TGZ=$(COLORAMA_DIR).tar.gz
COLORAMA_URL=https://files.pythonhosted.org/packages/76/53/e785891dce0e2f2b9f4b4ff5bc6062a53332ed28833c7afede841f46a5db/$(COLORAMA_TGZ)

PREFIX=$(BUILD_DIR)/$(name)/$(name)
PACKAGE=$(BUILD_DIR)/$(name)-$(platform)-$(version).tar.gz

all : $(PACKAGE)

$(PACKAGE) : $(BUILD_DIR)/$(name).d \
		$(PACKAGES_DIR)/install.py \
		$(ETC_DIR)/modulefile \
		$(ETC_DIR)/package.info
	$(Q)mkdir -p $(PREFIX)/etc
	$(Q)cp $(PACKAGES_DIR)/install.py $(PREFIX)/etc
	$(Q)sed -e "s/{{version}}/$(version)/g" \
		-e "s/{{name}}/$(name)/g" \
		$(ETC_DIR)/modulefile  > $(PREFIX)/etc/modulefile
	$(Q)cp $(ETC_DIR)/package.info $(PREFIX)/etc
	$(Q)cd $(PREFIX) ; tar czf $@ *

$(BUILD_DIR)/$(name).d : \
		$(PACKAGES_DIR)/$(VUNIT_TGZ) \
		$(PACKAGES_DIR)/$(COLORAMA_TGZ)
	$(Q)rm -rf $(BUILD_DIR)/$(name)
	$(Q)mkdir -p $(BUILD_DIR)/$(name)
	$(Q)cd $(BUILD_DIR)/$(name) ; $(UNTARGZ) $(PACKAGES_DIR)/$(VUNIT_TGZ)
	$(Q)cd $(BUILD_DIR)/$(name) ; $(UNTARGZ) $(PACKAGES_DIR)/$(COLORAMA_TGZ)
	$(Q)mkdir -p $(BUILD_DIR)/$(name)/$(name)
	$(Q)cp -r $(BUILD_DIR)/$(name)/$(VUNIT_DIR)/* \
		$(BUILD_DIR)/$(name)/$(name) 
	$(Q)cp -r $(BUILD_DIR)/$(name)/$(COLORAMA_DIR)/colorama \
		$(BUILD_DIR)/$(name)/$(name) 
	$(Q)touch $@

$(PACKAGES_DIR)/$(VUNIT_TGZ) :
	$(Q)mkdir -p $(PACKAGES_DIR)
	$(Q)$(WGET) -O $@ $(VUNIT_URL) || (rm -f $@ ; exit 1)

$(PACKAGES_DIR)/$(COLORAMA_TGZ) :
	$(Q)mkdir -p $(PACKAGES_DIR)
	$(Q)$(WGET) -O $@ $(COLORAMA_URL) || (rm -f $@ ; exit 1)

$(PACKAGES_DIR)/install.py : 
	$(Q)mkdir -p $(PACKAGES_DIR)
	$(Q)$(WGET) -O $@ $(EDAPACK_BUILD_URL)/raw/master/scripts/install.py

release : $(PACKAGE) $(PACKAGES_DIR)/upload.py
	$(Q)python3 $(PACKAGES_DIR)/upload.py \
		--org EDAPack --repo $(name) \
		--key $(GITHUB_API_TOKEN) --version $(version) $(PACKAGE)

$(PACKAGES_DIR)/upload.py :
	$(Q)mkdir -p $(PACKAGES_DIR)
	$(Q)$(WGET) -O $@ $(EDAPACK_BUILD_URL)/raw/master/scripts/upload.py

clean : 
	$(Q)rm -rf $(BUILD_DIR)

clean-all : 
	$(Q)rm -rf $(BUILD_DIR) $(PACKAGES_DIR)

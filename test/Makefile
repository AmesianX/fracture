#===- test/Makefile ----------------------------------------*- Makefile -*--===#
#
#                     The LLVM Compiler Infrastructure
#
# This file is distributed under the University of Illinois Open Source
# License. See LICENSE.TXT for details.
#
#===------------------------------------------------------------------------===#

LEVEL = ..
DIRS  =

all:: check-local

# Include other test rules
include Makefile.tests

#===------------------------------------------------------------------------===#
# DejaGNU testing support
#===------------------------------------------------------------------------===#

ifneq ($(GREP_OPTIONS),)
$(warning GREP_OPTIONS environment variable may interfere with test results)
endif

ifdef VERBOSE
LIT_ARGS := -v
else
LIT_ARGS := -s -v
endif

ifdef TESTSUITE
LIT_TESTSUITE := $(TESTSUITE)
else
LIT_TESTSUITE := .
endif

# Check what to run for -all.
LIT_ALL_TESTSUITES := $(LIT_TESTSUITE)

extra-site-cfgs::
.PHONY: extra-site-cfgs

# ulimits like these are redundantly enforced by the buildbots, so
# just removing them here won't work.
# Both AuroraUX & Solaris do not have the -m flag for ulimit
ifeq ($(HOST_OS),SunOS)
ULIMIT=ulimit -t 600 ; ulimit -d 512000 ; ulimit -v 512000 ;
else # !SunOS
ifeq ($(HOST_OS),AuroraUX)
ULIMIT=ulimit -t 600 ; ulimit -d 512000 ; ulimit -v 512000 ;
else # !AuroraUX
# Newer versions of python try to allocate an insane amount of address space for
# its thread-local storage, don't set a limit here.
# When -v is not used, then -s has to be used to limit the stack size.
# FIXME: Those limits should be enforced by lit instead of globally.
ULIMIT=ulimit -t 600 ; ulimit -d 512000 ; ulimit -m 512000 ; ulimit -s 8192 ;
endif # AuroraUX
endif # SunOS

check-local:: lit.site.cfg
	( $(ULIMIT) \
	  $(PYTHON) $(LLVM_SRC_ROOT)/utils/lit/lit.py $(LIT_ARGS) $(LIT_TESTSUITE) )

# This is a legacy alias dating from when both DejaGNU and lit were in use.
check-local-lit:: check-local

check-local-all:: lit.site.cfg
	( $(ULIMIT) \
	  $(PYTHON) $(LLVM_SRC_ROOT)/utils/lit/lit.py $(LIT_ARGS) $(LIT_ALL_TESTSUITES) )

clean::
	$(RM) -rf `find $(PROJ_OBJ_ROOT)/test -name Output -type d -print`

ifneq ($(OCAMLOPT),)
CC_FOR_OCAMLOPT := $(shell $(OCAMLOPT) -config | grep native_c_compiler | sed -e 's/native_c_compiler: //')
CXX_FOR_OCAMLOPT := $(subst gcc,g++,$(CC_FOR_OCAMLOPT))
endif

FORCE:

ifeq ($(DISABLE_ASSERTIONS),1)
ENABLE_ASSERTIONS=0
else
ENABLE_ASSERTIONS=1
endif

# Derive whether or not LTO is enabled by checking the extra options.
LTO_IS_ENABLED := 0
ifneq ($(findstring -flto,$(CompileCommonOpts)),)
LTO_IS_ENABLED := 1
else
ifneq ($(findstring -O4,$(CompileCommonOpts)),)
LTO_IS_ENABLED := 1
endif
endif

lit.site.cfg: FORCE
	@echo "Making LLVM 'lit.site.cfg' file..."
	@$(ECHOPATH) s=@LLVM_HOST_TRIPLE@=$(HOST_TRIPLE)=g > lit.tmp
	@$(ECHOPATH) s=@TARGET_TRIPLE@=$(TARGET_TRIPLE)=g >> lit.tmp
	@$(ECHOPATH) s=@LLVM_SOURCE_DIR@=$(LLVM_SRC_ROOT)=g >> lit.tmp
	@$(ECHOPATH) s=@LLVM_BINARY_DIR@=$(LLVM_OBJ_ROOT)=g >> lit.tmp
	@$(ECHOPATH) s=@PROJ_SRC_DIR@=$(PROJ_SRC_ROOT)=g >> lit.tmp
	@$(ECHOPATH) s=@PROJ_OBJ_DIR@=$(PROJ_OBJ_ROOT)=g >> lit.tmp
	@$(ECHOPATH) s=@LLVM_TOOLS_DIR@=$(ToolDir)=g >> lit.tmp
	@$(ECHOPATH) s=@SHLIBDIR@=$(SharedLibDir)=g >> lit.tmp
	@$(ECHOPATH) s=@SHLIBEXT@=$(SHLIBEXT)=g >> lit.tmp
	@$(ECHOPATH) s=@PYTHON_EXECUTABLE@=$(PYTHON)=g >> lit.tmp
	@$(ECHOPATH) s=@OCAMLOPT@=$(OCAMLOPT) -cc $(subst *,'\\\"',*$(subst =,"\\=",$(CXX_FOR_OCAMLOPT))*) -I $(LibDir)/ocaml=g >> lit.tmp
	@$(ECHOPATH) s=@ENABLE_SHARED@=$(ENABLE_SHARED)=g >> lit.tmp
	@$(ECHOPATH) s=@ENABLE_ASSERTIONS@=$(ENABLE_ASSERTIONS)=g >> lit.tmp
	@$(ECHOPATH) s=@LTO_IS_ENABLED@=$(LTO_IS_ENABLED)=g >> lit.tmp
	@$(ECHOPATH) s=@TARGETS_TO_BUILD@=$(TARGETS_TO_BUILD)=g >> lit.tmp
	@$(ECHOPATH) s=@LLVM_BINDINGS@=$(BINDINGS_TO_BUILD)=g >> lit.tmp
	@$(ECHOPATH) s=@HOST_OS@=$(HOST_OS)=g >> lit.tmp
	@$(ECHOPATH) s=@HOST_ARCH@=$(HOST_ARCH)=g >> lit.tmp
	@$(ECHOPATH) s=@HAVE_LIBZ@=$(HAVE_LIBZ)=g >> lit.tmp
	@sed -f lit.tmp $(PROJ_SRC_DIR)/lit.site.cfg.in > $@
	@-rm -f lit.tmp

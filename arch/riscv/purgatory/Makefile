# SPDX-License-Identifier: GPL-2.0
OBJECT_FILES_NON_STANDARD := y

purgatory-y := purgatory.o sha256.o entry.o string.o ctype.o rvstring.o

targets += $(purgatory-y)
PURGATORY_OBJS = $(addprefix $(obj)/,$(purgatory-y))

$(obj)/string.o: $(srctree)/lib/string.c FORCE
	$(call if_changed_rule,cc_o_c)

$(obj)/ctype.o: $(srctree)/lib/ctype.c FORCE
	$(call if_changed_rule,cc_o_c)

$(obj)/rvstring.o: $(srctree)/arch/riscv/lib/string.c FORCE
	$(call if_changed_rule,cc_o_c)

$(obj)/sha256.o: $(srctree)/lib/crypto/sha256.c FORCE
	$(call if_changed_rule,cc_o_c)

CFLAGS_sha256.o := -D__DISABLE_EXPORTS
CFLAGS_string.o := -D__DISABLE_EXPORTS
CFLAGS_ctype.o := -D__DISABLE_EXPORTS

# When linking purgatory.ro with -r unresolved symbols are not checked,
# also link a purgatory.chk binary without -r to check for unresolved symbols.
PURGATORY_LDFLAGS := -e purgatory_start -z nodefaultlib
LDFLAGS_purgatory.ro := -r $(PURGATORY_LDFLAGS)
LDFLAGS_purgatory.chk := $(PURGATORY_LDFLAGS)
targets += purgatory.ro purgatory.chk

# Sanitizer, etc. runtimes are unavailable and cannot be linked here.
GCOV_PROFILE	:= n
KASAN_SANITIZE	:= n
UBSAN_SANITIZE	:= n
KCSAN_SANITIZE	:= n
KCOV_INSTRUMENT := n

# These are adjustments to the compiler flags used for objects that
# make up the standalone purgatory.ro

PURGATORY_CFLAGS_REMOVE := -mcmodel=kernel
PURGATORY_CFLAGS := -mcmodel=medany -ffreestanding -fno-zero-initialized-in-bss
PURGATORY_CFLAGS += $(DISABLE_STACKLEAK_PLUGIN) -DDISABLE_BRANCH_PROFILING
PURGATORY_CFLAGS += -fno-stack-protector -g0

# Default KBUILD_CFLAGS can have -pg option set when FTRACE is enabled. That
# in turn leaves some undefined symbols like __fentry__ in purgatory and not
# sure how to relocate those.
ifdef CONFIG_FUNCTION_TRACER
PURGATORY_CFLAGS_REMOVE		+= $(CC_FLAGS_FTRACE)
endif

ifdef CONFIG_STACKPROTECTOR
PURGATORY_CFLAGS_REMOVE		+= -fstack-protector
endif

ifdef CONFIG_STACKPROTECTOR_STRONG
PURGATORY_CFLAGS_REMOVE		+= -fstack-protector-strong
endif

CFLAGS_REMOVE_purgatory.o	+= $(PURGATORY_CFLAGS_REMOVE)
CFLAGS_purgatory.o		+= $(PURGATORY_CFLAGS)

CFLAGS_REMOVE_sha256.o		+= $(PURGATORY_CFLAGS_REMOVE)
CFLAGS_sha256.o			+= $(PURGATORY_CFLAGS)

CFLAGS_REMOVE_string.o		+= $(PURGATORY_CFLAGS_REMOVE)
CFLAGS_string.o			+= $(PURGATORY_CFLAGS)

CFLAGS_REMOVE_ctype.o		+= $(PURGATORY_CFLAGS_REMOVE)
CFLAGS_ctype.o			+= $(PURGATORY_CFLAGS)

AFLAGS_REMOVE_entry.o		+= -Wa,-gdwarf-2
AFLAGS_REMOVE_memcpy.o		+= -Wa,-gdwarf-2
AFLAGS_REMOVE_memset.o		+= -Wa,-gdwarf-2

$(obj)/purgatory.ro: $(PURGATORY_OBJS) FORCE
		$(call if_changed,ld)

$(obj)/purgatory.chk: $(obj)/purgatory.ro FORCE
		$(call if_changed,ld)

$(obj)/kexec-purgatory.o: $(obj)/purgatory.ro $(obj)/purgatory.chk

obj-y += kexec-purgatory.o

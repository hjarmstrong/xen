
XEN_ROOT=$(CURDIR)/../../..
include $(XEN_ROOT)/tools/Rules.mk

TARGET := test_x86_emulator

.PHONY: all
all: $(TARGET)

.PHONY: run
run: $(TARGET)
	./$(TARGET)

cflags-x86_32 := "-mno-accumulate-outgoing-args -Dstatic="

blowfish.h: blowfish.c blowfish.mk Makefile
	rm -f $@.new blowfish.bin
	$(foreach arch,$(filter-out $(XEN_COMPILE_ARCH),x86_32) $(XEN_COMPILE_ARCH), \
	    for cflags in "" $(cflags-$(arch)); do \
		$(MAKE) -f blowfish.mk XEN_TARGET_ARCH=$(arch) BLOWFISH_CFLAGS="$$cflags" all; \
		flavor=$$(echo $${cflags} | sed -e 's, .*,,' -e 'y,-=,__,') ; \
		(echo "static unsigned int blowfish_$(arch)$${flavor}[] = {"; \
		 od -v -t x blowfish.bin | sed -e 's/^[0-9]* /0x/' -e 's/ /, 0x/g' -e 's/$$/,/'; \
		 echo "};") >>$@.new; \
		rm -f blowfish.bin; \
	    done; \
	)
	mv $@.new $@

$(TARGET): x86_emulate.o test_x86_emulator.o
	$(HOSTCC) -o $@ $^

.PHONY: clean
clean:
	rm -rf $(TARGET) *.o *~ core blowfish.h blowfish.bin x86_emulate asm

.PHONY: distclean
distclean: clean

.PHONY: install
install:

x86_emulate/x86_emulate.c x86_emulate/x86_emulate.h:
	[ -L x86_emulate ] || ln -sf $(XEN_ROOT)/xen/arch/x86/x86_emulate .

asm/x86-vendors.h asm/x86-defns.h asm/msr-index.h:
	[ -L asm ] || ln -sf $(XEN_ROOT)/xen/include/asm-x86 asm

HOSTCFLAGS += $(CFLAGS_xeninclude) -I.

x86.h := asm/x86-vendors.h asm/x86-defns.h asm/msr-index.h
x86_emulate.h := x86_emulate.h x86_emulate/x86_emulate.h $(x86.h)

x86_emulate.o: x86_emulate.c x86_emulate/x86_emulate.c $(x86_emulate.h)
	$(HOSTCC) $(HOSTCFLAGS) -D__XEN_TOOLS__ -c -g -o $@ $<

test_x86_emulator.o: test_x86_emulator.c blowfish.h $(x86_emulate.h)
	$(HOSTCC) $(HOSTCFLAGS) -c -g -o $@ $<

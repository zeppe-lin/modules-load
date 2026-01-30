.POSIX:

include ../config.mk

all:

install:
	mkdir -p $(DESTDIR)$(BASHCOMPDIR)
	cp -f bash_completion $(DESTDIR)$(BASHCOMPDIR)/modules-load

uninstall:
	rm -f $(DESTDIR)$(BASHCOMPDIR)/modules-load

clean:

.PHONY: all install uninstall clean

bin_PROGRAMS = slirp4netns

AM_CFLAGS = @GLIB_CFLAGS@ @SLIRP_CFLAGS@ @LIBCAP_CFLAGS@ @LIBSECCOMP_CFLAGS@

noinst_LIBRARIES = libparson.a

AM_TESTS_ENVIRONMENT = PATH="$(abs_top_builddir):$(PATH)"
TESTS = tests/test-slirp4netns.sh tests/test-slirp4netns-configure.sh tests/test-slirp4netns-exit-fd.sh tests/test-slirp4netns-ready-fd.sh tests/test-slirp4netns-api-socket.sh tests/test-slirp4netns-disable-host-loopback.sh tests/test-slirp4netns-cidr.sh

EXTRA_DIST = \
	slirp4netns.1.md \
	slirp4netns.1 \
	$(TESTS) \
	tests/common.sh \
	slirp4netns.h \
	api.h \
	sandbox.h \
	seccompfilter.h \
	tests/slirp4netns-no-unmount.sh \
	vendor/parson/LICENSE \
	vendor/parson/README.md \
	vendor/parson/parson.h

# define specific commit if git available or it was replaced during git-archive creation
COMMIT := $(shell V=6a7b16babc95b6a3056b33fb45b74a6f62262dd4 ; \
	expr match "$$V" ormat: >/dev/null \
		&& (cd "$$abs_srcdir" && [ -d .git ] && git describe --always --abbrev=0 --dirty --exclude=\* || echo unknown) \
		|| echo "$$V" )
DEFINE_COMMIT = -DCOMMIT="\"$(COMMIT)\""

slirp4netns_CFLAGS = $(AM_CFLAGS) $(DEFINE_COMMIT)
libparson_a_CFLAGS = $(AM_CFLAGS) -I$(abs_top_builddir)/vendor/parson
libparson_a_SOURCES = vendor/parson/parson.c

slirp4netns_SOURCES = main.c slirp4netns.c api.c sandbox.c seccompfilter.c
slirp4netns_LDADD = libparson.a @GLIB_LIBS@ @SLIRP_LIBS@ @LIBSECCOMP_LIBS@ -lpthread
man1_MANS = slirp4netns.1

generate-man:
	go-md2man -in slirp4netns.1.md -out slirp4netns.1

CLANGTIDY = clang-tidy -warnings-as-errors='*'

CLANGFORMAT = clang-format

lint:
	$(CLANGTIDY) $(slirp4netns_SOURCES) -- $(AM_CFLAGS)

lint-full:
	$(CLANGTIDY) $(slirp4netns_SOURCES) $(libparson_a_SOURCES) -- $(AM_CFLAGS)

indent:
	$(CLANGFORMAT) -i $(slirp4netns_SOURCES)

benchmark:
	benchmarks/benchmark-iperf3.sh
	benchmarks/benchmark-iperf3-reverse.sh

ci:
	$(MAKE) indent
	git diff --exit-code
# TODO: make sure ./vendor is synced with ./vendor.sh
	$(MAKE) lint
	$(MAKE) -j $(shell nproc) distcheck || ( find . -name test-suite.log | xargs cat; exit 1 )
	PATH=$(shell pwd):$$PATH $(MAKE) benchmark MTU=1500
	PATH=$(shell pwd):$$PATH $(MAKE) benchmark MTU=65520

.PHONY: generate-man lint lint-full indent benchmark ci

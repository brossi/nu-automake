AM_CPPFLAGS =  $(INCLUDES)
AM_LDFLAGS = $(PTHREAD_CFLAGS)


if EMBEDDED_LEVELDB
LEVELDB_CPPFLAGS += -I$(srcdir)/leveldb/include
LEVELDB_CPPFLAGS += -I$(srcdir)/leveldb/helpers/memenv
LIBLEVELDB += $(builddir)/leveldb/libleveldb.a
LIBMEMENV  += $(builddir)/leveldb/libmemenv.a

# NOTE: This dependency is not strictly necessary, but without it make may try to build both in parallel, which breaks the LevelDB build system in a race
$(LIBLEVELDB): $(LIBMEMENV)

$(LIBLEVELDB) $(LIBMEMENV):
	@echo "Building LevelDB ..." && $(MAKE) -C $(@D) $(@F) CXX="$(CXX)" \
	  CC="$(CC)" PLATFORM=$(TARGET_OS) AR="$(AR)" $(LEVELDB_TARGET_FLAGS) \
          OPT="$(CXXFLAGS) $(CPPFLAGS)"
endif

NU_CONFIG_INCLUDES=-I$(builddir)/config
NU_INCLUDES=-I$(builddir) -I$(builddir)/obj $(BOOST_CPPFLAGS) $(LEVELDB_CPPFLAGS)

LIBNU_SERVER=libnu_server.a
LIBBITCOIN_WALLET=libnu_wallet.a
LIBBITCOIN_COMMON=libnu_common.a
LIBBITCOIN_UTIL=libnu_util.a

LIBNUQT=qt/libnuqt.a

noinst_LIBRARIES = \
  libnu_server.a \
  libnu_common.a \
  libnu_util.a
if ENABLE_WALLET
NU_INCLUDES += $(BDB_CPPFLAGS)
noinst_LIBRARIES += libnu_wallet.a
endif

bin_PROGRAMS =
TESTS =

if BUILD_BITCOIND
  bin_PROGRAMS += Nud
endif

.PHONY: FORCE
# nu core #
NU_CORE_H = \
  addrman.h \
  alert.h \
  allocators.h \
  base58.h \
  # bloom.h \
  bignum.h \
  bitcoinrpc.h \
  checkpoints.h \
  checkpointsync.h \
  checkqueue.h \
  clientversion.h \
  coincontrol.h \
  compat.h \
  crypter.h \
  datafeed.h \
  #sph_groestl.h \
  #sph_types.h \
  #hashgroestl.h \
  db.h \
  distribution.h \
  hash.h \
  init.h \
  kernel.h \
  key.h \
  keystore.h \
  leveldb.h \
  limitedmap.h \
  liquidityinfo.h \
  main.h \
  mruset.h \
  net.h \
  netbase.h \
  protocol.h \
  scanbalance.h \
  script.h \
  serialize.h \
  sync.h \
  txdb.h \
  #txdb-leveldb.h \
  #irc.h \
  #strlcpy.h \
  ui_interface.h \
  uint256.h \
  unitmap.h
  util.h \
  version.h \
  vote.h \
  wallet.h \
  walletdb.h \
  walletthread.h

JSON_H = \
  json/json_spirit.h \
  json/json_spirit_error_position.h \
  json/json_spirit_reader.h \
  json/json_spirit_reader_template.h \
  json/json_spirit_stream_reader.h \
  json/json_spirit_utils.h \
  json/json_spirit_value.h \
  json/json_spirit_writer.h \
  json/json_spirit_writer_template.h

obj/build.h: FORCE
	@$(MKDIR_P) $(builddir)/obj
	@$(top_srcdir)/share/genbuild.sh $(abs_top_builddir)/src/obj/build.h \
	  $(abs_top_srcdir)
libnu_util_a-version.$(OBJEXT): obj/build.h

# server: shared between nud and nu
libnu_server_a_CPPFLAGS = $(NU_INCLUDES)
libnu_server_a_SOURCES = \
  addrman.cpp \
  alert.cpp \
  bitcoinrpc.cpp \
  blockmap.cpp
  bloom.cpp \
  checkpoints.cpp \
  checkpointsync.cpp \
  datafeed.cpp \
  db.cpp \
  distribution.cpp \
  init.cpp \
  keystore.cpp \
  #irc.cpp \
  #txdb-leveldb.cpp \
  leveldb.cpp \
  liquidityinfo.cpp \
  main.cpp \
  net.cpp \
  noui.cpp \
  rpcblockchain.cpp \
  rpcmining.cpp \
  rpcnet.cpp \
  rpcrawtransaction.cpp \
  scanbalance.cpp \
  txdb.cpp \
  vote.cpp \
  $(JSON_H) \
  $(NU_CORE_H)

# wallet: shared between nud and nu, but only linked
# when wallet enabled
libnu_wallet_a_CPPFLAGS = $(NU_INCLUDES)
libnu_wallet_a_SOURCES = \
  crypter.cpp \
  rpcdividend.cpp \
  rpcdump.cpp \
  rpcliquidity.cpp \
  rpcpark.cpp \
  rpctransaction.cpp \
  rpcvote.cpp \
  rpcwallet.cpp \
  wallet.cpp \
  walletdb.cpp \
  $(NU_CORE_H)

# common: shared between nud, nu and non-server tools
libnu_common_a_CPPFLAGS = $(NU_INCLUDES)
libnu_common_a_SOURCES = \
  key.cpp \
  netbase.cpp \
  protocol.cpp \
  script.cpp \
  hash.cpp \
  kernel.cpp \
  #groestl.c \
  sync.cpp \
  $(NU_CORE_H)

# util: shared between all executables.
# This library *must* be included to make sure that the glibc
# backward-compatibility objects and their sanity checks are linked.
libnu_util_a_CPPFLAGS = $(NU_INCLUDES)
libnu_util_a_SOURCES = \
  util.cpp \
  version.cpp \
  compat/glibc_sanity.cpp \
  compat/glibcxx_sanity.cpp \
  $(NU_CORE_H)

if GLIBC_BACK_COMPAT
libnu_util_a_SOURCES += compat/glibc_compat.cpp
libnu_util_a_SOURCES += compat/glibcxx_compat.cpp
endif

nodist_libnu_util_a_SOURCES = $(srcdir)/obj/build.h

# nud binary #
Nud_LDADD = \
  $(LIBNU_SERVER) \
  $(LIBBITCOIN_COMMON) \
  $(LIBBITCOIN_UTIL) \
  $(LIBLEVELDB) \
  $(LIBMEMENV)

if ENABLE_WALLET
Nud_LDADD += libnu_wallet.a
endif
Nud_SOURCES = daemon.cpp

Nud_LDADD += $(BOOST_LIBS) $(BDB_LIBS)
Nud_CPPFLAGS = $(NU_INCLUDES)

CLEANFILES = leveldb/libleveldb.a leveldb/libmemenv.a *.gcda *.gcno

DISTCLEANFILES = obj/build.h

EXTRA_DIST = leveldb

clean-local:
	-$(MAKE) -C leveldb clean
	rm -f leveldb/*/*.gcno leveldb/helpers/memenv/*.gcno
	-rm -f config.h

.rc.o:
	@test -f $(WINDRES)
	$(AM_V_GEN) $(WINDRES) -i $< -o $@

.mm.o:
	$(AM_V_CXX) $(OBJCXX) $(DEFS) $(DEFAULT_INCLUDES) $(INCLUDES) $(AM_CPPFLAGS) \
	  $(CPPFLAGS) $(AM_CXXFLAGS) $(QT_INCLUDES) $(CXXFLAGS)  -c -o $@ $<

%.pb.cc %.pb.h: %.proto
	@test -f $(PROTOC)
	$(AM_V_GEN) $(PROTOC) --cpp_out=$(@D) --proto_path=$(abspath $(<D) $<)

if ENABLE_TESTS
include Makefile.test.include
endif

if ENABLE_QT
include Makefile.qt.include
endif

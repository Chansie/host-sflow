# This software is distributed under the following license:
# http://sflow.net/license.html

FEATURES_ALL= ULOG NFLOG PCAP DOCKER KVM XEN NVML OVS CUMULUS OS10
BINDIR     ?= /usr/sbin
INITDIR    ?= /etc/init.d
CONFDIR    ?= /etc
MODDIR     ?= /etc/hsflowd/modules
SYSTEMDDIR ?= /lib/systemd/system

$(info $$FEATURES is [$(FEATURES)] from $(origin FEATURES))

XTGTS= $(FEATURES)
ifeq ("$(strip $(FEATURES))","ALL")
  XTGTS= $(FEATURES_ALL)
endif
$(info $$XTGTS is [$(XTGTS)])

# INSTROOT may be passed in
BIN_D=$(BINDIR)
INIT_D=$(INITDIR)
CONF_D=$(CONFDIR)
MOD_D=$(MODDIR)
SYSTEMD_D=$(SYSTEMDDIR)
ifneq ($(INSTROOT),)
	BIN_D=$(INSTROOT)/$(BINDIR)
	INIT_D=$(INSTROOT)/$(INITDIR)
	CONF_D=$(INSTROOT)/$(CONFDIR)
	MOD_D=$(INSTROOT)/$(MODDIR)
	SYSTEMD_D=$(INSTROOT)/$(SYSTEMDDIR)
endif

#########  compilation flags  #########

HEADERS= util.h evbus.h hsflowd.h hsflowtokens.h hsflow_ethtool.h cpu_utils.h Makefile

# compiler
#CC= g++
CC= gcc -std=gnu99
LD=ld

# optimization
OPT_ALL= -O3 -DNDEBUG
OPT_REG= -g -O2
OPT_DBG= -g -ggdb
ifndef OPT
  OPT=$(OPT_REG)
endif

# other source directories
SFLOWDIR=../sflow
JSONDIR=../json

# CFLAGS and LIBS - common
CFLAGS_HSFLOWD= -I. -I$(JSONDIR) -I$(SFLOWDIR)
LIBS_HSFLOWD= $(JSONDIR)/libjson.a $(SFLOWDIR)/libsflow.a -lm -pthread -ldl

CFLAGS_SHARED= -fPIC
LDFLAGS_SHARED= -shared
# LDFLAGS_SHARED += --build-id

# CFLAGS and LIBS - modules
CFLAGS_JSON=
LIBS_JSON=

CFLAGS_DNSSD=
LIBS_DNSSD=-lresolv

CFLAGS_XEN=
LIBS_XEN= -lxenstore -lxenctrl

CFLAGS_KVM= -DHSP_KVM -I/usr/include/libvirt -I/usr/include/libxml2
LIBS_KVM= -lvirt -lxml2

CFLAGS_DOCKER=
LIBS_DOCKER=

CFLAGS_ULOG=
LIBS_ULOG=

CFLAGS_NFLOG= -I/usr/include/libnfnetlink
LIBS_NFLOG= -lnfnetlink

CFLAGS_PCAP=
LIBS_PCAP=-lpcap

CFLAGS_NVML= -I/usr/include/nvidia/gdk
LIBS_NVML= -lnvidia-ml

CFLAGS_CUMULUS=
LIBS_CUMULUS=

CFLAGS_OVS=
LIBS_OVS=

CFLAGS_OS10=
LIBS_OS10=

# common CFLAGS and LIBS	
CFLAGS = $(CFLAGS_HSFLOWD) $(CFLAGS_SHARED) $(OPT) -D_GNU_SOURCE -DHSP_VERSION=$(VERSION)
CFLAGS += -DUTHEAP
CFLAGS += -DHSP_OPTICAL_STATS
CFLAGS += -DHSP_MOD_DIR=$(MODDIR)
CFLAGS += -Wall -Wstrict-prototypes -Wunused-value
CFLAGS += -Wunused-function
LIBS=

# Check to see if container counters are under /sys/fs/cgroup/cpuacct/system.slice/ (Fedora)
# or /sys/fs/cgroup/cpuacct/docker/ (Ubuntu). The docker directory on Ubuntu only
# appears when the first container starts, so test for system.slice here.  This
# difference stems originally from the fact that Ubuntu systemd does not use slices.
ifndef SYSTEM_SLICE
  SYSTEM_SLICE=$(shell if test -r /sys/fs/cgroup/cpuacct/system.slice; then echo "yes"; else echo "no"; fi)
endif
ifeq ($(SYSTEM_SLICE),yes)
  CFLAGS += -DHSP_SYSTEM_SLICE
endif

# xs.h became xenstore.h by 6.5. Detect that and set a flag.  Could drive this by reading
# /etc/redhat-release to get the version number as in xenserver-ddk/Makefile,  but then it
# might not work properly for a XenSource build.  Seems safer to just check for the file:
ifndef XENSTORE_H	
  XENSTORE_H=$(shell if ls /usr/include/xenstore.h 2>&1 >/dev/null; then echo "yes"; else echo "no"; fi)
endif
ifeq ($(XENSTORE_H),yes)
  CFLAGS_XEN += -DHSP_XENSTORE_H
endif

#########  platform checks  #########

# detect Debian systems
ifndef DEBIAN
  DEBIAN=$(shell if [ -r /etc/debian_version ]; then echo "yes"; else echo "no"; fi)
endif

# detect Red Hat systems
ifndef REDHAT
  REDHAT=$(shell if [ -r /etc/redhat-release ]; then echo "yes"; else echo "no"; fi)
endif

HSFLOWD_INITSCRIPT=scripts/hsflowd
HSFLOWD_SYSTEMD_SERVICE=scripts/hsflowd.service

ifeq ($(DEBIAN),yes)
  HSFLOWD_INITSCRIPT=scripts/hsflowd.deb
endif
ifeq ($(REDHAT),yes)
  HSFLOWD_INITSCRIPT=scripts/hsflowd.redhat
endif

INSTALL=install

#########  object files  #########

OBJS_HSFLOWD= hsflowconfig.o \
              hsflowd.o \
	      evbus.o \
              util.o \
	      readInterfaces.o \
              readCpuCounters.o \
              readMemoryCounters.o \
              readDiskCounters.o \
              readHidCounters.o \
              readNioCounters.o \
	      readTcpipCounters.o \
	      readPackets.o

OBJS_JSON=mod_json.o
OBJS_DNSSD=mod_dnssd.o
OBJS_XEN=mod_xen.o
OBJS_KVM=mod_kvm.o
OBJS_DOCKER=mod_docker.o
OBJS_ULOG=mod_ulog.o
OBJS_NFLOG=mod_nflog.o
OBJS_PCAP=mod_pcap.o
OBJS_NVML=mod_nvml.o
OBJS_OVS=mod_ovs.o
OBJS_CUMULUS=mod_cumulus.o
OBJS_OS10=mod_os10.o

BUILDTGTS=hsflowd \
          mod_json.so \
          mod_dnssd.so \
          $(XTGTS)

all: $(BUILDTGTS)

#########  feature dependencies #########

XEN: mod_xen.so OVS

KVM: mod_kvm.so

DOCKER: mod_docker.so

ULOG: mod_ulog.so

NFLOG: mod_nflog.so

PCAP: mod_pcap.so

NVML: mod_nvml.so

OVS: mod_ovs.so

CUMULUS: mod_cumulus.so

OS10: mod_os10.so

#########  hsflowd  #########

hsflowd: $(OBJS_HSFLOWD)
	$(CC) $(CFLAGS) -o $@ $(OBJS_HSFLOWD) $(LIBS) $(LIBS_HSFLOWD) -rdynamic

#########  modules  #########

mod_dnssd.o: mod_dnssd.c $(HEADERS)
	$(CC) $(CFLAGS) -c $*.c $(CFLAGS_DNSSD)

mod_dnssd.so: $(OBJS_DNSSD)
	$(LD) -o $@ $(OBJS_DNSSD) $(LDFLAGS_SHARED) $(LIBS_DNSSD)

#----------------------------

mod_json.o: mod_json.c $(HEADERS)
	$(CC) $(CFLAGS) -c $*.c $(CFLAGS_JSON)

mod_json.so: $(OBJS_JSON)
	$(LD) -o $@ $(OBJS_JSON) $(LDFLAGS_SHARED)

#----------------------------


mod_ulog.o: mod_ulog.c $(HEADERS)
	$(CC) $(CFLAGS) -c $*.c $(CFLAGS_ULOG)

mod_ulog.so: $(OBJS_ULOG)
	$(LD) -o $@ $(OBJS_ULOG) $(LDFLAGS_SHARED) $(LIBS_ULOG)

#----------------------------


mod_nflog.o: mod_nflog.c $(HEADERS)
	$(CC) $(CFLAGS) -c $*.c  $(CFLAGS_NFLOG)

mod_nflog.so: $(OBJS_NFLOG)
	$(LD) -o $@ $(OBJS_NFLOG) $(LDFLAGS_SHARED) $(LIBS_NFLOG)

#----------------------------


mod_pcap.o: mod_pcap.c $(HEADERS)
	$(CC) $(CFLAGS) -c $*.c $(CFLAGS_PCAP)

mod_pcap.so: $(OBJS_PCAP)
	$(LD) -o $@ $(OBJS_PCAP) $(LDFLAGS_SHARED) $(LIBS_PCAP)

#----------------------------


mod_xen.o: mod_xen.c $(HEADERS)
	$(CC) $(CFLAGS) -c $*.c $(CFLAGS_XEN)

mod_xen.so: $(OBJS_XEN)
	$(LD) -o $@ $(OBJS_XEN) $(LDFLAGS_SHARED) $(LIBS_XEN)

#----------------------------


mod_kvm.o: mod_kvm.c $(HEADERS)
	$(CC) $(CFLAGS) -c $*.c $(CFLAGS_KVM)

mod_kvm.so: $(OBJS_KVM)
	$(LD) -o $@ $(OBJS_KVM) $(LDFLAGS_SHARED) $(LIBS_KVM)

#----------------------------


mod_docker.o: mod_docker.c $(HEADERS)
	$(CC) $(CFLAGS) -c $*.c $(CFLAGS_DOCKER)

mod_docker.so: $(OBJS_DOCKER)
	$(LD) -o $@ $(OBJS_DOCKER) $(LDFLAGS_SHARED) $(LIBS_DOCKER)

#----------------------------


mod_nvml.o: mod_nvml.c $(HEADERS)
	$(CC) $(CFLAGS) -c $*.c $(CFLAGS_NVML)

mod_nvml.so: $(OBJS_NVML)
	$(LD) -o $@ $(OBJS_NVML) $(LDFLAGS_SHARED) $(LIBS_NVML)

#----------------------------


mod_ovs.o: mod_ovs.c $(HEADERS)
	$(CC) $(CFLAGS) -c $*.c $(CFLAGS_OVS)

mod_ovs.so: $(OBJS_OVS)
	$(LD) -o $@ $(OBJS_OVS) $(LDFLAGS_SHARED) $(LIBS_OVS)

#----------------------------


mod_cumulus.o: mod_cumulus.c $(HEADERS)
	$(CC) $(CFLAGS) -c $*.c $(CFLAGS_CUMULUS)

mod_cumulus.so: $(OBJS_CUMULUS)
	$(LD) -o $@ $(OBJS_CUMULUS) $(LDFLAGS_SHARED) $(LIBS_CUMULUS)

#----------------------------


mod_os10.o: mod_os10.c $(HEADERS)
	$(CC) $(CFLAGS) -c $*.c $(CFLAGS_OS10)

mod_os10.so: $(OBJS_OS10)
	$(LD) -o $@ $(OBJS_OS10) $(LDFLAGS_SHARED) $(LIBS_OS10)


#########  install  #########

install:
	$(INSTALL) -d $(BIN_D)
	$(INSTALL) -m 700 hsflowd $(BIN_D)
	$(INSTALL) -d $(CONF_D)
	$(INSTALL) -m 644 scripts/hsflowd.conf $(CONF_D)
	$(INSTALL) -d $(SYSTEMD_D)
	$(INSTALL) -m 644 $(HSFLOWD_SYSTEMD_SERVICE) $(SYSTEMD_D)/hsflowd.service
	$(INSTALL) -d $(INIT_D)
	$(INSTALL) -m 755 $(HSFLOWD_INITSCRIPT) $(INIT_D)/hsflowd
	$(INSTALL) -d $(MOD_D)
	$(INSTALL) -m 755 *.so $(MOD_D)

#########  schedule  #########

ifndef SYSTEMD
  SYSTEMD=$(shell if hash systemctl 2>/dev/null; then echo "yes"; else echo "no"; fi)
endif

schedule:
ifeq ($(SYSTEMD),yes)
	systemctl daemon-reload
	systemctl enable hsflowd
else
  ifeq ($(DEBIAN),yes)
	update-rc.d hsflowd defaults
  else
	/sbin/chkconfig --add hsflowd
	/sbin/chkconfig --list hsflowd
  endif
endif

#########  clean   #########

clean: 
	rm -f hsflowd *.o *.so

#########  dependencies  #########

.c.o: $(HEADERS)
	$(CC) $(CFLAGS) -c $*.c

evbus.o: evbus.c $(HEADERS)
hsflowconfig.o: hsflowconfig.c $(HEADERS)
hsflowd.o: hsflowd.c $(HEADERS)
readInterfaces.o: readInterfaces.c $(HEADERS)
readCpuCounters.o: readCpuCounters.c $(HEADERS)
readMemoryCounters.o: readMemoryCounters.c $(HEADERS)
readDiskCounters.o: readDiskCounters.c $(HEADERS)
readHidCounters.o: readHidCounters.c $(HEADERS)
readNioCounters.o: readNioCounters.c $(HEADERS)
readPackets.o: readPackets.c $(HEADERS)
readContainerCounters.o: readContainerCounters.c $(HEADERS)
readTcpipCounters.o: readTcpipCounters.c $(HEADERS)
mod_json.o: mod_json.c $(HEADERS)
mod_dnssd.o: mod_dnssd.c $(HEADERS)
mod_xen.o: mod_xen.c $(HEADERS)
mod_kvm.o: mod_kvm.c $(HEADERS)
mod_docker.o: mod_docker.c $(HEADERS)
mod_ulog.o: mod_ulog.c $(HEADERS)
mod_nflog.o: mod_nflog.c $(HEADERS)
mod_pcap.o: mod_pcap.c $(HEADERS)
mod_nvml.o: mod_nvml.c $(HEADERS)
mod_cumulus.o: mod_cumulus.c $(HEADERS)
mod_ovs.o: mod_ovs.c $(HEADERS)
mod_os10.o: mod_os10.c $(HEADERS)

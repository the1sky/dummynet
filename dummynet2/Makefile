# $Id: Makefile 5511 2010-02-26 10:42:15Z svn_magno $
# gnu Makefile to build linux/Windows module for ipfw+dummynet.
#
# The defaults are set to build without modifications on PlanetLab
# and possibly 2.6 versions.
# On Windows, we use gnu-make and MSC

# Some variables need to have specific names, because they are used
# by the build infrastructure on Linux and OpenWrt. They are:
# 
#   ccflags-y	additional $(CC) flags
#   M		used by Kbuild, we must set it to `pwd`
#   obj-m	list of .o modules to build
#   $(MOD)-y	for each $MOD in obj-m, the list of objects
#   obj-y	same as above, for openwrt
#   O_TARGET	the link target, for openwrt
#   EXTRA_CFLAGS as the name says... in openwrt
#   EXTRA_CFLAGS is used in 2.6.22 module kernel compilation too
#   KERNELPATH	the path to the kernel sources or headers
#
# Not sure about this (the name might be reserved)
#   ipfw-cflags		our flags for building the module
#
# Other variables are only private and can be renamed. They include:
#
#   VER		linux version we are building for (2.4 2.6 or openwrt)
#---

UNAME:=$(shell uname)
$(warning including dummynet/Makefile)

# lets default for 2.6 for planetlab builds
VER ?= 2.6

#--- General values for all types of build ---
# obj-m is the target module
obj-m := ipfw_mod.o

#-- the list of source files. IPFW_SRCS is our own name.
# Original ipfw and dummynet sources + FreeBSD stuff,
IPFW_SRCS := ip_fw2.c ip_fw_pfil.c ip_fw_sockopt.c
IPFW_SRCS += ip_fw_dynamic.c ip_fw_table.c ip_fw_log.c
IPFW_SRCS += radix.c in_cksum.c
IPFW_SRCS += ip_dummynet.c ip_dn_io.c ip_dn_glue.c
IPFW_SRCS += dn_heap.c
IPFW_SRCS += dn_sched_fifo.c dn_sched_wf2q.c
IPFW_SRCS += dn_sched_rr.c dn_sched_qfq.c
# Module glue and functions missing in linux
IPFW_SRCS += ipfw2_mod.c bsd_compat.c

# generic cflags used on all systems
#ipfw-cflags += -DIPFW_HASHTABLES
ipfw-cflags += -DIPFIREWALL_DEFAULT_TO_ACCEPT
# _BSD_SOURCE enables __FAVOR_BSD (udp/tcp bsd structs instead of posix)
ipfw-cflags += -D_BSD_SOURCE
ipfw-cflags += -DKERNEL_MODULE	# build linux kernel module
# the two header trees for empty and override files
ipfw-cflags += -I $(M)/include_e
ipfw-cflags += -I $(M)/include
ipfw-cflags += -include $(M)/../glue.h	# headers
ipfw-cflags += -include $(M)/missing.h	# headers

$(warning "---- Building dummynet kernel module for Version $(VER)")

ifneq (,$(findstring CYGWIN,$(shell uname)))
    ISWIN=1
endif
ifneq ($(TCC),)
    ISWIN=1
endif
ifeq ($(ISWIN),1)
    M ?= $(shell pwd)
    WIN_SRCS += md_win.c
    WIN_SRCS += miniport.c protocol.c passthru.c debug.c
    #compiler, linker, target, sources and objects
    DDK = C:/WinDDK/7600.16385.0
    OBJDIR=objchk_wxp_x86/i386/

    TARGET = ipfw

    CSOURCES = $(IPFW_SRCS) $(WIN_SRCS)

    COBJS := $(CSOURCES:.c=.obj)
    COBJS := $(addprefix $(OBJDIR),$(COBJS))

    #include paths
    INCLUDE_PATHS = -Ii386 -Iinclude -Iinclude_e -I.
    # INCLUDE_PATHS += -I$(OBJDIR)
    INCLUDE_PATHS += -I$(DDK)/inc/api
    INCLUDE_PATHS += -I$(DDK)/inc/ddk
    INCLUDE_PATHS += -I$(DDK)/inc/crt

    # #preprocessor MS defines
    PREPROC  = -D_X86_=1 -Di386=1 -DSTD_CALL -DCONDITION_HANDLING=1
    PREPROC += -DNT_UP=0 -DNT_INST=0 -DWIN32=100 -D_NT1X_=100 -DWINNT=1
    PREPROC += -D_WIN32_WINNT=0x0501 -DWINVER=0x0501 -D_WIN32_IE=0x0603
    PREPROC += -DWIN32_LEAN_AND_MEAN=1 
    PREPROC += -D__BUILDMACHINE__=WinDDK -DFPO=0 -D_DLL=1
    PREPROC += -DNDIS_MINIPORT_DRIVER -DNDIS_WDM=1
    PREPROC += -DNDIS51_MINIPORT=1 -DNDIS51=1
    PREPROC += -DMSC_NOOPT -DNTDDI_VERSION=0x05010200
    PREPROC += -DKMDF_MAJOR_VERSION_STRING=01 -DKMDF_MINOR_VERSION_STRING=009
    #PREPROC += -DDBG=1 #debug
    PREPROC += -DNDEBUG #always up, seems no effect, possibly no debug?
    PREPROC += -DDEVL=1 #always up, seems no effect
    #macroing module name, WARNING: must match the one in .inf files
    PREPROC += -DMODULENAME=Ipfw 

    #our defines
    OUR_PREPROC  = -D_KERNEL -DKERNEL_MODULE -DKLD_MODULE
    OUR_PREPROC += -D__BSD_VISIBLE -DIPFIREWALL_DEFAULT_TO_ACCEPT
    OUR_PREPROC += -D__LITTLE_ENDIAN -DSYSCTL_NODE -DEMULATE_SYSCTL

ifeq ($(TCC),)
    CC = $(DDK)/bin/x86/x86/cl.exe
    LD = $(DDK)/bin/x86/x86/link.exe
    # #complier options
    CFLAGS  = -Fo$(OBJDIR)  -c -FC -Zc:wchar_t-
    CFLAGS += -Zl -Zp8 -Gy -Gm- -GF -cbstring -Gz -hotpatch -EHs-c-
    CFLAGS += -W2 # -W3 gives too many conversion errors
    CFLAGS += -GR- -GF -GS -Zi	# XXX do we need this ?
    CFLAGS += -Fd$(OBJDIR)
    CFLAGS += -wd4603 -wd4627 -typedil-
    CFLAGS += -FI $(DDK)/inc/api/warning.h
    CFLAGS += -FI winmissing.h
    CFLAGS += -FI missing.h	# headers
    CFLAGS += -FI ../glue.h	# headers

    #optimization options
    OPTIMIZE = -Od -Oi -Oy-

    #linker options
    LDFLAGS  = /MERGE:_PAGE=PAGE /MERGE:_TEXT=.text
    LDFLAGS += /SECTION:INIT,d /OPT:REF /OPT:ICF
    LDFLAGS += /IGNORE:4198,4010,4037,4039,4065,4070,4078,4087,4089,4221
    LDFLAGS += /INCREMENTAL:NO /release /NODEFAULTLIB /WX
    LDFLAGS += /debug /debugtype:cv,fixup,pdata
    LDFLAGS += /version:6.1 /osversion:6.1 /functionpadmin:5
    LDFLAGS += /safeseh /pdbcompress
    LDFLAGS += /STACK:0x40000,0x1000 /driver /base:0x10000 /align:0x80
    LDFLAGS += /stub:$(DDK)\\lib\\wxp\\stub512.com
    LDFLAGS += /subsystem:native,5.01 /entry:GsDriverEntry@8
    LDFLAGS += /out:$(OBJDIR)/ipfw.sys

    #libraries to build against
    LIBS  = $(DDK)/lib/wxp/i386/BufferOverflowK.lib
    LIBS += $(DDK)/lib/wxp/i386/ntoskrnl.lib
    LIBS += $(DDK)/lib/wxp/i386/hal.lib
    LIBS += $(DDK)/lib/wxp/i386/wmilib.lib
    LIBS += $(DDK)/lib/wxp/i386/ndis.lib
    LIBS += $(DDK)/lib/wxp/i386/sehupd.lib
else
    # TCC points to the root of tcc tree
    CC=$(TCC)/bin/wintcc
    EXTRA_CFLAGS += -DTCC -I..
    EXTRA_CFLAGS += -I$(TCC)/include/winapi -I$(TCC)/include
    EXTRA_CFLAGS += -nostdinc

    CFLAGS += -include winmissing.h -include missing.h -include ../glue.h
    CFLAGS += -I../../inc/api -I../../inc/ddk -I../../inc/crt
    CFLAGS += -DRC_INVOKED
endif

    #empty include directory to be built
    M ?= $(shell pwd)
    EDIRS += asm linux
    EFILES += asm/div64.h
    EFILES += linux/if.h linux/random.h linux/errno.h
    EFILES += net/if_types.h net/inet_hashtables.h net/route.h

    #targets
all: $(TARGET)

$(TARGET): include_e
	rm -rf objchk_wxp_x86
	mkdir -p objchk_wxp_x86/i386
	$(CC) $(INCLUDE_PATHS) $(PREPROC) $(OUR_PREPROC) $(CFLAGS) $(OPTIMIZE) $(CSOURCES)
	$(LD) $(LDFLAGS) $(COBJS) $(LIBS)

else # !windows

# We have three sections for OpenWrt, Linux 2.4 and Linux 2.6

ifeq ($(VER),openwrt)
  #--- The Makefile section for openwrt ---
  # We do not include a dependency on include_e as it is called
  # by Makefile.openwrt in Build/Prepare
  M=.
  obj-y := $(IPFW_SRCS:%.c=%.o)
  O_TARGET := $(obj-m)

  # xcflags-y is a temporary variable where we store build options
  xcflags-y += -O1 -DLINUX_24
  xcflags-y += -g

  EXTRA_CFLAGS := $(xcflags-y) $(ipfw-cflags)

  # we should not export anything
  #export-objs := ipfw2_mod.o
-include $(TOPDIR)/Rules.make

else	# !openwrt, below we do linux builds for 2.4 and 2.6

  # KERNELPATH is where the kernel headers reside. On PlanetLab
  # it is set already by the build system.
  # We can override it from the command line, or let the system guess.

ifneq ($(shell echo $(VER)|grep '2.4'),)
  # Makefile section for the linux 2.4 version
  # tested on linux-2.4.35.4, does not work with 2.4.37
  #
  # guess the kernel path -- or is it under /lib/modules ?
  KERNELPATH ?= /usr/src/`uname -r`

  # We need to figure out the gcc include directory, if not
  # set by the user through MYGCC_INCLUDE
  # Find compiler version (3rd field in last line returned by gcc -v)
  # e.g.	gcc version 4.3.2 (Debian 4.3.2-1.1)
  MYGCC_VER ?= $(shell $(CC) -v 2>&1 |tail -n 1 | cut -d " " -f 3)
  # We don't know the exact directory under /usr/lib/gcc so we guess
  MYGCC_INCLUDE ?= $(shell echo /usr/lib/gcc/*/$(MYGCC_VER) | cut -d " " -f 1)/include
  $(warning "---- gcc includes guessed to $(MYGCC_INCLUDE)")

  # additional warning
  WARN += -Wall -Wundef
  WARN += -Wstrict-prototypes -Wno-trigraphs -fno-strict-aliasing
  WARN += -fno-common -Werror-implicit-function-declaration
  # WARN += -O2  -fno-stack-protector -m32 -msoft-float -mregparm=3
  # -mregparm=3 gives a printk error
  WARN += -m32 -msoft-float # -mregparm=3
  #WARN += -freg-struct-return -mpreferred-stack-boundary=2
  WARN += -Wno-sign-compare
  WARN += -Wdeclaration-after-statement
  ifneq ($(MYGCC_VER),3.4.6)
        WARN += -Wno-pointer-sign
  endif

  ccflags-y += -O1 -DLINUX_24
  CFLAGS = -DMODULE -D__KERNEL__ -nostdinc \
	-isystem ${KERNELPATH}/include -isystem $(MYGCC_INCLUDE) \
	${ccflags-y}
  # The Main target
all: mod24

else # !2.4 --

  # This is the Makefile section for Linux 2.6.x including planetlab

ifeq ($(IPFW_PLANETLAB),1)
  $(warning "---- Building for PlanetLab")
  ipfw-cflags += -DIPFW_PLANETLAB        # PlanetLab compilation
endif
  # if not set, use the version from the installed system
  KERNELPATH ?= /lib/modules/`uname -r`/build
  # Otherwise, if you have kernel sources, try something like this:
  #KERNELPATH = /usr/src/linux-2.6.22
  $(warning "---- Building Version 2.6 $(VER) in $(KERNELPATH)")
  WARN := -O1 -Wall -Werror -DDEBUG_SPINLOCK -DDEBUG_MUTEXES
  # The main target

  # Required by kernel <= 2.6.22, ccflags-y is used on newer version
  LINUX_VERSION_CODE := $(shell grep LINUX_VERSION_CODE $(KERNELPATH)/include/linux/version.h|cut -d " " -f3)
  ifeq ($(shell if [ -z $(LINUX_VERSION_CODE) ] ; then echo "true"; fi),true)
    $(warning "---- Perhaps you miss a (cd $(KERNELPATH); make oldconfig; make prepare; make scripts)");
  endif
  ifeq ($(shell if [ $(LINUX_VERSION_CODE) -le 132630 ] ; then echo "true"; fi),true)
    EXTRA_CFLAGS += $(ccflags-y)
  endif

all: include_e
	$(MAKE) -C $(KERNELPATH) V=1 M=`pwd` modules
endif # !2.4

#-- back to the common section of code for Linux 2.4 and 2.6

# the list of objects used to build the module
ipfw_mod-y = $(IPFW_SRCS:%.c=%.o)

# additional $(CC) flags
ccflags-y += $(WARN)
ccflags-y += $(ipfw-cflags)
# if we really want debug symbols...
ccflags-y += -g

mod24: include_e $(obj-m)

$(obj-m): $(ipfw_mod-y)
	$(LD) $(LDFLAGS) -m elf_i386 -r -o $@ $^

# M is the current directory, used in recursive builds
# so we allow it to be overridden
M ?= $(shell pwd)
endif # !openwrt
endif # !windows

#--- various common targets
clean:
	-rm -f *.o *.ko Module.symvers *.mod.c
	-rm -rf objchk_wxp_x86
	-rm -rf include_e

distclean: clean
	-rm -f .*cmd modules.order opt_*
	-rm -rf .tmp_versions include_e
	-rm -rf .*.o.d _CL_*

# support to create empty dirs and files in include_e/
# EDIRS is the list of directories, EFILES is the list of files.

EDIRS  += altq arpa machine net netinet netinet6 sys

EFILES += opt_inet6.h opt_ipfw.h opt_ipsec.h opt_mpath.h
EFILES += opt_mbuf_stress_test.h opt_param.h

EFILES += altq/if_altq.h
EFILES += arpa/inet.h
EFILES += machine/in_cksum.h
EFILES += net/ethernet.h net/netisr.h net/pf_mtag.h
EFILES += net/bpf.h net/if_types.h
EFILES += net/vnet.h

EFILES += netinet/ether.h netinet/icmp6.h netinet/if_ether.h
EFILES += netinet/in.h netinet/in_pcb.h netinet/in_var.h
EFILES += netinet/in_systm.h
EFILES += netinet/ip_carp.h netinet/ip_var.h netinet/pim.h
EFILES += netinet/sctp.h netinet/tcp_timer.h netinet/tcpip.h
EFILES += netinet/udp_var.h

EFILES += netinet6/ip6_var.h

EFILES += sys/_lock.h sys/_rwlock.h sys/_mutex.h sys/jail.h
EFILES += sys/condvar.h sys/eventhandler.h sys/domain.h
EFILES += sys/limits.h sys/lock.h sys/mutex.h sys/priv.h
EFILES += sys/proc.h sys/rwlock.h sys/socket.h sys/socketvar.h
EFILES += sys/sysctl.h sys/time.h sys/ucred.h

include_e:
	echo "running in $M"
	-@rm -rf $(M)/include_e opt_*
	-@mkdir -p $(M)/include_e
	-@(cd $(M)/include_e; mkdir -p $(EDIRS); touch $(EFILES) )


#--- some other targets for testing purposes
test_radix: test_radix.o radix.o
test_lookup: ip_fw_lookup.o
test_radix test_lookup: CFLAGS=-Wall -Werror -O1

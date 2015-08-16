# Contributor: Manuel Gaul <inkaine (at) hotmail (dot) com>
# Credits for the original kernel26-one-dev PKGBUILD go to Antti "Tera" Oja <antti.bofh@gmail.com>

pkgname=linux-one-dev
_basekernel=3.1
pkgver=${_basekernel}.rc4
pkgrel=1
_kernelname=${pkgname#linux}
pkgdesc="The Linux Kernel instable development version for the Acer Aspire One A110L"
arch=('i686')
license=('GPL2')
url="http://www.kernel.org"
groups=(one)
depends=('coreutils' 'linux-firmware' 'module-init-tools')
makedepends=('gcc>=4.5')
provides=('kernel26-one-dev')
conflicts=('kernel26-one-dev')
replaces=('kernel26-one-dev')
install=linux-one-dev.install
source=(ftp://ftp.kernel.org/pub/linux/kernel/v3.0/testing/linux-${pkgver/.r/-r}.tar.bz2
	'config'
	fix-i915.patch
	change-default-console-loglevel.patch)
md5sums=('cd28799ad61707da720f889dde4cf4f6'
	 '9b2e0a5363de4c2fb0fa9b409a428898'
	 '263725f20c0b9eb9c353040792d644e5'
	 '9d3c56a4b999c8bfbd4018089a62f662')

build() {
   # if the user hasn't set his makepkg.conf
   export CFLAGS="-march=atom -Os -pipe -fomit-frame-pointer"
   export CXXFLAGS="${CFLAGS}"
   export MAKEFLAGS="-j3"
   export CARCH="i686"

   export KARCH="x86"

   # get into the linux source directory and start some magic
   cd ${srcdir}/linux-${pkgver/.r/-r}

   # fix #19234 i1915 display size
   patch -Np1 -i "${srcdir}/fix-i915.patch"

   # set DEFAULT_CONSOLE_LOGLEVEL to 4 (same value as the 'quiet' kernel param)
   # remove this when a Kconfig knob is made available by upstream
   # (relevant patch sent upstream: https://lkml.org/lkml/2011/7/26/227)
   patch -Np1 -i "${srcdir}/change-default-console-loglevel.patch"

   # remove the extraversion from Makefile
   sed -i 's|^EXTRAVERSION = .*$|EXTRAVERSION =|g' Makefile

   echo load configuration
   cp ../config ./.config

   # set kernel version libpath to basekernel
   . ./.config
   make silentoldconfig
   # Uncomment to configure the kernel!
   # return 1
   echo making the kernel!
   make bzImage modules || return 1
}

package() {
   cd ${srcdir}/linux-${pkgver/.r/-r}

   KARCH="x86"

   # get kernel version
   _kernver="$(make kernelrelease)"

   mkdir -p ${pkgdir}/{lib/{firmware,modules},boot}
   # install our modules
   make INSTALL_MOD_PATH=${pkgdir} modules_install || return 1
   # install the kernel
   install -T System.map ${pkgdir}/boot/System.map${_kernelname}
   install -T arch/x86/boot/bzImage ${pkgdir}/boot/vmlinuz${_kernelname}

   # We need a decent /usr/src to build modules, so

  install -D -m644 Makefile \
    ${pkgdir}/usr/src/linux-${_kernver}/Makefile
  install -D -m644 kernel/Makefile \
    ${pkgdir}/usr/src/linux-${_kernver}/kernel/Makefile
  install -D -m644 .config \
    ${pkgdir}/usr/src/linux-${_kernver}/.config
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/include

  for i in acpi asm-generic config linux math-emu media net pcmcia scsi sound video; do
    cp -a include/$i ${pkgdir}/usr/src/linux-${_kernver}/include/
  done

  # copy files necessary for later builds, like nvidia and vmware
  cp Module.symvers ${pkgdir}/usr/src/linux-${_kernver}
  cp -a scripts ${pkgdir}/usr/src/linux-${_kernver}
  # fix permissions on scripts dir
  chmod og-w -R ${pkgdir}/usr/src/linux-${_kernver}/scripts
  #mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/.tmp_versions

  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/kernel

  cp arch/$KARCH/Makefile ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/
  if [ "$CARCH" = "i686" ]; then
    cp arch/$KARCH/Makefile_32.cpu ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/
  fi
  cp arch/$KARCH/kernel/asm-offsets.s ${pkgdir}/usr/src/linux-${_kernver}/arch/$KARCH/kernel/

  # add headers for lirc package
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video
  cp drivers/media/video/*.h  ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video/
  for i in bt8xx cpia2 cx25840 cx88 em28xx et61x251 pwc saa7134 sn9c102 uvc; do
   mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video/$i
   cp -a drivers/media/video/$i/*.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/video/$i
  done
  # add dm headers
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/md
  cp drivers/md/*.h  ${pkgdir}/usr/src/linux-${_kernver}/drivers/md
  # add inotify.h
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/include/linux
  cp include/linux/inotify.h ${pkgdir}/usr/src/linux-${_kernver}/include/linux/
  # add CLUSTERIP file for iptables
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/net/ipv4/netfilter/
  # add wireless headers
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/net/mac80211/
  cp net/mac80211/*.h ${pkgdir}/usr/src/linux-${_kernver}/net/mac80211/
  # add dvb headers for external modules
  # in reference to:
  # http://bugs.archlinux.org/task/9912
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/dvb-core
  cp drivers/media/dvb/dvb-core/*.h ${pkgdir}/usr/src/linux-${_kernver}/drivers/media/dvb/dvb-core/
  # add dvb headers for external modules
  # in reference to:
  # http://bugs.archlinux.org/task/11194
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/include/linux/dvb/
  cp include/linux/dvb/*.h ${pkgdir}/usr/src/linux-${_kernver}/include/linux/dvb/
  # add xfs and shmem for aufs building
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/fs/xfs
  mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/mm
  cp fs/xfs/xfs_sb.h ${pkgdir}/usr/src/linux-${_kernver}/fs/xfs/xfs_sb.h
  # add vmlinux
  cp vmlinux ${pkgdir}/usr/src/linux-${_kernver}
  # copy in Kconfig files
  for i in `find . -name "Kconfig*"`; do 
    mkdir -p ${pkgdir}/usr/src/linux-${_kernver}/`echo $i | sed 's|/Kconfig.*||'`
    cp $i ${pkgdir}/usr/src/linux-${_kernver}/$i
  done

  cd ${pkgdir}/usr/src/linux-${_kernver}/include && ln -s asm-$KARCH asm

  chown -R root.root ${pkgdir}/usr/src/linux-${_kernver}
  find ${pkgdir}/usr/src/linux-${_kernver} -type d -exec chmod 755 {} \;
  cd ${pkgdir}/lib/modules/${_kernver} && \
    (rm -f source build; ln -sf ../../../usr/src/linux-${_kernver} build)

  # set correct depmod command for install
  sed -i -e "s/KERNEL_VERSION=.*/KERNEL_VERSION=${_kernver}/g" $startdir/linux-one-dev.install

  # remove unneeded architectures
  rm -rf ${pkgdir}/usr/src/linux-${_kernver}/arch/{alpha,arm,arm26,avr32,blackfin,cris,frv,h8300,ia64,m32r,m68k,m68knommu,mips,mn10300,parisc,powerpc,ppc,s390,sh,sh64,sparc,sparc64,um,v850,xtensa} 

  # remove build and source links
  rm -f ${pkgdir}/lib/modules/${_kernver}/build
  # remove the firmware
  rm -rf ${pkgdir}/lib/firmware
}

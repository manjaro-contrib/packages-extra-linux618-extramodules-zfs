# Maintainer: Bernhard Landauer <bernhard@manjaro.org>
# Maintainer: Philip Müller <philm[at]manjaro[dot]org>

_linuxprefix=linux618

pkgname="${_linuxprefix}-zfs"
pkgver=2.3.4
pkgrel=0.1
pkgdesc='Kernel modules for the Zettabyte File System.'
arch=('x86_64')
url="http://zfsonlinux.org/"
license=('CDDL-1.0')
groups=("${_linuxprefix}-extramodules")
depends=("${_linuxprefix}" "zfs-utils=${pkgver}")
makedepends=("${_linuxprefix}-headers" "zfs-dkms=${pkgver}")
provides=("zfs=${pkgver}" "ZFS-MODULE=${pkgver}")
options=('!strip')

prepare() {
  mkdir -p zfs/${pkgver}/source_patched
  cp -av /usr/src/zfs-${pkgver}/* zfs/${pkgver}/source_patched
  cd zfs/${pkgver}/source_patched
  sed -i -e 's/6.17/6.18/' META
  # https://github.com/torvalds/linux/commit/4055526d35746ce8b04bfa5e14e14f28bb163186
  sed -i -e 's/ns->ops->type != CLONE_NEWUSER/ns->ns_type != CLONE_NEWUSER/' module/os/linux/spl/spl-zone.c
  sed -i -e 's/sha256_update/zfs_sha256_update/g' module/icp/algs/sha2/sha2_generic.c
  sed -i -e 's/sha512_update/zfs_sha512_update/g' module/icp/algs/sha2/sha2_generic.c
  sed -i -e 's/sha256_final/zfs_sha256_final/g' module/icp/algs/sha2/sha2_generic.c
  sed -i -e 's/sha512_final/zfs_sha512_final/g' module/icp/algs/sha2/sha2_generic.c
  # https://lwn.net/Articles/1034764/
  sed -i -e 's/page = nth_page(sg_page(aiter->iter_sg),/page = sg_page(aiter->iter_sg),/' module/os/linux/zfs/abd_os.c
  sed -i -e 's/aiter->iter_offset >> PAGE_SHIFT);/aiter->iter_offset >> PAGE_SHIFT;/' module/os/linux/zfs/abd_os.c
  sed -i -e 's/pg = nth_page(sg_page(sg), sgoff >> PAGE_SHIFT);/pg = sg_page(sg), sgoff >> PAGE_SHIFT;/' module/os/linux/zfs/abd_os.c
  # https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/commit/?id=9c5518f1bacf98b20c3ad0fa5873b4da92122ced
  # TODO
  cd ..
  ln -sfv source_patched source
}

build() {
    _kernver="$(cat /usr/src/${_linuxprefix}/version)"

    fakeroot dkms build --dkmstree "${srcdir}" -m zfs/${pkgver} -k ${_kernver}
}

package() {
    _kernver="$(cat /usr/src/${_linuxprefix}/version)"

    install -Dt "${pkgdir}/usr/lib/modules/${_kernver}/extramodules" -m644 zfs/${pkgver}/${_kernver}/${CARCH}/module/*

    # compress each module individually
    find "$pkgdir" -name '*.ko' -exec xz -T1 {} +

    # systemd module loading
    printf '%s\n' spl zfs |
    install -Dm 644 /dev/stdin "${pkgdir}/usr/lib/modules-load.d/${pkgname}.conf"
}

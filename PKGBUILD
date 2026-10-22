# Maintainer: Abuamar <abuamar.albadawi@gmail.com>
#
# libfprint resmi Arch + driver elanmoc2 untuk ELAN Match-on-Chip.
#
# Driver elanmoc2 karya Davide Depau (LGPL-2.1-or-later), belum masuk
# upstream. Paket ini menambahkannya tanpa mengubah driver lain, sehingga
# daftar device yang didukung sama dengan paket resmi, ditambah 04f3:0c00,
# 04f3:0c4c, dan 04f3:0c5e.
#
# 04f3:0c00 memakai paket enroll 6 byte. Sebagian firmware menolak format
# 7 byte yang dipakai driver asli, jadi 6 byte dipilih sebagai default yang
# paling aman untuk firmware yang belum diketahui.

pkgname=libfprint-elanmoc2
pkgver=1.94.100
pkgrel=1
pkgdesc="Library for fingerprint readers, with the elanmoc2 driver for ELAN Match-on-Chip (04f3:0c00/0c4c/0c5e)"
arch=(x86_64)
url="https://fprint.freedesktop.org/"
license=(LGPL-2.1-or-later)
depends=(libgcc glib2 glibc libgudev libgusb openssl pixman)
makedepends=(git glib2-devel gobject-introspection gtk-doc meson ninja
             python-cairo python-gobject systemd)
checkdepends=(cairo umockdev)
provides=(libfprint libfprint-2.so=2-64)
conflicts=(libfprint)
replaces=(libfprint)
groups=(fprint)
source=("https://gitlab.freedesktop.org/libfprint/libfprint/-/archive/v${pkgver}/libfprint-v${pkgver}.tar.gz"
        "elanmoc2-0c00.patch"
        "71-elanmoc2-04f3-0c00-power.rules")
sha256sums=('edc90e02f330a7595ceaf37f2c6ec32ed43541347fe936d3273b0bb2524fd19c'
            '976ea132653498972fc0b7a1569bf0545b81816207877c2344d520c53893f509'
            '0215b49c4916f02333814f79b7f95d8a532ddb6d7359f5590da1ffd47502a324')

prepare() {
  cd "libfprint-v${pkgver}"
  patch -p1 --forward < "${srcdir}/elanmoc2-0c00.patch"
}

build() {
  cd "libfprint-v${pkgver}"

  # Flag build Arch diambil dari makepkg.conf (CFLAGS/LDFLAGS sudah di-set
  # oleh makepkg). arch-meson tidak dipakai karena hanya tersedia di paket
  # pacman resmi Arch, bukan di semua turunan.
  meson setup build \
    --prefix=/usr \
    --libdir=/usr/lib \
    --buildtype=release \
    -Ddrivers=all \
    -Dinstalled-tests=false \
    -Ddoc=false \
    -Dudev_hwdb=enabled
  meson compile -C build
}

check() {
  cd "libfprint-v${pkgver}"
  meson test -C build --print-errorlogs
}

package() {
  cd "libfprint-v${pkgver}"
  meson install -C build --destdir "$pkgdir"

  # Sensor ini masuk daftar "known unsupported" pada hwdb autosuspend,
  # sehingga udev menyetel power/control=auto. Aturan ini mengembalikannya
  # ke "on" karena sensor tidak selalu bangun dari autosuspend.
  install -Dm644 "${srcdir}/71-elanmoc2-04f3-0c00-power.rules" \
    "$pkgdir/usr/lib/udev/rules.d/71-elanmoc2-04f3-0c00-power.rules"

  install -Dm644 "${srcdir}/elanmoc2-0c00.patch" \
    "$pkgdir/usr/share/doc/${pkgname}/elanmoc2-0c00.patch"
}

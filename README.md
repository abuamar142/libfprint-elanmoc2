# libfprint-elanmoc2

libfprint resmi Arch dengan tambahan driver `elanmoc2` untuk sensor
ELAN Match-on-Chip: `04f3:0c00`, `04f3:0c4c`, `04f3:0c5e`.

## Atribusi

**Driver `elanmoc2` bukan karya pemelihara paket ini.** Driver ditulis oleh
Davide Depau dan belum masuk upstream libfprint (MR
[!330](https://gitlab.freedesktop.org/libfprint/libfprint/-/merge_requests/330),
terbuka sejak 2021).

| Bagian | Penulis / sumber | Lisensi |
|---|---|---|
| `libfprint/drivers/elanmoc2/` | Davide Depau | LGPL-2.1-or-later |
| Detail paket enroll 6 byte untuk `0c00` | riset komunitas: [bassemabdelbaset](https://github.com/bassemabdelbaset/libfprint-elan-04f3-0c00-ubuntu), [JQx-999](https://github.com/JQx-999/ELAN-0c00-Support-Unofficial-) | — |
| Porting ke libfprint 1.94.100, perbaikan data test, pemaketan Arch | Abu Amar (`abuamar142`) | LGPL-2.1-or-later |

Yang dikerjakan pemelihara paket hanya:

1. Registrasi driver di `meson.build` (2 baris).
2. Regenerasi `data/autosuspend.hwdb`.
3. Menghapus `0c00`/`0c4c`/`0c5e` dari daftar "known unsupported" di
   `fprint-list-udev-hwdb.c`.
4. Paket Arch dan skrip pendukungnya.

Header copyright asli tetap utuh di dalam `elanmoc2.c` dan `elanmoc2.h`.
Isi `elanmoc2-0c00.patch` memuat kode Davide Depau sebagaimana adanya.

## Pasang

```bash
git clone https://github.com/abuamar142/libfprint-elanmoc2
cd libfprint-elanmoc2
makepkg -si
```

Belum tersedia di AUR (registrasi AUR sedang ditutup sementara).

Paket ini menggantikan `libfprint` (`conflicts`/`replaces`), jadi `fprintd`
tetap bekerja tanpa perubahan.

Setelah terpasang:

```bash
fprintd-enroll -f right-index-finger
fprintd-verify
```

## Catatan 0c00

`04f3:0c00` memakai paket enroll 6 byte. Sebagian firmware dilaporkan
menolak format 7 byte yang dipakai driver asli, jadi 6 byte dipilih sebagai
default yang paling aman untuk firmware yang belum diketahui.

Pada firmware `0283` (diuji), **kedua** format diterima:

| Firmware | 6 byte | 7 byte |
|---|---|---|
| `0281` (laporan rujukan) | bekerja | ditolak (klaim mereka) |
| `0283` (diuji di sini) | bekerja | bekerja |

Untuk memakai 7 byte, ubah satu baris di `elanmoc2-0c00.patch`:

```
{.vid = ELANMOC2_VEND_ID, .pid = 0x0c00, .driver_data = ELANMOC2_ALL_DEV},
```

lalu perbarui `sha256sums` di `PKGBUILD` dan jalankan `makepkg --printsrcinfo > .SRCINFO`.

## Lock screen lama → "too many attempts"

Driver asli memberi timeout tetap 10 detik pada transfer yang **menunggu
jari**. Kalau jari belum menempel, transfer dibatalkan (`transfer timed out`),
fprintd mengulang 3×, lalu Noctalia menampilkan "too many attempts" —
padahal tidak ada yang pernah dipindai.

Patch di repo ini membuat command yang menunggu pengguna (`identify`,
`enroll`) memakai timeout 0 dan berhenti hanya lewat `cancellable`, sama
seperti driver `elanmoc` di upstream. Command cepat tetap dibatasi 10 detik.

## Verifikasi

```
meson test -Ddrivers=all  ->  98 OK / 0 fail
```

Diuji pada libfprint v1.94.100 dan master `6f9479c3` (2026-09-02), serta pada
hardware: enroll, verify, unlock lock screen, dan login via greetd.

## Kembali ke libfprint resmi

```bash
sudo pacman -Rdd libfprint-elanmoc2
sudo pacman -S libfprint
sudo rm /usr/lib/udev/rules.d/71-elanmoc2-04f3-0c00-power.rules
```

Template sidik jari tersimpan di chip sensor, tidak ikut terhapus.

## Lisensi

LGPL-2.1-or-later, sama dengan libfprint. Lihat `LICENSE`.

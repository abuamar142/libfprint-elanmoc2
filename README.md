# libfprint-elanmoc2

libfprint resmi Arch dengan tambahan driver `elanmoc2` untuk sensor
ELAN Match-on-Chip: `04f3:0c00`, `04f3:0c4c`, `04f3:0c5e`.

Driver `elanmoc2` karya Davide Depau (LGPL-2.1-or-later), belum masuk
upstream. Paket ini menambahkannya tanpa mengubah driver lain, sehingga
daftar device yang didukung sama dengan paket resmi, ditambah tiga PID itu.

## Pasang

```bash
paru -S libfprint-elanmoc2
# atau
yay -S libfprint-elanmoc2
```

Paket ini menggantikan `libfprint` (`conflicts`/`replaces`), jadi `fprintd`
tetap bekerja tanpa perubahan.

Setelah terpasang:

```bash
fprintd-enroll -f right-index-finger
fprintd-verify
```

## Catatan 0c00

`04f3:0c00` memakai paket enroll 6 byte. Sebagian firmware menolak format
7 byte yang dipakai driver asli, jadi 6 byte dipilih sebagai default yang
paling aman untuk firmware yang belum diketahui. Pada firmware `0283`
kedua format diterima.

## Kembali ke libfprint resmi

```bash
sudo pacman -Rdd libfprint-elanmoc2
sudo pacman -S libfprint
sudo rm /usr/lib/udev/rules.d/71-elanmoc2-04f3-0c00-power.rules
```

Template sidik jari tersimpan di chip sensor, tidak ikut terhapus.

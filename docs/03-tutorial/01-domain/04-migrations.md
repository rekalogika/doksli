---
title: Migrations
---

Migrations adalah cara untuk mengelola perubahan skema database.

## Membuat Migrasi

Setelah memodifikasi file konfigurasi ORM, jalankan perintah berikut untuk
menghasilkan file migrasi:

```bash
$ make migrations-make
```

Perintah ini akan membuat file migrasi baru di direktori `migrations/`. File
migrasi ini berisi perintah SQL untuk membuat, mengubah, atau menghapus tabel
atau kolom di database.

Silakan cek isi file migrasi yang baru dibuat. Kadang perlu dilakukan
penyesuaian manual, misalnya jika menambahkan kolom baru yang `not null`, maka
perlu diberikan nilai default. Atau dengan strategi membuat kolom baru dengan
`null`, melakukan `update` pada data yang sudah ada, lalu mengubah kolom menjadi
`not null`.

## Menjalankan Migrasi

Setelah file migrasi sudah disesuaikan, jalankan perintah berikut untuk
menjalankan migrasi:

```bash
$ make migrations-migrate
```
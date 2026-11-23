# Sistem Pakar Pemilihan Spesifikasi Laptop

**[https://github.com/yurikofarhan/sistem-pakar-laptop](https://github.com/yurikofarhan/sistem-pakar-laptop)**

## Tentang Aplikasi

Aplikasi ini adalah sistem pakar berbasis web yang membantu mahasiswa memilih laptop sesuai kebutuhan. Daripada menebak-nebak dari daftar spesifikasi, pengguna cukup menjawab beberapa pertanyaan soal kebutuhan pemakaian, lalu sistem yang menyaring kandidatnya.

Alurnya berjalan seperti ini. Pengguna mengisi kebutuhan lewat wizard tiga langkah, mulai dari jenis pemakaian, aplikasi yang sering dipakai, sampai anggaran. Jawaban itu jadi fakta awal yang diproses dengan metode Forward Chaining. Sistem mencocokkannya dengan basis rule yang aktif, menyimpulkan kebutuhan perangkat seperti kelas prosesor, kapasitas RAM, dan jenis kartu grafis, kemudian mengevaluasi setiap laptop di katalog terhadap kebutuhan tersebut. Hasilnya berupa daftar rekomendasi yang paling sesuai, ditambah beberapa alternatif terdekat kalau anggaran mepet.

Dari sisi pengguna, ada fitur untuk memilih kebutuhan, melihat hasil rekomendasi, membandingkan beberapa kandidat, dan membuka riwayat konsultasi yang pernah dilakukan. Sisi admin punya dashboard tersendiri untuk mengelola rule, mengelola katalog laptop, mengelola pengguna, dan memantau riwayat konsultasi seluruh pengguna.

## Contoh Rule (If-Then)

Aturan sistem disimpan sebagai rule, dan tiap rule terdiri dari beberapa kondisi plus satu atau lebih kesimpulan. Semua kondisi dalam satu rule digabung dengan logika AND, dan rule dievaluasi dari prioritas tertinggi. Kesimpulan bertipe spesifikasi diberi awalan `rec_`, misalnya `rec_ram_gb` dan `rec_gpu`.

Bentuk umumnya seperti ini:

```text
IF <condition 1> AND <condition 2> AND ...
THEN <conclusion 1>, <conclusion 2>, ...
```

Contoh aturan untuk kebutuhan pemrograman yang berat:

```text
Rule  : Heavy Programming (priority 10)
IF    : usage            in   ["programming"]
        usage_intensity  =    "berat"
        graphics_need    =    "tinggi"
THEN  : rec_processor = "kelas-menengah"
        rec_ram_gb    = 16
        rec_gpu       = "dedicated"
```

Dalam bentuk data, kondisi dan kesimpulan disimpan terpisah. Field `operator` menentukan jenis pencocokan (`in` dan `not_in` menerima nilai berupa array, operator lain menerima nilai tunggal).

```json
{
  "name": "Heavy Programming",
  "activation": "aktif",
  "priority": 10,
  "conditions": [
    { "fact_key": "usage",           "operator": "in", "expected_value": ["programming"] },
    { "fact_key": "usage_intensity", "operator": "=",  "expected_value": "berat" },
    { "fact_key": "graphics_need",   "operator": "=",  "expected_value": "tinggi" }
  ],
  "conclusions": [
    { "conclusion_code": "rec_processor", "conclusion_value": "kelas-menengah" },
    { "conclusion_code": "rec_ram_gb",    "conclusion_value": 16 },
    { "conclusion_code": "rec_gpu",       "conclusion_value": "dedicated" }
  ]
}
```

Ada juga rule untuk kebutuhan yang lebih ringan, yang praktis jadi cabang sebaliknya:

```text
Rule  : Light Usage (priority 5)
IF    : usage_intensity  =    "ringan"
        multitasking     =    "rendah"
THEN  : rec_ram_gb    = 8
        rec_gpu       = "integrated"
```

Kalau tidak ada kondisi yang cocok, rule itu sekadar dilewati dan sistem lanjut memeriksa rule berikutnya, jadi tidak ada rule yang wajib terpenuhi. Kesimpulan dari rule yang terpenuhi lalu digabung: untuk RAM dan Storage diambil kapasitas terbesar, untuk kelas prosesor diambil yang tertinggi, dan GPU dedicated dianggap lebih kuat daripada integrated.

## Dokumen

Semua dokumen ada di dalam folder `docs/`.

`SRS.docx` adalah dokumen utamanya, versi 1.0 bertanggal 28 September 2026. Isinya dibagi jadi beberapa bagian:

- **Pendahuluan** - latar belakang, ruang lingkup, dan referensi.
- **Gambaran umum** - fitur produk, jenis pengguna, lingkungan operasi, dan batasan.
- **Fitur sistem** - dua belas fitur dibahas satu per satu: beranda pengguna, pilih kebutuhan, proses forward chaining, hasil rekomendasi, riwayat rekomendasi, dashboard admin, pertanyaan dan pilihan konsultasi, kelola rule, kelola katalog laptop, riwayat rekomendasi admin, autentikasi dan kelola pengguna, serta antarmuka API.
- **Kebutuhan data** - tujuh entitas data beserta relasinya.
- **Antarmuka eksternal** - tampilan pengguna dan antarmuka perangkat keras, perangkat lunak, serta komunikasi.
- **Kebutuhan nonfungsional** - aspek usability, performa, keamanan, keandalan, maintainability, dan portabilitas.
- **Lampiran** - daftar istilah dan seluruh model analisis.

`Latar-Belakang.docx` memuat bagian pendahuluan laporan, meliputi latar belakang, rumusan masalah, analisis dan pemodelan sistem, perancangan antarmuka, serta implementasi dan fungsionalitas.

## Diagram

Semua diagram disimpan di folder `diagrams/`, dalam bentuk gambar PNG.

Diagram yang tersedia:

- **Use Case** - `usecase.png`, `usecase-user.png`, `usecase-admin.png`
- **Activity** - `activity.png`, `activity-auth.png`, `activity-hasil-banding.png`, `activity-admin-rule.png`, `activity-admin-laptop.png`, `activity-admin-user.png`
- **Sequence** - `sequence.png`, `sequence-auth.png`, `sequence-api.png`, `sequence-admin-dashboard.png`, `sequence-admin-rule.png`, `sequence-admin-laptop.png`, `sequence-admin-user.png`, `sequence-admin-session.png`
- **Class** - `class.png`
- **Object** - `object.png`
- **State Machine** - `state-session.png`, `state-rule.png`, `state-laptop.png`
- **Deployment** - `deployment.png`

## Teknologi

**Bahasa dan framework**

- Backend dibangun dengan PHP 8.3 atau lebih baru, memakai framework Laravel 13.
- Sisi tampilan memakai Vue 3 yang dijembatani oleh Inertia.js, dengan Tailwind CSS untuk penataan gaya.
- Proses build aset memakai Node.js dan Vite.

**Arsitektur**

- Bentuknya aplikasi web monolitik. Halaman dirender dari sisi server lewat Inertia, sementara pengolahan data dan proses inferensi berjalan di server.
- Basis data relasional yang mendukung kolom JSON. Untuk pengembangan dipakai SQLite, sedangkan produksi memakai PostgreSQL.
- Komunikasi browser dan server memakai HTTP di pengembangan dan HTTPS di produksi. Data dikirim sebagai payload Inertia untuk halaman dan JSON untuk API.
- Autentikasi halaman web memakai session, sedangkan API memakai token dari Sanctum.

## Struktur Folder

```text
.
|-- README.md
|-- docs/
|   |-- SRS.docx
|   `-- Latar-Belakang.docx
`-- diagrams/
    `-- *.png
```

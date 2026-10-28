# Panduan Melanjutkan Proyek

Dokumen ini adalah panduan kerja ringkas untuk melanjutkan proyek `alfazarqom/e-com` tanpa perlu mengulang konteks.

## Perintah utama

Kirim pesan berikut di ChatGPT:

```text
lanjutkan REQUIREMENT.md
```

Perintah tersebut berarti:
1. Buka `docs/REQUIREMENTS.md` pada branch kerja yang tercatat untuk proyek.
2. Baca keputusan yang sudah ditetapkan, status progres, dan keputusan yang masih menunggu.
3. Lanjutkan dari titik terakhir yang belum selesai.
4. Tampilkan pilihan keputusan yang masih diperlukan secara ringkas dan dikelompokkan.
5. Jangan meminta pengguna mengulang keputusan yang sudah terdokumentasi.

Nama perintah adalah singkatan percakapan; file kanonis yang harus dibaca adalah **`docs/REQUIREMENTS.md`**.

## Dokumen acuan

- `docs/REQUIREMENTS.md` — kebutuhan, keputusan, status, dan kriteria penerimaan.
- `README.md` — gambaran proyek dan panduan teknis tingkat tinggi.
- `docs/PROJECT-WORKFLOW.md` — aturan melanjutkan pekerjaan dan batas perubahan ini.

Jika dokumen berbeda, jangan diam-diam memilih salah satunya. Tunjukkan konflik, utamakan keputusan terbaru yang secara eksplisit disetujui pengguna, lalu catat penyelesaiannya.

## Alur kerja

Gunakan urutan berikut:

**Diskusi → Audit read-only → Rekomendasi → Konfirmasi requirement → Implementasi dengan persetujuan → Verifikasi.**

- Diskusi dan rekomendasi tidak berarti izin implementasi.
- Mulai dengan audit read-only saat perlu mengetahui kondisi repository, database, konfigurasi, atau deployment.
- Bedakan fakta yang ditemukan, keputusan pengguna, rekomendasi, dan asumsi yang belum diverifikasi.
- Jangan menyatakan fitur selesai hanya karena sudah tercatat sebagai requirement.
- Catat keputusan baru, alasan singkat, status implementasi, dan bukti verifikasi di dokumen yang sesuai.

## Persetujuan perubahan

Tanpa persetujuan eksplisit untuk tindakan tertentu, **jangan**:
- mengubah kode aplikasi;
- mengubah database, schema, migration, RLS, grants, atau data;
- membaca, menampilkan, atau mengubah nilai secret;
- melakukan deployment, merge, atau perubahan production;
- menjalankan transaksi pembayaran nyata.

Persetujuan hanya berlaku untuk lingkup yang disebutkan. Persetujuan menulis dokumentasi tidak otomatis mengizinkan perubahan kode, database, secrets, deployment, atau merge.

## Format status

Gunakan status terpisah:
- **Requirement:** belum dibahas / menunggu keputusan / ditetapkan.
- **Implementasi:** belum diaudit / belum ada bukti / sebagian / selesai berdasarkan bukti.
- **Verifikasi:** belum dijalankan / gagal / lulus dengan bukti.

Jangan menandai implementasi atau tes lulus tanpa hasil pemeriksaan yang dapat ditunjukkan.

## Kategori wajib saat membahas proyek

### Supabase Backend — Database & Edge Functions
Database, RLS/grants, Edge Functions, validasi transaksi, Duitku POP, ongkir, keamanan, dan tes backend.

### Frontend — Layout, Design, UI, Feature & Component
Layout, desain responsif, halaman, alur belanja, komponen, integrasi API, aksesibilitas, dan tes frontend.

Tampilkan kedua judul kategori ini pada setiap respons terkait proyek, meskipun salah satunya hanya diberi keterangan bahwa belum menjadi fokus.

## Status awal dokumen ini

- Kebutuhan utama V1 tercatat di `docs/REQUIREMENTS.md`.
- Kelompok keamanan L–O masih menunggu keputusan pengguna.
- Audit kesesuaian repository dan pengujian end-to-end belum dinyatakan selesai.
- Branch dokumentasi saat ini adalah `docs/v1-requirements-tracker`; belum berarti sudah digabung ke `main` atau di-deploy.

Terakhir diperbarui: 2026-10-28.

# Spesifikasi Kebutuhan V1 — E-Commerce Guest Checkout

> **Jenis dokumen:** catatan keputusan kebutuhan dan pelacak progres.  
> **Status:** dalam penyusunan; keputusan bisnis yang tercantum sudah dipilih dalam diskusi, tetapi implementasi dan pengujian belum diverifikasi.  
> **Terakhir diperbarui:** 2026-10-28.  
> **Batas perubahan:** dokumen ini tidak memberi izin untuk mengubah kode aplikasi, database, secrets, deployment, atau melakukan merge.

Dokumen ini menyimpan keputusan kebutuhan V1 agar diskusi dapat dilanjutkan dari titik terakhir. Perubahan status progres harus mencerminkan bukti yang tersedia; jangan menandai fitur sebagai selesai hanya karena kebutuhannya sudah diputuskan.

## 1. Status progres

| Area | Status kebutuhan | Status implementasi |
|---|---|---|
| Teknologi dan arsitektur umum | Ditetapkan sebagian | Belum diverifikasi secara menyeluruh |
| Katalog produk dan varian | Ditetapkan | Belum diverifikasi |
| Harga, ongkir, dan snapshot pesanan | Ditetapkan | Belum diverifikasi |
| Checkout guest dan stok | Ditetapkan | Belum diverifikasi |
| Pembayaran Duitku POP | Aturan bisnis dasar ditetapkan; kontrak resmi perlu diverifikasi | Belum diverifikasi |
| Status pesanan, pembayaran, pengiriman | Ditetapkan sebagai kebutuhan | Belum diverifikasi |
| Tracking pesanan tamu | Ditetapkan sebagai kebutuhan | Belum diverifikasi |
| Keamanan dan akses admin (Kelompok L–O) | **Menunggu keputusan** | Belum diverifikasi |
| Audit repository terhadap dokumen ini | Belum dilakukan sampai selesai | Belum diverifikasi |
| Pengujian end-to-end | Belum direncanakan secara final | Belum dijalankan/dikonfirmasi |

## 2. Teknologi dan ruang lingkup

- **Frontend:** React, Vite, TypeScript, Tailwind CSS, React Router, lucide-react, dan Zustand.
- **Backend:** Supabase Database (PostgreSQL) dan Supabase Edge Functions.
- **Pembayaran:** Duitku POP.
- **Pelanggan:** guest checkout; tidak menggunakan Supabase Auth untuk autentikasi pelanggan pada V1.
- **Pengiriman:** ongkir tetap Rp52.000 untuk semua alamat di Indonesia. V1 tidak membutuhkan API penyedia ekspedisi untuk menghitung ongkir.
- **Promo:** tidak ada kupon, diskon produk, atau promo gratis ongkir pada V1.
- **Dashboard admin:** untuk mengelola produk/varian, pesanan, pengiriman, dan penanganan konflik pembayaran/refund.
- **Sumber kebenaran:** backend memvalidasi harga, stok, ongkir, total, status pembayaran, dan transisi transaksi.

Teknologi atau fitur di atas adalah target kebutuhan, bukan pernyataan bahwa semuanya sudah tersedia di repository.

## 3. Katalog produk dan varian

Katalog mendukung produk sederhana dan produk dengan varian.

- Produk sederhana memiliki satu harga dan satu stok pada produk.
- Produk bervarian mengelola harga dan stok per varian; jangan menggandakan stok aktif pada produk induk dan varian.
- Setiap varian mempunyai ID stabil, SKU unik, harga, stok, atribut identitas, dan status aktif/nonaktif.
- Kombinasi atribut identitas dalam satu produk harus unik. SKU yang berbeda tidak boleh digunakan untuk menyamarkan kombinasi atribut duplikat.
- Validasi status aktif, hubungan varian dengan produk induk, harga, stok, keunikan SKU, dan keunikan kombinasi atribut dilakukan di backend.
- Snapshot item pesanan menyimpan nama produk, SKU, atribut varian, harga satuan, dan kuantitas saat pesanan dibuat.
- Referensi relasional menggunakan ID varian, bukan SKU.

### 3.1 Penghapusan dan penonaktifan

- **Tidak ada penghapusan permanen varian**, baik varian pernah digunakan dalam transaksi maupun belum pernah digunakan.
- Semua varian hanya dapat dinonaktifkan; varian nonaktif tidak tersedia untuk pesanan baru.
- Varian lama tetap tersedia untuk menjaga riwayat transaksi dan proses rekonsiliasi stok.
- Varian dapat diaktifkan kembali jika diizinkan kebijakan bisnis dan validasi backend.

### 3.2 Penguncian SKU dan atribut identitas

- Sebelum transaksi pertama, SKU dan atribut identitas boleh diubah jika tetap memenuhi aturan keunikan.
- Setelah transaksi pertama, SKU dikunci dan tidak boleh diubah.
- Setelah transaksi pertama, atribut identitas seperti warna, ukuran, dan atribut pembeda lain dikunci.
- Tidak ada mekanisme untuk mengganti SKU pada varian yang sudah pernah digunakan. Varian lama hanya dapat dinonaktifkan.
- Jika identitas atribut perlu berubah setelah transaksi pertama, buat varian baru dengan ID baru; jangan mengubah identitas varian lama.
- Harga dan stok bukan atribut identitas. Keduanya dapat dikelola menurut validasi dan aturan harga/stok di bagian lain dokumen.
- Penguncian harus ditegakkan pada jalur backend/database, bukan hanya pada form admin.

## 4. Harga, ongkir, dan total pesanan

### 4.1 Harga

- Harga katalog boleh berubah.
- Pesanan menyimpan snapshot harga satuan saat pesanan dibuat.
- Perubahan harga katalog tidak mengubah harga atau total pesanan yang sudah tercatat.
- Jika pesanan kedaluwarsa dan pelanggan membuat pesanan baru, pesanan baru menggunakan harga katalog terbaru yang divalidasi backend.
- Checkout harus mendapatkan validasi backend sebelum pesanan dibuat.

### 4.2 Ongkir dan total

- Ongkir V1 adalah **Rp52.000 untuk semua alamat di Indonesia**.
- Ongkir disimpan sebagai snapshot pada pesanan saat dibuat dan tidak dihitung ulang saat halaman pembayaran dibuka.
- Tidak ada promo gratis ongkir pada V1.
- Rumus total: **total pesanan = subtotal produk + Rp52.000**.
- Backend adalah sumber kebenaran untuk harga, subtotal, ongkir, dan total.
- Halaman pembayaran dan tracking menampilkan nilai pesanan tersimpan, bukan menghitung ulang menggunakan katalog terbaru.

## 5. Checkout guest dan pengelolaan stok

- Pelanggan dapat checkout tanpa akun pelanggan/Supabase Auth.
- Backend memvalidasi penuh harga, produk dan varian aktif, kuantitas, stok, ongkir, dan total ketika pesanan dibuat.
- Pembuatan pesanan dan pengurangan stok harus atomik. Jika satu item gagal divalidasi, tidak boleh ada pengurangan stok parsial.
- Pesanan menyimpan snapshot item dan nilai transaksi.
- Stok dikurangi ketika pesanan dibuat, sebelum pembayaran.
- Batas pembayaran adalah **24 jam sejak pesanan dibuat**.
- Jika pembayaran tidak diterima sebelum tenggat, backend mengakhiri pesanan/pembayaran yang tertunda dan memulihkan stok tepat satu kali.
- Operasi pengurangan dan pemulihan stok harus idempoten serta tahan terhadap callback duplikat, proses kedaluwarsa berulang, dan permintaan bersamaan.
- Pembatalan pesanan hanya boleh dilakukan admin; pelanggan tidak memiliki tombol pembatalan mandiri.
- Pembatalan atau kedaluwarsa tidak boleh menghapus riwayat transaksi.

## 6. Duitku POP dan rekonsiliasi pembayaran

Keputusan kebutuhan berikut bukan pengganti verifikasi kontrak resmi Duitku. Detail request, signature, callback, status, dan refund harus diperiksa terhadap dokumentasi merchant Duitku yang berlaku sebelum implementasi.

- Backend membuat dan memvalidasi invoice; frontend hanya memulai alur POP dan menampilkan hasil yang dikonfirmasi backend.
- Pelanggan menggunakan invoice yang sama selama invoice masih berlaku, belum kedaluwarsa, dan statusnya mengizinkan pembayaran.
- Backend memverifikasi signature callback dan kecocokan transaksi.
- Callback diproses secara idempoten. Callback duplikat tidak boleh menggandakan perubahan stok atau transaksi.
- Redirect/return URL, parameter browser, dan state lokal frontend bukan bukti pembayaran berhasil.
- Status pembayaran dan status pesanan adalah hal terpisah.
- Jika callback pembayaran valid datang setelah pesanan kedaluwarsa, backend merekonsiliasi transaksi dan memeriksa apakah semua stok dapat dipesan kembali secara atomik.
- Jika semua stok tersedia, backend dapat mengaktifkan kembali pesanan sesuai aturan rekonsiliasi.
- Jika stok tidak tersedia, pesanan **tidak diaktifkan kembali**. Konflik ditandai untuk admin dan refund menjadi pilihan penanganan yang diutamakan.
- Refund hanya dinyatakan berhasil setelah hasilnya diverifikasi. Jangan mengasumsikan adanya API refund otomatis tanpa konfirmasi dokumentasi resmi.
- Jika refund gagal, tandai kasus bermasalah dan eskalasi ke admin. Tidak ada retry refund otomatis yang sudah disepakati.
- Pelanggan melihat informasi konflik/refund melalui halaman tracking; notifikasi email tidak termasuk V1.

## 7. ID pesanan dan tracking tamu

- Backend membuat `orderId` menggunakan randomness kriptografis, tidak berurutan, dengan entropi yang direkomendasikan minimal 128 bit serta constraint unik di database.
- Tracking tersedia melalui input `orderId` manual dan URL `/tracking/{orderId}`.
- Keputusan V1 tidak menambahkan token akses tamu terpisah.
- **Peringatan keamanan:** `orderId` adalah identifier, bukan bukti kepemilikan. Karena itu, endpoint harus menerapkan rate limiting, respons dengan data minimum, pembatasan detail pribadi, dan penanganan permintaan yang tidak valid secara aman.
- Data tracking harus bersumber dari backend; frontend tidak menentukan status pembayaran atau total pesanan dari state lokal.

## 8. Status transaksi

Nama berikut adalah rancangan kebutuhan dan harus dicocokkan dengan implementasi aktual sebelum dianggap sebagai enum/database contract.

### 8.1 Status pesanan

Alur dasar: `menunggu_pembayaran` → `diproses` → `siap_dikirim` → `dikirim` → `selesai`.

Status alternatif/akhir: `dibatalkan`.

- Tidak ada status `menunggu_ongkir`, karena ongkir V1 tetap.
- Transisi harus divalidasi backend; pelanggan tidak dapat mengubah status pesanan.
- Konflik pembayaran terlambat perlu ditandai dan ditangani admin, tanpa menganggap pesanan otomatis aktif.

### 8.2 Status pembayaran

Status utama ringkas:
- `menunggu`
- `berhasil`
- `gagal`
- `kedaluwarsa`
- `refund`
- `bermasalah`

Detail refund disimpan terpisah, termasuk sekurang-kurangnya jumlah, status proses, waktu, referensi terkait, dan hasil verifikasi. Detail harus dapat membedakan refund sebagian dan refund penuh.

Jika refund penuh telah berhasil diverifikasi untuk konflik yang dibatalkan, keputusan sebelumnya menetapkan status pesanan `dibatalkan` dan status pembayaran utama `refund`.

### 8.3 Status pengiriman

- `menunggu_diproses`
- `sedang_disiapkan`
- `dikirim`
- `terkirim`
- `gagal`

Admin mencatat nomor resi secara manual. Status pengiriman tidak menggantikan status pembayaran.

## 9. Halaman dan pengalaman frontend

Halaman/fitur yang diperlukan untuk dicakup dalam rancangan frontend:
- Beranda dan katalog produk.
- Kategori dan pencarian.
- Detail produk serta pemilihan varian aktif.
- Keranjang dan checkout guest.
- Ringkasan pesanan, pembayaran Duitku POP, dan hasil pembayaran dari backend.
- Tracking pesanan melalui input orderId dan URL langsung.
- Dashboard admin untuk katalog/varian, pesanan, status pengiriman, nomor resi, konflik pembayaran, dan tindak lanjut refund.

Prinsip UI:
- Produk dan varian nonaktif tidak dapat dipilih untuk pesanan baru.
- SKU/atribut identitas varian terkunci ditampilkan read-only.
- Tidak ada tombol hapus permanen varian.
- Ringkasan pembayaran dan tracking menggunakan snapshot pesanan.
- Status refund yang belum terverifikasi tidak boleh ditampilkan sebagai berhasil.
- UI tidak menjadi sumber kebenaran untuk harga, stok, total, atau status pembayaran.

## 10. Keputusan keamanan yang masih menunggu

Kelompok berikut **belum dipilih oleh pengguna**. Jangan menganggap salah satu opsi sebagai keputusan final.

- [ ] **L — Autentikasi admin**
  - L1: pelanggan tetap guest; dashboard admin menggunakan autentikasi dan otorisasi aman.
  - L2: autentikasi admin dikelola dashboard terpisah; backend tetap memverifikasi akses.
- [ ] **M — Akses API publik**
  - M1: endpoint publik dibatasi untuk katalog dan operasi guest yang diperlukan; operasi sensitif divalidasi backend.
  - M2: operasi order, pembayaran, tracking, dan admin melewati Edge Functions; frontend tidak mengakses tabel transaksi secara langsung.
- [ ] **N — Audit tindakan admin**
  - N1: audit wajib untuk tindakan sensitif, mencatat aktor, tindakan, waktu, objek, dan hasil.
  - N2: audit minimum untuk pembayaran, refund, dan pembatalan.
- [ ] **O — Rate limiting**
  - O1: wajib untuk tracking, pembuatan pesanan, dan invoice.
  - O2: wajib untuk tracking dan invoice; endpoint lain mengikuti hasil audit risiko.

Rekomendasi untuk diskusi berikutnya adalah menyelesaikan Kelompok L–O. Setelah itu, lakukan audit read-only terhadap repository dan skema Supabase untuk menemukan perbedaan implementasi, tanpa menerapkan perubahan.

## 11. Kriteria penerimaan V1 yang perlu diuji

- [ ] Harga dan total dihitung serta divalidasi oleh backend.
- [ ] Ongkir selalu Rp52.000 dan tersimpan sebagai snapshot pesanan.
- [ ] Perubahan harga katalog tidak mengubah pesanan yang sudah dibuat.
- [ ] SKU dan kombinasi atribut unik; varian nonaktif ditolak saat checkout.
- [ ] SKU/atribut identitas varian terkunci setelah transaksi pertama.
- [ ] Tidak ada operasi penghapusan permanen varian.
- [ ] Kegagalan validasi salah satu item tidak meninggalkan pengurangan stok parsial.
- [ ] Callback Duitku dengan signature tidak valid ditolak.
- [ ] Callback duplikat tidak menggandakan efek transaksi.
- [ ] Invoice yang kedaluwarsa tidak diperlakukan sebagai invoice valid.
- [ ] Kedaluwarsa pesanan memulihkan stok tepat satu kali.
- [ ] Pembayaran terlambat dan konflik stok direkonsiliasi sesuai kebijakan.
- [ ] Tracking menggunakan orderId acak dan tidak membocorkan data berlebihan.
- [ ] Pelanggan tidak dapat membatalkan pesanan melalui endpoint publik.
- [ ] Status pesanan, pembayaran, dan pengiriman konsisten dan terpisah.
- [ ] Pengujian refund tidak mengklaim berhasil tanpa verifikasi hasil.

Daftar ini adalah target pengujian, bukan bukti bahwa tes sudah dijalankan atau lulus.

## 12. Riwayat dan aturan perubahan

Saat melanjutkan pekerjaan:
1. Periksa bagian status progres dan keputusan yang masih menunggu.
2. Catat keputusan baru dengan tanggal dan alasan singkat.
3. Bedakan keputusan kebutuhan, implementasi, dan hasil verifikasi.
4. Audit repository terlebih dahulu sebelum menyatakan implementasi sesuai.
5. Jangan mengubah kode, database, secrets, deployment, atau merge tanpa persetujuan eksplisit.
6. Jangan mengubah keputusan yang sudah ditetapkan tanpa keputusan baru yang jelas dari pengguna.

### Riwayat keputusan awal

- **2026-10-28:** keputusan kebutuhan katalog/varian, penghapusan varian, penguncian SKU dan atribut identitas, snapshot harga dan ongkir, validasi checkout, invoice valid, tracking orderId, penanganan pembayaran terlambat, status pesanan/pembayaran/pengiriman, dan pembatalan admin dicatat dari diskusi.
- **Belum diputuskan:** Kelompok L–O mengenai autentikasi admin, akses API, audit tindakan admin, dan rate limiting.

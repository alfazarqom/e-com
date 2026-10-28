# Asri Collection

Aplikasi e-commerce modern dengan antarmuka **mobile-first, responsif, dan mudah digunakan** untuk pengalaman belanja yang konsisten di ponsel, tablet, maupun desktop.

> **Status dokumen:** spesifikasi dan rancangan awal. Daftar fitur, tabel, serta Edge Functions di bawah menjelaskan target arsitektur dan tidak berarti semuanya sudah diimplementasikan atau diverifikasi di repository.

## Daftar Isi

- [Gambaran Umum](#gambaran-umum)
- [Tujuan Produk](#tujuan-produk)
- [Teknologi](#teknologi)
- [Ruang Lingkup Fitur](#ruang-lingkup-fitur)
- [Daftar Halaman](#daftar-halaman)
- [Alur Belanja](#alur-belanja)
- [Arsitektur](#arsitektur)
- [Rancangan Database Supabase](#rancangan-database-supabase)
- [Supabase Edge Functions](#supabase-edge-functions)
- [Integrasi Duitku POP](#integrasi-duitku-pop)
- [Keamanan dan Integritas Data](#keamanan-dan-integritas-data)
- [Konfigurasi Environment](#konfigurasi-environment)
- [Menjalankan Proyek](#menjalankan-proyek)
- [Pengujian dan Kriteria Penerimaan](#pengujian-dan-kriteria-penerimaan)
- [Tahapan Pengembangan](#tahapan-pengembangan)
- [Status Implementasi](#status-implementasi)
- [Lisensi](#lisensi)

## Gambaran Umum

**Asri Collection** dirancang sebagai toko online untuk menelusuri katalog produk, melihat detail produk, mengelola keranjang, checkout sebagai tamu (*guest checkout*), membayar melalui Duitku POP, dan memantau pesanan.

Aplikasi menggunakan React, Vite, dan TypeScript di sisi frontend. Supabase menjadi backend melalui PostgreSQL, Storage bila diperlukan, dan Edge Functions untuk operasi server yang tervalidasi. Tidak ada kredensial rahasia pembayaran yang boleh ditanam di frontend.

## Tujuan Produk

- Menyediakan UI modern yang mengutamakan penggunaan di perangkat mobile.
- Memudahkan pelanggan menemukan, membandingkan, dan membeli produk.
- Mendukung checkout tanpa mewajibkan pelanggan membuat akun.
- Menjaga konsistensi harga, stok, pesanan, dan pembayaran di backend.
- Memisahkan status pesanan, pembayaran, dan pengiriman.
- Menyediakan fondasi yang dapat diuji dan dikembangkan tanpa menganggap rancangan sebagai implementasi aktif.

## Teknologi

| Area | Teknologi / rancangan |
|---|---|
| Frontend | React, Vite, TypeScript |
| UI dan styling | Tailwind CSS dan lucide-react (rancangan) |
| Routing | React Router (rancangan) |
| State keranjang | Zustand; penyimpanan lokal untuk pengalaman pengguna |
| Backend | Supabase Edge Functions dengan TypeScript/Deno |
| Database | Supabase PostgreSQL |
| Penyimpanan gambar | Supabase Storage, bila dibutuhkan |
| Payment gateway | Duitku POP |
| Pengujian | Unit, integrasi, dan end-to-end sesuai kemampuan proyek |

Versi paket dan konfigurasi aktual mengikuti berkas proyek seperti `package.json` dan konfigurasi build yang tersedia.

## Ruang Lingkup Fitur

### Katalog dan penemuan produk

- Beranda dengan banner promosi dan koleksi pilihan.
- Kategori dan daftar produk.
- Pencarian produk.
- Filter dan pengurutan.
- Detail produk, gambar, harga, deskripsi, dan ketersediaan.
- Produk terkait atau rekomendasi.
- Tampilan kondisi kosong, loading, dan error.

### Keranjang dan checkout

- Keranjang belanja tanpa akun.
- Mengubah jumlah item dan menghapus item.
- Validasi harga dan ketersediaan ulang di backend sebelum membuat pesanan.
- Form nama, email, nomor telepon, dan alamat pengiriman.
- Ringkasan item, diskon, ongkos kirim, dan total.
- Checkout sebagai tamu.
- Halaman status dan detail pesanan dengan akses yang aman.

### Pembayaran dan pesanan

- Pembuatan invoice Duitku POP dari backend.
- Halaman menunggu pembayaran, gagal, dan berhasil.
- Konfirmasi pembayaran berdasarkan callback backend yang tervalidasi.
- Riwayat atau pencarian pesanan untuk guest melalui mekanisme akses aman.
- Pelacakan pengiriman dan nomor resi.
- Pembatalan, retur, dan refund mengikuti kebijakan bisnis yang ditetapkan.

### Fitur lanjutan yang direncanakan

- Wishlist.
- Voucher dan promo.
- Ulasan produk.
- Pusat bantuan dan FAQ.
- Kontak layanan pelanggan.
- Panel admin untuk mengelola katalog, pesanan, pembayaran, dan pengiriman; akses admin harus dilindungi secara terpisah.

Fitur lanjutan tidak dianggap tersedia sampai implementasi dan pengujiannya terkonfirmasi.

## Daftar Halaman

| No. | Halaman | Fungsi |
|---:|---|---|
| 1 | Splash / App Loading | Menampilkan identitas aplikasi saat inisialisasi |
| 2 | Onboarding (opsional) | Pengenalan singkat aplikasi |
| 3 | Beranda | Banner, kategori, dan produk pilihan |
| 4 | Pencarian | Mencari produk |
| 5 | Kategori | Menjelajahi kategori |
| 6 | Daftar Produk | Menampilkan katalog produk |
| 7 | Filter & Sort | Menyaring dan mengurutkan katalog |
| 8 | Detail Produk | Informasi, gambar, harga, dan ketersediaan |
| 9 | Wishlist (lanjutan) | Menyimpan produk favorit |
| 10 | Keranjang | Memeriksa item yang dipilih |
| 11 | Edit Keranjang | Mengubah kuantitas atau menghapus item |
| 12 | Voucher & Promo (lanjutan) | Memasukkan kode promo |
| 13 | Checkout Guest | Memulai checkout tanpa login |
| 14 | Informasi Pembeli | Mengisi nama dan informasi kontak |
| 15 | Alamat Pengiriman | Mengisi alamat penerima |
| 16 | Pengiriman | Menampilkan opsi atau hasil penentuan ongkir sesuai kebijakan toko |
| 17 | Metode Pembayaran | Memulai pembayaran melalui Duitku POP |
| 18 | Ringkasan Pesanan | Memeriksa item dan perhitungan total |
| 19 | Pengalihan Pembayaran | Mengarahkan pelanggan ke halaman pembayaran Duitku |
| 20 | Menunggu Pembayaran | Menampilkan status pending |
| 21 | Pembayaran Gagal | Menampilkan status gagal dan langkah berikutnya |
| 22 | Konfirmasi Pesanan | Menampilkan hasil yang telah dikonfirmasi backend |
| 23 | Detail Pesanan | Menampilkan ringkasan pesanan dengan akses aman |
| 24 | Lacak Pesanan | Menampilkan status pengiriman dan resi |
| 25 | Pembatalan Pesanan | Mengajukan pembatalan sesuai aturan |
| 26 | Retur & Refund (lanjutan) | Informasi atau pengajuan retur/refund |
| 27 | Pusat Bantuan / FAQ | Menjawab pertanyaan umum |
| 28 | Kontak Layanan Pelanggan | Kanal komunikasi bantuan |
| 29 | Tentang Asri Collection | Informasi toko |
| 30 | Syarat & Ketentuan | Ketentuan penggunaan |
| 31 | Kebijakan Privasi | Penjelasan pemrosesan data |

Navigasi utama mobile dapat menggunakan tab **Beranda**, **Kategori**, **Keranjang**, dan **Akun/Bantuan** sesuai hasil desain. Tampilan desktop menyesuaikan lebar layar tanpa mengubah alur inti.

## Alur Belanja

1. Pelanggan membuka katalog, mencari produk, lalu melihat detail produk.
2. Pelanggan memasukkan produk ke keranjang.
3. Saat checkout, frontend mengirim pilihan item dan data pembeli ke Edge Function.
4. Backend mengambil harga dan status ketersediaan terkini dari database; total tidak dipercaya hanya dari nilai frontend.
5. Backend memvalidasi alamat, ongkos kirim, diskon, stok, dan aturan pesanan.
6. Pesanan serta snapshot item disimpan secara konsisten.
7. Backend membuat transaksi Duitku POP dan menyimpan referensi transaksi serta URL pembayaran.
8. Pelanggan menyelesaikan pembayaran di halaman Duitku.
9. Duitku mengirim callback ke Edge Function.
10. Backend memvalidasi signature, identitas transaksi, nominal, dan status; callback diproses secara idempoten.
11. Frontend menampilkan status dari backend, bukan menganggap redirect browser sebagai bukti pembayaran.
12. Pesanan diteruskan ke proses pemenuhan dan pengiriman sesuai aturan bisnis.

**Catatan ongkos kirim:** jika ongkos kirim ditentukan manual oleh admin, invoice pembayaran baru dibuat setelah ongkir ditetapkan dan disetujui pelanggan. Detail final alur ini harus diselaraskan dengan kebijakan operasional toko.

## Arsitektur

```text
React + Vite + TypeScript
          |
          | HTTPS / request tervalidasi
          v
Supabase Edge Functions
  |          |            |
  |          |            +--> Duitku POP API / callback
  |          +---------------> Supabase Storage (jika diperlukan)
  +--------------------------> Supabase PostgreSQL
```

Prinsip arsitektur:

- Frontend menangani presentasi, navigasi, dan interaksi pengguna.
- Edge Functions menangani operasi tepercaya: validasi, pembuatan pesanan, pembayaran, callback, dan akses data yang perlu otorisasi khusus.
- PostgreSQL menjadi sumber data utama untuk harga, stok, pesanan, dan status pembayaran.
- Gunakan transaksi database atau fungsi SQL atomik untuk operasi yang harus konsisten.
- Data sensitif pelanggan dan pesanan tidak boleh dapat dibaca hanya dengan menebak ID.
- Jangan menganggap nama endpoint, tabel, kolom, atau payload di dokumen ini sebagai kontrak final sebelum dibandingkan dengan implementasi aktual.

## Rancangan Database Supabase

Berikut adalah **model logis awal**, bukan instruksi untuk langsung membuat migration. Nama tabel dan kolom harus disesuaikan dengan schema aktif sebelum implementasi.

| Tabel | Tujuan dan data utama |
|---|---|
| `products` | Katalog: ID, nama/judul, deskripsi, harga, harga promo bila ada, gambar, kategori, ketersediaan, stok, timestamp |
| `categories` | Nama, slug, deskripsi, dan urutan kategori |
| `carts` | Keranjang guest berdasarkan pengenal sesi yang tidak mudah ditebak |
| `cart_items` | Item keranjang, produk, dan kuantitas |
| `orders` | Referensi pesanan, kontak pembeli, alamat pengiriman, total, status pesanan, timestamp |
| `order_items` | Snapshot nama produk, harga satuan, kuantitas, dan subtotal saat transaksi |
| `payments` | Referensi merchant dan Duitku, metode, nominal, URL pembayaran, status, waktu pembayaran |
| `shipments` | Kurir, nomor resi, status dan waktu pengiriman |
| `promotions` (opsional) | Aturan promo, masa berlaku, dan batas pemakaian |
| `wishlists` (opsional) | Produk favorit jika fitur wishlist diaktifkan |
| `reviews` (opsional) | Ulasan dan rating dengan aturan validasi yang jelas |
| `callback_events` (opsional) | Audit callback untuk idempotensi dan penelusuran insiden |

### Prinsip integritas data

- Gunakan primary key, foreign key, constraint, indeks, dan tipe data yang sesuai.
- Simpan nilai uang dalam satuan integer Rupiah bila mata uangnya IDR; hindari floating-point untuk perhitungan uang.
- Buat `merchant_order_id` unik.
- Simpan snapshot nama dan harga di `order_items` agar detail transaksi tidak berubah saat katalog diperbarui.
- Hitung ulang harga, diskon, ongkir, dan total di backend.
- Periksa serta reservasi/perbarui stok secara atomik untuk mencegah overselling.
- Pisahkan status pesanan, pembayaran, dan pengiriman.
- Jangan menjalankan migration atau mengubah database aktif hanya berdasarkan rancangan README ini.

Contoh status awal yang dapat ditinjau:

- Pembayaran: `pending`, `success`, `failed`, `expired`, `refunded`.
- Pesanan: `pending_payment`, `processing`, `ready_to_ship`, `shipped`, `completed`, `cancelled`.
- Pengiriman: `not_shipped`, `shipped`, `delivered`, `failed`.

Status dan transisi final harus mengikuti aturan bisnis serta kontrak Duitku yang benar-benar digunakan.

## Supabase Edge Functions

Nama berikut adalah **usulan tanggung jawab endpoint**, bukan pernyataan bahwa function tersebut sudah ada.

| Function (usulan) | Tanggung jawab |
|---|---|
| `get-products` | Membaca katalog publik dengan kolom yang memang aman dipublikasikan |
| `get-product-detail` | Mengambil detail produk dan ketersediaan |
| `create-order` | Memvalidasi data checkout, harga, stok, dan menyimpan pesanan secara konsisten |
| `calculate-shipping` (opsional) | Menghitung atau mengambil ongkir jika layanan pengiriman terintegrasi |
| `duitku-create-invoice` | Membuat invoice melalui API Duitku dari backend |
| `duitku-callback` | Memvalidasi callback dan memperbarui status transaksi secara idempoten |
| `get-payment-status` | Mengembalikan status pembayaran setelah validasi akses |
| `get-order-status` | Mengembalikan status pesanan/pengiriman dengan mekanisme akses yang aman |
| `reconcile-payment` (opsional) | Menyelaraskan transaksi yang statusnya belum pasti menggunakan sumber tepercaya |
| `expire-pending-payments` (opsional) | Menangani pesanan atau reservasi yang kedaluwarsa sesuai aturan |

Setiap function perlu mendefinisikan kontrak request/response, validasi input, metode HTTP, CORS, autentikasi/otorisasi bila diperlukan, bentuk error, logging yang tidak membocorkan data sensitif, serta pengujian. Jangan menonaktifkan verifikasi JWT atau membuka endpoint publik tanpa analisis kebutuhan dan kontrol akses yang sesuai.

## Integrasi Duitku POP

Integrasi pembayaran harus mengikuti dokumentasi resmi Duitku POP dan mode sandbox sebelum produksi:

- Dokumentasi POP: https://docs.duitku.com/pop/id/
- Dokumentasi API: https://docs.duitku.com/api/id/

Prinsip implementasi:

1. Kredensial Duitku hanya disimpan sebagai secret backend, misalnya `DUITKU_MERCHANT_CODE` dan `DUITKU_API_KEY`.
2. Invoice dibuat dari Edge Function; nominal dan identitas pesanan berasal dari data backend yang tervalidasi.
3. Simpan ID merchant yang unik, referensi dari Duitku, URL pembayaran, nominal, dan status yang diperlukan.
4. Verifikasi signature callback sesuai dokumentasi resmi yang berlaku.
5. Validasi referensi transaksi, nominal, merchant, dan transisi status sebelum memperbarui database.
6. Terapkan idempotensi agar callback berulang tidak menggandakan efek.
7. Redirect atau callback JavaScript di browser hanya untuk pengalaman pengguna, bukan sumber kebenaran pembayaran.
8. Uji callback valid, signature tidak valid, nominal tidak cocok, callback duplikat, timeout, dan status transaksi yang tidak berurutan di sandbox.

Jangan menebak format signature, nama field request, kode status, atau payload callback. Konfirmasikan semuanya dari dokumentasi resmi dan kontrak API aktual.

## Keamanan dan Integritas Data

- Jangan memasukkan `SUPABASE_SERVICE_ROLE_KEY`, API key Duitku, atau secret lain ke kode frontend maupun variabel berawalan `VITE_`.
- Variabel `VITE_*` dapat terekspos ke browser; hanya simpan konfigurasi publik di sana.
- Aktifkan dan tinjau Row Level Security (RLS) pada tabel yang dapat diakses melalui Data API.
- Berikan izin minimum; jangan membuka seluruh kolom atau tabel hanya demi mempermudah UI.
- Batasi akses pesanan guest melalui token acak berentropi tinggi atau mekanisme verifikasi yang sesuai. Jangan mengandalkan nomor pesanan yang mudah ditebak.
- Validasi semua input di backend dan jangan mempercayai harga, subtotal, diskon, ongkir, atau status dari klien.
- Terapkan batasan request, validasi format, CORS yang sesuai, serta pesan error yang tidak membocorkan detail internal.
- Gunakan operasi atomik untuk membuat pesanan dan mengelola stok.
- Jangan mencatat API key, token akses pesanan, data pembayaran sensitif, atau seluruh informasi pribadi ke log.
- Terapkan retensi dan penghapusan data pribadi sesuai kebijakan privasi.
- Gunakan sandbox untuk pengujian pembayaran; jangan melakukan transaksi nyata selama pengembangan.
- Sebelum mengubah schema, periksa schema aktif, policy, grants, function, dan dependensi yang sudah digunakan.

## Konfigurasi Environment

Contoh variabel frontend untuk konfigurasi publik:

```dotenv
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=
```

Contoh secret backend untuk Supabase Edge Functions:

```dotenv
DUITKU_MERCHANT_CODE=
DUITKU_API_KEY=
DUITKU_BASE_URL=
```

Nama variabel harus disesuaikan dengan kode dan dokumentasi Duitku yang digunakan. Jangan isi nilai secret dalam README, commit file `.env`, atau mengirim kredensial melalui chat/log. Untuk deployment Supabase, atur secret melalui mekanisme secret management yang sesuai.

## Menjalankan Proyek

Instruksi berikut berlaku jika repository sudah memiliki konfigurasi React/Vite yang sesuai. Periksa `package.json` dan berkas proyek terlebih dahulu.

```bash
git clone https://github.com/alfazarqom/e-com.git
cd e-com
npm install
npm run dev
```

Untuk build dan pemeriksaan tipe, gunakan script yang memang tersedia di `package.json`, misalnya:

```bash
npm run build
npm run typecheck
npm run lint
npm test
```

Tidak semua script di atas tentu sudah didefinisikan. Jalankan hanya script yang tersedia atau tambahkan melalui perubahan proyek yang ditinjau secara terpisah.

### Konfigurasi Supabase

1. Siapkan proyek Supabase untuk lingkungan development/sandbox.
2. Tinjau schema, RLS, grants, dan Storage bucket sebelum menghubungkan frontend.
3. Konfigurasikan URL dan public key Supabase pada environment lokal.
4. Konfigurasikan secret Duitku hanya di lingkungan backend.
5. Deploy Edge Functions dan migration hanya setelah kontrak, review keamanan, dan persetujuan perubahan tersedia.
6. Uji alur pembayaran di sandbox sebelum mempertimbangkan produksi.

## Pengujian dan Kriteria Penerimaan

- Layout dapat digunakan pada layar ponsel, tablet, dan desktop.
- Navigasi, pencarian, filter, detail produk, serta keranjang memiliki state loading, kosong, dan error yang jelas.
- Harga dan stok divalidasi ulang di backend saat checkout.
- Request checkout duplikat tidak membuat pesanan atau pembayaran ganda.
- Total pesanan dihitung dengan benar dan disimpan sebagai snapshot transaksi.
- Akses detail pesanan guest tidak dapat dilakukan hanya dengan menebak ID.
- Callback pembayaran memverifikasi signature, nominal, referensi, dan status.
- Callback duplikat atau terlambat tidak menyebabkan status mundur atau efek ganda.
- Stok tetap konsisten pada checkout bersamaan atau kegagalan pembayaran.
- Secret tidak ada dalam bundle frontend, repository, atau log.
- Kebijakan RLS dan akses publik diuji dengan role yang relevan.
- Build, pemeriksaan tipe, lint, dan tes yang tersedia lulus sebelum rilis.

## Tahapan Pengembangan

1. **Audit repository:** verifikasi struktur, dependensi, script, schema Supabase aktif, dan function yang sudah tersedia.
2. **Konfirmasi requirement:** tetapkan aturan harga, stok, ongkir, masa berlaku pesanan, guest order access, dan transisi status.
3. **Desain kontrak:** tetapkan schema, API request/response, error, RLS, dan kontrak Duitku berdasarkan sumber resmi.
4. **Implementasi UI:** kerjakan halaman dan alur mobile-first menggunakan data yang sesuai.
5. **Implementasi backend:** buat atau sesuaikan Edge Functions dan operasi database atomik setelah disetujui.
6. **Pengujian:** lakukan tes unit, integrasi, keamanan, dan pembayaran sandbox.
7. **Review dan rilis:** tinjau perubahan, siapkan rollback, dan lakukan deployment hanya setelah persetujuan eksplisit.

## Status Implementasi

README ini adalah dokumentasi target produk. Status aktual tiap halaman, tabel, Edge Function, integrasi pembayaran, dan tes perlu dikonfirmasi melalui audit repository dan Supabase. Jangan menganggap komponen yang disebutkan di sini sudah tersedia hanya karena tercantum dalam dokumen.

## Lisensi

Ikuti lisensi yang benar-benar disediakan di repository. Jika belum ada berkas lisensi, status lisensi perlu ditentukan sebelum distribusi publik.

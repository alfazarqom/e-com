# Asri Collection — E-Commerce

Aplikasi e-commerce berbasis web dengan antarmuka modern dan responsif, dirancang dengan pendekatan **mobile-first**. Teknologi utama: React, Vite, TypeScript, Supabase Database, Supabase Edge Functions, dan Duitku POP.

> **Status dokumentasi:** dokumen spesifikasi/target arsitektur. Fitur, tabel, dan Edge Functions yang disebutkan sebagai rancangan tidak otomatis berarti sudah tersedia. Status implementasi harus diverifikasi terhadap kode, konfigurasi, dan schema Supabase yang aktif.

## Daftar Isi

- [Gambaran Umum](#gambaran-umum)
- [Tujuan dan Ruang Lingkup](#tujuan-dan-ruang-lingkup)
- [Teknologi](#teknologi)
- [Fitur yang Direncanakan](#fitur-yang-direncanakan)
- [Daftar Halaman](#daftar-halaman)
- [Alur Checkout dan Pembayaran](#alur-checkout-dan-pembayaran)
- [Arsitektur](#arsitektur)
- [Rancangan Database](#rancangan-database)
- [Kontrak Supabase Edge Functions](#kontrak-supabase-edge-functions)
- [Integrasi Duitku POP](#integrasi-duitku-pop)
- [Keamanan](#keamanan)
- [Konfigurasi Environment](#konfigurasi-environment)
- [Menjalankan Proyek](#menjalankan-proyek)
- [Pengujian dan Kriteria Penerimaan](#pengujian-dan-kriteria-penerimaan)
- [Tahapan Pengembangan](#tahapan-pengembangan)
- [Status Implementasi](#status-implementasi)
- [Referensi Resmi](#referensi-resmi)
- [Lisensi](#lisensi)

## Gambaran Umum

Asri Collection dirancang sebagai toko online untuk melihat katalog produk, mencari produk, mengelola keranjang, checkout sebagai tamu, mendapatkan perhitungan ongkos kirim otomatis, dan membayar melalui Duitku POP.

Pelanggan **tidak diwajibkan menggunakan Supabase Auth**. Karena itu, akses ke detail pesanan dan informasi pribadi pelanggan harus menggunakan mekanisme verifikasi/otorisasi yang aman; ID pesanan yang mudah ditebak tidak cukup sebagai bukti kepemilikan.

Supabase digunakan sebagai backend melalui PostgreSQL dan Edge Functions. Semua operasi tepercaya—termasuk validasi harga, perhitungan total, pembuatan pesanan, invoice, serta pemrosesan callback pembayaran—harus dilakukan di server.

## Tujuan dan Ruang Lingkup

- Menyediakan pengalaman belanja responsif dengan prioritas penggunaan mobile.
- Menampilkan katalog, kategori, pencarian, detail produk, dan keranjang.
- Mendukung guest checkout tanpa mewajibkan akun pelanggan.
- Menghitung ongkos kirim otomatis sebelum pembayaran, melalui penyedia/aturan pengiriman yang ditetapkan proyek.
- Menjaga konsistensi harga, stok, pesanan, dan pembayaran di backend.
- Mengintegrasikan pembayaran melalui Duitku POP.
- Memisahkan status pesanan, pembayaran, dan pengiriman.

## Teknologi

| Area | Teknologi / pilihan |
|---|---|
| Frontend | React, Vite, TypeScript |
| UI/styling | Tailwind CSS dan lucide-react jika digunakan |
| Routing | React Router jika digunakan |
| State keranjang | Zustand dan penyimpanan lokal jika dipilih |
| Database | Supabase PostgreSQL |
| Backend | Supabase Edge Functions (TypeScript/Deno) |
| Pembayaran | Duitku POP |
| Ongkir | Integrasi penyedia pengiriman atau layanan tarif yang harus ditentukan |
| Testing | Unit, integrasi, dan end-to-end sesuai konfigurasi proyek |

Versi dependensi dan scripts aktual harus mengikuti `package.json`, lockfile, dan konfigurasi repository.

## Fitur yang Direncanakan

### Katalog produk
- Beranda, banner, kategori, dan daftar produk.
- Pencarian, filter, dan pengurutan jika diimplementasikan.
- Detail produk, gambar, harga, deskripsi, dan ketersediaan.
- Tampilan loading, kosong, dan error.

### Keranjang dan checkout
- Keranjang tanpa login pelanggan.
- Mengubah kuantitas dan menghapus item.
- Form kontak dan alamat penerima.
- Validasi ulang produk, harga, dan stok di backend.
- Menghitung subtotal, diskon yang valid, ongkir, dan total di backend.
- Membuat pesanan dengan ID unik dan snapshot harga/item.

### Ongkos kirim otomatis
- Mengumpulkan data alamat dan parameter yang diperlukan penyedia tarif.
- Mengambil atau menghitung tarif pengiriman melalui backend.
- Menampilkan opsi dan biaya yang valid sebelum pelanggan mengonfirmasi checkout/pembayaran.
- Memvalidasi ulang tarif dan alamat saat pesanan dibuat.
- Menangani kegagalan layanan ongkir, respons kosong, tarif kedaluwarsa, dan alamat tidak didukung.

**Penyedia ongkir, format API, autentikasi, cakupan wilayah, dan aturan tarif belum ditetapkan dalam README ini.** Jangan mengklaim integrasi ongkir sudah aktif sampai penyedia dan implementasinya diverifikasi. Secret penyedia ongkir tidak boleh disimpan di frontend.

### Pembayaran dan pesanan
- Pembuatan invoice Duitku dari backend.
- Pengalihan pelanggan ke halaman pembayaran sesuai metode POP yang dipilih.
- Status menunggu, berhasil, gagal, atau kedaluwarsa sesuai kontrak yang diterapkan.
- Konfirmasi status melalui callback server-side yang diverifikasi.
- Tampilan detail/status pesanan melalui mekanisme akses guest yang aman.
- Informasi pengiriman dan nomor resi jika fungsi pengiriman telah tersedia.

Wishlist, voucher, ulasan, refund, dan panel admin merupakan fitur opsional sampai kebutuhan dan implementasinya dikonfirmasi.

## Daftar Halaman

| Halaman | Fungsi |
|---|---|
| Beranda | Banner, kategori, dan produk pilihan |
| Kategori | Menjelajahi katalog berdasarkan kategori |
| Pencarian | Mencari produk |
| Daftar Produk | Menampilkan produk dan ketersediaannya |
| Detail Produk | Informasi produk dan harga |
| Keranjang | Meninjau item dan kuantitas |
| Checkout Guest | Mengisi informasi pembeli dan alamat |
| Ongkos Kirim | Menampilkan opsi/tarif pengiriman otomatis |
| Ringkasan Pesanan | Meninjau item, ongkir, dan total |
| Pembayaran | Memulai pembayaran Duitku POP |
| Menunggu Pembayaran | Menampilkan status transaksi yang belum final |
| Status Pesanan | Menampilkan status dari backend setelah akses diverifikasi |
| Pelacakan Pengiriman | Menampilkan status/resi jika tersedia |
| Kebijakan dan Bantuan | Syarat, privasi, FAQ, dan kontak jika disiapkan |

Daftar ini merupakan rancangan halaman, bukan inventaris halaman yang sudah terimplementasi.

## Alur Checkout dan Pembayaran

1. Pelanggan menelusuri katalog dan menambahkan produk ke keranjang.
2. Pelanggan checkout sebagai tamu dan memasukkan kontak serta alamat pengiriman.
3. Frontend mengirim pilihan produk dan alamat ke backend; nilai harga/total dari browser tidak dipercaya.
4. Backend mengambil harga, ketersediaan, dan stok terkini dari database.
5. Backend meminta atau menghitung ongkir otomatis melalui integrasi yang telah dikonfigurasi.
6. Backend memvalidasi hasil ongkir, menghitung total akhir, dan menyimpan pesanan serta snapshot item secara konsisten.
7. Backend membuat invoice Duitku POP menggunakan nominal dan ID transaksi yang tervalidasi.
8. Frontend menampilkan atau membuka alur pembayaran berdasarkan respons Duitku.
9. Duitku mengirim HTTP callback ke Edge Function.
10. Backend memverifikasi signature, merchant, ID pesanan, nominal, referensi, dan status transaksi.
11. Callback diproses secara idempoten; event duplikat tidak menggandakan efek transaksi.
12. Frontend mengambil status dari backend. Redirect browser atau callback JavaScript bukan bukti final pembayaran.
13. Pesanan yang telah dibayar diteruskan ke pemenuhan/pengiriman sesuai aturan bisnis.

Jika ongkir tidak dapat dihitung atau diverifikasi, sistem harus menghentikan pembuatan invoice atau meminta pelanggan mencoba kembali—bukan menebak biaya pengiriman.

## Arsitektur

```text
Browser
  |
  v
React + Vite + TypeScript
  |
  | HTTPS
  v
Supabase Edge Functions
  |             |                |
  v             v                v
PostgreSQL   Layanan Ongkir    Duitku POP
                                  |
                                  v
                           HTTP Callback
                                  |
                                  v
                         Edge Function
                                  |
                                  v
                          PostgreSQL
```

Tanggung jawab komponen:

- **Frontend:** presentasi, navigasi, keranjang, formulir checkout, dan menampilkan status dari backend.
- **Edge Functions:** validasi input, harga/total, akses guest, perhitungan ongkir melalui provider, pembuatan pesanan/invoice, callback, dan pemeriksaan status.
- **PostgreSQL:** sumber data tepercaya untuk katalog, snapshot pesanan, pembayaran, dan status.
- **Penyedia ongkir:** menyediakan tarif sesuai kontrak API yang nantinya dipilih.
- **Duitku POP:** menyediakan alur pembayaran sesuai dokumentasi dan konfigurasi merchant.

Nama endpoint, payload, tabel, serta hubungan komponen di atas adalah rancangan dan perlu dicocokkan dengan implementasi aktual.

## Rancangan Database

Tabel berikut adalah model logis awal, **bukan instruksi untuk langsung membuat migration**.

| Entitas | Tujuan |
|---|---|
| `products` | Katalog, harga, gambar, ketersediaan, dan stok |
| `categories` | Pengelompokan produk jika digunakan |
| `orders` | ID pesanan, kontak/alamat, ringkasan total, dan status |
| `order_items` | Snapshot produk, harga satuan, kuantitas, dan subtotal |
| `payments` | ID transaksi merchant, referensi Duitku, nominal, dan status |
| `shipments` | Opsi/biaya ongkir, layanan, status, dan resi bila relevan |
| `callback_events` | Catatan callback untuk audit dan idempotensi jika diperlukan |

Nama dan kolom final harus mengikuti schema aktif dan hasil audit. Jangan membuat tabel duplikat atau migration sebelum schema, constraint, RLS, grants, dan dependensi yang ada diperiksa.

Prinsip integritas data:
- Simpan nominal IDR sebagai integer dalam satuan Rupiah, bukan floating-point.
- Buat ID pesanan/transaksi merchant yang unik.
- Simpan snapshot item dan harga saat checkout.
- Hitung ulang harga, diskon, ongkir, dan total di backend.
- Gunakan constraint dan operasi database atomik untuk menjaga stok serta status pesanan.
- Pisahkan status pesanan, pembayaran, dan pengiriman.
- Terapkan akses terbatas untuk data pribadi dan pesanan guest.

## Kontrak Supabase Edge Functions

Nama di bawah adalah usulan tanggung jawab, bukan klaim bahwa function telah tersedia.

| Function usulan | Tanggung jawab |
|---|---|
| `get-products` | Mengambil katalog publik dengan kolom yang aman dipublikasikan |
| `calculate-shipping` | Memvalidasi alamat dan mengambil/menghitung tarif melalui provider yang dipilih |
| `create-order` | Memvalidasi produk, harga, stok, ongkir, dan menyimpan pesanan |
| `duitku-create-invoice` | Memanggil Create Invoice Duitku dari backend |
| `duitku-callback` | Memvalidasi callback dan memproses status secara idempoten |
| `get-payment-status` | Mengembalikan status pembayaran setelah validasi akses |
| `get-order-status` | Menampilkan status pesanan melalui mekanisme guest access yang aman |

Setiap endpoint perlu mendefinisikan metode HTTP, request/response, validasi, autentikasi/otorisasi yang sesuai, CORS, error, logging aman, dan tes. Jangan menonaktifkan verifikasi JWT atau membuka akses database hanya demi mempermudah frontend tanpa analisis keamanan.

### Guest checkout tanpa Supabase Auth

Pelanggan tidak diwajibkan masuk melalui Supabase Auth. Namun, endpoint publik tetap memerlukan perlindungan yang tepat:
- Validasi input, batas request, dan pembatasan penyalahgunaan.
- Token akses pesanan acak berentropi tinggi atau mekanisme verifikasi lain yang sesuai.
- Jangan memakai ID pesanan yang berurutan sebagai satu-satunya otorisasi.
- Jangan memberikan akses luas ke tabel pesanan melalui role publik.
- Simpan dan tampilkan informasi pribadi seminimal mungkin.

Keputusan bahwa pelanggan tidak menggunakan Supabase Auth **tidak berarti seluruh backend atau panel admin tanpa autentikasi**. Akses administratif harus memiliki perlindungan terpisah.

## Integrasi Duitku POP

Dokumentasi resmi:
- POP: https://docs.duitku.com/pop/id/
- API: https://docs.duitku.com/api/id/
- FAQ: https://www.duitku.com/frequently-asked-questions/

Prinsip integrasi:
1. Pembuatan invoice dilakukan melalui backend tepercaya.
2. API key dan kredensial merchant disimpan sebagai secret backend.
3. Nominal dan merchant order ID berasal dari data pesanan yang telah divalidasi.
4. Request signature dan callback signature harus mengikuti kontrak POP resmi yang digunakan; jangan mencampur formula dari produk/API lain.
5. Callback HTTP diverifikasi sebelum status transaksi diperbarui.
6. Cocokkan merchant, ID pesanan, nominal, referensi, dan hasil transaksi.
7. Terapkan idempotensi untuk callback yang dikirim ulang.
8. Jangan menganggap redirect browser atau JavaScript callback sebagai konfirmasi pembayaran.
9. Tangani timeout dan hasil yang belum pasti tanpa membuat transaksi ganda secara sembarangan.
10. Uji di sandbox sebelum mengaktifkan production.

Duitku POP mendokumentasikan pembuatan invoice dan pilihan alur frontend seperti JavaScript checkout atau window redirection. Pilihan yang dipakai aplikasi harus ditentukan dan diverifikasi saat implementasi. Field, signature, status, serta aturan retry harus mengacu pada dokumentasi resmi terbaru dan kontrak merchant yang berlaku.

## Keamanan

- Jangan menaruh `DUITKU_API_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, atau secret provider ongkir di frontend.
- Anggap seluruh variabel `VITE_*` dapat terekspos ke browser.
- Aktifkan dan tinjau RLS/grants untuk tabel yang dapat diakses melalui Data API.
- Jangan membuka semua kolom/tabel kepada publik tanpa peninjauan risiko.
- Validasi harga, stok, ongkir, diskon, dan total di backend.
- Verifikasi signature callback dan kecocokan data transaksi.
- Terapkan idempotensi dan transisi status yang valid.
- Lindungi akses guest terhadap data pesanan.
- Hindari mencatat API key, token akses, signature sensitif, atau data pribadi berlebihan di log.
- Pisahkan environment development/sandbox dan production.
- Tinjau schema, policy, grants, dan fungsi aktif sebelum perubahan database.
- Perubahan kode, database, secret, dan deployment harus melalui review serta persetujuan yang sesuai.

## Konfigurasi Environment

Contoh variabel publik frontend (sesuaikan dengan kode aktual):

```dotenv
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=
```

Contoh nama secret backend (bukan nilai rahasia):

```dotenv
DUITKU_MERCHANT_CODE=
DUITKU_API_KEY=
DUITKU_BASE_URL=
SHIPPING_PROVIDER_API_KEY=
```

Nama variabel penyedia ongkir bersifat ilustratif dan bergantung pada provider yang dipilih. Jangan menaruh nilai secret di README, commit file environment lokal, atau mengekspos secret melalui variabel frontend. Atur secret backend melalui mekanisme secret management yang sesuai.

## Menjalankan Proyek

Instruksi berikut berlaku jika konfigurasi proyek memang sesuai:

```bash
git clone https://github.com/alfazarqom/e-com.git
cd e-com
npm install
npm run dev
```

Untuk build dan pemeriksaan, gunakan hanya scripts yang tersedia di `package.json`. Contoh yang mungkin tersedia:

```bash
npm run build
npm run lint
npm run typecheck
npm test
```

Tidak semua script di atas pasti sudah dikonfigurasi. Audit file proyek sebelum menyatakan perintah berhasil.

## Pengujian dan Kriteria Penerimaan

### Frontend
- Layout dan navigasi diuji pada ukuran layar target.
- Katalog, detail, keranjang, dan checkout menampilkan kondisi loading/error/kosong.
- Total yang ditampilkan konsisten dengan hasil backend.
- Pelanggan dapat menjalani checkout tanpa Supabase Auth.

### Ongkos kirim
- Tarif dihitung/diambil melalui provider yang dikonfigurasi.
- Tarif dan alamat divalidasi ulang saat pesanan dibuat.
- Kegagalan provider tidak menghasilkan ongkir tebakan atau invoice dengan total keliru.
- Total pesanan sama dengan rincian item, diskon yang valid, dan ongkir yang disetujui.

### Backend dan pembayaran
- Harga dan stok tidak dapat dimanipulasi dengan mengubah request frontend.
- Request checkout duplikat tidak membuat efek transaksi ganda.
- Callback dengan signature salah atau nominal tidak cocok ditolak.
- Callback duplikat aman diproses.
- Redirect browser tidak mengubah status pembayaran secara langsung.
- Timeout dan status transaksi yang belum pasti memiliki prosedur pemulihan.
- Akses guest tidak membocorkan pesanan pengguna lain.
- RLS/grants dan endpoint publik diuji.
- Secret tidak muncul di bundle frontend, repository, atau log.
- Build dan tes yang dikonfigurasi berhasil sebelum rilis.

## Tahapan Pengembangan

1. Audit repository, dependensi, scripts, schema Supabase, dan Edge Functions yang sudah ada.
2. Tetapkan provider ongkir dan verifikasi kontrak API, tarif, wilayah, serta error.
3. Finalisasi kontrak database dan endpoint termasuk akses guest.
4. Implementasikan UI dan alur checkout sesuai requirement yang disetujui.
5. Implementasikan ongkir, pesanan, dan Duitku POP secara server-side.
6. Uji integrasi dan pembayaran di sandbox, termasuk callback berulang dan kegagalan.
7. Tinjau keamanan dan lakukan deployment hanya setelah persetujuan eksplisit.

## Status Implementasi

README ini mendokumentasikan target produk dan rancangan arsitektur. Keberadaan halaman, tabel, Edge Functions, integrasi ongkir, callback Duitku, dan pengujian harus diverifikasi dari kode dan konfigurasi aktif. Tidak ada fitur yang dinyatakan production-ready hanya karena dicantumkan di dokumen ini.

## Referensi Resmi

- Supabase Docs: https://supabase.com/docs
- Supabase Edge Functions: https://supabase.com/docs/guides/functions
- Duitku POP: https://docs.duitku.com/pop/id/
- Duitku API: https://docs.duitku.com/api/id/
- Duitku FAQ: https://www.duitku.com/frequently-asked-questions/

## Lisensi

Gunakan lisensi yang benar-benar berlaku di repository. Jika belum ada file lisensi, status lisensi perlu ditentukan oleh pemilik proyek.

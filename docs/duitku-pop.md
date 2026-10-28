# Integrasi Pembayaran Duitku POP

> **Status dokumen:** panduan kontrak integrasi dan checklist implementasi. Dokumen ini bukan bukti bahwa endpoint, Edge Function, database, atau transaksi telah tersedia dan lulus pengujian. Cocokkan kontrak dengan dokumentasi merchant Duitku terbaru sebelum implementasi.

## 1. Sumber resmi

- [Dokumentasi Duitku POP (Bahasa Indonesia)](https://docs.duitku.com/pop/id/)
- [Dokumentasi Duitku API](https://docs.duitku.com/api/id/)
- [FAQ resmi Duitku](https://www.duitku.com/frequently-asked-questions/)

Dokumentasi POP adalah acuan utama untuk endpoint, parameter, signature, callback, dan perilaku pembayaran. Jangan mengambil formula dari produk Duitku lain atau contoh lama tanpa memeriksa kesesuaiannya.

## 2. Arsitektur yang dituju

1. Frontend meminta backend membuat invoice untuk pesanan yang sudah divalidasi.
2. Backend mengambil data pesanan tersimpan dan menghitung nominal tepercaya; nilai total dari browser tidak menjadi sumber kebenaran.
3. Backend mengirim permintaan Create Invoice ke Duitku menggunakan kredensial server-side.
4. Backend menyimpan referensi dan informasi invoice yang diperlukan, lalu mengembalikan informasi minimum untuk memulai pembayaran.
5. Duitku mengirim HTTP callback ke endpoint backend.
6. Backend memverifikasi signature dan mencocokkan merchant, pesanan, nominal, referensi, serta transisi status yang diizinkan.
7. Frontend mengambil status terkini dari backend.

Nama fungsi seperti `duitku-create-invoice`, `duitku-callback`, dan `get-payment-status` adalah nama usulan kontrak; keberadaan atau kesiapan implementasinya belum diverifikasi.

## 3. Endpoint Create Invoice

Dokumentasi POP mencantumkan endpoint berikut:

| Environment | Endpoint |
|---|---|
| Sandbox | `https://api-sandbox.duitku.com/api/merchant/createInvoice` |
| Production | `https://api-prod.duitku.com/api/merchant/createInvoice` |

Gunakan HTTP `POST` dengan payload `application/json`. Pilih environment secara eksplisit di backend dan jangan mencampur kredensial Sandbox dengan endpoint Production.

Header yang didokumentasikan:

- `x-duitku-merchantcode`
- `x-duitku-timestamp`
- `x-duitku-signature`

Periksa dokumentasi resmi untuk format timestamp, daftar parameter wajib/opsional, batas nilai, dan detail encoding yang berlaku saat integrasi dilakukan.

### Signature request

Dokumentasi POP menjelaskan signature Create Invoice menggunakan HMAC-SHA256 dengan string yang ditandatangani:

```text
stringToSign = merchantCode + timestamp
signature = HMAC_SHA256(stringToSign, apiKey)
```

Formula tersebut harus diimplementasikan di backend. Jangan pernah mengirim API key ke frontend atau menyimpan nilainya dalam repository. Pastikan format timestamp dan representasi signature sesuai dokumentasi yang berlaku.

### Data request dan response

Request Create Invoice umumnya memuat data merchant/order, jumlah pembayaran, rincian produk, email, callback URL, dan return URL. Nama field, tipe, validasi, serta field opsional harus mengikuti dokumentasi resmi terbaru.

Response sukses dapat memuat antara lain `reference`, `paymentUrl`, `statusCode`, dan `statusMessage`. Validasi HTTP status **dan** isi respons provider; HTTP sukses saja tidak membuktikan invoice berhasil dibuat.

- Simpan relasi antara order internal, `merchantOrderId`, referensi Duitku, nominal, dan environment.
- Jangan membuat `paymentUrl` palsu atau menganggap invoice sukses jika respons provider tidak valid.
- Tangani timeout atau kegagalan penyimpanan setelah provider menerima request sebagai kasus yang perlu direkonsiliasi.
- Dokumentasi menyebut perilaku khusus ketika `merchantOrderId` yang sama dikirim berulang dalam rentang waktu tertentu, termasuk kemungkinan respons transaksi masih diproses atau ketidakcocokan nominal. Periksa aturan terbaru sebelum merancang retry; jangan membuat ID acak baru sebagai workaround yang tidak terpetakan.

## 4. HTTP callback

Callback server-to-server didokumentasikan sebagai HTTP `POST` dengan content type `application/x-www-form-urlencoded`. Parser backend harus membaca format ini dengan benar, bukan mengasumsikan JSON.

Field callback yang didokumentasikan mencakup informasi seperti `merchantCode`, `amount`, `merchantOrderId`, `resultCode`, `reference`, dan `signature`. Verifikasi daftar field dan format persisnya terhadap dokumentasi terbaru.

### Signature callback

Dokumentasi POP menjelaskan pemeriksaan signature callback menggunakan HMAC-SHA256 dengan string yang ditandatangani:

```text
stringToSign = merchantCode + amount + merchantOrderId
signature = HMAC_SHA256(stringToSign, apiKey)
```

Gunakan API key yang sesuai dengan environment merchant dan ikuti detail format/encoding dari dokumentasi. Metode signature MD5 yang lama tidak boleh dipakai sebagai fallback untuk callback POP apabila kontrak resmi mensyaratkan HMAC-SHA256.

Sebelum mengubah status pembayaran, backend harus:

1. Menolak callback dengan field wajib yang hilang atau format yang tidak valid.
2. Memverifikasi signature secara server-side; gagal tertutup jika secret tidak tersedia atau signature salah.
3. Memastikan merchant cocok dengan konfigurasi aktif.
4. Menemukan transaksi berdasarkan identitas merchant order yang tersimpan.
5. Mencocokkan nominal dan referensi provider dengan transaksi yang diharapkan.
6. Memeriksa status pembayaran saat ini dan apakah transisi yang diminta diizinkan.
7. Menerapkan perubahan secara atomik dan idempoten.

Signature yang valid saja tidak cukup untuk mengizinkan perubahan pada pesanan yang tidak cocok. Jangan menggunakan IP allowlist sebagai pengganti verifikasi signature.

### Hasil callback dan retry

Dokumentasi POP menggunakan `resultCode` untuk hasil transaksi; kode `00` berarti berhasil dan `01` berarti gagal menurut kontrak yang didokumentasikan. Pastikan kode dan interpretasi lain diperiksa kembali saat implementasi.

Duitku dapat mengirim ulang callback ketika tidak menerima respons HTTP yang diharapkan. Karena itu:

- Callback duplikat harus aman dan tidak menggandakan perubahan status, stok, atau pemenuhan pesanan.
- Kembalikan respons yang diharapkan provider setelah pemrosesan ditangani dengan benar.
- Jangan mengirim respons sukses setelah callback diam-diam gagal diproses atau belum tersimpan.
- Simpan catatan pemrosesan minimum yang aman agar kegagalan dapat diselidiki dan direkonsiliasi.

Periksa jumlah percobaan dan aturan retry terbaru pada dokumentasi resmi; jangan menjadikan angka retry yang tertulis di dokumen ini sebagai kontrak permanen.

## 5. Return URL dan JavaScript callback

Return URL dan callback JavaScript di browser hanya untuk navigasi dan pengalaman pengguna. Keduanya bukan bukti pembayaran yang tepercaya dan tidak boleh langsung mengubah status pembayaran di database.

Setelah pelanggan kembali dari Duitku, frontend harus meminta status terbaru dari backend. Backend mengembalikan status yang telah tersimpan dan/atau diverifikasi sesuai desain endpoint yang aman.

## 6. Status, idempotensi, dan rekonsiliasi

Pisahkan setidaknya konsep berikut:

- **Status pesanan:** siklus pemenuhan pesanan.
- **Status pembayaran:** hasil transaksi Duitku yang sudah diverifikasi.
- **Status pengiriman:** proses pengiriman dan resi.

Jangan menyimpulkan status pembayaran hanya dari return URL, state lokal frontend, atau HTTP 200 dari endpoint provider.

Callback dan pekerjaan rekonsiliasi harus dirancang agar idempoten. Constraint database dan transisi status bersyarat lebih kuat daripada mengandalkan pemeriksaan di memori saja. Jika status lokal dan provider berbeda, gunakan mekanisme pemeriksaan status yang benar-benar didukung dokumentasi dan konfigurasi merchant; jangan mengarang endpoint inquiry atau menganggap endpoint Create Invoice sebagai endpoint pemeriksaan status.

Kebijakan pesanan kedaluwarsa, pemulihan stok, pembayaran terlambat, konflik stok, dan refund mengikuti `docs/REQUIREMENTS.md`. Jangan menandai refund berhasil tanpa bukti hasil yang terverifikasi. Implementasi detailnya masih harus diaudit dan diuji.

## 7. Keamanan dan konfigurasi

- Simpan `DUITKU_MERCHANT_CODE` dan `DUITKU_API_KEY` sebagai secret backend, bukan sebagai variabel `VITE_*`.
- Jangan menampilkan nilai secret, signature sensitif, atau data pelanggan berlebihan pada log.
- Gunakan validasi server-side untuk nominal, ID pesanan, referensi, dan status.
- Batasi dan lindungi endpoint pembuatan invoice; rate limiting untuk pembuatan invoice diwajibkan oleh kebutuhan V1.
- Pastikan Sandbox dan Production terpisah secara eksplisit.
- Jangan mengubah database, RLS/grants, secret, atau deployment tanpa audit dan persetujuan perubahan yang sesuai.

Nama secret di atas adalah nama yang direncanakan untuk dokumentasi; konfigurasi aktual belum diverifikasi.

## 8. Checklist pengujian Sandbox

Sebelum menyatakan integrasi siap, jalankan dan catat hasil untuk:

- [ ] Create Invoice valid dengan kredensial Sandbox.
- [ ] Signature request sesuai formula resmi.
- [ ] Respons HTTP sukses dengan status bisnis provider yang valid.
- [ ] Respons provider gagal, timeout, dan respons yang tidak lengkap.
- [ ] Callback sukses dengan signature valid.
- [ ] Callback gagal dengan signature salah atau secret tidak tersedia.
- [ ] Merchant, merchant order ID, nominal, atau referensi tidak cocok.
- [ ] Callback duplikat dan callback bersamaan untuk transaksi yang sama.
- [ ] Callback terlambat, tidak diterima, atau dikirim ulang.
- [ ] Kegagalan database setelah callback valid diterima.
- [ ] Return URL dan parameter browser yang dimanipulasi tidak mengubah status.
- [ ] Konfigurasi Sandbox/Production yang tidak cocok gagal dengan aman.
- [ ] Rekonsiliasi status dan konflik pembayaran mengikuti kebutuhan V1.

Gunakan transaksi uji atau mock terisolasi untuk kasus negatif yang berpotensi membuat transaksi provider. Jangan melakukan pembayaran nyata sebagai bagian dari checklist ini tanpa persetujuan eksplisit.

## 9. Batas klaim implementasi

Dokumen ini merangkum kontrak POP yang perlu diikuti dan langkah verifikasi yang diperlukan. Dokumen ini **tidak** menyatakan bahwa:

- Edge Functions yang disebutkan telah dibuat atau diamankan;
- database telah memiliki tabel, constraint, RLS, atau RPC yang diperlukan;
- secret sudah dikonfigurasi;
- seluruh skenario Sandbox telah dijalankan;
- integrasi telah siap untuk Production.

Sebelum implementasi, periksa ulang [dokumentasi resmi Duitku POP](https://docs.duitku.com/pop/id/) dan catat tanggal serta bagian yang diverifikasi. Status akhir harus didasarkan pada bukti kode, konfigurasi, dan hasil pengujian aktual.

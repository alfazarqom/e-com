E-Commerce Mobile UI

Versi: 1.2

Aplikasi e-commerce dengan tampilan modern dan responsif yang dirancang untuk memberikan pengalaman belanja online yang nyaman di perangkat mobile maupun desktop.

✨ Fitur Utama
Responsive mobile UI
Halaman beranda produk
Kategori dan pencarian produk
Filter dan pengurutan produk
Detail produk
Keranjang belanja
Guest Checkout tanpa login atau registrasi
Informasi pembeli dan alamat pengiriman
Integrasi pembayaran melalui Duitku POP
Konfirmasi pembayaran dan detail pesanan
Pelacakan status pesanan
📱 Daftar Halaman Aplikasi
No.	Nama Halaman	Fungsi
1	Splash Screen	Tampilan awal aplikasi
2	Onboarding	Pengenalan aplikasi
3	Home	Menampilkan produk dan promo
4	Search	Mencari produk
5	Category	Menampilkan kategori produk
6	Product Listing	Menampilkan daftar produk
7	Filter & Sort	Menyaring dan mengurutkan produk
8	Product Detail	Menampilkan detail produk
9	Wishlist	Menyimpan produk favorit
10	Cart	Menampilkan keranjang belanja
11	Edit Cart	Mengubah jumlah atau menghapus produk
12	Voucher & Promo	Memasukkan kode promo
13	Guest Checkout	Memulai checkout tanpa login
14	Buyer Information	Mengisi nama, email, dan nomor telepon
15	Shipping Address	Mengisi alamat pengiriman
16	Shipping Method	Memilih layanan pengiriman
17	Payment Method	Memilih metode pembayaran melalui Duitku POP
18	Order Summary	Memeriksa detail dan total pesanan
19	Payment Processing	Memulai proses pembayaran
20	Payment Pending	Menampilkan status menunggu pembayaran
21	Payment Failed	Menampilkan status pembayaran gagal
22	Order Success	Konfirmasi pesanan berhasil
23	Order Detail	Menampilkan detail transaksi
24	Track Order	Melacak status pengiriman
25	Cancel Order	Membatalkan pesanan sesuai ketentuan
26	Return & Refund	Mengajukan pengembalian barang atau dana
27	Help Center / FAQ	Menampilkan panduan bantuan
28	Contact Customer Service	Menghubungi layanan pelanggan
29	About Us	Informasi tentang aplikasi
30	Terms & Conditions	Syarat dan ketentuan penggunaan
31	Privacy Policy	Kebijakan privasi
🗄️ Daftar Database Table
No.	Tabel	Kolom Utama	Fungsi
1	products	id, name, description, price, stock, category, product_image	Menyimpan data produk
2	carts	id, session_id, created_at	Menyimpan keranjang guest
3	cart_items	id, cart_id, product_id, quantity	Menyimpan item keranjang
4	orders	id, customer_name, email, phone, address, total_amount, order_status, created_at	Menyimpan data pembeli dan pesanan
5	order_items	id, order_id, product_id, product_name, price, quantity, subtotal	Menyimpan detail produk pesanan
6	payments	id, order_id, merchant_order_id, duitku_reference, payment_method, payment_status, payment_url, paid_at	Menyimpan data transaksi Duitku POP
7	shipments	id, order_id, courier, tracking_number, shipping_status	Menyimpan data pengiriman
Catatan Struktur Database
Produk: Data kategori dan gambar produk disimpan langsung di tabel products.
Pesanan: Data pembeli dan alamat pengiriman disimpan langsung di tabel orders.
Keranjang guest: Keranjang diidentifikasi melalui session_id, tanpa memerlukan akun pengguna.
Snapshot transaksi: Kolom product_name dan price pada order_items menyimpan data produk saat transaksi dibuat agar riwayat pesanan tetap akurat meskipun data produk berubah.
ID transaksi: merchant_order_id harus unik untuk setiap transaksi.
Relasi tabel: cart_items mengacu pada carts dan products; order_items mengacu pada orders dan products; payments serta shipments mengacu pada orders.
Integritas data: Gunakan primary key, foreign key, indeks, dan constraint yang sesuai. Kolom serta relasi dapat disesuaikan dengan kebutuhan implementasi dan payment gateway.
💳 Integrasi Duitku POP

Integrasi pembayaran menggunakan Duitku POP untuk mengarahkan pembeli ke halaman pembayaran yang dikelola Duitku.

Kolom Pembayaran
Kolom	Keterangan
merchant_order_id	ID transaksi unik yang dibuat oleh sistem toko
duitku_reference	Referensi transaksi yang dikembalikan Duitku
payment_url	URL halaman pembayaran Duitku POP
payment_method	Metode pembayaran yang dipilih atau digunakan
payment_status	Status pembayaran, misalnya pending, success, atau failed
paid_at	Waktu pembayaran terkonfirmasi
Proses Integrasi

Saat membuat transaksi, backend mengirim permintaan ke Duitku POP. Setelah transaksi berhasil dibuat, sistem menyimpan referensi transaksi dan URL pembayaran ke tabel payments.

Pembeli kemudian diarahkan ke payment_url untuk menyelesaikan pembayaran melalui halaman Duitku.

Catatan: Pembuatan transaksi, penyimpanan kredensial, verifikasi callback, dan pembaruan status pembayaran harus dilakukan di backend. Detail request, signature, dan callback mengikuti dokumentasi resmi Duitku yang berlaku.

🔄 Alur Pembayaran
Pembeli memilih produk dan memasukkannya ke keranjang.
Pembeli melakukan checkout sebagai guest tanpa login.
Pembeli mengisi informasi kontak dan alamat pengiriman.
Sistem menghitung total produk, ongkos kirim, diskon, dan jumlah pembayaran.
Sistem membuat data pada tabel orders dan order_items.
Backend meminta pembuatan transaksi ke Duitku POP.
Backend menyimpan merchant_order_id, duitku_reference, dan payment_url pada tabel payments.
Pembeli diarahkan ke halaman pembayaran Duitku.
Duitku mengirim HTTP callback ke backend setelah status transaksi berubah.
Backend memverifikasi signature dan validitas callback.
Sistem memperbarui payment_status dan order_status sesuai hasil pembayaran yang telah diverifikasi.
Pembeli dapat melihat hasil pembayaran dan status pesanan melalui halaman konfirmasi atau pelacakan pesanan.
🔐 Catatan Keamanan

Gunakan HTTP callback Duitku yang diverifikasi di backend sebagai sumber kebenaran status pembayaran. Jangan hanya mengandalkan redirect setelah pembayaran atau callback JavaScript di browser.

Pastikan hal-hal berikut:

ID transaksi unik: merchant_order_id harus unik untuk setiap transaksi.
Validasi signature: Verifikasi signature callback sesuai spesifikasi resmi Duitku.
Idempotensi: Callback yang sama dapat diterima berulang kali tanpa memproses transaksi dua kali.
Validasi transaksi: Cocokkan identitas transaksi, nominal pembayaran, dan status yang diterima dengan data pesanan.
Pembaruan status: Status pembayaran dan pesanan hanya diperbarui setelah validasi berhasil.
Keamanan akses: Halaman Order Detail dan Track Order harus menggunakan token akses atau mekanisme verifikasi yang aman, bukan hanya nomor pesanan yang mudah ditebak.
Pengelolaan stok: Pastikan stok diperiksa dan diperbarui secara konsisten agar tidak terjadi penjualan melebihi stok tersedia.
Pengelolaan rahasia: Simpan merchant key dan kredensial Duitku di backend, bukan di aplikasi frontend.
Validasi input: Validasi seluruh input dari pengguna di backend dan gunakan query database yang aman.
Transaksi database: Gunakan transaksi database saat membuat pesanan, menyimpan detail item, dan memperbarui stok untuk menjaga konsistensi data.
Aturan Status Pesanan

Pesanan baru diproses setelah pembayaran terkonfirmasi jika kebijakan bisnis mengharuskan pembayaran di muka.

Pesanan yang belum dibayar tetap berstatus menunggu pembayaran dan dapat dibatalkan sesuai batas waktu serta aturan yang ditetapkan.

Status pembayaran dan status pesanan harus dikelola secara terpisah agar status transaksi tidak tercampur dengan status pengiriman.

Contoh status pembayaran:

pending
success
failed

Contoh status pesanan:

pending_payment
processing
shipped
completed
cancelled

Daftar status tersebut merupakan rancangan awal dan dapat disesuaikan dengan kebutuhan bisnis.

🎯 Ruang Lingkup MVP

Versi awal aplikasi berfokus pada:

Katalog, pencarian, dan detail produk.
Keranjang belanja guest.
Checkout tanpa login.
Informasi pembeli dan alamat pengiriman.
Pembayaran melalui Duitku POP.
Konfirmasi pembayaran dan detail pesanan.
Pelacakan pengiriman.

Fitur berikut dapat dikembangkan pada tahap berikutnya:

Wishlist.
Voucher & Promo.
Ulasan produk.
Pembatalan pesanan.
Pengembalian barang dan dana.
🛠️ Teknologi

Teknologi frontend awal:

HTML
CSS
JavaScript

Backend, database, dan konfigurasi Duitku POP akan ditentukan sesuai kebutuhan implementasi.

🚀 Instalasi
Clone repository:
bash
git clone https://github.com/username/ecommerce-mobile-ui.git

Masuk ke direktori proyek:
bash
cd ecommerce-mobile-ui

Buka file index.html di browser untuk melihat UI frontend.

Catatan: Langkah di atas hanya untuk frontend statis. Integrasi pembayaran Duitku POP membutuhkan backend yang dikonfigurasi secara terpisah.

📐 Responsive Design

Dirancang untuk berbagai ukuran layar:

Mobile
Tablet
Desktop
📂 Struktur Proyek
text
ecommerce-mobile-ui/
├── index.html
├── css/
│   └── style.css
├── js/
│   └── script.js
├── images/
├── README.md
└── backend/        # Rencana implementasi backend


Struktur di atas merupakan gambaran awal dan dapat berkembang sesuai arsitektur proyek.

📸 Preview

Tambahkan screenshot aplikasi di sini.

📄 License

MIT License

Version History
v1.0 — Baseline awal README E-Commerce Mobile UI.
v1.1 — Menambahkan daftar 31 halaman aplikasi, catatan Guest Checkout, integrasi Duitku POP, verifikasi status pembayaran, akses pesanan yang aman, dan penyesuaian fitur MVP.
v1.2 — Menambahkan daftar database table, catatan struktur database, kolom integrasi Duitku POP, alur pembayaran, keamanan transaksi, aturan status pesanan, dan ruang lingkup MVP.

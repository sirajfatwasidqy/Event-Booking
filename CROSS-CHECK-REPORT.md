# Laporan Cross-Check — Revisi 2 (permintaan_perbaikan.docx)

Hanya 5 perbaikan + penyeragaman bahasa Inggris yang diterapkan; desain, komponen, dan halaman lain tidak diubah.

| # | Permintaan | Hasil |
|---|---|---|
| 1 | Booking History: hapus "Catalog" & "Browse Catalog" | Dihapus. Empty state kini "Scan a product barcode to book an item." tanpa tombol. Kolom pencarian melebar penuh. |
| 2 | Scanner: hapus "Enter Barcode Manually", "Find Product", "Browse Catalog" | Dihapus (form manual + validasinya ikut dibuang). Tersisa tombol **Simulate Scan**. |
| 3 | Product detail: quantity dengan tombol +; hapus Booking Confirmation; langsung Booking Success | Quantity (− / +, 1–5) ada di kartu Booking Summary dengan Total Price yang ikut berubah. **Add to Booking** langsung membuat booking → `booking-success.html`. `booking-confirmation.html` dihapus. |
| 4 | Continue Shopping → scanner | `booking-success.html` dan `booking-detail.html` → `scanner.html`. |
| 5 | Hapus Change Password dari menu Profile | Dihapus dari sidebar akun dan dropdown akun; tetap tersedia sekali di Privacy & Security. |
| — | Bahasa Inggris konsisten | "Tiket"→"Ticket", "Profil"→"Profile", subjudul Ticket, "orang"→"person/people", "Registrasi…"→"Registration for…", "Memproses…"→"Processing…", "Mencari event…"→"Searching events…", "Terverifikasi"→"Verified", "Email or Number Phone"→"Email or Phone Number", placeholder "Cari data barang…"→"Search items or barcode...", judul tab/breadcrumb, teks check-in. Data contoh aktivitas juga diterjemahkan. |

## Perubahan tambahan yang perlu Anda konfirmasi (konsekuensi langsung dari permintaan)
- **Halaman `catalog.html` dihapus** beserta tautannya di footer ("Catalog") dan breadcrumb. Alasannya: "guest tidak boleh melihat katalog" tidak terpenuhi jika halaman masih bisa dibuka lewat footer/URL.
- **Halaman `booking.html` (Product Booking) dihapus**: isinya (quantity + ringkasan) kini ada di product detail, sehingga halaman itu tak lagi terjangkau.
- Breadcrumb produk kini *Booking › Scan Product Barcode › Product Detail* (sebelumnya melewati Catalog).
- Pada halaman Change Password, item aktif di sidebar sekarang **Privacy & Security** (item Change Password sudah tidak ada).
- Catatan "Demo barcodes: …" di scanner **dibiarkan** karena tidak ada di daftar hapus.
- "Simulate Scan" masih simulasi kamera (sama seperti sebelumnya); pemindaian kamera sungguhan tidak diminta/dibuat.

## Pengujian revisi 2 (Chromium)
- 33 asersi end-to-end lulus: tidak ada tautan/teks katalog; scanner tanpa input manual; scan → product detail; quantity mulai 1, minus nonaktif di 1, plus nonaktif di 5, total Rp 45.534.000 untuk 3 unit; Add to Booking → Booking Success langsung (4 item = Rp 60.712.000); Continue Shopping → scanner; riwayat memuat booking; tidak ada Change Password di sidebar/dropdown; Change Password tetap bisa dibuka dan berfungsi dari Privacy & Security; halaman yang dihapus mengembalikan 404.
- Pemindaian teks tampilan 19 halaman/state: tidak ada kata Indonesia yang tersisa.
- Crawl 22 halaman + 404/index: 0 error JavaScript, 0 respons ≥400, 0 aset/tautan hilang. Swirl, `file://`, dan tampilan mobile tetap lolos.
- Pemeriksaan visual desktop & mobile untuk booking history, scanner, product detail, profile, dan booking success.

---

# Laporan Cross-Check — Revisi 1 (restrukturisasi + Swirl, arsip)
*(Catatan: jumlah halaman dan daftar alur di bawah merujuk ke versi sebelum revisi 2.)*

Pengujian memakai Chromium (Playwright) terhadap ZIP asli (`event-booking-website-animated.zip`) dan hasil restrukturisasi.

## A. Verifikasi desain — LULUS
- **78 screenshot** (desktop 1440 px dan mobile 390 px; 4 skenario data: logout, login kosong, draft, data contoh; seluruh halaman termasuk state "not found") dibandingkan piksel demi piksel dengan versi asli.
- **Identik** untuk semua halaman selain Home. Selisih sisanya: (1) hero Home — memang diganti Swirl; (2) toast "Sample data loaded" yang kini terbawa ke halaman berikutnya (perilaku SPA asli) dan (3) selisih bayangan pill navigasi aktif ≤0,07 % pada beberapa tangkapan.
- Bagian Home **di bawah hero** (Upcoming Events, Quick Access, footer) identik dengan asli.
- CSS dipecah menjadi 17 file; **527 baris aturan** sebelum dan sesudah, tanpa selisih (hanya `.empty` dan `.bnav{display:none}` dipindah ke file shared). `motion.css` asli dipertahankan.
- Font Plus Jakarta Sans 400–800 termuat dari `assets/fonts` (dicek di server dan `file://`).

## B. Verifikasi fungsi — LULUS (58/59 asersi; 1 ekspektasi tes salah, bukan bug)
Alur yang dijalankan end-to-end: `index.html` & tautan hash lama → guard login → validasi login (kosong/salah) → toast terbawa → pencarian hero → pencarian live & filter event (+ sinkron URL) → registrasi 2 langkah (validasi, jumlah peserta, ringkasan) → e-ticket, unduh SVG/ICS, modal QR + Escape → simulasi check-in → scanner (barcode salah/simulasi) → produk → booking (qty, total Rp 45.534.000 untuk 3 unit) → konfirmasi → riwayat + pencarian → detail + unduh → katalog `?q=` → notifikasi (tab, tandai dibaca, buka tautan) → edit profil → toggle login alert, modal sesi → ganti password (validasi) → logout → registrasi akun baru.
- Satu asersi gagal karena tes mengira registrasi berakhir di Home; aplikasi (seperti aslinya) kembali ke halaman yang diminta sebelum login. Perilaku benar.
- Crawl 26 file HTML: **0 error JavaScript, 0 respons HTTP ≥400, 0 gambar rusak, 0 aset hilang, 0 tautan tujuan hilang** (16 target unik + semua route `data-go` dipetakan ke file yang ada).
- Perbaikan yang diperlukan karena multi-page: `startReg` kini menyimpan `S.ctx` sebelum berpindah halaman (di SPA cukup di memori); halaman yang dipulihkan dari back/forward cache dimuat ulang agar status login selalu segar.

## C. Verifikasi background — LULUS
- 3 bentuk Swirl, filter turbulence, keyframe, dan loop `requestAnimationFrame` aktif (transform inline berubah antar frame); keyframe CSS `swirlLiquidOne` aktif.
- Tombol/input hero tetap di atas swirl dan bisa diklik (`elementFromPoint`); lapisan swirl `pointer-events:none`.
- Loop berhenti saat hero di luar layar dan lanjut saat kembali; re-render tidak menggandakan lapisan (1 layer); *reduced motion* → statis.
- Tidak ada bentrok CSS/JS (semua id/class/selector berawalan `swirl-`, aturan global demo tidak dibawa). Mobile (390 px): hero terbaca dan tanpa overflow horizontal di Home.
- **Catatan:** pada pengukuran kontras, teks putih hero di atas swirl asli ±1,1:1 di titik terang; ditambahkan lapisan gelap tipis (satu blok CSS terpisah, lihat README) sehingga rata-rata kontras ±10:1. Performa dalam Chromium headless (render software) tidak mewakili GPU nyata; filter SVG turbulence tergolong berat pada perangkat lemah — karena itu loop dijeda saat tidak terlihat.

## D. Verifikasi multi-page — LULUS
- Setiap halaman dapat dibuka langsung (termasuk `?id=` dan `?q=`), path relatif benar, CSS/JS/font/gambar termuat.
- Data (sesi login, draft registrasi/booking, tiket, booking, notifikasi) konsisten antarhalaman lewat `localStorage`; draft jumlah peserta terbawa ke ringkasan.
- Dibuka via `file://` tanpa server: navigasi, login, dan sesi bekerja di Chromium.

## Keterbatasan yang perlu diketahui
- Konten halaman tetap dirender JavaScript (bukan HTML statis) karena bergantung pada data `localStorage`; tanpa JavaScript halaman hanya menampilkan pesan.
- Swirl dipasang di hero Home, bukan seluruh layar Home, karena sisa halaman berlatar terang dengan teks gelap yang tidak terbaca di atas background chrome gelap.
- Pengujian dilakukan di Chromium; Firefox/Safari belum dijalankan.

# Event Booking — PT Duta Abadi Primantara (versi multi-page)

Website Event Booking PT DAP yang sebelumnya satu `index.html` (SPA dengan routing `#/…`) kini dipecah menjadi **satu file HTML per halaman**, ditambah animated background **Swirl** pada Home Screen. Desain, konten, fitur, dan animasi asli tidak diubah.

## Cara menjalankan secara lokal

Pilih salah satu (semuanya tanpa build / tanpa `npm install`):

1. **Server lokal (disarankan)** — dari folder ini:
   - `python3 -m http.server 8000` lalu buka <http://localhost:8000/>
   - atau `npx serve .`
   - atau VS Code → ekstensi *Live Server* → *Go Live*.
2. **Langsung dari file** — buka `index.html` (atau `home.html`) dengan klik dua kali. Sudah diuji di Chromium; JavaScript memakai skrip klasik (bukan ES module), jadi tidak butuh server. Bila browser Anda memisahkan `localStorage` per file `file://` (mis. Firefox dengan pengaturan ketat), data login tidak terbawa antarhalaman — gunakan cara 1.

**Akun demo:** `siraj.fatwa@example.com` · password `Demo@2026`.
Tombol **Demo tools** di footer memuat data contoh (registrasi, 2 booking, 4 notifikasi) atau mereset data.
Data prototype disimpan di `localStorage` browser (kunci `ecbs.v1`).

## Alur booking (revisi 2)

`scanner.html` (scan barcode) → `product-detail.html` (pilih quantity dengan tombol +/−, maks. 5) → **Add to Booking** langsung menampilkan `booking-success.html` → *Continue Shopping* kembali ke `scanner.html`.
Tidak ada katalog yang dapat dilihat pengguna; satu-satunya jalan ke produk adalah memindai barcode item. Halaman Catalog, Product Booking, dan Booking Confirmation sudah dihapus.
Seluruh teks antarmuka berbahasa Inggris (mis. "Ticket", "Profile"). Pengecualian: nama produk resmi "KING KOIL Chiro Endorsed – Kasur Spring Bed" dan alamat lokasi event tetap sesuai data aslinya.
**Change Password** hanya ada di dalam *Privacy & Security*.

## Struktur folder

```
index.html                    pintu masuk → membuka home.html (tautan lama index.html#/events?q=… tetap bekerja)
home.html                     Home (hero + Swirl, event, quick access)
events.html                   Explore Events (pencarian + filter, ?q= & ?c=)
event-detail.html             Detail event (?id=sampoerna | gathering | showcase)
login.html  register.html     Login, pendaftaran akun peserta
register-event.html           Registrasi event — langkah 1
register-event-summary.html   Registrasi event — langkah 2 (ringkasan)
registration-success.html     Registrasi berhasil + e-ticket QR (?id=REG-…)
tickets.html                  Ticket (daftar registrasi + aktivitas)
ticket-detail.html            Detail tiket + QR check-in (?id=REG-…)
check-in.html                 Check-in berhasil (?id=REG-…)
scanner.html                  Scan Product Barcode (hanya pemindaian barcode, simulasi kamera)
product-detail.html           Detail produk + quantity (+/−) + Add to Booking (?id=kingkoil | serta)
booking-success.html          Booking terkonfirmasi (?id=BKG-…); Continue Shopping → scanner.html
booking-history.html          Riwayat booking (?q=)
booking-detail.html           Detail booking (?id=BKG-…)
profile.html  notifications.html  privacy-security.html  change-password.html
404.html                      Tampilan "Page not found" asli
assets/
  css/   fonts · base · header · buttons · layout · home · events · auth · registration ·
         tickets · booking · account · history · footer · overlays · responsive · fixes ·
         motion (animasi asli) · swirl (background baru)
  js/
    core/     routes (daftar halaman) · utils · icons · data · state · qr-data · qr · ui · files ·
              forms · components · router · chrome · actions · listeners · boot
    shared/   registration · booking      (dipakai beberapa halaman)
    pages/    satu modul per halaman (home.js, events.js, login.js, …)
    effects/  swirl.js (background) · motion.js (animasi asli) · fx-boot.js
  img/  fonts/   gambar, logo, dan Plus Jakarta Sans 400–800 (lokal)
```

Setiap halaman memuat daftar CSS yang sama (urutan cascade sama dengan file asli) dan skrip `core/*` + modul halamannya, diakhiri `core/boot.js` lalu `effects/motion.js`. Skrip memakai lingkup global bersama (tanpa bundler), jadi **urutan `<script>` penting**. Isi halaman tetap dirender dari JavaScript karena bergantung pada data di `localStorage` (login, tiket, booking).

## Menambah halaman baru

1. Tambahkan entri di `PAGES` pada `assets/js/core/routes.js` (key, file, pola route, nav, wajib login, judul).
2. Buat view `V.<key> = (id, qs) => '…html…'` di `assets/js/pages/<nama>.js`.
3. Salin salah satu file `.html` yang ada, ganti `data-page`, `<title>`, dan skrip halamannya.

## Animated background (Swirl)

- Sumber: `swirl.js`, `style.css`, dan markup SVG dari ZIP Background (path, gradien, filter turbulence, keyframe, dan rumus gerak per-frame asli).
- Dipasang **di dalam hero Home** sebagai lapisan `position:absolute` di belakang konten (`z-index:0`, `pointer-events:none`). Halaman lain tidak berubah.
- Penyesuaian agar tidak bentrok: id/class diberi awalan `swirl-`, aturan global demo (`*`, `body`, `main`, `h1`, `p`, `button`) tidak dibawa, loop berhenti saat hero tidak terlihat / tab disembunyikan, dan statis untuk pengguna *reduced motion*.
- **Lapisan gelap tipis untuk keterbacaan** (`.hero > .swirl-bg::after` di `assets/css/swirl.css`): tanpa ini, teks putih hero hampir tidak terbaca di atas pita putih/pink swirl (kontras terukur ±1,1:1). Warna dan gerakan swirl tidak diubah; hapus blok itu bila ingin melihat efek mentah.

## Penyesuaian teknis akibat multi-page (perilaku tetap sama)

- Navigasi memakai file asli (`go('/tickets')` → `tickets.html`); route lama yang tersimpan di data browser (notifikasi, tujuan setelah login) tetap dipetakan.
- Halaman yang butuh login mengarahkan ke `login.html` lalu kembali ke halaman tujuan; login/register saat sudah masuk diarahkan ke Home — seperti sebelumnya.
- Toast yang masih tampil terbawa ke halaman berikutnya; animasi masuk header/bottom-nav hanya di halaman pertama kunjungan; animasi background melanjutkan fasenya antarhalaman.

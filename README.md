## Integrasi Fullstack Vue, REST API, dan Alur Transaksi (Chapter 10)
**Developed by:** Syaifullah Hidayat (NIM: 23367052)

* **Versi Node.js & Database:** Node.js v22.18.0 (ESM) dan PostgreSQL (`sisabaik_dev` dengan tambahan migrasi `database/12_alur_transaksi.sql`).
* **Jalur Pengembangan (Dev) & Proxy:**
  * Backend API berjalan di `http://127.0.0.1:3000` (`npm run dev` pada root).
  * Frontend Vue berjalan di `http://127.0.0.1:5173` (`npm --prefix frontend run dev`), dengan Vite proxy yang meneruskan awalan `/api` langsung ke port `3000`.
* **Jalur Preview & Build Lokal:**
  * Preview hasil kompilasi frontend berjalan di `http://127.0.0.1:4173` (`npm --prefix frontend run preview`).
  * Build satu origin dijalankan dari root dengan `npm run build` lalu `npm run start:local` sehingga aplikasi Vue (`frontend/dist`) dan `/api` disajikan bersamaan pada `http://127.0.0.1:3000`.
* **Autentikasi & Peninjauan Usaha:**
  * Pendaftaran mandiri dipisahkan untuk peran pembeli (`/api/auth/registrasi`) dan penyedia (`/api/auth/registrasi-penyedia`) dengan kata sandi minimal 15 karakter (maksimal 72 byte UTF-8).
  * Akun penyedia baru berstatus awal *menunggu peninjauan* dan harus diverifikasi oleh operator melalui perintah `npm run penyedia:verifikasi -- <email>` sebelum dapat mengelola penawaran dan pesanan.
* **Payload Foto, Optimasi Gambar, & Kontrol Versi:**
  * Unggahan foto penawaran (JPEG, PNG, atau WebP maksimal 2 MiB) dikirim dalam format data URL dengan batas payload JSON `3mb` khusus rute `POST`/`PUT /api/penawaran`, lalu dikonversi server menjadi WebP terkompresi menggunakan `sharp`.
  * Pembaruan penawaran menyertakan nomor `versi` (*optimistic locking*) untuk mencegah penimpaan stok akibat pembaruan usang (`409 OFFER_CHANGED`).
* **Alur Transaksi, Pemulihan Stok, & Metrik Dashboard:**
  * Pemesanan hanya mengirimkan `penawaranId` dan `kuantitas`; harga dan total dihitung dari snapshot server.
  * Stok berkurang saat reservasi dibuat dan dipulihkan satu kali jika pesanan dibatalkan sebelum diambil atau ditolak penyedia sebelum konfirmasi.
  * Pembayaran dilakukan di tempat (`di-tempat`) atau otomatis `gratis` untuk harga Rp 0.
  * Metrik dashboard memisahkan pesanan berjalan, pesanan selesai tersalurkan (`pendapatanDiterima` / `totalBelanja`), serta selisih harga normal (`penghematanPembeli` / `diskonDiberikan`).

### Batasan Sistem Aktual & Hasil Pengujian
* **Batasan Aktual:**
  * Token akses hanya disimpan di memori RAM aplikasi (sesi otomatis hilang saat halaman di-refresh).
  * Belum tersedia integrasi *payment gateway* daring, fitur reset kata sandi, verifikasi email otomatis, *idempotency key*, pembaruan *realtime* (WebSocket/SSE), serta perhitungan penghematan biaya operasional penjual.
* **Ringkasan Pengujian:**
  * Seluruh 26 skenario pengujian otomatis (`npm run workflow:test`) lulus mencakup validasi foto, cakupan peran, snapshot harga, transisi status pesanan, pembatalan, dan agregasi dashboard.
  * Pengujian manual pada mode dev (`5173`), preview (`4173`), dan build satu origin (`3000`) berjalan sesuai skenario tanpa menyertakan kredensial sensitif maupun token pada repositori.

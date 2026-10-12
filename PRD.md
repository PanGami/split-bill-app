# PRD — Split Bill

**Versi:** 1.0 (MVP) · **Platform:** Web (mobile-first)

## 1. Ringkasan

Split Bill adalah aplikasi web untuk membagi tagihan makan bareng secara adil, termasuk item yang dimakan bersama, pajak, service charge, dan diskon. Tanpa login, tanpa instalasi, dan bisa dipakai offline.

## 2. Masalah

- Hitung manual ribet dan sering salah.
- Pesanan tiap orang berbeda, dan ada item yang dimakan bersama.
- Pajak dan service sering dibagi rata padahal tidak adil bagi yang pesan lebih sedikit.
- Ujung-ujungnya ada yang tekor atau canggung saat menagih.

## 3. Target Pengguna

Teman, rekan kerja, dan keluarga yang makan atau nongkrong bareng (usia 17-40, pengguna utama mobile).

**Persona singkat:** *Rina (26)*, karyawan yang biasa jadi "bendahara dadakan" saat makan tim. Ia ingin hasil cepat yang tidak bisa diperdebatkan, lalu langsung dikirim ke grup WhatsApp.

## 4. Tujuan

- Hitung patungan kurang dari 2 menit.
- Hasil akurat dan mudah dipahami.
- Bisa dipakai tanpa login atau instalasi.

## 5. Fitur Utama (MVP)

| # | Fitur | Keterangan | Kriteria penerimaan |
| --- | --- | --- | --- |
| 1 | Kelola peserta | Tambah/hapus nama orang | Nama wajib, unik, maks 30 karakter; hapus peserta membersihkan assign-nya |
| 2 | Input item | Nama, harga, jumlah | Harga dan jumlah bilangan bulat ≥ 1; total baris otomatis |
| 3 | Assign item | Pilih siapa saja yang makan item (satu atau banyak orang) | Minimal 1 orang per item; ada tombol "Semua"; item tanpa assign ditandai dan memblokir hasil |
| 4 | Pajak & service | Input persen (PB1/PPN, service), dibagi proporsional | Rentang 0-100, desimal diizinkan |
| 5 | Diskon | Nominal atau persen (opsional) | Tidak boleh melebihi subtotal |
| 6 | Hasil akhir | Rincian per orang + total | Jumlah semua orang = total tagihan, selalu |
| 7 | Salin/bagikan hasil | Teks siap kirim ke WhatsApp (wa.me) | Teks terenkode benar; ada fallback salin |
| 8 | Simpan otomatis | localStorage | Data bertahan setelah refresh; data rusak tidak membuat error |

## 6. Fitur Lanjutan (Nanti)

- Riwayat bill
- Pembulatan (ke 100/500/1000)
- Siapa yang membayar di muka + hitung siapa utang ke siapa
- Cetak/ekspor hasil (`window.print()`)
- Dark mode

## 7. Alur Pengguna

1. Tambah peserta
2. Input item pesanan
3. Tandai siapa makan apa
4. Atur pajak/service/diskon
5. Lihat hasil dan bagikan

## 8. Aturan Perhitungan

**Prinsip dari brief:**

- Subtotal orang = jumlah (harga item ÷ jumlah pemakan).
- Pajak/service/diskon dibagi proporsional terhadap subtotal masing-masing.
- Total semua orang harus sama dengan total tagihan (selisih pembulatan dialokasikan ke satu orang).

**Spesifikasi rinci (keputusan desain):**

| Langkah | Rumus |
| --- | --- |
| Diskon | Nominal, atau persen × subtotal semua |
| Service | persen service × (subtotal − diskon) |
| Pajak | persen pajak × (subtotal − diskon + service) |
| Total tagihan | subtotal − diskon + service + pajak, dibulatkan ke Rupiah terdekat |
| Bagian per orang | total tagihan × (subtotal orang ÷ subtotal semua) |
| Pembulatan | Metode *largest remainder*: lantai-kan tiap orang, bagikan sisa Rupiah satu per satu ke pecahan terbesar (seri: urutan daftar) |

**Contoh hitung (acuan uji):**

- Peserta: Ani, Budi, Citra.
- Item: Nasi Goreng 2 × 25.000 (Ani, Budi); Es Teh 3 × 8.000 (semua); Pizza 1 × 90.000 (Budi, Citra).
- Subtotal: Ani 33.000 · Budi 78.000 · Citra 53.000 = 164.000.
- Diskon 10% = 16.400 → 147.600. Service 5% = 7.380 → 154.980. Pajak 10% = 15.498.
- **Total tagihan: Rp170.478.**
- Hasil: Ani **Rp34.304**, Budi **Rp81.081**, Citra **Rp55.093** (jumlah = 170.478).
- Ani dan Citra sama-sama punya sisa pecahan 0,5; sisa 1 Rupiah diberikan ke Ani karena urutan lebih awal.

## 9. Tech Stack

- HTML + CSS murni (Flexbox/Grid, CSS variables)
- JavaScript vanilla
- localStorage untuk penyimpanan
- `Intl.NumberFormat` untuk format Rupiah
- Satu file `index.html`, tanpa library eksternal

## 10. Kebutuhan Non-Fungsional

| Area | Persyaratan |
| --- | --- |
| Mobile-first | Dirancang mulai lebar 360px, tanpa scroll horizontal, tap target ≥ 44px |
| Offline | Seluruh fungsi berjalan tanpa jaringan; tanpa CDN/font eksternal |
| Validasi | Tidak boleh kosong, negatif, atau NaN; pesan error jelas dalam Bahasa Indonesia |
| Aksesibilitas | Label form, kontras WCAG AA, fokus terlihat, error tidak hanya dengan warna |
| Keamanan | Input pengguna tidak pernah disisipkan lewat `innerHTML` |
| Keandalan | Nol error di konsol; localStorage rusak jatuh ke keadaan kosong |
| Performa | Hitung ulang instan untuk hingga 20 peserta dan 50 item |

## 11. Metrik Keberhasilan

- Hasil akurat: total per orang = total tagihan pada 100% skenario uji.
- Proses selesai < 2 menit untuk bill tipikal (4 orang, 6 item).
- Berfungsi di browser mobile modern (Chrome Android, Safari iOS) tanpa error.

## 12. Model Data (usulan)

```
state = {
  version: 1,
  people:  [{ id, name }],
  items:   [{ id, name, price, qty, assignees: [personId] }],
  charges: { servicePct, taxPct,
             discount: { mode: "nominal" | "percent", value } },
  createdAt
}
```

## 13. Kasus Tepi

- Belum ada peserta atau item: tampilkan *empty state*, bukan error.
- Item tanpa pemakan: ditandai, hasil diblokir sampai diperbaiki.
- Peserta dihapus saat masih di-assign: minta konfirmasi, lalu bersihkan assign.
- Diskon melebihi subtotal: tolak dengan pesan jelas.
- Satu item dibagi 3 orang dengan harga yang tidak habis dibagi: ditangani oleh aturan pembulatan.
- Penyimpanan penuh atau diblokir (mode privat): aplikasi tetap jalan, tampilkan info bahwa data tidak tersimpan.

## 14. Di Luar Cakupan MVP

Riwayat bill, pembulatan 100/500/1000, pelacakan pembayar di muka dan utang, cetak/ekspor, dark mode, akun pengguna, dan backend.

## 15. Risiko & Pertanyaan Terbuka

| Item | Catatan |
| --- | --- |
| Urutan pajak vs service | Default: pajak dihitung setelah service. Perlu opsi "pajak atas subtotal saja" di versi berikutnya? |
| Struk dengan harga sudah termasuk pajak | Belum didukung di MVP |
| Format pesan WhatsApp | Panjang pesan bisa terpotong pada bill besar; ringkas bila perlu |
| Clipboard API | Butuh HTTPS; siapkan fallback `execCommand` / seleksi manual |

## 16. Rencana Rilis

- **MVP:** 8 fitur di atas dalam satu `index.html`.
- **v1.1:** pembulatan, dark mode, cetak.
- **v1.2:** riwayat bill, pembayar di muka + utang antar-orang. 
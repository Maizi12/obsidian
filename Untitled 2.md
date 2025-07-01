Berikut adalah contoh pengisian _Risk Assessment_ berdasarkan format yang Anda berikan. Untuk mempermudah pemahaman, kita ambil **Universitas** sebagai institusi yang dianalisis, dan scope-nya adalah **Layanan Sistem Informasi Akademik (SIAKAD)**.

---

### ✅ Contoh Pengisian Risk Assessment – _Scope: Sistem Informasi Akademik Universitas_

| ID Risiko | Aset                                      | Proses Bisnis                       | Pemilik Risiko       | Vulnerability                                   | Threat                                       | Impact                       | Kategori Risiko (CIA)          | Kemungkinan (1–5) | Dampak (1–5) | Level Risiko | Kontrol Mitigasi                                               | Strategi Mitigasi | Rencana Mitigasi                                             | Tenggat        | Realisasi Mitigasi | Kemungkinan (Setelah) | Dampak (Setelah) | Risiko Residual | Diterima / Ditransfer |
| --------- | ----------------------------------------- | ----------------------------------- | -------------------- | ----------------------------------------------- | -------------------------------------------- | ---------------------------- | ------------------------------ | ----------------- | ------------ | ------------ | -------------------------------------------------------------- | ----------------- | ------------------------------------------------------------ | -------------- | ------------------ | --------------------- | ---------------- | --------------- | --------------------- |
| A-1       | INFORMASI AKADEMIK MAHASISWA (elektronik) | Akses data mahasiswa melalui SIAKAD | Kepala Biro Akademik | Sistem tidak memiliki enkripsi penyimpanan data | Peretasan / pencurian data                   | Kebocoran data pribadi       | Confidentiality                | 4                 | 5            | 20 (Tinggi)  | Implementasi enkripsi AES-256 dan kontrol akses berbasis peran | Mengendalikan     | Terapkan enkripsi, audit akses, dan pelatihan keamanan data  | 30 Juni 2025   | Dalam Proses       | 2                     | 3                | 6 (Rendah)      | Diterima              |
| B-1       | SIAKAD (Software Aplikasi)                | Login Mahasiswa/Dosen               | Kepala UPT TIK       | Password lemah, tidak ada MFA                   | Brute-force login                            | Akses ilegal ke sistem       | Confidentiality & Integrity    | 3                 | 4            | 12 (Sedang)  | Terapkan autentikasi dua faktor (2FA) dan kebijakan password   | Mengendalikan     | Terapkan 2FA, pengingat password, dan pembatasan login gagal | 15 Juli 2025   | Belum dimulai      | 1                     | 2                | 2 (Rendah)      | Diterima              |
| C-1       | SERVER SIAKAD                             | Hosting layanan SIAKAD              | Kepala UPT TIK       | Tidak ada UPS atau backup failover              | Server down karena listrik padam             | Sistem tidak bisa diakses    | Availability                   | 2                 | 4            | 8 (Sedang)   | Pasang UPS dan backup server                                   | Mengendalikan     | Pengadaan UPS dan replikasi sistem di server cadangan        | 1 Agustus 2025 | Dalam Proses       | 1                     | 2                | 2 (Rendah)      | Diterima              |
| D-1       | RUANG SERVER                              | Pengelolaan infrastruktur           | Kepala UPT TIK       | Akses ruang server terbuka                      | Akses fisik tidak sah                        | Perusakan perangkat keras    | Availability & Confidentiality | 2                 | 4            | 8 (Sedang)   | Pasang kunci akses & kamera CCTV                               | Mengendalikan     | Kontrol akses fisik dengan finger scan dan pemantauan CCTV   | 10 Juli 2025   | Dalam Proses       | 1                     | 2                | 2 (Rendah)      | Diterima              |
| E-1       | STAFF OPERATOR SIAKAD                     | Pemrosesan data akademik            | Kepala UPT TIK       | Kurangnya pelatihan keamanan informasi          | Kesalahan input atau kebocoran tidak sengaja | Data tidak akurat atau bocor | Integrity & Confidentiality    | 3                 | 3            | 9 (Sedang)   | Pelatihan rutin dan SOP validasi data                          | Mengendalikan     | Adakan pelatihan keamanan informasi & SOP penanganan data    | 20 Juli 2025   | Dalam Proses       | 1                     | 2                | 2 (Rendah)      | Diterima              |

---

### ✅ Penjelasan Tambahan:

- **CIA**: Mengacu pada prinsip _Confidentiality, Integrity, Availability_.
    
- **Level Risiko**: Hasil perkalian kemungkinan dan dampak. Biasanya:
    
    - 1–4 = Rendah
        
    - 5–14 = Sedang
        
    - 15–25 = Tinggi
        
- **Strategi Mitigasi**:
    
    - **Menghindari**: Tidak melakukan proses yang menimbulkan risiko
        
    - **Mengendalikan**: Mengurangi kemungkinan/dampak risiko
        
    - **Mentransfer**: Asuransi, kontrak pihak ketiga
        
    - **Menerima**: Jika risiko sudah cukup rendah atau biayanya terlalu tinggi untuk ditangani
        

---

Kalau kamu ingin saya bantu **isi file tabel Word/Excel-nya langsung**, atau ingin **versi lain untuk perusahaan atau instansi tertentu**, cukup beri tahu scope dan jenis organisasi yang dipilih.
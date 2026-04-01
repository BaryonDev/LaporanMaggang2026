# Laporan Lengkap Pelaksanaan Magang: Pengembangan Sistem Absensi Anti-Spoofing Terintegrasi BKNOA
**Periode Magang**: 3 Februari 2026 – 1 April 2026

Laporan ini mendokumentasikan progres teknis pengembangan sistem absensi presisi tinggi berbasis kecerdasan buatan (Face Anti-Spoofing). Selama dua bulan masa pengembangan, platform mengalami iterasi teknologi yang sangat dinamis untuk mengantisipasi celah keamanan (*spoofing* lokasi maupun *spoofing* identitas wajah). 

Berikut adalah rincian fungsionalitas dan detail cara pembuatan tiap fitur berdasarkan linimasa proyek:

---

## 1. Modul Dasbor Admin Panel (Selesai 10 Februari 2026)
Pengembangan difokuskan untuk memfasilitasi kebutuhan administrasi *back-office* menggunakan framework **PHP Laravel**.

*   **Pembuatan Entitas & Skema Database Dasar**
    Diinisialisasi dengan menggunakan perintah `composer create-project laravel/laravel`. Database di-*design* secara relasional di MySQL dengan sistem migrasi bawaan Artisan, membentuk file fundamental `antispoof_attendance.sql` yang menaungi struktur tabel seperti `users`, `departments`, `events`, `venues` dan `attendances`. Lingkungan `env` disesuaikan untuk kredensial basis data.

*   **Autentikasi & Role Access Control (RAC)**
    Sistem form login direkayasa dengan memodifikasi bawaan kontrol `Auth::attempt`. Hak akses fungsional (`role_enum`) disuntikkan parameter pengecekan *Middleware* ketat agar bisa mendelegasikan visibilitas menu berbeda untuk fungsi `User`, `Admin`, `Superadmin`, dan belakangan diperluas menopang hirarki otorisasi struktural level `Supervisor`.

*   **Manajemen Master Data & Event Venue**
    Dibangun modul antarmuka antrean Manajemen Pegawai (CRUD Data Pegawai) yang tertaut relasi satu arah ke struktur Departemen (*One-to-Many*). Disematkan pula relasi majemuk di level model Eloquent ORM di mana tabel pembuatan agenda kalender (**Event**) bisa memanggil basis identitas tempat/fasilitas lokasi acara (**Master Venue**) lewat struktur tabel intervensi ID jembatan. 

*   **Monitoring Harian, Pagination, & Log Akses**
    Mendesain halaman khusus tabular (*datatables*) untuk menampung riwayat pantauan jam masuk karyawan mutakhir (*real-time attendance*). Untuk mengatasi kelambatan penarikan riwayat pada query SQL beban besar, query dilengkapi metode `->paginate(20)` langsung dari *controller*. Seluruh jejak aksi klik kursor Admin terekam lewat fungsionalitas log aktivitas khusus (`admin_log`).

*   **Fitur Export Laporan (PDF & Excel)**
    Fitur ini diciptakan dengan mengintegrasikan ekstensi *library* eksternal `barryvdh/laravel-dompdf` (menerjemahkan output spesifik tag HTML ke struktur elemen PDF), dan `Maatwebsite/Laravel-Excel` untuk mengekalkan kalkulasi rekap format *spreadsheet* (CSV/XLSX). Disisipkan juga optimasi CSS bawaan agar proses lompatan spasi halaman tabel antar rentang data (*page breaks PDF*) tak lagi terpotong patah menyebalkan.

---

## 2. Modul Web-Based App & Webview (11 - 13 Februari) [DIBATALKAN / PIVOT]
Berselang 3 hari tepat setelah pengerjaan Dasbor Admin disepakati 100% tuntas, inisiasi pengerjaan klien karyawan (*mobile app*) langsung diselenggarakan. 

*   **Eksperimen Pembuatan Webview .APK**
    Pada awalnya, antarmuka aplikasi absen dicetuskan dengan metode pengerjaan hemat waktu: membangun aplikasi berbasis HTML/JS murni (*web-base UI*) lalu membungkus sistem *routing* URL-nya agar terinstal menyerupai instrumen aplikasi genggam Android `.apk` dengan metode *Native Webview* (Setara konsep Capacitor/Cordova murni).

*   **Celah Kejahatan Pemalsuan Lokasi (Fake GPS)**
    Begitu fungsi tangkapan koordinat diimplementasikan menggunakan arsitektur Geolokasi Web API standar browser di dalam *Webview*, tim pengembangan menyadari kerentanan celah keamanan parah. Basis browser tak bisa mengekstrak identitas manipulatif (parameter proteksi OS) jika pengguna memakai trik spesialis modifikasi *Fake GPS / Mock Location emulator* yang lazim ditemukan di Play Store Android. 
    
    Karena hal tersebut bisa menggugurkan keabsahan presensi berbasis geofencing radius kantor, **metode Webview dibatalkan total sepenuhnya**. Diambil keputusan tegas dari sisi *engineering* untuk sepenuhnya pindah (pivot mutlak) menggunakan kerangka ekosistem native/cross-platform mumpuni: yakni **Flutter** untuk mendeteksi status pemalsuan hardware asli GPS.

---

## 3. Modul Backend REST API & Face AI Microservice (16 - 21 Februari 2026)
Sebagai penopang Flutter Android nantinya, dibangun dua mesin komputasi internal, yakni API Integrasi (Laravel) dan API Wajah Pintar (Python).

*   **Endpoint API RESTful Utama (APIBKNOA)**
    Dirangkai agar memproses parameter masuk (koordinat geolokasi lat/long dan Base64 foto muka) dari aplikasi genggam (Flutter). Koneksinya dilindungi otentikasi ketat menggunakan pustaka pelindung `tymon/jwt-auth` bertugas membangkitkan dan menagih konfirmasi kunci sesi *Bearer Token*. Disempurnakan pula sinkronisasi rekapan absensi backend untuk perizinan flag tipe status libur '*Day Off*' (DO).

*   **Riset Artificial Intelligence Face Anti-Spoofing**
    Proses algoritma deteksi manipulasi diuji coba membandingkan banyak infrastruktur dari arsitektur *Computer Vision library OpenCV*. Tantangannya mengulik beban pre-trained model open-source yang berpresisi akurat mutlak mengendus tipu foto layar perangkat. Namun sebagian besar berakibat sangat berat kalkulasinya pada *resource* CPU / memori alat mid-end.

*   **Penemuan Algoritma 'Minifast v2'**
    Tepat di tangga 19 Februari, ditemukan inovasi arsitektur CNN super ringan nan gesit yakni arsitektur standar **Minifast v2**. Dibanding model lain, Minifast v2 diakui brilian karena sukses menahan *spoofing* wajah foto fisik statis (anti-liveness) tanpa memerlukan kalkulasi lama mutlak yang memakan daya hardware besar beruntun.

*   **Pembuatan Server Microservice (Flask Phtyhon)**
    Diwadahkan pada infrastruktur _wrapper mini-server_ HTTP memakai ekosistem Phyton (`app.py`). Kerangka spesifik *Flask* / *FastAPI* ini murni difungsikan melayani penerimaan *routing Base64 string* objek foto muka dari APIBKNOA Laravel, lalu menyerahkannya sepenuhnya kepada model sistem inferensi *Minifast* dan melontarkan hasil kembalian konklusi tipe Boolean True/False (Verifikasi Lolos Wajah / Gagal).

---

## 4. Modul Aplikasi Mobile Native (Flutter) & Rilis Android (23 Februari - 19 Maret 2026)
Pengembangan platform antarmuka klien presensi karyawan mutlak utama yang sukses dibangun memakai framework cross-platform Flutter.

*   **Slicing UI/UX Desain & Arsitektur Riverpod**  
    Proses translasi mendesain antarmuka login yang responsibilitasnya memandu konversi visual ke widget komponen susun Flutter (`Scaffold`, `Column`, `TextField`). Penanganan sesi login (State Management token autentikasi API) seluruhnya diterapkan mematuhi aturan manajemen status otomatis canggih milik pustaka struktur arsitektur **Riverpod**.

*   **Pembekalan Hak Akses Kamera Asli (Camera V2 API)**
    Tidak seperti *webview*, tangkapan kualitas foto *live* Flutter dibuka secara native menyelami fitur plugin mutlak `camera` Flutter package guna mengkonfigurasi rendering kualitas potret berkecepatan *high-precision*.

*   **Integrasi Plugin Geolokasi Canggih (Anti Fake GPS)**  
    Fokus utama pivot Flutter dibuktikan di sini: parameter lokasi diamankan pustaka `geolocator`. Sistem mengurai perhitungan radius metrik kelengkungan bumi (Haversine Formula) menuju target kordinat kantor, namun dibentengi kode khusus `<position.isMocked>` milik OS native Android. Sekali Flutter mengidentifikasi lokasi karyawan termodifikasi fiktif oleh persekongkolan *tools Fake GPS*, sinkronisasi absensi akan ditolak paksa seutuhnya (*Rejected*). 

*   **Refactoring Fitur Modul Wajah (Dari 5-Step menuju Spesifikasi 1-Step Loop)**  
    Awalnya, modul pergerakan kamera disuntikkan parameter scanning wujud pendeteksi kontur fisik memutar lelah (meminta respon karyawan kedip mata, menoleh kanan, buka senyum, dll) hingga 5 jeda (*5-step scanning*). Hal ini terbukti amat membebani daya tahan limit memori perangkat kelas menengah sehingga memantik error *App Freeze*.
    Logika absensi mutlak di-*refactor*. Pemindaian diubah drastis ke pergerakan cepat **1-step verifikasi**. Menggunakan siklus loop cepat potret bingkai foto (*real-time frame drop*), foto sekejap diserahkan utuh lewat transmisi Face API backend server *(Minifast v2)* untuk menilai kelayakan 3D Liveness muka karyawan seketika tanpa perlu gestur memutar menyiksakan lagi. Waktu antrean berhasil ditekan sangat singkat namun dengan kualitas keamanan (*false-rejection rate*) akurasi yang dijamin mutlak aman berpresisi ketat tinggi.

*   **Mode Kiosk Otomatis & Trigger Intervensi Triple-Click**  
    Antarmuka perangkat presensi instalasi menetap khusus (*Kiosk Tablet*) juga dibangun berlapis perpaduan desain mode geladak hitam minimalis (*monokrom elegan*). Ditanam pula pengawas aksi sentuh modul `<GestureDetector>` di pojok logo. Jika beridentifikasi sentuhan gesit tiga kali *tap* seketika (*triple-click*), platform sekonyong memerintah mode integrasi penyembunyian bar fungsi utuh OS sistem gawai ke dalam format presentasi utuh mutlak: `SystemChrome.setEnabledSystemUIMode(SystemUiMode.immersiveSticky)`. 

*   **Desain Logo Identitas Aplikasi & Revisi UI Hierarki (12 Maret 2026)**
    Mengalokasikan waktu operasional khusus merancang ulang identitas visual merek aplikasi (Logo BKNOA / MAVEN) menjadi format `.png` mutlak beresolusi tinggi. Aset dirancang secara selektif agar tampil lebih kontras pada *app drawer* OS Android maupun *homescreen* iPhone, lengkap dengan pembungkusan wadah putih bundar agar selaras dengan skema warna layar pangkalan Kiosk Mode.

*   **Pemecahan Blokade Google Play Protect & Distribusi APK (16 Maret 2026)**
    Saat uji instalasi kompilasi debug mentah, Android mencekal APK dengan red flag OS Play Protect internal Google. Rintangan ini diberantas sukses berkat penataan implementasi arsitektur penyusunan tanda rahasia sertifikat `key.jks` dan parameter aktivasi konfigurasi keamanan murni *V2 Scheme Signature* (`v2SigningEnabled true`) secara sistematis masuk di kerangka file direktori fundamental `build.gradle` properti build Android, selaras dengan pembenaman penyandian nama kode fungsi variabel susunan kelas (`Proguard obfuscation`).
    Tepat di tanggal **16 Maret**, berkas final *Alpha-Test Production* secara masal dikompilasi mutlak melalui rute param CLI: `flutter build apk --release` dan sukses menata langkah Deployment fase krusial aplikasi didistribusikan. 

*   **Masalah OS iOS iPhone Apple Mandek Fatal (19 Maret 2026)**  
    Langkah pengembangan ditransfer menyongsong sinkronisasi library base iOS CocoaPods dan sisipan permintaan parameter hak cipta sensor lokasi Apple `NSLocationWhenInUseUsageDescription` di manifest OS `Info.plist`. Akan tetapi, langkah mentok membentur tembok limitasi di hari kamis rilis terkahir *development* jelang dimulainya linimasa rehat liburan 29 Maret: Seluruh operasi rute kompilasi render platform XCode spesifik aplikasi platform Apple native harus diberlakukan di aras fisik *hardware device Mac/iMac*. Karena eksistensi komputer tersebut benar-benar kosong nihil secara inventaris lingkungan kerja kantor, operasi pembangunan infrastruktur utuh iOS Native app murni ditangguhkan (Deadlock terblokir mutlak total menyeluruh).

---

## 5. Strategi Pivot Web PWA iOS Web App Khusus Safari (30 Maret - 1 April 2026)
Sebagai langkah penyelamatan kompilasi fungsional spesifikasi device karyawan genggam gawai iPhone, arsitektur di-*pivot* kilat membidik browser bawaan iOS (Safari). 

*   **Slicing UI React JS, Vite, + Tailwind CSS**
    Proyek sekunder (*BKNOAweb-ios*) diinisiasi membangun halaman basis UI meniru desain absolut native Flutter Android murni dikerakan menggunakan komposisi komponen library dinamis mutakhir **React.JS** dengan konfigurasi ketat kompilasi bundler performa secepat kilat milik **Vite**. Format antarmuka pewarnaan CSS digubah responsif gesit lewat framework kelas utilitas **Tailwind CSS**. Ditambahkan modifikasi khusus orientasi resolusi properti css `env(safe-area-inset-top)` guna menghindari proporsi lengkungan terpotong bezel notch lengkung layar depan *Dynamic Island* iPhone mutlak.

*   **Integrasi Modul WebRTC Camera Safari & Sinkronisasi Ekstrem API Web**
    Tangkapan stream kamera didongkrak secara browser dengan menyisipkan intervensi fitur parameter API Browser natif fundamental yakni metode standar canggih `navigator.mediaDevices.getUserMedia()`.

*   **Skema Penjagaan Hardware & Resolusi Anti Desktop (Penyelesaian Akhir)**
    Karena berbasis web-app PWA murni URL, terdapat risiko tautan dibuka melalui peramban raksasa Komputer PC kantor (Desktop Bypass). Skema mitigasi disuntik lapis validasi berlapis mengekstrak murni header pengecekan orientasi identifikasi perangkat mutlak `navigator.userAgent`. Lapis keamanan tambahan menyertakan blok dimensi script JS `Aspect Ratio` spesialis portrait dimensi batas minimum handphone rasio 16:9 vertical utuh (memaksa akses laman diblokir murni mati tertutup layar penuh bilamana orientasi pixel membuktikan browser disulap melalui dekstop macOS / OS Windows PC sekalipun).
    
Milestone per tanggal **1 April 2026**, instalasi folder `/dist` arsitektur final PWA Web tersebut diekstrak merangkai fase krusial *Deployment* rilis akhir integrasi spesifikasi ke pelabuhan internal jaringan server Nginx produk mutakhir. Total operasional rangkaian proyek disepakati tuntas sepenuhnya (*Finalized 100%*).

# Laporan Harian Magang: Pengembangan Sistem Absensi Terintegrasi BKNOA
**Periode Magang**: 3 Februari 2026 – 1 April 2026

Laporan ini mendokumentasikan progres harian pengembangan sistem absensi berbasis kecerdasan buatan (*Face Anti-Spoofing*) dan validasi lokasi (*Geofencing*). Selama masa pengerjaan, sistem terbagi menjadi beberapa bagian: Admin Panel, Backend API, Face API, Aplikasi Mobile (Flutter), dan Progressive Web App (PWA).

Berikut adalah rincian fungsionalitas harian yang dikerjakan:

---

## FASE 1: Dasbor Admin Panel (Laravel)

### Selasa, 3 Februari 2026
**Kegiatan:** Setup awal sistem, instalasi kerangka dasar Laravel, dan perancangan skema database.
**Fokus Teknis:** Proyek dimulai dengan melakukan instalasi framework Laravel menggunakan Composer (`composer create-project`). Selanjutnya, merancang struktur database relasional di MySQL (`antispoof_attendance.sql`) yang diimplementasikan melalui *migration* bawaan Laravel. Tabel utama yang disiapkan meliputi `users`, `departments`, `events`, `venues`, dan data absensi.

### Rabu, 4 Februari 2026
**Kegiatan:** Pembuatan antarmuka visual (Slicing UI) untuk Dasbor Admin dan pembuatan hak akses.
**Fokus Teknis:** Merancang desain tampilan menggunakan template Blade dan *Tailwind CSS*. Untuk sistem keamanan, fitur login dibuat dengan menyesuaikan fungsi `Auth::attempt`. Selain itu, hak akses dibedakan menggunakan *Middleware* agar menu yang muncul dapat disesuaikan untuk peran seperti *User*, *Admin*, dan *Superadmin*.

### Kamis, 5 Februari 2026
**Kegiatan:** Pengerjaan halaman manajemen Master Data Pegawai dan Departemen.
**Fokus Teknis:** Membuat fitur dasar CRUD (Create, Read, Update, Delete) untuk mendata profil pegawai. Tabel pegawai kemudian dihubungkan dengan tabel departemen menggunakan relasi *One-to-Many* pada tingkat model Eloquent, sehingga satu departemen dapat menaungi banyak pekerja.

### Jumat, 6 Februari 2026
**Kegiatan:** Pembuatan halaman tabel Rekapitulasi Presensi Harian.
**Fokus Teknis:** Membuat komponen tabel interaktif (*Datatables*) untuk memantau jam kehadiran karyawan setiap harinya. Untuk mencegah kemacetan (*loading lambat*) saat database mengambil kueri data dalam jumlah besar, sistem penarikan data diberi perintah `->paginate(20)` agar diload per halaman secara langsung dari backend.

### Sabtu, 7 Februari 2026
**Kegiatan:** Pembuatan manajemen pengelolaan Event dan Master Venue.
**Fokus Teknis:** Mengembangkan modul untuk menjadwalkan suatu acara absensi (*Event*). Karena sebuah acara dapat menempati beberapa tempat sekaligus atau sebaliknya, relasi tabel diatur menggunakan tipe relasi *Many-to-Many* dengan tabel referensi lokasi gedung/fasilitas absensi (*Venue*).

### Senin, 9 Februari 2026
**Kegiatan:** Implementasi fitur filter pencarian dan cetak laporan PDF/Excel.
**Fokus Teknis:** Fitur penarikan berkas laporan ditambahkan menggunakan pustaka `barryvdh/laravel-dompdf` untuk melakukan konversi file HTML ke bentuk PDF. Sedangkan untuk rekap data berbentuk format *spreadsheet* menggunakan `Maatwebsite/Laravel-Excel`. Kode CSS khusus (`page-break-inside: avoid`) juga diaplikasikan agar format tabel dasar PDF tidak terpotong antara halaman pertama dan kedua.

### Selasa, 10 Februari 2026
**Kegiatan:** Penyesuaian responsivitas Panel dan pembuatan fitur perekam rekam jejak.
**Fokus Teknis:** Mengatasi bug kecil pada pergeseran tata letak UI saat Dasbor Admin dibuka menggunakan browser di ponsel kecil. Bersamaan dengan penyelesaian Admin Panel, fitur `admin_log` ditambahkan untuk merekam setiap aktivitas signifikan yang dilakukan pengguna setingkat Admin (misal merekam siapa yang menambah atau mengubah jam absen secara manual).

---

## FASE 2: Backend API Pokok, Pivot Batalnya Webview, dan Face AI

### Rabu, 11 Februari 2026
**Kegiatan:** Mulai mengatur rute untuk sistem API Utama, seraya memulai desain versi awal antarmuka karyawan (*Web-Base*).
**Fokus Teknis:** Fokus berpindah ke penyusunan rute REST API di Laravel untuk menangani lalu lintas data dari aplikasi. Di waktu yang sama, tim meriset pembuatan aplikasi (*font-end*) berbasis HTML/JS dinamis biasa, yang rencananya akan dipaket menjadi format aplikasi `.apk` menggunakan teknologi *Native Webview*.

### Kamis, 12 Februari 2026
**Kegiatan:** Konfigurasi modul keamanan *Bearer Token* di sisi API.
**Fokus Teknis:** Menyiapkan parameter perisai pelindung pada setiap jalur API. Memakai ekstensi library `tymon/jwt-auth` untuk menagih, membaca, dan membangkitkan kunci sementara *JSON Web Token* (JWT) yang didapatkan saat pengguna berhasil login ke dalam sistem aplikasi.

### Jumat, 13 Februari 2026
**Kegiatan:** Pembuatan rute API absensi dan temuan kerentanan *Fake GPS* pada eksperimen Webview.
**Fokus Teknis:** Ketika API absen disiapkan, sistem Geolocation API bawaan browser dalam implementasi purwarupa aplikasi *Webview* menunjukkan kendala fatal. Teknologi ini tidak memiliki dukungan ke level *Operating System* (OS) perangkat untuk mendeteksi apakah kordinat kustomer dimanipulasi dengan peretasan seperti alat aplikasi tiruan spesifik *Mock Location (Fake GPS)* yang populer di pasaran. 

### Sabtu, 14 Februari 2026
**Kegiatan:** Pivot kerangka pengembangan pindah ke *Native* dan uji ketahanan API.
**Fokus Teknis:** Mengingat integritas area absen (`Radius Geofence`) akan hancur jika pengguna memalsukan lokasi, maka konsep pembuatan perangkat perantara metode **Webview resmi dibatalkan total**. Langkah pivot diambil dengan merancang instruksi untuk berpindah membangun platform karyawan beralih adopsi total ke teknologi kerangka kerja **Flutter (Native Cross-Platform)** agar sanggup menembus sistem keamanan parameter deteksi Android *Mock Location Provider*. Skema rute endpoint API dieksekusi simulasi menggunakan *Postman HTTP Client*.

### Senin, 16 Februari 2026
**Kegiatan:** Memikirkan inisialisasi layanan microservice terpisah pengenalan Face API dan riset model liveness wajah.
**Fokus Teknis:** Karena fitur autentikasi menuntut pemeriksaan parameter wajah, tim mengawasi pembuatan arsitektur program Python mandiri dan membandingkan beberapa *library* tipe fungsional *OpenCV Computer Vision*. Fokusnya pada model kecerdasan buatan (*Artificial Intelligence*) spesifik untuk verifikasi keaslian parameter fisik objek *Anti-Spoofing Liveness*.

### Selasa, 17 Februari 2026
**Kegiatan:** Melakukan eksplorasi daya kinerja model pre-trained algoritma deteksi manipulasi di format mobile.
**Fokus Teknis:** Simulasi tes dengan menaruh topologi model kecerdasan pendeteksi identitas foto pada gawai spesifikasi kelas bawah. Banyak ditemukan bahwa arsitektur bawaan yang kompleks terlalu banyak memberikan beban komputasi CPU dan memori, sehingga tangkapan kamera dari gawai kelas *mid-end* menjadi mandek alias berhenti sementara (*freeze*).

### Rabu, 18 Februari 2026
**Kegiatan:** Memilah referensi kualitas akurasi tangkal pemalsuan tipe media fisik/layar HP.
**Fokus Teknis:** Sistem terus menuntut mekanisasi filter protektor algoritma *open-source* yang tidak hanya sigap mencegah manipulasi fisik (seperti disodorkan kerta cetakan foto / gambar gawai dari peranti eksternal), tetapi juga sangat ringan memakan RAM perangkat ketika melakukan inferensi perbandingan wajah.

### Kamis, 19 Februari 2026
**Kegiatan:** Penemuan model cerdas algoritma *Convolutional Neural Network (CNN) 'Minifast v2'*.
**Fokus Teknis:** Pencarian panjang menyingkap sebuah penemuan terbaik yang disebut arsitektur algoritma **"Minifast v2"**. Keistimewaan metode ini dibanding sistem serupa adalah ia berukuran super ringkas dalam menghitung nilai prediksi ancaman percaloan wajah foto fisik 3D secara presisi, namun tidak membuat perangkat pengguna patah-patah kala memprosesnya di Android murah.

### Jumat, 20 Februari 2026
**Kegiatan:** Penyusunan kode program mini-server Python wrapper menggunakan *Flask / FastAPI*.
**Fokus Teknis:** Kode program *Minifast v2* dikemas rapi menjadi server kecil backend berbasis port independen web di skrip internal Python `app.py` menggunakan framework Flask. Logic dirancang sangat sepele: ia akan menagih string terjemahan gambar karakter (`Base64 format image`) dari layanan Backend Utama Laravel, kemudian *Minifast* ini akan menganalisisnya dan menerbitkan jawaban prediksi tipe Boolean (`True / False` Wajah Lolos).

### Sabtu, 21 Februari 2026
**Kegiatan:** Integrasi service eksternal internal *Face API Python* dari Backend API Utama.
**Fokus Teknis:** Merangkai agar skrip *controller* di Laravel bisa mengoperkan setiap data rilis Base64 foto dari klien menyeberang langsung diteruskan lewat pemanggilan proksi internal (`cURL / Request`) langsung jatuh ke tangkapan port alamat peladen *Face Python AI*. Tes *response time* ditinjau lancar dalam bilangan detik. Selesai.

---

## FASE 3: Klien Android Native (Flutter), Kiosk Sistem, & Resolusi Bug Integrasi Kompilasi

### Senin, 23 Februari 2026
**Kegiatan:** Inisialisasi pondasi proyek aplikasi berbasis kerangka kerja ekosistem aplikasi lintas OS Flutter (*BKNOAapp*).
**Fokus Teknis:** Meneruskan wacana migrasi pivot ke native dengan menata *repository codebase* di baris perintah awal `flutter create`. Pemilihan konfigurasi paket *dependencies* pendukung ditaruh secara berurutan persis pada perancangan direktori *pubspec.yaml*. Segala berkas diatur menjadi modular (pisahkan UI, model, route, service).

### Selasa, 24 Februari 2026
**Kegiatan:** Merancang desain translasi grafis *Slicing UI Front-End* untuk halaman Login ke skema Widget *Dashboard*.
**Fokus Teknis:** Menyalin logika estetika panduan *mockup* menu antarmuka login yang dibuat lewat penyusunan instrumen struktural berlapis komponen turunan komponen dasar ekosistem Widget OS material (Membikin *Scaffold*, kolom `Column`, `Container`, dan fungsional tombol penadah pengiriman). Validasi tipe masukan tipe data `Regex Email` form ditanam.

### Rabu, 25 Februari 2026
**Kegiatan:** Mendesain antarmuka riwayat kehadiran plus menyajikan spesifikasi penyusunan estetika antarmuka UI spesial layar *Kiosk Mode*.
**Fokus Teknis:** Tata letak menu disiapkan mengakomodir rincian data jam ketidakhadiran dalam elemen gulir data *ListView*. Bersamaan dengan merakit khusus halaman presensi tablet Kiosk di mana tampilan bernuansa warna gelap nan *elegant black and white monokrom* guna ditampilan anggun proporsional menyambut resepsionis tamu kantor perusahaan. 

### Kamis, 26 Februari 2026
**Kegiatan:** Pergantian metode manipulasi navigasi kontrol memori asali via pengelolaan state mutakhir ekosistem pustaka `Riverpod`.
**Fokus Teknis:** Cara lama melakukan pengaturan perubahan layar UI dengan menyetel kondisi primitif statis (`setState()`) dibuang murni diganti memakai pola pengaturan *provider state architecture Package flutter_riverpod*. Sinkronisasi penyimpanan status login token diamankan terenkripsi di parameter ekosistem gawai dan akan ditukar reaktif perihal perintah *Observer Ref.Watch* di transisi halaman.

### Jumat, 27 Februari 2026
**Kegiatan:** Eksekusi plugin sensor Geolokasi dan pengecekan proteksi parameter kecurangan alat *Fake GPS*.
**Fokus Teknis:** Menyelesaikan masalah rentan lama dengan melibatkan *package geolocator* untuk meminta titik asli spesimen koordinat satelit modul sistem HP. Menggunakan rumusan matematis pengukur radius bengkok bumi *Haversine Formula* menakar batas area Geofencing jarak gedung pabrik presensi. Di titik keamanannya, program mendeteksi jika flag native tersembunyi berlabel `position.isMocked` menunjukkan indikator "Iya / True", maka absensi karyawan *dijegal langsung digagalkan* di tempat atas pertanda intervensi pemalsuan OS Fake Emulator GPS lokasi. Fitur sangat krusial.

### Sabtu, 28 Februari 2026
**Kegiatan:** Mengaktifkan hak pemakaian alat sistem Native sensor Lensa plugin Camera.
**Fokus Teknis:** Membuka komponen bawaaan hak izin pembedahan pengoperasian fungsi kontrol lensa *camera package* di gawai perangkat klien. Melakukan pembentukan *controller setup Preview layer UI tangkapan stream kamera HD layar muka resolusi real-time*.

### Senin, 2 Maret 2026
**Kegiatan:** Memprogram alur aliran parameter data menuju kerangka backend infrastruktur REST API utama (Laravel).
**Fokus Teknis:** Operasi penekanan tombol presensi menautkan rutinitas pengiriman form protokol asali JSON `HTTP Request`. Kode dikumpulkan mencakup kompilasi angka posisi *latitude/longitude*, lalu foto dirapatkan kedalam nilai tipe `image binary format base64 encoding`, dilampirkan otorisasi sandi gembok token JWT. Modul rilis aman dan menyuplai data murni ke form validasi penerimaan `Gateway API Laravel Attendance`.

### Selasa, 3 Maret 2026
**Kegiatan:** Membangkitkan rekayasa visual notifikaktor pendeteksi hasil validasi Face Anti Spoofing kegagalan/kelolosan.
**Fokus Teknis:** Jika deteksi peladen server AI Python di baris backend Minifast v2 menyimpulkan verifikasi muka tersebut curang, nilai API bakal mengirim tag False. Klien HP menangkap intervensi format kembalian OS tag ini lalu UI mengkonversi menerbitkan antarmuka *Snackbar error rejection notification* kotak dialog bertulis "Wajah Bukan Manusia / Fake Liveness Detected Rejection". 

### Rabu, 4 Maret 2026
**Kegiatan:** Melangsungkan pengujian lapangan *Quality Assurance (QA) User* perangkat interaktif gawai lapangan (*Android Test Device*).
**Fokus Teknis:** Eksekusi ujicoba pemindaian geofence radius dari sudut pojok lapangan, kemudian sengaja memutus putus sinyal perangkat kuota operator tipe paket internet lemah lemot *(Throttle Testing)*. Hal ditujukan untuk meneliti apabila ekosistem akan berujung me-*freeze/hang limit crash Out Of Server timeout*.

### Kamis, 5 Maret 2026
**Kegiatan:** Pemulihan kendala *Memory Leak* penumpukan kamera interaktif serta penerapan eksekusi pengulangan penagihan API logik otomatis.
**Fokus Teknis:** RAM limit RAM OS dicegah meluap dengan senantiasa melakukan fungsi pelepasan siklus sumber kontroler `dispose camera sensor` setiap beralih halaman menu. Diintegrasikan tambahan kode retry di perantara network HTTP sehingga bilamana request terbuang asbab putus nyala paket data internet kustomer, form dapat mengantre kembali mengirim *request post data payload* otomatis. 

### Jumat, 6 Maret 2026
**Kegiatan:** Menciptakan manuver intervensi trigger menu tersembunyi `Triple-Click` eksklusif tablet operasional Kiosk.
**Fokus Teknis:** Disembunyikan widget penangkap perabaan kontrol sentuh `<GestureDetector>` murni tak terlihat terhalang persis tertimbun di ikonik letak sudut *Logo Utama UI Kiosk Mode*. Sistem mengenali sekuen spesifik rutinitas ritmis: Menekan cepat berkala logo tap ketukan *sebanyak Tiga Kali berturutan presisi waktu miliseconds kaku jeli riel*, otomatis langsung platform menyembunyikan navigasi peramban Status Bar HP memahat berputar paksa utuh masuk rintangan ke `SystemChrome.setEnabledSystemUIMode(SystemUiMode.immersiveSticky) / Android Fullscreen Limit`. Pengguna dikunci pada perambanan absensi ini tiada dapat navigasi kabur *kembali/back* manipulatif paksa tutup ke beranda desktop antarmuka HP tablet OS. 

### Sabtu, 7 Maret 2026
**Kegiatan:** Pembuatan rupa *shape masking layer* ornamen pemandu pelurusan tata letak sudut potret proporsional sudut orientasi pandangan mata hidung kamera.
**Fokus Teknis:** Dibangun lapisan bentuk pelapis form *Oval Overlay Clipping* yang berada pas mengatapi lapis jernih frame kamera lensa Preview HP Android. Berfungsi mengisyaratkan tata ruang sudut simetris area wajah panduan potret supaya pengguna meluruskan sudut *portrait face contour tracking titik sentral proporsi parameter* demi mempertinggi hasil kelincahan skor pengenalan konklusi model pendeteksi liveness engine backend mutakhir. 

### Senin, 9 Maret 2026
**Kegiatan:** Eksekusi optimalisasi file penertiban struktur repositori *Clean Code Cleanup File Structure*.
**Fokus Teknis:** Rilis build yang berantakan direduksi size-nya. Memangkas seratusan elemen modul bereksperimen kepingan program fungsi tak terpakai riel usang mati terbengkela sisa sisa *(Dead code snippets).* Besaran file *Asset Grafis berkas statis Vector Image format SVG* diekspor konversi format dikompres ukurannya guna membikin kerangka spesifikasi *built .apk Application rilis* berberat dimensi ukuran instalasi storage memory gawai jauh lebih super mungil optimal ringan dipasang OS Android level standar murahan. 

### Selasa, 10 Maret 2026
**Kegiatan:** [*Fixing Bug Kalender Jadwal*] Membenahi logika inkonsistensi pertukaran kalender hari izin 'Day Off (DO)' mutlak Backend DB Sinkron Aplikasi Klien.
**Fokus Teknis:** Menyelesaikan selisih sinkronisasi cacat laporan nilai ketidakhadiran berkedok kalender libur operasional jadwal kerja "DO". Pengujian berhasil memulihkan alir fungsi kalendar backend server ke aplikasi genggam agar rentang kalkulasi jam ketidakhadiran DO mingguan dibaca konsisten jernih stabil dan bukan disetel kecacatan kalkulasi hitung jam salah OS error. UI riwayat membaik.

### Rabu, 11 Maret 2026
**Kegiatan:** Penambahan rute intervensi API pengayaan khusus pengerjaan kebutuhan fungsi unggah sketsa grafis coretan konversi gurat `Canvas Base64 asali digital ttd Signature Supervisor.
**Fokus Teknis:** Mendesain susunan panel fungsional arsitektur pengerat antarmuka tipe blok sentuh konversi coret sentuhan tangan di monitor gawai. Eksekusi ini merubah guratan logis peraba ujung layar jari menjadi rentetan wujud konversi pengarsipan string matriks representatif lukisan potret jenis format resolusi berwujud wujud `.PNG` format Base64 *encoding*. Modul mutlak dibenamkan difabrikasi perihal perlengkapan mutlak validasi surat wewenang tugas persetujuan persetujuan arsip dari perangkat hierarki OS superior akun berlevel mutlak Supervisor struktural jabatan logis perusahaan.

### Kamis, 12 Maret 2026
**Kegiatan:** [DESAIN IDENTITAS VISUAL MEREK LOGO APPS MAVEN RES TINGGI] Desain perombakan ulang wujud identitas paten penjenamaan arsitektur komersial aplikasi (Logo BKNOAapp MAVEN) bermutu piksel tinggi dan menutup revisi sistem UI fungsional spesifikasi konjungsi rincian hierarki admin persetujuan form file dokumen surat.
**Fokus Teknis:** Dirancang alokasi spesifik meresolusi menyusun desain grafik properti pemformat ulang aset properti komersial gambar vektor mutakhir paten hak cipta visual merk Logo App (MAVEN OS LOGO). Dipastikan ekspor format komersial grafik wujud kanvas tipe gambar menembus kualitas resolusi tak retak batas format tak buram wujud format absolut HD PNG bersih asali format Launcher App Drawer Launcher peramban gawai UI OS utuh Android HP. Logo pun dibungkus diletakan persis merangkul batas sentral properti lingkaran berwujud lingkaran bulatan kontainer berbalut solid blok warna putihan. Skema warna ini dikhususkan agar desain warna gelap Logo menyala tampak tak samar bersinergi wujud padu padan elegan dengan properti blok UI geladak estetika nuansa hitam *layout geladak minimalis Mode Kiosk Gelap berpadu harmonis beriringan cantik serasi asali kontras.* Logika pelengkap level fungsi hierarki kontrol hirarki tuntas serentak dirapikan paripurna pada ujung hari ini juga utuh aman.

### Jumat, 13 Maret 2026
**Kegiatan:** [*Fixing Bug Protektor Instalasi OS*] Penanggulangan dan pembobolan atas hambatan perlakuan pemblokan peringatan merah rilis paket build Google Play Protect spesifikasi isolasi penangkal aplikasi instalasi paket di dalam HP OS asali Android internal OS uji coba Android kawan-kawan.
**Fokus Teknis:** Proses spesimen uji *mentah (build raw .apk)* yang disebarkan kepada grup relawan dicegat instan oleh penjaga pintu internal benteng Android sandbox yakni tameng sistem OS OS *Google Play Protect alert rejection* yang menyebutnya perangkat lunak luar membahayakan entitas malnutrisi eksternal tidak aman tanpa identitas (*Untrusted Source developer unknown error blocked maleware fatal flag OS).* Solusi taktis dicapai sigap merombak pangkalan izin penanda kredibilitas pembuat aplikasi perusahaan ke level asali mutlak. Merangkai pendaftaran sandi kunci rilis penanda enkripsi perusahaan spesifk mengawinkan file tanda pendaftaran arsip asuransi rahasia pengembang file valid sertifikat identiti identiti root berpadanan fail: `key.jks / key store developer identity trust flag`. Kunci ini kemudian dilinkkan dikait di file rilis sistem fungsional instalatur ke parameter struktur file utuh mutlak android: properti baris sinkron basis ekosistem Gradle struktur: flag paten V2 Signature asali OS tipe (*v2SigningEnabled parameter bernilai boolean 'true'*). Masalah *alert OS blocked Play Protect OS utuh murni* sukses besar dijegal dan terselesaikan mutlak tak ada komplain kembali mulus sinkron terpasang utuh sempurna merdeka!. 

### Sabtu, 14 Maret 2026
**Kegiatan:** Penutupan pengerjaan tahapan persiapan pembentukan kerangka konfigurasi build properti modul paket berkas aplikasi gawai dan perumusan susunan baris Proguard pengabur ekosistem Android Native Obfuscation.
**Fokus Teknis:** Hari ditutup menyisipkan susunan modul kode pengamanan pertahanan enkripsi. Konsep rilis paket dikesahkan mengusung konfigurasi pangkalan asali pengaman paten kode rahasia rilis berparameter file sandi sketsa rahasia rilis yakni skrip pengkabur arsitektur sistem kerangka file struktur ekosistem arsitektural Java ekosistem OS asali *Proguard Obfuscation Rilis Proteksi*. Memastikan arsitek tak dapat diekstrak atau dibongkar isinya via rekayasa reverse *engineer pembaca kelas struktur OS gawai* oknum hacker luaran dekstop utuh aman stabil mutlak rilis persiapannya kompilasi siap dilepas besok senin utuh asali mantap.

### Senin, 16 Maret 2026
**Kegiatan:** Eksekusi peluncuran spesifikasi arsitektur tahapan Build Release Alpha-Test .APK *Production Bundle Compile*.
**Fokus Teknis:** Melaksanakan penarikan kompilasi peluncuran keluaran build utuh dengan format mutlak perintah OS Terminal eksekusi paten rilis: `flutter build apk --release --obfuscate --split-debug-info` murni asali Android Native SDK utuh. Proses berhasil mencetak berkas rilis perakitan akhir produksi asali paket perakitan (*Package APK Asali OS OS Production Size kecil gesit*). Bersamaan pula disebar distribusikan menyasar jajaran unit *tester asali mutlak kru operasional fungsi karyawan internal grup staf pengetesan lapangan spesimen OS testing stability Release Alpha-Test fase awal uji coba*. Berjalan tuntas optimal stabil tanpa ada fatal crash crash error sedikitpun sukses kelar tuntas asali mantap rilis Android rampung. 

### Selasa, 17 Maret 2026
**Kegiatan:** Rencana penyikapan platform migrasi kerangka adaptabilitas ekosistem kompilator murni transisi pergerakan OS ke target kompilasi lingkungan Apple iPhone native asali platform framework ekosistem dasar internal platform iOS iOS OS Mutlak murni Apple hardware.
**Fokus Teknis:** Bersantai sejenak usai android tuntas, langkah proyek lanjut bersikeras mempersiapkan penataan adaptasi spesifikasi parameter library arsitektur internal kerangka konfigurasi ekosistem OS rilis asali dependensi perpustakaan fungsional asali bahasa rilis iOS Apple native *Objective-C / parameter spesifikasi perbendaharaan Apple Swift bahasa asli iOS OS*. Penyesuaian konfigurasi file setup perizinan pubspec dikesahkan disesuaikan menanti pembongkaran sinkron esok persiapan target kompilator hardware utuh mutlak.

### Rabu, 18 Maret 2026
**Kegiatan:** Injeksi paten tag otorisasi pengamanan modul file komponen perlengkapan privasi sentral rilis Apple (Manifest Data iOS Info.plist Target Info.plist) OS beserta pelarasan instalatur komponen modul dependensi kerangka instalatur Apple framework CocoaPods kompilator iOS bawaan native rilis mutlak.
**Fokus Teknis:** Mengintegrasikan delegasi parameter penjamin tag komponen konfigurasi otoritas hak persetujuan permintaan akses komponen hardware pelacak pengguna asali OS Sandbox rilis Apple OS OS yakni menancapkan spesifikasi pembedahan asali hak akses sensor tangkapan potret Lensa Apple Kamera parameter:  `<key>NSCameraUsageDescription</key>`  dibuntuti rilis pendaftaran pelacak satelit radar mutlak asali Apple iOS parameter titik presisi sensor GPS tracker spesifikasi OS: `<key>NSLocationWhenInUseUsageDescription</key>` yang diunggah dicolok paten di modul pusat arsitektural privasi perizinan fungsional root direktori paten Apple `IOS/Runner/Info.plist parameter sandbox rilis iOS utuh aman`. Menyukseskan perampungan sinkron terminal command kompilator modul utilitas OS bawaan OS Apple spesifikasi parameter pemecah arsitektur rilis asali instruksi: `pod install` asali library file sinkron berhasil dirampungkan 100% tuntas bersyarat bersih tiada log error rilis mantap siap tes.

### Kamis, 19 Maret 2026
**Kegiatan:** [HARI KEMATIAN KEBUNTUAN BENCANA FATAL DEADLOCK] Gagalnya fasa ujung kerangka rilis target platform sistem akibat konfrontasi hambatan logis rilis arsitektur paten platform arsitektural tidak disediakannya satu pun fasilitas properti sarana operasional unit kerja yakni alat eksistensi ketiadaan komputer perangkat gawai sistem Mac OS PC iMac/Macbook. Proyek diputuskan *Crash Berhenti Mandek Total Disokong Liburan Tuntas Hangus!*.
**Fokus Teknis:** **Ini merupakan kecelakaan logis tragedi kebuntuan hardware asali yang tak dicek awal!** Karena kebijakan konfigurasi manufaktur rilis eksklusif sistem properti Apple OS OS murni mengalihkan *syarat rilis ekosistem kompilator* rilis iOS app (*Native App SDK Flutter to Apple Binary IPA Release OS*) **mewajibkan sistemnya dirender dihidupkan memakai arsitektur paket perkakas program kompilator fungsional eksklusif penamaan perangkat peranti spesifikasi mutlak "Software Komputasi Developer XCode App Program Apple Mutlak"**. XCode App ini adalah penengah dewa kompilator render **HANYA bisa berjalan secara operasional fungsional OS beroperasi secara murni eksklusif diikat direkatkan paksa hanya ke arsitektur peranti kelas properti mesin sistem komputasi ber-titel *sistem operasi asali paten macOS Apple Hardware PC PC Mac berwadah laptop model gawai Macbook Pro eksklusif atau jajaran famili properti fisik perakitan mesin meja Apple layar Desktop PC iMac spesifik mutlak absolut ekosistem paten penjamin properti fisik mutlak OS murni***.
Ketika realitanya di meja lingkungan asali infrastruktur meja kantor penampakan keseluruhan sarana inventaris ruangan perabotan di sini sama sekali tak berwawasan asali nihil total *tak memiliki satu buah sebijipun asali komputer mesin macOS mutlak tiada eksistensinya!* Hal ini memicu runtuhnya arsitektur proses *Build Native iOS eksekusi iOS flutter code sinkron OS tercekik logis*. Operasional ditangguhkan ditutup mati gembok. Seluruh proses dihentikan mendadak menyambut cuti panjang massal rutinitas pengerjaan asali di hari esoknya. Jeda kalendar total rehat dihentikan!.

*(20 Maret 2026 s/d 29 Maret 2026: Aktivitas Pekerjaan Cuti Libur Nasional Proyek Pekerjaan Eksekusi Dihentikan Serempak Tutup Produksi Total)*

---

## FASE 4: Peralihan Strategis ke PWA iOS, Optimalisasi Pemindaian Wajah, dan Finalisasi

### Senin, 30 Maret 2026
**Kegiatan:** Pembuatan *Progressive Web App* (PWA) khusus iOS (BKNOAweb-ios) menggunakan React.JS dan Vite.
**Fokus Teknis:** Menyikapi masalah tiadanya perangkat Mac untuk merilis aplikasi bawaan iPhone, proyek diubah haluannya dengan membuat alternatif berbasis web (PWA). Kerangka aplikasi ini dibangun menggunakan framework React dipadukan dengan Vite agar proses render lebih cepat. Desain antarmuka (UI) dikerjakan menggunakan Tailwind CSS dengan tujuan menyontek persis tampilan aplikasi Flutter sebelumnya. Selain itu, ditambahkan modifikasi ukuran margin (CSS `safe-area-inset-top`) agar desain bagian atas layar tidak terpotong oleh bentuk fisik *notch/Dynamic Island* pada iPhone. 

### Selasa, 31 Maret 2026
**Kegiatan:** Refactoring logika pemindaian wajah (*Face Scanning*) dari metode 5-Step menjadi 1-Step.
**Fokus Teknis:** Mengoptimalkan fitur pemindaian wajah di aplikasi yang sebelumnya dikeluhkan berlajan lambat. Pada versi lama, pengguna diwajibkan mengikuti lima tahapan verifikasi gerakan (seperti menengok, tersenyum, berkedip, dll) yang memproses banyak memori sekaligus membebani CPU sistem secara berat. Algoritma ini dibongkar dan diganti menjadi verifikasi **1-Step Verification**. Lewat pembaruan ini, aplikasi hanya perlu menangkap satu lembar foto (frame drop snapshot) secara *real-time* dan langsung mentransfernya ke server Face API (Minifast v2) di backend. Proses validasi wajah berjalan instan tanpa menghilangkan akurasi deteksi pemalsuan, sehingga berhasil mengefisiensikan waktu antrean presensi karyawan yang sebelumnya memakan waktu sangat lama.

### Rabu, 1 April 2026
**Kegiatan:** Integrasi lensa WebRTC Safari, Proteksi Desktop Bypass, dan *Deployment* Akhir Proyek Keseluruhan.
**Fokus Teknis:** Pada aplikasi PWA, akses pengaktifan perangkat keras kamera depan pengguna direalisasikan meggunakan API bawaan browser modern yang spesifik bernama *WebRTC (`navigator.mediaDevices.getUserMedia()`)* agar *video streaming* wajah masuk utuh di halaman web iPhone. 
Sebagai perlindungan keamanan *geofencing* tambahan, skrip web ditambahkan perlindungan untuk mendeteksi pembatasan browser (User-Agent). Ini bertujuan menggagalkan karyawan yang secara curang mengakali web PWA menggunakan sambungan komputer dekstop PC/Laptop yang rentan dimanipulasi peretas ekstensi lokasi (*Fake GPS Chrome*). Proteksi tambahan ini dirancang untuk membatasi nilai rasio monitor ke batasan *Aspect Ratio* spesifik ukuran ponsel posisi tegap (Portrait 16:9). Jika PWA dibuka di layar lebar desktop, sistem akan menolak sinkronisasi dengan paksa dan layar tertutup secara fungsional.
Di penghujung hari tanggal 1 April 2026, keseluruhan isi folder PWA hasil kompilasi (`/dist` pada React) segera didistribusikan (*deployment*) ke layanan server internal (Nginx). Dengan langkah ini, keseluruhan modul fungsional proyek BKNOA dikonfirmasi usai, dan sistem diserahkan ke manajemen untuk fase *User Acceptance Test* (UAT). Selesai.

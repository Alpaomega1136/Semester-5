---
title: Performance Tuning & Konsep DBMS
aliases:
  - Latihan IF3140 Performance Tuning
  - Soal Sistem Basis Data
tags:
  - IF3140
  - sistem-basis-data
  - database
  - performance-tuning
  - latihan-soal
cssclasses:
  - study-note
---

# Kumpulan Soal Sistem Basis Data
## Performance Tuning & Konsep DBMS

> [!info] Tentang catatan
> Catatan ini berisi soal, pilihan jawaban, **kunci jawaban**, pembahasan, dan acuan materi.  
> Kunci dan pembahasan dibuat sebagai **collapsible callout**, sehingga cocok dipakai untuk latihan di Obsidian.

> [!tip] Cara menggunakan
> Kerjakan soal terlebih dahulu, lalu klik bagian **Kunci Jawaban** dan **Pembahasan** untuk membukanya.

---

## Daftar Isi
- [[#Soal 1 — Higher Level Database Design|Soal 1 — Higher Level Database Design]]
- [[#Soal 2 — Pernyataan Terkait Indeks|Soal 2 — Pernyataan Terkait Indeks]]
- [[#Soal 3 — Pernyataan Terkait Perbaikan Kinerja Basis Data|Soal 3 — Pernyataan Terkait Perbaikan Kinerja Basis Data]]
- [[#Soal 4 — Fungsi Database Administrator|Soal 4 — Fungsi Database Administrator]]
- [[#Soal 5 — Pernyataan Terkait RAID|Soal 5 — Pernyataan Terkait RAID]]
- [[#Soal 6 — Jenis Arsitektur Database|Soal 6 — Jenis Arsitektur Database]]
- [[#Soal 7 — Level Perbaikan Basis Data|Soal 7 — Level Perbaikan Basis Data]]
- [[#Soal 8 — Komponen Query Processor|Soal 8 — Komponen Query Processor]]
- [[#Soal 9 — Database Performance Tuning|Soal 9 — Database Performance Tuning]]
- [[#Soal 10 — Transaksi|Soal 10 — Transaksi]]
- [[#Soal 11 — Komponen Fungsional Sistem Basis Data|Soal 11 — Komponen Fungsional Sistem Basis Data]]
- [[#Soal 12 — RAID 1 dan RAID 5|Soal 12 — RAID 1 dan RAID 5]]
- [[#Soal 13 — Hardware Tuning|Soal 13 — Hardware Tuning]]
- [[#Soal 14 — Schema Tuning|Soal 14 — Schema Tuning]]
- [[#Soal 15 — Faktor yang Memengaruhi Kinerja Basis Data|Soal 15 — Faktor yang Memengaruhi Kinerja Basis Data]]
- [[#Soal 16 — Tugas Storage Manager|Soal 16 — Tugas Storage Manager]]

---

## Soal 1 — Higher Level Database Design

> [!question] Soal
> Mana sajakah yang termasuk teknik perbaikan kinerja basis data pada *higher level database design*?

### Pilihan Jawaban
1. Menambahkan indeks di suatu kolom di sebuah tabel.
2. Mengubah setting parameter DBMS.
3. Melakukan perubahan skema dengan cara mendekomposisi tabel secara vertikal maupun horizontal.
4. Mengganti CPU dengan CPU berkinerja lebih tinggi.

> [!success]- Kunci Jawaban
> ✅ **1 dan 3**

> [!note]- Pembahasan
> - **(1) Benar.** Index termasuk bagian dari *physical schema* pada higher level database design.
> - **(2) Salah.** Mengubah setting parameter DBMS termasuk level **database system parameters**.
> - **(3) Benar.** Horizontal/vertical decomposition merupakan perubahan desain/skema basis data.
> - **(4) Salah.** Mengganti CPU termasuk **hardware tuning**.

> [!cite]- Acuan
> Slide 8 — *Tuning Levels*.
>
> ---

---

## Soal 2 — Pernyataan Terkait Indeks

> [!question] Soal
> Pernyataan yang tepat terkait meningkatkan kinerja basis data dengan indeks adalah:

### Pilihan Jawaban
1. Indeks mendukung operasi join, sorting, searching data.
2. Indeks tidak memengaruhi operasi perubahan (insert/update/delete) data.
3. Indeks sebaiknya diterapkan pada semua kolom di sebuah tabel.
4. Sebaiknya sebuah tabel hanya punya 1 indeks karena jika ada lebih dari 1 indeks pada sebuah tabel, kinerja basis data akan menurun.

> [!success]- Kunci Jawaban
> ✅ **1 saja**

> [!note]- Pembahasan
> - **(1) Benar.** Index dapat mempercepat searching, sorting, dan join sehingga tidak selalu perlu melakukan full table scan.
> - **(2) Salah.** Operasi insert/update/delete tetap dapat terdampak karena index juga perlu dipelihara ketika data berubah.
> - **(3) Salah.** Index sebaiknya dibuat berdasarkan workload dan kolom yang memang sering dipakai sebagai search key/join/sort, bukan semua kolom.
> - **(4) Salah.** Satu tabel boleh memiliki beberapa index. Yang penting adalah pemilihannya sesuai workload.

> [!cite]- Acuan
> Slide 12, 14, dan 15 — *Decisions to Make* dan *Index*.
>
> ---

---

## Soal 3 — Pernyataan Terkait Perbaikan Kinerja Basis Data

> [!question] Soal
> Mana sajakah pernyataan yang tepat terkait perbaikan kinerja basis data berikut?

### Pilihan Jawaban
1. Memperbaiki kinerja basis data dapat dilakukan dengan menulis query SQL yang memiliki kinerja lebih baik dari yang lain.
2. Optimizer hints adalah salah satu teknik yang digunakan oleh DBMS untuk melakukan optimisasi query basis data.
3. Tuning terhadap transaksi dapat dilakukan dengan cara mengubah single transaction menjadi mini-batch transactions yang masing-masing menjalankan bagian update tertentu.
4. Database performance benchmarks dapat digunakan untuk membandingkan kinerja berbagai sistem basis data dalam pemenuhan terhadap standar.

> [!success]- Kunci Jawaban
> ✅ **1, 2, 3, dan 4**

> [!note]- Pembahasan
> - **(1) Benar.** Query dapat ditulis ulang agar query plan lebih efisien.
> - **(2) Benar.** Optimizer hint adalah instruksi khusus yang ditanamkan dalam SQL untuk memengaruhi optimizer.  
>   *Catatan:* Secara lebih presisi, hint biasanya diberikan programmer/DBA kepada optimizer.
> - **(3) Benar.** Transaksi update besar dapat dipecah menjadi mini-transactions untuk mengurangi masalah lock/log yang terlalu besar.
> - **(4) Benar.** Performance benchmark digunakan untuk mengukur dan membandingkan kinerja sistem basis data.

> [!cite]- Acuan
> Slide 20, 25, dan 40.
>
> ---

---

## Soal 4 — Fungsi Database Administrator

> [!question] Soal
> Mana sajakah yang **bukan** merupakan fungsi Database Administrator?

### Pilihan Jawaban
1. Mendefinisikan skema basis data.
2. Melakukan maintenance rutin terhadap basis data, misalnya melakukan back-up rutin.
3. Memberikan hak akses terhadap data kepada pengguna.
4. Mengembangkan aplikasi yang mengakses basis data.

> [!success]- Kunci Jawaban
> ✅ **4**

> [!note]- Pembahasan
> - **(1) Fungsi DBA.** DBA dapat terlibat dalam definisi dan pengelolaan skema.
> - **(2) Fungsi DBA.** Backup dan maintenance merupakan tugas penting DBA.
> - **(3) Fungsi DBA.** DBA mengelola authorization/privilege.
> - **(4) Bukan fungsi utama DBA.** Pengembangan aplikasi biasanya merupakan tanggung jawab application developer/programmer.
>
> **Acuan:** Tidak dibahas langsung dalam PDF *Performance Tuning*; mengikuti konsep DBMS standar.
>
> ---

> [!cite]- Acuan
> Tidak dibahas langsung dalam PDF *Performance Tuning*; mengikuti konsep DBMS standar.
>
> ---

---

## Soal 5 — Pernyataan Terkait RAID

> [!question] Soal
> Pernyataan yang tepat terkait RAID adalah:

### Pilihan Jawaban
1. RAID terdiri atas sejumlah disk yang masing-masing akan mengandung data yang sama sehingga dapat menyediakan view dari sebuah single-disk.
2. Bit-level striping atau block-level striping digunakan untuk membagi data pada berbagai disk dalam RAID sehingga dapat meningkatkan kecepatan transfer.
3. Model RAID yang saat ini tersedia di pasaran antara lain adalah RAID 0, 1, 2, 3, 4, 5, 10.
4. RAID memungkinkan kehandalan sistem yang tinggi karena dimungkinkan menyimpan data secara redundan di lebih dari 1 disk, sehingga jika 1 disk fail, disk lain masih dapat menggantikan perannya.

> [!success]- Kunci Jawaban
> ✅ **2 dan 4**

> [!note]- Pembahasan
> - **(1) Salah.** Tidak semua RAID menyimpan data yang sama pada setiap disk. Mirroring adalah karakteristik RAID 1.
> - **(2) Benar.** Striping membagi data ke beberapa disk untuk meningkatkan transfer rate.
> - **(3) Salah.** Materi menyatakan RAID 2, 3, dan 4 tidak lagi digunakan dalam praktik.
> - **(4) Benar secara konsep.** RAID yang memiliki redundansi dapat meningkatkan reliability/fault tolerance.  
>   *Catatan:* RAID 0 tidak memiliki fault tolerance.

> [!cite]- Acuan
> Slide 32–36.
>
> ---

---

## Soal 6 — Jenis Arsitektur Database

> [!question] Soal
> Mana sajakah pasangan jenis arsitektur database dan definisinya yang tepat?

### Pilihan Jawaban
1. Centralized database: Terdapat satu pusat penyimpanan data, yang berjalan di atas sebuah sistem komputer tunggal yang tidak berinteraksi dengan sistem komputer lain.
2. Client-server: terdapat satu server penyimpan data yang melayani banyak client.
3. Parallel databases: Data tersebar di beberapa mesin (sering disebut site atau node).
4. Distributed databases: Sistem terdiri atas banyak prosesor dan banyak disk yang terhubung melalui jaringan interkoneksi yang cepat.

> [!success]- Kunci Jawaban
> ✅ **1 dan 2**

> [!note]- Pembahasan
> - **(1) Benar.** Centralized database berpusat pada satu sistem utama.
> - **(2) Benar.** Client-server menggunakan server basis data yang melayani banyak client.
> - **(3) Salah.** Definisi tersebut lebih cocok untuk distributed database.
> - **(4) Salah.** Definisi tersebut lebih cocok untuk parallel database dengan interkoneksi berkecepatan tinggi.
>
> **Acuan:** Tidak dibahas langsung dalam PDF *Performance Tuning*; mengikuti konsep arsitektur DBMS standar.
>
> ---

> [!cite]- Acuan
> Tidak dibahas langsung dalam PDF *Performance Tuning*; mengikuti konsep arsitektur DBMS standar.
>
> ---

---

## Soal 7 — Level Perbaikan Basis Data

> [!question] Soal
> Basis data dapat diperbaiki pada berbagai level, yaitu:

### Pilihan Jawaban
1. Higher level database design.
2. Database system parameter.
3. Database user.
4. Hardware.

> [!success]- Kunci Jawaban
> ✅ **1, 2, dan 4**

> [!note]- Pembahasan
> Materi membagi tuning menjadi tiga level:
> 1. Higher level database design,
> 2. Database system parameters,
> 3. Hardware.
>
> Database user bukan level tuning.

> [!cite]- Acuan
> Slide 8 — *Tuning Levels*.
>
> ---

---

## Soal 8 — Komponen Query Processor

> [!question] Soal
> Yang merupakan komponen yang terdapat dalam query processor:

### Pilihan Jawaban
1. DDL interpreter.
2. DML compiler.
3. Query evaluation engine.
4. Authorization and integrity manager.

> [!success]- Kunci Jawaban
> ✅ **1, 2, dan 3**

> [!note]- Pembahasan
> - **DDL interpreter** menangani perintah definisi data.
> - **DML compiler** menerjemahkan query/perintah manipulasi data.
> - **Query evaluation engine** menjalankan query plan.
> - **Authorization and integrity manager** umumnya ditempatkan sebagai bagian dari storage manager, bukan query processor.
>
> **Acuan:** Tidak dibahas langsung dalam PDF *Performance Tuning*; mengikuti arsitektur DBMS standar.
>
> ---

> [!cite]- Acuan
> Tidak dibahas langsung dalam PDF *Performance Tuning*; mengikuti arsitektur DBMS standar.
>
> ---

---

## Soal 9 — Database Performance Tuning

> [!question] Soal
> Mana sajakah yang tepat terkait database performance tuning?

### Pilihan Jawaban
1. Peningkatan kinerja basis data dilakukan menyesuaikan berbagai parameter dan pilihan desain basis data.
2. Peningkatan kinerja basis data ditujukan untuk menurunkan throughput dan meningkatkan contention.
3. Prosedur umum dalam database performance tuning adalah dengan mengidentifikasi bottleneck pada sistem dan menghapusnya.
4. Peningkatan kinerja basis data ditargetkan untuk meminimalkan I/O disk karena biasanya I/O mendominasi potensi terjadinya bottleneck pada sistem.

> [!success]- Kunci Jawaban
> ✅ **1, 3, dan 4**

> [!note]- Pembahasan
> - **(1) Benar.** Tuning dilakukan dengan penyesuaian resource, parameter, dan design choices.
> - **(2) Salah.** Tujuannya justru **meningkatkan throughput** dan **meminimalkan contention**.
> - **(3) Benar.** Prosedur umum tuning adalah menemukan bottleneck lalu mengatasinya.
> - **(4) Benar.** Disk I/O merupakan salah satu bottleneck utama sehingga sering menjadi target optimisasi.

> [!cite]- Acuan
> Slide 5, 7, dan 9.
>
> ---

---

## Soal 10 — Transaksi

> [!question] Soal
> Yang benar terkait transaksi:

### Pilihan Jawaban
1. Transaksi berisi satu buah operasi yang menjalankan fungsi logic dari aplikasi berbasis data.
2. Concurrency control manager bertugas untuk mengendalikan interaksi antar transaksi konkuren.
3. Transaction manager memastikan basis data selalu dalam keadaan konsisten.
4. Transaksi adalah kumpulan operasi yang digunakan untuk menjalankan suatu fungsi logic dari aplikasi berbasis data.

> [!success]- Kunci Jawaban
> ✅ **2, 3, dan 4**

> [!note]- Pembahasan
> - **(1) Salah.** Transaksi tidak harus hanya terdiri dari satu operasi.
> - **(2) Benar.** Concurrency control manager mengatur interaksi transaksi yang berjalan bersamaan.
> - **(3) Benar.** Transaction manager membantu menjaga konsistensi dan keandalan eksekusi transaksi.
> - **(4) Benar.** Transaksi merupakan satu unit kerja logis yang dapat terdiri dari beberapa operasi.
>
> **Acuan:** Definisi dasar transaksi dan transaction manager tidak dibahas langsung pada PDF *Performance Tuning*; mengikuti konsep DBMS standar.
>
> ---

> [!cite]- Acuan
> Definisi dasar transaksi dan transaction manager tidak dibahas langsung pada PDF *Performance Tuning*; mengikuti konsep DBMS standar.
>
> ---

---

## Soal 11 — Komponen Fungsional Sistem Basis Data

> [!question] Soal
> Yang **bukan** merupakan komponen fungsional sistem basis data adalah:

### Pilihan Jawaban
1. Storage manager.
2. Query processor.
3. Transaction manager.
4. Disk processor.

> [!success]- Kunci Jawaban
> ✅ **4 — Disk processor**

> [!note]- Pembahasan
> Storage manager, query processor, dan transaction manager merupakan komponen fungsional DBMS. **Disk processor** bukan nama komponen fungsional standar pada arsitektur DBMS.
>
> **Acuan:** Tidak dibahas langsung dalam PDF *Performance Tuning*; mengikuti arsitektur DBMS standar.
>
> ---

> [!cite]- Acuan
> Tidak dibahas langsung dalam PDF *Performance Tuning*; mengikuti arsitektur DBMS standar.
>
> ---

---

## Soal 12 — RAID 1 dan RAID 5

> [!question] Soal
> Pernyataan yang tepat terkait RAID 1 dan RAID 5 adalah:

### Pilihan Jawaban
1. RAID 5 lebih disarankan digunakan jika jarang ada operasi write dan jumlah data sangat besar.
2. RAID 5 lebih disarankan jika banyak operasi write dengan jumlah data tidak terlalu besar.
3. Jika suatu aplikasi membutuhkan operasi read sebanyak `r` per detik dan operasi write sebanyak `w` per detik, maka RAID 1 membutuhkan operasi I/O sebanyak `r + 2w` per detik.
4. Jika suatu aplikasi membutuhkan operasi read sebanyak `r` per detik dan operasi write sebanyak `w` per detik, maka RAID 1 membutuhkan operasi I/O sebanyak `r + 4w` per detik.

> [!success]- Kunci Jawaban
> ✅ **1 dan 3**

> [!note]- Pembahasan
> - **(1) Benar.** Rule of thumb: RAID 5 cocok ketika write jarang dan data sangat besar.
> - **(2) Salah.** RAID 5 memiliki write overhead lebih tinggi.
> - **(3) Benar.** RAID 1 membutuhkan `r + 2w` operasi I/O per detik.
> - **(4) Salah.** `r + 4w` adalah kebutuhan I/O RAID 5.

> [!cite]- Acuan
> Slide 37 — *Choice of RAID Level*.
>
> ---

---

## Soal 13 — Hardware Tuning

> [!question] Soal
> Pernyataan yang **tidak tepat** terkait hardware tuning adalah:

### Pilihan Jawaban
1. Banyaknya operasi I/O per transaksi dapat dikurangi dengan menambah data yang di-buffer di memori.
2. Menurut five-minute rule, data yang diakses setiap 400 detik atau kurang layak untuk di-buffer di memory.
3. Pada one-minute rule, data yang diakses secara sekuensial sekali atau lebih dalam 1 menit seharusnya di-buffer di memory.
4. Five-minute rule tidak berubah, misalnya menjadi 1 hour rule atau 1 second rule, untuk random access, bahkan sampai pada tahun 2017.

> [!success]- Kunci Jawaban
> ✅ **4** sebagai pernyataan yang **tidak tepat**.

> [!note]- Pembahasan
> - **(1) Benar.** Menyimpan lebih banyak data di memori dapat mengurangi disk I/O.
> - **(2) Benar.** Pada contoh tahun 1987, break-even sekitar 400 detik ≈ 5 menit.
> - **(3) Benar.** One-minute rule berlaku pada sequential access.
> - **(4) Salah.** Rule untuk random access berubah seiring perkembangan harga/performa hardware; materi memberi contoh sekitar 1,5 jam pada 2007 dan 4 jam pada 2017.

> [!cite]- Acuan
> Slide 27, 29, 30, dan 31.
>
> ---

---

## Soal 14 — Schema Tuning

> [!question] Soal
> Perbaikan kinerja basis data dengan cara memperbaiki skema dapat dilakukan dengan teknik yang mana saja?

### Pilihan Jawaban
1. Horizontal/vertical splitting sebuah tabel.
2. Denormalisasi sebuah tabel dari skema BCNF menjadi 2NF.
3. Melakukan duplikasi sebuah tabel untuk kolom-kolom tertentu.
4. Menambahkan indeks pada kolom tertentu di sebuah tabel.

> [!success]- Kunci Jawaban
> ✅ **1, 2, dan 3**

> [!note]- Pembahasan
> - **(1) Benar.** Schema tuning dapat dilakukan dengan horizontal/vertical splitting.
> - **(2) Benar dalam konteks denormalisasi.** Materi menyebut kita dapat memilih *lower normal form* demi workload/performa.
> - **(3) Benar.** Duplicating tables termasuk teknik schema tuning/denormalization yang tercantum pada materi.
> - **(4) Salah untuk konteks pertanyaan ini.** Index memang teknik performance tuning, tetapi berada pada **index tuning**, bukan perubahan schema itu sendiri.
>
> > **Catatan penting untuk opsi 2:** Slide secara eksplisit memberi contoh memilih **3NF daripada BCNF**, bukan BCNF → 2NF. Namun slide lain juga menyebut secara umum *settle for a lower normal form (denormalization)*. Karena itu pada pembahasan sebelumnya opsi 2 dinilai benar.

> [!cite]- Acuan
> Slide 12, 17, dan 18.
>
> ---

---

## Soal 15 — Faktor yang Memengaruhi Kinerja Basis Data

> [!question] Soal
> Berikut dapat memengaruhi kinerja basis data, **kecuali**:

### Pilihan Jawaban
1. Kinerja CPU.
2. Penentuan autorisasi pengguna.
3. Query ke basis data yang digunakan pada aplikasi.
4. Skema basis data.

> [!success]- Kunci Jawaban
> ✅ **2 — Penentuan autorisasi pengguna**

> [!note]- Pembahasan
> Materi *What affects performance?* menyebut:
> - Hardware (CPU, network, RAM, disk),
> - Database server parameter settings,
> - Database design/schema,
> - Indices,
> - SQL statement.
>
> Penentuan autorisasi pengguna tidak tercantum sebagai faktor utama performance tuning pada slide tersebut.

> [!cite]- Acuan
> Slide 6 — *What affects performance?*
>
> ---

---

## Soal 16 — Tugas Storage Manager

> [!question] Soal
> Yang merupakan tugas storage manager adalah:

### Pilihan Jawaban
1. Berinteraksi dengan OS file manager.
2. Menyimpan, mendapatkan, dan memperbaharui data secara efisien.
3. Mengeksekusi perintah low-level yang di-generate oleh DML compiler.
4. Memastikan basis data selalu dalam keadaan konsisten.

> [!success]- Kunci Jawaban
> ✅ **1 dan 2**

> [!note]- Pembahasan
> - **(1) Benar.** Storage manager menjadi penghubung antara DBMS dan penyimpanan/file system.
> - **(2) Benar.** Storage manager bertanggung jawab atas penyimpanan, retrieval, dan update data secara efisien.
> - **(3) Salah.** Eksekusi instruksi low-level hasil DML compiler lebih tepat dilakukan oleh **query evaluation engine**.
> - **(4) Salah.** Penjagaan konsistensi transaksi lebih terkait dengan **transaction manager**.
>
> **Acuan:** Tidak dibahas langsung dalam PDF *Performance Tuning*; mengikuti arsitektur DBMS standar.
>
> ---

> [!cite]- Acuan
> Tidak dibahas langsung dalam PDF *Performance Tuning*; mengikuti arsitektur DBMS standar.
>
> ---

---


> [!abstract] Ringkasan Cepat
> Gunakan bagian berikut untuk review sebelum kuis atau ujian.

## Ringkasan Kunci Jawaban

| No. | Topik | Kunci |
|---|---|---|
| 1 | Higher level database design | **1, 3** |
| 2 | Index | **1** |
| 3 | Performance improvement | **1, 2, 3, 4** |
| 4 | Fungsi DBA — yang bukan | **4** |
| 5 | RAID | **2, 4** |
| 6 | Arsitektur database | **1, 2** |
| 7 | Tuning levels | **1, 2, 4** |
| 8 | Query processor | **1, 2, 3** |
| 9 | Performance tuning | **1, 3, 4** |
| 10 | Transaction | **2, 3, 4** |
| 11 | Komponen fungsional — yang bukan | **4** |
| 12 | RAID 1 dan RAID 5 | **1, 3** |
| 13 | Hardware tuning — yang tidak tepat | **4** |
| 14 | Schema tuning | **1, 2, 3** |
| 15 | Faktor kinerja — kecuali | **2** |
| 16 | Storage manager | **1, 2** |

---

## Ringkasan Konsep yang Perlu Diingat

- **Higher level database design:** index, materialized view, splitting/decomposition, query, transaction, logical schema.
- **Database system parameters:** buffer size, checkpoint interval, dan parameter DBMS lainnya.
- **Hardware tuning:** CPU, RAM, disk, RAID, dan optimisasi I/O.
- **Index:** membantu searching, sorting, dan join, tetapi memiliki biaya pemeliharaan ketika data berubah.
- **Performance tuning:** meningkatkan throughput, mengurangi contention, dan mencari/mengatasi bottleneck.
- **Schema tuning:** splitting, denormalization, redundant/derived attributes, collapsing/duplicating tables.
- **RAID 1:** mirroring, fault tolerant, kebutuhan I/O `r + 2w`.
- **RAID 5:** parity, cocok bila write jarang dan data besar, kebutuhan I/O `r + 4w`.
- **Five-minute rule:** contoh historis untuk random access; nilainya dapat berubah mengikuti perkembangan hardware.
- **One-minute rule:** terkait sequential access.
- **Query processor:** DDL interpreter, DML compiler, query evaluation engine.
- **Storage manager:** berhubungan dengan penyimpanan/file system dan pengelolaan data secara efisien.

---

> [!abstract] Catatan sumber
> Bagian yang memang terdapat di PDF mengikuti materi **IF3140 – Sistem Basis Data: Performance Tuning**.  
> Topik yang tidak dibahas langsung dalam PDF ditandai pada bagian **Acuan** masing-masing soal.

#IF3140 #SistemBasisData #PerformanceTuning #LatihanSoal

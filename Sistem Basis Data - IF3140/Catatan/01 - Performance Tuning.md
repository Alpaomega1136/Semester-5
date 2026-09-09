---
tags:
  - IF3140
  - Sistem-Basis-Data
  - Performance-Tuning
aliases:
  - Database Performance Tuning
---

# Performance Tuning

## 1. Apa itu Performance Tuning?

**Database Performance Tuning** adalah proses menyesuaikan desain maupun konfigurasi sistem basis data agar sistem dapat bekerja dengan lebih efisien.

Tujuan utamanya adalah:
- meningkatkan **throughput**,
- mengurangi penggunaan resource yang tidak diperlukan,
- mengurangi **contention**,
- dan memungkinkan sistem menangani workload yang lebih besar.

Secara umum proses tuning dilakukan dengan:

> **Identify Bottleneck → Investigate → Remove/Reduce Bottleneck → Evaluate Again**

Contohnya, ketika sebuah aplikasi web membutuhkan waktu lama untuk mengambil data, masalahnya belum tentu berada pada aplikasi. Setelah diperiksa, ternyata query melakukan **full relation scan**. Solusinya dapat berupa penambahan index.

Namun jika query sudah menggunakan index dan masih lambat, kemungkinan penyebabnya dapat berada pada query, database configuration, memory, disk, atau komponen lainnya.

---

## 2. Apa yang Mempengaruhi Performance Database?

Performa database dapat dipengaruhi oleh beberapa bagian sistem:

### Hardware

Beberapa komponen hardware yang berpengaruh:
- CPU
- RAM
- Disk
- Network

### Database Server Parameters

DBMS memiliki berbagai konfigurasi yang dapat memengaruhi performa, seperti ukuran buffer atau interval checkpoint.

### Database Design

Struktur schema yang digunakan juga memengaruhi bagaimana data diakses.

### Index

Index dapat mempercepat proses pencarian data sehingga DBMS tidak harus membaca seluruh tabel.

### SQL Statement

Dua query yang menghasilkan data sama belum tentu memiliki performa yang sama.

> [!important]
> Performance tuning tidak hanya berarti **membuat query lebih cepat**. Masalah dapat muncul pada desain database, konfigurasi DBMS, transaksi, maupun hardware.

---

# 3. Bottleneck

**Bottleneck** adalah bagian dari sistem yang menjadi pembatas utama performa keseluruhan sistem.

Misalnya:

```text
Disk sangat sibuk
CPU relatif idle
↓
Disk kemungkinan merupakan bottleneck
```

Sering kali hanya sebagian kecil dari sistem yang menggunakan sebagian besar waktu pemrosesan.

Konsep sederhananya menyerupai:

```text
20% bagian sistem
        ↓
menggunakan sekitar 80% waktu/resource
```

Karena itu, tuning sebaiknya difokuskan pada bagian yang paling berpengaruh.

Hal penting lainnya:

> Menghilangkan satu bottleneck dapat menyebabkan bottleneck lain menjadi terlihat.

Artinya performance tuning biasanya merupakan proses **iteratif**, bukan dilakukan hanya sekali.

---

# 4. Tiga Level Performance Tuning

Database dapat dioptimasi pada tiga level utama.

## 4.1 Database Design

Meliputi perubahan pada:
- physical schema,
- index,
- materialized view,
- horizontal splitting,
- query,
- transaction,
- logical schema.

## 4.2 Database System Parameters

Contohnya:
- memperbesar buffer agar disk I/O berkurang,
- mengatur interval checkpoint agar ukuran log dapat dikendalikan.

## 4.3 Hardware

Contohnya:
- menambah disk untuk meningkatkan kemampuan I/O,
- menambah RAM agar lebih banyak data berada dalam memory,
- menggunakan processor yang lebih cepat.

---

# 5. Mengapa Disk I/O Sangat Penting?

Salah satu target utama database tuning adalah:

> **Meminimalkan Disk I/O**

Hal ini terjadi karena akses disk jauh lebih lambat dibandingkan akses memory.

Pada contoh materi:

| Media | Random Block Read |
|---|---:|
| HDD | sekitar 10 ms |
| SSD | sekitar 0.1 ms |
| Memory | sekitar 100 ns |

Karena perbedaannya sangat besar, semakin sering DBMS harus membaca data dari disk, semakin besar kemungkinan I/O menjadi bottleneck.

Konsep sederhananya:

```text
Data di Memory
     ↓
akses sangat cepat

Data harus dibaca dari Disk
     ↓
lebih lambat
     ↓
potensi bottleneck
```

---

# 6. Understanding the Workload

Sebelum melakukan tuning, kita harus mengetahui terlebih dahulu **workload database**.

Workload menjelaskan operasi apa saja yang paling sering dilakukan terhadap database.

Hal yang perlu diketahui:

### Untuk Query

Perhatikan:
- relation/table apa yang digunakan,
- attribute apa yang diambil,
- attribute yang digunakan pada selection,
- attribute yang digunakan pada join,
- seberapa selective kondisi tersebut.

### Untuk Update

Perhatikan:
- kondisi selection/join,
- jenis operasi:
  - `INSERT`
  - `DELETE`
  - `UPDATE`
- attribute yang berubah.

Selain itu perlu diketahui:

```text
Query mana yang penting?
        ↓
Seberapa sering dijalankan?
        ↓
Berapa performance yang diharapkan?
```

> [!important]
> Tidak ada satu desain database yang selalu paling cepat untuk semua workload.

Desain yang baik harus mempertimbangkan pola penggunaan database sebenarnya.

---

# 7. Indexing dan Index Tuning

## 7.1 Apa itu Index?

**Index** adalah struktur data yang digunakan untuk mempercepat akses terhadap data.

Analogi sederhananya adalah **katalog buku di perpustakaan**.

Tanpa katalog:

```text
Cari buku
↓
periksa setiap buku satu per satu
```

Dengan katalog:

```text
Cari melalui katalog
↓
dapatkan lokasi buku
↓
ambil buku
```

Database bekerja dengan konsep yang mirip.

---

## 7.2 Search Key

**Search Key** adalah attribute atau kumpulan attribute yang digunakan untuk mencari record.

Index berisi **index entries** yang secara sederhana berbentuk:

```text
Search Key → Pointer
```

Pointer menunjukkan lokasi record yang sesuai.

Contohnya:

```text
student_id
    ↓
  Index
    ↓
Pointer ke record mahasiswa
```

Karena index biasanya jauh lebih kecil dibandingkan tabel aslinya, mencari melalui index dapat jauh lebih efisien dibandingkan melakukan **full table scan**.

Index juga membantu operasi seperti:
- searching,
- sorting,
- join.

---

## 7.3 Keputusan Dalam Membuat Index

Ketika melakukan index tuning, beberapa pertanyaan perlu dipertimbangkan:

- Table mana yang membutuhkan index?
- Attribute mana yang menjadi search key?
- Apakah dibutuhkan lebih dari satu index?
- Jenis index apa yang digunakan?

Beberapa karakteristik index yang dapat dipilih:

```text
Clustered / Non-clustered
Hash / Tree
Dynamic / Static
Dense / Sparse
```

Pemilihannya harus disesuaikan dengan workload database.

---

# 8. Schema Tuning

**Schema Tuning** adalah perubahan desain schema agar lebih sesuai dengan workload.

Normalisasi tetap penting untuk mengurangi redundancy, tetapi desain yang sangat ternormalisasi tidak selalu memberikan performa terbaik untuk setiap workload.

Beberapa pilihan yang dapat dilakukan antara lain:

```text
Horizontal Decomposition
Vertical Decomposition
3NF daripada BCNF
Further Decomposition
Denormalization
```

Workload dapat memengaruhi apakah sebuah relation tetap dipertahankan, dipecah lagi, atau justru digabungkan kembali.

---

## 8.1 Splitting Tables

Tabel dapat dibagi menjadi:

- **Horizontal splitting**
- **Vertical splitting**

Dalam kasus tertentu, membagi tabel dapat meningkatkan performa.

Namun konsekuensinya adalah aplikasi dapat menjadi lebih kompleks karena data tidak lagi berada dalam satu tabel.

---

## 8.2 Denormalization

**Denormalization** berarti mengembalikan sebagian hasil decomposition atau menambahkan redundancy demi alasan performa.

Bentuknya dapat berupa:

- menambahkan redundant column,
- menambahkan derived attribute,
- collapsing tables,
- duplicating tables.

Tujuannya adalah mengurangi pekerjaan yang diperlukan ketika data sering dibaca.

Namun trade-off-nya adalah:

```text
Lebih sedikit operasi saat membaca
              ↕
Lebih banyak redundancy / maintenance
```

---

## 8.3 Schema Evolution

Jika perubahan terhadap schema dilakukan ketika database sudah digunakan, proses tersebut disebut:

**Schema Evolution**

Views dapat digunakan untuk membantu menyembunyikan sebagian perubahan schema dari aplikasi.

---

# 9. Query Tuning

Query tuning bertujuan membuat DBMS mengeksekusi query dengan cara yang lebih efisien.

## 9.1 Query Plan

Sebelum menjalankan query, optimizer akan menentukan **query execution plan**.

Namun:

> Optimizer tidak selalu memilih plan terbaik.

Untuk melihat plan yang digunakan, banyak DBMS menyediakan:

```sql
EXPLAIN
```

Query optimizer membuat keputusan berdasarkan **statistics database**.

Masalahnya, statistics tersebut dapat menjadi tidak akurat atau sudah lama.

Untuk memperbaruinya digunakan:

```sql
ANALYZE
```

Secara sederhana:

```text
Query
  ↓
Optimizer
  ↓
Statistics
  ↓
Execution Plan
  ↓
Execution
```

---

## 9.2 Nested Subquery

Query kompleks yang memiliki banyak nested subquery dalam beberapa kasus sulit dioptimasi.

Salah satu pendekatan adalah melakukan **query rewriting**, misalnya mengubah nested subquery menjadi operasi `JOIN`.

Namun materi juga mencatat bahwa teknik rewriting menjadi semakin kurang penting karena optimizer modern semakin baik.

---

## 9.3 Optimizer Hint

**Optimizer Hint** adalah instruksi tambahan yang diberikan kepada optimizer untuk memengaruhi execution plan.

Contohnya dapat digunakan untuk:
- memilih index tertentu,
- memprioritaskan waktu mendapatkan row pertama,
- atau mengoptimalkan keseluruhan hasil query.

---

# 10. Set Orientation

Dalam aplikasi sering terjadi pola seperti:

```text
Loop setiap department
    ↓
jalankan query
    ↓
Loop lagi
    ↓
jalankan query lagi
```

Ini menyebabkan banyak komunikasi antara aplikasi dan database.

**Set Orientation** mencoba mengubah operasi tersebut menjadi lebih sedikit database calls.

Contoh pendekatan berulang:

```sql
SELECT SUM(salary)
FROM instructor
WHERE dept_name = ?;
```

Daripada menjalankannya satu per satu untuk setiap department, gunakan operasi terhadap satu set:

```sql
SELECT dept_name, SUM(salary)
FROM instructor
GROUP BY dept_name;
```

Hasilnya:

```text
Banyak query kecil
        ↓
Satu query terhadap sekumpulan data
        ↓
Lebih sedikit database calls
```

---

# 11. Transaction Tuning

Transaction tuning berkaitan dengan bagaimana transaksi dijalankan secara bersamaan.

## 11.1 Long Read-Only Transaction

Transaksi read-only yang berjalan lama dan membaca sebagian besar relation dapat menyebabkan:

**Lock Contention**

Misalnya:

```text
Query statistik bank besar
           +
Transaksi nasabah biasa
           ↓
       contention
```

Beberapa pendekatan untuk menguranginya:

### Multi-Version Concurrency Control

DBMS mempertahankan beberapa versi data sehingga transaksi pembacaan dapat menggunakan versi tertentu tanpa terlalu mengganggu transaksi update.

### Degree-Two Consistency

Dapat digunakan untuk long transaction agar kebutuhan locking berkurang.

Namun terdapat trade-off:

> Hasil yang diperoleh dapat bersifat approximate.

---

# 12. Long Update Transaction

Transaksi update yang terlalu besar dapat menyebabkan:

- lock space habis,
- log space habis,
- recovery setelah crash menjadi lebih lama.

Solusinya adalah menggunakan:

## Mini-Batch Transaction

Satu transaksi besar dibagi menjadi beberapa transaksi kecil.

```text
Large Transaction
      ↓
┌─────────┐
│ Batch 1 │
├─────────┤
│ Batch 2 │
├─────────┤
│ Batch 3 │
└─────────┘
```

Setiap mini-transaction menangani sebagian update.

Jika terjadi failure pada suatu mini-batch, proses recovery harus memastikan bagian tersebut diselesaikan dengan benar agar **atomicity** tetap terjaga.

---

# 13. Hardware Tuning

Walaupun query dan transaksi sudah dioptimasi, database tetap membutuhkan operasi I/O.

Pada contoh materi:

> Disk biasa dapat menangani sekitar **100 random I/O operations/second**.

Jika satu transaksi membutuhkan:

```text
2 random I/O
```

maka satu disk secara kasar dapat menangani:

$$
\frac{100}{2}=50
$$

transaksi per detik.

Untuk mendukung $n$ transaksi per detik, secara sederhana dibutuhkan:

$$
\frac{n}{50}
$$

disk dengan data di-stripe antar disk.

---

# 14. Menyimpan Data di Memory

Salah satu cara mengurangi disk I/O adalah mempertahankan data yang sering digunakan di memory.

```text
Lebih banyak data di memory
          ↓
Lebih banyak buffer hit
          ↓
Lebih sedikit disk access
          ↓
Performance meningkat
```

Tetapi memory juga memiliki biaya.

Karena itu muncul pertanyaan:

> Data mana yang layak disimpan di memory?

---

# 15. Break-Even Memory dan Disk

Pertimbangan dilakukan dengan membandingkan:

**Disk Access Cost**

$$
DiskCost =
\frac{PricePerDisk}
{AccessPerSecond \times m}
$$

dengan:

**Memory Cost**

$$
MemoryCost =
\frac{PricePerMBMemory}
{BlocksPerMB}
$$

di mana $m$ adalah interval rata-rata suatu block diakses.

Jika biaya mempertahankan block di memory lebih rendah dibandingkan biaya akses disk, block tersebut lebih baik disimpan di memory.

---

# 16. Five-Minute Rule

Contoh historis pada tahun 1987 menghasilkan break-even:

$$
m = 400\ seconds
$$

atau sekitar:

$$
m \approx 5\ minutes
$$

Sehingga muncul **Five-Minute Rule**:

> Data yang cukup sering diakses sehingga interval aksesnya berada di bawah break-even tersebut layak dipertahankan dalam memory.

Perlu diperhatikan bahwa aturan ini bergantung pada harga dan kemampuan hardware.

Materi menunjukkan bahwa nilai break-even berubah dari waktu ke waktu.

---

# 17. One-Minute Rule

Untuk data yang dibaca secara **sequential**, lebih banyak block dapat dibaca dalam satu detik dibandingkan random access.

Dengan asumsi pembacaan sequential sebesar 1 MB:

> **Sequentially accessed data yang diakses setidaknya sekali dalam satu menit sebaiknya disimpan di memory.**

Konsep ini dikenal sebagai **One-Minute Rule**.

---

# 18. RAID

**RAID — Redundant Array of Independent Disks** adalah teknik mengorganisasikan beberapa disk agar terlihat sebagai satu kesatuan.

Tujuannya dapat berupa:

- meningkatkan capacity,
- meningkatkan speed,
- meningkatkan reliability.

Dua konsep penting RAID:

### Striping

Data dibagi ke beberapa disk agar transfer dapat dilakukan secara paralel.

### Redundancy

Informasi tambahan disimpan agar data dapat direkonstruksi ketika sebuah disk gagal.

---

# 19. Perbandingan RAID

| RAID | Konsep | Keuntungan | Kekurangan |
|---|---|---|---|
| RAID 0 | Striping | Read cepat | Tidak ada fault tolerance |
| RAID 1 | Mirroring | Fault tolerant, mudah rebuild | Hanya ±50% kapasitas untuk data |
| RAID 5 | Striping + parity | Tahan 1 disk failure | Write dan rebuild lebih mahal |
| RAID 10 | RAID 0 + RAID 1 | Cepat + fault tolerant | Minimal 4 disk, ±50% kapasitas efektif |

### RAID 0

```text
Disk 1 : A C E G
Disk 2 : B D F H
```

Data tersebar di beberapa disk.

Keuntungan: **fast read**

Kekurangan:

> Jika satu disk gagal, data dapat hilang.

### RAID 1

```text
Disk 1 : A B C D
Disk 2 : A B C D
```

Disk kedua merupakan mirror.

Keuntungan:
- fault tolerance,
- recovery relatif mudah.

Tetapi membutuhkan kapasitas penyimpanan lebih besar.

### RAID 5

RAID 5 menggunakan **parity**.

Dengan 4 disk, sekitar:

$$
75\%
$$

kapasitas digunakan untuk actual data.

Keuntungannya dapat bertahan terhadap **1 disk failure**.

Kekurangannya adalah operasi write dan rebuild lebih mahal.

### RAID 10

Menggabungkan:

```text
RAID 0 → Striping
          +
RAID 1 → Mirroring
```

Sehingga memperoleh:

**Speed + Fault Tolerance**

Namun membutuhkan setidaknya 4 disk.

---

# 20. RAID 1 vs RAID 5

Pemilihan RAID juga bergantung pada jumlah read dan write.

Misalkan:

- $r$ = read per second
- $w$ = write per second

Untuk RAID 1:

$$
I/O = r + 2w
$$

Untuk RAID 5:

$$
I/O = r + 4w
$$

RAID 5 memiliki write cost lebih besar karena parity harus dibaca dan diperbarui.

> [!tip]
> Rule of Thumb pada materi:
>
> **RAID 5 cocok ketika write jarang dan jumlah data sangat besar.**
>
> Untuk kondisi lainnya, RAID 1 umumnya lebih disukai.

---

# 21. Performance Simulation

Sebelum perubahan diterapkan pada sistem sebenarnya, performa dapat diperkirakan menggunakan **performance simulation**.

Materi menggunakan pendekatan **queuing model**.

Model dapat digunakan untuk memperkirakan:

- bottleneck,
- throughput,
- response time,
- efek penambahan disk,
- efek penambahan memory,
- perubahan algoritma.

Alurnya:

```text
Model Sistem
    ↓
Ubah Parameter
    ↓
Jalankan Simulation
    ↓
Evaluasi Performance
    ↓
Terapkan ke Real System
```

Simulation tidak harus meniru seluruh detail sistem. Beberapa nilai dapat disederhanakan, misalnya menggunakan rata-rata disk read time.

---

# 22. Performance Benchmark

**Performance Benchmark** adalah sekumpulan task yang digunakan untuk mengukur performa suatu sistem.

Benchmark penting agar dua database system dapat dibandingkan dengan workload yang relatif sama.

Tiga ukuran utama:

### Throughput

Jumlah transaksi yang dapat diselesaikan per satuan waktu.

Biasanya:

$$
transactions/second
$$

atau **TPS**.

### Response Time

Waktu sejak transaksi diberikan hingga hasil diterima.

### Availability

Mengukur ketersediaan sistem, misalnya melalui **mean time to failure**.

---

# 23. Menghitung Average Throughput

Average throughput tidak selalu dapat dihitung dengan arithmetic mean.

Misalkan:

```text
Transaction A = 99 TPS
Transaction B = 1 TPS
```

Tidak berarti:

$$
\frac{99+1}{2}=50\ TPS
$$

Waktu untuk menjalankan masing-masing adalah:

$$
\frac{1}{99} + \frac{1}{1}
\approx 1.01\ seconds
$$

Sehingga throughput gabungannya sekitar:

$$
1.98\ TPS
$$

Untuk beberapa transaction type digunakan **Harmonic Mean**:

$$
H =
\frac{n}
{\frac{1}{t_1}+\frac{1}{t_2}+\cdots+\frac{1}{t_n}}
$$

Namun ketika transaksi berjalan secara concurrent, interaksi seperti **lock contention** dapat membuat perhitungan teoritis tersebut tidak lagi akurat.

---

# 24. OLTP vs OLAP

## OLTP — Online Transaction Processing

OLTP menangani banyak transaksi kecil dan update secara bersamaan.

Fokus utamanya:

```text
High Concurrency
Fast Transaction
Fast Commit
High Update Rate
```

## OLAP — Online Analytical Processing

OLAP digunakan untuk analisis atau decision support.

Fokusnya:

```text
Complex Query
Large Data
Aggregation
Query Optimization
```

Ringkasannya:

| OLTP | OLAP |
|---|---|
| Banyak transaksi | Query analitik |
| Banyak update | Banyak pembacaan |
| Transaksi relatif pendek | Query dapat kompleks |
| Fokus concurrency | Fokus query optimization |

Beberapa DBMS lebih dioptimalkan untuk salah satu workload, sedangkan lainnya mencoba menyeimbangkan keduanya.

---

# 25. TPC Benchmark

**Transaction Processing Council (TPC)** menyediakan benchmark yang digunakan untuk membandingkan performa database.

## TPC-A dan TPC-B

Benchmark OLTP sederhana yang memodelkan aplikasi teller bank.

Pada materi disebutkan benchmark ini sudah tidak digunakan lagi.

## TPC-C

Benchmark OLTP yang lebih kompleks dan memodelkan **inventory system**.

TPC-C menjadi standar benchmark OLTP.

## TPC-D

Benchmark untuk complex decision support dengan 17 query.

Kemudian dikembangkan menjadi TPC-H dan TPC-R.

## TPC-H

Huruf **H** berarti *ad hoc*.

Karakteristik:
- 22 query,
- banyak operasi aggregation,
- query tidak diketahui sebelumnya,
- materialized view tidak diperbolehkan,
- index hanya diperbolehkan pada primary dan foreign key.

## TPC-R

Mirip TPC-H, tetapi tidak memiliki pembatasan terhadap:
- materialized view,
- index.

## TPC-W

Benchmark end-to-end Web Service yang memodelkan sebuah **Web Bookstore**.

Mencakup:
- static pages,
- dynamically generated pages.

---

# 26. TPC Performance Measures

TPC menggunakan beberapa ukuran performa.

### Transactions Per Second

Jumlah transaksi yang dapat diproses sambil tetap memenuhi batas response time.

### Transactions Per Second Per Dollar

Mengukur performa dengan mempertimbangkan biaya sistem.

Secara konsep:

```text
Performance
    ÷
System Cost
```

Benchmark TPC juga mengharuskan ukuran database meningkat ketika throughput meningkat.

Tujuannya agar benchmark lebih menyerupai kondisi dunia nyata:

```text
Lebih banyak pengguna
        ↓
Lebih banyak transaksi
        ↓
Database juga lebih besar
```

Hasil benchmark TPC juga membutuhkan **external audit** sebelum dapat diklaim secara resmi.

---

# 27. Power Test dan Throughput Test

Pada TPC-H dan TPC-R terdapat dua pengujian utama.

## Power Test

Query dan update dijalankan secara **sequential**.

Tujuannya mengukur kemampuan sistem menjalankan query individual.

```text
Query 1
  ↓
Query 2
  ↓
Query 3
  ↓
...
```

## Throughput Test

Beberapa query dan update dijalankan secara **concurrent**.

```text
Stream 1 ─┐
Stream 2 ─┤
Stream 3 ─┼→ Database
Update   ─┘
```

Setiap stream menghasilkan 22 query sementara terdapat parallel update stream.

Hasil Power Test dan Throughput Test kemudian dapat digabungkan menjadi **Composite Query per Hour Metric**.

---

# 28. Gambaran Besar Performance Tuning

Performance tuning dapat dipahami sebagai alur berikut:

```text
              Database Performance
                       │
              Understand Workload
                       │
                       ▼
              Identify Bottleneck
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
   Query/Index      Transaction     Hardware
     Tuning           Tuning         Tuning
        │              │              │
        └──────────────┼──────────────┘
                       ▼
                Measure Result
                       │
               Benchmark/Simulation
                       │
                       ▼
                 Tune Again
```

> [!important] Inti Materi
> Performance tuning bukan sekadar mencari **query tercepat**.
>
> Kita harus memahami **workload**, menemukan **bottleneck**, lalu menentukan apakah masalah berada pada **index, schema, query, transaction, database configuration, memory, disk, atau hardware**.
>
> Setelah perubahan dilakukan, performa harus diukur kembali menggunakan throughput, response time, atau benchmark yang sesuai.

---

# 29. Ringkasan Cepat

| Konsep | Inti |
|---|---|
| Performance Tuning | Optimasi penggunaan resource |
| Bottleneck | Bagian yang membatasi performa |
| Workload | Pola query dan update pada database |
| Index | Mempercepat pencarian data |
| Schema Tuning | Menyesuaikan schema terhadap workload |
| Query Tuning | Memperbaiki execution plan/query |
| Set Orientation | Mengurangi jumlah database calls |
| Transaction Tuning | Mengurangi contention dan transaksi besar |
| Hardware Tuning | Mengurangi I/O dan memanfaatkan memory |
| RAID | Speed/reliability menggunakan banyak disk |
| Simulation | Memperkirakan performa sebelum diterapkan |
| Throughput | Transaksi per satuan waktu |
| Response Time | Lama waktu memperoleh hasil |
| OLTP | Banyak transaksi/update |
| OLAP | Query analitik dan aggregation |
| TPC | Standard benchmark database |

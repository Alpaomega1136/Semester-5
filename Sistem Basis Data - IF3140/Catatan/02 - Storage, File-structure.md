---
tags:
  - IF3140
  - Sistem-Basis-Data
  - Storage
  - File-Structure
aliases:
  - Storage and File Structure
---

# Storage and File Structure

## 1. Gambaran Umum

Materi **Storage and File Structure** membahas bagaimana data basis data benar-benar disimpan pada media fisik dan bagaimana DBMS mengatur file serta record agar proses membaca dan menulis data dapat dilakukan secara efisien.

Topik utamanya meliputi:

- klasifikasi media penyimpanan,
- struktur magnetic disk,
- ukuran performa disk,
- disk block access,
- file organization,
- penyimpanan fixed-length dan variable-length record,
- operasi terhadap record,
- organisasi record di dalam file,
- serta data dictionary.

> [!important]
> Walaupun kita sering melihat database sebagai kumpulan tabel, pada level fisik data sebenarnya disimpan sebagai **record di dalam file**, sedangkan file disimpan dalam kumpulan **block** pada storage.

---

# 2. Classification of Physical Storage Media

Media penyimpanan dapat dibagi menjadi dua kelompok utama.

## 2.1 Volatile Storage

**Volatile storage** kehilangan isinya ketika daya listrik dimatikan.

Contoh utamanya adalah:

- cache,
- main memory.

Storage jenis ini biasanya cepat, tetapi tidak digunakan sebagai tempat penyimpanan permanen.

## 2.2 Non-Volatile Storage

**Non-volatile storage** tetap mempertahankan data walaupun daya listrik dimatikan.

Contohnya:

- flash memory,
- magnetic disk,
- optical disk,
- magnetic tape,
- battery-backed main memory.

Dalam memilih media penyimpanan, terdapat tiga faktor utama:

1. **Access speed** — seberapa cepat data dapat dibaca atau ditulis.
2. **Cost per unit of data** — biaya untuk menyimpan sejumlah data.
3. **Reliability** — seberapa dapat dipercaya media tersebut dalam mempertahankan data.

---

# 3. Storage Hierarchy

Storage disusun dalam sebuah **hierarchy**.

```text
        Cache
          ↓
     Main Memory
          ↓
     Flash Memory
          ↓
    Magnetic Disk
          ↓
     Optical Disk
          ↓
    Magnetic Tape
```

Secara umum, semakin ke atas:

```text
lebih cepat
lebih mahal per unit data
kapasitas relatif lebih kecil
```

Sedangkan semakin ke bawah:

```text
lebih lambat
lebih murah per unit data
kapasitas lebih besar
```

## 3.1 Primary Storage

**Primary storage** adalah media tercepat, tetapi umumnya bersifat volatile.

Contoh:
- cache,
- main memory.

## 3.2 Secondary Storage

**Secondary storage** bersifat non-volatile dan memiliki kecepatan sedang.

Disebut juga **online storage**.

Contohnya:
- flash memory,
- magnetic disk.

## 3.3 Tertiary Storage

**Tertiary storage** berada pada tingkat paling bawah.

Karakteristiknya:
- non-volatile,
- access time lambat,
- digunakan terutama untuk archival storage.

Contohnya:
- magnetic tape,
- optical storage.

Magnetic tape menggunakan **sequential access**, sehingga tidak cocok untuk random access yang sering dilakukan database aktif.

---

# 4. Magnetic Hard Disk

Magnetic hard disk terdiri atas beberapa **platter** yang berputar.

Permukaan platter dibagi menjadi lingkaran-lingkaran yang disebut **track**.

Setiap track kemudian dibagi menjadi **sector**.

```text
Platter
 └── Track
      └── Sector
```

**Sector** adalah unit data terkecil pada magnetic disk yang dapat dibaca atau ditulis.

## 4.1 Read-Write Head

Setiap permukaan disk memiliki **read-write head** yang digunakan untuk membaca atau menulis data.

Head dipindahkan menggunakan disk arm menuju track yang dibutuhkan.

Karena disk berputar, sector yang ingin dibaca juga harus terlebih dahulu berada tepat di bawah read-write head.

## 4.2 Cylinder

Sebuah **cylinder** adalah kumpulan track dengan posisi yang sama pada seluruh platter.

```text
Platter 1 → Track i
Platter 2 → Track i
Platter 3 → Track i
                ↓
            Cylinder i
```

---

# 5. Disk Controller

**Disk controller** adalah penghubung antara computer system dan disk drive hardware.

Controller menerima command seperti:

```text
Read sector
Write sector
```

Kemudian controller mengubahnya menjadi operasi fisik.

Tugas disk controller antara lain:
- menggerakkan disk arm ke track yang benar,
- membaca dan menulis data,
- menghitung dan menyimpan **checksum**,
- memeriksa hasil pembacaan,
- melakukan **remapping bad sector**.

Beberapa disk dapat terhubung ke satu computer system melalui sebuah disk controller.

## 5.1 Checksum

Checksum digunakan untuk memeriksa apakah data yang dibaca masih sesuai dengan data yang disimpan.

```text
Data ditulis
    ↓
Checksum disimpan
    ↓
Data dibaca kembali
    ↓
Checksum dihitung ulang
    ↓
Bandingkan
```

Jika nilai berbeda, terdapat kemungkinan data mengalami corruption.

## 5.2 Disk Interface

Beberapa keluarga interface disk yang disebutkan dalam materi:
- **ATA**
- **SATA**
- **SCSI**
- **SAS**

---

# 6. Performance Measures of Disks

Untuk memahami performa disk, terdapat beberapa ukuran penting.

## 6.1 Access Time

**Access time** adalah waktu sejak request read/write diberikan hingga transfer data mulai dilakukan.

Access time terutama terdiri dari:

```text
Seek Time
    +
Rotational Latency
```

## 6.2 Seek Time

**Seek time** adalah waktu yang dibutuhkan untuk memindahkan disk arm ke track yang tepat.

Pada disk tipikal dalam materi:

```text
sekitar 4–10 ms
```

## 6.3 Rotational Latency

Setelah head berada di track yang benar, platter masih harus berputar sampai sector yang dibutuhkan berada tepat di bawah head.

Waktu menunggu tersebut disebut **rotational latency**.

Pada disk dalam materi:

```text
sekitar 4–11 ms
```

Average rotational latency kira-kira setengah dari satu putaran.

## 6.4 Data Transfer Rate

**Data-transfer rate** adalah kecepatan data dapat dibaca dari atau ditulis ke disk.

Materi memberikan kisaran maksimum sekitar:

```text
25–200 MB/s
```

## 6.5 Mean Time to Failure — MTTF

**MTTF** adalah rata-rata waktu sebuah disk diharapkan dapat bekerja terus-menerus tanpa failure.

Materi menyebutkan umur praktis disk biasanya sekitar:

```text
3–5 tahun
```

> [!note]
> MTTF yang sangat besar seperti ratusan ribu jam merupakan ukuran statistik untuk populasi disk, bukan berarti satu disk benar-benar diperkirakan akan hidup selama puluhan atau ratusan tahun.

---

# 7. Disk Block Access

Database file dibagi menjadi unit penyimpanan dengan ukuran tetap yang disebut **block**.

Block merupakan unit untuk:
- storage allocation,
- transfer data antara disk dan main memory.

Secara fisik, block terdiri dari kumpulan sector yang berurutan.

```text
Disk
 └── Block
      ├── Sector
      ├── Sector
      ├── Sector
      └── ...
```

Ukuran block tipikal yang disebutkan dalam materi adalah sekitar:

```text
4 KB – 16 KB
```

## 7.1 Trade-Off Block Size

### Block terlalu kecil

```text
lebih banyak block
        ↓
lebih banyak transfer disk
```

### Block terlalu besar

```text
lebih sedikit transfer
        ↓
potensi ruang terbuang lebih besar
```

Karena akses disk jauh lebih lambat dibandingkan main memory, DBMS berusaha meminimalkan jumlah **block transfer**.

---

# 8. Record Access Mechanism

Data pada storage biasanya dipindahkan ke main memory dalam bentuk **block**.

Namun program biasanya membutuhkan satu **record** tertentu.

```text
Storage
   │
   │  satu block
   ▼
I/O Buffer
   │
   │  satu record
   ▼
User Buffer
```

Transfer storage → memory relatif lambat.

Sebaliknya, manipulasi record yang sudah berada di main memory jauh lebih cepat.

---

# 9. Optimization of Disk Block Access

Pendekatan yang dibahas dalam materi:

- buffering of blocks,
- disk-arm scheduling algorithms,
- file organization,
- nonvolatile write buffers,
- log disk.

Tujuannya adalah mengurangi biaya akses fisik ke disk.

---

# 10. Buffer dan Buffer Manager

## 10.1 Buffer

**Buffer** adalah sebagian main memory yang digunakan untuk menyimpan salinan block dari disk.

Dengan buffer, block yang sering digunakan tidak selalu perlu dibaca ulang dari disk.

## 10.2 Buffer Manager

**Buffer manager** adalah subsystem DBMS yang mengelola buffer.

Jika block sudah berada di buffer:

```text
Request Block
      ↓
Block ditemukan di memory
      ↓
Return address block
```

Jika block belum berada di buffer, buffer manager akan:

1. menyediakan ruang di buffer,
2. jika buffer penuh, memilih block yang akan digantikan,
3. jika block yang digantikan telah berubah, block tersebut ditulis kembali ke disk,
4. membaca block baru dari disk,
5. memberikan address block kepada requester.

> [!important]
> Buffering efektif karena disk jauh lebih lambat dibandingkan main memory.

---

# 11. File Organization dan Fragmentation

**File organization** mengatur posisi block berdasarkan pola bagaimana data akan diakses.

Related information dapat ditempatkan pada cylinder yang sama atau berdekatan agar pergerakan disk arm berkurang.

Namun file dapat mengalami **fragmentation** akibat:
- insert,
- delete,
- free block yang tersebar.

Pada file yang fragmented, sequential access membutuhkan lebih banyak perpindahan disk arm.

Beberapa sistem menyediakan utility untuk melakukan **defragmentation**.

---

# 12. File, Record, dan Field

Struktur fisik database dapat dilihat sebagai:

```text
Database
   ↓
Files
   ↓
Records
   ↓
Fields
```

Sebuah database merupakan kumpulan file, satu file terdiri atas sequence of records, dan satu record terdiri atas sequence of fields.

---

# 13. Blocking

Proses memasukkan record ke dalam block disebut **blocking**.

Blocking dapat berupa:
- **spanned**
- **unspanned**

Pada **unspanned blocking**, sebuah record harus sepenuhnya berada di satu block.

## 13.1 Blocking Factor

**Blocking Factor (`Bfr`)** menyatakan jumlah record yang dapat dimasukkan ke satu block.

Untuk fixed-length record dan unspanned blocking:

$$
Bfr = \left\lfloor \frac{B}{R} \right\rfloor
$$

dengan:
- $B$ = block size
- $R$ = record size

Contoh:

```text
Block size  = 4096 byte
Record size = 500 byte
```

maka:

$$
Bfr = \left\lfloor \frac{4096}{500} \right\rfloor = 8
$$

Artinya satu block dapat menampung maksimal **8 record**.

---

# 14. Fixed-Length Records

Pada **fixed-length records**, setiap record memiliki ukuran yang sama.

Jika ukuran setiap record adalah $n$, maka record ke-$i$ dapat ditempatkan mulai dari:

$$
n \times i
$$

dengan $i$ dimulai dari 0.

Keuntungan pendekatan ini adalah posisi record relatif mudah dihitung.

Namun record sebaiknya tidak dibiarkan melintasi batas block.

---

# 15. Deletion pada Fixed-Length Records

Jika sebuah record dihapus, terdapat beberapa pendekatan.

## Alternatif 1 — Shift Records

Semua record setelah record yang dihapus digeser.

```text
R0 R1 R2 [R3] R4 R5
             ↓ delete

R0 R1 R2 R4 R5
```

## Alternatif 2 — Move Last Record

Record terakhir dipindahkan ke posisi record yang dihapus.

```text
R0 R1 R2 [R3] ... R11

delete R3
    ↓

R11 dipindah ke posisi R3
```

## Alternatif 3 — Free List

Record tidak dipindahkan. Posisi kosong disimpan dalam sebuah **free list**.

Di awal file terdapat **file header** yang menyimpan address dari free record pertama.

```text
File Header
     ↓
Free Slot 1
     ↓
Free Slot 2
     ↓
Free Slot 3
```

---

# 16. Variable-Length Records

Variable-length record muncul ketika:
- satu file menyimpan beberapa tipe record,
- terdapat field seperti `VARCHAR`,
- terdapat repeating fields.

Untuk attribute dengan panjang variabel, record dapat menyimpan pasangan:

```text
(offset, length)
```

Sedangkan data sebenarnya ditempatkan setelah bagian fixed-length.

## 16.1 Null-Value Bitmap

Nilai `NULL` dapat direpresentasikan menggunakan **null-value bitmap**.

Setiap bit menunjukkan apakah attribute tertentu bernilai NULL.

---

# 17. Slotted Page Structure

Untuk variable-length records digunakan struktur **Slotted Page**.

```text
┌───────────────────────────────┐
│ Page Header / Slot Directory  │
├───────────────────────────────┤
│                               │
│          Free Space           │
│                               │
├───────────────────────────────┤
│ Record N                      │
│ Record ...                    │
│ Record 2                      │
│ Record 1                      │
└───────────────────────────────┘
```

Header menyimpan:
- jumlah record entries,
- akhir free space,
- lokasi setiap record,
- ukuran setiap record.

Record dapat dipindahkan di dalam page agar free space tetap contiguous.

Karena record dapat berpindah:

> Pointer sebaiknya tidak langsung menunjuk ke record, melainkan ke **slot entry** pada header.

```text
Pointer
   ↓
Slot Entry
   ↓
Record
```

---

# 18. File Performance Parameters

| Parameter | Arti |
|---|---|
| `R` | Storage yang dibutuhkan satu record |
| `TF` | Waktu mengambil arbitrary record |
| `TN` | Waktu mendapatkan next record |
| `TI` | Waktu insert record |
| `TU` | Waktu update record |
| `TX` | Waktu membaca seluruh file |
| `TY` | Waktu reorganisasi file |

Organisasi file yang berbeda dapat memiliki karakteristik performa berbeda.

---

# 19. Fetch a Record

Agar data dapat digunakan, record harus dibaca ke memory.

**Fetch** adalah pengambilan data berdasarkan suatu **key value**.

Fetching terdiri dari dua tahap:

```text
1. Locate record
2. Read record
```

Secara umum record tidak selalu dapat langsung ditemukan hanya menggunakan nomor record, sehingga diperlukan struktur atau organisasi file yang membantu menentukan lokasi data.

---

# 20. Get the Next Record

**Get-Next** digunakan untuk mendapatkan successor record berdasarkan suatu hubungan atau urutan.

Operasi ini akan cepat jika record yang berkaitan disimpan berdekatan.

```text
Related records berdekatan
          ↓
locality kuat
          ↓
Get-Next lebih cepat
```

Jika record secara fisik tidak mengikuti urutan yang diperlukan, operasi Get-Next menjadi lebih mahal.

---

# 21. Insert a Record

Insert sebuah record umumnya melibatkan:

1. mencari block tujuan,
2. membaca block,
3. memasukkan record,
4. menulis kembali block yang telah berubah.

Jika record harus dimasukkan pada lokasi tertentu, record lain mungkin perlu dipindahkan.

## 21.1 Append

Insert pada akhir file disebut **append**.

Append biasanya lebih sederhana karena jika address block terakhir sudah diketahui, DBMS dapat langsung menuju block tersebut.

---

# 22. Update a Record

Untuk memperbarui record:

1. data lama dibaca,
2. perubahan digabungkan dengan data lama,
3. record baru dibuat,
4. record baru ditempatkan pada posisi record lama,
5. block ditulis kembali.

Jika ukuran record bertambah dan tidak lagi muat pada lokasi lama:

```text
Delete old record
        +
Insert new record
```

dapat dilakukan.

## 22.1 Logical Delete / Tombstone

Deletion tidak selalu langsung menghapus record dari storage.

Record dapat hanya ditandai sebagai:

```text
invalid / deleted
```

Penanda seperti ini sering disebut **tombstone**.

---

# 23. File Reorganization

Reorganization bertujuan:

- membuang deleted/invalid record,
- memperoleh kembali free space,
- memulihkan clustering data.

Konsepnya mirip dengan **garbage collection**.

Frekuensi reorganization bergantung pada file organization dan pola penggunaan aplikasi.

---

# 24. Organization of Records in Files

Materi mengenalkan beberapa organisasi record:

1. **Heap**
2. **Sequential**
3. **Multitable Clustering**
4. **B+-Tree**
5. **Hashing**

---

# 25. Heap File Organization

Pada **Heap File**, record tidak memiliki urutan tertentu.

Record dapat ditempatkan di mana saja selama terdapat free space.

```text
Block 1 → R3, R8
Block 2 → R1, R7
Block 3 → R4, R2
```

Setelah ditempatkan, record biasanya tidak dipindahkan.

Karena itu DBMS harus memiliki cara efisien untuk menemukan free space.

## 25.1 Kelebihan Heap File

Heap file cocok untuk:
- **bulk loading**,
- relation yang relatif kecil,
- kondisi di mana indexing overhead ingin dihindari,
- query yang membaca sebagian besar record.

## 25.2 Kekurangan Heap File

Heap file kurang cocok untuk:
- selective retrieval berdasarkan key pada file besar,
- operasi yang membutuhkan sorting,
- volatile tables.

---

# 26. Sequential File Organization

Pada **Sequential File Organization**, record disimpan secara berurutan berdasarkan **search key**.

Search key tidak harus:
- primary key,
- ataupun superkey.

```text
Search key ascending

10101
12121
15151
22222
32343
...
```

Organisasi ini cocok untuk aplikasi yang sering melakukan **sequential processing terhadap seluruh file**.

---

# 27. Insert dan Delete pada Sequential File

## Deletion

Deletion dapat menggunakan **pointer chains**.

## Insertion

DBMS mencari posisi record yang seharusnya.

Jika tersedia free space:

```text
insert langsung
```

Jika tidak tersedia:

```text
record dimasukkan ke overflow block
```

Pointer chain kemudian diperbarui agar urutan logis tetap benar.

Karena banyak insert/delete dapat mengganggu urutan fisik file, sequential file perlu direorganisasi dari waktu ke waktu.

---

# 28. Multitable Clustering File Organization

Dalam **multitable clustering**, record dari beberapa relation dapat disimpan pada file yang sama.

Motivasinya adalah menyimpan record yang sering digunakan bersama pada block yang sama.

Contoh:

```text
department
instructor
```

Jika query sering melakukan:

```sql
department JOIN instructor
```

maka menyimpan department bersama para instructor terkait dapat mengurangi I/O.

## 28.1 Trade-Off Multitable Clustering

Bagus untuk query seperti:

```text
satu department + semua instructornya
```

atau:

```text
department JOIN instructor
```

Namun kurang bagus jika query hanya membutuhkan seluruh data `department`.

---

# 29. B+-Tree dan Hashing

## B+-Tree File Organization

B+-Tree mempertahankan penyimpanan dalam keadaan **ordered** walaupun terjadi:
- insert,
- delete.

## Hashing

Pada hashing, DBMS menghitung:

```text
hash(search_key)
        ↓
block tujuan
```

Hasil hash menentukan block tempat record disimpan.

---

# 30. Data Dictionary

**Data Dictionary**, atau **System Catalog**, menyimpan:

> **metadata — data tentang data**

Informasi yang disimpan dapat meliputi:

### Relation Metadata
- nama relation,
- nama attribute,
- tipe attribute,
- panjang attribute.

### View Metadata
- nama view,
- definisi view.

### Constraints
- integrity constraints.

### User Information
- user,
- accounting information,
- password-related information.

### Statistical Information
- jumlah tuple dalam relation.

### Physical Organization
- relation disimpan menggunakan sequential/hash/etc.,
- lokasi fisik relation,
- informasi index.

---

# 31. Relational Representation of Metadata

Metadata dapat disimpan dalam bentuk relation pada disk.

Contohnya secara konseptual:

```text
Relation_Metadata
Attribute_Metadata
Index_Metadata
View_Metadata
User_Metadata
```

Untuk mempercepat akses, DBMS juga dapat membangun specialized data structures di main memory.

---

# 32. Disk-Arm Scheduling

Jika terdapat banyak pending disk access, urutannya dapat diatur agar disk arm tidak bergerak secara tidak efisien.

Pendekatan ini disebut **Disk-arm-scheduling algorithm**.

Tujuan utamanya:

```text
minimize disk arm movement
```

Salah satu contoh pada materi adalah **Elevator Algorithm**.

Ide sederhananya mirip elevator:

```text
bergerak dalam satu arah
melayani request yang dilewati
kemudian berbalik arah
```

---

# 33. Nonvolatile Write Buffer

Write ke disk dapat dipercepat menggunakan **Nonvolatile Write Buffer**.

Buffer ini dapat menggunakan:
- battery-backed RAM,
- flash memory.

```text
DBMS
 ↓
Nonvolatile Buffer
 ↓
Disk
```

Data dapat dianggap aman setelah masuk ke nonvolatile buffer, kemudian disk melakukan penulisan fisik ketika memungkinkan.

Keuntungannya:
- aplikasi tidak selalu harus menunggu mechanical disk write,
- write dapat diurutkan kembali,
- disk arm movement dapat dikurangi.

---

# 34. Log Disk

**Log Disk** merupakan disk khusus untuk menulis sequence dari block updates.

Karena penulisannya bersifat sequential:

```text
Update 1 → Update 2 → Update 3 → ...
```

maka penulisan dapat dilakukan dengan cepat karena hampir tidak memerlukan seek.

Materi menyebutkan log disk dapat digunakan dengan tujuan yang mirip nonvolatile RAM tanpa memerlukan hardware khusus NV-RAM.

---

# 35. Journaling dan Write Reordering

File system sering mengubah urutan write untuk meningkatkan performa.

Namun write reordering memiliki risiko jika sistem gagal sebelum semua write selesai.

Karena itu **journaling file systems** menulis data dalam urutan yang aman menggunakan:
- NV-RAM,
- atau log disk.

> [!warning]
> Reordering write tanpa mekanisme journaling dapat meningkatkan risiko corruption pada struktur file system jika terjadi failure.

---

# 36. Gambaran Besar Storage dan File Structure

```text
Database
   │
   ▼
File
   │
   ▼
Record
   │
   ▼
Field

File disimpan pada
   │
   ▼
Disk Blocks
   │
   ▼
Sectors
   │
   ▼
Physical Storage
```

Ketika record digunakan:

```text
Storage
   ↓
Block dibaca
   ↓
I/O Buffer
   ↓
Record
   ↓
User / DBMS
```

Karena disk lebih lambat dibandingkan memory, performa database sangat dipengaruhi oleh:
- berapa block yang perlu dibaca,
- posisi block,
- bagaimana record disusun,
- bagaimana buffer digunakan,
- serta jenis file organization.

---

# 37. Perbandingan File Organization

| File Organization | Karakteristik | Cocok untuk | Kekurangan |
|---|---|---|---|
| Heap | Record tidak terurut | Bulk load, scan banyak record | Selective search lambat |
| Sequential | Record terurut berdasarkan search key | Sequential processing | Insert/delete lebih kompleks |
| Multitable Clustering | Beberapa relation dalam satu file | Join antarrelation yang sering | Query satu relation tertentu bisa kurang efisien |
| B+-Tree | Ordered dan mendukung insert/delete | Ordered access | Dibahas lebih lanjut pada materi indexing |
| Hashing | Block ditentukan hash search key | Search berdasarkan key | Dibahas lebih lanjut pada materi indexing |

---

# 38. Ringkasan Inti Materi

> [!important] Inti
> Database tidak hanya bergantung pada SQL dan logical schema. Cara data disimpan secara fisik mempunyai pengaruh langsung terhadap performa.

Beberapa konsep utama yang perlu diingat:

1. **Storage hierarchy** memperlihatkan perbedaan speed, cost, dan persistence.
2. Magnetic disk memiliki **track, sector, head, dan cylinder**.
3. Disk access dipengaruhi oleh **seek time, rotational latency, dan transfer rate**.
4. DBMS melakukan transfer data dalam bentuk **block**.
5. **Buffer manager** mengurangi jumlah disk access dengan mempertahankan block di memory.
6. Record dapat berupa **fixed-length** atau **variable-length**.
7. Variable-length record dapat dikelola menggunakan **slotted page**.
8. Organisasi file memengaruhi biaya fetch, get-next, insert, update, dan scan.
9. **Heap** sederhana tetapi tidak bagus untuk selective lookup.
10. **Sequential file** bagus untuk ordered/sequential processing tetapi insert/delete lebih sulit.
11. **Multitable clustering** dapat mengurangi I/O untuk relation yang sering diakses bersama.
12. **Data dictionary** menyimpan metadata mengenai struktur dan organisasi database.
13. Disk access dapat dioptimalkan melalui buffering, scheduling, file organization, nonvolatile write buffer, dan log disk.

---

# 39. Hubungan Konsep yang Perlu Dipahami

```text
Physical Storage
       ↓
Disk Performance
       ↓
Block Access
       ↓
Record Layout
       ↓
File Organization
       ↓
Database Performance
```

> **Semakin sedikit block yang harus dibaca dan semakin baik locality data, semakin kecil biaya I/O yang diperlukan.**

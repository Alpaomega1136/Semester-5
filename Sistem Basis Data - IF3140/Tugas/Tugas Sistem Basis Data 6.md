![[Pasted image 20260917192812.png]]
![[Pasted image 20260917192826.png]]

# Evaluation Plan dengan Index

## Diketahui

Terdapat tiga relasi pada sistem persewaan buku.

### Pelanggan

`Pelanggan(pid, nama, umur, alamat)`

- Jumlah tuple = 10.000
- Jumlah blok = 1.000
- Primary key = `pid`

### Buku

`Buku(bid, judul, penulis)`

- Jumlah tuple = 50.000
- Jumlah blok = 5.000
- Primary key = `bid`

### Peminjaman

`Peminjaman(pid, bid, tanggal)`

- Jumlah tuple = 300.000
- Jumlah blok = 15.000
- `pid` mengacu pada `Pelanggan(pid)`
- `bid` mengacu pada `Buku(bid)`

---

## Informasi Tambahan

- Terdapat 500 penulis buku yang berbeda.
- Umur pelanggan berada pada rentang 7 sampai 24 tahun.
- Distribusi atribut `penulis` dan `umur` diasumsikan uniform.
- Ukuran tuple hasil join Buku dan Peminjaman adalah 2 kali ukuran tuple Peminjaman.
- Terdapat **primary index** pada atribut `penulis` di relasi Buku.
  - Kedalaman index = 2.
- Terdapat **secondary index** pada atribut `bid` di relasi Peminjaman.
  - Kedalaman index = 3.

---

## Query

Query yang digunakan sama seperti soal sebelumnya:

$$
\Pi_{nama}
\left(
  \sigma_{umur \geq 21}(Pelanggan)
  \bowtie
  \left(
    \sigma_{penulis = \text{Andrea Hirata}}(Buku)
    \bowtie
    Peminjaman
  \right)
\right)
$$

Untuk mempermudah pembahasan, didefinisikan:

$$
A = \sigma_{penulis = \text{Andrea Hirata}}(Buku)
$$

$$
B = A \bowtie Peminjaman
$$

$$
C = \sigma_{umur \geq 21}(Pelanggan)
$$

$$
D = C \bowtie B
$$

Hasil akhirnya adalah:

$$
\Pi_{nama}(D)
$$

---

## Nomor 1

### Evaluation Plan dengan Memanfaatkan Kedua Index

Evaluation plan yang digunakan adalah:

1. Menggunakan primary index `penulis` untuk mencari buku karya Andrea Hirata.
2. Untuk setiap buku yang ditemukan, menggunakan secondary index `bid` pada Peminjaman untuk mencari semua peminjaman buku tersebut.
3. Mematerialisasi hasil join Buku dan Peminjaman.
4. Melakukan linear scan pada Pelanggan untuk memilih pelanggan dengan umur minimal 21 tahun.
5. Melakukan nested-loop join antara hasil seleksi Pelanggan dan hasil join Buku-Peminjaman.
6. Melakukan projection atribut `nama` menggunakan pipeline.

---

## 1. Seleksi Buku Menggunakan Primary Index

Operasi:

$$
A = \sigma_{penulis = \text{Andrea Hirata}}(Buku)
$$

Relasi Buku mempunyai 50.000 tuple yang disimpan dalam 5.000 blok.

Jumlah tuple per blok:

$$
f_{Buku} = \frac{50000}{5000} = 10
$$

Jadi satu blok Buku menampung 10 tuple.

Karena terdapat 500 penulis dan distribusinya uniform, jumlah buku karya Andrea Hirata adalah:

$$
n_A = \frac{50000}{500} = 100
$$

Jadi terdapat **100 tuple Buku** yang memenuhi kondisi.

Karena satu blok menampung 10 tuple, jumlah blok yang memuat buku Andrea Hirata adalah:

$$
b_A = \frac{100}{10} = 10
$$

---

### Cost Primary Index

Kedalaman primary index pada atribut `penulis` adalah 2.

Untuk menemukan lokasi awal data diperlukan:

$$
2
$$

block access untuk traversal index.

Karena primary index bersifat terurut atau clustered terhadap key tersebut, 100 tuple yang dicari diperkirakan berada pada 10 blok data.

Maka:

$$
Cost(A) = 2 + 10 = 12
$$

Jadi cost seleksi Buku menggunakan primary index adalah **12 block transfer**.

Hasilnya:

- Cost = **12**
- Banyak tuple = **100**

---

## 2. Join A dengan Peminjaman Menggunakan Secondary Index

Operasi:

$$
B = A \bowtie Peminjaman
$$

Join dilakukan berdasarkan atribut `bid`.

Untuk setiap tuple Buku pada A, secondary index `bid` pada Peminjaman digunakan untuk menemukan tuple Peminjaman yang sesuai.

---

### Rata-Rata Peminjaman per Buku

Jumlah Peminjaman adalah 300.000 dan jumlah Buku adalah 50.000.

Dengan asumsi distribusi uniform:

$$
\frac{300000}{50000} = 6
$$

Artinya, setiap buku diperkirakan muncul pada **6 tuple Peminjaman**.

---

### Cost Secondary Index

Kedalaman secondary index adalah 3.

Untuk satu nilai `bid`, cost traversal index:

$$
3
$$

Karena terdapat sekitar 6 tuple Peminjaman untuk satu buku, dan secondary index tidak menjamin tuple berada pada blok yang berdekatan, digunakan asumsi worst case bahwa keenam tuple berada pada blok yang berbeda.

Dengan demikian cost untuk satu `bid` adalah:

$$
3 + 6 = 9
$$

Terdapat 100 buku pada A.

Maka:

$$
Cost_{join} = 100 \times 9
$$

$$
Cost_{join} = 900
$$

Jadi pencarian tuple Peminjaman menggunakan secondary index membutuhkan **900 block transfer**.

---

## 3. Banyak Tuple Hasil Join B

Terdapat 100 buku hasil seleksi dan setiap buku rata-rata mempunyai 6 tuple Peminjaman.

Maka:

$$
n_B = 100 \times 6 = 600
$$

Jadi hasil join mempunyai **600 tuple**.

---

## 4. Menghitung Jumlah Blok B

Peminjaman mempunyai:

- 300.000 tuple
- 15.000 blok

Jumlah tuple Peminjaman per blok:

$$
f_{Peminjaman} = \frac{300000}{15000} = 20
$$

Diketahui ukuran tuple hasil join Buku-Peminjaman adalah dua kali ukuran tuple Peminjaman.

Maka jumlah tuple hasil join yang dapat disimpan dalam satu blok menjadi:

$$
f_B = \frac{20}{2} = 10
$$

Karena B mempunyai 600 tuple:

$$
b_B = \frac{600}{10} = 60
$$

Jadi hasil B membutuhkan **60 blok**.

---

## 5. Materialisasi B

Hasil B akan digunakan sebagai inner relation pada join berikutnya.

Oleh karena itu, B dimaterialisasi ke disk.

Cost materialisasi:

$$
Cost_{materialisasi} = 60
$$

Dengan demikian total cost sampai menghasilkan B adalah:

$$
12 + 900 + 60 = 972
$$

---

## 6. Seleksi Pelanggan Umur Minimal 21 Tahun

Operasi:

$$
C = \sigma_{umur \geq 21}(Pelanggan)
$$

Tidak terdapat index pada atribut `umur`, sehingga dilakukan linear scan terhadap seluruh relasi Pelanggan.

Karena Pelanggan tersimpan dalam 1.000 blok:

$$
Cost(C) = 1000
$$

---

### Banyak Tuple Hasil C

Umur pelanggan berada dari 7 sampai 24 tahun.

Jumlah kemungkinan umur:

$$
24 - 7 + 1 = 18
$$

Umur yang memenuhi kondisi minimal 21 tahun adalah:

`21, 22, 23, 24`

Jadi terdapat 4 nilai yang memenuhi.

Selectivity:

$$
\frac{4}{18}
$$

Jumlah tuple:

$$
n_C = \frac{4}{18} \times 10000
$$

$$
n_C \approx 2222
$$

Jadi hasil C diperkirakan mempunyai **2.222 tuple**.

---

## 7. Join C dengan B

Operasi:

$$
D = C \bowtie B
$$

Digunakan:

- C sebagai outer relation.
- B sebagai inner relation.

Diketahui:

- Banyak tuple C = 2.222.
- Banyak blok B = 60.

Dengan tuple nested-loop join:

$$
Cost(D) = n_C \times b_B
$$

$$
Cost(D) = 2222 \times 60
$$

$$
Cost(D) = 133320
$$

Jadi cost join adalah **133.320 block transfer**.

---

## 8. Banyak Tuple Hasil D

B mempunyai 600 tuple.

Proporsi pelanggan dengan umur minimal 21 tahun adalah:

$$
\frac{4}{18}
$$

Sehingga:

$$
n_D = \frac{4}{18} \times 600
$$

$$
n_D \approx 133
$$

Jadi hasil join akhir diperkirakan mempunyai **133 tuple**.

---

## 9. Projection Nama

Operasi:

$$
\Pi_{nama}(D)
$$

Projection dilakukan menggunakan pipeline sehingga:

$$
Cost = 0
$$

Jumlah tuple akhir diasumsikan tetap sekitar **133 tuple**.

---

## Tabel Evaluation Plan Nomor 1

| Operasi | Cost (#block transfer) | Banyak Tuple |
| --- | ---: | ---: |
| Seleksi Buku menggunakan primary index | $2 + 10 = 12$ | $100$ |
| Index nested-loop join dengan Peminjaman | $100(3 + 6) = 900$ | $600$ |
| Materialisasi B | $60$ | $600$ |
| Seleksi Pelanggan umur minimal 21 | $1000$ | $2222$ |
| Join C dengan B | $2222 \times 60 = 133320$ | $133$ |
| Projection nama | $0$ | $133$ |
| **Total** | **135.292** | **133** |

---

## Total Cost Nomor 1

Total cost:

$$
12 + 900 + 60 + 1000 + 133320
$$

$$
= 135292
$$

Jadi:

$$
\boxed{Cost = 135292}
$$

block transfer.

Jumlah tuple akhir:

$$
\boxed{133}
$$

tuple.

---

## Nomor 2

### Membentuk Ekspresi Aljabar Relasional yang Ekivalen

Query awal adalah:

$$
\Pi_{nama}
\left(
  \sigma_{umur \geq 21}(Pelanggan)
  \bowtie
  \left(
    \sigma_{penulis = \text{Andrea Hirata}}(Buku)
    \bowtie
    Peminjaman
  \right)
\right)
$$

Query tersebut sebenarnya sudah cukup baik karena operasi selection sudah dilakukan sedekat mungkin dengan relasi asal.

Namun, menggunakan equivalence rules, atribut yang tidak dibutuhkan pada operasi berikutnya dapat diproyeksikan lebih awal.

Atribut yang akhirnya diperlukan adalah:

- Dari Pelanggan: `pid` dan `nama`.
- Dari Buku: `bid`.
- Dari Peminjaman: `pid` dan `bid`.

Maka dapat dibentuk:

$$
A' =
\Pi_{bid}
\left(
  \sigma_{penulis = \text{Andrea Hirata}}(Buku)
\right)
$$

$$
C' =
\Pi_{pid,nama}
\left(
  \sigma_{umur \geq 21}(Pelanggan)
\right)
$$

dan:

$$
P' = \Pi_{pid,bid}(Peminjaman)
$$

Sehingga ekspresi ekivalennya:

$$
\Pi_{nama}
\left(
  C'
  \bowtie_{pid}
  \left(
    A'
    \bowtie_{bid}
    P'
  \right)
\right)
$$

atau jika dituliskan secara lengkap:

$$
\Pi_{nama}
\left(
  \Pi_{pid,nama}
  \left(
    \sigma_{umur \geq 21}(Pelanggan)
  \right)
  \bowtie_{pid}
  \left(
    \Pi_{bid}
    \left(
      \sigma_{penulis = \text{Andrea Hirata}}(Buku)
    \right)
    \bowtie_{bid}
    \Pi_{pid,bid}(Peminjaman)
  \right)
\right)
$$

---

## Equivalence Rules yang Digunakan

Beberapa equivalence rules yang digunakan adalah:

### 1. Selection Pushdown

Kondisi:

`umur >= 21`

hanya menggunakan atribut Pelanggan, sehingga selection dilakukan langsung pada Pelanggan.

Begitu juga:

`penulis = Andrea Hirata`

hanya berhubungan dengan Buku, sehingga selection dilakukan langsung pada Buku.

Tujuannya adalah mengurangi jumlah tuple sebelum join.

---

### 2. Projection Pushdown

Atribut yang tidak diperlukan pada operasi selanjutnya dapat dibuang lebih awal.

Pada Buku hanya `bid` yang diperlukan setelah selection.

Maka:

$$
\Pi_{bid}
\left(
  \sigma_{penulis = \text{Andrea Hirata}}(Buku)
\right)
$$

Pada Pelanggan hanya `pid` dan `nama` yang diperlukan.

Maka:

$$
\Pi_{pid,nama}
\left(
  \sigma_{umur \geq 21}(Pelanggan)
\right)
$$

Sedangkan dari Peminjaman hanya `pid` dan `bid` yang diperlukan.

---

### 3. Join Associativity dan Commutativity

Join bersifat associative dan commutative sehingga urutan join dapat dipilih berdasarkan ukuran intermediate result.

Karena selection penulis hanya menghasilkan sekitar **100 Buku**, maka join Buku dengan Peminjaman tetap lebih baik dilakukan terlebih dahulu.

---

## Evaluation Plan untuk Ekspresi Baru

Evaluation plan yang digunakan:

1. Seleksi Buku karya Andrea Hirata.
2. Projection hanya atribut `bid`.
3. Join dengan Peminjaman berdasarkan `bid`.
4. Seleksi Pelanggan dengan umur minimal 21 tahun.
5. Projection hanya `pid` dan `nama`.
6. Join berdasarkan `pid`.
7. Projection akhir `nama`.

Projection dapat dilakukan menggunakan pipeline, sehingga tidak memerlukan tambahan block transfer.

---

## Cost Plan Nomor 2

Karena soal tidak memberikan informasi mengenai ukuran masing-masing atribut setelah projection, pengurangan ukuran blok akibat projection **tidak dapat dihitung secara pasti**.

Oleh karena itu, estimasi cost dilakukan secara konservatif menggunakan ukuran blok yang sudah diketahui.

Jika index tetap digunakan, cost fisiknya sama dengan plan nomor 1:

| Operasi | Cost (#block transfer) | Banyak Tuple |
| --- | ---: | ---: |
| Primary index Buku + projection bid | $12$ | $100$ |
| Secondary index Peminjaman | $900$ | $600$ |
| Materialisasi hasil join | $60$ | $600$ |
| Scan Pelanggan + projection pid,nama | $1000$ | $2222$ |
| Join dengan hasil B | $133320$ | $133$ |
| Projection akhir nama | $0$ | $133$ |
| **Total** | **135.292** | **133** |

Dengan statistik yang tersedia:

$$
\boxed{Cost = 135292}
$$

dan:

$$
\boxed{133\ tuple}
$$

---

## Alternatif Plan Nomor 2 Tanpa Menggunakan Index

Karena pada nomor 2 penggunaan index bersifat opsional, ekspresi baru juga dapat dievaluasi tanpa index.

Seleksi Buku harus melakukan linear scan:

$$
Cost = 5000
$$

Join dengan Peminjaman menggunakan nested-loop join:

$$
100 \times 15000 = 1500000
$$

Materialisasi hasil join:

$$
60
$$

Seleksi Pelanggan:

$$
1000
$$

Join akhir:

$$
2222 \times 60 = 133320
$$

Sehingga:

$$
Total
=
5000 + 1500000 + 60 + 1000 + 133320
$$

$$
Total = 1639380
$$

Jadi tanpa menggunakan index:

$$
\boxed{Cost = 1639380}
$$

block transfer.

Jumlah tuple akhir tetap:

$$
\boxed{133}
$$

tuple.

---

## Perbandingan

| Plan | Cost (#block transfer) | Tuple Akhir |
| --- | ---: | ---: |
| Plan menggunakan dua index | **135.292** | **133** |
| Ekspresi ekivalen + index | **135.292** | **133** |
| Ekspresi ekivalen tanpa index | **1.639.380** | **133** |

Dari perhitungan tersebut terlihat bahwa penggunaan index memberikan pengurangan cost yang sangat besar.

Pada pencarian Buku, cost berubah dari linear scan 5.000 blok menjadi sekitar 12 block transfer.

Pada join Buku dengan Peminjaman, penggunaan secondary index juga menghindari scanning seluruh 15.000 blok Peminjaman untuk setiap tuple Buku.

---

## Kesimpulan

### Nomor 1

Evaluation plan memanfaatkan:

- Primary index `penulis` pada Buku.
- Secondary index `bid` pada Peminjaman.

Total cost:

$$
\boxed{135292}
$$

block transfer.

Jumlah tuple akhir:

$$
\boxed{133}
$$

tuple.

### Nomor 2

Ekspresi ekivalen dapat diperoleh dengan menerapkan:

- Selection pushdown.
- Projection pushdown.
- Join associativity.
- Join commutativity.

Ekspresi hasil optimasi:

$$
\Pi_{nama}
\left(
  \Pi_{pid,nama}
  \left(
    \sigma_{umur \geq 21}(Pelanggan)
  \right)
  \bowtie_{pid}
  \left(
    \Pi_{bid}
    \left(
      \sigma_{penulis = \text{Andrea Hirata}}(Buku)
    \right)
    \bowtie_{bid}
    \Pi_{pid,bid}(Peminjaman)
  \right)
\right)
$$

Dengan menggunakan index, cost konservatif berdasarkan statistik yang tersedia adalah **135.292 block transfer**.

Jika tidak menggunakan index, cost-nya adalah **1.639.380 block transfer**.

Jumlah tuple akhir untuk kedua plan tetap diperkirakan sekitar **133 tuple**.



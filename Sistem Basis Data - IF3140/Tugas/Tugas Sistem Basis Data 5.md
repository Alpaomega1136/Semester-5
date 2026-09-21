# Evaluation Plan Query Basis Data

![[Pasted image 20260916102203.png]]
![[Pasted image 20260916102254.png]]

## Diketahui

Terdapat tiga relasi:

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
- Foreign key:
  - `pid` mengacu ke `Pelanggan(pid)`
  - `bid` mengacu ke `Buku(bid)`

### Informasi Tambahan

- Terdapat 500 penulis berbeda.
- Umur pelanggan berkisar dari 7 sampai 24 tahun.
- Distribusi nilai `penulis` dan `umur` diasumsikan uniform.
- Ukuran tuple hasil join `Buku` dengan `Peminjaman` adalah 2 kali ukuran tuple `Peminjaman`.
- Evaluasi menggunakan pipeline, kecuali apabila diperlukan materialisasi.
- Join menggunakan nested-loop join dengan asumsi worst case.

---

## Query

Query yang diberikan:

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

Untuk mempermudah perhitungan, didefinisikan:

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

Hasil akhirnya:

$$
\Pi_{nama}(D)
$$

---

## 1. Seleksi Buku Berdasarkan Penulis

Operasi pertama adalah:

$$
A = \sigma_{penulis = \text{Andrea Hirata}}(Buku)
$$

Karena tidak terdapat informasi mengenai adanya index pada atribut `penulis`, operasi seleksi dilakukan menggunakan **linear scan** terhadap seluruh relasi Buku.

Jumlah blok Buku adalah:

$$
b_{Buku} = 5000
$$

Maka cost operasi seleksi:

$$
Cost(A) = b_{Buku}
$$

$$
Cost(A) = 5000
$$

Jadi cost yang dibutuhkan adalah **5.000 block transfer**.

### Banyak Tuple Hasil Seleksi

Terdapat 500 penulis berbeda dan distribusi atribut `penulis` dianggap uniform.

Oleh karena itu, setiap penulis diperkirakan mempunyai jumlah buku yang sama.

Jumlah tuple hasil seleksi:

$$
n_A = \frac{50000}{500}
$$

$$
n_A = 100
$$

Jadi hasil operasi A adalah **100 tuple**.

---

## 2. Join A dengan Peminjaman

Operasi berikutnya adalah:

$$
B = A \bowtie Peminjaman
$$

Join dilakukan berdasarkan atribut `bid`.

Nested-loop join menggunakan:

- `A` sebagai outer relation.
- `Peminjaman` sebagai inner relation.

Diketahui:

$$
n_A = 100
$$

dan:

$$
b_{Peminjaman} = 15000
$$

Cost nested-loop join sebelum memperhitungkan materialisasi adalah:

$$
Cost = n_A \times b_{Peminjaman}
$$

$$
Cost = 100 \times 15000
$$

$$
Cost = 1500000
$$

### Banyak Tuple Hasil Join

Atribut `bid` merupakan primary key pada Buku dan foreign key pada Peminjaman.

Hasil A hanya berisi buku karya Andrea Hirata.

Karena terdapat 500 penulis berbeda dengan distribusi uniform, maka diperkirakan bagian Peminjaman yang berhubungan dengan buku Andrea Hirata adalah:

$$
\frac{1}{500}
$$

dari seluruh Peminjaman.

Maka:

$$
n_B = \frac{300000}{500}
$$

$$
n_B = 600
$$

Jadi hasil join B mempunyai **600 tuple**.

---

## 3. Menghitung Jumlah Blok Hasil B

Relasi Peminjaman mempunyai:

$$
n_{Peminjaman} = 300000
$$

tuple dan:

$$
b_{Peminjaman} = 15000
$$

blok.

Jumlah tuple Peminjaman per blok adalah:

$$
f_{Peminjaman}
=
\frac{n_{Peminjaman}}{b_{Peminjaman}}
$$

$$
f_{Peminjaman}
=
\frac{300000}{15000}
$$

$$
f_{Peminjaman} = 20
$$

Jadi satu blok Peminjaman dapat menampung **20 tuple**.

Diketahui bahwa ukuran tuple hasil join `Buku` dengan `Peminjaman` adalah **2 kali ukuran tuple Peminjaman**.

Karena ukuran tuple menjadi dua kali lebih besar, jumlah tuple yang dapat disimpan dalam satu blok menjadi setengahnya.

Maka:

$$
f_B = \frac{20}{2}
$$

$$
f_B = 10
$$

Jadi satu blok hasil join B dapat menampung **10 tuple**.

Karena B mempunyai 600 tuple, jumlah blok B adalah:

$$
b_B = \frac{n_B}{f_B}
$$

$$
b_B = \frac{600}{10}
$$

$$
b_B = 60
$$

Jadi hasil B membutuhkan **60 blok**.

---

## 4. Materialisasi B

Hasil B akan digunakan sebagai inner relation pada operasi join berikutnya.

Oleh karena itu, hasil B perlu **dimaterialisasi**, yaitu dituliskan ke disk terlebih dahulu.

Diketahui:

$$
b_B = 60
$$

Maka tambahan cost untuk materialisasi adalah:

$$
Cost_{materialisasi} = 60
$$

Total cost untuk menghasilkan B adalah:

$$
Cost(B)
=
n_A \times b_{Peminjaman} + b_B
$$

$$
Cost(B)
=
100 \times 15000 + 60
$$

$$
Cost(B)
=
1500000 + 60
$$

$$
Cost(B)
=
1500060
$$

Jadi total cost untuk operasi B adalah **1.500.060 block transfer**.

---

## 5. Seleksi Pelanggan dengan Umur Minimal 21 Tahun

Operasi berikutnya adalah:

$$
C = \sigma_{umur \geq 21}(Pelanggan)
$$

Karena tidak terdapat informasi mengenai adanya index pada atribut `umur`, maka dilakukan **linear scan** terhadap seluruh relasi Pelanggan.

Jumlah blok Pelanggan:

$$
b_{Pelanggan} = 1000
$$

Maka:

$$
Cost(C) = b_{Pelanggan}
$$

$$
Cost(C) = 1000
$$

Jadi cost operasi seleksi Pelanggan adalah **1.000 block transfer**.

---

## 6. Menghitung Banyak Tuple C

Umur pelanggan berada pada rentang 7 sampai 24 tahun.

Jumlah kemungkinan nilai umur:

$$
24 - 7 + 1 = 18
$$

Umur yang memenuhi kondisi:

$$
umur \geq 21
$$

adalah:

$$
21, 22, 23, 24
$$

Jadi terdapat 4 nilai umur yang memenuhi kondisi.

Karena distribusi umur uniform, selectivity kondisi tersebut adalah:

$$
Selectivity = \frac{4}{18}
$$

Jumlah tuple hasil seleksi:

$$
n_C
=
\frac{4}{18}
\times
10000
$$

$$
n_C
=
2222.22
$$

Digunakan estimasi:

$$
n_C \approx 2222
$$

Jadi terdapat sekitar **2.222 tuple** hasil seleksi.

---

## 7. Join C dengan B

Operasi berikutnya:

$$
D = C \bowtie B
$$

Pada operasi ini:

- `C` menjadi outer relation.
- `B` menjadi inner relation.
- `B` sudah dimaterialisasi sebelumnya.

Diketahui:

$$
n_C = 2222
$$

dan:

$$
b_B = 60
$$

Cost nested-loop join:

$$
Cost(D)
=
n_C \times b_B
$$

$$
Cost(D)
=
2222 \times 60
$$

$$
Cost(D)
=
133320
$$

Jadi cost join C dengan B adalah **133.320 block transfer**.

---

## 8. Banyak Tuple Hasil D

Relasi B mempunyai:

$$
n_B = 600
$$

tuple.

Proporsi pelanggan dengan umur minimal 21 tahun adalah:

$$
\frac{4}{18}
$$

Maka estimasi jumlah tuple hasil join adalah:

$$
n_D
=
\frac{4}{18}
\times
600
$$

$$
n_D
=
133.33
$$

Digunakan estimasi:

$$
n_D \approx 133
$$

Jadi hasil join D diperkirakan mempunyai **133 tuple**.

---

## 9. Projection Nama

Operasi terakhir adalah:

$$
\Pi_{nama}(D)
$$

Projection dapat dilakukan secara **pipeline**, sehingga tidak membutuhkan tambahan block transfer.

Maka:

$$
Cost = 0
$$

Jumlah tuple hasil akhirnya diasumsikan:

$$
n_{final} = 133
$$

> [!NOTE]
> Secara formal, projection pada aljabar relasional dapat menghilangkan tuple duplikat. Namun, soal tidak memberikan informasi mengenai jumlah nilai `nama` yang berbeda. Oleh karena itu, estimasi yang digunakan mengikuti solusi soal, yaitu 133 tuple.

---

## Tabel Evaluation Plan

| Operasi | Cost (#block transfer) | Banyak Tuple |
| --- | ---: | ---: |
| Seleksi Buku berdasarkan penulis, hasil A | $5000$ | $100$ |
| Join A dengan Peminjaman, hasil B + materialisasi | $100 \times 15000 + 60 = 1500060$ | $600$ |
| Seleksi Pelanggan umur minimal 21, hasil C | $1000$ | $2222$ |
| Join C dengan B, hasil D | $2222 \times 60 = 133320$ | $133$ |
| Projection nama | $0$ | $133$ |
| **Total** | **1.639.380** | **133** |

---

## Total Cost

Total cost seluruh evaluation plan:

$$
Total
=
5000
+
1500060
+
1000
+
133320
+
0
$$

$$
Total = 1639380
$$

Jadi total cost adalah:

$$
\boxed{1639380}
$$

block transfer.

Jumlah tuple akhir adalah:

$$
\boxed{133}
$$

tuple.

---

## Kesimpulan

Evaluation plan yang digunakan adalah:

1. Melakukan linear scan pada Buku untuk mendapatkan buku karya Andrea Hirata.
2. Melakukan nested-loop join antara hasil seleksi Buku dan Peminjaman.
3. Mematerialisasi hasil join B karena akan digunakan sebagai inner relation pada join berikutnya.
4. Melakukan linear scan pada Pelanggan untuk mendapatkan pelanggan dengan umur minimal 21 tahun.
5. Melakukan nested-loop join antara hasil seleksi Pelanggan dan B.
6. Melakukan projection terhadap atribut `nama` secara pipeline.

Total cost yang diperoleh adalah:

$$
\boxed{1639380}
$$

block transfer.

Jumlah tuple hasil akhir:

$$
\boxed{133}
$$

tuple.


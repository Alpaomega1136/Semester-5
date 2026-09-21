
Diberikan skema basis data berikut ini. Atribut dengan garis bawah adalah *primary key*, sedangkan atribut tercetak *miring* adalah *foreign key* (ke atribut *primary key* dengan nama sama):
![[Pasted image 20260911070729.png]]
Berdasarkan hasil analisis terhadap *workload* sistem yang telah berjalan dan data yang ada di basis data, diperoleh fakta berikut ini.

- Terdapat 10.000 *record* pada SUPPLIER, 12.000 pada PRODUCT, 1.500.000 pada SUPPLY, dan 100 pada PRODUCT\_CATEGORY.
- 90% akses terhadap produk selalu membutuhkan nama kategori dari produk tersebut.
- 80% akses terhadap *supply* hanya mengakses data pesanan dari *supplier* yang belum diterima saja. Setiap saat, rata-rata hanya ada 20.000 hingga 30.000 pesanan dari *supplier* yang belum diterima. Pesanan yang belum diterima ditandai dengan atribut RECEIVEDATE pada suatu baris data pesanan di tabel SUPPLY bernilai NULL.
- Atribut Alamat, Kota, dan Rating dari *supplier* hanya diakses setiap awal bulan, pada saat akan membuat laporan kegiatan pada bulan sebelumnya. Atribut Balance sering sekali berubah, yaitu setiap kali ada pesanan ke *supplier* tersebut.
- Pencarian data supplier dan product sangat sering dilakukan berdasarkan nama.
- Untuk memenuhi kebutuhan dari pihak eksekutif, query untuk mendapatkan total harga terhutang dari seluruh pesanan yang belum diterima, baik per *supplier* maupun per tanggal pesanan, sering dijalankan.

**Tugas Anda:**

Berikan usulan schema tuning untuk meningkatkan kinerja sistem basis data di atas, termasuk kemungkinan untuk memanfaatkan materialized view dan kebutuhan akan indeks. Indeks yang tidak perlu dituliskan adalah indeks yang berkaitan dengan primary key. Untuk setiap usulan:

- Tuliskan jenis tuning-nya dan jelaskan.
- Tuliskan skema baru yang dihasilkan (jika ada).
- Tuliskan kebutuhan proses tambahan di basis data untuk tetap menjaga konsistensi data (jika ada).
- Jika perlu index, tuliskan dengan jelas relasi dan atribut indeks dan jenis indeks yang dibutuhkan.
- Untuk setiap materialized view yang Anda usulkan, tuliskan perintah create view untuk mendapatkannya.

Di akhir jawaban, tuliskan kembali seluruh skema basis data final yang Anda usulkan, termasuk skema materialized view (jika ada).

# Jawaban
## 1. Vertical Partitioning pada Supplier

**SUPPLIER (SID, SNAME, SADDRESS, SCITY, RATING, BALANCE)**  
Atribut SADDRESS, SCITY, dan RATING hanya digunakan sekali setiap awal bulan, sedangkan BALANCE sangat sering diperbarui dan SNAME sangat sering digunakan untuk pencarian sehingga lebih baik tabel SUPPLIER dipisahkan secara vertikal menjadi   
**SUPPLIER(SID, SNAME, BALANCE)**  
**SUPPLIER_DETAIL (****_SID_****,SADDRESS, SCITY, RATING) 

**SUPPLIER_DETAIL.SID → SUPPLIER.SID**  
*dengan warna merah adalah PK

Karena pencarian suplier sering dilakukan berdasarkan nama, perlu indeks :

CREATE INDEX IDX_SUPPLIER_SNAME  
ON SUPPLIER(SNAME);

jenis indeks : Secondary B+-Tree Index

## 2.Denormalisasi PRODUCT dan PRODUCT_CATEGORY

Pada fakta yang ada, "90% akses terhadap PRODUCT membutuhkan nama kategorinya" namun pada desain awal harus melalukan JOIN terlebih dahulu dengan PRODUCT_CATEGORY. Sehingga diperlukan selective denormalization, yaitu menyimpan CNAME langsung pada PRODUCT.  
**PRODUCT(PID, PNAME,** **_CID_****, CNAME,WEIGHT)**

**PRODUCT_CATEGORY( CID, CNAME, DESCRIPTION)**

PRODUCT.CID → PRODUCT_CATEGORY.CID  
*dengan warna merah adalah PK

karena CNAME terdapat di dua tempat sehingga dibutuhkan mekanisme konsistensi dengan menggunakan trigger atau transaction/stored procedure. Jika PRODUCT_CATEGORY.CNAME berubah, seluruh PRODUCT.CNAME dengan CID yang sama harus ikut diperbarui. Karena product sangat sering dicari berdasarkan nama:

CREATE INDEX IDX_PRODUCT_PNAME  
ON PRODUCT(PNAME);

jenis indeks : Secondary B+-Tree Index

## 3.Horizontal Partitioning pada SUPPLY

Karena jumlah data SUPPLY berjumlah 1.500.000 record tetapi jumlah pesanan yang belum diterima hanya : 20.000 - 30.000 record yang artinya data aktif hanya sekitar 1,33% - 2% dan karena 80% akses terhadap SUPPLY hanya mengakses pesanan yang belum diterima, maka SUPPLY perlu dilakukan horizontal partition berdasarkan status penerimaan. Sehingga :  
**SUPPLY_PENDING(ORDERDATE,PID,SID,QUANTITY,PRICE)**  
SUPPLY_PENDING.PID → PRODUCT.PID

SUPPLY_PENDING SID → SUPPLIER.SID

**SUPPLY_RECEIVED(ORDERDATE, PID, SID QUANTITY, PRICE, RECEIVEDATE)**

SUPPLY_RECEIVED.PID → PRODUCT.PID

SUPPLY_RECEIVED.SID → SUPPLIER.SID

*dengan warna merah adalah PK  
Ketika pesanan baru dibuat, data dimasukkan ke SUPPLY_PENDING. Ketika pesanan diterima, data dipindahkan dari SUPPLY_PENDING ke SUPPLY_RECEIVED menggunakan transaction atau stored procedure agar konsistensi data tetap terjaga.

## 4. Index untuk SUPPLY_PEDDING

Karena pending order sering dibutuhkan berdasarkan supplier, maka diperlukan indeks :

CREATE INDEX IDX_SUPPLY_PENDING_SID

ON SUPPLY_PENDING(SID)

jenis : Secondary B+-Tree Index

## 5. Materialized View total hutang per supplier

karena diperlukan query untuk “Total harga terhutang seluruh pesanan yang belum diterima per supplier” maka dibuatkan materialized view : 

CREATE MATERIALIZED VIEW LOAN_BY_SUPPLIER AS  
SELECT  
SID, SUM(QUANTITY * PRICE) AS TOTAL_HUTANG  
FROM SUPPLY_PENDING  
GROUP BY SID;

skema : LOAN_BY_SUPPLIER(SID, TOTAL_HUTANG)

yang kemudian dilakukan index dengan jenis : Unique B+-Tree Index (SID)

CREATE UNIQUE INDEX IDX_LOAN_BY_SUPPLIER_SID

ON LOAN_BY_SUPPLIER(SID);

Materialized view harus di-refresh ketika isi SUPPLY_PENDING berubah.

## 6. Materialized View total hutang per tanggal

karena diperlukan query untuk “Total harga  terhutang seluruh pesanan yang belum diterima per tanggal” maka dibuatkan materialized view :

CREATE MATERIALIZED VIEW LOAN_BY_DATE AS  
SELECT   
ORDERDATE, SUM(QUANTITY * PRICE) AS TOTAL_HUTANG

FROM SUPPLY_PENDING

GROUP BY ORDERDATE

skema : LOAN_BY_DATE(ORDERDATE, TOTAL_HUTANG)

yang kemudian dilakukan index dengan jenis : Unique B+-Tree index (ORDERDATE)

CREATE UNIQUE INDEX IDX_LOAN_BY_DATE_ORDERDATE

ON LOAN_BY_DATE(ORDERDATE);

Materialized view harus di-refresh ketika isi SUPPLY_PEDDING berubah

## HASIL AHKIR

Skema final :

SUPPLIER(**SID**,SNAME, BALANCE)

SUPPLIER_DETAIL(**SID**, SADDRESS, SCITY, RATING)

PRODUCT_CATEGORY(**CID**, CNAME, DESCRIPTION)

PRODUCT (**PID**, PNAME, CID, CNAME, WEIGHT)

SUPPLY_PENDING(**ORDERDATE, PID, SID**, QUANTITY, PRICE)

SUPPLY_RECEIVED(**ORDERDATE, PID, SID**, QUANTITY, PRICE, RECEIVEDATE)

Materialized view:

LOAN_BY_SUPPLIER(SID, TOTAL_HUTANG)

LOAN_BY_DATE(ORDERDATE, TOTAL_HUTANG)

FK :

SUPPLIER_DETAIL.SID → SUPPLIER.SID

PRODUCT.CID → PRODUCT_CATEGORY.CID

SUPPLY_PENDING.PID → PRODUCT.PID

SUPPLY_PENDING.SID → SUPPLIER.SID

SUPPLY_RECEIVED.PID → PRODUCT.PID

SUPPLY_RECEIVED.SID → SUPPLIER.SID
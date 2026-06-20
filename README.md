# Lake-House
<img width="1672" height="941" alt="lakehouse-image" src="https://github.com/user-attachments/assets/3f7bbf6d-ba58-4f56-acef-ace64d034d3a" />

Menurut saya, untuk presentasi ke manajemen atau tim arsitektur, narasi sebaiknya tidak hanya menjelaskan fungsi masing-masing komponen, tetapi juga **menceritakan perjalanan data (data journey)** dari awal hingga menjadi informasi yang siap digunakan. Dengan begitu audiens lebih mudah memahami mengapa setiap komponen diperlukan.

Berikut narasi yang saya rekomendasikan.

---

# Arsitektur Lakehouse

Arsitektur Lakehouse merupakan platform data modern yang mengintegrasikan proses **ingestion**, **penyimpanan**, **manajemen metadata**, dan **analitik** ke dalam satu ekosistem yang terintegrasi. Pendekatan ini memungkinkan organisasi menyimpan seluruh jenis data dalam satu repositori terpusat tanpa harus melakukan duplikasi data ke berbagai sistem analitik.

Pada arsitektur ini terdapat lima komponen utama yang bekerja secara berurutan, dimulai dari proses integrasi data hingga penyajian data untuk kebutuhan analisis bisnis.

---

# 1. Integration Layer (Apache NiFi)

Tahap pertama adalah **Integration Layer**, yang berfungsi sebagai pintu masuk seluruh data dari berbagai sistem operasional.

Sumber data dapat berasal dari:

* Core Banking
* Mobile Banking
* Internet Banking
* BI-FAST
* ATM
* CRM
* Database
* REST API
* Batch File
* Streaming Platform

Seluruh data tersebut diterima oleh **Apache NiFi** untuk diproses sebelum disimpan ke Lakehouse.

Selain melakukan pemindahan data (data ingestion), Apache NiFi juga memiliki kemampuan untuk:

* melakukan validasi data,
* melakukan transformasi sederhana,
* melakukan routing ke tujuan yang sesuai,
* mengelola retry apabila terjadi kegagalan,
* mencatat seluruh perjalanan data (Data Provenance),
* mengendalikan lonjakan trafik melalui mekanisme Back Pressure.

Pendekatan visual drag-and-drop yang dimiliki Apache NiFi memungkinkan pipeline data dibangun dengan cepat serta mudah dipelihara tanpa memerlukan pengembangan kode yang kompleks.

Output dari layer ini adalah data yang telah berhasil diintegrasikan dan siap disimpan ke Object Storage.

---

# 2. Lakehouse Storage Layer (Object Storage)

Data yang diterima dari Apache NiFi kemudian disimpan pada **Object Storage** yang berfungsi sebagai media penyimpanan utama Lakehouse.

Berbeda dengan data warehouse tradisional yang menggunakan storage khusus dengan biaya tinggi, Object Storage menawarkan kapasitas penyimpanan yang elastis sehingga dapat berkembang mengikuti pertumbuhan volume data.

Seluruh jenis data dapat disimpan pada layer ini, baik berupa:

* JSON
* CSV
* XML
* Parquet
* Log aplikasi
* File transaksi
* Dokumen
* maupun data tidak terstruktur lainnya.

Selain memiliki biaya penyimpanan yang rendah, Object Storage juga menyediakan mekanisme redundansi otomatis sehingga data tetap aman meskipun terjadi kegagalan perangkat keras.

Object Storage menjadi fondasi utama seluruh platform Lakehouse karena seluruh engine analitik akan membaca data langsung dari lokasi ini.

---

# 3. Open Table Format (Apache Iceberg)

Walaupun data telah tersimpan di Object Storage, file-file tersebut masih belum memiliki struktur layaknya sebuah database.

Di sinilah **Apache Iceberg** berperan sebagai **Open Table Format** yang mengubah kumpulan file pada Object Storage menjadi tabel yang dapat diakses menggunakan SQL.

Apache Iceberg menyediakan berbagai kemampuan penting, di antaranya:

### Hidden Partition

Partisi dikelola secara otomatis sehingga pengguna tidak perlu mengetahui lokasi fisik data ketika melakukan query.

---

### Schema Evolution

Perubahan struktur tabel, seperti penambahan kolom atau perubahan tipe data, dapat dilakukan tanpa harus memindahkan ataupun menulis ulang seluruh data.

---

### Time Travel

Setiap perubahan data disimpan sebagai snapshot sehingga pengguna dapat melihat kondisi data pada waktu tertentu maupun melakukan rollback apabila diperlukan.

---

### ACID Transaction

Apache Iceberg menjamin konsistensi data walaupun terdapat banyak proses membaca dan menulis secara bersamaan.

Dengan adanya Apache Iceberg, Object Storage yang semula hanya berupa kumpulan file berubah menjadi media penyimpanan yang memiliki karakteristik layaknya database modern.

---

# 4. Catalog & Metadata Layer (Apache Polaris)

Setelah tabel terbentuk, seluruh metadata dikelola oleh **Apache Polaris**.

Apache Polaris bertindak sebagai pusat informasi mengenai seluruh aset data yang berada di dalam Lakehouse.

Metadata yang dikelola meliputi:

* lokasi tabel,
* struktur kolom,
* partisi,
* versi tabel,
* hak akses,
* namespace,
* hingga informasi governance.

Dengan adanya Apache Polaris, seluruh aplikasi maupun query engine mengakses metadata yang sama sehingga tidak terjadi inkonsistensi antar platform.

Selain itu, penggunaan REST Catalog menjadikan Lakehouse tetap terbuka dan kompatibel dengan berbagai engine analitik seperti Apache Doris, Spark, Flink maupun Trino.

Apache Polaris berperan sebagai **Single Source of Truth** terhadap seluruh metadata dalam ekosistem Lakehouse.

---

# 5. Query Engine (Apache Doris)

Tahap terakhir adalah proses konsumsi data menggunakan **Apache Doris** sebagai Query Engine.

Apache Doris membaca metadata dari Apache Polaris, kemudian mengakses data yang berada pada Object Storage melalui tabel Apache Iceberg.

Karena menggunakan pendekatan **Zero-ETL Federation**, Apache Doris tidak perlu melakukan proses copy ataupun memindahkan data ke database lain.

Seluruh query dieksekusi langsung terhadap data yang tersimpan di Lakehouse.

Keunggulan pendekatan ini adalah:

* mengurangi biaya penyimpanan,
* menghilangkan proses sinkronisasi data,
* mempercepat penyajian informasi,
* mengurangi kompleksitas arsitektur.

Dengan arsitektur Massively Parallel Processing (MPP), Apache Doris mampu melakukan analisis data dalam hitungan detik walaupun volume data telah mencapai skala petabyte.

Selain itu Apache Doris mampu melayani ribuan query secara bersamaan sehingga sangat sesuai digunakan sebagai platform dashboard operasional maupun analitik bisnis.

---

# Alur Data (Lakehouse Flow)



# Kesimpulan

Arsitektur ini membentuk sebuah **pipeline data end-to-end** yang sederhana namun kuat. Data dari berbagai sistem operasional pertama kali diintegrasikan menggunakan **Apache NiFi**, kemudian disimpan secara terpusat pada **Object Storage**. Agar data tersebut dapat diperlakukan sebagai tabel analitik yang andal, **Apache Iceberg** menyediakan kemampuan seperti ACID Transaction, Schema Evolution, Hidden Partition, dan Time Travel. Seluruh metadata dan tata kelola tabel dikelola secara terpusat oleh **Apache Polaris**, sehingga semua engine memiliki pandangan metadata yang konsisten. Terakhir, **Apache Doris** memanfaatkan metadata tersebut untuk menjalankan query SQL langsung terhadap data di Object Storage tanpa proses ETL tambahan, menghasilkan analitik berperforma tinggi dengan latensi rendah.

Pendekatan ini memungkinkan organisasi memiliki **satu sumber data (Single Source of Truth)** yang dapat digunakan secara bersamaan untuk kebutuhan pelaporan, dashboard operasional, analitik bisnis, audit, hingga pengembangan AI dan Machine Learning, tanpa perlu membuat banyak salinan data di berbagai sistem.



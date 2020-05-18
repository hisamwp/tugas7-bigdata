# Tugas 7 - Melakukan Analisa Iris Meter Spark Menggunakan KNIME

Workflow yang akan dijalankan pada tugas ini adalah sebagai berikut

![](/screenshoot/1.PNG)

Workflow ini berisi 3 meta node, yaitu

![](/screenshoot/1.1.PNG)

``Load Data Node``

![](/screenshoot/1.2.PNG)

``Extract date-time attributes``

![](/screenshoot/1.3.PNG)

``Aggregation and time series``

### 1 Business Understanding
Data yang digunakan pada workflow ini adalah data Irish Energy Meter, dimana data tersebut berisi data penggunaan listrik di Irlandia dalam jangka waktu tertentu. Dengan data tersebut dapat dilakukan peng-cluster-an, dengan tujuan memberikan informasi yang berguna seperti informasi konsumsi energi, atau hal lainnya.


### 2 Data Understanding
Data ini berisi data penggunaan listrik di Irlandia, yang terdiri dari 3 kolom, yaitu:

- meterID, merupakan integer ID untuk tiap meteran yang ada.
- enc_datetime, sebuah integer yang berisi informasi waktu penggunaan.
- reading, sebuah float yang berisi informasi penggunaan KWh pada meterID dan waktu tertentu.

### 3 Data Preparation

![](/screenshoot/2.PNG)

Pada data preparation kita akan mempersiapkan data Irish Energy Meter yang telah disediakan pada KNIME.
Node yang dijalankan pertama kali adalah file manager, yaitu melakukan load data Irish Energy Meter, kemudian membuat local big data env, dan menjalankan Meta Node ``Load Data``.

![](/screenshoot/5.PNG)

Melakukan load data Irish Energy Meter.

![](/screenshoot/3.PNG)

Meta node ``Load Data`` berisi 2 node, dimana node pertama adalah pembuatan table pada hive dan node kedua melakukan load table yang telah dibuat, hasil table yang telah di buat adalah sebagai berikut.

![](/screenshoot/4.PNG)

Setelah melakukan load data dan membuatnya menjadi table hive, kita ubah table hive tadi menjadi spark dengan menjalankan node ``Hive to Spark``. Hasil dari table spark tersebut adalah

![](/screenshoot/6.PNG)


### 4 Modeling
Selanjutnya adalah melakukan modeling untuk merubah isi table yang ada, dengan melakukan pemecahan data untuk dilakukan analisa, workflow yang dijalankan adalah

![](/screenshoot/7.PNG)

Pertama kita akan menjalankan meta node ``Extract date-time attributes``, dimana meta node ini akan melakukan pemisahan data yang nantinya akan di lakukan analisa,

![](/screenshoot/8.PNG)

Ada 4 tahapan pada node ini, dimana tahapan tersebut semua menggunakan node ``Spark SQL Query``, namun dengan pengaturan yang berbeda. Tahap pertama berisi:

![](/screenshoot/9.PNG)

Pada tahap ini, kita melakukan query select data yang berada pada table, lalu melakukan konversi data enc_datetime. Hasilnya adalah sebagai berikut:

![](/screenshoot/10.PNG)

Setelah kita mendapatkan kolom eventDate, kita ekstraksi kolom tersebut untuk mendapatkan tahun, bulan, minggu, hari, dengan menjalankan ``Spark SQL Query`` tahap kedua,

![](/screenshoot/11.1.PNG)

Hasil dari query tahap kedua adalah sebagai berikut:

![](/screenshoot/11.PNG)

Tahap ketiga dari meta node ``Extract Time`` adalah melakukan klasifikasi dari kolom dayOfWeek, menggunakan query sebagai berikut:

![](/screenshoot/12.1.PNG)

Query ini akan melakukan pembuatan column baru bernama dayOfClassifier, dimana berisi nilai WE apabila dayOfWeek bernilai ('Saturday', 'Sunday'), atau berisi nilai BD apabila selain ('Saturday', 'Sunday'), sehingga mendapatkan hasil:

![](/screenshoot/12.PNG)

Tahap keempat atau tahap terakhir adalah melakukan segmentasi berdasarkan kolom hour, dimana menggunakan query sebagai berikut:

![](/screenshoot/13.1.PNG)

Query ini akan menghasilkan segmentasi jam, yang akan dimasukkan nilainya di kolom daySegment. Berikut hasil querynya:

![](/screenshoot/13.PNG)

Semua node telah dijalankan, dan hasil column nantinya akan dilakukan analisa pada meta node ``Aggregation and time series``, meta node ini berisi sejumlah node seperti berikut

![](/screenshoot/1.3.PNG)

Pada meta node ini, kita menerima input dan menyimpannya didalam memory sementara menggunakan node Persist Spark Dataframe/RDD.

![](/screenshoot/14.PNG)

Lalu data tersebut dihitung rata-ratanya per segment yang sesuai (tahun, bulan, hari, dsb..) menggunakan maksimal 3 node yaitu Spark GroupBy, Spark Pivot dan Spark Column Rename.

![](/screenshoot/15.PNG)

Menghitung usage keseluruhan dan menghitung rata-rata per segment tahun.

![](/screenshoot/15.1.PNG)

Menghitung rata-rata per segment bulan.

![](/screenshoot/15.2.PNG)

Menghitung rata-rata per segment minggu.

![](/screenshoot/15.3.PNG)

Menghitung rata-rata per segment hari di 1 minggu.

![](/screenshoot/15.4.PNG)

Menghitung rata-rata per segment harian.

![](/screenshoot/15.5.PNG)

Menghitung rata-rata per segment jam di 1 hari.

![](/screenshoot/15.6.PNG)

Menghitung rata-rata per segment klasifikasi hari.

![](/screenshoot/15.7.PNG)

Menghitung rata-rata per segment jam.

Setelah itu, data rata-rata tersebut dijoin menggunakan node Spark Joiner dan diteruskan ke general workflow.

Hasil akhir dari table yang telah selesai di proses adalah sebagai berikut:

![](/screenshoot/26.PNG)

Setelah mendapatkan data tersebut, kita hitung persentase nya per minggu / hari sesuai dengan segmentnya, misalkan segmentnya adalah jam, maka dihitung per hari, atau segmentnya hari, maka dihitung per minggu, menggunakan node SQL Spark Query dengan syntax:

![](/screenshoot/16.PNG)

Hasilnya adalah sebagai berikut:

![](/screenshoot/17.PNG)

### 5 Evaluation
Pada evaluation akan dijalankan workflow

![](/screenshoot/27.PNG)

![](/screenshoot/27.1.PNG)

Dapat dilihat pada node Normalizer, semua data kecuali ID, dinormalisasi menjadi range 0 - 1. Pada node setelah Denormalizer data dioutputkan menjadi 2 bentuk yaitu visualisasi dan data table yang diteruskan ke general workflow. Selanjutnya, data output tadi dimasukkan kembali ke Local Big Data Environment menggunakan 2 node, yaitu Spark to Hive untuk load menjadi Apache Hive dan Spark to Parquet untuk load menjadi HDFS.

Hasilnya adalah sebagai berikut:

![](/screenshoot/30.PNG)
![](/screenshoot/31.PNG)


### 6 Deployment
Selanjutnya pada tahap deployment kita akan menjalankan workflow

![](/screenshoot/32.PNG)

Pada tahap ini dilakukan perubahan data dari spark kembali mejadi hive serta menyimpan spark kedalam HDFS dalam bentuk parquet, hasil dari data tersebut adalah sebagai berikut

![](/screenshoot/33.PNG)

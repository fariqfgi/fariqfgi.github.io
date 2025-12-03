---
title: "Testing IDOR with Predictable MongoDB ObjectIDs"
date: 2025-12-03 08:50:00 +07:00
tags: [bugbounty]
description: MongoDB Object ID sering dianggap sebagai identifier yang aman karena formatnya terlihat random seperti 68d7d596bf98cf7b08a79f92. Namun, struktur Object ID memiliki komponen yang relatif dapat diprediksi.
---

### IDOR through MongoDB Object IDs Prediction
MongoDB Object ID sering dianggap sebagai identifier yang aman karena formatnya terlihat random seperti `68d7d596bf98cf7b08a79f92`. Namun, struktur Object ID memiliki komponen yang relatif dapat diprediksi.
Meski demikian, penting untuk dipahami bahwa prediktabilitas Object ID bukanlah root cause dari kerentanan [IDOR](https://portswigger.net/web-security/access-control/idor) - masalah sebenarnya terletak pada kurangnya validasi otorisasi di level aplikasi.

![Image](/assets/img/posts/mongodb-objectids.png)

MongoDB Object ID terdiri dari 12-byte (24 karakter heksadesimal) dengan format berikut:
- 4 byte pertama: Timestamp Unix
- 3 byte berikutnya: Machine Identifier
- 2 byte berikutnya: Process ID
- 3 byte terakhir: Counter Incremental

### Mengapa Prediksi Object ID Memudahkan Testing IDOR
Karena `machine identifier` dan `process ID` cenderung statis, attacker hanya perlu mengiterasi timestamp dan counter untuk menghasilkan Object ID yang `mungkin` valid. Pada sistem dengan traffic tinggi, MongoDB dapat menghasilkan ribuan Object ID per detik, sehingga counter antar resource bisa berbeda signifikan meskipun dibuat hampir bersamaan. Tool seperti [mongo-objectid-predict](https://github.com/andresriancho/mongo-objectid-predict) dapat menghasilkan sekitar 1000 kemungkinan Object ID berdasarkan satu seed Object ID yang diketahui, memudahkan testing IDOR.

### Root Cause Sebenarnya: Kurangnya Authorization Check
Prediktabilitas Object ID hanya memudahkan eksploitasi, tetapi bukan penyebab kerentanan IDOR. Kerentanan terjadi karena aplikasi tidak memverifikasi apakah user yang melakukan request memiliki hak akses ke resource dengan ID tersebut. Bahkan dengan UUID yang sepenuhnya random, tanpa authorization check yang proper, aplikasi tetap rentan IDOR.

### Rekomendasi
Solusi efektif adalah menerapkan validasi otorisasi pada setiap endpoint: "Apakah user ini memiliki izin untuk mengakses resource dengan ID ini?". Menggunakan UUID atau identifier yang lebih random tanpa authorization check hanya memberikan false sense of security. Implementasi proper access control adalah kunci mencegah IDOR.
# HA Database Cluster Monitor for Kubernetes (k8s / k3s)

Aplikasi dasbor web berbasis PHP ringan untuk memantau status *real-time* (Health Check) serta pengujian operasi CRUD pada 5 jenis klaster database High Availability (MySQL, MariaDB, PostgreSQL, MSSQL, dan MongoDB) di lingkungan Kubernetes (k3s / k8s).

---

## 📌 Fitur Utama

- **Real-Time Monitoring**: Pengecekan otomatis status koneksi (🟢 Online / 🔴 Offline) setiap 3 detik.
- **Multi-DBMS Support**: Mendukung MySQL Cluster, MariaDB Replication, PostgreSQL Cluster, MSSQL AlwaysOn Availability Groups, dan MongoDB Replica Set.
- **Unified CRUD & Log Terminal**: Pengujian langsung operasi *Create*, *Read*, *Delete*, dan *Clear* serta inspeksi 3 log transaksi terakhir per database.
- **Zero-Build Deployment**: Aplikasi berjalan menggunakan *container* basis `ubuntu:24.04` dengan kode PHP diinjeksi via Kubernetes `ConfigMap` tanpa memerlukan pembuatan Dockerfile / image terpisah.

---

## 📁 Berkas Manifest & Perbedaan Struktur

Repositori ini menyediakan dua opsi struktur manifest deployment:

| Nama Berkas | Deskripsi & Peruntukan |
| :--- | :--- |
| `configmap.yaml` | Menyimpan variabel lingkungan (kredensial DB) dan seluruh kode sumber `index.php`. |
| `deployment.yaml` | Konfigurasi Pod web server (Ubuntu 24.04 + PHP CLI & ekstensi driver DB). |
| `service.yaml` | Menghubungkan Pod ke jaringan luar klaster via `NodePort` (Port `30400`). |
| `all-in-one.yaml` | Gabungan ketiga manifest di atas dalam 1 berkas tunggal untuk instalasi cepat. |

> **Tips Penggunaan:**
> - **Opsi Modular (`configmap.yaml`, `deployment.yaml`, `service.yaml`)**: Direkomendasikan saat pengembangan. Perubahan pada kredensial atau kode PHP di `configmap.yaml` cukup di-apply ulang tanpa perlu menghapus Pod / Service.
> - **Opsi Single (`all-in-one.yaml`)**: Direkomendasikan untuk deployment instan 1 kali perintah (`kubectl apply -f all-in-one.yaml`).

---

## ⚙️ Penjelasan Konfigurasi & Parameter Wajib (Customization)

Sebelum melakukan deployment ke klaster, periksa dan sesuaikan variabel berikut di `configmap.yaml` atau `all-in-one.yaml`:

### 1. Kredensial & Endpoint Host Database (`ConfigMap`)

| Variabel Environment | Default Value / Example | Keterangan & Penyesuaian |
| :--- | :--- | :--- |
| `MYSQL_HOST` | `mysql-cluster-mysql.default.svc.cluster.local` | FQDN / Name Service DNS MySQL di klaster Kubernetes |
| `MYSQL_USER` | `root` | Username administrator MySQL |
| `MYSQL_PASS` | `password_mysql_kamu` | Kata sandi akses database MySQL |
| `MARIADB_HOST` | `mariadb-repl-primary` | Host DNS Service MariaDB Primary (Master) |
| `MARIADB_USER` | `root` | Username MariaDB |
| `MARIADB_PASS` | `Wc3Q~w_7+7rM8(5E` | Kata sandi MariaDB |
| `PG_HOST` | `pg-cluster-rw.default.svc.cluster.local` | Endpoint Service Read-Write PostgreSQL Cluster |
| `PG_USER` | `postgres` atau `app` | Username pengakses PostgreSQL |
| `PG_PASS` | `password_pg_kamu` | Kata sandi PostgreSQL |
| `MSSQL_HOST` | `mssql-ag-listener` | Listener AG (Availability Group) SQL Server |
| `MSSQL_USER` | `sa` | User Sistem Administrator MSSQL |
| `MSSQL_PASS` | `YourStrong!Passw0rd` | Kata sandi SQL Server |
| `MONGO_HOST` | `mongo-cluster-mongodb.default.svc.cluster.local` | Host Service MongoDB Replica Set |
| `MONGO_USER` | `root` | Root user MongoDB |
| `MONGO_PASS` | `password_mongo_kamu` | Kata sandi MongoDB |

### 2. Nama Target Database (`index.php`)
Di dalam berkas `index.php` pada `ConfigMap`, koneksi PDO/MongoDB secara default mengarah ke database `testdb` (atau `app` pada PostgreSQL). Sesuaikan array `$config` jika nama database di klaster Anda berbeda:

```php
$config = [
    'mysql'     => ['dsn' => 'mysql:host=' . getenv('MYSQL_HOST') . ';dbname=NAMA_DB_KAMU', 'user' => getenv('MYSQL_USER'), 'pass' => getenv('MYSQL_PASS')],
    'mariadb'   => ['dsn' => 'mysql:host=' . getenv('MARIADB_HOST') . ';dbname=NAMA_DB_KAMU', 'user' => getenv('MARIADB_USER'), 'pass' => getenv('MARIADB_PASS')],
    'postgres'  => ['dsn' => 'pgsql:host=' . getenv('PG_HOST') . ';dbname=NAMA_DB_KAMU', 'user' => getenv('PG_USER'), 'pass' => getenv('PG_PASS')],
    'sqlserver' => ['dsn' => 'dblib:host=' . getenv('MSSQL_HOST') . ';dbname=NAMA_DB_KAMU', 'user' => getenv('MSSQL_USER'), 'pass' => getenv('MSSQL_PASS')],
    'mongodb'   => ['uri' => 'mongodb://' . getenv('MONGO_USER') . ':' . getenv('MONGO_PASS') . '@' . getenv('MONGO_HOST')]
];

# Panduan Instalasi Redis di Windows (Melalui WSL - Fedora)

Dokumen ini merangkum langkah-langkah yang telah kita lakukan untuk menginstal Redis di lingkungan Windows menggunakan Windows Subsystem for Linux (WSL) dengan distribusi Fedora.

## Prasyarat

*   Windows 10/11
*   WSL sudah terinstal dan diaktifkan.
*   Distribusi Fedora Linux terinstal di WSL.

## Langkah-langkah Instalasi

### Langkah 1: Aktifkan Fitur WSL dan Platform Mesin Virtual (Jika Belum)

Jika Anda belum mengaktifkan fitur-fitur ini, ikuti langkah-langkah berikut di **PowerShell sebagai Administrator**:

1.  Aktifkan Windows Subsystem for Linux:
    ```powershell
    dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
    ```
2.  Aktifkan Virtual Machine Platform:
    ```powershell
    dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
    ```
3.  **Restart komputer Anda** setelah menjalankan kedua perintah di atas.

### Langkah 2: Masuk ke Lingkungan WSL Fedora

Buka Command Prompt atau PowerShell dan ketik:

```bash
wsl
```

Anda akan masuk ke terminal Fedora Anda, ditandai dengan prompt seperti `[user@hostname ~]$` atau `[user@21-1805-LT012 itadmin]$`.

### Langkah 3: Perbarui Sistem dan Instal Redis di Fedora

Di dalam terminal WSL Fedora Anda, jalankan perintah berikut:

1.  **Perbarui daftar paket dan sistem:**
    ```bash
    sudo dnf update -y
    ```
    Perintah ini akan memperbarui semua paket yang terinstal ke versi terbaru.

2.  **Instal Redis:**
    ```bash
    sudo dnf install redis -y
    ```
    Perintah ini akan menginstal paket Redis (yang di Fedora mungkin disebut `valkey` atau `valkey-compat-redis` karena perubahan lisensi, tetapi berfungsi sebagai Redis).

### Langkah 4: Jalankan Server Redis

Karena `systemd` mungkin tidak berjalan sebagai init system di WSL Anda, kita akan menjalankan Redis secara langsung.

1.  **Jalankan server Redis di foreground:**
    ```bash
    redis-server
    ```
    Server Redis akan mulai berjalan dan menampilkan log di terminal ini. **Jangan tutup terminal ini** jika Anda ingin Redis tetap berjalan.

2.  **Verifikasi Redis Berjalan (Buka Terminal WSL Baru):**
    Buka terminal WSL baru (ketik `wsl` lagi di Command Prompt/PowerShell baru) dan jalankan perintah berikut untuk memastikan Redis merespons:
    ```bash
    redis-cli ping
    ```
    Jika berhasil, Anda akan melihat balasan:
    ```
    PONG
    ```
    Anda juga bisa mulai berinteraksi dasar dengan Redis di terminal WSL ini:
    1.  Masuk ke antarmuka `redis-cli`:
        ```bash
        redis-cli
        ```
    2.  Setel sebuah kunci:
        ```
        SET mykey "Hello from Redis!"
        ```
    3.  Dapatkan nilai kunci:
        ```
        GET mykey
        ```
        Output:
        ```
        "Hello from Redis!"
        ```
    4.  Keluar dari `redis-cli`:
        ```
        exit
        ```

### Langkah 5: Verifikasi Koneksi Redis dari Windows (Opsional)

Setelah server Redis berjalan di WSL, Anda dapat memverifikasi koneksi dari Command Prompt (CMD) Windows Anda. Ini berguna untuk memastikan Redis dapat diakses dari aplikasi Windows.

1.  **Dapatkan Alamat IP WSL Anda:**
    Di terminal WSL Anda, jalankan:
    ```bash
    ip addr show eth0 | grep -oP 'inet \K[\d.]+'
    ```
    Cari alamat IP setelah `inet`. Contoh: `inet 172.19.21.31/20`.

2.  **Verifikasi dari Windows CMD:**
    Buka Command Prompt (CMD) di Windows dan gunakan alamat IP WSL yang Anda dapatkan:
    ```cmd
    redis-cli -h <IP_WSL_Anda> -p 6379 ping
    ```
    Contoh:
    ```cmd
    C:\Users\itadmin>redis-cli -h 172.19.21.31 -p 6379 ping
    ```
    Jika berhasil, Anda akan melihat balasan:
    ```
    PONG
    ```
    Ini menandakan bahwa Redis di WSL Anda dapat dijangkau dari Windows.

### Langkah 6: Konfigurasi Redis (Opsional)

Redis memiliki file konfigurasi utama bernama `redis.conf` (atau `valkey.conf` jika Anda menggunakan Valkey) yang memungkinkan Anda menyesuaikan perilakunya.

1.  **Menemukan File Konfigurasi:**
    Secara default, file konfigurasi mungkin berada di `/etc/redis/redis.conf` atau `/etc/valkey/valkey.conf`. Anda dapat mencarinya di terminal WSL Anda:
    ```bash
    sudo ls /etc/redis/redis.conf
    sudo ls /etc/valkey/valkey.conf
    ```
    Jika ditemukan, Anda akan melihat jalur file tersebut.

2.  **Mengubah Pengaturan Jaringan untuk Akses Eksternal:**
    Secara default, Redis mungkin hanya mendengarkan koneksi dari `127.0.0.1` (localhost) dan berjalan dalam `protected-mode`. Untuk mengizinkan koneksi dari Windows atau komputer eksternal lainnya, Anda perlu mengubah dua pengaturan ini di file konfigurasi Anda (`Redish/redis.conf`).

    *   Buka file konfigurasi Anda menggunakan editor teks (misalnya `nano`) di WSL:
        ```bash
        sudo nano /mnt/d/Belajar\ Web/Redis/Rekayasa-Konteks-Gemini-main/Redish/redis.conf
        ```
    *   **Ubah `bind` Address:**
        Cari baris yang dimulai dengan `bind`. Biasanya terlihat seperti:
        ```
        bind 127.0.0.1 -::1
        ```
        Ubah baris tersebut menjadi `bind 0.0.0.0 -::1` (untuk mendengarkan semua antarmuka) atau komentari baris tersebut dengan menambahkan `#` di depannya:
        ```
        bind 0.0.0.0 -::1
        # ATAU
        # bind 127.0.0.1 -::1
        ```
        *Penjelasan:* Mengubah `bind` ke `0.0.0.0` atau mengomentarinya akan membuat Redis mendengarkan koneksi dari semua alamat IP yang tersedia di WSL Anda, termasuk yang dapat diakses dari Windows.

    *   **Nonaktifkan `protected-mode`:**
        Cari baris `protected-mode yes` dan ubah menjadi `protected-mode no`:
        ```
        protected-mode no
        ```
        *Penjelasan:* `protected-mode` adalah fitur keamanan yang mencegah koneksi dari luar `localhost` jika tidak ada kata sandi yang diatur. Menonaktifkannya diperlukan untuk akses eksternal dalam skenario pengembangan ini. **Untuk lingkungan produksi, sangat disarankan untuk mengatur kata sandi (`requirepass`) daripada menonaktifkan `protected-mode` tanpa otentikasi.**

    *   Simpan perubahan (di `nano`: `Ctrl+X`, `Y`, `Enter`).

3.  **Memulai Redis dengan File Konfigurasi Kustom:**
    Jika Anda memiliki file `redis.conf` kustom (misalnya, `Redish/redis.conf`), Anda dapat memberi tahu Redis untuk menggunakannya saat startup.
    *   Pastikan server Redis yang sedang berjalan dihentikan.
    *   Mulai Redis dengan menentukan jalur lengkap ke file konfigurasi Anda:
        ```bash
        redis-server /mnt/d/Belajar\ Web/Redis/Rekayasa-Konteks-Gemini-main/Redish/redis.conf
        ```
        (Sesuaikan jalur jika file Anda berada di lokasi lain).

4.  **Penting: Restart Server Redis:**
    Setelah Anda membuat perubahan pada file konfigurasi, Anda **harus me-restart** server Redis agar perubahan tersebut diterapkan. Jika Anda menjalankannya di foreground, cukup tutup terminal dan jalankan kembali perintah `redis-server` (dengan atau tanpa jalur konfigurasi kustom).

5.  **Verifikasi Konfigurasi yang Dimuat:**
    Untuk memastikan Redis menggunakan file konfigurasi yang benar, Anda dapat menggunakan `redis-cli INFO`.
    *   Buka terminal WSL baru dan jalankan:
        ```bash
        redis-cli
        INFO Server
        ```
    *   Cari baris `config_file` di output. Ini akan menunjukkan jalur lengkap ke file konfigurasi yang sedang digunakan oleh instance Redis yang terhubung.

### Langkah 8: Interaksi Dasar dengan Redis (Contoh Praktis)

Setelah Redis berjalan dan dapat diakses, Anda dapat mulai berinteraksi dengannya menggunakan `redis-cli`. Berikut adalah contoh perintah dasar untuk mengelola data string:

1.  **Menyetel nilai string pada sebuah kunci (`SET`):**
    ```
    localhost:6379> set test "ini isi test"
    OK
    ```

2.  **Mendapatkan nilai dari kunci (`GET`):**
    ```
    localhost:6379> get test
    "ini isi test"
    ```

3.  **Memeriksa keberadaan kunci (`EXISTS`):**
    ```
    localhost:6379> exists test
    (integer) 1
    ```

4.  **Menambahkan nilai ke string yang sudah ada (`APPEND`):**
    ```
    localhost:6379> append test "add more data"
    (integer) 25
    ```

5.  **Melihat nilai kunci setelah `APPEND`:**
    ```
    localhost:6379> get test
    "ini isi testadd more data"
    ```

6.  **Mencari semua kunci yang ada (`KEYS *`):**
    ```
    localhost:6379> keys *
    1) "test"
    ```

7.  **Mencari kunci dengan pola tertentu (`KEYS test*`):**
    ```
    localhost:6379> keys test*
    1) "test"
    ```

8.  **Menghapus kunci (`DEL`):**
    ```
    localhost:6379> del test
    (integer) 1
    ```

9.  **Memverifikasi kunci telah dihapus (`GET`):**
    ```
    localhost:6379> get test
    (nil)
    ```

### Langkah 9: Manipulasi String Lanjutan (`SETRANGE`, `GETRANGE`)

Redis juga menyediakan perintah untuk memanipulasi bagian dari nilai string. Berikut adalah contoh penggunaan `SETRANGE` dan `GETRANGE`:

1.  **Menyetel nilai awal untuk kunci (`SET`):**
    ```
    localhost:6379> set eko "Eko Kurniawan"
    OK
    ```

2.  **Mendapatkan nilai kunci untuk verifikasi (`GET`):**
    ```
    localhost:6379> get eko
    "Eko Kurniawan"
    ```

3.  **Mengganti bagian dari string dengan `SETRANGE`:**
    `SETRANGE <key> <offset> <value>` menimpa bagian dari string dimulai dari `offset`.
    ```
    localhost:6379> setrange eko 4 "Kurniawan Khannedy"
    (integer) 22
    ```
    *Penjelasan:* String asli "Eko Kurniawan" (panjang 15). Dimulai dari indeks 4 (karakter 'K' dari 'Kurniawan'), string diganti dengan "Kurniawan Khannedy". Panjang string baru menjadi 22.

4.  **Melihat hasil setelah `SETRANGE` (`GET`):**
    ```
    localhost:6379> get eko
    "Eko Kurniawan Khannedy"
    ```

5.  **Mendapatkan substring dengan `GETRANGE`:**
    `GETRANGE <key> <start> <end>` mengembalikan substring dari indeks `start` hingga `end` (inklusif).
    ```
    localhost:6379> getrange eko 4 12
    "Kurniawan"
    ```
    *Penjelasan:* Mengambil substring dari indeks 4 hingga 12 dari "Eko Kurniawan Khannedy", yang menghasilkan "Kurniawan".

### Langkah 9: Operasi Batch pada String (`MSET`, `MGET`)

Redis memungkinkan Anda untuk mengatur dan mendapatkan banyak kunci string sekaligus dalam satu operasi, yang sangat efisien untuk mengurangi *round-trip time*.

1.  **Menyetel banyak kunci sekaligus (`MSET`):**
    ```
    localhost:6379> mset budi "100" joko "200" rully "300"
    OK
    ```

2.  **Mendapatkan nilai dari banyak kunci sekaligus (`MGET`):**
    ```
    localhost:6379> mget budi joko rully
    1) "100"
    2) "200"
    3) "300"
    ```

3.  **Contoh `MGET` dengan kunci yang sudah ada:**
    Anda juga dapat menyertakan kunci yang sudah ada dalam perintah `MGET`.
    ```
    172.19.21.31:6379[1]> mget budi joko rully eko
    1) "100"
    2) "200"
    3) "300"
    4) "Eko Kurniawan Khannedy"
    ```

### Pemecahan Masalah Umum:

*   **`Failed to write PID file: Permission denied`**:
    Pesan ini muncul jika `redis-server` dijalankan tanpa `sudo` dan mencoba menulis file PID di lokasi yang memerlukan izin root. Solusinya adalah menjalankan `redis-server` dengan `sudo` (misalnya, `sudo redis-server /path/to/your/redis.conf`).

*   **`WARNING Memory overcommit must be enabled!`**:
    Peringatan ini dapat diatasi sementara dengan `sudo sysctl vm.overcommit_memory=1` di WSL. Untuk membuatnya persisten setelah reboot, tambahkan `vm.overcommit_memory = 1` ke `/etc/sysctl.conf` dan jalankan `sudo sysctl -p`.

*   **Kesalahan Koneksi dari Windows CMD (`No connection could be made...` atau `DENIED Running in protected mode...`)**:
    Kesalahan ini biasanya disebabkan oleh konfigurasi jaringan atau mode keamanan Redis. Pastikan Anda telah melakukan hal berikut di file konfigurasi Redis (`Redish/redis.conf`) dan **me-restart server Redis di WSL setelah setiap perubahan**:
    1.  **Ubah `bind` address**: Pastikan `bind` diatur ke `0.0.0.0 -::1` atau dikomentari (`# bind 127.0.0.1 -::1`).
    2.  **Nonaktifkan `protected-mode`**: Pastikan `protected-mode` diatur ke `no`.
    3.  Pastikan server Redis di WSL Anda **sedang berjalan**.

*   **Verifikasi Startup Redis yang Berhasil**:
    Ketika Redis berhasil dimulai dengan konfigurasi yang benar dan tanpa masalah, Anda akan melihat output di terminal WSL yang diakhiri dengan `Ready to accept connections tcp`. Ini menandakan server siap digunakan.


---

**Catatan Penting:**

*   Jika Anda menjalankan Redis di foreground, server akan berhenti jika terminal ditutup. Untuk menjalankan di latar belakang (sebagai daemon), Anda perlu mengedit `redis.conf` dan mengubah `daemonize no` menjadi `daemonize yes`, lalu jalankan `redis-server /path/to/redis.conf`.

### Langkah 10: Praktik Caching (Pelajaran Kita)

Di bagian ini, kita akan mendokumentasikan simulasi praktis yang telah kita lakukan. Ini adalah salah satu kegunaan paling umum dari Redis: sebagai *cache* (penyimpanan sementara) untuk mempercepat aplikasi.

**Konsep dasarnya sederhana:** Daripada selalu bertanya ke database yang lambat, aplikasi akan bertanya ke Redis terlebih dahulu.

Mari kita rangkum alur kerja yang telah kita praktikkan.

#### Tahap 1: "Cache Miss" - Saat Data Tidak Ada di Cache

Ini adalah kondisi awal. Aplikasi butuh data, tetapi cache masih kosong.

1.  **Aplikasi Bertanya ke Redis:** Kita mencoba mengambil data untuk kunci `artikel:123`.
    ```
    172.19.21.31:6379> GET artikel:123
    (nil)
    ```
    *   **Artinya:** Hasil `(nil)` berarti "tidak ada". Ini adalah **Cache Miss**.

2.  **Aplikasi Bekerja Keras:** Karena data tidak ada di cache, aplikasi terpaksa mengambilnya dari database utama (proses yang lambat).

3.  **Aplikasi Menyimpan ke Cache:** Setelah mendapatkan data, aplikasi menyimpannya ke Redis agar permintaan berikutnya lebih cepat. Kita menggunakan `SETEX` untuk menyimpan data sekaligus memberinya "timer" kedaluwarsa 15 detik.
    ```
    172.19.21.31:6379> SETEX artikel:123 15 "Ini adalah isi artikel penting dari database."
    OK
    ```
    *   **Artinya:** Data sekarang aman di dalam cache Redis.

#### Tahap 2: "Cache Hit" - Kemenangan Caching!

Sekarang, sebelum 15 detik berlalu, ada permintaan lagi untuk data yang sama.

1.  **Aplikasi Bertanya ke Redis (Lagi):**
    ```
    172.19.21.31:6379> GET artikel:123
    "Ini adalah isi artikel penting dari database."
    ```
    *   **Artinya:** Berhasil! Ini adalah **Cache Hit**. Data langsung didapat dari Redis tanpa perlu menyentuh database yang lambat. Inilah tujuan utama dari caching.

#### Tahap 3: "Expiration" - Saat Data Menjadi Basi

Data tidak bisa selamanya di cache, ia harus diperbarui. Itulah gunanya "timer" yang kita atur.

1.  **Melihat Sisa Waktu (TTL):** Kita bisa mengintip sisa waktu hidup kunci tersebut.
    ```
    172.19.21.31:6379> TTL artikel:123
    (integer) 12
    ```
    *   **Artinya:** Sisa waktu hidup kunci ini adalah 12 detik.

2.  **Kunci Menghilang:** Setelah waktu habis, Redis secara otomatis menghapus kunci tersebut. Jika kita memeriksa TTL-nya lagi, Redis akan memberitahu kita bahwa kuncinya sudah tidak ada.
    ```
    172.19.21.31:6379> TTL artikel:123
    (integer) -2
    ```
    *   **Artinya:** Kode `-2` dari `TTL` berarti "kunci tidak ada".

3.  **Pembuktian Akhir:** Jika kita mencoba mengambil datanya lagi, kita akan kembali ke titik awal.
    ```
    172.19.21.31:6379> GET artikel:123
    (nil)
    ```
    *   **Artinya:** Kita kembali mengalami **Cache Miss**. Siklus akan berulang dari Tahap 1.

Pelajaran ini menunjukkan seluruh siklus hidup data di dalam cache, dari dibuat, digunakan, hingga akhirnya dihapus.

### Langkah 11: Menghindari Race Condition dengan Operasi Atomik (INCR, DECR)

Ini adalah salah satu pelajaran paling krusial dalam menggunakan Redis secara aman. Kita akan membahas masalah **Race Condition** yang muncul dari contoh kode ini:

```javascript
// race-condition.js - CONTOH YANG SALAH!
let value = await redis.get("key");
value = Number(value) + 1;
await redis.set("key", value);
```

**Masalahnya:** Proses "ambil, ubah, simpan" ini tidak terjadi dalam sekejap. Ada jeda waktu, dan di situlah letak bahayanya jika ada banyak pengguna.

**Skenario Bencana (Race Condition):**
Bayangkan ada dua pengunjung, A dan B, menaikkan penghitung `visitor_count` yang nilainya `100`.
1.  Pengunjung A `GET` nilai -> mendapat `100`.
2.  Pengunjung B `GET` nilai -> juga mendapat `100` (karena A belum selesai).
3.  Pengunjung A menghitung `100 + 1` dan `SET` nilainya menjadi `101`.
4.  Pengunjung B juga menghitung `100 + 1` dan `SET` nilainya menjadi `101`.

**Hasil Akhir:** `visitor_count` menjadi `101`.
**Hasil yang Seharusnya:** `102`. Satu penambahan telah hilang!

#### Solusi: Perintah Atomik Redis

Redis menyediakan perintah yang bersifat **atomik**, artinya seluruh operasi (ambil, ubah, simpan) dijamin selesai tanpa bisa diinterupsi.

Mari kita praktikkan perbedaannya.

**1. Siapkan Penghitung Awal**
```
172.19.21.31:6379> SET visitor_count 100
OK
```

**2. Cara yang Salah (Manual)**
Ini adalah cara yang kita simulasikan sebagai "tidak aman".
```
172.19.21.31:6379> GET visitor_count
"100"
172.19.21.31:6379> SET visitor_count 101
OK
```
*   **Artinya:** Berhasil, tetapi tidak aman karena ada jeda waktu antara `GET` dan `SET`.

**3. Cara yang Benar dan Aman (Atomik)**
Gunakan perintah `INCR` yang dirancang khusus untuk ini.
```
172.19.21.31:6379> INCR visitor_count
(integer) 102
```
*   **Artinya:** Redis langsung menaikkan nilainya dan mengembalikan hasil **setelah** operasi selesai. Ini 100% aman. Setiap perintah `INCR` berikutnya akan menaikkan nilainya dengan benar (`103`, `104`, dst.).

**4. Keluarga Perintah Atomik Lainnya**
*   **Menurunkan nilai:** `DECR visitor_count`
*   **Menambah dalam jumlah banyak:** `INCRBY visitor_count 5`
*   **Mengurangi dalam jumlah banyak:** `DECRBY visitor_count 5`

**Aturan Emas:**
Untuk operasi hitung-menghitung (counters, likes, views, inventory), **JANGAN PERNAH** menggunakan pola `GET-Ubah-SET` manual. **SELALU** gunakan `INCR`, `DECR`, `INCRBY`, atau `DECRBY`.

### Langkah 12: Implementasi Penghitung Pengunjung Website (Praktik INCR/DECR)

Mari kita terapkan pemahaman kita tentang operasi atomik dengan membuat sebuah **penghitung pengunjung website** sederhana. Ini adalah skenario umum di mana `INCR` dan `DECR` sangat dibutuhkan untuk menjaga konsistensi data.

**Skenario:** Kita ingin menghitung jumlah pengunjung aktif di website kita. Setiap kali pengunjung datang, penghitung bertambah. Setiap kali pengunjung pergi, penghitung berkurang.

**Prasyarat:**
*   Pastikan server Redis Anda berjalan.
*   Pastikan Anda berada di `redis-cli`.

#### 1. Inisialisasi Penghitung Pengunjung

Kita akan mengatur nilai awal untuk penghitung kita.

```
172.19.21.31:6379> SET website_visitors 0
OK
172.19.21.31:6379> GET website_visitors
"0"
```
*   **Penjelasan:** Kita memulai penghitung `website_visitors` dari `0`.

#### 2. Simulasikan Pengunjung Baru (Menggunakan `INCR`)

Setiap kali ada pengunjung baru datang, kita akan menaikkan penghitungnya.

```
172.19.21.31:6379> INCR website_visitors
(integer) 1
172.19.21.31:6379> INCR website_visitors
(integer) 2
172.19.21.31:6379> INCR website_visitors
(integer) 3
```
*   **Penjelasan:** Perintah `INCR` secara atomik menaikkan nilai kunci sebesar 1. Perhatikan bagaimana Redis langsung mengembalikan nilai terbaru setelah penambahan.

#### 3. Simulasikan Kedatangan Banyak Pengunjung Sekaligus (Menggunakan `INCRBY`)

Jika ada beberapa pengunjung datang bersamaan, kita bisa menaikkan penghitung dalam jumlah tertentu.

```
172.19.21.31:6379> INCRBY website_visitors 5
(integer) 8
172.19.21.31:6379> GET website_visitors
"8"
```
*   **Penjelasan:** `INCRBY` menaikkan nilai kunci sebesar jumlah yang ditentukan (dalam contoh ini, 5).

#### 4. Simulasikan Pengunjung Meninggalkan Website (Menggunakan `DECR` dan `DECRBY`)

Untuk mengurangi jumlah pengunjung, kita gunakan `DECR` atau `DECRBY`.

```
172.19.21.31:6379> DECR website_visitors
(integer) 7
172.19.21.31:6379> DECRBY website_visitors 2
(integer) 5
172.19.21.31:6379> GET website_visitors
"5"
```
*   **Penjelasan:** `DECR` mengurangi nilai sebesar 1, dan `DECRBY` mengurangi nilai sebesar jumlah yang ditentukan (dalam contoh ini, 2).

**Kesimpulan Implementasi:**
Anda telah berhasil mengimplementasikan penghitung yang aman dari *race condition* menggunakan perintah atomik Redis. Ini adalah fondasi penting untuk banyak fitur aplikasi yang membutuhkan penghitungan yang akurat.

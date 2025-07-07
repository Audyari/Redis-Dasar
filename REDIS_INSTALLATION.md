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

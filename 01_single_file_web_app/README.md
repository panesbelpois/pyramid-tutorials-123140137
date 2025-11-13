Nama : Anisah Octa Rohila
NIM : 123140137
Kelas : Pengembangan Aplikasi Web RA

# 01: Aplikasi Web Single-File

## Penjelasan Singkat

Apa cara paling sederhana untuk memulai dengan Pyramid? Modul single-file. Tidak perlu paket Python, tidak perlu `pip install -e .`, tidak perlu mesin lainnya.

## Latar Belakang

Microframework dahulu sangat populer, hingga hal baru yang berkilau muncul. "Microframework" adalah istilah pemasaran, bukan istilah teknis. Mereka memiliki overhead mental yang rendah: mereka melakukan sangat sedikit, sehingga satu-satunya hal yang perlu Anda khawatirkan adalah hal-hal Anda sendiri.

Pyramid istimewa karena dapat bertindak sebagai microframework modul single-file. Anda dapat memiliki satu file Python yang dapat dieksekusi langsung oleh Python. Tetapi Pyramid juga menyediakan fasilitas untuk penskalaan hingga aplikasi terbesar sekalipun.

Python memiliki standar bernama WSGI yang mendefinisikan bagaimana aplikasi web Python terhubung ke server standar, menerima permintaan masuk, dan mengembalikan respons. Sebagian besar kerangka web Python modern mematuhi pola aplikasi "MVC" (model-view-controller), di mana data dalam model memiliki tampilan yang memediasi interaksi dengan sistem luar.

Dalam langkah ini kami akan melihat sekilas server WSGI, aplikasi WSGI, permintaan, respons, dan tampilan (views).

## Tujuan

1. Mendapatkan aplikasi web Pyramid yang berjalan, sesederhana mungkin.
2. Menggunakan itu sebagai dasar yang dipahami dengan baik untuk menambahkan setiap unit kompleksitas.
3. Paparan awal terhadap aplikasi WSGI, permintaan, tampilan, dan respons.

## Instalasi (Windows PowerShell)

### Langkah 1: Buat dan Aktifkan Virtual Environment

```powershell
python -m venv .venv
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

### Langkah 2: Pasang Dependensi

```powershell
pip install pyramid waitress
```

### Langkah 3: Jalankan Aplikasi

```powershell
.\.venv\Scripts\python.exe app.py
```

Setelah itu, buka browser Anda dan kunjungi `http://localhost:6543/` untuk melihat pesan "Hello World!".

**Untuk pengguna Unix/macOS/Linux:**

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install pyramid waitress
python app.py
```

## Penjelasan Kode

File `app.py` berisi kode berikut:

```python
from waitress import serve
from pyramid.config import Configurator
from pyramid.response import Response


def hello_world(request):
    print('Incoming request')
    return Response('<body><h1>Hello World!</h1></body>')


if __name__ == '__main__':
    with Configurator() as config:
        config.add_route('hello', '/')
        config.add_view(hello_world, route_name='hello')
        app = config.make_wsgi_app()
    serve(app, host='0.0.0.0', port=6543)
```

### Poin-Poin Penting:

- **Fungsi `hello_world(request)`**: Ini adalah "view" yang dipetakan ke rute `/`. Ia menerima objek `request` dan mengembalikan `Response` berisi HTML.
- **Blok `if __name__ == '__main__':`**: Memastikan konfigurasi dan server hanya dijalankan saat modul dieksekusi langsung dari baris perintah, bukan saat modul di-import oleh modul lain.
- **`Configurator()`**: Digunakan untuk mengatur rute (`add_route`) dan menghubungkan view ke rute tersebut (`add_view`). Setelah selesai, `make_wsgi_app()` menghasilkan aplikasi WSGI.
- **`serve(app, host='0.0.0.0', port=6543)`**: Menjalankan server Waitress yang menerima koneksi di host `0.0.0.0` dan port `6543`.

## Analisis

### Alur Aplikasi

1. **Konfigurasi**: `Configurator` membangun pemetaan antara URL (rute) → view → Response.
2. **Server WSGI**: Waitress mendengarkan permintaan HTTP dan meneruskannya ke aplikasi.
3. **Pemrosesan Permintaan**: Ketika browser mengakses `/`, rute cocok dengan `hello_world` view.
4. **Respons**: View mengembalikan HTML yang dibungkus dalam objek `Response`, yang dikirim kembali ke browser.

### Kasus Khusus:

**Mengembalikan string HTML biasa:**
- Jika Anda mengembalikan string HTML langsung tanpa membungkusnya dalam `Response`, Pyramid mungkin memerlukan renderer yang dikonfigurasi khusus atau akan menganggap ini sebagai kesalahan. Cara terbaik adalah selalu gunakan `Response()` untuk kontrol eksplisit.

**Mengembalikan sequence of integers:**
- WSGI mengharapkan respons berupa bytes atau iterable yang menghasilkan bytes/strings. Mengembalikan list atau tuple integer tidak valid untuk WSGI dan akan menyebabkan TypeError atau respons yang tidak valid.

**Error di dalam view (contoh: `print xyz`):**
- Saat view menjalankan kode yang tidak valid, Python akan melempar exception. Server akan menangkap ini dan mengembalikan HTTP 500 (Internal Server Error) untuk permintaan itu.
- Stacktrace akan terlihat di konsol tempat Anda menjalankan `python app.py`, sangat berguna untuk debugging.
- Setelah memperbaiki kode dan me-restart server, aplikasi akan kembali bekerja normal.

## Jawaban Extra Credit

### 1. Mengapa Menggunakan `print('Incoming request')` Bukan `print 'Incoming request'`?

**Jawaban:**
- Bentuk `print('...')` dengan tanda kurung adalah sintaks Python 3.
- Bentuk `print '...'` tanpa tanda kurung adalah sintaks Python 2 (sudah usang).
- Python 3 adalah standar modern; kode yang ditulis saat ini harus menggunakan `print()` dengan tanda kurung.

### 2. Apa yang Terjadi Jika Anda Mengembalikan String HTML? Atau Sequence of Integers?

**Jawaban:**

**String HTML Biasa:**
- Mengembalikan string HTML langsung tanpa `Response()` dapat bekerja dengan renderer tertentu, tetapi tidak konsisten.
- Praktik terbaik: selalu bungkus output dalam `Response()` untuk kontrol eksplisit atas tipe konten, header, dan encoding.
- Contoh: `return Response('<h1>Halo</h1>', content_type='text/html')`

**Sequence of Integers:**
- WSGI secara teknis menerima iterable yang menghasilkan chunks bytes (string bytes).
- Mengembalikan list/tuple integer akan menghasilkan error atau respons yang tidak valid karena WSGI tidak tahu cara mengubah integer menjadi bytes.
- Ini adalah pelanggaran kontrak WSGI dan akan menyebabkan kesalahan pada saat runtime.

### 3. Jika Anda Memasukkan Kode Tidak Valid (Contoh: `print xyz`), Apa yang Terjadi?

**Jawaban:**
- Ketika permintaan HTTP tiba dan view `hello_world` dipanggil, Python mengeksekusi kode.
- Jika ada kesalahan (seperti `NameError: name 'xyz' is not defined`), exception dilempar.
- Server WSGI menangkap exception ini dan mengembalikan respons HTTP 500 (Internal Server Error) ke klien.
- **Di sisi server (konsol tempat `python app.py` dijalankan)**: Anda akan melihat stacktrace lengkap, menunjukkan file, nomor baris, dan jenis error.
- **Langkah recovery**: Hentikan server (Ctrl+C), perbaiki kode di `app.py`, lalu jalankan `python app.py` lagi untuk me-restart.
- Setelah itu, permintaan baru akan berjalan dengan kode yang sudah diperbaiki.

### 4. "GI" pada WSGI Adalah "Gateway Interface" — Standar Apa?

**Jawaban:**
- WSGI adalah singkatan dari "Web Server Gateway Interface".
- **GI** = "Gateway Interface" (Antarmuka Gateway).
- Standar WSGI ini terinspirasi dari standar lama bernama **CGI** (Common Gateway Interface), yang telah ada sejak era awal web.
- **CGI** mendefinisikan bagaimana server web mengeksekusi program eksternal (seperti script Perl, PHP, atau Python) dan meneruskan data HTTP kepada mereka.
- **WSGI** adalah evolusi modern dari CGI untuk lingkungan Python, memungkinkan aplikasi web Python terhubung dengan server web standar (seperti Apache dengan mod_wsgi, Nginx dengan Gunicorn, atau standalone server seperti Waitress).

## Catatan Cepat

- File `app.py` di folder ini sudah siap dijalankan setelah Anda memasang `pyramid` dan `waitress`.
- Jika ingin mengubah port, edit baris `serve(app, host='0.0.0.0', port=6543)` menjadi port yang Anda inginkan.
- Untuk debugging lebih lanjut, Anda bisa menambahkan `print()` statement di view atau menggunakan debugger Python.
- Dokumentasi Pyramid: https://docs.pylonsproject.org/projects/pyramid/

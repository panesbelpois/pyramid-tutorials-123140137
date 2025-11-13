# 04: Easier Development with debugtoolbar

## Deskripsi singkat

pyramid_debugtoolbar adalah add-on untuk Pyramid yang menampilkan toolbar debugging di browser saat aplikasi dijalankan dalam mode development. Toolbar ini menyediakan traceback interaktif, inspeksi variabel, dan panel tambahan (mis. SQL) yang membantu menemukan dan menganalisis bug tanpa perlu mengubah kode aplikasi.

## Instalasi

- Tambahkan `pyramid_debugtoolbar` ke daftar dependensi development di `debugtoolbar/setup.py` (extras_require 'dev'):

```py
dev_requires = [
	'pyramid_debugtoolbar',
]
extras_require = {
	'dev': dev_requires,
}
```

- Buat virtual environment dan pasang paket development (PowerShell):

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -e ".[dev]"
```

- Aktifkan add-on lewat file konfigurasi `development.ini` dengan menambahkan `pyramid_debugtoolbar` ke `pyramid.includes`:

```
[app:main]
use = egg:tutorial
pyramid.includes =
	pyramid_debugtoolbar

[server:main]
use = egg:waitress#main
listen = localhost:6543
```

- Jalankan aplikasi dengan autoreload dan buka `http://localhost:6543/`:

```powershell
pserve development.ini --reload
```

## Analisis

- Cara kerja: `pyramid_debugtoolbar` adalah paket PyPI yang menyertakan konfigurasi Pyramid; konfigurasi ini bisa di-include lewat kode (`config.include(...)`) atau lewat `.ini` (`pyramid.includes`). Memasukkannya lewat `.ini` memudahkan pengaktifan/penonaktifan berdasarkan environment.
- Mengapa extras_require 'dev': Menaruh toolbar di `extras_require['dev']` membuatnya hanya terpasang di lingkungan pengembangan (menghindari memasukkan tool debug ke produksi).
- Efek samping: toolbar menyuntikkan sedikit HTML/CSS sebelum penutup `</body>` untuk menampilkan UI-nya. Jika terjadi keanehan di sisi klien, nonaktifkan dengan menghapus atau mengomentari `pyramid_debugtoolbar` di `pyramid.includes`.
- Debug interaktif: saat error (mis. karena typo `xResponse`), toolbar menampilkan traceback interaktif; klik ikon layar untuk membuka konsol interaktif yang bisa mengeksplorasi variabel lokal/global.

## Contoh latihan

- Ubah fungsi `hello_world` dari:

```py
def hello_world(request):
	return Response('<body><h1>Hello World!</h1></body>')
```

ke:

```py
def hello_world(request):
	return xResponse('<body><h1>Hello World!</h1></body>')
```

Lalu buka `http://localhost:6543/` dan perhatikan traceback interaktif di toolbar.

## Extra credit

- **Alasan menaruh `pyramid_debugtoolbar` di `extras_require['dev']`:**
	- Memisahkan dependensi development dari dependensi runtime produksi. Dengan meletakkannya di `dev`, paket debug hanya terpasang saat pengembang secara eksplisit menjalankan `pip install -e ".[dev]"`.
	- Mengurangi ukuran dan potensi permukaan serangan di lingkungan produksi. Toolbar menyediakan fitur interaktif (konsol, introspeksi) yang sebaiknya tidak tersedia pada server publik.
	- Memudahkan pengelolaan lingkungan: tim dapat memiliki kumpulan dependensi yang berbeda untuk `dev`, `test`, dan `prod` tanpa mengubah kode aplikasi.
	- Praktis untuk deployment: file konfigurasi (`.ini`) dapat mengaktifkan atau menonaktifkan add-on tanpa mengubah source code, sehingga mekanisme extras + konfigurasi `.ini` memberi fleksibilitas operasional.

Ringkasnya, menaruh `pyramid_debugtoolbar` di extras 'dev' menjaga lingkungan produksi tetap bersih dan aman, sambil memudahkan pengembang mengaktifkan tool ketika mereka membutuhkannya.
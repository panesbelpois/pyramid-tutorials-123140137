## 02: Python Packages untuk Aplikasi Pyramid

### Penjelasan Singkat

Pengembangan Python modern umumnya dilakukan menggunakan paket Python. Pyramid mendukung cara ini dengan baik. Pada langkah ini kita akan mengubah "Hello World" menjadi paket Python minimal di dalam proyek Python minimal.

### Latar Belakang

Pengembang Python dapat mengorganisasikan koleksi modul dan file menjadi unit bernama paket. Jika sebuah direktori ada di `sys.path` dan memiliki file khusus bernama `__init__.py`, direktori tersebut dianggap sebagai paket Python.

Paket dapat dikemas, dibuat tersedia untuk instalasi, dan dipasang melalui toolchain yang berpusat pada file `setup.py`. Untuk tujuan tutorial ini, itu saja yang perlu diketahui:

- Kita akan memiliki direktori untuk setiap langkah tutorial sebagai proyek.
- Proyek ini akan berisi `setup.py` yang menyuntikkan fitur proyek ke dalam direktori.
- Dalam proyek ini kita akan membuat subdirektori `tutorial` menjadi paket Python menggunakan file modul `__init__.py`.
- Kita akan menjalankan `pip install -e .` untuk memasang proyek dalam mode development (editable).

Singkatnya: Anda akan mengembangkan dalam paket Python, dan paket itu menjadi bagian dari proyek.

### Tujuan

1. Membuat direktori paket Python dengan `__init__.py`.
2. Menyediakan file `setup.py` untuk menjadikan direktori sebagai proyek Python.
3. Menginstal proyek tutorial dalam mode development.

### Langkah-Langkah (contoh untuk PowerShell/Windows)

1. Buat area proyek (ada di struktur repo ini sebagai `package/` sudah):

```powershell
cd ..; mkdir package; cd package
```

2. Isi `package/setup.py` dengan konten berikut:

```python
from setuptools import setup

# List of dependencies installed via `pip install -e .`
# by virtue of the Setuptools `install_requires` value below.
requires = [
	'pyramid',
	'waitress',
]

setup(
	name='tutorial',
	install_requires=requires,
)
```

3. Pasang proyek pada mode development (editable):

```powershell
.\.venv\Scripts\pip.exe install -e .
```

4. Buat direktori paket actual dan file modul:

```powershell
mkdir tutorial
New-Item tutorial\__init__.py -ItemType File -Value "# package"
```

5. Isi `package/tutorial/app.py` dengan kode aplikasi (sama seperti single-file):

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

6. Jalankan aplikasi:

```powershell
.\.venv\Scripts\python.exe tutorial\app.py
```

Kemudian buka `http://localhost:6543/`.

### Analisis

Paket Python memberi kita unit pekerjaan yang terorganisir. Berkas `setup.py` memberikan fitur tambahan saat paket dipasang — dalam contoh ini kita hanya menggunakan `install_requires` yang membuat dependensi dipasang ketika pengguna memasang paket.

Catatan: Menjalankan modul di dalam paket secara langsung dengan `python tutorial/app.py` agak tidak biasa untuk produksi; ini dilakukan di tutorial ini hanya untuk tujuan pembelajaran.

### Catatan Tambahan

- Struktur proyek di folder ini sudah memuat `package/setup.py` dan folder `tutorial/`.
- Jika ingin mengubah nama paket atau menambahkan metadata, edit `setup.py` sesuai kebutuhan.

## Penutup

Dokumentasi ini menyediakan dua pendekatan: menjalankan Pyramid sebagai modul single-file, dan mengemas aplikasi ke dalam paket Python yang dapat di-install secara editable untuk pengembangan. Gunakan single-file untuk eksperimen cepat, dan paket untuk pengembangan proyek yang lebih terstruktur.
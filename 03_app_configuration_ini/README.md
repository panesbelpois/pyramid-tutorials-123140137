# 03: Konfigurasi Aplikasi dengan File .ini
---

## Penjelasan Umum

Dalam tutorial ini, kita akan mempelajari cara menggunakan perintah `pserve` Pyramid dengan file konfigurasi `.ini` untuk menjalankan aplikasi dengan lebih sederhana dan lebih baik. Pyramid memiliki konsep konfigurasi tingkat pertama yang terpisah dari kode, membedakannya dari framework Python web lainnya.

### Latar Belakang

Pyramid menggunakan pendekatan berbasis **entry points** dari Python Setuptools untuk menghubungkan WSGI app dengan aplikasi. Hal ini memungkinkan:

- **Pemisahan konfigurasi dari kode**: Konfigurasi aplikasi disimpan di file `.ini` yang terpisah
- **Fleksibilitas**: Mudah mengganti konfigurasi tanpa mengubah kode Python
- **Standar industri**: Mengikuti konvensi Python dan Setuptools yang sudah mapan

---

## Tujuan Pembelajaran

1. ✅ Memodifikasi `setup.py` untuk menambahkan entry point yang memberitahu Pyramid lokasi WSGI app
2. ✅ Membuat aplikasi yang didorong oleh file `.ini`
3. ✅ Menjalankan aplikasi menggunakan perintah `pserve` Pyramid
4. ✅ Memindahkan kode startup ke file `__init__.py` package

---

## Langkah Instalasi dan Konfigurasi

### Langkah 1: Menyiapkan Struktur Direktori

```bash
cd ..
cp -r package ini
cd ini
```

Perintah ini meng-copy hasil dari tutorial sebelumnya ke direktori `ini`.

### Langkah 2: Konfigurasi setup.py

File `setup.py` harus dimodifikasi untuk menambahkan entry point. Entry point ini memberitahu Pyramid di mana menemukan WSGI app factory:

```python
from setuptools import setup

# Daftar dependensi yang dipasang via `pip install -e .`
# oleh virtue of the Setuptools `install_requires` value di bawah.
requires = [
    'pyramid',
    'waitress',
]

setup(
    name='tutorial',
    install_requires=requires,
    entry_points={
        'paste.app_factory': [
            'main = tutorial:main'
        ],
    },
)
```

**Penjelasan Entry Point:**

| Komponen | Penjelasan |
|----------|-----------|
| `'paste.app_factory'` | Jenis entry point yang dikenali oleh Pyramid/Paste |
| `'main = tutorial:main'` | Menunjukkan bahwa fungsi `main` di package `tutorial` adalah WSGI app factory |
| `tutorial` | Nama package Python |
| `main` | Nama fungsi yang mengembalikan WSGI application |

### Langkah 3: Instalasi Package

Pasang package dalam mode editable (development):

```bash
pip install -e .
```

**Apa yang terjadi:**
- ✅ Menginstal dependensi yang belum ada di virtual environment
- ✅ Membuat egg metadata di `tutorial.egg-info/`
- ✅ Membuat symlink sehingga perubahan kode langsung terlihat
- ✅ Registrasi entry points yang didefinisikan di setup.py

### Langkah 4: Membuat File Konfigurasi development.ini

File ini mengontrol startup aplikasi dan konfigurasi server:

```ini
[app:main]
use = egg:tutorial

[server:main]
use = egg:waitress#main
listen = localhost:6543
```

**Penjelasan Bagian-bagian:**

| Section | Key | Nilai | Fungsi |
|---------|-----|-------|--------|
| `[app:main]` | `use` | `egg:tutorial` | Mereferensikan entry point di setup.py |
| `[server:main]` | `use` | `egg:waitress#main` | Menggunakan Waitress sebagai WSGI server |
| `[server:main]` | `listen` | `localhost:6543` | Server mendengarkan di localhost:6543 |

### Langkah 5: Implementasi Kode di tutorial/__init__.py

Refactor kode startup dari `app.py` ke `__init__.py`:

```python
from pyramid.config import Configurator
from pyramid.response import Response


def hello_world(request):
    """View function yang mengembalikan respons HTML sederhana"""
    return Response('<body><h1>Hello World!</h1></body>')


def main(global_config, **settings):
    """
    WSGI application factory.
    
    Args:
        global_config: Konfigurasi global dari file .ini
        **settings: Settings spesifik dari section [app:main] di .ini
    
    Returns:
        WSGI application yang siap digunakan
    """
    config = Configurator(settings=settings)
    config.add_route('hello', '/')
    config.add_view(hello_world, route_name='hello')
    return config.make_wsgi_app()
```

**Penjelasan Fungsi `main`:**

```
Parameter:
  global_config  → Dictionary berisi konfigurasi global dari .ini
  **settings     → Keyword arguments dari section [app:main]

Operasi:
  1. Inisialisasi Configurator dengan settings
  2. Tambahkan route '/' dengan nama 'hello'
  3. Asosiasikan view function hello_world dengan route
  4. Buat dan kembalikan WSGI application
```

### Langkah 6: Menghapus File Lama

```bash
rm tutorial/app.py
```

File ini tidak lagi diperlukan karena semua kode sudah dipindahkan ke `__init__.py`.

### Langkah 7: Menjalankan Aplikasi

```bash
pserve development.ini --reload
```

Kemudian buka http://localhost:6543/ di browser.

**Opsi Command Line:**

| Opsi | Fungsi |
|------|--------|
| `development.ini` | File konfigurasi yang digunakan |
| `--reload` | Mengamati filesystem dan auto-restart saat ada perubahan |
| `--log-file` (optional) | Menyimpan logs ke file |
| `--debug-all` (optional) | Mengaktifkan debug mode untuk semua logger |

---

## Analisis Detail

### Alur Eksekusi pserve

Ketika menjalankan `pserve development.ini --reload`, berikut adalah alur yang terjadi:

```
┌─────────────────────────────────────────────────────────────┐
│ 1. pserve membaca file development.ini                      │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│ 2. pserve mencari section [app:main]                        │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│ 3. pserve menemukan "use = egg:tutorial"                    │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│ 4. pserve mencari entry point "tutorial:main" di setup.py  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│ 5. pserve menemukan fungsi main() di tutorial/__init__.py  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│ 6. pserve memanggil main(global_config, **settings)         │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│ 7. main() mengembalikan WSGI application                    │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│ 8. pserve menggunakan [server:main] untuk konfigurasi server│
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│ 9. Waitress mendengarkan di localhost:6543                  │
└─────────────────────────────────────────────────────────────┘
```

### Fungsi File .ini

File `.ini` melayani tiga fungsi utama dalam Pyramid:

#### 1️⃣ **Bootstrapping Aplikasi**

```ini
[app:main]
use = egg:tutorial
```

**Fungsi:**
- Menginstruksikan Pyramid untuk menggunakan entry point dari setup.py
- Memungkinkan multiple entry points untuk different apps
- Menghubungkan file config dengan WSGI app factory

**Alur:**
```python
egg:tutorial  →  setup.py entry points  →  tutorial:main  →  tutorial/__init__.py main()
```

#### 2️⃣ **Konfigurasi WSGI Server**

```ini
[server:main]
use = egg:waitress#main
listen = localhost:6543
```

**Fungsi:**
- Menentukan server WSGI mana yang akan digunakan (dalam hal ini Waitress)
- Mengatur parameter server seperti host, port, dan number of threads
- Memisahkan konfigurasi server dari kode aplikasi

**Keuntungan:**
- ✅ Mudah mengganti server tanpa mengubah kode (Waitress → Gunicorn)
- ✅ Configuration berbeda untuk environment berbeda
- ✅ Manajemen resource lebih fleksibel

#### 3️⃣ **Konfigurasi Logging**

```ini
[loggers]
keys = root, tutorial

[handlers]
keys = console

[formatters]
keys = generic

[logger_root]
level = INFO
handlers = console

[logger_tutorial]
level = DEBUG
handlers =
qualname = tutorial

[handler_console]
class = StreamHandler
args = (sys.stderr,)
format = %(asctime)s %(levelname)-5.5s [%(name)s][%(threadName)s] %(message)s

[formatter_generic]
format = %(asctime)s %(levelname)-5.5s [%(name)s][%(threadName)s] %(message)s
datefmt = %Y-%m-%d %H:%M:%S
```

**Fungsi:**
- Python menggunakan modul `logging` standar untuk output logs
- File `.ini` mengonfigurasi level log, handlers, dan formatters
- Menyediakan output console yang informatif dan terstruktur
- Memudahkan debugging dan monitoring

---

### Mengapa Kode Dipindah ke __init__.py?

Dalam tutorial ini, kami memindahkan kode dari `app.py` ke `tutorial/__init__.py`:

```python
# Sebelumnya: di app.py
# def main(): ...

# Sesudah: di tutorial/__init__.py
# def main(): ...
```

**Alasan:**

| Alasan | Penjelasan |
|--------|-----------|
| **Konvensi Pyramid** | Standar untuk menempatkan bootstrapping code di `__init__.py` |
| **Entry Point Resolution** | Ketika Pyramid mencari `tutorial:main`, ia otomatis mencari di `tutorial/__init__.py` |
| **Kebersihan Code** | Memisahkan kode runtime dari kode inisialisasi |
| **Skalabilitas** | Memudahkan pengorganisasian module yang lebih kompleks |
| **Package Structure** | `__init__.py` adalah tempat natural untuk public APIs package |

---

### Instalasi Mode Editable (-e)

```bash
pip install -e .
```

**Apa itu mode editable?**

Mode editable menginstal package dalam development mode, bukan copy ke site-packages.

**Manfaat:**

| Manfaat | Penjelasan |
|--------|-----------|
| **Live Changes** | Perubahan kode langsung terlihat tanpa reinstall |
| **Development Friendly** | Tidak perlu re-run install setiap kali ubah kode |
| **Auto Dependency Install** | Install requirements otomatis jika belum ada |
| **Symbolic Link** | Package di-link, bukan di-copy |

**Perbandingan:**

```bash
# Mode regular (copy ke site-packages)
pip install .

# Mode editable (symlink, development)
pip install -e .
```

---

## Perbandingan: Sebelum vs Sesudah

### Tutorial 02: Tanpa .ini

```bash
# Menjalankan aplikasi langsung
python app.py
```

**Karakteristik:**
- ❌ Startup logic ter-hardcode di app.py
- ❌ Configuration ter-hardcode di dalam kode
- ❌ Sulit untuk multiple configurations
- ❌ Tidak ada reload otomatis
- ❌ Tidak mengikuti konvensi industri

### Tutorial 03: Dengan .ini

```bash
# Menjalankan aplikasi dengan configuration
pserve development.ini --reload
```

**Karakteristik:**
- ✅ Startup logic di __init__.py (terpisah dari config)
- ✅ Configuration di .ini (mudah diubah)
- ✅ Mudah membuat multiple configurations
- ✅ Auto-reload saat development
- ✅ Mengikuti best practices Pyramid
- ✅ Extensible untuk production

---

## Pertanyaan Tambahan dan Jawaban

### ❓ 1. Bisakah Kita Lakukan Ini Tanpa File .ini?

**Jawaban:** Ya, bisa! Kita dapat membuat WSGI app langsung di Python:

```python
# app.py
from pyramid.config import Configurator
from pyramid.response import Response


def hello_world(request):
    return Response('<body><h1>Hello World!</h1></body>')


if __name__ == '__main__':
    config = Configurator()
    config.add_route('hello', '/')
    config.add_view(hello_world, route_name='hello')
    app = config.make_wsgi_app()
    
    # Menggunakan waitress secara langsung
    from waitress import serve
    serve(app, host='localhost', port=6543)
```

**Jalankan:**
```bash
python app.py
```

**Kekurangan Pendekatan Ini:**
- ❌ Hardcoded configuration
- ❌ Tidak fleksibel untuk environment berbeda
- ❌ Tidak mengikuti konvensi Pyramid/Paste
- ❌ Sulit untuk deployment production-ready

**Kesimpulan:** Menggunakan `.ini` jauh lebih fleksibel dan mengikuti best practices industri! 👍

---

### ❓ 2. Bisakah Kita Memiliki Multiple .ini Configuration Files?

**Jawaban:** Ya! Ini sangat common dalam praktik development:

```
project/
├── development.ini    # Untuk development
├── testing.ini        # Untuk testing
└── production.ini     # Untuk production
```

**Contoh Structure:**

```ini
# development.ini
[app:main]
use = egg:tutorial
pyramid.reload_templates = true
pyramid.debug_authorization = false

[server:main]
use = egg:waitress#main
listen = localhost:6543

[logger_root]
level = DEBUG
```

```ini
# production.ini
[app:main]
use = egg:tutorial
pyramid.reload_templates = false
pyramid.debug_authorization = false

[server:main]
use = egg:waitress#main
listen = 0.0.0.0:80

[logger_root]
level = INFO
```

**Mengapa Berguna?**

| Environment | Tujuan | Perbedaan |
|-----------|--------|-----------|
| **development.ini** | Development lokal | Debug=true, reload=true, localhost, verbose logging |
| **testing.ini** | Unit & integration tests | Test database, mock services, no external calls |
| **production.ini** | Server production | Debug=false, reload=false, optimized, security hardened |

**Penggunaan:**

```bash
# Development
pserve development.ini --reload

# Testing
pserve testing.ini

# Production
pserve production.ini
```

**Keuntungan:**
- ✅ Easy environment switching
- ✅ Settings terpisah jelas
- ✅ Reduce configuration errors
- ✅ Security best practice

---

### ❓ 3. Mengapa setup.py Tidak Menyebutkan __init__.py?

**Jawaban:** Karena Python package resolution otomatis mencarinya!

```python
# Entry point di setup.py
entry_points={
    'paste.app_factory': [
        'main = tutorial:main'
    ],
}
```

**Penjelasan:**
- `tutorial`: Nama package Python
- `:main`: Nama fungsi atau atribut di dalam package
- Python **otomatis** mencari di `tutorial/__init__.py` ketika import `tutorial`

**Alur Python Import:**

```python
# Ketika Pyramid mencari 'tutorial:main'
# Python melakukan:

from tutorial import main
# Yang sebenarnya adalah:
from tutorial.__init__ import main
```

**Alternatif Valid:**

```python
# Jika main ada di module yang berbeda
entry_points={
    'paste.app_factory': [
        'main = tutorial.app:create_app'
    ],
}
# File: tutorial/app.py
# def create_app(): ...
```

**Kesimpulan:** Python secara otomatis mencari di `__init__.py`, jadi tidak perlu disebutkan secara eksplisit! ✨

---

### ❓ 4. Apa Fungsi `**settings`? Apa Signifikansi `**`?

**Jawaban:** `**settings` adalah **keyword argument unpacking** untuk menerima settings dinamis dari file .ini:

```python
def main(global_config, **settings):
    # settings adalah dictionary dari semua config di [app:main]
    config = Configurator(settings=settings)
```

**Penjelasan Operator `**`:**

| Operator | Nama | Fungsi |
|----------|------|--------|
| `*args` | Unpacking operator (tuple/list) | Menerima jumlah positional arguments yang tidak diketahui |
| `**kwargs` | Unpacking operator (dictionary) | Menerima jumlah keyword arguments yang tidak diketahui |

**Contoh Penggunaan:**

```python
# Tanpa **kwargs
def greet(name, age):
    print(f"Hello {name}, age {age}")

greet("Alice", 30)  # OK
greet("Bob", 25, "Engineer")  # ERROR: too many arguments

# Dengan **kwargs
def greet(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

greet(name="Alice", age=30)  # OK
greet(name="Bob", age=25, job="Engineer")  # OK - semua masuk ke kwargs
```

**Contoh Dengan Settings Custom di .ini:**

```ini
[app:main]
use = egg:tutorial
pyramid.reload_templates = true
pyramid.debug_authorization = false
my_custom_setting = some_value
database.url = postgresql://localhost/mydb
```

```python
def main(global_config, **settings):
    print("Settings received:")
    for key, value in settings.items():
        print(f"  {key}: {value}")
    
    # Output:
    # Settings received:
    #   pyramid.reload_templates: true
    #   pyramid.debug_authorization: false
    #   my_custom_setting: some_value
    #   database.url: postgresql://localhost/mydb
    
    config = Configurator(settings=settings)
    # Sekarang semua settings tersedia di config.registry.settings
```

**Mengakses Settings di View:**

```python
def my_view(request):
    # Akses custom settings dari request
    db_url = request.registry.settings.get('database.url')
    debug = request.registry.settings.get('pyramid.debug_authorization')
    
    return Response(f"DB: {db_url}, Debug: {debug}")
```

**Keuntungan `**settings`:**
- ✅ Flexible: Bisa tambah settings baru tanpa ubah function signature
- ✅ Dynamic: Settings dibaca dari .ini, tidak hardcoded
- ✅ Extensible: Future settings tidak perlu code changes
- ✅ Pythonic: Mengikuti Python conventions

---

## Kesimpulan

### Ringkasan Penting

File `.ini` dalam Pyramid menyediakan:

| Aspek | Manfaat |
|-------|---------|
| **Pemisahan Concern** | Konfigurasi terpisah dari kode → Mudah maintenance |
| **Fleksibilitas** | Multiple configurations untuk different environments |
| **Standar Industri** | Mengikuti konvensi Python Setuptools → Industry best practice |
| **Development Friendly** | Built-in reload mechanism → Faster development |
| **Logging Configuration** | Centralized logging setup → Better debugging |
| **Extensibility** | Easy to add new settings → Future-proof |
| **Production Ready** | Scalable solution → Suitable for enterprise |

### Best Practices

✅ **DO:**
- Gunakan `.ini` untuk konfigurasi environment-specific
- Pisahkan development.ini, testing.ini, dan production.ini
- Gunakan entry points di setup.py
- Place startup code di __init__.py
- Gunakan `--reload` saat development

❌ **DON'T:**
- Hardcode configuration di Python code
- Gunakan sama configuration untuk semua environment
- Bypass entry points untuk direct imports
- Lupa `pip install -e .` setelah ubah setup.py
- Gunakan `--reload` di production

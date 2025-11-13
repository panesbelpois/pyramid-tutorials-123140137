# 12: Templating With Jinja2

## Deskripsi

Tutorial ini mendemonstrasikan fleksibilitas Pyramid dalam mendukung berbagai sistem templating. Kami akan mengintegrasikan **Jinja2**, sistem templating populer yang digunakan oleh Flask dan dimodelkan berdasarkan template Django, ke dalam aplikasi Pyramid menggunakan add-on `pyramid_jinja2`.

Jinja2 menawarkan sintaks yang kuat dan intuitif untuk rendering template HTML, dengan fitur-fitur seperti:
- Variable interpolation (`{{ variable }}`)
- Control structures (if, for, while)
- Filters dan functions
- Template inheritance dan inclusion
- Macro support

Pyramid tidak membatasi pilihan templating engine, memungkinkan developers untuk memilih solusi terbaik untuk kebutuhan mereka.

## Objektif

1. Mendemonstrasikan dukungan Pyramid terhadap berbagai sistem templating
2. Mempelajari cara menginstall dan mengkonfigurasi Pyramid add-ons
3. Memahami perbedaan antara Jinja2 dan Chameleon templating
4. Mengimplementasikan rendering template Jinja2 dalam views

## Instalasi

### Prasyarat

- Python virtual environment yang sudah aktif
- Pyramid dan dependencies dasar sudah terinstall

### Langkah-Langkah Instalasi

#### 1. Salin direktori dari step sebelumnya
```bash
cd ..
cp -r view_classes jinja2
cd jinja2
```

#### 2. Tambahkan `pyramid_jinja2` ke dependencies dalam `setup.py`

```python
from setuptools import setup

# List of dependencies installed via `pip install -e .`
# by virtue of the Setuptools `install_requires` value below.
requires = [
    'pyramid',
    'pyramid_chameleon',
    'pyramid_jinja2',
    'waitress',
]

# List of dependencies installed via `pip install -e ".[dev]"`
# by virtue of the Setuptools `extras_require` value in the Python
# dictionary below.
dev_requires = [
    'pyramid_debugtoolbar',
    'pytest',
    'webtest',
]

setup(
    name='tutorial',
    install_requires=requires,
    extras_require={
        'dev': dev_requires,
    },
    entry_points={
        'paste.app_factory': [
            'main = tutorial:main'
        ],
    },
)
```

#### 3. Instalasi project dan dependencies baru

```bash
$VENV/bin/pip install -e .
```

#### 4. Konfigurasi Pyramid untuk menggunakan Jinja2 dalam `tutorial/__init__.py`

```python
from pyramid.config import Configurator

def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_jinja2')
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()
```

#### 5. Update views untuk menggunakan Jinja2 renderer dalam `tutorial/views.py`

```python
from pyramid.view import (
    view_config,
    view_defaults
    )

@view_defaults(renderer='home.jinja2')
class TutorialViews:
    def __init__(self, request):
        self.request = request

    @view_config(route_name='home')
    def home(self):
        return {'name': 'Home View'}

    @view_config(route_name='hello')
    def hello(self):
        return {'name': 'Hello View'}
```

#### 6. Buat template Jinja2 `tutorial/home.jinja2`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: {{ name }}</title>
</head>
<body>
<h1>Hi {{ name }}</h1>
</body>
</html>
```

#### 7. Jalankan tests

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

Expected output:
```
....
4 passed in 0.40 seconds
```

#### 8. Jalankan aplikasi Pyramid

```bash
$VENV/bin/pserve development.ini --reload
```

Kemudian buka `http://localhost:6543/` di browser Anda.

## Analisis

### Integrasi Add-On Pyramid

Mengintegrasikan Pyramid add-on ke dalam aplikasi adalah proses yang straightforward:

1. **Package Installation**: Gunakan tools Python standar (pip) untuk menginstall package add-on ke dalam virtual environment
2. **Configurator Integration**: Gunakan `config.include()` untuk memberitahu Pyramid's Configurator menjalankan setup code dari add-on
3. **Renderer Registration**: Setup code dari `pyramid_jinja2` mendaftarkan renderer baru yang mengenali file dengan extension `.jinja2`

### Komparasi Templating Systems

**Similarities antara Jinja2 dan Chameleon:**
- Basic variable insertion menggunakan sintaks yang mirip: `{{ variable }}`
- Keduanya mendukung control structures
- Keduanya dapat terintegrasi seamlessly dengan Pyramid views

**Perbedaan:**
- **Jinja2**: Template-centric, digunakan oleh Flask, powerful filtering system, template inheritance
- **Chameleon**: Berbasis XML attributes, lebih strict pada valid XHTML/XML

### Kunci Pembelajaran

Fleksibilitas Pyramid memungkinkan developers untuk:
- Memilih templating engine yang paling sesuai dengan kebutuhan project
- Menggunakan multiple templating systems dalam satu aplikasi
- Mudah beralih antara templating systems dengan minimal code changes

## Extra Credit - Jawaban

### Pertanyaan 1: Alternatif untuk menambahkan dependency `pyramid_jinja2`

**Pertanyaan:** Our project now depends on `pyramid_jinja2`. We installed that dependency manually. What is another way we could have made the association?

**Jawaban:**

Selain menambahkan `pyramid_jinja2` secara manual ke `setup.py`, ada beberapa alternatif:

1. **Menggunakan Pyramid Scaffolds/Cookiecutters**: 
   - Gunakan template project yang sudah pre-configured dengan Jinja2
   - Contoh: `pcreate -s jinja2_starter myproject`
   - Scaffold ini akan otomatis include semua dependencies

2. **Menggunakan `requirements.txt`**:
   - Bisa mendefinisikan dependencies dalam file terpisah (misalnya `requirements.txt`)
   - Kemudian install dengan: `pip install -r requirements.txt`
   - File `requirements.txt` kemudian direferensi di `setup.py`:
   ```python
   with open('requirements.txt') as f:
       requires = [line.strip() for line in f if line.strip()]
   ```

3. **Pyramid Project Templates**:
   - Menggunakan `cookiecutter` dengan template yang sudah dikonfigurasi untuk Jinja2
   - Ini akan scaffold project dengan semua dependencies yang diperlukan

4. **Declarative Configuration di `development.ini`**:
   - Meskipun jarang, dependency management bisa juga dibuat lebih modular melalui configuration system
   - Namun, dependencies tetap perlu di-install via pip terlebih dahulu

5. **Using pip with URL**:
   ```bash
   pip install git+https://github.com/Pylons/pyramid_jinja2.git
   ```
   - Install langsung dari repository tanpa melalui setup.py

**Best Practice**: Menyimpan dependencies di `setup.py` adalah cara yang paling Pythonic dan reproducible karena:
- Memastikan dependencies terinstall otomatis saat `pip install -e .`
- Kompatibel dengan package distribution
- Mudah mengelola versi dependencies

---

### Pertanyaan 2: Alternatif untuk menggunakan `config.include()`

**Pertanyaan:** We used `config.include` which is an imperative configuration to get the Configurator to load `pyramid_jinja2`'s configuration. What is another way we could include it into the config?

**Jawaban:**

Selain menggunakan `config.include('pyramid_jinja2')` secara imperative dalam code, ada beberapa alternatif:

#### 1. **Declarative Configuration via `development.ini`** (Rekomendasi)

Modifikasi file `development.ini`:
```ini
[app:main]
use = egg:tutorial
pyramid.includes =
    pyramid_jinja2
    pyramid_chameleon
```

Kemudian `tutorial/__init__.py` akan memprosesnya otomatis:
```python
from pyramid.config import Configurator

def main(global_config, **settings):
    config = Configurator(settings=settings)
    # Pyramid otomatis memproses pyramid.includes dari settings file
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()
```

**Keuntungan**:
- Mudah untuk mengubah add-ons tanpa memodifikasi code
- Environment-specific configuration (development vs production)
- Decoupling konfigurasi dari code

#### 2. **Menggunakan Autoinclude Entry Points**

Jika package add-on mendukung entry points dengan "pyramid.includes" entry point, Pyramid bisa secara otomatis menemukan dan meload configuration:

Setup di package add-on:
```python
entry_points={
    'pyramid.includes': [
        'pyramid_jinja2 = pyramid_jinja2:includeme'
    ]
}
```

Kemudian Pyramid akan otomatis meload tanpa perlu `config.include()` atau konfigurasi tambahan.

#### 3. **Configurator Factory Approach**

Membuat helper function yang pre-configured:
```python
def make_configured_app(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_jinja2')
    config.include('pyramid_chameleon')
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()

def main(global_config, **settings):
    return make_configured_app(global_config, **settings)
```

#### 4. **Module-level Configuration**

Membuat modul terpisah untuk konfigurasi:

File `tutorial/config.py`:
```python
def add_renderers(config):
    config.include('pyramid_jinja2')
    config.include('pyramid_chameleon')

def add_routes(config):
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
```

File `tutorial/__init__.py`:
```python
from pyramid.config import Configurator
from .config import add_renderers, add_routes

def main(global_config, **settings):
    config = Configurator(settings=settings)
    add_renderers(config)
    add_routes(config)
    config.scan('.views')
    return config.make_wsgi_app()
```

#### 5. **ZCML Configuration (Legacy)**

Menggunakan Zope Configuration Markup Language:

File `tutorial/configure.zcml`:
```xml
<configure xmlns="http://pylonshq.com/configure">
    <include package="pyramid_jinja2" />
</configure>
```

File `tutorial/__init__.py`:
```python
from pyramid.config import Configurator
from pyramid.path import AssetResolver

def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.load_zcml('tutorial:configure.zcml')
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()
```

---

## Perbandingan Alternatif

| Metode | Imperative/Declarative | Pros | Cons |
|--------|------------------------|------|------|
| `config.include()` | Imperative | Eksplisit, mudah debug | Harus edit code untuk perubahan |
| `development.ini` | Declarative | Environment-specific, flexible | Perlu mengerti format file |
| Entry Points | Declarative | Otomatis, clean | Package harus support |
| Module Factory | Imperative | Terorganisir | Tambah kompleksitas |
| ZCML | Declarative | XML-based config | Ketinggalan zaman, verbose |

**Best Practice Recommendation**:
- Untuk **development**: Gunakan `config.include()` di code (lebih explicit dan mudah di-debug)
- Untuk **production dengan environment yang berbeda**: Gunakan declarative configuration di `development.ini` atau `production.ini`
- Kombinasi keduanya adalah ideal: use `config.include()` untuk konfigurasi dasar, dan `development.ini` untuk overrides environment-specific

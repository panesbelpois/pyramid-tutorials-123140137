# 13: Pyramid Jinja2 Templating Tutorial

## Deskripsi

Tutorial ini mendemonstrasikan bagaimana mengintegrasikan **Jinja2** sebagai sistem templating dalam aplikasi Pyramid. Jinja2 adalah template engine yang populer dan digunakan oleh Flask, serta dimodelkan berdasarkan Django's templates. Melalui tutorial ini, kita akan belajar menggunakan `pyramid_jinja2`, sebuah add-on Pyramid yang memungkinkan Jinja2 berfungsi sebagai renderer dalam aplikasi Pyramid.

### Objektif Pembelajaran
- Menunjukkan dukungan Pyramid terhadap berbagai sistem templating
- Mempelajari cara instalasi Pyramid add-ons
- Memahami integrasi Jinja2 dengan Pyramid
- Membandingkan sintaks templating Jinja2 dengan Chameleon

---

## Instalasi

### Prasyarat
- Python 3.x
- Virtual Environment (venv)
- pip (Python package installer)

### Langkah-Langkah Instalasi

#### 1. Setup Proyek
```bash
# Buat direktori proyek
cd /path/to/projects
cp -r view_classes jinja2
cd jinja2
```

#### 2. Update Dependensi di `setup.py`
Tambahkan `pyramid_jinja2` ke dalam daftar dependencies:

```python
from setuptools import setup

requires = [
    'pyramid',
    'pyramid_chameleon',
    'pyramid_jinja2',  # Tambahan untuk Jinja2
    'waitress',
]

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

#### 3. Install Proyek dan Dependensi
```bash
# Aktifkan virtual environment
source $VENV/bin/activate  # Linux/macOS
# atau
$VENV\Scripts\activate  # Windows

# Install proyek dengan dependencies
pip install -e .
```

#### 4. Konfigurasi di `tutorial/__init__.py`
```python
from pyramid.config import Configurator

def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_jinja2')  # Aktifkan Jinja2
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()
```

#### 5. Update Views di `tutorial/views.py`
```python
from pyramid.view import view_config, view_defaults

@view_defaults(renderer='home.jinja2')  # Ganti ke extension Jinja2
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

#### 6. Buat Template `tutorial/home.jinja2`
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

#### 7. Jalankan Tests
```bash
pytest tutorial/tests.py -q
```

Output yang diharapkan:
```
....
4 passed in 0.40 seconds
```

#### 8. Jalankan Aplikasi
```bash
pserve development.ini --reload
```

Akses aplikasi di: `http://localhost:6543/`

---

## Analisis

### Integrasi Pyramid Add-on

Proses integrasi Pyramid add-on sangat sederhana dan mengikuti pola standar:

1. **Instalasi Package**: Menggunakan tools Python standard (`pip`) untuk menginstal add-on package ke dalam Python virtual environment
2. **Registrasi dengan Configurator**: Menggunakan `config.include()` untuk memberitahu Pyramid's configurator agar menjalankan setup code dari add-on
3. **Konfigurasi Renderer**: Setup code dari pyramid_jinja2 mendaftarkan renderer baru yang mengenali file extension `.jinja2`

### Perubahan pada View Code

View code tetap relatif konsisten dengan struktur sebelumnya. Perubahan utama adalah:
- Mengganti file extension renderer dari `.pt` (Chameleon) menjadi `.jinja2` (Jinja2)
- Tidak ada perubahan pada logika view atau return values

### Perbandingan Jinja2 vs Chameleon

Dari segi syntax dasar untuk insersi variabel, keduanya sangat mirip:

**Chameleon (template.pt):**
```html
<h1>Hi ${name}</h1>
```

**Jinja2 (template.jinja2):**
```html
<h1>Hi {{ name }}</h1>
```

Perbedaan utama hanya pada delimiter syntax. Jinja2 menggunakan double curly braces `{{ }}` sementara Chameleon menggunakan dollar-sign notation `${}`.

### Keuntungan Menggunakan Jinja2

- **Familiar untuk Flask developers**: Jinja2 adalah default template engine di Flask
- **Django-inspired**: Konsep dan syntax mirip dengan Django templates
- **Fleksibel**: Mendukung fitur-fitur template engine yang advanced seperti macros, filters, dan inheritance
- **Multi-framework support**: Tidak terikat pada satu framework saja

---

## Extra Credit - Jawaban

### Pertanyaan 1: Cara Lain untuk Mendeklarasikan Dependensi

**Pertanyaan:** "Proyek kami sekarang bergantung pada pyramid_jinja2. Kami menginstal dependensi tersebut secara manual. Apa cara lain untuk membuat asosiasi?"

**Jawaban:**

Ada beberapa cara lain untuk mendeklarasikan dependensi pyramid_jinja2:

#### **1. Via `setup.cfg` (Setup Configuration File)**
```ini
[metadata]
name = tutorial
version = 1.0

[options]
install_requires =
    pyramid
    pyramid_chameleon
    pyramid_jinja2
    waitress

[options.extras_require]
dev =
    pyramid_debugtoolbar
    pytest
    webtest

[options.entry_points]
paste.app_factory =
    main = tutorial:main
```

#### **2. Via `pyproject.toml` (Modern Python Packaging - PEP 517/518)**
```toml
[build-system]
requires = ["setuptools>=45", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "tutorial"
version = "1.0.0"
dependencies = [
    "pyramid",
    "pyramid_chameleon",
    "pyramid_jinja2",
    "waitress",
]

[project.optional-dependencies]
dev = [
    "pyramid_debugtoolbar",
    "pytest",
    "webtest",
]

[project.entry-points."paste.app_factory"]
main = "tutorial:main"
```

#### **3. Via `requirements.txt` (Manual Management)**
```
pyramid
pyramid_chameleon
pyramid_jinja2
waitress
```
Kemudian install dengan:
```bash
pip install -r requirements.txt
```

#### **4. Via Environment-specific Files**
```
requirements-base.txt (production dependencies)
requirements-dev.txt (development dependencies)
requirements-test.txt (testing dependencies)
```

**Rekomendasi Modern:** Gunakan `pyproject.toml` karena ini adalah standar modern Python packaging (PEP 517/518/621) yang lebih fleksibel dan maintainable.

---

### Pertanyaan 2: Cara Lain untuk Include Konfigurasi

**Pertanyaan:** "Kami menggunakan `config.include` yang merupakan imperative configuration untuk mendapatkan Configurator memuat konfigurasi pyramid_jinja2. Apa cara lain untuk mengincludenya ke dalam config?"

**Jawaban:**

Ada beberapa alternatif untuk menginclude konfigurasi pyramid_jinja2:

#### **1. Via Declarative Configuration (INI file)**
Edit `development.ini`:
```ini
[app:main]
use = egg:tutorial
pyramid.includes =
    pyramid_jinja2
```

Kemudian di `tutorial/__init__.py`:
```python
from pyramid.config import Configurator

def main(global_config, **settings):
    config = Configurator(settings=settings)
    # config.include('pyramid_jinja2')  # Dihapus
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()
```

#### **2. Via Entry Points (Automatic Discovery)**
Di `setup.py`, tambahkan entry point untuk pyramid plugins:
```python
setup(
    name='tutorial',
    install_requires=requires,
    extras_require={'dev': dev_requires},
    entry_points={
        'paste.app_factory': [
            'main = tutorial:main'
        ],
        'pyramid.includes': [  # Automatic loading
            'pyramid_jinja2',
        ]
    },
)
```

Keuntungan: Jinja2 akan otomatis di-load tanpa perlu `config.include()` atau declarative config.

#### **3. Via Pyramid Configurator `__enter__`/`__exit__` (Context Manager)**
```python
from pyramid.config import Configurator

def main(global_config, **settings):
    with Configurator(settings=settings) as config:
        config.include('pyramid_jinja2')
        config.add_route('home', '/')
        config.add_route('hello', '/howdy')
        config.scan('.views')
        return config.make_wsgi_app()
```

#### **4. Via Action Chain (Multiple Includes)**
```python
def main(global_config, **settings):
    config = Configurator(settings=settings)
    
    # Include multiple add-ons dalam satu chain
    config.include('pyramid_jinja2')
    config.include('pyramid_chameleon')
    
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()
```

#### **5. Via Configurator include dengan Callable**
```python
def include_jinja2(config):
    """Custom inclusion function"""
    config.include('pyramid_jinja2')

def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include(include_jinja2)
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()
```

**Perbandingan Metode:**

| Metode | Kelebihan | Kekurangan |
|--------|-----------|-----------|
| **Imperative (config.include)** | Eksplisit, mudah dikontrol | Harus di-code di setiap proyek |
| **Declarative (INI)** | Fleksibel, bisa diubah tanpa restart | Lebih kompleks untuk logika conditional |
| **Entry Points** | Otomatis, zero-config | Kurang eksplisit |
| **Context Manager** | Clean, pythonic | Hanya untuk Python 3.x |

**Rekomendasi:** Gunakan **Declarative Configuration (INI file)** untuk production karena memisahkan konfigurasi dari code, memudahkan deployment di berbagai environment.

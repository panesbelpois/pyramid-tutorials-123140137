# 13: Pyramid Static Assets Tutorial (CSS/JS/Images)

## Deskripsi

Tutorial ini menunjukkan cara mengelola dan melayani **static assets** (CSS, JavaScript, dan gambar) dalam aplikasi Pyramid. Kami akan belajar bagaimana mengonfigurasi direktori static assets, menghubungkan file CSS ke template, dan menggunakan Pyramid's helper functions untuk menghasilkan URL yang fleksibel ke static files. Ini adalah aspek penting dari pengembangan web modern yang memisahkan concerns antara markup dan styling/scripting.

### Objektif Pembelajaran
- Mempublikasikan direktori static assets pada suatu URL
- Menggunakan Pyramid untuk membantu generate URLs ke files dalam direktori static
- Memahami konsep asset management dalam web applications
- Mempelajari best practices dalam linking static resources

---

## Instalasi

### Prasyarat
- Python 3.x
- Virtual Environment (venv)
- pip (Python package installer)
- Pyramid framework sudah terinstal

### Langkah-Langkah Instalasi

#### 1. Setup Proyek
```bash
# Copy dari step view_classes
cd /path/to/projects
cp -r view_classes static_assets
cd static_assets

# Aktifkan virtual environment
source $VENV/bin/activate  # Linux/macOS
# atau
$VENV\Scripts\activate  # Windows

# Install dependencies
pip install -e .
```

#### 2. Konfigurasi Static View di `tutorial/__init__.py`
Tambahkan `config.add_static_view()` untuk mendaftarkan direktori static:

```python
from pyramid.config import Configurator

def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_chameleon')
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    
    # Tambahkan static view - map /static/ URLs ke tutorial:static directory
    config.add_static_view(name='static', path='tutorial:static')
    
    config.scan('.views')
    return config.make_wsgi_app()
```

**Penjelasan parameter:**
- `name='static'`: Nama prefix URL untuk static assets
- `path='tutorial:static'`: Path ke direktori static relative ke package

#### 3. Update Template di `tutorial/home.pt`
Tambahkan link CSS dan gunakan `request.static_url()`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: ${name}</title>
    <link rel="stylesheet"
          href="${request.static_url('tutorial:static/app.css') }"/>
</head>
<body>
<h1>Hi ${name}</h1>
</body>
</html>
```

**Penjelasan:**
- `request.static_url()`: Helper function untuk generate full URL ke static assets
- Parameter mengikuti format yang sama dengan `path` di `config.add_static_view()`

#### 4. Buat File CSS di `tutorial/static/app.css`
```css
body {
    margin: 2em;
    font-family: sans-serif;
}
```

#### 5. Tambah Test Untuk Static Assets di `tutorial/tests.py`
```python
def test_css(self):
    res = self.testapp.get('/static/app.css', status=200)
    self.assertIn(b'body', res.body)
```

Test ini memverifikasi bahwa:
- File CSS dapat diakses melalui URL `/static/app.css`
- Response status adalah 200 (OK)
- Content berisi `body` CSS selector

#### 6. Jalankan Tests
```bash
pytest tutorial/tests.py -q
```

Output yang diharapkan:
```
.....
5 passed in 0.50 seconds
```

#### 7. Jalankan Aplikasi
```bash
pserve development.ini --reload
```

Akses aplikasi di: `http://localhost:6543/`

Anda akan melihat halaman dengan styling yang diaplikasikan dari `app.css`.

---

## Analisis

### Konfigurasi Static Assets

Pyramid membutuhkan explicit configuration untuk melayani static files. Ini dilakukan menggunakan `config.add_static_view()`:

```python
config.add_static_view(name='static', path='tutorial:static')
```

**Apa yang terjadi:**
1. **Routing**: Semua request ke `/static/*` akan di-handle oleh static view
2. **Mapping**: Path yang di-request di-map ke direktori `tutorial:static` di file system
3. **Serving**: Pyramid akan mengambil file dari direktori tersebut dan mengirimnya ke client

### Struktur Direktori
```
tutorial/
├── __init__.py
├── app.py
├── home.pt
├── views.py
├── tests.py
└── static/           # Direktori untuk static assets
    └── app.css       # CSS file
```

### request.static_url() vs Hard-coded URLs

#### Hard-coded URL (Tidak Recommended):
```html
<link rel="stylesheet" href="/static/app.css"/>
```

**Masalah:**
- Brittle: Jika struktur berubah, semua URLs harus diupdate manual
- Tidak fleksibel: Sulit untuk deployment di sub-path
- Risk of bugs: Mudah terjadi typo atau miss saat refactor

#### Menggunakan request.static_url() (Recommended):
```html
<link rel="stylesheet" href="${request.static_url('tutorial:static/app.css')}"/>
```

**Keuntungan:**
- **Fleksibel**: Perubahan di config otomatis tercermin di semua templates
- **Maintainable**: Single source of truth untuk static path configuration
- **Refactoring-friendly**: Bisa move static assets tanpa mengubah templates
- **DRY Principle**: Tidak perlu hardcode path di multiple places

### Asset Path Notation

Pyramid menggunakan **asset specification** format:
```
package_name:relative/path/to/asset
```

Contoh:
```python
# File CSS di tutorial/static/app.css
'tutorial:static/app.css'

# JavaScript di tutorial/static/js/main.js
'tutorial:static/js/main.js'

# Image di tutorial/static/images/logo.png
'tutorial:static/images/logo.png'
```

### Serving Static Files dalam Development vs Production

#### Development (pserve dengan --reload):
- Pyramid secara otomatis serve static files
- Hot-reload enabled

#### Production:
- Biasanya static files dilayani oleh web server (nginx, Apache)
- Pyramid bisa dikonfigurasi untuk tidak serve static files
- Lebih performant dan scalable

---

## Extra Credit - Jawaban

### Pertanyaan: `request.static_path()` vs `request.static_url()`

**Pertanyaan:** "Ada juga API `request.static_path`. Bagaimana ini berbeda dari `request.static_url`?"

**Jawaban:**

Kedua API ini digunakan untuk generate path ke static assets, tetapi dengan output yang berbeda:

#### **1. `request.static_url()` - Generate Full URL**

**Definisi:** Mengembalikan URL lengkap yang dapat langsung digunakan di browser (dengan protocol dan domain).

**Sintaks:**
```python
request.static_url('tutorial:static/app.css')
```

**Output:**
```
http://localhost:6543/static/app.css
```

**Penggunaan:**
```html
<!-- HTML href attribute memerlukan full URL -->
<link rel="stylesheet" href="${request.static_url('tutorial:static/app.css')}"/>

<!-- atau di script -->
<script src="${request.static_url('tutorial:static/js/main.js')}"></script>

<!-- atau gambar -->
<img src="${request.static_url('tutorial:static/images/logo.png')}" alt="Logo"/>
```

**Kegunaan:**
- HTML templates (href, src)
- Ketika memerlukan full URL lengkap dengan protocol dan domain
- Ketika URL akan digunakan di context yang berbeda (AJAX calls, redirects)

#### **2. `request.static_path()` - Generate Relative Path**

**Definisi:** Mengembalikan hanya path (tanpa protocol dan domain), relative ke application root.

**Sintaks:**
```python
request.static_path('tutorial:static/app.css')
```

**Output:**
```
/static/app.css
```

**Penggunaan:**
```python
# Di dalam view function
from pyramid.response import Response

def my_view(request):
    css_path = request.static_path('tutorial:static/app.css')
    # css_path = '/static/app.css'
    return Response(css_path)

# Atau di template ketika perlu hanya path
def get_asset_path(request):
    return request.static_path('tutorial:static/style.css')
    # Returns: '/static/style.css'
```

**Kegunaan:**
- Ketika hanya memerlukan relative path saja
- Menggunakan path di Python code untuk manipulasi
- Ketika sudah tahu domain/protocol-nya
- URL rewriting atau proxy scenarios

#### **Perbandingan Detail**

| Aspek | `static_url()` | `static_path()` |
|-------|---|---|
| **Output** | Full URL: `http://localhost:6543/static/app.css` | Path saja: `/static/app.css` |
| **Includes** | Protocol, domain, port, path | Hanya path |
| **Use Case** | HTML attributes (href, src), links | Path manipulation, redirects |
| **Template Usage** | Recommended | Jarang |
| **Performance** | Sedikit lebih heavy | Lebih lightweight |
| **Cross-domain** | Bisa dengan config | Tidak applicable |

#### **Contoh Praktis Perbandingan**

```python
# View function
from pyramid.view import view_config

@view_config(route_name='asset_info')
def asset_info(request):
    # Menggunakan static_url()
    full_url = request.static_url('tutorial:static/app.css')
    # Result: 'http://localhost:6543/static/app.css'
    
    # Menggunakan static_path()
    relative_path = request.static_path('tutorial:static/app.css')
    # Result: '/static/app.css'
    
    return {
        'full_url': full_url,
        'path': relative_path
    }
```

#### **Template Example**

```html
<!DOCTYPE html>
<html>
<head>
    <!-- static_url() untuk HTML attributes -->
    <link rel="stylesheet" href="${request.static_url('tutorial:static/app.css')}"/>
    <script src="${request.static_url('tutorial:static/js/main.js')}"></script>
</head>
<body>
    <img src="${request.static_url('tutorial:static/images/logo.png')}" alt="Logo"/>
    
    <!-- Jarang menggunakan static_path() di template, tapi bisa jika perlu -->
    <!-- Path value: ${request.static_path('tutorial:static/app.css')} -->
</body>
</html>
```

#### **Kapan Menggunakan Masing-masing**

**Gunakan `static_url()`:**
- ✅ HTML `href` dan `src` attributes
- ✅ AJAX requests ke static files
- ✅ Ketika perlu full URL dengan scheme
- ✅ External links atau API responses
- ✅ Untuk umum dan safe, gunakan ini

**Gunakan `static_path()`:**
- ✅ Manipulasi path di Python code
- ✅ Ketika sudah tahu domainnya
- ✅ Path construction atau joining
- ✅ Ketika perlu hanya relative path
- ✅ Performance-critical scenarios (jarang diperlukan)

#### **Best Practice Rekomendasi**

**Gunakan `request.static_url()` sebagai default untuk HTML templates.** Ini adalah yang paling aman dan fleksibel. `request.static_path()` lebih jarang digunakan dan lebih untuk kasus-kasus spesifik di backend code.

```python
# RECOMMENDED untuk templates
href="${request.static_url('tutorial:static/app.css')}"

# Jarang digunakan
path = "${request.static_path('tutorial:static/app.css')}"
```

---

## Best Practices

### 1. Struktur Direktori Static Assets
```
tutorial/static/
├── css/
│   ├── app.css
│   └── responsive.css
├── js/
│   ├── main.js
│   └── utils.js
└── images/
    ├── logo.png
    └── favicon.ico
```

### 2. Gunakan `request.static_url()` di Templates
```html
<!-- Good -->
<link rel="stylesheet" href="${request.static_url('tutorial:static/css/app.css')}"/>

<!-- Bad - hard-coded -->
<link rel="stylesheet" href="/static/css/app.css"/>
```

### 3. Cache Busting untuk Production
Tambahkan query parameter untuk force-refresh cache:
```html
<link rel="stylesheet" href="${request.static_url('tutorial:static/css/app.css')}?v=1.0"/>
```

### 4. Asset Organization
- Organize by type (css, js, images)
- Use descriptive filenames
- Minify untuk production
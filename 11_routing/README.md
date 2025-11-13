# 11: Dispatching URLs To Views With Routing

## Deskripsi

Routing adalah mekanisme yang mencocokkan pola URL yang masuk ke kode view. Tutorial ini menjelaskan fitur-fitur penting dalam sistem routing Pyramid, khususnya bagaimana mengekstrak bagian dari URL ke dalam Python dictionary dan menggunakannya dalam view.

Pyramid menggunakan pendekatan yang unik dalam routing dibandingkan framework Python web lainnya:
- **Route Registration**: Di file konfigurasi (`__init__.py`), kita mendaftarkan nama route dengan pola URL
- **View Configuration**: Di file view, kita mengkonfigurasi view untuk dipanggil berdasarkan nama route

Pendekatan dua langkah ini memberikan kontrol eksplisit atas urutan pencocokan route, yang penting ketika beberapa route mungkin cocok dengan pola URL yang sama.

### Objective (Tujuan)
1. Mendefinisikan route dengan replacement pattern untuk mengekstrak bagian URL
2. Menggunakan data matchdict dalam view
3. Memahami mekanisme routing dan pencocokan URL di Pyramid

---

## Instalasi

### 1. Setup Environment

Pastikan Anda sudah berada di direktori `11_routing`:

```bash
# Jika belum ada folder routing, copy dari view_classes
cd ..
cp -r view_classes routing
cd routing
```

### 2. Install Dependencies

```bash
# Install virtual environment packages
$VENV/bin/pip install -e .
```

Atau jika menggunakan Python langsung:

```bash
pip install -e .
```

### 3. Setup Struktur File

Pastikan struktur folder adalah sebagai berikut:

```
routing/
├── setup.py
├── development.ini
├── README.md
└── tutorial/
    ├── __init__.py
    ├── views.py
    ├── home.pt
    ├── tests.py
    └── __pycache__/
```

### 4. Jalankan Aplikasi

```bash
# Development mode dengan auto-reload
$VENV/bin/pserve development.ini --reload
```

Buka browser dan akses: `http://localhost:6543/howdy/amy/smith`

### 5. Jalankan Test

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

Expected output:
```
..
2 passed in 0.39 seconds
```

---

## Analisis

### Konsep Dasar Routing

#### 1. Route Declaration di `__init__.py`

```python
from pyramid.config import Configurator

def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_chameleon')
    config.add_route('home', '/howdy/{first}/{last}')
    config.scan('.views')
    return config.make_wsgi_app()
```

**Penjelasan:**
- `config.add_route('home', '/howdy/{first}/{last}')`: 
  - Parameter pertama (`'home'`) adalah **nama route** yang unik
  - Parameter kedua (`'/howdy/{first}/{last}'`) adalah **pola URL** dengan replacement pattern
  - Kurly braces `{first}` dan `{last}` adalah **variabel dinamis** yang akan diambil dari URL

#### 2. Replacement Pattern

Pola URL `/howdy/{first}/{last}` akan cocok dengan URL seperti:
- `/howdy/amy/smith` → `first='amy'`, `last='smith'`
- `/howdy/john/doe` → `first='john'`, `last='doe'`
- `/howdy/jane/williams` → `first='jane'`, `last='williams'`

Nilai-nilai ini disimpan dalam `request.matchdict` sebagai dictionary.

#### 3. View Implementation di `views.py`

```python
from pyramid.view import view_config, view_defaults

@view_defaults(renderer='home.pt')
class TutorialViews:
    def __init__(self, request):
        self.request = request

    @view_config(route_name='home')
    def home(self):
        # Mengakses nilai dari matchdict
        first = self.request.matchdict['first']
        last = self.request.matchdict['last']
        
        return {
            'name': 'Home View',
            'first': first,
            'last': last
        }
```

**Penjelasan:**
- `@view_config(route_name='home')`: Menghubungkan view ini dengan route bernama `'home'`
- `self.request.matchdict`: Dictionary yang berisi nilai-nilai yang diekstrak dari URL
- Return value adalah dictionary yang akan dipass ke template

#### 4. Template di `home.pt`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: ${name}</title>
</head>
<body>
<h1>${name}</h1>
<p>First: ${first}, Last: ${last}</p>
</body>
</html>
```

Template Chameleon menggunakan syntax `${variable}` untuk menampilkan nilai dari context dictionary.

#### 5. Testing di `tests.py`

Terdapat dua jenis test:

**Unit Test:**
```python
def test_home(self):
    request = testing.DummyRequest()
    request.matchdict['first'] = 'First'
    request.matchdict['last'] = 'Last'
    inst = TutorialViews(request)
    response = inst.home()
    self.assertEqual(response['first'], 'First')
    self.assertEqual(response['last'], 'Last')
```

**Functional Test:**
```python
def test_home(self):
    res = self.testapp.get('/howdy/Jane/Doe', status=200)
    self.assertIn(b'Jane', res.body)
    self.assertIn(b'Doe', res.body)
```

### Alur Kerja Pyramid Routing

```
URL Request
    ↓
/howdy/amy/smith
    ↓
Route Matching (di __init__.py)
Pattern: /howdy/{first}/{last}
    ↓
Matchdict Creation
{'first': 'amy', 'last': 'smith'}
    ↓
View Dispatching (route_name='home')
    ↓
TutorialViews.home()
    ↓
Template Rendering (home.pt)
    ↓
HTML Response
```

### Keuntungan Pendekatan Dua-Langkah Pyramid

1. **Kontrol Eksplisit**: Urutan route dapat dikontrol dengan jelas
2. **Mencegah Ambiguitas**: Ketika multiple route cocok, tidak ada guessing
3. **Fleksibilitas**: Mudah menambah view baru untuk route yang sama
4. **Testability**: Lebih mudah untuk di-test secara terpisah

---

## Extra Credit: Jawaban

### Pertanyaan
> What happens if you go to the URL `http://localhost:6543/howdy`? Is this the result that you expected?

### Jawaban

#### Hasil yang Didapatkan
Ketika mengakses URL `http://localhost:6543/howdy` (tanpa parameter `{first}` dan `{last}`), aplikasi akan menampilkan:

```
404 Not Found
The resource at /howdy could not be found.
```

#### Penjelasan

Hal ini terjadi karena:

1. **Pattern Matching Fail**: Pola route yang didefinisikan adalah `/howdy/{first}/{last}` yang **memerlukan exactly 2 parameter** (first dan last)

2. **URL `/howdy` tidak cocok** dengan pola tersebut karena:
   - `/howdy` hanya memiliki 1 segmen setelah `/howdy/`
   - Pola memerlukan format `/howdy/[nilai]/[nilai]`

3. **No Route Handler**: Pyramid tidak menemukan route yang cocok, sehingga tidak ada view yang dipanggil, menghasilkan HTTP 404

#### Apakah Ini Hasil yang Diharapkan?

**Ya, ini adalah hasil yang diharapkan** karena:

- Route pattern didefinisikan secara **eksplisit** dan **strict**
- Tidak ada fallback atau default route untuk `/howdy`
- Pyramid tidak melakukan "guessing" - jika pattern tidak cocok, request akan gagal
- Ini adalah design philosophy Pyramid: **explicit is better than implicit**

#### Cara Menangani Variasi URL

Jika ingin menangani berbagai variasi URL, ada beberapa opsi:

**Opsi 1: Membuat route dengan parameter opsional**
```python
config.add_route('home', '/howdy')  # hanya untuk /howdy
config.add_route('home_with_params', '/howdy/{first}/{last}')
```

**Opsi 2: Menggunakan catch-all route (dengan regex)**
```python
config.add_route('home', '/howdy/*traverse')
```

**Opsi 3: Membuat parameter opsional dengan wildcard**
```python
config.add_route('home', '/howdy/{first:.*}/{last:.*}')
```

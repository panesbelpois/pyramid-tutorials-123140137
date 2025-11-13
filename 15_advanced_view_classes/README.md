# 15: More With View Classes

## Deskripsi

Tutorial ini mengajarkan cara mengelompokkan tampilan (views) terkait dalam sebuah kelas, berbagi konfigurasi, status, dan logika. Pyramid menyediakan fitur-fitur canggih untuk bekerja dengan view classes yang memungkinkan organisasi kode lebih baik, terutama untuk aplikasi web yang lebih kompleks.

### Konsep Utama

Dalam Pyramid, view adalah "callable" Python yang dapat berbentuk:
- Fungsi standalone
- Objek dengan method `__call__`
- Kelas Python dengan method yang didekorasi `@view_config`

Mengelompokkan views yang berhubungan dalam satu kelas memberikan beberapa manfaat:
- **Organisasi kode**: Mengelompokkan operasi yang terkait pada data yang sama
- **Konfigurasi terpusat**: Menggunakan `@view_defaults` untuk mengurangi duplikasi
- **Berbagi state**: Instance variables dan helper methods dapat digunakan bersama

### Objectives

- Mengelompokkan views yang berhubungan ke dalam satu view class
- Memusatkan konfigurasi dengan `@view_defaults` di level class
- Mengirim satu route/URL ke multiple views berdasarkan data request
- Berbagi state dan logic antara views dan templates melalui view class

---

## Instalasi

### Prasyarat

- Python 3.6+
- Virtual environment (venv) sudah dikonfigurasi
- Pyramid dan dependencies sudah diinstal

### Langkah-langkah Instalasi

1. **Masuk ke direktori project**:
   ```bash
   cd more_view_classes
   ```

2. **Install package dalam mode development**:
   ```bash
   $VENV/bin/pip install -e .
   ```

3. **Verifikasi instalasi** dengan menjalankan tests:
   ```bash
   $VENV/bin/pytest tutorial/tests.py -q
   ```

   Output yang diharapkan:
   ```
   ..
   2 passed in 0.40 seconds
   ```

4. **Jalankan aplikasi**:
   ```bash
   $VENV/bin/pserve development.ini --reload
   ```

5. **Buka browser** dan navigasi ke:
   - Home page: http://localhost:6543/
   - Hello view: http://localhost:6543/howdy/jane/doe

---

## Analisis

### Struktur View Class

Tutorial ini mendemonstrasikan 4 views yang dikelompokkan dalam satu kelas `TutorialViews`:

#### 1. **Home View** (GET /)
- Menampilkan halaman utama dengan link ke form
- Route: `home`
- Renderer: `home.pt`

#### 2. **Hello View** (GET /howdy/{first}/{last})
- Menampilkan sapaan dengan nama depan dan belakang
- Route: `hello` (didefinisikan via `@view_defaults`)
- Renderer: `hello.pt`
- Fitur: Menggunakan `@property` untuk computed value `full_name`

#### 3. **Edit View** (POST /howdy/{first}/{last})
- Menampilkan nama baru yang disubmit via form
- Dipicu saat tombol "Save" ditekan
- Renderer: `edit.pt`

#### 4. **Delete View** (POST /howdy/{first}/{last} dengan request_param)
- Menampilkan konfirmasi delete
- Dipicu saat tombol "Delete" ditekan
- Menggunakan `request_param='form.delete'` sebagai discriminator

### Mekanisme View Dispatch

Pyramid menggunakan **view predicates** untuk menentukan view mana yang akan dipilih berdasarkan:

1. **HTTP Request Method**: GET, POST, PUT, DELETE, etc.
2. **Route**: Path pattern dan route name
3. **Request Parameters**: Nama-nama form field yang disubmit
4. **Content Type**: Media type request

Dalam contoh ini:

```python
@view_defaults(route_name='hello')
class TutorialViews:
    # Ini menggunakan route default 'hello'
    @view_config(renderer='hello.pt')
    def hello(self):
        # GET /howdy/jane/doe
        pass
    
    # Ini override default route
    @view_config(route_name='home', renderer='home.pt')
    def home(self):
        # GET /
        pass
    
    # POST dengan kondisi tambahan
    @view_config(request_method='POST', renderer='edit.pt')
    def edit(self):
        # POST /howdy/jane/doe (tanpa form.delete)
        pass
    
    @view_config(request_method='POST', request_param='form.delete', renderer='delete.pt')
    def delete(self):
        # POST /howdy/jane/doe (dengan form.delete)
        pass
```

### Berbagi State dan Logic

#### Inisialisasi dalam `__init__`

```python
def __init__(self, request):
    self.request = request
    self.view_name = 'TutorialViews'
```

Setiap instance view memiliki akses ke request object dan state yang di-share.

#### Computed Properties

```python
@property
def full_name(self):
    first = self.request.matchdict['first']
    last = self.request.matchdict['last']
    return first + ' ' + last
```

Property ini dapat diakses:
- Dalam view methods: `self.full_name`
- Dalam templates: `${view.full_name}`

### URL Generation

Menggunakan dynamic URL generation daripada hardcoding:

**Sebelumnya** (tidak fleksibel):
```html
<a href="/howdy/jane/doe">Howdy</a>
```

**Sesudahnya** (fleksibel):
```html
<a href="${request.route_url('hello', first='jane', last='doe')}">form</a>
```

Keuntungan:
- Perubahan route patterns tidak memerlukan update template
- Type-safe routing
- Automatic parameter validation

---

## Jawaban Extra Credit

### 1. **Why could our template do `${view.full_name}` and not have to do `${view.full_name()}`?**

**Jawaban**: Python properties adalah descriptors yang secara otomatis memanggil method getter ketika atribut diakses. Ketika template engine (Chameleon dalam hal ini) mengakses `view.full_name`, Python secara otomatis memanggil property getter `full_name(self)` tanpa memerlukan explicit function call.

Chameleon smart enough untuk mendeteksi dan memanggil callables. Karena `@property` membuat method terlihat seperti atribut biasa, akses tanpa `()` sudah cukup.

```python
@property
def full_name(self):
    # Ini dipanggil secara otomatis saat diakses sebagai atribut
    return first + ' ' + last

# Di template:
${view.full_name}  # ✓ Bekerja - property memanggil getter
${view.full_name()} # ✗ Akan error - property bukan callable
```

---

### 2. **The edit and delete views both receive POST requests. Why does the edit view configuration not catch the POST used by delete?**

**Jawaban**: Pyramid menggunakan **specificity-based matching** untuk view predicates. View dengan predicates lebih spesifik memiliki prioritas lebih tinggi daripada yang lebih umum.

```python
@view_config(request_method='POST', renderer='edit.pt')
def edit(self):
    # Matches: POST requests TANPA form.delete parameter
    pass

@view_config(request_method='POST', request_param='form.delete', renderer='delete.pt')
def delete(self):
    # Matches: POST requests DENGAN form.delete parameter
    # Lebih SPESIFIK - priority lebih tinggi!
    pass
```

Urutan matching:
1. POST dengan `form.delete=Delete` → Memenuhi kondisi `delete()` yang lebih spesifik ✓
2. POST tanpa `form.delete` → Hanya memenuhi kondisi `edit()` ✓

Tanpa parameter `request_param`, `edit()` akan intercept SEMUA POST requests. Parameter tambahan membuat predicates lebih spesifik dan memberikan Pyramid cara untuk membedakan antara submit buttons yang berbeda.

---

### 3. **We used Python `@property` on full_name. If we reference this many times in a template or view code, it would re-compute this every time. Does Pyramid provide something that will cache the initial computation on a property?**

**Jawaban**: Pyramid sendiri tidak menyediakan built-in caching untuk properties, tetapi ada beberapa solusi:

#### **A. Menggunakan `functools.cached_property` (Python 3.8+)**

```python
from functools import cached_property

class TutorialViews:
    @cached_property
    def full_name(self):
        first = self.request.matchdict['first']
        last = self.request.matchdict['last']
        return first + ' ' + last
```

Keuntungan: Computed sekali per instance, cached di memory.

#### **B. Manual Caching dalam `__init__`**

```python
def __init__(self, request):
    self.request = request
    self.view_name = 'TutorialViews'
    # Cache computed value
    first = self.request.matchdict.get('first', '')
    last = self.request.matchdict.get('last', '')
    self._full_name = first + ' ' + last

@property
def full_name(self):
    return self._full_name
```

#### **C. Menggunakan Pyramid's `reify` decorator**

```python
from pyramid.decorator import reify

class TutorialViews:
    @reify
    def full_name(self):
        first = self.request.matchdict['first']
        last = self.request.matchdict['last']
        return first + ' ' + last
```

`reify` adalah decorator khusus Pyramid yang caching hasilnya dalam `__dict__` instance, menjadikan akses subsequent O(1).

**Rekomendasi**: Gunakan `@reify` untuk Pyramid apps, atau `@cached_property` untuk Python 3.8+.

---

### 4. **Can you associate more than one route with the same view?**

**Jawaban**: Ya, ada beberapa cara:

#### **A. Dekorasi Multiple dengan `@view_config`**

```python
@view_config(route_name='hello', renderer='hello.pt')
@view_config(route_name='goodbye', renderer='hello.pt')
def greet(self):
    return {'message': 'Hello or Goodbye'}
```

#### **B. Menggunakan `@view_defaults` untuk default, override untuk exceptions**

```python
@view_defaults(route_name='hello')
class TutorialViews:
    @view_config(renderer='hello.pt')
    def hello(self):
        # route_name='hello' dari defaults
        pass
    
    @view_config(route_name='goodbye', renderer='hello.pt')
    def goodbye(self):
        # route_name='goodbye' override defaults
        pass
```

#### **C. Mendaftar route yang sama ke multiple URL patterns**

```python
# Dalam __init__.py
def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.add_route('greet', '/hello')
    config.add_route('greet', '/hey')  # Same route, different URL
    config.add_route('greet', '/howdy')
    return config.make_wsgi_app()
```

Semua URL akan di-route ke view yang sama.

**Best Practice**: Gunakan `@view_config` multiple decorator untuk view method yang berbeda dengan logic yang berbeda tapi related.

---

### 5. **There is also a `request.route_path` API. How does this differ from `request.route_url`?**

**Jawaban**: Kedua API menghasilkan URL/path untuk named routes, tapi dengan perbedaan:

| Fitur | `route_url()` | `route_path()` |
|-------|-------------|-----------------|
| **Output** | Full URL dengan scheme dan host | Hanya path component |
| **Contoh Output** | `http://localhost:6543/howdy/jane/doe` | `/howdy/jane/doe` |
| **Gunakan saat** | Redirect, absolute URLs, cross-domain | Links dalam HTML, relative URLs |
| **Query String** | `route_url('hello', first='jane', last='doe', _query={'page': 2})` | `route_path('hello', first='jane', last='doe', _query={'page': 2})` |

#### **Perbandingan Praktis**

```html
<!-- Dalam template -->

<!-- route_path(): lebih cocok untuk internal links -->
<a href="${request.route_path('hello', first='jane', last='doe')}">Link</a>
<!-- Output: href="/howdy/jane/doe" -->

<!-- route_url(): full URL untuk redirect atau API -->
<meta property="og:url" content="${request.route_url('hello', first='jane', last='doe')}" />
<!-- Output: content="http://localhost:6543/howdy/jane/doe" -->
```

#### **Dalam View Code**

```python
def edit(self):
    new_name = self.request.params['new_name']
    
    # Redirect ke URL absolut
    raise HTTPFound(self.request.route_url('hello', first='jane', last='doe'))
    
    # Atau generate path untuk template
    next_path = self.request.route_path('home')
```

**Rekomendasi**: 
- Gunakan `route_path()` dalam HTML templates untuk relative links
- Gunakan `route_url()` untuk HTTP redirects, API responses, dan social media meta tags

---

## Kesimpulan

Tutorial ini mendemonstrasikan power dari view classes dalam Pyramid:

1. ✅ **Organisasi**: Logically related views dalam satu kelas
2. ✅ **DRY**: `@view_defaults` menghilangkan duplikasi konfigurasi
3. ✅ **Flexibility**: View predicates memungkinkan sophisticated routing logic
4. ✅ **Code Reuse**: Shared state dan properties meningkatkan maintainability
5. ✅ **Dynamic URLs**: Route generation yang fleksibel dan maintainable

Dengan memahami konsep-konsep ini, Anda dapat membangun aplikasi Pyramid yang lebih scalable dan well-organized.

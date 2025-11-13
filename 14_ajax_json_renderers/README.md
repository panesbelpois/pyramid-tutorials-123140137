# 14: AJAX Development With JSON Renderers

## Deskripsi

Tutorial ini mendemonstrasikan cara mengembangkan aplikasi web modern yang menggunakan AJAX dengan JSON renderers di Pyramid. Aplikasi web modern tidak hanya menampilkan HTML yang di-render, melainkan menggunakan JavaScript untuk memperbarui UI di browser dengan meminta data server dalam format JSON. Pyramid menyediakan JSON renderer bawaan yang memudahkan proses ini.

Dengan menggunakan JSON renderer, kita dapat:
- Mengembalikan data Python yang secara otomatis di-serialize menjadi JSON
- Memisahkan logika view dari templating
- Mendukung AJAX requests dengan mudah
- Membuat endpoint yang dapat mengembalikan data dalam format JSON

## Konsep Dasar

### Renderer di Pyramid
Dalam Pyramid, renderer adalah komponen yang mengubah output view menjadi response yang siap dikirim. Sebelumnya kita telah menggunakan:
- **Chameleon renderer**: untuk menghasilkan HTML
- **Jinja2 renderer**: untuk templating dengan Jinja2
- **JSON renderer**: untuk mengubah data Python menjadi JSON

### Keuntungan Data-Oriented View Layer
Dengan mengubah view menjadi mengembalikan data Python (bukan string HTML langsung), kita mendapatkan:
- Pemisahan concern yang lebih baik
- Testing yang lebih mudah
- Fleksibilitas dalam output format (HTML atau JSON)

---

## Instalasi

### Prasyarat
- Python 3.x
- Virtual Environment (venv)
- pip

### Langkah-Langkah Instalasi

1. **Navigasi ke direktori project**
   ```bash
   cd 14_ajax_json_renderers/json
   ```

2. **Install project dalam development mode**
   ```bash
   $VENV/bin/pip install -e .
   ```

3. **Verifikasi instalasi**
   ```bash
   $VENV/bin/pip list
   ```

### File Konfigurasi Penting
- `setup.py`: File konfigurasi project
- `development.ini`: File konfigurasi Pyramid untuk development

---

## Implementasi

### 1. Tambah Route untuk JSON Endpoint

Di `tutorial/__init__.py`, tambahkan route baru:
```python
from pyramid.config import Configurator

def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_chameleon')
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.add_route('hello_json', '/howdy.json')  # Route baru untuk JSON
    config.scan('.views')
    return config.make_wsgi_app()
```

### 2. Stack Decorator pada View

Di `tutorial/views.py`, gunakan multiple `@view_config` decorators:
```python
from pyramid.view import view_config, view_defaults

@view_defaults(renderer='home.pt')
class TutorialViews:
    def __init__(self, request):
        self.request = request

    @view_config(route_name='home')
    def home(self):
        return {'name': 'Home View'}

    @view_config(route_name='hello')
    @view_config(route_name='hello_json', renderer='json')  # Renderer JSON
    def hello(self):
        return {'name': 'Hello View'}
```

### 3. Testing

Jalankan unit tests:
```bash
$VENV/bin/pytest tutorial/tests.py -q
```

Expected output:
```
.....
5 passed in 0.47 seconds
```

### 4. Menjalankan Application

```bash
$VENV/bin/pserve development.ini --reload
```

Buka browser ke `http://localhost:6543/howdy.json` untuk melihat JSON response.

---

## Analisis

### Pola Decorator Stacking

Pyramid memungkinkan kita untuk "stack" multiple decorators pada satu method view. Teknik ini sangat powerful karena:

1. **Code Reuse**: Satu method dapat melayani multiple routes
2. **Flexibility**: Setiap decorator dapat memiliki renderer berbeda
3. **Maintainability**: Logika view terpusat, hanya renderer yang berbeda

```python
@view_config(route_name='hello')                              # HTML
@view_config(route_name='hello_json', renderer='json')        # JSON
def hello(self):
    return {'name': 'Hello View'}
```

### Data-Oriented View Layer

Perubahan paradigma dari templating langsung ke data-oriented approach memberikan keuntungan:

| Aspek | Sebelum | Sesudah |
|-------|---------|--------|
| Output View | String HTML | Python Dictionary |
| Testing | Perlu parse HTML | Mudah cek dictionary |
| Reusability | Terikat renderer | Bisa multiple renderer |
| Decoupling | View + Template | View dan Renderer terpisah |

### JSON Renderer

Pyramid JSON renderer menggunakan Python's base JSON encoder, sehingga:
- **Kelebihan**: Sederhana, cepat, built-in
- **Keterbatasan**: 
  - Tidak bisa encode DateTime objects secara native
  - Tidak bisa encode custom objects
  
**Solusi**: Extend JSON renderer dengan custom encoder

```python
# Contoh custom encoder untuk DateTime
def datetime_handler(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    raise TypeError(f"Object of type {type(obj)} is not JSON serializable")
```

### Content Type Handling

JSON renderer secara otomatis mengatur:
- `Content-Type: application/json`
- Serialization dari Python dict ke JSON string

Ini terlihat dalam test:
```python
def test_hello_json(self):
    res = self.testapp.get('/howdy.json', status=200)
    self.assertIn(b'{"name": "Hello View"}', res.body)
    self.assertEqual(res.content_type, 'application/json')
```

### Best Practices AJAX dengan Pyramid

1. **Gunakan Route Names**: Terpisahkan dari path actual
2. **Return Data dari Views**: Jangan hardcode HTML di view
3. **Leverage View Predicates**: Bisa menggunakan `Accepts:` header untuk content negotiation
4. **Extend Renderers**: Untuk kebutuhan custom serialization

### View Predicates untuk Content Negotiation

Daripada membuat route terpisah, bisa menggunakan view predicate:
```python
@view_config(route_name='hello', renderer='json', 
             accept='application/json')
@view_config(route_name='hello', renderer='home.pt', 
             accept='text/html')
def hello(self):
    return {'name': 'Hello View'}
```

---

## Kesimpulan

Tutorial ini menunjukkan bagaimana Pyramid mempermudah development AJAX modern dengan:
1. JSON renderer built-in
2. Flexibility dalam stacking decorators
3. Data-oriented view layer untuk better testability
4. Separation of concerns antara view logic dan rendering

Dengan pemahaman ini, developers dapat membangun aplikasi web yang responsif dengan mudah memanfaatkan Pyramid framework.

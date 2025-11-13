# 05: Functional Testing dengan WebTest pada Pyramid Framework


## Deskripsi

Proyek ini merupakan contoh implementasi **functional testing** menggunakan WebTest untuk aplikasi berbasis Pyramid Framework. Functional testing memverifikasi perilaku aplikasi secara menyeluruh melalui simulasi HTTP request dan validasi response yang dihasilkan. Proyek ini mendemonstrasikan:

- Setup aplikasi Pyramid yang sederhana dengan satu endpoint (`/`)
- Unit test menggunakan `pyramid.testing` untuk testing view
- Functional test menggunakan `webtest.TestApp` untuk testing HTTP request/response
- Verifikasi status code HTTP dan konten response

---

## Instalasi

### Prasyarat
- Python 3.x
- pip package manager

### Langkah-langkah Instalasi

1. **Navigasi ke direktori proyek:**
   ```bash
   cd functional_testing
   ```

2. **Install dependencies utama:**
   ```bash
   pip install -e .
   ```

3. **Install development dependencies (termasuk WebTest dan pytest):**
   ```bash
   pip install -e ".[dev]"
   ```

4. **Verifikasi instalasi:**
   ```bash
   pytest tutorial/tests.py -v
   ```

### Dependencies
- `pyramid`: Framework web Python
- `waitress`: WSGI server untuk production
- `pyramid_debugtoolbar`: Development toolbar (dev)
- `pytest`: Test runner (dev)
- `webtest`: HTTP testing library (dev)

---

## Analisis

### Struktur Kode

**`tutorial/app.py`** - Aplikasi Utama
- Mendefinisikan view function `hello_world()` yang mengembalikan HTML response
- Mengkonfigurasi Pyramid dengan satu route: `GET /`
- Function `main()` adalah WSGI application factory

**`tutorial/tests.py`** - Test Suite
- `TutorialViewTests`: Unit test untuk view function
  - `setUp()`: Inisialisasi Pyramid testing configuration
  - `test_hello_world()`: Memverifikasi response dari view function
- `TutorialFunctionalTests`: Functional test untuk HTTP request/response
  - `setUp()`: Membuat WSGI app dan membuat TestApp wrapper
  - `test_hello_world()`: Simulasi GET request ke `/` dan verifikasi konten

### Perbedaan Unit Test vs Functional Test

| Aspek | Unit Test | Functional Test |
|-------|-----------|-----------------|
| **Scope** | Test function individual | Test aplikasi end-to-end |
| **Tools** | `pyramid.testing.DummyRequest` | `webtest.TestApp` |
| **Simulasi HTTP** | Tidak, langsung call function | Ya, simulasi request HTTP |
| **Kegunaan** | Test logic bisnis | Verifikasi perilaku aplikasi |

### Test Execution Flow

```
1. setUp() → Inisialisasi Pyramid config / Create TestApp
2. test_hello_world() → Kirim GET request ke '/'
3. Verifikasi status code 200 (OK)
4. Verifikasi response body mengandung '<h1>Hello World!</h1>'
5. tearDown() → Cleanup resources
```

---

## Extra Credit

### Q: Mengapa menggunakan WebTest daripada requests library?

**A:** WebTest lebih sesuai untuk testing aplikasi WSGI karena:
1. **Direct WSGI Access**: WebTest langsung memanggil aplikasi WSGI tanpa network overhead
2. **Testing Tools**: Memiliki API yang dirancang khusus untuk HTTP testing (assertion helpers)
3. **Pyramid Integration**: Terintegrasi sempurna dengan Pyramid testing utilities
4. **Performance**: Lebih cepat karena tidak perlu menjalankan server HTTP terpisah

### Q: Apa keuntungan functional testing untuk web application?

**A:** Functional testing memberikan:
1. **Confidence**: Memverifikasi aplikasi berfungsi dari perspektif user
2. **Integration**: Mendeteksi bugs pada integrasi antar components
3. **Real Scenarios**: Mensimulasikan real HTTP interactions
4. **Regression Prevention**: Mengkatch bugs ketika ada perubahan kode
# Tutorial Pyramid Framework - Pengembangan Aplikasi Web


- **Nama**: Anisah Octa Rohila
- **NIM**: 123140137
- **Kelas**: Pengembangan Aplikasi Web RA

---

## Penjelasan Tugas

Repositori ini berisi **tutorial** untuk mempelajari framework web **Pyramid** secara progresif, dari konsep dasar hingga implementasi lanjutan. Setiap folder berisi:

1. **README.md** — Penjelasan materi, instalasi, analisis kode, dan jawaban pertanyaan extra credit
2. **Contoh kode** — File Python siap dijalankan untuk mempraktikkan konsep
3. **Virtual environment** — Setup dan dependensi yang diperlukan

### Kompetensi yang Dipelajari

- **Microframework**: Cara membuat aplikasi web dari single-file hingga multi-module
- **WSGI & Server Web**: Memahami standar WSGI dan server Waitress
- **Routing & Views**: Memetakan URL ke views dan menangani request/response
- **Templating**: Menggunakan Chameleon dan Jinja2 untuk rendering HTML
- **Static Assets**: Mengelola CSS, JavaScript, dan file statis lainnya
- **Testing**: Unit testing dengan pytest dan functional testing dengan WebTest
- **Database**: Integrasi SQLAlchemy untuk ORM dan manajemen database
- **Authentication & Authorization**: Implementasi login, security, dan ACL
- **Advanced Topics**: Session management, logging, form validation, dan AJAX

---

### Bagian 1: Dasar-Dasar Pyramid

1. **[01_single_file_web_app](https://github.com/panesbelpois/pyramid-tutorials-123140137/tree/main/01_single_file_web_app)**
   - Cara paling sederhana untuk memulai dengan Pyramid
   - Single-file module tanpa paket Python
   - Pengenalan WSGI, views, dan responses

2. **[02_python_packages](https://github.com/panesbelpois/pyramid-tutorials-123140137/tree/main/02_python_packages)**
   - Struktur paket Python profesional
   - Instalasi mode development dengan `pip install -e .`
   - Organisasi kode yang scalable

3. **[03_app_configuration_ini](https://github.com/panesbelpois/pyramid-tutorials-123140137/tree/main/03_app_configuration_ini)**
   - Pemisahan konfigurasi dari kode
   - File `.ini` untuk settings aplikasi
   - Perintah `pserve` untuk menjalankan aplikasi

4. **[04_debugtoolbar](https://github.com/panesbelpois/pyramid-tutorials-123140137/tree/main/04_debugtoolbar)**
   - Debug toolbar untuk inspeksi request/response
   - Analisis performa aplikasi
   - Development dependencies dengan setuptools extras

5. **[05_unit_tests_pytest](https://github.com/panesbelpois/pyramid-tutorials-123140137/tree/main/05_unit_tests_pytest)**
   - Unit testing dengan pytest framework
   - Test organization dan best practices
   - Coverage reporting untuk code quality

6. **[06_functional_testing_webtest](https://github.com/panesbelpois/pyramid-tutorials-123140137/tree/main/06_functional_testing_webtest)**
   - Functional testing dengan WebTest
   - Simulasi HTTP requests dan assertions
   - Integration testing untuk workflow aplikasi

### Bagian 2: Views & Routing

7. **[07_basic_web_views](https://github.com/panesbelpois/pyramid-tutorials-123140137/tree/main/07_basic_web_views)**
   - Dasar-dasar view functions
   - Decorator `@view_config` untuk routing
   - Pemisahan logika ke `views.py`

8. **[08_html_templating](https://github.com/panesbelpois/pyramid-tutorials-123140137/tree/main/08_html_templating)**
   - HTML templating dengan Chameleon
   - Pemisahan presentation logic dari business logic
   - Template inheritance dan reusable components

9. **[09_view_classes](https://github.com/panesbelpois/pyramid-tutorials-123140137/tree/main/09_view_classes)**
   - Mengorganisir views dengan class-based views
   - `@view_defaults` untuk sharing configuration
   - Stateful views dan property management

### Bagian 3: Request & Response Handling

10. **[10_requests_responses](https://github.com/panesbelpois/pyramid-tutorials-123140137/tree/main/10_requests_responses)**
    - Objek request dan response di Pyramid
    - Akses parameters dan headers
    - Manipulasi response untuk output yang custom

11. **[11_routing](https://github.com/panesbelpois/pyramid-tutorials-123140137/tree/main/11_routing)**
    - URL routing dan pattern matching
    - Ekstraksi parameter dari URL
    - Route ordering dan ambiguity resolution

### Bagian 4: Templating Engine

12. **[12_templating_jinja2](https://github.com/panesbelpois/pyramid-tutorials-123140137/tree/main/12_templating_jinja2)**
    - Jinja2 templating engine
    - Perbandingan Jinja2 vs Chameleon
    - Advanced Jinja2 features (filters, macros, inheritance)

13. **[13_static_assets](https://github.com/panesbelpois/pyramid-tutorials-123140137/tree/main/13_static_assets)**
    - Pengelolaan CSS, JavaScript, dan images
    - `add_static_view()` untuk serving assets
    - `request.static_url()` helper untuk versioning

### Bagian 5: API & AJAX

14. **[14_ajax_json_renderers](https://github.com/panesbelpois/pyramid-tutorials-123140137/tree/main/14_ajax_json_renderers)**
    - JSON renderer untuk AJAX endpoints
    - RESTful API responses
    - Custom serialization untuk complex objects

15. **[15_advanced_view_classes](https://github.com/panesbelpois/pyramid-tutorials-123140137/tree/main/15_advanced_view_classes)**
    - Advanced class-based view patterns
    - Predicate matching untuk flexible routing
    - Composition over inheritance

## Cara Menggunakan Repository

### Kloning Repository

```bash
git clone https://github.com/panesbelpois/pyramid-tutorials-123140137.git
cd pyramid-tutorials-123140137
```

### Menjalankan Tutorial Spesifik

1. Masuk ke folder tutorial yang ingin dipelajari:
   ```bash
   cd 01_single_file_web_app
   ```

2. Buat virtual environment:
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Linux/macOS
   .\.venv\Scripts\Activate.ps1  # Windows PowerShell
   ```

3. Pasang dependensi:
   ```bash
   pip install -r requirements.txt
   # atau untuk folder dengan setup.py:
   pip install -e .
   ```

4. Jalankan aplikasi:
   ```bash
   python app.py
   # atau dengan pserve (untuk folder dengan .ini):
   pserve development.ini
   ```

5. Buka browser dan kunjungi `http://localhost:6543/`

### Struktur Folder

Setiap folder tutorial memiliki struktur serupa:

```
XX_folder_name/
├── README.md              # Penjelasan, instalasi, analisis, extra credit
├── app.py                 # Aplikasi single-file (untuk tutorial awal)
├── setup.py               # Package configuration (untuk tutorial lanjutan)
├── development.ini        # Configuration file (untuk tutorial dengan pserve)
├── requirements.txt       # Python dependencies
└── tutorial/              # Package folder dengan modules
    ├── __init__.py
    ├── views.py
    ├── models.py
    └── templates/
```

---

## Prasyarat

- **Python 3.8+** (versi terbaru direkomendasikan)
- **pip** dan **virtualenv** (usually included dengan Python)
- **Git** untuk cloning repository
- **Text editor** atau **IDE** (VS Code, PyCharm, dll.)

---

## Tips Pembelajaran

1. **Ikuti urutan**: Tutorial dirancang progresif dari dasar hingga advanced
2. **Pelajari setiap folder**: Baca README.md dan pahami contoh kode
3. **Eksperimen**: Modifikasi kode dan lihat hasilnya
4. **Jalankan tests**: Menjalankan unit tests untuk verify understanding
5. **Dokumentasi**: Baca docs Pyramid resmi di https://docs.pylonsproject.org/

---

## Referensi & Resources

- **Pyramid Documentation**: https://docs.pylonsproject.org/projects/pyramid/
- **Python WSGI Standard**: https://www.python.org/dev/peps/pep-3333/
- **GitHub Repository**: https://github.com/panesbelpois/pyramid-tutorials-123140137
- **Pyramid Cookbook**: https://docs.pylonsproject.org/projects/pyramid_cookbook/
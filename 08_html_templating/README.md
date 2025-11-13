
# 08: HTML Generation With Templating

## Deskripsi

Langkah ini menunjukkan cara menghasilkan HTML menggunakan sistem templating di Pyramid. Alih-alih menyisipkan HTML langsung di dalam fungsi Python, view akan mengembalikan data (struktur dictionary) dan Pyramid akan menghubungkan data tersebut ke file template (Chameleon, Mako, Jinja2, dll.) yang menghasilkan HTML akhir.

Contoh yang ada pada proyek ini menggunakan `pyramid_chameleon` (Chameleon) sehingga file template berada di `tutorial/home.pt`. View di `tutorial/views.py` hanya mengembalikan data seperti `{'name': 'Home View'}` dan renderer `home.pt` yang membuat markup HTML.

File penting dalam folder ini:

- `setup.py` : mendefinisikan dependensi paket (`pyramid`, `pyramid_chameleon`, `waitress`, dll.).
- `development.ini` : konfigurasi untuk menjalankan aplikasi selama pengembangan (termasuk `pyramid.reload_templates`).
- `tutorial/__init__.py` : fungsi `main` yang mendaftarkan `pyramid_chameleon` dan route aplikasi.
- `tutorial/views.py` : view yang mengembalikan data untuk template.
- `tutorial/home.pt` : file template Chameleon yang menghasilkan HTML.
- `tutorial/tests.py` : unit dan functional tests yang memverifikasi perilaku data dan output HTML.

## Instalasi

Berikut langkah cepat menyiapkan lingkungan development di Windows (PowerShell):

1. Buat dan aktifkan virtual environment (jika belum):

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

2. Instal paket local (editable) dan dependensi development:

```powershell
pip install -e .
pip install -e .[dev]
```

Catatan: `setup.py` di proyek ini sudah mencantumkan `pyramid_chameleon` di `install_requires` dan `pyramid_debugtoolbar`, `pytest`, `webtest` di `extras_require['dev']`.

3. Menjalankan test unit dan functional:

```powershell
# Menjalankan test spesifik
pytest tutorial/tests.py -q
```

Jika konfigurasi benar, Anda akan melihat output seperti `4 passed`.

4. Menjalankan aplikasi untuk development:

```powershell
pserve development.ini --reload
```

Lalu buka:

- `http://localhost:6543/`
- `http://localhost:6543/howdy`

## Analisis

- **Pemecahan tanggung jawab (Separation of Concerns)**: Dengan templating, logika Python (menyiapkan data) dipisahkan dari presentasi (HTML). View menjadi lebih sederhana dan fokus pada data.
- **Renderer**: `@view_config(..., renderer='home.pt')` menghubungkan view ke template. View mengembalikan dictionary data saja; Pyramid menangani proses rendering.
- **Pengujian lebih mudah**: Unit test dapat memeriksa kontrak data dari view (mis. `response['name'] == 'Home View'`) tanpa terikat markup. Functional test memverifikasi HTML final di endpoint yang dipasang.
- **Konfigurasi pengembangan**: `pyramid.reload_templates = true` dalam `development.ini` memungkinkan perubahan template terlihat tanpa restart server, mempercepat iterasi UI.
- **Fleksibilitas templating**: Pyramid tidak memaksakan engine templating tertentu. Proyek ini menggunakan Chameleon via `pyramid_chameleon`, namun Anda bisa mengganti dengan `pyramid_jinja2` atau `mako` sesuai preferensi.
- **Reuse template**: Contoh memakai satu template (`home.pt`) untuk dua view yang berbeda, cukup dengan mengirimkan data berbeda — ini mengurangi duplikasi markup.

## Saran pengembangan selanjutnya

- Pertimbangkan menambahkan layout/base template untuk header/footer bersama.
- Tambahkan caching jika rendering menjadi bottleneck pada produksi.
- Ganti atau tambahkan engine templating (mis. Jinja2) bila membutuhkan fitur templating spesifik.

Jika ingin, saya dapat:

- Menjalankan `pytest` di workspace ini dan melaporkan hasilnya.
- Menambahkan contoh `README` singkat untuk deployment (Waitress/production).

---

Dokumentasi ini ditulis singkat agar mudah dipakai sebagai petunjuk cepat untuk langkah ke-08 tutorial Pyramid: HTML Generation With Templating.


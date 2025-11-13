
**07: Basic Web Handling With Views**

**Deskripsi:**
- **Ringkasan:** Modul ini menata views terpisah di dalam paket `tutorial`. Views didaftarkan secara deklaratif memakai dekorator `@view_config` dan di-scan oleh `Configurator` melalui `config.scan('.views')`. Contoh menyediakan dua view sederhana: `home` (route `/`) dan `hello` (route `/howdy`) yang saling menunjuk satu sama lain.

**Instalasi:**
- **Persyaratan:** Python 3.x dan `pip`.
- **Langkah singkat (PowerShell):**

```powershell
python -m venv .venv
. .venv\Scripts\Activate.ps1
pip install -e .
pip install pytest webtest
python -m pytest tutorial/tests.py -q
```

- **Menjalankan aplikasi:** setelah mengaktifkan virtualenv, jalankan:

```powershell
pserve development.ini --reload
# lalu buka http://localhost:6543/ dan http://localhost:6543/howdy
```

**Analisis:**
- **Pemecahan kode startup:** Kode pendaftaran view dipindah dari startup (`tutorial/__init__.py`) ke `tutorial/views.py`. Startup kini cukup menambahkan route dan memanggil `config.scan('.views')` sehingga dekorator `@view_config` pada modul views akan dieksekusi dan mendaftarkan view.
- **Deklaratif vs Imperatif:** Dekorator `@view_config` memberikan konfigurasi deklaratif di atas definisi fungsi view, sedangkan `config.add_view` adalah cara imperatif yang setara. Pilihan di antara keduanya biasanya soal gaya dan organisasi kode.
- **Perilaku URL/Route:** Nama route, nama view, dan nama yang muncul di URL bisa berbeda — routing memetakan URL ke view berdasarkan nama route.

**Extra Credit:**
- **Apa arti titik pada `.views`?**: Titik berarti modul relatif dalam paket saat ini — `.views` menunjuk ke `tutorial.views` ketika dipanggil dari dalam paket `tutorial`. `config.scan` menerima nama modul bertitik untuk diimpor dan dipindai.
- **Mengapa `assertIn` sering lebih baik daripada `assertEqual` pada pengujian teks?**: `assertIn` memeriksa bahwa substring penting ada di dalam respons tanpa membutuhkan kecocokan persis pada seluruh isi. Ini lebih tahan terhadap perubahan format HTML, whitespace, atau tambahan konten lain; sehingga tes lebih robust. Juga respons sering berupa bytes, sehingga biasanya kita cek keberadaan substring bytes tertentu (mis. `b'Visit'`).

**Perubahan file:**
- Diperbarui: `README.md` (isi deskripsi, instalasi, analisis, extra credit)

Jika Anda mau, saya dapat menjalankan `pytest` di lingkungan Anda atau membuat commit untuk perubahan ini.


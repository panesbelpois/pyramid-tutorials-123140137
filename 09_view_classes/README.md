# 09: Organizing Views With View Classes

Deskripsi
---------

Langkah ini menunjukkan cara mengorganisir view di aplikasi Pyramid dengan menggunakan "view classes". Alih-alih menulis view sebagai fungsi bebas, kita memindahkan view tersebut menjadi metode pada sebuah kelas view. Tujuannya:

- Mengelompokkan view yang berkaitan.
- Menyimpan konfigurasi bersama (mis. renderer) di tingkat kelas menggunakan `@view_defaults`.
- Memudahkan berbagi state dan helper melalui atribut instance `self.request`.

Perubahan pada contoh ini minimal: fungsi view diubah menjadi metode pada `TutorialViews`, dan pengujian unit diperbarui untuk menginstansiasi kelas view sebelum memanggil metode yang diuji.

Instalasi
---------

Petunjuk berikut diasumsikan dijalankan di Windows PowerShell. Gunakan virtual environment untuk mengisolasi dependensi.

1. Buat dan aktifkan virtual environment:

```powershell
python -m venv .venv
. .\.venv\Scripts\Activate.ps1
```

2. Instal paket dalam mode editable dan dependensi yang diperlukan:

```powershell
pip install -e .
pip install pytest webtest
```

3. Jalankan unit test contoh:

```powershell
# dari direktori `09_view_classes\view_classes`
pytest tutorial/tests.py -q
```

4. Jalankan aplikasi Pyramid:

```powershell
# Pastikan virtualenv aktif, lalu:
pserve development.ini --reload
```

5. Buka browser dan kunjungi:

- http://localhost:6543/ (home view)
- http://localhost:6543/howdy (hello view)

Analisis
--------

Perubahan yang dilakukan di langkah ini bersifat struktural dan tidak menambah fitur baru. Poin penting:

- Struktur: dua view yang sebelumnya merupakan fungsi kini menjadi metode pada `TutorialViews`. Ini membuat hubungan antar-view lebih jelas.
- Konfigurasi bersama: karena kedua view menggunakan template yang sama, kita memindahkan renderer ke dekorator kelas `@view_defaults(renderer='home.pt')`. Ini mengurangi pengulangan di level metode.
- Request injection: setiap instance `TutorialViews` menerima `request` di konstruktor (`__init__`) dan menyimpannya sebagai `self.request`. Metode view dapat mengakses request dan helper lain melalui instance ini.
- Pengujian: test unit diubah untuk mengimpor `TutorialViews`, membuat instance dengan `testing.DummyRequest()` lalu memanggil metode view secara langsung. Functional tests tetap menggunakan `webtest.TestApp` untuk memanggil endpoint HTTP.

Kenapa ini berguna?

- Ketika aplikasi tumbuh dan view berkaitan menjadi lebih banyak, view classes membuat kode lebih modular dan mudah dikelola.
- Menempatkan konfigurasi seperti `renderer` di tingkat kelas membuat perubahan konsisten dan meminimalkan duplikasi.
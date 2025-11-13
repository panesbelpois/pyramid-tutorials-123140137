
# 05: Unit Tests and pytest

## Deskripsi

Pada langkah ini kita menambahkan pengujian unit untuk kode Python proyek menggunakan `pytest`. Tujuannya memastikan fungsi-fungsi (termasuk view Pyramid) berperilaku seperti yang diharapkan dan memudahkan perbaikan serta refaktor di masa datang. Tutorial ini menunjukkan cara menulis tes sederhana yang membuat permintaan palsu (`DummyRequest`) ke view dan memeriksa atribut pada respons.

## Instalasi

Ikuti langkah berikut (contoh untuk Windows PowerShell):

1. Buat virtual environment dan aktifkan:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate
```

2. Pasang paket proyek beserta dependency development (termasuk `pytest`):

```powershell
pip install -e ".[dev]"
```

3. Jalankan tes contoh:

```powershell
pytest tutorial/tests.py -q
```

Perintah-perintah di atas memasang `pytest` sebagai bagian dari extra `dev` yang didefinisikan di `setup.py` dan menjalankan file tes contoh.

## Analisis

- Tes contoh `tutorial/tests.py` menggunakan modul `unittest` dari standar Python dan bantuan dari `pyramid.testing`.
- Pada tiap tes, dibuat `DummyRequest()` dan kemudian memanggil fungsi view (`hello_world`) sehingga kita dapat memeriksa `response.status_code` (atau isi respons) tanpa menjalankan server penuh.
- `setUp()` / `tearDown()` memanggil `testing.setUp()` / `testing.tearDown()` — ini berguna apabila tes perlu memodifikasi konfigurasi Pyramid (`Configurator`); untuk tes sederhana yang hanya memanggil view, ini tidak selalu diperlukan.
- Import view dilakukan di dalam fungsi tes untuk mengurangi efek samping import global. Ini membantu menjaga isolasi tiap tes dan memungkinkan memodifikasi state (mis. monkeypatch) sebelum import bila perlu.

Bagaimana menguji isi HTML respons?
- Jika view mengembalikan objek `Response` (dari WebOb), Anda dapat memeriksa teks tubuh respons dengan `response.text` (jika tersedia) atau `response.body.decode('utf-8')`:

```python
assert 'expected html fragment' in response.text
# atau
assert 'expected html fragment' in response.body.decode('utf-8')
```

## Extra Credit

- Jika mengubah assert menjadi `self.assertEqual(response.status_code, 404)` dan jalankan `pytest`, tes akan gagal. Laporan kegagalan (`AssertionError`) dari pytest akan menunjukkan file dan baris tes yang gagal serta nilai yang diharapkan vs nilai aktual (mis. expected 404 but was 200). Laporan ini membantu cepat menemukan kontrak yang dilanggar.

- Jika memasukkan bug di view (mis. referensi variabel yang tidak ada), menjalankan tes akan menunjukkan traceback yang jelas menunjuk ke lokasi error di kode view — ini sering lebih cepat daripada memuat ulang aplikasi di browser.

- Untuk mengubah kode agar view mengembalikan kode status tertentu, pelajari `pyramid.response.Response` / WebOb Response dan set `status` atau gunakan `Response(status=404)`.

- Kita meng-import `hello_world` di dalam `test_hello_world` agar setiap tes tetap terisolasi dan supaya efek samping import tidak memengaruhi setup tes lainnya. Import di dalam fungsi juga memudahkan manipulasi (monkeypatching) sebelum import bila diperlukan.

---

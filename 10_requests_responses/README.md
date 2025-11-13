10: Handling Web Requests and Responses
======================================

Deskripsi
---------
Modul contoh ini menunjukkan cara menangani HTTP requests dan responses menggunakan Pyramid (berdasarkan WebOb). Aplikasi memiliki dua route dan dua view:

- `/` yang mengembalikan redirect ke `/plain`.
- `/plain` yang menampilkan URL saat ini dan mengambil parameter `name` dari query string (jika tidak ada, tampilkan teks pengganti).

Contoh ini juga berisi unit test dan functional test untuk memverifikasi redirect dan perilaku ketika parameter `name` ada atau tidak.

Instalasi
---------
Petunjuk berikut diasumsikan Anda menggunakan virtual environment Python.

1. Buat dan aktifkan virtual environment (Windows PowerShell):

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

2. Pasang paket proyek dalam mode editable dari root folder `request_response`:

```powershell
pip install -e .
```

3. Jalankan test unit dan functional untuk memastikan semuanya berfungsi:

```powershell
pytest tutorial/tests.py -q
```

4. Untuk menjalankan aplikasi Pyramid secara lokal (reload on change):

```powershell
pserve development.ini --reload
```

Lalu buka `http://localhost:6543/` di browser — Anda akan diarahkan ke `http://localhost:6543/plain`.

Analisis
--------
1. Integrasi WebOb: Pyramid menggunakan WebOb untuk membungkus informasi request dan response. Objek `request` yang diterima view merupakan objek WebOb yang menyediakan properti seperti `url`, `params`, dan struktur untuk bekerja dengan header dan body.

2. Mengambil data dari request: Pada view `plain`, parameter query `name` diambil dengan `self.request.params.get('name', 'No Name Provided')`. Pada testing unit contoh menggunakan `testing.DummyRequest()` yang memudahkan pembuatan request tiruan.

3. Redirects: View `home` menghasilkan redirect dengan mengembalikan `HTTPFound(location='/plain')`. Ini adalah objek khusus dari Pyramid yang menghasilkan response 302 ketika dikembalikan dari view.

4. Pengaturan response: Untuk view `plain` kita membangun `Response` secara eksplisit dan mengatur `content_type='text/plain'` dan `body` berisi URL dan nama — sehingga content type dan body dapat di-custom sesuai kebutuhan.

5. Testing: Unit tests memeriksa perilaku metode view secara langsung (tanpa WSGI), dan functional tests menggunakan `webtest.TestApp` untuk melakukan request terhadap WSGI app yang dihasilkan oleh `main()`.

Contoh output untuk visit `http://localhost:6543/plain?name=alice`:

```
URL http://localhost:6543/plain?name=alice with name: alice
```

Extra credit — Apakah kita bisa `raise HTTPFound(location='/plain')` alih-alih `return`?
---------------------------------------------------------------------------------
Ya, kita bisa. Ada dua cara menghasilkan redirect dengan `HTTPFound`:

- Mengembalikan `HTTPFound(...)` dari view: view mengembalikan objek Response (atau exception-responses seperti `HTTPFound`) yang kemudian dikirim ke klien sebagai response 302.
- Melempar (raise) `HTTPFound(...)`: Pyramid menerima exception khusus ini dan menangani sebagai response juga. Secara fungsional hasilnya sama (klien menerima redirect 302), tetapi ada perbedaan idiomatis:

	- `return HTTPFound(...)`: Menunjukkan bahwa view bertindak sebagai factory/konstructor response dan langsung mengembalikan objek response.
	- `raise HTTPFound(...)`: Dipandang sebagai kontrol aliran yang memotong eksekusi view saat itu juga dan mengandalkan mekanisme exception handling Pyramid untuk mengubahnya menjadi response. Ini berguna jika Anda perlu keluar lebih awal dari eksekusi view (mis. setelah validasi gagal) tanpa menulis lebih banyak logika kontrol.

Keduanya valid; pilih berdasarkan gaya dan kebutuhan alur kontrol di view Anda.

Catatan singkat penggunaan pada Windows
------------------------------------
- Perintah pembuatan dan aktivasi virtualenv pada contoh di atas untuk PowerShell. Jika Anda menggunakan Command Prompt, aktivasi dilakukan dengan `..\.venv\Scripts\activate.bat`.
- Perintah di tutorial asli menggunakan `$VENV/bin/...` yang berlaku untuk lingkungan Unix-like. Pada Windows gunakan path `\.venv\Scripts\...` atau pastikan variabel lingkungan $VENV diatur jika Anda mengikuti konvensi lain.
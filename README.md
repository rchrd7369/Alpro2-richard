# Flask Distributor Firebase Auto

Ini adalah proyek Flask yang berinteraksi dengan Firebase untuk mengelola data distributor secara otomatis. README ini akan memandu Anda dalam menyiapkan, menjalankan, dan menggunakan aplikasi.

## Daftar Isi

1. [Instalasi](#instalasi)
2. [Persiapan](#persiapan)
3. [Menjalankan Aplikasi](#menjalankan-aplikasi)
4. [Variabel Lingkungan](#variabel-lingkungan)
5. [Struktur Proyek](#struktur-proyek)
6. [Penjelasan Kode](#penjelasan-kode)
    - [app.py](#app.py)
    - [about.html](#about.html)
    - [admin.html](#admin.html)
    - [index.html](#index.html)
    - [login.html](#login.html)
    - [confirm_order.html](#confirm_order.html)
7. [Penggunaan](#penggunaan)
8. [Endpoint API](#endpoint-api) *(opsional)*
9. [Kontribusi](#kontribusi)
10. [Lisensi](#lisensi)

## Instalasi

### Requirements

- Python 3.6+
- pip (penginstal paket Python)
- Environment Virtual (opsional, tapi disarankan)

### Instalasi Flask

#### 1. Instalasi Python

Pastikan bahwa Python sudah terinstal di sistem Anda. Anda dapat mengunduh dan menginstalnya dari [situs web resmi Python](https://www.python.org/downloads/).

#### 2. Instal Flask

Flask dapat diinstal secara global atau di dalam lingkungan virtual. Disarankan untuk menginstalnya di dalam lingkungan virtual untuk menghindari konflik dependensi.

```bash
pip install Flask
```

#### 3. Verifikasi Instalasi Flask

Anda dapat memverifikasi apakah Flask sudah terinstal dengan benar dengan menjalankan perintah:

```bash
flask --version
```

### Clone Repositori

```bash
git clone https://github.com/username/flask_distributor_firebase_auto.git
cd flask_distributor_firebase_auto
```

### Membuat dan Mengaktifkan Lingkungan Virtual

Disarankan untuk membuat lingkungan virtual agar dependensi terisolasi:

```bash
# Windows
python -m venv env
.\env\Scripts\Activate

# MacOS/Linux
python3 -m venv env
source env/bin/activate
```

## Preparing Database

### Konfigurasi Firebase

1. Buat proyek Firebase di [Firebase Console](https://console.firebase.google.com/).
2. Hasilkan file `serviceAccountKey.json` dari pengaturan proyek Firebase.
3. Letakkan file `serviceAccountKey.json` di direktori root proyek.

### Variabel Lingkungan

Buat file `.env` di direktori root proyek dan tambahkan variabel berikut:

```
FLASK_APP=app.py
FLASK_ENV=development
SECRET_KEY=your_secret_key
FIREBASE_ADMIN_CREDENTIALS=path_to_serviceAccountKey.json
```

Ganti `your_secret_key` dengan kunci rahasia yang aman dan `path_to_serviceAccountKey.json` dengan jalur menuju kunci akun layanan Firebase Anda.

## 3. Menjalankan Aplikasi

### Pengembangan Lokal

Untuk menjalankan aplikasi secara lokal:

```bash
# Aktifkan lingkungan virtual terlebih dahulu jika belum diaktifkan
flask run
```

Ini akan memulai server Flask pada `http://127.0.0.1:5000/`.


## 5. Struktur Proyek

```
flask_distributor_firebase_auto/
│
├── app.py                   # File aplikasi utama
├── requirements.txt         # Dependensi Python
├── serviceAccountKey.json   # Kunci SDK Admin Firebase
├── .env                     # Variabel lingkungan
├── templates/               # Template HTML (login.html, admin.html, about.html, index.html)
├── static/                  # File statis (CSS, JS, gambar)
└── ...                      # File dan direktori lainnya
```

## 6. Penjelasan Kode

### App Route /login.

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']

        if check_login(username, password):
            session['username'] = username  # Simpan status login dalam session
            flash('Login successful!', 'success')
            return redirect(url_for('admin'))
        else:
            flash('Invalid username or password!', 'danger')

    return render_template('login.html')
```
Alur:
- Kami menggunakan metode request yaitu POST, Lalu sistem akan mengambil username dan password dari form.
- Memeriksa validitas login menggunakan check_login
- Jika valid, menyimpan username dalam session dan mengarahkan ke halaman admin.
- Jika tidak valid, menampilkan pesan error.
- Jika metode request adalah GET, akan menampilkan halaman login.

### Login.html
Pada halaman login, kami menggunakan beberapa file css eksternal untum tampilan dan tata letak:
- **Bootstrap** digunakan untuk tata letak grid yang responsif
- **AOS** (Animate On Scroll) digunkan untuk animasi halus saat elemen muncul dalam tampilan saat user melakukan refresh halaman.
- **Magnific Popup, Owl Carousel, dan Flaticon** untuk mendukung elemen interaktif dan ikon visual.

  #### Java Script
```python
const togglePassword = document.querySelector('#togglePassword');
const password = document.querySelector('#password');
const eyeIcon = document.querySelector('#eyeIcon');

togglePassword.addEventListener('click', function () {
  const type = password.getAttribute('type') === 'password' ? 'text' : 'password';
  password.setAttribute('type', type);
  
  if (type === 'password') {
    eyeIcon.classList.remove('icon-eye-slash');
    eyeIcon.classList.add('icon-eye');
  } else {
    eyeIcon.classList.remove('icon-eye');
    eyeIcon.classList.add('icon-eye-slash');
  }
});


```

### App Route /admin.

```python
@app.route('/admin')
def admin():
    if 'username' not in session:  # Cek apakah user sudah login
        flash('Please log in to access this page.', 'warning')
        return redirect(url_for('login'))

    # Ambil data dari Firestore collection 'tb_pesanan' dan 'tb_ongkos_kirim'
    pesanan_docs = db.collection('tb_pesanan').stream()
    ongkos_docs = db.collection('tb_ongkos_kirim').stream()

    # Filter pesanan berdasarkan status
    pesanan_list = [{'new_id': doc.id, **doc.to_dict()} for doc in pesanan_docs]
    ongkos_list = [{'new_id': doc.id, **doc.to_dict()} for doc in ongkos_docs if doc.to_dict().get('id_status') != 'Pesanan Selesai']
    histori_list = [{'new_id': doc.id, **doc.to_dict()} for doc in db.collection('tb_ongkos_kirim').where('id_status', '==', 'Pesanan Selesai').stream()]

    return render_template('admin.html', pesanan=pesanan_list, ongkos=ongkos_list, histori=histori_list, status_list=STATUS_LIST)

```
Fungsi:
Menampilkan halaman admin dengan data pesanan dan ongkos kirim.


Alur
- Memeriksa apakah pengguna sudah login; jika tidak, mengarahkan ke halaman login.
- Mengambil data dari koleksi tb_pesanan dan tb_ongkos_kirim di Firestore.
- Memproses data untuk ditampilkan di template admin.html.
- Menyediakan daftar status pesanan (STATUS_LIST) untuk digunakan dalam template.

### app.py

Ini adalah file aplikasi utama yang berisi rute Flask dan logika inti dari aplikasi.

```python
from flask import Flask, render_template, request, redirect, url_for
import firebase_admin
from firebase_admin import credentials, firestore

# Inisialisasi aplikasi Flask
app = Flask(__name__)

# Memuat kredensial SDK Admin Firebase
cred = credentials.Certificate('serviceAccountKey.json')
firebase_admin.initialize_app(cred)

# Inisialisasi Firestore DB
db = firestore.client()

@app.route('/')
def index():
    # Mengambil semua data distributor dari Firestore
    distributors = db.collection('distributors').get()
    distributor_list = [doc.to_dict() for doc in distributors]
    return render_template('index.html', distributors=distributor_list)

@app.route('/confirm_order/<string:distributor_id>', methods=['GET', 'POST'])
def confirm_order(distributor_id):
    if request.method == 'POST':
        # Memperbarui status pesanan di Firestore berdasarkan pengiriman formulir
        db.collection('distributors').document(distributor_id).update({
            'order_status': 'confirmed'
        })
        return redirect(url_for('index'))
    distributor = db.collection('distributors').document(distributor_id).get().to_dict()
    return render_template('confirm_order.html', distributor=distributor)

if __name__ == '__main__':
    app.run(debug=True)
```

- **Import dan Inisialisasi**:
  - `firebase_admin` dan `firestore` digunakan untuk menghubungkan ke Firebase Firestore.
  - `app = Flask(__name__)` menginisialisasi aplikasi Flask.

- **Rute**:
  - `/`: Menampilkan daftar distributor yang diambil dari Firestore.
  - `/confirm_order/<string:distributor_id>`: Mengonfirmasi pesanan untuk distributor.

### index.html

File template ini menampilkan daftar distributor dengan opsi untuk mengonfirmasi pesanan.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Daftar Distributor</title>
</head>
<body>
    <h1>Daftar Distributor</h1>
    <ul>
        {% for distributor in distributors %}
            <li>
                {{ distributor.name }} - {{ distributor.order_status }}
                <a href="{{ url_for('confirm_order', distributor_id=distributor.id) }}">Konfirmasi Pesanan</a>
            </li>
        {% endfor %}
    </ul>
</body>
</html>
```

- **Menampilkan daftar distributor** dan status pesanan mereka.
- Setiap distributor memiliki tautan untuk mengonfirmasi pesanan.

### confirm_order.html

Template ini menyediakan formulir untuk mengonfirmasi pesanan dari distributor tertentu.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Konfirmasi Pesanan</title>
</head>
<body>
    <h1>Konfirmasi Pesanan untuk {{ distributor.name }}</h1>
    <form method="POST">
        <p>Apakah Anda yakin ingin mengonfirmasi pesanan untuk {{ distributor.name }}?</p>
        <button type="submit">Konfirmasi</button>
    </form>
    <a href="{{ url_for('index') }}">Kembali ke daftar</a>
</body>
</html>
```

- **Formulir untuk konfirmasi pesanan**: Menampilkan pesan konfirmasi dan tombol untuk mengonfirmasi pesanan.
- Formulir dikirimkan ke rute yang sama menggunakan metode `POST`, yang akan memperbarui status pesanan di Firestore.

## Penggunaan

1. Pastikan konfigurasi Firebase telah diatur dengan benar.
2. Jalankan aplikasi Flask.
3. Buka browser web dan pergi ke `http://127.0.0.1:5000/`.

## Endpoint API

*(Opsional: Daftar endpoint API dan fungsionalitasnya)*

## Kontribusi

Kontribusi sangat disambut! Ikuti langkah-langkah berikut untuk berkontribusi:

1. Fork repositori ini.
2. Buat cabang fitur baru (`git checkout -b feature-branch`).
3. Commit perubahan Anda (`git commit -m 'Tambah fitur baru'`).
4. Push ke cabang (`git push origin feature-branch`).
5. Buat Pull Request baru.

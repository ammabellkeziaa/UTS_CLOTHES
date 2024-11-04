# UTS_CLOTHES
## Nama : Ammabell Kezia
## NIM : 21.01.55.0008
## Deskripsi Project
Project UTS membuat manajemen clothes menggunakan PHP dan MySQL dengan pengujian yang menggunakan postman.
## Alat yang Dibutuhkan
1. XAMPP (atau server web lain dengan PHP dan MySQL)
2. Text editor : Visual Studio Code
3. Postman
## Cara Instalasi dan Peggunaan
## 1. Persiapan lingkungan
a. Instal XAMPP jika belum ada.  
b. Buat folder baru bernama clothes di dalam direktori htdocs XAMPP Anda.
## 2. Membuat Database
a. Buka phpMyAdmin (http://localhost/phpmyadmin)  
b. Buat database baru bernama clothes  
c. Pilih database clothes, lalu buka tab SQL  
d. Jalankan query SQL berikut untuk membuat tabel dan menambahkan data sampel:
```sql
   CREATE TABLE `clothes` (
  `id` int(11) NOT NULL,
  `name` varchar(45) NOT NULL,
  `size` varchar(15) NOT NULL,
  `prize` int(100) NOT NULL,
  `stok` int(10) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

INSERT INTO `clothes` (`id`, `name`, `size`, `prize`, `stok`) VALUES
(1, 'kaos berkerah', 'M', 35, 10),
(2, 'hoodie', 'L', 210, 15),
(3, 'kemeja lengan panjang', 'S', 150, 12),
(4, 'kemeja lengan pendek', 'L', 150, 10),
(5, 'kaos berkerah', 'L', 75, 25);
```
## 3. Membuat file PHP untuk Web Service
a. Buka text editor Anda.  
b. Buat file baru dan simpan sebagai clothes.php di dalam folder clothes.  
c. Salin dan tempel kode berikut ke dalam clothes.php: 
```sql
<?php
header("Content-Type: application/json; charset=UTF-8");
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");
header("Access-Control-Allow-Headers: Content-Type, Access-Control-Allow-Headers, sizeization, X-Requested-With");

$method = $_SERVER['REQUEST_METHOD'];
$request = [];

if (isset($_SERVER['PATH_INFO'])) {
    $request = explode('/', trim($_SERVER['PATH_INFO'], '/'));
}

function getConnection()
{
    $host = 'localhost';
    $db   = 'clothes';
    $user = 'root';
    $pass = ''; // Ganti dengan password MySQL Anda jika ada
    $charset = 'utf8mb4';

    $dsn = "mysql:host=$host;dbname=$db;charset=$charset";
    $options = [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false,
    ];
    try {
        return new PDO($dsn, $user, $pass, $options);
    } catch (\PDOException $e) {
        throw new \PDOException($e->getMessage(), (int)$e->getCode());
    }
}

function response($status, $data = NULL)
{
    header("HTTP/1.1 " . $status);
    if ($data) {
        echo json_encode($data);
    }
    exit();
}

$db = getConnection();

switch ($method) {
    case 'GET':
        if (!empty($request) && isset($request[0])) {
            $id = $request[0];
            $stmt = $db->prepare("SELECT * FROM clothes WHERE id = ?");
            $stmt->execute([$id]);
            $clothes = $stmt->fetch();
            if ($clothes) {
                response(200, $clothes);
            } else {
                response(404, ["message" => "clothes not found"]);
            }
        } else {
            $stmt = $db->query("SELECT * FROM clothes");
            $clothess = $stmt->fetchAll();
            response(200, $clothess);
        }
        break;

    case 'POST':
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->name) || !isset($data->size) || !isset($data->prize) || !isset($data->stok)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "INSERT INTO clothes (name, size, prize, stok) VALUES (?, ?, ?, ?)";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->name, $data->size, $data->prize, $data->stok])) {
            response(201, ["message" => "clothes created", "id" => $db->lastInsertId()]);
        } else {
            response(500, ["message" => "Failed to create clothes"]);
        }
        break;

    case 'PUT':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "clothes ID is required"]);
        }
        $id = $request[0];
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->name) || !isset($data->size) || !isset($data->prize) || !isset($data->stok)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "UPDATE clothes SET name = ?, size = ?, prize = ?, stok = ? WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->name, $data->size, $data->prize, $data->stok, $id])) {
            response(200, ["message" => "clothes updated"]);
        } else {
            response(500, ["message" => "Failed to update clothes"]);
        }
        break;

    case 'DELETE':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "clothes ID is required"]);
        }
        $id = $request[0];
        $sql = "DELETE FROM clothes WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$id])) {
            response(200, ["message" => "clothes deleted"]);
        } else {
            response(500, ["message" => "Failed to delete clothes"]);
        }
        break;

    default:
        response(405, ["message" => "Method not allowed"]);
        break;
}
```
## 4. Pengujian menggunakan Postman
a. Buka Postman  
b. Buat request baru untuk setiap operasi berikut:
## 1. GET ALL CLOTHES
- Method: GET
- URL : `http://localhost/clothes/clothes.php`
- Klik "Send"
## 2. POST NEW CLOTHES
- Method: POST
- URL: `http://localhost/clothes/clothes.php`
  Headers:
    - Key: Content-Type
    - Value: application/json
  Body:
    - Pilih "raw" dan "JSON"
Masukkan:
```sql
{
        "name": "kaos berkerah",
        "size": "L",
        "prize": 75,
        "stok": 8
    }
```
- Klik "Send"
## 3. GET Spesifik CLOTHES
- Method: GET
- URL : `http://localhost/clothes/clothes.php/2 (untuk buku dengan ID 2)`
- Klik "Send"
## 4. PUT CLOTHES (Update)
- Method: PUT
- URL : `http://localhost/rest_buku/book_api.php/5 (asumsikan ID buku baru adalah 5)`
- Headers:
  - Key: Content-Type
  - Value: application/json
- Body:
  - Pilih "raw" dan "JSON"
  - Masukkan :
    ```sql
    {
    "name": "kaos berkerah",
    "size": "L",
    "prize": 75,
    "stok": 25
    }
    ```
  - Klik "Send"
## 5. DELETE CLOTHES
- Method: DELETE
- URL : http://localhost/clothes/clothes/6 (untuk menghapus buku dengan ID 6)
- Klik "Send"

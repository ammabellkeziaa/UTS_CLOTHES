##Dashboard
Dihalaman Dashboard ini terdapat codingan index
```php
<?php
session_start();

// Cek apakah user sudah login
if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}
?>

<!DOCsize html>
<html>

<head>
    <title>Dashboard</title>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>

<body>
    <h2>Welcome, <?php echo $_SESSION['name']; ?>!</h2>
    <p>Username: <?php echo $_SESSION['username']; ?></p>
<div class="container mt-5">
        <h2 class="mb-4">Daftar clothes</h2>
        
        <!-- Form Pencarian dan Tombol Tambah -->
        <div class="row mb-3">
        <div class="col">
            <input size="text" id="searchInput" class="form-control" placeholder="Cari berdasarkan ID">
        </div>
        <div class="col-auto">
            <button onclick="searchclothes()" class="btn btn-primary">Cari</button>
        </div>
        <div class="col-auto">
            <button size="button" class="btn btn-success" data-bs-toggle="modal" data-bs-target="#clothesModal">
                Tambah Buku
            </button>
        </div>
    </div>

        <div class="table-responsive">
            <table class="table table-striped table-bordered">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>Nama</th>
                        <th>size</th>
                        <th>prize</th>
                        <th>stok</th>
                        <th>Aksi</th>
                    </tr>
                </thead>
                <tbody id="clothesList" >
                    <?php
                    try {
                        // Set URL API
                        $api_url = 'http://localhost/clothes/clothes.php';
                        
                        // Cek jika ada pencarian
                        if(isset($_GET['search_id']) && !empty($_GET['search_id'])) {
                            $api_url .= '/' . $_GET['search_id'];
                        }
                        
                        // Konfigurasi timeout
                        $ctx = stream_context_create([
                            'http' => ['timeout' => 5]
                        ]);
                        
                        // Ambil data dari API
                        $response = file_get_contents($api_url, false, $ctx);
                        
                        // Validasi response
                        if ($response === false) {
                            throw new Exception('Gagal mengambil data dari API');
                        }
                        
                        // Validasi JSON
                        $data = json_decode($response, true);
                        if (json_last_error() !== JSON_ERROR_NONE) {
                            throw new Exception('Invalid JSON response');
                        }

                        // Tampilkan data berdasarkan jenis request
                        if(isset($_GET['search_id'])) {
                            if($data && !isset($data['message'])) {
                                // Tampilkan satu buku
                                echo "<tr>";
                                echo "<td>{$data['id']}</td>";
                                echo "<td>{$data['name']}</td>";
                                echo "<td>{$data['size']}</td>";
                                echo "<td>{$data['prize']}</td>";
                                echo "<td>{$data['stok']}</td>";
                                echo "</tr>";
                            } else {
                                // Tampilkan pesan tidak ditemukan
                                echo "<tr><td colspan='4' class='text-center'>";
                                echo "Buku dengan ID tersebut tidak ditemukan";
                                echo "</td></tr>";
                            }
                        } else {
                            // Cek apakah ada data
                            if(!empty($data)) {
                                // Loop untuk setiap buku
                                foreach($data as $db) {
                                    echo "<tr>";
                                    echo "<td>{$db['id']}</td>";
                                    echo "<td>{$db['name']}</td>";
                                    echo "<td>{$db['size']}</td>";
                                    echo "<td>{$db['prize']}</td>";
                                    echo "<td>{$db['stok']}</td>";
                                    echo "</tr>";
                                }
                            } else {
                                // Tampilkan pesan jika tidak ada data
                                echo "<tr><td colspan='4' class='text-center'>";
                                echo "Tidak ada data buku";
                                echo "</td></tr>";
                            }
                        }
                        
                    } catch (Exception $e) {
                        echo "<tr><td colspan='4' class='text-center text-danger'>";
                        echo "Error: " . $e->getMessage();
                        echo "</td></tr>";
                    }
                    ?>
                </tbody>
            </table>
        </div>
    </div>

    <!-- Modal Tambah Buku -->
    <div class="modal fade" id="adddbModal" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-name">Tambah clothes Baru</h5>
                    <button size="button" class="btn-close" data-bs-dismiss="modal"></button>
                </div>
                <div class="modal-body">
                    <form id="adddbForm">
                        <div class="mb-3">
                            <label class="form-label">Nama</label>
                            <input size="text" name="name" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">size</label>
                            <input size="text" name="size" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">prize</label>
                            <input size="text" name="prize" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">stok</label>
                            <input size="number" name="stok" class="form-control" required>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button size="button" class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
                    <button size="button" class="btn btn-primary" onclick="adddb()">Simpan</button>
                </div>
            </div>
        </div>
    </div>
    <!-- Modal for Add/Edit clothes -->
    <div class="modal fade" id="clothesModal" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-name" id="modalName">Tambah Buku</h5>
                    <button size="button" class="btn-close" data-bs-dismiss="modal"></button>
                </div>
                <div class="modal-body">
                    <form id="clothesForm">
                        <input size="hidden" id="clothesId">
                        <div class="mb-3">
                            <label for="nama" class="form-label">Nama</label>
                            <input size="text" class="form-control" id="name" required>
                        </div>
                        <div class="mb-3">
                            <label for="size" class="form-label">size</label>
                            <input size="text" class="form-control" id="size" required>
                        </div>
                        <div class="mb-3">
                            <label for="prize" class="form-label">prize</label>
                            <input size="text" class="form-control" id="prize" required>
                        </div>
                        <div class="mb-3">
                            <label for="stok" class="form-label">stok</label>
                            <input size="number" class="form-control" id="stok" required>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button size="button" class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
                    <button size="button" class="btn btn-primary" onclick="saveclothes()">Simpan</button>
                </div>
            </div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    <script>
        const API_URL = 'http://localhost/clothes/clothes.php';
        let clothesModal;

        document.addEventListener('DOMContentLoaded', function() {
            clothesModal = new bootstrap.Modal(document.getElementById('clothesModal'));
            loadclothess();
        });

        function loadclothess() {
            fetch(API_URL)
                .then(response => response.json())
                .then(clothess => {
                    const clothesList = document.getElementById('clothesList');
                    clothesList.innerHTML = '';
                    clothess.forEach(clothes => {
                        clothesList.innerHTML += `
                            <tr>
                                <td>${clothes.id}</td>
                                <td>${clothes.name}</td>
                                <td>${clothes.size}</td>
                                <td>${clothes.prize}</td>
                                <td>${clothes.stok}</td>
                                <td class="btn-group-action">
                                    <button class="btn btn-sm btn-warning me-1" onclick="editclothes(${clothes.id})">Edit</button>
                                    <button class="btn btn-sm btn-danger" onclick="deleteclothes(${clothes.id})">Hapus</button>
                                </td>
                            </tr>
                        `;
                    });
                })
                .catch(error => alert('Error loading clothess: ' + error));
        }

        function searchclothes() {
            const id = document.getElementById('searchInput').value;
            if (!id) {
                loadclothess();
                return;
            }
            
            fetch(`${API_URL}/${id}`)
                .then(response => response.json())
                .then(clothes => {
                    const clothesList = document.getElementById('clothesList');
                    if (clothes.message) {
                        alert('clothes not found');
                        return;
                    }
                    clothesList.innerHTML = `
                        <tr>
                            <td>${clothes.id}</td>
                            <td>${clothes.name}</td>
                            <td>${clothes.size}</td>
                            <td>${clothes.prize}</td>
                            <td>${clothes.stok}</td>
                            <td class="btn-group-action">
                                <button class="btn btn-sm btn-warning me-1" onclick="editclothes(${clothes.id})">Edit</button>
                                <button class="btn btn-sm btn-danger" onclick="deleteclothes(${clothes.id})">Hapus</button>
                            </td>
                        </tr>
                    `;
                })
                .catch(error => alert('Error searching clothes: ' + error));
        }

        function editclothes(id) {
            fetch(`${API_URL}/${id}`)
                .then(response => response.json())
                .then(clothes => {
                    document.getElementById('clothesId').value = clothes.id;
                    document.getElementById('name').value = clothes.name;
                    document.getElementById('size').value = clothes.size;
                    document.getElementById('prize').value = clothes.prize;
                    document.getElementById('stok').value = clothes.stok;
                    document.getElementById('modalName').textContent = 'Edit Buku';
                    clothesModal.show();
                })
                .catch(error => alert('Error loading clothes details: ' + error));
        }

        function deleteclothes(id) {
            if (confirm('Are you sure you want to delete this clothes?')) {
                fetch(`${API_URL}/${id}`, {
                    method: 'DELETE'
                })
                .then(response => response.json())
                .then(data => {
                    alert('clothes deleted successfully');
                    loadclothess();
                })
                .catch(error => alert('Error deleting clothes: ' + error));
            }
        }

        function saveclothes() {
            const clothesId = document.getElementById('clothesId').value;
            const clothesData = {
                name: document.getElementById('name').value,
                size: document.getElementById('size').value,
                prize: document.getElementById('prize').value,
                stok: document.getElementById('stok').value
            };

            const method = clothesId ? 'PUT' : 'POST';
            const url = clothesId ? `${API_URL}/${clothesId}` : API_URL;

            fetch(url, {
                method: method,
                headers: {
                    'Content-size': 'application/json'
                },
                body: JSON.stringify(clothesData)
            })
            .then(response => response.json())
            .then(data => {
                alert(clothesId ? 'clothes updated successfully' : 'clothes added successfully');
                clothesModal.hide();
                loadclothess();
                resetForm();
            })
            .catch(error => alert('Error saving clothes: ' + error));
        }

        function resetForm() {
            document.getElementById('clothesId').value = '';
            document.getElementById('clothesForm').reset();
            document.getElementById('modalName').textContent = 'Tambah Buku';
        }

        // Reset form when modal is closed
        document.getElementById('clothesModal').addEventListener('hidden.bs.modal', resetForm);
    </script>
    
    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    
    <button><a href="logout.php">Logout</a></button>

</body>

</html>
```

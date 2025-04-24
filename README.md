<?php
$servername = "localhost";
$username = "root";
$password = "";
$dbname = "rumah_sakit";

// Buat koneksi
$conn = new mysqli($servername, $username, $password, $dbname);

// Cek koneksi
if ($conn->connect_error) {
    die("Koneksi gagal: " . $conn->connect_error);
}
<!DOCTYPE html>
<html>
<head>
    <title>Sistem Rumah Sakit</title>
    <style>
        body { font-family: Arial; margin: 20px; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #4CAF50; color: white; }
        form { margin-bottom: 20px; }
        input, select { margin: 5px; padding: 5px; }
        button { background-color: #4CAF50; color: white; padding: 8px; border: none; cursor: pointer; }
    </style>
</head>
<body>
    <h2>Sistem Manajemen Rumah Sakit</h2>

    <!-- Form Tambah/Edit Pasien -->
    <h3>Tambah/Edit Pasien</h3>
    <?php
    $conn = new mysqli("localhost", "root", "", "rumah_sakit");
    if ($conn->connect_error) die("Koneksi gagal: " . $conn->connect_error);

    $id_pasien = $nama_pasien = $tanggal_lahir = $alamat = "";
    $edit_mode = false;

    if (isset($_GET['edit'])) {
        $edit_mode = true;
        $id_pasien = $_GET['edit'];
        $result = $conn->query("SELECT * FROM pasien WHERE id_pasien='$id_pasien'");
        if ($result->num_rows > 0) {
            $row = $result->fetch_assoc();
            $nama_pasien = $row['nama_pasien'];
            $tanggal_lahir = $row['tanggal_lahir'];
            $alamat = $row['alamat'];
        }
    }

    if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_POST['save_pasien'])) {
        $id_pasien = $_POST['id_pasien'];
        $nama_pasien = $_POST['nama_pasien'];
        $tanggal_lahir = $_POST['tanggal_lahir'];
        $alamat = $_POST['alamat'];

        if ($edit_mode) {
            $sql = "UPDATE pasien SET nama_pasien='$nama_pasien', tanggal_lahir='$tanggal_lahir', alamat='$alamat' WHERE id_pasien='$id_pasien'";
        } else {
            $sql = "INSERT INTO pasien (id_pasien, nama_pasien, tanggal_lahir, alamat) VALUES ('$id_pasien', '$nama_pasien', '$tanggal_lahir', '$alamat')";
        }

        if ($conn->query($sql)) echo "<p>Data pasien tersimpan!</p>";
        else echo "<p>Error: " . $conn->error . "</p>";
    }
    ?>
    <form method="POST">
        <input type="text" name="id_pasien" placeholder="ID Pasien" value="<?php echo $id_pasien; ?>" <?php echo $edit_mode ? 'readonly' : ''; ?> required>
        <input type="text" name="nama_pasien" placeholder="Nama Pasien" value="<?php echo $nama_pasien; ?>" required>
        <input type="date" name="tanggal_lahir" value="<?php echo $tanggal_lahir; ?>" required>
        <input type="text" name="alamat" placeholder="Alamat" value="<?php echo $alamat; ?>" required>
        <button type="submit" name="save_pasien"><?php echo $edit_mode ? 'Update' : 'Tambah'; ?></button>
    </form>

    <!-- Tabel Pasien -->
    <h3>Data Pasien</h3>
    <table>
        <tr><th>ID</th><th>Nama</th><th>Tanggal Lahir</th><th>Alamat</th><th>Aksi</th></tr>
        <?php
        $result = $conn->query("SELECT * FROM pasien");
        while ($row = $result->fetch_assoc()) {
            echo "<tr>
                <td>{$row['id_pasien']}</td>
                <td>{$row['nama_pasien']}</td>
                <td>{$row['tanggal_lahir']}</td>
                <td>{$row['alamat']}</td>
                <td>
                    <a href='?edit={$row['id_pasien']}'>Edit</a> |
                    <a href='?delete={$row['id_pasien']}' onclick='return confirm(\"Yakin hapus?\")'>Hapus</a>
                </td>
            </tr>";
        }
        ?>
    </table>

    <!-- Hapus Pasien -->
    <?php
    if (isset($_GET['delete'])) {
        $id_pasien = $_GET['delete'];
        $sql = "DELETE FROM pasien WHERE id_pasien='$id_pasien'";
        if ($conn->query($sql)) echo "<script>alert('Data dihapus!'); window.location='?';</script>";
        else echo "<p>Error: " . $conn->error . "</p>";
    }
    ?>

    <!-- Form Tambah Rekam Medis -->
    <h3>Tambah Rekam Medis</h3>
    <?php
    if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_POST['save_rekam'])) {
        $id_rekam_medis = $_POST['id_rekam_medis'];
        $id_pasien = $_POST['id_pasien'];
        $id_dokter = $_POST['id_dokter'];
        $tanggal_periksa = $_POST['tanggal_periksa'];
        $diagnosis = $_POST['diagnosis'];

        $sql = "INSERT INTO rekam_medis (id_rekam_medis, id_pasien, id_dokter, tanggal_periksa, diagnosis) 
                VALUES ('$id_rekam_medis', '$id_pasien', '$id_dokter', '$tanggal_periksa', '$diagnosis')";
        if ($conn->query($sql)) echo "<p>Rekam medis tersimpan!</p>";
        else echo "<p>Error: " . $conn->error . "</p>";
    }
    ?>
    <form method="POST">
        <input type="text" name="id_rekam_medis" placeholder="ID Rekam Medis" required>
        <select name="id_pasien" required>
            <option value="">Pilih Pasien</option>
            <?php
            $result = $conn->query("SELECT id_pasien, nama_pasien FROM pasien");
            while ($row = $result->fetch_assoc()) {
                echo "<option value='{$row['id_pasien']}'>{$row['nama_pasien']}</option>";
            }
            ?>
        </select>
        <select name="id_dokter" required>
            <option value="">Pilih Dokter</option>
            <?php
            $result = $conn->query("SELECT id_dokter, nama_dokter FROM dokter");
            while ($row = $result->fetch_assoc()) {
                echo "<option value='{$row['id_dokter']}'>{$row['nama_dokter']}</option>";
            }
            ?>
        </select>
        <input type="date" name="tanggal_periksa" required>
        <input type="text" name="diagnosis" placeholder="Diagnosis" required>
        <button type="submit" name="save_rekam">Tambah</button>
    </form>

    <!-- Tabel Rekam Medis -->
    <h3>Data Rekam Medis</h3>
    <table>
        <tr><th>ID</th><th>Pasien</th><th>Dokter</th><th>Tanggal</th><th>Diagnosis</th></tr>
        <?php
        $result = $conn->query("SELECT r.id_rekam_medis, p.nama_pasien, d.nama_dokter, r.tanggal_periksa, r.diagnosis 
                                FROM rekam_medis r 
                                JOIN pasien p ON r.id_pasien = p.id_pasien 
                                JOIN dokter d ON r.id_dokter = d.id_dokter");
        while ($row = $result->fetch_assoc()) {
            echo "<tr>
                <td>{$row['id_rekam_medis']}</td>
                <td>{$row['nama_pasien']}</td>
                <td>{$row['nama_dokter']}</td>
                <td>{$row['tanggal_periksa']}</td>
                <td>{$row['diagnosis']}</td>
            </tr>";
        }
        $conn->close();
        ?>
    </table>
</body>
</html>

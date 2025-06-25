<?php
$conn = new mysqli("localhost", "root", "", "taixiu");
if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $username = $_POST["username"];
    $email = $_POST["email"];
    $password = password_hash($_POST["password"], PASSWORD_BCRYPT);
    $conn->query("INSERT INTO users (username, email, password) VALUES ('$username', '$email', '$password')");
    header("Location: index.php");
    exit();
}
?>
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Đăng ký - ToolSunWin</title>
    <style>
        body {
            background: #0a0a0a;
            color: white;
            font-family: Arial;
            text-align: center;
            padding-top: 60px;
        }
        input {
            display: block;
            margin: 10px auto;
            padding: 10px;
            width: 240px;
            border-radius: 8px;
        }
        button {
            padding: 10px 30px;
            background: #00f0ff;
            border: none;
            color: black;
            font-weight: bold;
            cursor: pointer;
            border-radius: 8px;
        }
    </style>
</head>
<body>
    <h2>TOOLSUNWIN</h2>
    <form method="POST">
        <input name="username" placeholder="Tên đăng nhập" required>
        <input name="email" placeholder="Gmail của bạn" type="email" required>
        <input name="password" placeholder="Mật khẩu" type="password" required>
        <button type="submit">Đăng ký</button>
    </form>
    <p>Đã có tài khoản? <a href="index.php" style="color:#00f0ff;">Đăng nhập</a></p>
</body>
</html>
<?php
session_start();
$conn = new mysqli("localhost", "root", "", "taixiu");

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $u = $_POST["username"];
    $p = $_POST["password"];
    $res = $conn->query("SELECT * FROM users WHERE username='$u'");
    $user = $res->fetch_assoc();
    if ($user && password_verify($p, $user["password"])) {
        $_SESSION["user"] = $user["username"];
        header("Location: sunwingame.php");
        exit();
    } else {
        $error = "Sai thông tin đăng nhập!";
    }
}
?>
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Đăng nhập - ToolSunWin</title>
    <style>
        body {
            background: #000;
            color: white;
            text-align: center;
            font-family: sans-serif;
            padding-top: 60px;
        }
        input {
            margin: 10px;
            padding: 10px;
            width: 220px;
            border-radius: 8px;
        }
        button {
            padding: 10px 30px;
            background: #00f0ff;
            border: none;
            border-radius: 8px;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <h2>Đăng nhập ToolSunWin</h2>
    <form method="POST">
        <input name="username" placeholder="Tên đăng nhập" required>
        <input name="password" placeholder="Mật khẩu" type="password" required>
        <button type="submit">Đăng nhập</button>
    </form>
    <p>Bạn chưa có tài khoản? <a href="register.php" style="color:#00f0ff;">Đăng ký</a></p>
    <?php if (isset($error)) echo "<p style='color:red;'>$error</p>"; ?>
</body>
</html>
<?php
session_start();
session_destroy();
header("Location: index.php");
exit();
?>
<?php
session_start();
if (!isset($_SESSION["user"])) {
    header("Location: index.php");
    exit();
}
$conn = new mysqli("localhost", "root", "", "taixiu");

function getDiceFromAPI() {
    $api_url = "https://wanglinapiws.up.railway.app/api/taixiu";
    $json = file_get_contents($api_url);
    return json_decode($json, true);
}

function du_doan_theo_xi_ngau($d1, $d2, $d3) {
    $total = $d1 + $d2 + $d3;
    $result_list = [];
    foreach ([$d1, $d2, $d3] as $d) {
        $tmp = $d + $total;
        if ($tmp === 4 || $tmp === 5) $tmp -= 4;
        elseif ($tmp >= 6) $tmp -= 6;
        $result_list[] = ($tmp % 2 == 0) ? "Tài" : "Xỉu";
    }
    return (array_count_values($result_list)["Tài"] ?? 0) >= 2 ? "Tài" : "Xỉu";
}

function is_cau_xau($cau) {
    $ds = ["TXXTX","TXTXT","XXTXX","XTXTX","TTXTX","XTTXT","TXXTT","TXTTX","XXTTX","XTXTT","TXTXX","XXTXT","TTXXT","TXTTT","XTXXT","XTTTX","TTXTT","TXXTX"];
    return in_array($cau, $ds);
}
function is_cau_dep($cau) {
    $ds = ["TTTTT","XXXXX","TTTXX","XXTTT","TXTXX","TTTXT","XTTTX","TXXXT","XXTXX","TXTTT","XTTTT","TTXTX","TXXTX","TXTXT","XTXTX","TTTXT","XTTXT","XXTXX","TXXXX"];
    return in_array($cau, $ds);
}

$data = getDiceFromAPI();
$session = $data["session"];
$dice = $data["dice"];
$result = $data["result"];
$d1 = $dice[0];
$d2 = $dice[1];
$d3 = $dice[2];
$prediction = du_doan_theo_xi_ngau($d1, $d2, $d3);
$is_correct = ($prediction === $result) ? 1 : 0;

$latest = $conn->query("SELECT prediction FROM taixiu_history ORDER BY id DESC LIMIT 5");
$cau = "";
while ($row = $latest->fetch_assoc()) {
    $cau = $row["prediction"][0] . $cau;
}
$cau_5 = strtoupper(substr($cau, -5));
$cuoi = $prediction;
$note = "ℹ️ Cầu trung lập ($cau_5)";
if (is_cau_xau($cau_5)) {
    $cuoi = ($prediction === "Tài") ? "Xỉu" : "Tài";
    $note = "⚠️ Cầu xấu ($cau_5) – Đảo kết quả";
} elseif (is_cau_dep($cau_5)) {
    $note = "✅ Cầu đẹp ($cau_5) – Giữ nguyên";
}

$exist = $conn->query("SELECT id FROM taixiu_history WHERE session_id=$session")->num_rows;
if (!$exist) {
    $conn->query("INSERT INTO taixiu_history (session_id, d1, d2, d3, prediction, result, is_correct)
        VALUES ($session, $d1, $d2, $d3, '$cuoi', '$result', $is_correct)");
}

$history = $conn->query("SELECT * FROM taixiu_history ORDER BY id DESC LIMIT 15");
?>
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Kết quả Tài Xỉu</title>
    <style>
        body { background: black; color: lime; font-family: monospace; padding: 20px; }
        .menu { position: fixed; left: 0; top: 0; padding: 10px; width: 160px; background: #111; height: 100%; border-right: 2px solid #0f0; }
        .menu a { display: block; color: #0ff; margin: 10px 0; text-decoration: none; }
        .content { margin-left: 180px; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        td, th { padding: 5px; border: 1px solid #0f0; text-align: center; }
    </style>
</head>
<body>
<div class="menu">
    <h3>☢ MENU ☢</h3>
    <a href="#">Trang Chủ</a>
    <a href="#">Hỗ Trợ</a>
    <a href="#">Lưu Ý</a>
    <a href="#">Cài Đặt</a>
    <a href="logout.php">Đăng Xuất</a>
</div>

<div class="content">
    <h2>Tool Phân Tích SUNWIN</h2>
    <p><b>Phiên:</b> <?= $session ?> | 🎲 Xí ngầu: <?= "$d1 - $d2 - $d3" ?> | Tổng: <?= $d1 + $d2 + $d3 ?></p>
    <p><b>Dự đoán:</b> <?= strtoupper($cuoi) ?> (<?= $note ?>)</p>
    <hr>
    <h3>Lịch sử kết quả:</h3>
    <table>
        <tr>
            <th>ID</th><th>Kết quả</th><th>Dự đoán</th><th>Đúng/Sai</th><th>Thời gian</th>
        </tr>
        <?php while($r = $history->fetch_assoc()): ?>
        <tr>
            <td><?= $r['session_id'] ?></td>
            <td><?= $r['result'] ?></td>
            <td><?= $r['prediction'] ?></td>
            <td style="color: <?= $r['is_correct'] ? 'lime' : 'red' ?>;">
                <?= $r['is_correct'] ? 'Đúng' : 'Sai' ?>
            </td>
            <td><?= $r['created_at'] ?></td>
        </tr>
        <?php endwhile; ?>
    </table>
</div>
</body>
</html>
CREATE DATABASE IF NOT EXISTS taixiu CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
USE taixiu;

CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(100),
    password VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS taixiu_history (
    id INT AUTO_INCREMENT PRIMARY KEY,
    session_id INT,
    d1 INT,
    d2 INT,
    d3 INT,
    prediction VARCHAR(10),
    result VARCHAR(10),
    is_correct BOOLEAN,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

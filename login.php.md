##Login
```php
<?php
session_start();
require_once 'config.php';

if (isset($_POST['login'])) {
    $username = $_POST['username'];
    $password = $_POST['password'];

    // Gunakan fungsi PASSWORD() untuk mencocokkan dengan hash di database
    $query = "SELECT * FROM users WHERE username='$username' AND password=PASSWORD('$password')";
    $result = mysqli_query($conn, $query);

    if (mysqli_num_rows($result) == 1) {
        $row = mysqli_fetch_assoc($result);
        $_SESSION['user_id'] = $row['id'];
        $_SESSION['username'] = $row['username'];
        $_SESSION['name'] = $row['name'];

        header("Location: dashboard.php");
        exit();
    } else {
        $error = "Username atau Password salah!";
    }
}
?>

<!DOCTYPE html>
<html>

<head>
    <title>Login</title>
</head>

<body>
    <h2>Login Form</h2>
    <?php if (isset($error)) { ?>
        <p style="color: red;"><?php echo $error; ?></p>
    <?php } ?>

    <form action="" method="POST">
        <p>
            Username: <input type="text" name="username" required>
        </p>
        <p>
            Password: <input type="password" name="password" required>
        </p>
        <p>
            <input type="submit" name="login" value="Login">
        </p>
    </form>
</body>

</html>
```

<?php
if ($_SERVER["REQUEST_METHOD"] == "GET") {
    $titulo = isset($_GET['titulo']) ? htmlspecialchars($_GET['titulo']) : '';
    $genero = isset($_GET['genero']) ? htmlspecialchars($_GET['genero']) : '';
    $autor = isset($_GET['autor']) ? htmlspecialchars($_GET['autor']) : '';
    $editorial = isset($_GET['editorial']) ? htmlspecialchars($_GET['editorial']) : '';
    $fecha_publicacion = isset($_GET['fecha_publicacion']) ? htmlspecialchars($_GET['fecha_publicacion']) : '';

    echo "<h1>Datos del Libro</h1>";
    echo "Título: $titulo<br>";
    echo "Género: $genero<br>";
    echo "Autor: $autor<br>";
    echo "Editorial: $editorial<br>";
    echo "Fecha de Publicación: $fecha_publicacion<br>";
}
?>
<?php
$generos_permitidos = ['Ficción', 'No Ficción', 'Ciencia', 'Historia', 'Biografía', 'Fantasía', 'Terror', 'Romance'];

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $titulo = isset($_POST['titulo']) ? trim($_POST['titulo']) : '';
    $genero = isset($_POST['genero']) ? trim($_POST['genero']) : '';
    $autor = isset($_POST['autor']) ? trim($_POST['autor']) : '';
    $editorial = isset($_POST['editorial']) ? trim($_POST['editorial']) : '';
    $fecha_publicacion = isset($_POST['fecha_publicacion']) ? trim($_POST['fecha_publicacion']) : '';

    $errores = [];

    if (strlen($titulo) > 150) {
        $errores[] = "El título no puede exceder los 150 caracteres.";
    }

    if (strlen($genero) > 8 || !in_array($genero, $generos_permitidos)) {
        $errores[] = "El género no es válido.";
    }

    if (strlen($autor) > 150) {
        $errores[] = "El autor no puede exceder los 150 caracteres.";
    }

    if (DateTime::createFromFormat('Y-m-d', $fecha_publicacion) === false) {
        $errores[] = "La fecha de publicación no es válida.";
    }

    if (strlen($editorial) > 150) {
        $errores[] = "La editorial no puede exceder los 150 caracteres.";
    }

    if (!empty($errores)) {
        foreach ($errores as $error) {
            echo "<p style='color:red;'>$error</p>";
        }
    } else {
        echo "<h1>Datos del Libro</h1>";
        echo "Título: $titulo<br>";
        echo "Género: $genero<br>";
        echo "Autor: $autor<br>";
        echo "Editorial: $editorial<br>";
        echo "Fecha de Publicación: $fecha_publicacion<br>";

      
    }
}
?<?php
$host = 'localhost';
$db = 'biblioteca';
$user = 'usuario';
$pass = 'contraseña';

$conn = new mysqli($host, $user, $pass, $db);

if ($conn->connect_error) {
    die("Conexión fallida: " . $conn->connect_error);
}

if ($_SERVER["REQUEST_METHOD"] == "POST" && empty($errores)) {
    $stmt = $conn->prepare("INSERT INTO libros (titulo, genero, autor, editorial, fecha_publicacion) VALUES (?, ?, ?, ?, ?)");
    $stmt->bind_param("sssss", $titulo, $genero, $autor, $editorial, $fecha_publicacion);
    
    if ($stmt->execute()) {
        echo "Libro registrado exitosamente.";
    } else {
        echo "Error: " . $stmt->error;
    }

    $stmt->close();
}
$conn->close();
?<?php
$host = 'localhost';
$db = 'biblioteca';
$user = 'usuario';
$pass = 'contraseña';

$conn = new mysqli($host, $user, $pass, $db);

if ($conn->connect_error) {
    die("Conexión fallida: " . $conn->connect_error);
}

$search = isset($_GET['search']) ? $_GET['search'] : '';

$sql = "SELECT * FROM libros WHERE titulo LIKE ? OR autor LIKE ?";
$stmt = $conn->prepare($sql);
$search_param = '%' . $search . '%';
$stmt->bind_param("ss", $search_param, $search_param);
$stmt->execute();
$result = $stmt->get_result();

$libros = [];
while ($row = $result->fetch_assoc()) {
    $libros[] = $row;
}

header('Content-Type: application/json');
echo json_encode($libros);

$stmt->close();
$conn->close();
?>
CREATE TABLE libros (
    id INT AUTO_INCREMENT PRIMARY KEY,
    titulo VARCHAR(150),
    genero VARCHAR(8),
    autor VARCHAR(150),
    editorial VARCHAR(150),
    fecha_publicacion DATE
);

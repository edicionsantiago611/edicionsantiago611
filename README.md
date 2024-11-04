<?php
$host = 'localhost';
$db = 'biblioteca';
$user = 'usuario';
$pass = 'contraseña';

$conn = new mysqli($host, $user, $pass, $db);
if ($conn->connect_error) {
    die("Conexión fallida: " . $conn->connect_error);
}

$generos_permitidos = ['Ficción', 'No Ficción', 'Ciencia', 'Historia', 'Biografía', 'Fantasía', 'Terror', 'Romance'];

function sanitize($data) {
    return htmlspecialchars(trim($data));
}

function mostrarErrores($errores) {
    foreach ($errores as $error) {
        echo "<p style='color:red;'>$error</p>";
    }
}

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $titulo = sanitize($_POST['titulo'] ?? '');
    $genero = sanitize($_POST['genero'] ?? '');
    $autor = sanitize($_POST['autor'] ?? '');
    $editorial = sanitize($_POST['editorial'] ?? '');
    $fecha_publicacion = sanitize($_POST['fecha_publicacion'] ?? '');

    $errores = [];

    if (strlen($titulo) > 150) {
        $errores[] = "El título no puede exceder los 150 caracteres.";
    }

    if (!in_array($genero, $generos_permitidos)) {
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
        mostrarErrores($errores);
    } else {
        $stmt = $conn->prepare("INSERT INTO libros (titulo, genero, autor, editorial, fecha_publicacion) VALUES (?, ?, ?, ?, ?)");
        $stmt->bind_param("sssss", $titulo, $genero, $autor, $editorial, $fecha_publicacion);
        
        if ($stmt->execute()) {
            echo "Libro registrado exitosamente.";
        } else {
            echo "Error: " . $stmt->error;
        }
        $stmt->close();
    }
}

if ($_SERVER["REQUEST_METHOD"] == "GET" && isset($_GET['search'])) {
    $search = sanitize($_GET['search']);
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
}

$conn->close();
?>



# Acceso a BBDD

## Conectarnos a una BBDD

### Opción 1: `try` + `finally`

```kotlin
import java.sql.*

fun main() {
    val url = "jdbc:mysql://localhost:3306/mydatabase"
    val usuario = "usuario"
    val contraseña = "contraseña"

    var conexion: Connection? = null

    try {
        Class.forName("com.mysql.cj.jdbc.Driver")
        conexion = DriverManager.getConnection(url, usuario, contraseña)
        println("Conexión realizada con éxito")
        // Aquí se pueden hacer consultas o inserciones
    } catch (e: SQLException) {
        println("Error en la conexión: ${e.message}")
    } catch (e: ClassNotFoundException) {
        println("No se encontró el driver JDBC: ${e.message}")
    } finally {
        try {
            conexion?.close()
            println("Conexión cerrada")
        } catch (e: SQLException) {
            println("Error al cerrar la conexión: ${e.message}")
        }
    }
}
```

| Línea | Explicación |
|-------|-------------|
| **(1)** `Class.forName(...)` | Fuerza la carga del driver JDBC necesario para MySQL. (En Java moderno puede no ser necesario, pero se usa por compatibilidad). |
| **(2)** `DriverManager.getConnection(...)` | Establece la conexión con la base de datos usando la URL, usuario y contraseña. Devuelve un objeto `Connection`. |
| **(3)** `catch (SQLException)` | Captura errores relacionados con la conexión o la ejecución de comandos SQL. |
| **(4)** `catch (ClassNotFoundException)` | Captura errores si el driver JDBC no se encuentra. |
| **(5)** `conexion?.close()` | Cierra la conexión si no es `null`. Esto libera recursos. Es **imprescindible** para evitar fugas. |


### Opción 2: `use {}` (idiomática en Kotlin)

```kotlin
import java.sql.*

fun main() {
    val url = "jdbc:mysql://localhost:3306/mydatabase"
    val usuario = "usuario"
    val contraseña = "contraseña"

    try {
        Class.forName("com.mysql.cj.jdbc.Driver")
        DriverManager.getConnection(url, usuario, contraseña).use { conexion -> 
            println("Conexión exitosa") 
            // Aquí puedes hacer operaciones con `conexion`
        } 
    } catch (e: SQLException) {
        println("Error en la conexión: ${e.message}")
    } catch (e: ClassNotFoundException) {
        println("No se encontró el driver JDBC: ${e.message}")
    }
}
```

| Línea | Explicación |
|-------|-------------|
| **(1)** `use {}` es una función Kotlin para manejar recursos que implementan `AutoCloseable`. Al finalizar el bloque, llama automáticamente a `conexion.close()`. |
| **(2)** Al salir del bloque, **`conexion.close()` se ejecuta automáticamente**, incluso si hubo una excepción. |

---

## CRUD básico con H2

### ¿Qué es H2?

- **H2** es una base de datos relacional escrita en Java.
- Se ejecuta **en memoria** o se guarda en un **archivo local**.
- Usa **sintaxis SQL estándar compatible con otras BBDD como PostgreSQL o MySQL**.
- Ideal para **aplicaciones pequeñas, pruebas unitarias, prototipos o entornos de desarrollo**.

### PASOS para configurar y crear tu base de datos con H2 en un proyecto Kotlin

#### 1. Agrega la dependencia en `build.gradle.kts`

```kotlin
dependencies {
    implementation("com.h2database:h2:2.2.224")
}
```

#### 2. Define la URL de conexión

En el código Kotlin, define la URL como:

```kotlin
// Modo archivo persistente (guarda la base de datos en ./data/dbtest.mv.db)
const val DB_URL = "jdbc:h2:./data/dbtest"

// Modo memoria (base de datos temporal, se borra al cerrar la app)
const val DB_URL = "jdbc:h2:mem:testdb"
```

También es posible añadir `;DB_CLOSE_DELAY=-1` al final para que **no se borre la base de datos en memoria** mientras la app siga viva:

```kotlin
const val DB_URL = "jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1"
```

#### 3. Crea la base de datos *(es automática)*

No es necesario utilizar comandos externos, ni consola. **Cuando hacemos `DriverManager.getConnection(...)`**, si no existe la base de datos, **¡se crea sola!**

```kotlin
val conn = DriverManager.getConnection(DB_URL, USER, PASS)
```

#### 4. Crea las tablas (una sola vez)

Debes ejecutar un script SQL para crear tu tabla al inicio:

```kotlin
fun initDatabase() {
    getConnection().use { conn ->
        val stmt = conn.createStatement()
        stmt.executeUpdate("""
            CREATE TABLE IF NOT EXISTS users (
                id INT AUTO_INCREMENT PRIMARY KEY,
                name VARCHAR(255),
                email VARCHAR(255)
            )
        """.trimIndent())
        stmt.close()
    }
}
```

#### 5. Ejecuta operaciones como en cualquier base de datos

Una vez creada la tabla, puedes hacer `INSERT`, `SELECT`, `UPDATE`, `DELETE` igual que lo harías en MySQL o PostgreSQL.

### Consejos útiles

| Opción | Uso |
|-------|-----|
| `AUTO_INCREMENT` | Genera IDs automáticamente en H2. |
| `VARCHAR(255)` | Define una cadena de texto de hasta 255 caracteres. |
| `.use { }` | Asegura que se cierre `Connection`, `Statement`, etc. automáticamente. |
| `IF NOT EXISTS` | Evita errores si ejecutas varias veces el script de creación. |

### ¿Dónde se guarda el archivo de la base de datos?

- Si usas: `jdbc:h2:./data/dbtest`  
  → se guarda en el proyecto en la ruta: `./data/dbtest.mv.db`

- Puedes abrir ese archivo con [H2 Console](http://www.h2database.com/html/main.html) o acceder desde código Kotlin.


```kotlin
import java.sql.*

const val DB_URL = "jdbc:h2:./data/dbtest"  // crea archivo ./data/dbtest.mv.db
const val USER = "sa"
const val PASS = ""

// Modelo
data class User(val id: Int, val name: String, val email: String)

fun getConnection(): Connection =
    DriverManager.getConnection(DB_URL, USER, PASS)

fun initDatabase() {
    getConnection().use { conn ->
        val stmt = conn.createStatement()
        stmt.executeUpdate(
            """CREATE TABLE IF NOT EXISTS users (
                id INT AUTO_INCREMENT PRIMARY KEY,
                name VARCHAR(255),
                email VARCHAR(255)
            )"""
        )
        stmt.close()
    }
}

fun createUser(name: String, email: String) {
    getConnection().use { conn ->
        val stmt = conn.prepareStatement("INSERT INTO users (name, email) VALUES (?, ?)")
        stmt.setString(1, name)
        stmt.setString(2, email)
        stmt.executeUpdate()
    }
}

fun readUsers(): List<User> {
    val users = mutableListOf<User>()
    getConnection().use { conn ->
        val stmt = conn.createStatement()
        val rs = stmt.executeQuery("SELECT * FROM users")
        while (rs.next()) {
            users += User(rs.getInt("id"), rs.getString("name"), rs.getString("email"))
        }
    }
    return users
}

fun updateUser(id: Int, name: String, email: String) {
    getConnection().use { conn ->
        val stmt = conn.prepareStatement("UPDATE users SET name = ?, email = ? WHERE id = ?")
        stmt.setString(1, name)
        stmt.setString(2, email)
        stmt.setInt(3, id)
        stmt.executeUpdate()
    }
}

fun deleteUser(id: Int) {
    getConnection().use { conn ->
        val stmt = conn.prepareStatement("DELETE FROM users WHERE id = ?")
        stmt.setInt(1, id)
        stmt.executeUpdate()
    }
}

fun main() {
    initDatabase()

    createUser("Juan", "juan@example.com")
    readUsers().forEach(::println)

    updateUser(1, "Juan Pérez", "juan.perez@example.com")
    readUsers().forEach(::println)

    deleteUser(1)
    readUsers().forEach(::println)
}
```

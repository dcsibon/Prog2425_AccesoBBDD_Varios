Claro, aquí tienes una explicación clara, ordenada y fácil de entender sobre los métodos de acceso a bases de datos relacionales. Puedes usarlo tal cual para tus apuntes o para explicárselo a tus alumnos.

⸻

3. Métodos de Acceso a Bases de Datos Relacionales

Cuando una aplicación necesita guardar, consultar o modificar información en una base de datos relacional (como MySQL, PostgreSQL, Oracle, etc.), puede hacerlo usando diferentes métodos o tecnologías. A continuación, se explican los más utilizados en el desarrollo con Java y Kotlin:

⸻

1. JDBC (Java Database Connectivity)

¿Qué es?

Es una API estándar de Java que permite conectar una aplicación a cualquier base de datos relacional mediante instrucciones SQL.

¿Cómo funciona?
	•	Se establece una conexión mediante una URL de conexión, usuario y contraseña.
	•	Se usa código Java y SQL directamente para hacer consultas (SELECT, INSERT, UPDATE, DELETE).
	•	Es necesario gestionar manualmente la conexión, preparar consultas y cerrar recursos.

Ventajas:
	•	Muy flexible y directo.
	•	Compatible con cualquier base de datos que tenga un driver JDBC.

Inconvenientes:
	•	Más código y mayor complejidad.
	•	Alto riesgo de errores si no se gestionan bien los recursos.

Ejemplo:

Connection conn = DriverManager.getConnection(url, user, pass);
PreparedStatement stmt = conn.prepareStatement("SELECT * FROM usuarios");
ResultSet rs = stmt.executeQuery();



⸻

2. ORM (Object-Relational Mapping)

¿Qué es?

Es una técnica que permite mapear clases y objetos del lenguaje de programación con tablas y filas de una base de datos. Permite trabajar con la base de datos usando objetos en lugar de SQL.

¿Cómo funciona?
	•	Define clases que representan tablas.
	•	Cada instancia representa una fila.
	•	Las operaciones se realizan a través de métodos de objetos, no de sentencias SQL manuales.

Ejemplos de ORMs:
	•	Hibernate (Java)
	•	Exposed (Kotlin)
	•	JPA (que veremos después) también es un ORM, pero estandarizado.

Ventajas:
	•	Código más limpio y orientado a objetos.
	•	Evita SQL repetitivo.
	•	Permite cambiar de base de datos con pocos cambios.

Inconvenientes:
	•	Puede ocultar detalles del funcionamiento real de la base de datos.
	•	Más difícil de depurar si no se conoce bien.

⸻

3. JPA (Java Persistence API)

¿Qué es?

Es una especificación oficial de Java para gestionar la persistencia de objetos (guardar objetos en bases de datos).

No es una librería, es una interfaz común. Para usar JPA necesitas una implementación concreta como Hibernate.

¿Cómo funciona?
	•	Utiliza anotaciones (@Entity, @Id, etc.) para definir qué clases se van a guardar.
	•	Permite hacer consultas con JPQL (Java Persistence Query Language), similar al SQL pero orientado a objetos.

Ventajas:
	•	Estandariza el uso del ORM en Java.
	•	Fácil integración con otros frameworks como Spring.
	•	Reduce el acoplamiento con la base de datos.

Inconvenientes:
	•	Curva de aprendizaje si no se conoce ORM.
	•	Puede ser excesivo para proyectos pequeños.

⸻

4. Spring Data

¿Qué es?

Es una abstracción del acceso a datos dentro del ecosistema Spring. Permite usar diferentes tecnologías de base de datos (JDBC, JPA, MongoDB…) de forma unificada y simplificada.

¿Cómo funciona?
	•	Define interfaces como UserRepository.
	•	Spring genera automáticamente las implementaciones necesarias (sin escribir SQL).
	•	Permite combinar varias fuentes de datos si es necesario.

Ventajas:
	•	Muy fácil de usar y rápido de configurar.
	•	Requiere poco código: solo definir interfaces.
	•	Integración automática con Spring Boot.

Inconvenientes:
	•	Está muy acoplado a Spring Framework.
	•	En proyectos muy pequeños puede ser innecesario.

⸻

🔁 Resumen Comparativo

Método	Nivel	SQL manual	Orientado a objetos	Ideal para…
JDBC	Bajo	✅ Sí	❌ No	Proyectos simples, máximo control
ORM (Hibernate, Exposed)	Medio	❌ No	✅ Sí	Aplicaciones grandes
JPA	Medio	❌ No	✅ Sí	Apps estándar Java EE/Spring
Spring Data	Alto	❌ No	✅ Sí	Apps Spring modernas



⸻

¿Quieres que prepare un esquema gráfico o infografía resumen para esto también?

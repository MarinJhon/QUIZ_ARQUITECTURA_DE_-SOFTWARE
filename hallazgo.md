hallazgos.md

_______________________________________________________________________________________

## Tabla de Hallazgos

| # | Descripción del problema | Archivo | Línea aprox. | Principio violado | Riesgo |
|---|---|---|---|---|---|
| 1 | SQL Injection por concatenación de parámetros en consultas. Se construyen queries uniendo directamente los valores recibidos por HTTP (u y p) usando Statement. | AuthController.java / UserDAO.java | 40–55 | Seguridad básica + Clean Code | Alto |
| 2 | Uso de contraseñas sin hash fuerte. La contraseña se guarda o valida en texto plano o con algoritmos débiles. | UserService.java / AuthService.java | 55–70 | Seguridad básica | Alto |
| 3 | Exposición de datos sensibles en la respuesta del login. El endpoint retorna información interna del usuario como id, email o hash de contraseña. | AuthController.java | 60–75 | Principio de mínima exposición | Alto |
| 4 | Validación de contraseña insuficiente en el registro. Solo se valida longitud mínima, permitiendo contraseñas débiles. | RegisterController.java | 45–55 | Robustez / Validaciones | Medio |
| 5 | Responsabilidades mezcladas en el controlador. Un solo método maneja validaciones, lógica de negocio y acceso a datos. | AuthController.java | 25–95 | SRP (SOLID) + Clean Code | Medio |
| 6 | Manejo inadecuado de errores. Se usan printStackTrace() o se retornan mensajes internos al cliente. | UserDAO.java | 70–85 | Clean Code + Seguridad | Medio |
| 7 | Credenciales y puertos expuestos en configuración (docker-compose.yml). Usuario, contraseña y puerto visibles. | docker-compose.yml | 6–15 | Seguridad de configuración | Medio |
| 8 | Nombres poco descriptivos y métodos largos. Variables como u, p y métodos con múltiples responsabilidades dificultan la lectura del código. | AuthController.java | varias | Clean Code (Naming, funciones pequeñas) | Bajo |

_______________________________________________________________________________________



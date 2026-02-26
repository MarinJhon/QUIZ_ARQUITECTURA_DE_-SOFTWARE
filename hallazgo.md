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


## FASE 3 — Pruebas funcionales (evidencia con Postman)

### Prueba 1 — Login válido  
**Request:**  
POST `http://localhost:8080/login?u=admin&p=12345`

**Resultado observado:**  
```json
{
  "ok": true,
  "user": "admin",
  "hash": "827ccb0eea8a706c4c34a16891f84e7b"
}
Datos sensibles que aparecen en la respuesta:

Se retorna el hash MD5 de la contraseña (hash).

Las credenciales se envían por query params (u, p), lo que puede quedar registrado en historiales y logs.

## ¿Debería retornarse eso?

No. Un endpoint de login no debe retornar hashes ni información derivada de la contraseña.
Lo correcto sería retornar solo un estado (ok) y, si aplica, un token de autenticación o datos públicos mínimos del usuario.

## Prueba 2 — SQL Injection

Request:
POST `http://localhost:8080/login?u=admin'--&p=cualquiercosa`

**Resultado observado:**  
```json
{
  "ok": false,
  "hash": "f73862908453012d17eb1d60240d95d1"
}

## ¿Qué ocurrió?

El login falla (ok: false), pero aun así el sistema retorna un hash de la contraseña ingresada.

El payload admin'-- es un intento clásico de SQL Injection (el -- comenta el resto de la consulta SQL).

## ¿Por qué es peligroso en producción?

Indica que el backend procesa directamente entradas del usuario en consultas (riesgo de SQL Injection).

Aunque este payload no logró autenticarse, la vulnerabilidad existe y puede explotarse con otros payloads.

Devolver hashes incluso en intentos fallidos es una fuga de información sensible.

## Prueba 3 — Registro con contraseña débil
Request:
POST `http://localhost:8080/register?u=test&p=123&e=test@test.com`

**Resultado observado:**  
```json
{ "ok": false }

Request:
POST `http://localhost:8080/register?u=test&p=1234&e=test@test.com`

**Resultado observado:**  
```json
{
  "ok": true,
  "user": "test"
}

## ¿Cuál fue rechazado?

❌ p=123 → Rechazado

✅ p=1234 → Aceptado

## ¿Es esa una validación suficiente?

No. Validar solo “mínimo 4 caracteres” es una política de contraseñas muy débil.
Una validación mínima aceptable debería incluir:

Longitud ≥ 8 caracteres

Combinación de letras y números (y opcionalmente símbolos)

Bloqueo de contraseñas comunes

Hashing fuerte para almacenamiento (BCrypt o Argon2)
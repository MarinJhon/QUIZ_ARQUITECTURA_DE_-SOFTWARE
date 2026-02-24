001-template
ADR


Contexto

El sistema actual implementa un módulo básico de autenticación y registro de usuarios que funciona, pero fue construido de forma rápida y sin tener en cuenta buenas prácticas de seguridad ni principios de Clean Code. Durante la auditoría se identificaron problemas críticos como consultas SQL construidas por concatenación de parámetros, lo que permite ataques de SQL Injection, además de un manejo inseguro de contraseñas y exposición de información sensible en las respuestas del login.
También se observó que los controladores concentran demasiadas responsabilidades (validación, lógica de negocio y acceso a datos en un mismo método), lo que hace que el código sea difícil de leer, mantener y extender.
Estos problemas no solo representan un riesgo alto en un entorno real de producción, sino que además complican el trabajo del equipo de desarrollo, ya que cualquier cambio pequeño en el módulo de autenticación puede generar errores colaterales.
La situación obliga a tomar una decisión de refactorización para mejorar la seguridad del sistema y, al mismo tiempo, organizar el código siguiendo principios de diseño más claros.
Si no se interviene, el sistema queda expuesto a accesos no autorizados y a una deuda técnica que crecerá con el tiempo.

Decisión

Se decidió refactorizar el módulo de autenticación aplicando buenas prácticas de seguridad y principios de diseño para reducir riesgos y mejorar la mantenibilidad del código.
Primero, se reemplazará el uso de Statement y la concatenación de strings en las consultas SQL por PreparedStatement, con el fin de eliminar la posibilidad de SQL Injection. Esta decisión se toma porque es una solución directa, probada y de bajo impacto en la arquitectura actual.
Segundo, se implementará el uso de hashing fuerte de contraseñas (por ejemplo, BCrypt) para el registro y la validación de usuarios. De esta manera, las contraseñas no se manejarán en texto plano ni con algoritmos débiles, reduciendo el impacto de una posible fuga de datos.
Tercero, se separarán las responsabilidades del controlador creando una capa de servicio para la lógica de autenticación y una capa de acceso a datos para las consultas a la base de datos. Esto permite aplicar el principio de responsabilidad única (SRP) y dejar al controlador como una capa ligera que solo recibe la petición y devuelve la respuesta.
Finalmente, se limitará la información devuelta en las respuestas del login, retornando solo lo necesario para el cliente (por ejemplo, un mensaje de éxito o datos básicos), evitando exponer información interna del usuario.

Consecuencias

Consecuencias positivas:
La principal ganancia es una mejora significativa en la seguridad del sistema, ya que se eliminan vectores críticos como SQL Injection y el manejo inseguro de contraseñas. Además, el código se vuelve más claro y fácil de mantener al separar responsabilidades y reducir métodos largos con múltiples funciones. Esto facilita que otros desarrolladores entiendan el módulo de autenticación y puedan modificarlo sin miedo a romper partes no relacionadas.

Consecuencias negativas o riesgos:
La refactorización implica un tiempo adicional de desarrollo y pruebas, lo que puede retrasar otras tareas del proyecto. También existe el riesgo de introducir errores durante el cambio si no se realizan pruebas suficientes. En el corto plazo, el proyecto puede verse afectado por una mayor complejidad estructural (más clases y capas), aunque este costo se justifica por los beneficios a mediano y largo plazo.


Alternativas consideradas

Reescribir completamente el módulo de autenticación desde cero.
Esta opción fue descartada porque implica un mayor esfuerzo de desarrollo, más tiempo de pruebas y un riesgo alto de introducir nuevos errores. Para el contexto del taller y del proyecto actual, una refactorización progresiva es más viable que una reescritura total.

Corregir solo los problemas de seguridad sin mejorar la estructura del código.
Esta alternativa se descartó porque, aunque reduciría el riesgo inmediato de ataques, dejaría el código con problemas de diseño (métodos largos, responsabilidades mezcladas, nombres poco claros). A mediano plazo, esto mantendría la deuda técnica y dificultaría futuras modificaciones o mejoras en el módulo de autenticación.
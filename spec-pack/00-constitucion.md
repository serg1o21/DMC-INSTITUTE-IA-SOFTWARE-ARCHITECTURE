# 00 - Constitución del proyecto miStorePe

miStorePe es una aplicación web responsiva que gestiona el inventario de emprendedores que venden ropa, perfumes y accesorios por redes sociales en Lima, Perú.

Este documento es lo primero que todo agente de IA y toda persona debe leer antes de trabajar en el proyecto. Sus principios son no negociables. Solo el responsable del proyecto (Sergio) puede modificarlos, mediante un ADR aprobado.

Historial de cambios: 2026-11-10, PRINCIPIO-003 (ADR 0002), PRINCIPIO-005 (ADR 0010) y PRINCIPIO-012 (ADR 0013).

---

## PRINCIPIO-001: Idioma del código y de la documentación
Todo el código fuente (identificadores, nombres de archivos, comentarios, mensajes de commit y nombres de ramas) se escribe en inglés. Toda la documentación y todos los textos visibles para el usuario se escriben únicamente en español (es-PE).

Ejemplo de cumplimiento: `ProductService.findLowStockItems()` muestra en pantalla el mensaje "Stock bajo: quedan 3 unidades".
Ejemplo de violación: una clase `ServicioProducto`, o un botón con el texto "Save" en la interfaz.

## PRINCIPIO-002: Stack tecnológico fijo
El backend usa Java 21 con Spring Boot 4.x, la base de datos es PostgreSQL 18 y el frontend es Angular. La app se despliega en hosting propio. Cualquier lenguaje, framework, base de datos o servicio externo distinto requiere un ADR aprobado antes de escribir código.

Ejemplo de cumplimiento: se añade un módulo de reportes con Spring Boot y Angular, sin tecnologías nuevas.
Ejemplo de violación: introducir MongoDB, React o un servicio de nube externo sin ADR aprobado.

## PRINCIPIO-003: Aislamiento total entre emprendedores
Cada emprendedor solo puede ver y modificar sus propios datos. Toda tabla de negocio tiene una columna `tenant_id` NOT NULL, y toda consulta filtra por el `tenant_id` del usuario autenticado, nunca por un valor enviado por el cliente. Acceder a un recurso de otro emprendedor devuelve HTTP 404. La cuenta con rol Administrador no tiene `tenant_id` y solo accede al nombre, el correo y el estado de las cuentas; nunca accede a datos de inventario. La tabla de cuentas no es una tabla de negocio (ADR 0002).

Ejemplo de cumplimiento: existe un test de integración por cada endpoint que verifica que el usuario A recibe 404 al pedir un producto del usuario B.
Ejemplo de violación: un endpoint `GET /products?tenantId=15` que confía en el `tenantId` del request.
Ejemplo de violación: un endpoint de administración que devuelve los productos o ventas de un emprendedor, o un parámetro que desactiva el filtro por `tenant_id` en las consultas de inventario.

## PRINCIPIO-004: Autenticación y cifrado
El login es con correo y contraseña. Las contraseñas se almacenan solo como hash Argon2id (`Argon2PasswordEncoder` de Spring Security), nunca en texto plano ni con MD5/SHA-1/SHA-256 sin sal. Las contraseñas tienen mínimo 12 caracteres y la cuenta se bloquea 15 minutos tras 5 intentos fallidos. Todo el tráfico usa HTTPS con TLS 1.2 o superior y cabecera HSTS. Los discos de la base de datos y los backups se cifran con AES-256.

Ejemplo de cumplimiento: la columna `password_hash` contiene valores con prefijo `$argon2id$`, y el puerto 80 solo redirige a 443.
Ejemplo de violación: guardar contraseñas con `MessageDigest.getInstance("SHA-256")`, o un backup `.sql` sin cifrar en el servidor.

## PRINCIPIO-005: Solo datos de inventario, nunca de clientes finales
El sistema almacena únicamente datos de inventario y los datos mínimos de la cuenta del emprendedor (nombre, correo y fecha de aceptación de la política de privacidad, ADR 0010). Está prohibido guardar datos personales de los clientes finales (nombre, teléfono, dirección, DNI) y datos de pago. El tratamiento de los datos de los emprendedores cumple la Ley N° 29733 y su reglamento (D.S. N° 016-2024-JUS): política de privacidad visible y aceptada en el registro, y notificación a la ANPD dentro de 48 horas tras conocer un incidente de seguridad.

Ejemplo de cumplimiento: la entidad `Product` tiene nombre y categoría, y la entidad `Variant` tiene SKU, talla, color, presentación, precio y stock, sin campos de comprador.
Ejemplo de violación: añadir una tabla `customers` con columnas `phone` o `dni`.

## PRINCIPIO-006: Eliminación de datos al cerrar una cuenta
Al cerrar una cuenta, todos sus datos (cuenta e inventario) se eliminan de forma definitiva en un plazo máximo de 30 días, mediante un proceso automático. Los backups que contengan esos datos no se conservan más de 30 días.

Ejemplo de cumplimiento: un job programado diario borra las cuentas con `closed_at` mayor a 30 días, y un test automatizado lo verifica.
Ejemplo de violación: marcar la cuenta como inactiva y conservar sus productos indefinidamente.

## PRINCIPIO-007: Gestión de secretos
Ningún secreto (contraseñas, claves, tokens, certificados) se escribe en el repositorio, en logs ni en documentación. Los secretos se inyectan por variables de entorno o un gestor de secretos del servidor. El CI ejecuta gitleaks en cada pull request y falla si detecta un secreto.

Ejemplo de cumplimiento: `application.yml` usa `${DB_PASSWORD}` y `.env` está en `.gitignore`.
Ejemplo de violación: `spring.datasource.password=MiClave123` en un archivo versionado, o imprimir el token JWT en un log.

## PRINCIPIO-008: Acciones prohibidas para agentes de IA
Ningún agente de IA puede, bajo ninguna circunstancia: (1) desplegar a producción; (2) ejecutar sentencias contra cualquier base de datos real, ni de datos ni de esquema (los cambios de esquema solo se proponen como migraciones dentro de un pull request); (3) añadir, quitar o actualizar dependencias sin aprobación previa de Sergio; (4) hacer merge de un pull request; (5) leer, crear, guardar o exponer secretos.

Ejemplo de cumplimiento: el agente abre un pull request con la migración `V12__add_size_column.sql` y una nota pidiendo aprobación para añadir una dependencia.
Ejemplo de violación: el agente ejecuta `ALTER TABLE` en el servidor, agrega una librería a `pom.xml` sin preguntar o hace merge de su propio PR.

## PRINCIPIO-009: Preguntar antes de asumir
Un agente de IA no implementa nada que no esté en una especificación aprobada por Sergio. Ante cualquier ambigüedad o decisión no documentada, el agente pregunta y espera respuesta antes de escribir código.

Ejemplo de cumplimiento: el agente pregunta "¿El stock puede ser negativo en preventas?" antes de implementar la validación.
Ejemplo de violación: el agente decide por su cuenta que el stock negativo está permitido y lo implementa.

## PRINCIPIO-010: Calidad y testing obligatorios
La lógica de negocio (capas de dominio y servicio) mantiene al menos 80% de cobertura de líneas, medida con JaCoCo en el backend y con el reporte de cobertura de Angular en el frontend. Cada funcionalidad tiene tests unitarios y al menos un test end-to-end de su flujo principal. Ningún pull request se fusiona sin el CI en verde y sin la aprobación de Sergio.

Ejemplo de cumplimiento: el CI ejecuta tests unitarios, E2E y cobertura, y bloquea el merge si la cobertura baja de 80%.
Ejemplo de violación: fusionar un PR con un test deshabilitado (`@Disabled`) para que el CI pase.

## PRINCIPIO-011: Accesibilidad y diseño responsivo
La interfaz cumple WCAG 2.2 nivel AA y funciona sin scroll horizontal desde 360 px de ancho. Los tests E2E ejecutan axe-core en cada pantalla y fallan ante cualquier violación de severidad seria o crítica.

Ejemplo de cumplimiento: todos los campos de formulario tienen `<label>` asociado y el texto tiene contraste mínimo de 4.5:1.
Ejemplo de violación: un botón solo con ícono y sin `aria-label`, o una tabla de inventario que se desborda en un celular.

## PRINCIPIO-012: Documentación en Markdown, ADRs y especificaciones
Toda la documentación vive en el repositorio en formato Markdown. Cada decisión técnica relevante se registra como ADR en `docs/adr/NNNN-titulo.md` y cada funcionalidad tiene su especificación en `docs/specs/`, aprobada antes de implementarla; esa especificación es el Spec Pack (reglas, historias, arquitectura y backlog), sin copias por funcionalidad (ADR 0013).

Ejemplo de cumplimiento: el cambio de estrategia de caché va acompañado de `docs/adr/0007-cache-strategy.md`.
Ejemplo de violación: documentar una decisión solo en un chat o en un archivo Word fuera del repositorio.

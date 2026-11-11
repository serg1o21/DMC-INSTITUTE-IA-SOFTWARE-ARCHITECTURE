# 02 - Alcance: miStorePe

Basado en `00-constitucion.md` y `01-project-brief.md`. Fecha: 2026-11-10.

Objetivo de esta versión: que 3 emprendedores usen miStorePe durante 1 mes de piloto en lugar del cuaderno, y que al menos 2 de 3 decidan seguir usándolo (KPI-02).

---

## 1. IN-SCOPE

### Must have (MVP)

| ID | Funcionalidad | Descripción | Prioridad |
|---|---|---|---|
| F-01 | Cuenta del emprendedor | Registro e inicio de sesión con correo y contraseña, cambio de contraseña, cierre de sesión tras 8 horas sin actividad, aceptación de la política de privacidad, datos aislados por emprendedor y cierre de cuenta (reabrible en 30 días) con borrado en 30 días. | M |
| F-02 | Catálogo de productos con variantes | Crear, editar y desactivar productos con categoría (ropa, perfume, accesorio), al menos una variante (Ropa: talla y color obligatorios; Perfume: presentación en ml obligatoria y vencimiento opcional; Accesorio: atributos opcionales, con variante "Única" automática) y precio de venta. Cada variante recibe un SKU automático. El costo no se define aquí: se registra en cada entrada (F-03). | M |
| F-03 | Registro de entradas de mercadería | Registrar el ingreso de unidades por variante con su costo unitario cuando llega mercadería de un proveedor, y anular una entrada registrada por error. | M |
| F-04 | Registro de salidas por venta | Registrar una venta con una o más líneas (una por variante), cada una con cantidad y precio final por unidad, descontando el stock al instante y bloqueando ventas sin stock suficiente, y anular una venta registrada por error. | M |
| F-05 | Consulta y ajuste de stock | Ver y buscar desde el celular cuántas unidades quedan de cada variante, y corregir el stock con un ajuste por conteo físico que queda en el historial. | M |
| F-06 | Ganancia por venta | Calcular y mostrar la ganancia de cada venta como (precio final por unidad menos costo unitario promedio) por la cantidad (RN-027), y el historial de ventas. | M |

### Should have (después del MVP)

| ID | Funcionalidad | Descripción | Prioridad |
|---|---|---|---|
| F-07 | Alerta de stock bajo | Mostrar dentro de la app las variantes que bajan de un mínimo definido por el emprendedor. | S |
| F-08 | Productos sin movimiento | Listar los productos sin ventas en los últimos 30 días para decidir qué no reponer. | S |
| F-09 | Resumen de ganancias por periodo | Mostrar ventas, costo y ganancia total por semana y por mes. | S |
| F-16 | Administración de cuentas | Pantalla para que el Administrador (Responsable del producto) vea las cuentas y suspenda o reactive las que no pagan; sin acceso al inventario. Necesaria antes de que termine la gratuidad. | S |
| F-17 | Registro de mermas con costo | Registrar salidas por merma (por ejemplo, perfumes vencidos) con su costo como pérdida, para ver cuánto se pierde por mes. Después del piloto. | S |

### Could have (deseables)

| ID | Funcionalidad | Descripción | Prioridad |
|---|---|---|---|
| F-10 | Alerta de vencimiento de perfumes | Mostrar las presentaciones que vencen en los próximos 30 días. | C |
| F-11 | Recuperación de contraseña por correo | Permitir restablecer la contraseña con un enlace temporal enviado al correo. | C |
| F-12 | Exportar reportes | Descargar el stock y las ventas en Excel y PDF. | C |
| F-13 | Gráficos estadísticos | Ver gráficos de ventas y ganancias por producto y por periodo. | C |
| F-14 | Ayudantes de solo lectura | Invitar a otra persona del negocio a consultar el stock sin poder registrar ventas. | C |
| F-15 | Registro de proveedores | Guardar proveedores y asociarlos a las entradas de mercadería. | C |

---

## 2. OUT-OF-SCOPE

| ID | Funcionalidad (Won't) | Razón |
|---|---|---|
| W-01 | Cobro de la suscripción dentro de la app | El piloto es gratuito y la constitución prohíbe guardar datos de pago (PRINCIPIO-005). El cobro de S/ 25 se gestiona fuera de la app. |
| W-02 | Boletas y facturas electrónicas SUNAT | Fuera de alcance según el brief; no se emiten comprobantes. |
| W-03 | Integración con TikTok, Facebook, Instagram o WhatsApp | Fuera de alcance según el brief; las ventas se registran manualmente. |
| W-04 | Registro de clientes finales (CRM, pedidos con datos del comprador) | La constitución prohíbe guardar datos de clientes finales (PRINCIPIO-005). |
| W-05 | Aplicación móvil nativa (Android/iOS) | La constitución fija una web responsiva con Angular (PRINCIPIO-002). |
| W-06 | Login con Google, código por SMS/WhatsApp o 2FA | Se definió solo correo y contraseña. |
| W-07 | Salidas por robo o devolución | Según el brief, las salidas se dan solo por ventas. Las diferencias se corrigen con el ajuste por conteo físico (F-05). Las mermas por vencimiento se retiran con ajuste en el MVP y tendrán registro propio en F-17. |
| W-08 | Varios almacenes o tiendas físicas | El segmento objetivo es el pequeño emprendedor que vende por redes. |
| W-09 | Interfaz en otros idiomas | La constitución exige textos de interfaz solo en español (PRINCIPIO-001). |
| W-10 | Predicción de demanda con IA | Con 30-40 productos y 3 usuarios no hay datos suficientes; no resuelve un problema del brief. |
| W-11 | Varios usuarios registrando ventas | El brief define un único responsable de las salidas por ventas. |
| W-12 | Cambio de correo de la cuenta | Decidido fuera del MVP. |

---

## 3. MVP

Con el MVP, el emprendedor crea su cuenta con correo y contraseña y registra desde el celular o la laptop sus 30-40 productos, cada uno con sus tallas y colores o sus presentaciones en ml (con fecha de vencimiento si la tienen) y su precio de venta. Cuando llega mercadería, registra la entrada de unidades con su costo; cuando vende en un live de TikTok o por Facebook Marketplace, registra la salida y el stock se descuenta al instante. En cualquier momento puede consultar cuántas unidades le quedan de cada variante sin volver a contar físicamente, corregir el stock con un ajuste por conteo físico si olvidó registrar una venta, y ver cuánto ganó en cada venta. Si deja de usar la app, puede cerrar su cuenta y sus datos se borran en 30 días.

---

## 4. Supuestos

| ID | Supuesto |
|---|---|
| SUP-01 | El emprendedor registrará cada venta, incluso durante un live de TikTok. |
| SUP-02 | El emprendedor conoce el costo unitario de cada producto que compra. |
| SUP-03 | 30-40 productos es el volumen típico y entra en una lista navegable desde el celular sin funciones extra de búsqueda avanzada. |
| SUP-04 | Una sola persona registra las ventas del negocio. |
| SUP-05 | Las salidas de inventario se dan solo por ventas; las diferencias por ventas no registradas se corrigen con el ajuste por conteo físico. |
| SUP-06 | El emprendedor pagará S/ 25 al mes después de la etapa gratuita. |
| SUP-07 | Con 3 usuarios en el piloto, Sergio puede restablecer contraseñas manualmente hasta tener F-11. |
| SUP-08 | Los accesorios no necesitan atributos obligatorios; la variante "Única" cubre los que no tienen tallas ni colores. |

---

## 5. Restricciones

### Técnicas
- Java 21, Spring Boot 4.x, PostgreSQL 18 y Angular, en hosting propio (PRINCIPIO-002).
- Web responsiva sin scroll horizontal desde 360 px y WCAG 2.2 AA (PRINCIPIO-011).
- Aislamiento por `tenant_id` en toda tabla de negocio (PRINCIPIO-003).
- Contraseñas con Argon2id, HTTPS con TLS 1.2 o superior, discos y backups con AES-256 (PRINCIPIO-004).
- Cobertura de 80% en lógica de negocio, tests unitarios y E2E, CI en verde antes de merge (PRINCIPIO-010).

### Legales
- Ley N° 29733 y su reglamento (D.S. N° 016-2024-JUS): política de privacidad aceptada en el registro y notificación a la ANPD en 48 horas ante un incidente (PRINCIPIO-005).
- Borrado definitivo de datos 30 días después del cierre de cuenta, incluidos los backups (PRINCIPIO-006).
- Sin datos de clientes finales ni datos de pago (PRINCIPIO-005).

### De recursos
- Una sola persona desarrolla y aprueba todos los cambios.
- Sin fecha de entrega definida.
- Hay presupuesto para hosting.
- Los agentes de IA no despliegan, no hacen merge, no tocan bases de datos reales ni añaden dependencias sin aprobación (PRINCIPIO-008).

---

## 6. Dependencias

| ID | Dependencia | Para qué | Necesaria desde |
|---|---|---|---|
| D-01 | Servidor de hosting propio | Ejecutar backend, frontend y base de datos | MVP |
| D-02 | PostgreSQL 18 | Almacenar cuentas, productos y movimientos | MVP |
| D-03 | Dominio y certificado TLS | Servir la app por HTTPS (PRINCIPIO-004) | MVP |
| D-04 | Plataforma de repositorio y CI con gitleaks | Ejecutar tests, cobertura y detección de secretos antes de cada merge | MVP |
| D-05 | Servicio de envío de correo (SMTP) | Enviar enlaces de recuperación de contraseña (F-11) | Después del MVP |

### Pendientes para el responsable
- Elegir el proveedor de hosting, de dominio/TLS, de repositorio/CI y de correo (requieren ADR según PRINCIPIO-002).

### Preguntas abiertas
- La revisión QA del 2026-11-10 dejó preguntas sin responder, listadas en `06-preguntas-abiertas.md`. Ningún agente implementa los casos afectados hasta que se respondan.

### Decisiones tomadas (2026-11-10)
- Costo unitario promedio ponderado para calcular la ganancia.
- Ajuste por conteo físico incluido en F-05.
- El precio final de una venta puede ser distinto del precio configurado.
- Las ventas y entradas registradas por error se anulan.
- Todo producto tiene al menos una variante; reglas por categoría según las reglas RN-058 a RN-060.
- Toda variante nace con stock 0 y se carga con una entrada.
- Una venta puede incluir varias variantes en líneas.
- Se aprobaron todas las recomendaciones pendientes, reglas RN-087 a RN-104.
- F-10 y F-11 bajan de Should a Could para respetar el máximo de 5 Should.
- Perfumes vencidos no se venden; se retiran con ajuste "Merma por vencimiento"; F-17 agrega el registro de mermas con costo.

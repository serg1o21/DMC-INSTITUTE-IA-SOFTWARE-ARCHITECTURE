# 09 - UI Specification: miStorePe

Basado en `00-constitucion.md`, `02-alcance.md`, `03-reglas-negocio.md`, `06-user-stories-refinadas.md` (HU-001 a HU-018) y `07-architecture.md` v1.0 (C-01, ADR-0005, ADR-0006, ADR-0007, secciones 4, 6.2 y 6.3). Fecha: 2026-11-10; actualizada el 2026-11-11 con las observaciones O-01 a O-14 de `10-validation-log.md`. Estado: **Aprobada por el Responsable del producto el 2026-11-11** (PRINCIPIO-009, PRINCIPIO-012). Ver el historial de decisiones en la sección 6.

Convenciones:
- IDs de pantalla: `P-0NN` (tres dígitos). No confundir con las preguntas abiertas de `05`, que usan dos dígitos (`P-31` a `P-77`); al citarlas se escribe "pregunta P-NN".
- Textos visibles en español es-PE (PRINCIPIO-001, RN-035). Rutas, componentes y claves en inglés.
- Montos `S/ 1,250.50` y `-S/ 4.00`; fechas `10/11/2026`; horas `21:30` (RN-036, RN-148, RN-149). Entrada de montos acepta punto o coma (RN-152). Singular "queda 1 unidad" / plural "quedan 0 unidades" (RN-150).
- Accesibilidad: WCAG 2.2 AA, sin scroll horizontal desde 360 px, axe-core sin violaciones serias o críticas (PRINCIPIO-011). Objetivos táctiles mínimos 24x24 px; 44x44 px en la venta rápida y en toda acción primaria.
- Restricciones técnicas de UI (ADR-0006, 6.3): CSS propio con variables, sin librería de componentes, fuente del sistema, íconos SVG en línea con `aria-label`, listas de 20 elementos, tarjetas apiladas (sin tablas anchas), sin animaciones pesadas.
- Ninguna pantalla tiene campos de datos del comprador (PRINCIPIO-005, W-04).

---

## 1. Inventario de pantallas

| ID | Nombre | Ruta | Actor principal | HUs | MoSCoW |
|---|---|---|---|---|---|
| P-001 | Registro | `/register` | Visitante | HU-001 | Must |
| P-002 | Inicio de sesión | `/login` | Visitante / Emprendedor | HU-002, HU-004 | Must |
| P-003 | Cuenta cerrada (reabrir) | `/account/closed` | Emprendedor (cuenta cerrada) | HU-004 | Must |
| P-004 | Inicio (lista de productos) | `/products` | Emprendedor | HU-001 (RN-153), HU-002, HU-006 | Must |
| P-005 | Crear producto | `/products/new` | Emprendedor | HU-005 | Must |
| P-006 | Detalle y edición de producto | `/products/:id` | Emprendedor | HU-005, HU-006 | Must |
| P-007 | Editar variante | `/variants/:id/edit` | Emprendedor | HU-006 | Must |
| P-008 | Registrar entrada | `/entries/new` | Emprendedor | HU-007 | Must |
| P-009 | Venta rápida | `/sales/new` | Emprendedor (responsable de ventas) | HU-009 | Must |
| P-010 | Historial de ventas | `/sales` | Emprendedor | HU-011 | Must |
| P-011 | Detalle de venta | `/sales/:id` | Emprendedor | HU-010, HU-011 | Must |
| P-012 | Consulta de stock | `/stock` | Emprendedor | HU-012 | Must |
| P-013 | Historial de una variante (movimientos) | `/variants/:id` | Emprendedor | HU-008, HU-012 | Must |
| P-014 | Ajuste por conteo físico | `/variants/:id/adjust` | Emprendedor | HU-012, HU-006 (RN-147) | Must |
| P-015 | Mi cuenta | `/account` | Emprendedor | HU-003, HU-004, HU-001 (RN-141) | Must |
| P-016 | Política de privacidad | `/privacy` | Visitante / Emprendedor | HU-001 | Must |
| P-017 | Administración de cuentas | `/admin/accounts` | Administrador | HU-013 | Should |
| P-018 | Alertas de stock bajo | `/alerts/low-stock` | Emprendedor | HU-014 | Should |
| P-019 | Productos sin movimiento | `/reports/slow-moving` | Emprendedor | HU-015 | Should |
| P-020 | Resumen de ganancias | `/reports/profit` | Emprendedor | HU-016 | Should |
| P-021 | Perfumes por vencer | `/alerts/expiry` | Emprendedor | HU-017 | Could |
| P-022 | Recuperar contraseña | `/password/forgot`, `/password/reset/:token` | Visitante | HU-018 | Could |
| P-023 | Error 404 / sin acceso | `**` | Todos | Transversal T-02 (HU-006 a HU-012) | Must |
| P-024 | Dashboard de inicio | `/home` | Emprendedor | HU-014, HU-015, HU-016 | Should |

Componentes transversales (no son pantallas): diálogo de confirmación, aviso "Sin conexión" (ADR-0005), toast de éxito, barra de navegación inferior, cabecera.

---

## 2. User flows

### UF-01. Alta y primer producto (HU-001, HU-005, HU-007)
1. P-001: completa correo, contraseña, nombre opcional, marca la política (enlace a P-016) -> "Crear cuenta".
2. Sesión iniciada automáticamente (RN-153) -> P-004 vacío con "Aún no tienes productos" -> "Crear producto".
3. P-005: categoría -> nombre -> variantes -> precio -> "Guardar producto".
4. P-006: aviso "Stock 0. Registra una entrada para cargar unidades" -> "Registrar entrada" -> P-008 con la variante preseleccionada.
5. P-008: cantidad y costo unitario -> "Registrar entrada" -> se queda en P-008 para la siguiente (RN-142).

### UF-00. Pantalla de inicio tras iniciar sesión
- MVP (solo Must): P-004 Lista de productos. El dashboard no entra al MVP porque sus datos dependen de F-07, F-08 y F-09 (Should).
- Cuando se implementen F-07, F-08 o F-09: P-024 Dashboard pasa a ser el inicio, y P-004 queda en la pestaña "Productos".
- Administrador: siempre P-017.

### UF-02. Inicio de sesión (HU-002, HU-004)
1. P-002: correo y contraseña -> "Iniciar sesión".
2. Activa -> P-004 (MVP) o P-024 (con Should). Bloqueada -> mensaje "Demasiados intentos. Intenta de nuevo en 15 minutos" (RN-130). Cerrada voluntariamente -> P-003. Suspendida (Should) -> "Tu cuenta está suspendida por falta de pago". Administrador (Should, con F-16 y P-70) -> P-017.
3. Sesión vencida (28,800 s sin actividad) en cualquier pantalla -> P-002 con "Tu sesión se cerró por inactividad".

### UF-03. Venta en un live (HU-009) - flujo crítico, máximo 4 toques (6.3, R-01)
1. Botón central "Vender" (barra inferior) -> P-009 con el foco en la búsqueda.
2. Escribe "polo negro" -> toca el resultado (toque 1). Se agrega línea con cantidad 1 y precio prellenado.
3. Ajusta cantidad con +/- (toque 2, opcional) y precio (toque 3, opcional).
4. "Registrar venta" (toque 4) -> toast "Venta registrada. Quedan 10 unidades" y formulario vacío listo para la siguiente.
5. Alternos: pérdida -> diálogo con suma de pérdidas (RN-073); stock insuficiente -> línea marcada; sin conexión -> aviso y reintento con la misma `Idempotency-Key` (ADR-0005).

### UF-04. Venta olvidada (HU-009, RN-076 a RN-078)
P-009 -> "Fecha de la venta: Ahora" -> "Otro día" -> elige día (hoy a hoy - 7) y hora aproximada -> registra.

### UF-05. Anular venta o línea (HU-010)
P-010 -> toca venta -> P-011 -> "Anular venta" o "Anular" en una línea -> diálogo de confirmación -> estado "Anulada" y stock restaurado. Si pasó el plazo o hay ajuste posterior, el botón no aparece o se muestra el motivo.

### UF-06. Entrada de mercadería y anulación (HU-007, HU-008)
Barra inferior "Más" -> "Registrar entrada" -> P-008 (busca variante) -> registra. Para anular de inmediato: en el éxito de P-008, "Ver en el historial" -> P-013 -> "Anular entrada" -> confirmación (O-03). Más tarde: P-013 de la variante -> entrada "Vigente" -> "Anular entrada" (solo visible si procede, O-02) -> confirmación.

### UF-07. Consulta y ajuste de stock (HU-012)
"Stock" -> P-012 -> busca -> toca variante -> P-013 -> "Ajustar stock" -> P-014 -> cantidad contada y motivo -> "Registrar ajuste" -> vuelve a P-013 con la diferencia en el historial.

### UF-08. Catálogo: editar, desactivar, reactivar (HU-006)
P-004 -> producto -> P-006 -> "Editar" variante (P-007) / "Desactivar producto" (confirmación) / "Reactivar producto". Filtro "Desactivados" en P-004 para encontrarlos.

### UF-09. Cuenta (HU-003, HU-004)
"Más" -> "Mi cuenta" -> P-015: cambiar nombre, cambiar contraseña o "Cerrar mi cuenta" (contraseña) -> P-002 con "Tu cuenta se cerró. Puedes reabrirla hasta el 31/12/2026". Reapertura: P-002 -> P-003 -> "Reabrir cuenta" -> P-004.

### UF-10. Administración (HU-013, Should)
Login Administrador -> P-017 -> busca cuenta -> "Suspender" / "Reactivar" -> confirmación. Sin acceso a ninguna ruta de inventario.

---

## 3. Screen specs

Plantilla común de pantallas autenticadas del emprendedor (layout base `AppShell`):

```
+----------------------------------------------+  360 px min
| [Header]  miStorePe            Hola, Ana     |  56 px, sticky
|           (Saludo: nombre o correo, RN-141)  |
+----------------------------------------------+
| [Banner offline] (solo si no hay conexión)   |  role="status"
+----------------------------------------------+
| <main> contenido de la pantalla              |
|   <h1> título de la pantalla                 |
|   ...                                        |
+----------------------------------------------+
| [Bottom nav] Productos | Stock | VENDER | Ventas | Más |  64 px, fixed
+----------------------------------------------+
```
- Desde 1024 px: la barra inferior pasa a barra lateral izquierda de 240 px; el contenido se centra con ancho máximo 960 px.
- "VENDER" es el botón destacado (fondo primario, 56x56 px).
- "Más" abre una hoja con: Registrar entrada, Mi cuenta, Política de privacidad, Cerrar sesión (y Should: Alertas, Reportes).
- Enlace "Saltar al contenido" como primer elemento enfocable. `aria-current="page"` en la pestaña activa.
- Mensajes de error de campo bajo cada input, vinculados con `aria-describedby`; resumen de errores con `role="alert"` al enviar.

### P-001 Registro
- Propósito: crear la cuenta. Actor: Visitante. HU: HU-001.
- Datos: ninguno previo.
- Acciones: Crear cuenta; ir a Iniciar sesión; ver política: abre P-016 en un diálogo a pantalla completa con "Cerrar", sin salir del formulario ni perder lo escrito (O-05; confirmar en la aprobación).
- Reglas: RN-001 (correo único: "Este correo ya tiene una cuenta"), RN-048 (correo se normaliza; la UI hace `trim` visual pero el servidor decide), RN-133 ("Revisa el correo"), RN-002/RN-134 (contraseña 12 a 128), RN-141 (nombre opcional, máx. 100), RN-003 (aceptación obligatoria), RN-153 (sesión automática).
- Wireframe:
```
<main> (sin AppShell; logo centrado)
  [Logo miStorePe]  <h1>Crea tu cuenta</h1>
  <p>Controla tu inventario desde el celular.</p>
  [Label] Nombre (opcional)        [input text, maxlength 100, autocomplete=name]
  [Label] Correo                    [input email, autocomplete=email, inputmode=email]
     error: "Revisa el correo" | "Este correo ya tiene una cuenta"
  [Label] Contraseña                [input password, autocomplete=new-password] [btn Mostrar 44x44, aria-pressed]
     ayuda: "Mínimo 12 caracteres." contador "0/12" vivo (aria-live=polite)
     error: "La contraseña debe tener entre 12 y 128 caracteres"
  [checkbox] Acepto la [política de privacidad] (abre diálogo con P-016; al cerrarlo, el foco vuelve al enlace y el formulario conserva sus datos)
     error: "Debes aceptar la política de privacidad"
  [Botón primario ancho completo] Crear cuenta
  ¿Ya tienes cuenta? [Inicia sesión]
Estados: inicial | validando (botón deshabilitado + spinner "Creando cuenta...") | error de campo | error de red ("No pudimos conectar. Intenta de nuevo").
```

### P-002 Inicio de sesión
- Propósito: entrar. Actor: Visitante. HU: HU-002, HU-004 (redirige si está cerrada).
- Acciones: Iniciar sesión; ir a Registro; (Could) "¿Olvidaste tu contraseña?" -> P-022. Mientras F-11 no exista: texto "¿Olvidaste tu contraseña? Escribe al correo de contacto" (SUP-07).
- Reglas: RN-004/RN-049 (bloqueo 5 intentos/900 s), RN-130/RN-131 (mensaje de bloqueo), RN-097/RN-132 (inactividad), RN-103, RN-128 (cerrada vencida = credenciales incorrectas), RN-063/RN-104 (Should, suspendida).
- Wireframe:
```
<main> sin AppShell
  [Logo]  <h1>Inicia sesión</h1>
  [Banner info opcional] "Tu sesión se cerró por inactividad" | "Tu cuenta se cerró. Puedes reabrirla hasta el 31/12/2026"
  Correo [input email, autocomplete=username]
  Contraseña [input password, autocomplete=current-password] [Mostrar]
  [Error role=alert] "Correo o contraseña incorrectos" | "Demasiados intentos. Intenta de nuevo en 15 minutos" | "Tu cuenta está suspendida por falta de pago"
  [Botón primario] Iniciar sesión
  ¿No tienes cuenta? [Regístrate]
Estados: inicial | enviando | error | bloqueado (botón activo; el mensaje se repite sin alargar el bloqueo).
```

### P-003 Cuenta cerrada
- Propósito: única opción disponible con cuenta cerrada. HU-004. Reglas: RN-007, RN-090, RN-104 (cierre por falta de pago: "Contacta al administrador", sin botón).
- Wireframe:
```
<main> sin bottom nav
  <h1>Tu cuenta está cerrada</h1>
  <p>Tus datos se borrarán definitivamente el 31/12/2026 a las 10:00.</p>
  [Botón primario] Reabrir cuenta  -> diálogo "¿Reabrir tu cuenta? Recuperarás todo tu inventario." [Cancelar] [Reabrir]
  [Botón secundario] Cerrar sesión
Variante falta de pago: <p>Contacta al administrador.</p> sin "Reabrir cuenta".
```

### P-004 Inicio: lista de productos
- Propósito: catálogo y punto de partida. HU-001 (destino tras registro), HU-002, HU-006 (activar/desactivar, filtro).
- Datos (`GET /api/products`): nombre, categoría, número de variantes, stock total, estado activo.
- Acciones: Crear producto; abrir producto; filtrar por categoría; pestañas Activos / Desactivados; buscar por nombre; paginación "Ver más" (20).
- Reglas: RN-016, RN-100 (desactivados visibles para reactivar), RN-150.
- Wireframe:
```
<h1>Mis productos</h1>                         [Botón primario "+ Crear producto" 44 px]
[Search input "Buscar producto" con label visible-oculto]
[Chips filtro, role=radiogroup] Todos | Ropa | Perfume | Accesorio
[Tabs] Activos (32) | Desactivados (2)
<ul> tarjetas apiladas:
  +------------------------------------------+
  | Polo oversize básico          [Ropa]     |
  | 4 variantes · quedan 11 unidades         |
  | [>] (toda la tarjeta es enlace, 64 px)   |
  +------------------------------------------+
  (desactivado: badge gris "Desactivado")
[Botón secundario] Ver más
Estados: cargando (3 tarjetas skeleton, aria-busy) | vacío ("Aún no tienes productos" + "Crear mi primer producto") | error ("No pudimos cargar tus productos" + Reintentar).
```

### P-005 Crear producto
- Propósito: alta con variantes según categoría. HU-005.
- Acciones: elegir categoría, nombre, agregar/quitar variantes, guardar, cancelar.
- Reglas: RN-008 (categoría obligatoria; no editable luego, RN-017), RN-093 (nombre 1-100), RN-058/RN-059/RN-060 (atributos por categoría), RN-111/RN-121 (normalización talla/color), RN-122 (talla ≤10), RN-123 (color ≤30), RN-011/RN-146 (1-9,999 ml enteros), RN-012 (vencimiento AAAA-MM-DD; usar `input type=date`), RN-013/RN-091 (precio 0.01-99,999.99, 2 decimales), RN-050/RN-135 (duplicados), RN-061/RN-024 (sin campo de stock), RN-064 ("Única" automática).
- Wireframe:
```
<h1>Crear producto</h1>
Paso 1  <fieldset><legend>Categoría</legend>
         (o) Ropa  (o) Perfume  (o) Accesorio   [radios grandes como tarjetas 56 px]
Paso 2  Nombre del producto [input, maxlength 100, contador "0/100"]
Paso 3  <h2>Variantes</h2>
  Tarjeta de variante n (repetible):
   Ropa:      Talla* [input, maxlength 10, autocapitalize=characters] Color* [input, maxlength 30]
   Perfume:   Presentación (ml)* [input inputmode=numeric] Vencimiento (opcional) [date]
   Accesorio: Talla (opcional) Color (opcional)
   Precio de venta* [prefijo "S/" + input inputmode=decimal]
   [Botón icono "Quitar variante" aria-label="Quitar variante 2"]
  [Botón secundario] + Agregar otra variante
  Ayuda accesorio: "Si no tiene tallas ni colores, se creará la variante Única."
  Nota fija: "El stock empieza en 0. Lo cargas registrando una entrada."
Errores: "Esta variante ya existe en el producto", "Ingresa la talla", "Ingresa un precio entre S/ 0.01 y S/ 99,999.99"
[Barra inferior fija de acciones] [Cancelar] [Guardar producto (primario)]
Estados: sin categoría (pasos 2-3 deshabilitados con texto "Elige una categoría") | guardando | error 409 en la variante afectada.
Tras guardar -> P-006 con toast "Producto creado. SKU asignados."
```

### P-006 Detalle de producto
- Propósito: ver variantes, editar, desactivar/reactivar, agregar o eliminar variantes. HU-005, HU-006.
- Datos: nombre, categoría, estado; por variante: SKU, etiqueta (talla/color o ml + vencimiento), precio, stock, costo promedio ("sin entradas" si no tiene entradas vigentes, RN-154), badge "Vencido".
- Acciones: Editar nombre; Agregar variante; Editar variante (P-007); Eliminar variante (solo sin movimientos, RN-099; si es la única, RN-136 ofrece desactivar); Registrar entrada (P-008 preseleccionado); Ver historial (P-013); Desactivar / Reactivar producto.
- Reglas: RN-016, RN-017 (categoría solo lectura), RN-098, RN-099, RN-136, RN-137, RN-140, RN-015.
- Wireframe:
```
[< Volver] (a la pantalla de origen: Mis productos o Stock, O-06)
<h1>Polo oversize básico</h1>  [badge Ropa] [badge Activo|Desactivado]
[Botón secundario "Editar nombre"]
[Banner info si desactivado] "Producto desactivado. No aparece en ventas ni entradas." [Reactivar producto]
[Banner info si todas las variantes tienen stock 0 y ninguna entrada] "Stock 0. Registra una entrada para cargar unidades." [Registrar entrada] (O-08, RN-061)
<h2>Variantes (4)</h2>
  +------------------------------------------+
  | M / Negro                 ROP-00001      |
  | S/ 35.90 · quedan 11 unidades            |
  | Costo prom. S/ 19.00 (o "sin entradas")  |
  | [Entrada] [Historial] [Editar] [⋯ Eliminar] |
  +------------------------------------------+
  Perfume: "100 ml · vence 30/06/2028" ; badge rojo "Vencido" si fecha < hoy.
[+ Agregar variante]
<hr>
[Botón peligro-secundario] Desactivar producto -> diálogo "¿Desactivar? Conserva su stock e historial."
Eliminar con movimientos -> diálogo "Esta variante tiene movimientos y no se puede eliminar. ¿Quieres desactivar el producto?"
```

### P-007 Editar variante
- HU-006. Reglas: RN-098 (atributos editables solo sin movimientos; con movimientos se muestran como solo lectura con texto "No se puede cambiar porque tiene movimientos"), RN-050, RN-115 (vencimiento editable siempre; se audita), RN-015 (advertencia no bloqueante), RN-041/RN-091.
- Wireframe:
```
<h1>Editar variante ROP-00001</h1>
Talla [M] (solo lectura si hay movimientos) · Color [Negro]
Accesorio con variante única: "Variante: Única" (sin campos de talla ni color, O-13)
Precio de venta [S/ 32.00]
  [Aviso amarillo role=status] "El precio de venta es menor que el costo" (si precio < costo promedio)
Perfume: Vencimiento [date] + ayuda "El cambio queda registrado."
[Cancelar] [Guardar cambios]
```

### P-008 Registrar entrada
- HU-007 (anulación desde P-013, HU-008).
- Datos: variante elegida (nombre, etiqueta, stock actual, costo promedio actual).
- Acciones: buscar/elegir variante; cantidad; costo unitario; Registrar entrada; "Registrar otra" sin salir.
- Reglas: RN-018/RN-092 (1-9,999 enteros), RN-014/RN-091 (costo 0.01-99,999.99), RN-152, RN-140 ("Reactiva el producto para registrar mercadería"), RN-142 (sin campo de fecha), RN-020 (mostrar nuevo costo promedio tras guardar).
- Wireframe:
```
<h1>Registrar entrada</h1>
Variante* [combobox con búsqueda, role=combobox, aria-expanded] "Busca por nombre, SKU, talla o color"
  Seleccionada: "Polo oversize básico · M / Negro · ROP-00001 · quedan 10 unidades · costo prom. S/ 18.00" (o "costo prom.: sin entradas")
Cantidad* [stepper - [ 12 ] +, inputmode=numeric]
Costo unitario* [S/ | 18.50]
Total: S/ 222.00 (calculado, aria-live)
[Botón primario] Registrar entrada
Éxito: toast "Entrada registrada. Stock: 22. Costo promedio: S/ 18.27" con enlace "Ver en el historial" (P-013 de la variante, O-03); el formulario se limpia (variante se conserva para repetir).
[< Volver] a la pantalla de origen (producto o stock), O-06.
Errores: producto desactivado (banner + enlace "Ir al producto"); fuera de rango; sin conexión (reintento).
```

### P-009 Venta rápida (crítica)
- Propósito: registrar ventas durante un live en ≤4 toques. HU-009.
- Datos (`GET /api/variants?q=`): etiqueta, SKU, precio de venta, stock, costo promedio (para calcular pérdida en cliente solo como aviso previo), vencida.
- Acciones: buscar variante; agregar línea (fusiona si mismo precio, RN-084); +/- cantidad; editar precio final; quitar línea; cambiar fecha ("Ahora" / "Otro día"); Registrar venta; confirmar pérdida.
- Reglas: RN-016 (desactivados no aparecen), RN-023/RN-150 (stock), RN-025/RN-091 (precio), RN-092/RN-151 (máx. 9,999 por línea), RN-074 (vencidos: aparecen deshabilitados con "Vencido"), RN-073 (una confirmación con la suma), RN-076 a RN-078/RN-126 (fecha), RN-080 (sin líneas: botón deshabilitado), RN-081 (todo o nada), RN-125 (misma variante con otro precio), RN-152, PRINCIPIO-005 (sin datos de comprador), ADR-0005 (idempotencia y borrador en memoria).
- Wireframe:
```
<h1>Nueva venta</h1>
[Search grande 48 px, autofocus] "Buscar: nombre, SKU, talla o color"
  Resultados (máx. 8, lista role=listbox), cada fila 56 px:
   | Polo oversize básico · M / Negro   S/ 35.90 |
   | ROP-00001 · quedan 11 unidades       [+]    |
   Vencido: fila atenuada, texto "Vencido" y aria-disabled.
   Sin stock: "quedan 0 unidades" en color de error, seleccionable (el servidor valida).
<h2>Líneas (2)</h2>
  +------------------------------------------+
  | Polo oversize básico M / Negro           |
  | [-] [ 1 ] [+]   Precio S/ [35.90]        |
  | Subtotal S/ 35.90         [Quitar línea] |
  | (error) "Stock insuficiente: queda 1 unidad" |
  +------------------------------------------+
Fecha de la venta: [Ahora ▾] -> opción "Otro día": [date min=hoy-7 max=hoy] [time], ambos precargados con la fecha y hora actuales (O-09)
  error: "Elige una fecha de los últimos 7 días, no futura"
[Barra fija inferior encima del bottom nav]
  Total S/ 95.00                [Registrar venta] (primario, 48 px alto, deshabilitado sin líneas)
Diálogo pérdida (role=alertdialog): "Esta venta genera una pérdida de S/ 5.50" [Volver] [Registrar igual]
Mensaje fusión con otro precio: "Esta variante ya está en la venta con otro precio. Edita esa línea"
Éxito: toast "Venta registrada" + lista "Quedan 10 unidades de M / Negro"; formulario vacío y foco en búsqueda.
Offline: banner "Sin conexión. Tu venta no se perdió: reintentaremos en cuanto vuelva la señal." + [Reintentar] tras 3 intentos.
Estados: vacío | con líneas | enviando (botón "Registrando...") | error por línea | confirmación pérdida | éxito | offline.
Sin campos de comprador (nombre, teléfono, dirección, DNI).
```

### P-010 Historial de ventas
- HU-011. Datos (`GET /api/sales`): fecha y hora de venta, número de líneas, total, ganancia, estado. Orden `sale_at` descendente.
- Acciones: filtrar por rango de fechas y estado (Todas/Vigentes/Anuladas); abrir venta; Ver más.
- Reglas: RN-054 (anuladas visibles, no suman), RN-148, RN-149, RN-036.
- Wireframe:
```
<h1>Historial de ventas</h1>
[Desde date] [Hasta date]  [Estado select: Todas | Vigentes | Anuladas]
<ul>
  | 10/11/2026 21:30          [Vigente]     |
  | 2 productos · Total S/ 95.00            |
  | Ganancia S/ 33.00 (verde) / -S/ 4.00 (rojo + signo, no solo color) |
  Anulada: badge gris "Anulada", importes tachados con texto "No suma".
[Ver más]
Vacío: "Aún no registras ventas" + [Registrar venta].
```

### P-011 Detalle de venta
- HU-011, HU-010. Datos: fecha y hora, registrada el, estado, líneas (etiqueta, cantidad, precio final, costo usado, ganancia, estado), total, ganancia total.
- Acciones: Anular venta; Anular línea (solo dentro de 86,400 s desde el registro y sin ajuste posterior).
- Reglas: RN-027, RN-028, RN-053, RN-082, RN-083, RN-086, RN-087, RN-109, RN-139, RN-147, RN-057.
- Wireframe:
```
[< Volver al historial]
<h1>Venta del 10/11/2026 21:30</h1>  [Vigente|Anulada]
Registrada el 10/11/2026 21:31 · Puedes anular hasta el 11/11/2026 21:31
<ul> líneas:
  | Polo oversize básico M / Negro            [Vigente] |
  | 1 x S/ 35.00 · costo S/ 19.00                       |
  | Ganancia S/ 16.00                    [Anular línea] |
Resumen: Total S/ 95.00 · Ganancia S/ 33.00
Venta anulada: mismo criterio que P-010: badge "Anulada" e importes originales tachados con el texto "No suma" (`voidedAmount`, `voidedProfit`; nunca "S/ 0.00"), O-10, RN-054
[Botón peligro] Anular venta
Diálogo: "¿Anular esta venta? Las unidades vuelven al stock." [Cancelar] [Anular]
Bloqueos (texto en lugar del botón): "Ya pasó el plazo de 24 horas para anular" | "No se puede anular: hay un ajuste de stock posterior a esta venta".
404: P-023.
```

### P-012 Consulta de stock
- HU-012. Datos (`GET /api/variants?q=`): producto, etiqueta, SKU, stock, vencido.
- Acciones: buscar (RN-138); filtrar por categoría; abrir variante (P-013).
- Wireframe:
```
<h1>Stock</h1>
[Search "Buscar: nombre, SKU, talla o color"]
[Chips] Todos | Ropa | Perfume | Accesorio
<ul> filas 56 px:
  | Polo oversize básico · M / Negro      11 |
  | ROP-00001                       unidades |
  stock 0: número en color de error + texto "Sin stock".
Vacío: "No tienes productos todavía" | búsqueda sin resultados: "No encontramos 'xyz'".
```

### P-013 Historial de la variante
- HU-012, HU-008. Datos (`GET /api/variants/{id}/movements`): tipo (Entrada, Venta, Ajuste), fecha y hora, cantidad/diferencia (+/-), costo, motivo, estado.
- Acciones: Ajustar stock (P-014); Registrar entrada; Anular entrada (solo si vigente, ≤24 h y sin movimientos posteriores, RN-056/RN-087/RN-088). Ajustes sin opción de anular (RN-145).
- Costo promedio: si la variante no tiene entradas vigentes se muestra "Costo promedio: sin entradas", nunca "S/ 0.00" (RN-062, RN-154). Aplica también a P-006, P-007 y P-008.
- Wireframe:
```
[< Volver] (a la pantalla de origen: Stock o Detalle de producto, O-06)
<h1>Polo oversize básico · M / Negro</h1>  ROP-00001
Stock actual: quedan 11 unidades · Costo promedio S/ 19.00
[Ajustar stock] [Registrar entrada]
<h2>Historial</h2>
  | Entrada · 10/11/2026 20:00    +12  [Vigente]   |
  | Costo S/ 18.50               [Anular entrada]  |   (botón solo si `voidable`; O-02)
  | Entrada · 01/11/2026 09:00    +5   [Vigente]   |
  | Costo S/ 17.00   "Ya pasó el plazo de 24 horas para anular" (texto en lugar del botón) |
  | Venta · 10/11/2026 21:30      -2               |
  | Ajuste · 11/11/2026 09:00     -1  Motivo: "Conteo físico..." |
Motivos en lugar del botón (igual que P-011, O-02): "Ya pasó el plazo de 24 horas para anular" | "No se puede anular: hay movimientos posteriores a esta entrada".
Si el servidor rechaza igual (otra pestaña registró un movimiento), se muestra ese mismo texto y se recarga el historial.
Diálogo de anulación (role=alertdialog):
  con costo anterior: "¿Anular esta entrada? Se restan 12 unidades del stock y el costo promedio vuelve a S/ 18.00." (RN-088)
  sin costo anterior: "¿Anular esta entrada? Se restan 12 unidades del stock. La variante quedará sin entradas." (RN-154, aprobada el 2026-11-11, P-64)
  [Cancelar] [Anular entrada]
Tras anular la única entrada: "Stock actual: quedan 0 unidades · Costo promedio: sin entradas".
```

### P-014 Ajuste por conteo físico
- HU-012, HU-006 (permitido en desactivados, RN-147).
- Reglas: RN-037 (diferencia), RN-040/RN-092 (0-9,999), RN-116 ("Registra primero una entrada con su costo"), RN-143 (motivo 1-200, sin solo espacios), RN-144 (diferencia 0 permitida), RN-075 (perfume: motivo rápido "Merma por vencimiento"), RN-024, PRINCIPIO-005 (aviso de no escribir datos de clientes).
- Wireframe:
```
<h1>Ajustar stock</h1>
Polo oversize básico · M / Negro · Stock en la app: 8
Cantidad contada* [stepper, inputmode=numeric]
Diferencia: -2 (aria-live, calculada)
Motivo* [textarea, contador "0/200" sobre el texto sin espacios al inicio y al final; error "Escribe un motivo de 1 a 200 caracteres" (O-11, RN-143)]
  Perfume: [chip] Merma por vencimiento (rellena el motivo)
  Ayuda: "No escribas datos de tus clientes."
[Cancelar] [Registrar ajuste]
Nota: "Un ajuste no se anula; si te equivocas, registra otro ajuste."
```

### P-015 Mi cuenta
- HU-003, HU-004, HU-001 (RN-141).
- Secciones: Nombre (editable, ≤100); Correo (solo lectura, W-12); Cambiar contraseña (actual, nueva 12-128, RN-095/RN-096/RN-105/RN-134/RN-156; aviso "Se cerrarán tus otras sesiones"); Cerrar mi cuenta (zona de peligro; diálogo con contraseña, RN-127, texto "Tus datos se borrarán en 30 días. Puedes reabrirla antes"); Derechos sobre tus datos (correo de contacto, P-49); Cerrar sesión.
- Wireframe:
```
<h1>Mi cuenta</h1>
<section><h2>Tus datos</h2> Nombre [input] [Guardar] · Correo ana@gmail.com (no editable)</section>
<section><h2>Cambiar contraseña</h2> Actual [pw] Nueva [pw] [Cambiar contraseña]
   errores: "Ingresa tu contraseña actual" | "La contraseña actual no es correcta" | "La contraseña debe tener entre 12 y 128 caracteres" | "La nueva contraseña debe ser distinta de la actual" (RN-156)</section>
<section><h2>Privacidad</h2> [Ver política] · "Para ejercer tus derechos escribe a <correo de contacto>"</section>
<section class=danger><h2>Cerrar mi cuenta</h2> [Botón peligro] Cerrar mi cuenta
   Diálogo: Contraseña [pw] [Cancelar] [Cerrar cuenta]</section>
[Cerrar sesión]
```

### P-016 Política de privacidad
- HU-001 (RN-003, Ley N° 29733). Texto estático largo con encabezados, correo de contacto ARCO y botón "Volver". Contenido legal: pendiente de redacción y validación legal (P-49); correo: `sergioc9916@gmail.com` (variable de entorno `ARCO_CONTACT_EMAIL`, P-67 resuelto).
- Se muestra como página (`/privacy`, desde "Mi cuenta" o "Más") y como diálogo desde P-001 (O-05).
- Wireframe:
```
<main> sin AppShell si no hay sesión
  [< Volver] (página) | [Cerrar] (diálogo, foco inicial en el título)
  <h1>Política de privacidad</h1>
  <p>Última actualización: DD/MM/AAAA</p>
  <h2>Qué datos guardamos</h2>        (nombre opcional, correo, fecha de aceptación; nada de tus clientes)
  <h2>Para qué los usamos</h2>
  <h2>Cuánto tiempo los guardamos</h2> (borrado 30 días después del cierre de la cuenta)
  <h2>Tus derechos</h2>               (acceso, rectificación, cancelación y oposición: escribe a sergioc9916@gmail.com)
  <h2>Seguridad</h2>
Texto con ancho máximo de 70 caracteres por línea; zoom al 200% sin pérdida.
```

### P-017 Administración de cuentas (Should)
- HU-013. Actor: Administrador. Datos: solo nombre, correo y estado (RN-114, PRINCIPIO-003). Sin bottom nav de inventario.
- Acciones: buscar por correo; Suspender (Activa); Reactivar (Suspendida); cerrar sesión.
- Wireframe:
```
<h1>Cuentas</h1>  [Cerrar sesión]
[Search por correo] [Filtro estado: Todas|Activas|Suspendidas|Cerradas]
  | Ana · ana@gmail.com        [Activa]   [Suspender] |
  | Luis · luis@gmail.com      [Suspendida] [Reactivar] |
Diálogo: "¿Suspender la cuenta de ana@gmail.com? No podrá iniciar sesión."
Nunca se muestran productos, ventas ni stock.
```

### P-018 Alertas de stock bajo (Should)
- HU-014 (RN-029, RN-101, RN-102). Lista de variantes con stock ≤ mínimo (mínimo > 0), excluye desactivados (P-41). Campo "Stock mínimo" se agrega en P-007 (0-9,999, defecto 0). Badge con conteo en "Más" > Alertas.
```
<h1>Stock bajo</h1>
  | Polo M / Negro   "Stock bajo: quedan 3 unidades"  [Registrar entrada] |
Vacío: "Ninguna variante está por debajo de su mínimo".
```

### P-019 Productos sin movimiento (Should)
- HU-015 (RN-030 a reformular, P-53). Lista de productos activos sin ventas vigentes en 30 días con "Última venta: 05/10/2026" o "Sin ventas".

### P-020 Resumen de ganancias (Should)
- HU-016 (RN-031, RN-044, RN-051, RN-053). Selector Semana (lunes a domingo) / Mes, navegación anterior/siguiente; tarjetas Ventas, Costo, Ganancia. Sin gráficos (F-13 es Could).
```
<h1>Ganancias</h1> [Semana | Mes]  [<] 09/11/2026 - 15/11/2026 [>]
[Ventas S/ 1,250.50] [Costo S/ 800.00] [Ganancia S/ 450.50]
Nota: "No incluye ventas anuladas ni ajustes."
```

### P-024 Dashboard de inicio (Should)
- Propósito: resumen del negocio al entrar. Actor: Emprendedor. HU: HU-014 (F-07), HU-015 (F-08), HU-016 (F-09).
- Datos: ganancia de la semana y del mes (vigentes, RN-031, RN-053); número de variantes con stock bajo; número de productos sin movimiento; últimas 3 ventas.
- Acciones: Vender (atajo); ver cada bloque en su pantalla (P-020, P-018, P-019, P-010).
- Reglas: RN-029, RN-030, RN-031, RN-044, RN-053, RN-148, RN-149. Sin gráficos (F-13 es Could); cada bloque se oculta si su funcionalidad aún no está implementada.
```
<h1>Hola, Ana</h1>
[Botón Filled ancho completo] Registrar venta
<section> <h2>Ganancia</h2>
  [Tarjeta] Esta semana S/ 450.50   [Tarjeta] Este mes S/ 1,250.50   [Text: Ver resumen]
<section> <h2>Para revisar</h2>
  | 3 variantes con stock bajo        [>] P-018 |
  | 5 productos sin ventas en 30 días [>] P-019 |
<section> <h2>Últimas ventas</h2>  3 filas como en P-010  [Text: Ver historial]
Estados: cargando (skeleton, aria-busy) | sin datos ("Registra tu primera venta para ver tus ganancias") | error con Reintentar.
```

### P-021 y P-022 (Could)
Solo inventariadas; spec detallada en un lote posterior.

### P-023 Error / no encontrado
- T-02. Mensaje "No encontramos lo que buscas" y botón "Ir a mis productos". No revela si el recurso existe para otro emprendedor. Se muestra en rutas desconocidas y cuando la API responde 404.
- Wireframe:
```
<main> dentro del AppShell si hay sesión; sin él si no la hay
  <h1>No encontramos lo que buscas</h1>
  <p>Puede que el enlace esté mal escrito o que ya no exista.</p>
  [Botón primario] Ir a mis productos   (sin sesión: [Iniciar sesión])
```

---

## 4. Design tokens

Estándar adoptado: **Material Design 3 (M3)** como referencia de escala tipográfica, tipos de botón y tamaños, implementado con CSS propio (sin Angular Material, ADR-0006). Las alturas suben a 48 px para cumplir el objetivo táctil de 44 px (PRINCIPIO-011).

### 4.1 Títulos y texto (escala tipográfica M3)

| Rol M3 | Uso en miStorePe | Tamaño / interlineado | Peso |
|---|---|---|---|
| Headline Small | `<h1>` de cada pantalla (móvil) | 24 / 32 px | 400 |
| Headline Medium | `<h1>` desde 1024 px; cifras del dashboard | 28 / 36 px | 400 |
| Title Large | `<h2>` de sección | 22 / 28 px | 400 |
| Title Medium | Nombre en tarjetas, `<h3>` | 16 / 24 px | 500 |
| Body Large | Texto e inputs | 16 / 24 px | 400 |
| Body Medium | Texto secundario, metadatos | 14 / 20 px | 400 |
| Label Large | Texto de botones y pestañas | 14 / 20 px | 500 |
| Label Medium | Badges, ayudas de campo | 12 / 16 px | 500 |

Un solo `<h1>` por pantalla; los niveles no se saltan (h1 > h2 > h3).

### 4.2 Botones (tipos M3)

| Tipo M3 | Uso | Estilo | Altura |
|---|---|---|---|
| Filled | Acción principal, una por pantalla (Guardar, Registrar venta) | Fondo primario, texto blanco | 48 px |
| Filled (acento) | Solo "Vender" en la navegación | Fondo acento | 56 px |
| Filled Tonal | Secundaria destacada (Registrar entrada, Ajustar stock) | Fondo primary-container | 48 px |
| Outlined | Alternativas (Cancelar, Ver más) | Borde 1 px, fondo transparente | 48 px |
| Text | Baja jerarquía (Volver, Ver historial) | Solo texto primario | 44 px |
| Filled (error) | Destructivas confirmadas (Anular, Cerrar cuenta) | Fondo error, texto blanco | 48 px |
| Icon button | Quitar, +/-, Mostrar contraseña | Ícono 24 px con `aria-label` | 44x44 px |

Común: radio 20 px (forma pill de M3), padding horizontal 24 px, texto Label Large con mayúscula inicial (no todo en mayúsculas). Estados: hover (+8% capa), foco con anillo visible, presionado (+12%), deshabilitado al 38% con `aria-disabled`. En barras de acciones: secundaria a la izquierda, primaria a la derecha; en 360 px se apilan con la primaria arriba.

En los wireframes de la sección 3, "Botón primario" = Filled, "Botón secundario" = Outlined y "Botón peligro" = Filled (error).

### 4.3 Paleta y demás tokens

Paleta sugerida (sin marca definida; todas las combinaciones de texto verificadas para contraste ≥4.5:1 sobre su fondo, a validar con axe-core).

```css
:root {
  /* Color */
  --color-primary: #0F5C8C;        /* azul petróleo: botones primarios, enlaces. Blanco sobre él: 7.0:1 */
  --color-primary-hover: #0B4A72;
  --color-on-primary: #FFFFFF;
  --color-secondary: #3F4A56;      /* gris pizarra: botones secundarios, íconos */
  --color-accent: #C2410C;         /* naranja quemado: botón VENDER y destacados. Blanco: 5.2:1 */
  --color-bg: #F7F8FA;
  --color-surface: #FFFFFF;
  --color-border: #C9CFD6;         /* bordes de inputs con contraste no textual ≥3:1: usar #8A94A0 en inputs */
  --color-border-input: #8A94A0;
  --color-text: #1A1F24;           /* 16:1 sobre bg */
  --color-text-muted: #4E5965;     /* 7:1 */
  --color-success: #15803D;        /* ganancia positiva, texto sobre blanco 5.0:1 */
  --color-error: #B91C1C;          /* errores, pérdidas, stock 0 */
  --color-warning-bg: #FEF3C7;  --color-warning-text: #78350F;
  --color-info-bg: #E0F2FE;     --color-info-text: #0C4A6E;
  --color-disabled: #6B7280;
  --focus-ring: 0 0 0 3px #FFFFFF, 0 0 0 5px #0F5C8C;  /* visible en cualquier fondo (WCAG 2.4.7/2.4.11) */

  /* Tipografía: fuente del sistema (ADR-0006, sin fuentes web) */
  --font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Noto Sans", sans-serif;
  --font-mono: ui-monospace, "Roboto Mono", monospace;  /* SKU */
  /* Escala M3 (sección 4.1) */
  --fs-label-md: 0.75rem;   /* 12 px */
  --fs-body-md: 0.875rem;   /* 14 px, también label-lg */
  --fs-body-lg: 1rem;       /* 16 px, inputs (evita zoom en móvil) y title-md */
  --fs-title-lg: 1.375rem;  /* 22 px, h2 */
  --fs-headline-sm: 1.5rem; /* 24 px, h1 móvil */
  --fs-headline-md: 1.75rem;/* 28 px, h1 desktop */
  --fw-regular: 400; --fw-medium: 500; --fw-bold: 700;
  --lh: 1.5;
  --num: tabular-nums; /* montos y stock alineados */

  /* Espaciado (base 4) */
  --sp-1: 4px; --sp-2: 8px; --sp-3: 12px; --sp-4: 16px; --sp-6: 24px; --sp-8: 32px;

  /* Componentes */
  --touch-min: 44px;          /* acciones primarias y venta rápida */
  --touch-min-secondary: 24px;/* WCAG 2.5.8 mínimo */
  --color-primary-container: #D3E8F5; --color-on-primary-container: #08324D; /* Filled Tonal */
  --btn-height: 48px;  --btn-radius: 20px;  --btn-padding-x: 24px;
  --input-height: 48px; --input-radius: 8px; --input-border: 1px solid var(--color-border-input);
  --card-radius: 12px; --card-padding: 16px; --card-shadow: 0 1px 2px rgba(0,0,0,.08);
  --header-height: 56px; --bottom-nav-height: 64px; --sell-fab-size: 56px;
  --sidebar-width: 240px; --content-max: 960px;
  --badge-height: 24px; --badge-radius: 12px;
  --dialog-max: 420px;

  /* Breakpoints (referencia para media queries) */
  /* móvil 360-599 | tablet 600-1023 | desktop ≥1024 */
}
@media (prefers-reduced-motion: reduce) { * { transition: none !important; } }
```

Reglas de uso: el estado nunca se comunica solo con color (badges con texto, ganancia negativa con signo "-"); todos los íconos con `aria-label` o `aria-hidden` si van con texto; zoom al 200% sin pérdida de contenido.

---

## 5. Navigation map

```
miStorePe
├── Públicas (sin sesión)
│   ├── /login ............................ P-002
│   │     ├─> /register ................... P-001 ──> /privacy (P-016)
│   │     ├─> /password/forgot ............ P-022 (Could)
│   │     └─> /account/closed ............. P-003 (cuenta cerrada) ──> /products
│   └── /privacy .......................... P-016
│
├── Emprendedor (AppShell, bottom nav)
│   ├── Inicio  /home ..................... P-024 (Should; inicio cuando exista)
│   ├── Productos  /products .............. P-004 (inicio en el MVP)
│   │     ├── /products/new ............... P-005
│   │     └── /products/:id ............... P-006
│   │           ├── /variants/:id/edit .... P-007
│   │           ├── /variants/:id ......... P-013
│   │           └── /entries/new?variant= . P-008
│   ├── Stock  /stock ..................... P-012
│   │     └── /variants/:id ............... P-013
│   │           ├── /variants/:id/adjust .. P-014
│   │           └── /entries/new?variant= . P-008
│   ├── VENDER  /sales/new ................ P-009
│   ├── Ventas  /sales .................... P-010
│   │     └── /sales/:id .................. P-011
│   └── Más (hoja)
│         ├── Registrar entrada /entries/new ... P-008
│         ├── Alertas
│         │     ├── /alerts/low-stock ...... P-018 (Should)
│         │     └── /alerts/expiry ......... P-021 (Could)
│         ├── Reportes
│         │     ├── /reports/profit ........ P-020 (Should)
│         │     └── /reports/slow-moving ... P-019 (Should)
│         ├── Mi cuenta /account ........... P-015 ──> /privacy
│         └── Cerrar sesión ──> /login
│
├── Administrador (sin bottom nav de inventario)
│   └── /admin/accounts ................... P-017 (Should)
│
└── ** (cualquier otra) ................... P-023
```

Guards: `authGuard` (sin sesión -> `/login`), `openAccountGuard` (cuenta cerrada -> `/account/closed`), `entrepreneurGuard` (Administrador no entra a inventario), `adminGuard` (`/admin/**`). Rutas con lazy loading por módulo (ADR-0006).

---

## 6. Supuestos, preguntas y lista de aprobación

Supuestos de esta spec (confirmados):
1. Estándar visual Material Design 3 como referencia, sin la librería. La paleta sigue siendo sugerida (no hay marca).
2. Inicio = P-004 en el MVP; P-024 Dashboard cuando se implementen F-07 a F-09.
3. El Administrador entra directo a P-017 sin navegación de inventario (Should, con F-16 y P-70).
4. En P-009 las variantes sin stock son seleccionables y las vencidas, deshabilitadas.

Preguntas: se consolidaron en `05-preguntas-abiertas.md`. UI-Q1 pasó a P-67 (correo ARCO), UI-Q2 a P-71 (stock mínimo) y UI-Q3 está cerrada (Angular CDK aprobada en `07`, sección 6.5).

### Lista de aprobación (Sergio)

Todos los puntos fueron aprobados por Sergio el 2026-11-11 (ver el historial de decisiones). La columna "Estado" refleja el resultado.

| # | Punto | Decisión propuesta | Estado |
|---|---|---|---|
| 1 | O-05: la política se abre en un diálogo desde P-001 | Aprobar diálogo (alternativa: guardar el borrador del formulario) | Aprobado 2026-11-11 |
| 2 | O-02: "Anular entrada" solo visible si procede, con motivo en texto | Aprobar (P-66 resuelta) | Aprobado 2026-11-11 |
| 3 | O-03: enlace "Ver en el historial" tras registrar una entrada | Aprobar | Aprobado 2026-11-11 |
| 4 | O-06: "Volver" a la pantalla de origen en P-006, P-008 y P-013 | Aprobar | Aprobado 2026-11-11 |
| 5 | O-08: banner de stock 0 en P-006 | Aprobar | Aprobado 2026-11-11 |
| 6 | O-09: "Otro día" precargado con fecha y hora actuales | Aprobar | Aprobado 2026-11-11 |
| 7 | O-10: venta anulada con importes tachados en P-010 y P-011 | Aprobar (P-66 resuelta) | Aprobado 2026-11-11 |
| 8 | O-11: motivo contado sin espacios al inicio y al final | Aprobar | Aprobado 2026-11-11 |
| 9 | O-13: "Variante: Única" en P-007 | Aprobar | Aprobado 2026-11-11 |
| 10 | Diálogo de anular la única entrada ("quedará sin entradas") | Opción A: el diálogo dice "La variante quedará sin entradas" | Aprobado (P-64 resuelta 2026-11-11) |
| 11 | Correo ARCO en P-015 y P-016 | sergioc9916@gmail.com desde `ARCO_CONTACT_EMAIL` | Aprobado (P-67 resuelta 2026-11-11) |
| 12 | Paleta sugerida | Aprobar como provisional | Aprobado como provisional |

### Historial de decisiones

| Fecha | Decisión |
|---|---|
| 2026-11-11 | Spec aprobada por Sergio con los 12 puntos de la lista (puntos 10 y 11 no bloquean la aprobación; se resuelven con P-64 y P-67 respectivamente). |
| 2026-11-11 | P-64 aprobada (opción A): diálogo de P-013 sin costo anterior dice "La variante quedará sin entradas". |
| 2026-11-11 | P-66 aprobada: `voidable` y campos de respuesta incluidos en las pantallas que los usan. |
| 2026-11-11 | P-67 resuelto: correo ARCO `sergioc9916@gmail.com` en P-015 y P-016. |
| 2026-11-11 | P-61 resuelto: job diario `@Scheduled` para el borrado definitivo. |
| 2026-11-12 | Spec Finalizer: lista de aprobación sincronizada con el historial y convención de IDs actualizada (preguntas hasta P-77). |
| 2026-11-12 | P-77 resuelta: P-015 agrega el error "La nueva contraseña debe ser distinta de la actual" (RN-156). |

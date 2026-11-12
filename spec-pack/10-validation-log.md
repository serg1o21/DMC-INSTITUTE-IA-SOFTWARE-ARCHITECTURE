# 10 - Validation Log: prototipo miStorePe

Agente: Prototype Reviewer (BMAD). Fecha: 2026-11-11.
Insumos: `06-user-stories-refinadas.md` (HU-001 a HU-012 Must), `03-reglas-negocio.md` (152 reglas, 135 MUST al 2026-11-11; con RN-154 son 153 y 136), `09-ui-spec.md` (P-001 a P-024), `miStorePe__prototipo_de_pantallas.html` (24 pantallas navegables, lógica en JS; el prototipo no forma parte del Spec Pack).
Alcance: solo gaps funcionales de las HU Must. Las pantallas Should/Could (P-017 a P-022, P-024) se revisaron solo para confirmar que no interfieren con el MVP (UF-00 se respeta: sin el interruptor Should, el inicio es P-004).

---

## 1. Cobertura de HUs Must

| HU | Pantallas en el prototipo | Estado | Comentario |
|---|---|---|---|
| HU-001 | P-001, P-016, P-004 | Cubierta | Validaciones de correo, 12-128, nombre opcional, política y sesión automática (RN-153). |
| HU-002 | P-002 | Parcial | Bloqueo, contador y mensajes OK. No se representa el cierre por inactividad (O-01). |
| HU-003 | P-015 | Cubierta | Contraseña actual, 12-128, aviso de cierre de otras sesiones. |
| HU-004 | P-015, P-002, P-003 | Cubierta | Cierre con contraseña, banner con fecha límite, reapertura y variante "falta de pago". |
| HU-005 | P-005, P-006 (agregar variante) | Cubierta | Campos por categoría, normalización, duplicados, "Única", SKU por categoría, sin stock inicial. |
| HU-006 | P-006, P-007, P-004 (pestaña Desactivados) | Cubierta | Atributos bloqueados con movimientos, vencimiento auditable, advertencia de precio, desactivar/reactivar. |
| HU-007 | P-008 | Cubierta | Cantidad, costo con coma, sin fecha, bloqueo de desactivados, nuevo costo promedio en el toast. |
| HU-008 | P-013 | Parcial | La anulación existe, pero el botón aparece aunque no proceda y no hay acceso rápido tras registrar (O-02, O-03, O-04). |
| HU-009 | P-009 | Cubierta | Fusión, otro precio, máx. 9,999, vencidos deshabilitados, pérdida con una sola confirmación, fecha pasada, offline. |
| HU-010 | P-011 | Cubierta | Anular venta y línea, plazo 24 h desde el registro, bloqueo por ajuste posterior, venta pasa a "Anulada". |
| HU-011 | P-010, P-011 | Cubierta | Ganancia por línea y total, costo usado por línea, anuladas visibles sin sumar, formatos. |
| HU-012 | P-012, P-013, P-014 | Cubierta | Búsqueda RN-138, ajuste con diferencia, motivo, "Merma por vencimiento", sin anular ajustes. |

Faltantes: ninguna HU Must está ausente.

---

## 2. Reglas de negocio

### 2.1 Reflejadas en la UI (MUST)

- Cuenta: RN-001 a RN-004, RN-007, RN-048, RN-049, RN-090, RN-094 a RN-096, RN-103, RN-105, RN-110, RN-127, RN-130, RN-131, RN-133, RN-134, RN-141, RN-153.
- Catálogo: RN-008 a RN-013, RN-015 a RN-017, RN-041, RN-050, RN-058 a RN-062, RN-064 a RN-066, RN-091, RN-093, RN-098 a RN-100, RN-108, RN-111 a RN-113, RN-115 (solo como aviso), RN-119, RN-121 a RN-124, RN-135 a RN-137, RN-146, RN-147.
- Entradas: RN-014, RN-018 a RN-020, RN-042, RN-047, RN-054, RN-055, RN-057, RN-092, RN-118, RN-140, RN-142.
- Ventas y ganancia: RN-021 a RN-023, RN-025, RN-027, RN-028, RN-043, RN-052, RN-053, RN-072 a RN-074, RN-076 a RN-086, RN-106, RN-107, RN-109, RN-125, RN-126, RN-139, RN-151.
- Stock y ajuste: RN-024, RN-026, RN-037 a RN-040, RN-044, RN-075, RN-089, RN-116, RN-138, RN-144, RN-145.
- Transversales: RN-035, RN-036, RN-148, RN-149, RN-150, RN-152. PRINCIPIO-005: P-009 no tiene campos del comprador y P-014 advierte no escribir datos de clientes.

### 2.2 Reflejadas parcialmente

| Regla | Dónde | Problema |
|---|---|---|
| RN-056, RN-087 (entradas) | P-013 | El botón "Anular entrada" se muestra siempre; el bloqueo solo aparece al tocarlo (O-02). |
| RN-088 | P-013 | Al anular la primera entrada el diálogo dice "el costo promedio vuelve a S/ 0.00": comportamiento sin definir (O-04). |
| RN-143 | P-014 | El límite de 200 se valida sin quitar espacios al inicio y al final (O-11). |

### 2.3 No reflejadas

| Regla | Esperado | Hallazgo |
|---|---|---|
| RN-097 (y RN-132) | P-002 con "Tu sesión se cerró por inactividad" (UF-02 paso 3) | No existe ese estado ni forma de simularlo (O-01). |

### 2.4 Invisibles por diseño (backend, sin expresión en UI)

RN-005, RN-006, RN-045, RN-046, RN-120, RN-128, RN-129, RN-132 (conteo de actividad). Se validan con tests de integración (T-02, T-05, T-07), no en el prototipo.

---

## 3. Flujo de usuario

| Flujo | Resultado | Fricción o callejón |
|---|---|---|
| UF-01 Alta y primer producto | Completo | Falta el aviso "Stock 0. Registra una entrada" en P-006 (O-08). Ir a la política desde P-001 borra el formulario (O-05). |
| UF-02 Inicio de sesión | Completo salvo inactividad | O-01. |
| UF-03 Venta en un live | Completo en 2 toques + texto (tocar resultado, "Registrar venta") | Cumple el máximo de 4 toques. En "Otro día" la hora por defecto es fija (O-09). |
| UF-04 Venta olvidada | Completo | O-09. |
| UF-05 Anular venta o línea | Completo | Venta anulada se muestra distinta en P-010 y P-011 (O-10). Buen acceso desde el banner de éxito de P-009 ("Ver venta"). |
| UF-06 Entrada y anulación | Completo con fricción | Tras registrar por error, anular exige: Stock > buscar > variante > Anular (O-03). Botón visible aunque no proceda (O-02). |
| UF-07 Consulta y ajuste | Completo | "Volver" de P-013 siempre va a Stock, aunque se haya llegado desde P-006 (O-06). |
| UF-08 Catálogo | Completo | - |
| UF-09 Cuenta | Completo | - |

Callejones sin salida: ninguno bloqueante (la navegación inferior siempre está). P-008 no tiene enlace de regreso al origen (O-06).

---

## 4. Observaciones

| # | Observación | Tipo | Severidad | Acción sugerida | HU afectada |
|---|---|---|---|---|---|
| O-01 | P-002 no tiene el estado "Tu sesión se cerró por inactividad" ni un modo para simularlo. | gap | Media | Agregar el banner al panel del prototipo (estado "sesión vencida") y verificar el retorno a P-002 (RN-097). | HU-002 |
| O-02 | En P-013 "Anular entrada" aparece en todas las entradas vigentes; fuera de plazo o con movimientos posteriores solo falla al tocar. P-011 en cambio reemplaza el botón por el motivo. | fricción | Media | Mostrar el botón solo si procede y, si no, el motivo en texto (RN-056, RN-087), igual que P-011. | HU-008 |
| O-03 | Tras registrar una entrada en P-008 no hay acceso directo para anularla; el caso típico del error (HU-008, borde "la anula de inmediato") exige 4 o más pasos. | fricción | Media | Añadir en el éxito de P-008 un enlace "Ver en el historial" a P-013, como "Ver venta" en P-009. | HU-008, HU-007 |
| O-04 | Anular la primera entrada deja "costo promedio S/ 0.00"; HU-008 (nota QA) dice que falta definir qué muestra la variante. | gap | Alta | Abrir pregunta en `05-preguntas-abiertas.md` y definir regla (propuesta: mostrar "sin entradas", coherente con RN-062 y P-006). No implementar hasta resolver (PRINCIPIO-009). | HU-008 |
| O-05 | El enlace a la política en P-001 navega fuera del formulario y al volver se pierden los datos escritos. | fricción | Media | Abrir P-016 en un diálogo o conservar el borrador del formulario (RN-003). | HU-001 |
| O-06 | "Volver" de P-013 siempre lleva a Stock y P-008 no tiene enlace de regreso, aunque se llegue desde P-006. | fricción | Baja | Back contextual al origen (producto o stock). | HU-012, HU-007 |
| O-07 | P-004 y P-010 no tienen "Ver más" (listas de 20, ADR-0006). | gap | Baja | Agregar paginación en ambas. | HU-006, HU-011 |
| O-08 | P-006 no muestra el aviso "Stock 0. Registra una entrada para cargar unidades" que pide UF-01. | gap | Baja | Agregar el banner cuando todas las variantes tengan stock 0 y sin entradas (RN-061). | HU-005, HU-007 |
| O-09 | En P-009 "Otro día" la hora por defecto es 20:00; si se elige hoy antes de esa hora, da error de fecha futura. | fricción | Baja | Precargar la hora actual (RN-078, RN-126). | HU-009 |
| O-10 | Venta anulada: P-010 muestra importes tachados con "no suma"; P-011 muestra Total y Ganancia S/ 0.00. | comprensión | Baja | Usar en P-011 el mismo criterio que P-010 (RN-054, RN-053). | HU-010, HU-011 |
| O-11 | El tope de 200 caracteres del motivo se cuenta con espacios al inicio y al final. | gap | Baja | Contar y validar sobre el texto recortado (RN-143). | HU-012 |
| O-12 | Variantes sin entradas muestran "Costo promedio S/ 0.00" en P-007, P-008 y P-013, mientras P-006 dice "sin entradas". | comprensión | Baja | Unificar en "sin entradas" (RN-062). | HU-006, HU-007, HU-012 |
| O-13 | En P-007 la variante "Única" bloqueada muestra "Talla: Sin talla / Color: Única". | comprensión | Baja | Mostrar "Variante: Única" (RN-059, P-55). | HU-006 |
| O-14 | La auditoría del vencimiento (RN-115) solo se anuncia con un texto de ayuda; no hay dónde consultarla. | mejora | Baja | Aceptable para el MVP; decidir si se lista en P-013. Sin cambio de alcance. | HU-006 |

---

## 5. Decisiones recomendadas antes de implementar

1. Definir la regla de costo promedio al anular la primera entrada y registrarla como pregunta abierta (O-04). Bloqueante para T-020/HU-008. Estado: aprobada por Sergio el 2026-11-11 (pregunta P-64, opción A; RN-154, nota en RN-118, criterio [AUDITORÍA v4] en HU-008, P-013 en la UI spec y prototipo). También cierra O-12.
2. Alinear P-013 con P-011: botón de anular solo si procede, motivo visible si no (O-02).
3. Añadir el enlace "Ver en el historial" en el éxito de P-008 (O-03).
4. Agregar el estado de sesión vencida en P-002 al prototipo y a la spec de pruebas E2E (O-01).
5. Evitar la pérdida del formulario de registro al ver la política (O-05).
6. Ajustes menores de la spec y el prototipo: back contextual (O-06), "Ver más" (O-07), aviso de stock 0 (O-08), hora actual por defecto (O-09), presentación de venta anulada (O-10), conteo recortado del motivo (O-11), texto "sin entradas" (O-12), etiqueta "Única" (O-13).

---

## 6. Score de completitud

Conteos al 2026-11-11, antes de RN-154. No se recalculan: RN-154 se refleja en P-006, P-007, P-008 y P-013 (sección 7).

| Métrica | Resultado |
|---|---|
| HUs Must representadas | 12 de 12 (100%) |
| HUs Must sin gaps | 10 de 12 (83%): parciales HU-002 y HU-008 |
| Reglas MUST con expresión en UI | 127 (135 menos 8 de backend) |
| Reglas MUST reflejadas completas | 122 de 127 (96.1%) |
| Reglas MUST reflejadas, incluidas parciales | 126 de 127 (99.2%) |
| Reglas MUST reflejadas sobre el total | 122 de 135 (90.4%) |

Veredicto: el prototipo está listo para implementación tras resolver la decisión 1 (bloqueante) y aplicar las decisiones 2 a 5. El resto puede corregirse durante la implementación.

---

## 7. Estado de las observaciones (actualizado 2026-11-12)

| # | Estado | Dónde quedó |
|---|---|---|
| O-01 | Resuelta | P-002 en `09` (banner de inactividad); T-026 y escenario E2E en T-036 |
| O-02 | Resuelta | P-013 en `09`; campos `voidable` y `voidBlockedReason` (pregunta P-66); T-028 |
| O-03 | Resuelta | Éxito de P-008 con "Ver en el historial"; UF-06; T-028 |
| O-04 | Resuelta | Pregunta P-64, opción A; RN-154; T-020 |
| O-05 | Resuelta | Política en un diálogo desde P-001 (lista de aprobación de `09`, punto 1); T-026 y T-038 |
| O-06 | Resuelta | "Volver" contextual en P-008 y P-013; T-028 |
| O-07 | Resuelta | "Ver más" en P-004 y P-010; T-027 y T-030 |
| O-08 | Resuelta | Banner de stock 0 en P-006; T-027 |
| O-09 | Resuelta | "Otro día" con fecha y hora actuales (lista de aprobación de `09`, punto 6); T-029c |
| O-10 | Resuelta | `voidedAmount` y `voidedProfit` (pregunta P-66); P-010 y P-011; T-030 |
| O-11 | Resuelta | Conteo del motivo sobre el texto recortado en pantalla (T-028); validación de RN-143 en el backend (T-021) |
| O-12 | Resuelta | "Costo promedio: sin entradas" (RN-062, RN-154); T-027 y T-028 |
| O-13 | Resuelta | "Variante: Única" en P-007; T-027 |
| O-14 | Aceptada sin cambio | Fuera del MVP: la auditoría del vencimiento no se lista en pantalla |

Con todas las observaciones cerradas, el veredicto de la sección 6 se cumple. Las decisiones posteriores (P-74 a P-77) se resolvieron el 2026-11-12 y no provienen del prototipo.

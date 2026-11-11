# 01 - Project Brief: miStorePe

| Campo | Valor |
|---|---|
| Producto | miStorePe |
| Tipo | Aplicación web responsiva (celular y laptop) |
| País y ciudad | Perú, Lima |
| Fase | Descubrimiento |
| Responsable | Sergio (desarrolla solo y es el único aprobador) |
| Fecha | 2026-11-10 |
| Origen de la información | Hipótesis del ejercicio académico, sin entrevistas a usuarios todavía |

---

## 1. Problema y oportunidad

### Problema
El pequeño emprendedor de Lima que vende ropa, perfumes y/o accesorios por redes sociales controla su inventario en un cuaderno. Esto le genera cuatro problemas concretos:

1. **Pérdida de tiempo:** no tiene los datos guardados de forma confiable, por lo que debe volver a contar físicamente su stock una y otra vez. Esto le consume varias horas a la semana y genera confusión.
2. **Pierde la cuenta de variantes:** no sabe con certeza qué tallas y colores (ropa) o qué presentaciones en ml (perfumes) le quedan.
3. **No sabe qué reponer:** no identifica qué productos se venden más ni cuáles no se están vendiendo, por lo que no puede decidir qué le conviene comprar a sus proveedores.
4. **No sabe cuánto gana:** no conoce la ganancia por cada salida de producto. Las salidas de inventario se dan únicamente por ventas.

### Oportunidad
- En 2023 participaron unas 332,000 empresas en el comercio electrónico peruano, que movió US$13,000 millones (+7%). Solo el 8% tenía tienda online propia; el resto vende por redes sociales u otras vías (Capece, Observatorio Ecommerce 2023-2024, vía [Minería y Energía](https://mineriaenergia.com/comercio-electronico-crece-7-en-peru-cuantos-negocios-venden-por-internet/)).
- En una encuesta de Perú Factura (CCL, 2021), "ropa y calzado" representó el 15.85% de los rubros de emprendedores que venden por redes ([La Cámara](https://lacamara.pe/mas-del-30-de-emprendedores-en-peru-usa-redes-sociales-desde-inicio-de-pandemia/)). La encuesta no publica metodología ni tamaño de muestra.
- Los ingresos de TikTok for Business en Perú crecieron 64% en 2025 y 76% en el primer trimestre de 2026, según datos de la propia empresa ([Forbes Perú](https://forbes.pe/negocios/2026-06-04/tiktok-for-business-crece-a-doble-digito-el-primer-trimestre-en-peru-estas-son-las-industrias-que-mas-la-usan/)). Es un indicador indirecto del crecimiento de la venta por esta red.

No existe una cifra oficial reciente de cuántos emprendedores en Lima venden exclusivamente por redes sociales. Dimensionar el mercado es un pendiente.

---

## 2. Usuarios y segmentos objetivo

### Usuario principal
Pequeño emprendedor de Lima que:
- Vende en lives de TikTok y publica fotos de sus productos en Facebook Marketplace.
- Maneja entre 30 y 40 productos (cantidad variable).
- Compra a varios proveedores, sin una frecuencia de reposición fija.
- Usa celular y, si tiene, laptop.
- Conoce Yape y Plin y sabe manejar Excel.
- Puede trabajar con otras personas, pero **solo una persona es responsable de registrar las salidas por ventas**.

### Segmentos (misma prioridad desde el inicio)

| Segmento | Qué necesita controlar de forma distinta |
|---|---|
| Ropa | Tallas y colores por producto |
| Perfumes | Presentaciones en ml y fechas de vencimiento |
| Accesorios | Sin atributos obligatorios; los que no tienen tallas ni colores usan la variante "Única". Otras necesidades se validan en el piloto. |

### Fuera de alcance
Negocios que no son pequeños emprendedores: tiendas físicas grandes y mayoristas.

---

## 3. Propuesta de valor

**Para** pequeños emprendedores de Lima que venden ropa, perfumes y accesorios por TikTok y Facebook Marketplace,
**que** pierden horas recontando su mercadería anotada en un cuaderno y no saben qué reponer ni cuánto ganan,
**miStorePe** es una aplicación web de inventario
**que** les permite saber en todo momento cuánto stock tienen por variante, qué productos no se venden y cuánto ganan por cada venta,
**a diferencia de** el cuaderno, que obliga a contar de nuevo, y de Excel, que el emprendedor percibe como poco amigable y poco práctico para el día a día.

### Por qué cambiarían
- Dejar de recontar físicamente el stock varias veces por semana.
- Tener información para decidir qué reponer.
- Conocer su ganancia por producto vendido.

---

## 4. Modelo de negocio preliminar

| Elemento | Definición |
|---|---|
| Modelo | Suscripción mensual |
| Precio estimado | S/ 25 al mes por emprendedor |
| Lanzamiento | Gratuito, para que los emprendedores comprueben que les sirve |
| Paso a pago | Se avisa a los usuarios gratuitos con 1 mes de anticipación que la gratuidad termina; después deben pagar |
| Canal inicial | Boca a boca con conocidos |
| Canal posterior | Redes sociales y contacto directo |
| Costos conocidos | Hosting propio (hay presupuesto asignado) |

Ingreso esperado tras el piloto: 3 emprendedores x S/ 25 = **S/ 75 al mes**.

---

## 5. Métricas de éxito (KPIs medibles)

| ID | KPI | Cómo se mide | Meta |
|---|---|---|---|
| KPI-01 | Emprendedores en el piloto | Cuentas activas registradas | 3 emprendedores en el piloto de 1 mes |
| KPI-02 | Validación del piloto | Pregunta directa al final del mes: "¿Te sirve y seguirás usándolo?" | Al menos 2 de 3 responden que sí (67%) |
| KPI-03 | Retención al mes | Emprendedores que registran al menos una venta en la última semana del piloto | Al menos 2 de 3 |
| KPI-04 | Conversión a pago | Emprendedores del piloto que pagan la primera mensualidad | 3 de 3 (100%) |
| KPI-05 | Tiempo semanal de conteo de stock | Horas por semana reportadas por el emprendedor, antes de usar la app y después de 4 semanas de uso | Línea base y resultado medidos; la meta de reducción se fija con esos datos |
| KPI-06 | Confiabilidad del inventario | Diferencia entre unidades en la app y el conteo físico al final del piloto | Medida en el conteo final; la meta se fija con esos datos |

---

## 6. Riesgos identificados y supuestos

### Riesgos

| ID | Riesgo | Impacto |
|---|---|---|
| R-01 | El responsable no registra cada venta durante un live de TikTok | El stock en la app deja de coincidir con el real y el emprendedor deja de confiar en los datos |
| R-02 | El emprendedor vuelve al cuaderno por costumbre | Abandono del producto antes de terminar el piloto |
| R-03 | La conversión a pago es menor que la esperada (meta 3 de 3) | Ingresos menores a S/ 75 al mes |
| R-04 | Muestra pequeña (3 emprendedores conocidos) | Resultados del piloto poco representativos del mercado |

### Supuestos a validar

| ID | Supuesto | Cómo validarlo |
|---|---|---|
| S-01 | El emprendedor dedica varias horas a la semana a recontar su stock | Medir línea base con los 3 emprendedores del piloto (KPI-05) |
| S-02 | Pagaría S/ 25 al mes | Conversión real al terminar la gratuidad (KPI-04) |
| S-03 | Excel no se usa porque se percibe como poco amigable | Preguntar a los emprendedores del piloto |
| S-04 | 30-40 productos es el volumen típico | Registrar cuántos productos carga cada emprendedor del piloto |
| S-05 | Ropa y perfumes tienen la misma prioridad | Ver qué segmento predomina en el piloto |
| S-06 | Un único responsable registra las ventas | Confirmar con los emprendedores del piloto |

---

## 7. Restricciones conocidas

| Tipo | Restricción |
|---|---|
| Contexto | Ejercicio académico del Diploma en AI-First Software Architecture |
| Equipo | Una persona (Sergio), que desarrolla y aprueba todos los cambios |
| Plazo | Sin fecha de entrega definida |
| Presupuesto | Hay presupuesto para hosting propio |
| Datos | Solo inventario y datos de la cuenta del emprendedor; no se guardan datos de clientes finales ni de pagos (ver `00-constitucion.md`) |
| Legal | Ley N° 29733 de Protección de Datos Personales y su reglamento (D.S. N° 016-2024-JUS) para los datos de los emprendedores |
| Alcance | Sin pagos ni comprobantes SUNAT, sin integración con redes sociales |
| Técnicas | Definidas en `00-constitucion.md` |

### Pendientes
- Dimensionar el mercado de pequeños emprendedores que venden por redes en Lima.
- Definir cuánto dura la etapa gratuita en total y la fecha de inicio del cobro.
- Identificar necesidades específicas del segmento de accesorios.
- Entrevistar a los emprendedores del piloto para validar los supuestos S-01 a S-06.

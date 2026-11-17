# CC-001 · Periodo de gracia configurable

| Campo | Contenido |
|---|---|
| ID | CC-001 |
| Solicitante | Rodrigo Valdespino Vertiz (00520621), dupla y titular de referencia de la entrevista del 30-09-2026 |
| Recibida | 12 de noviembre de 2026 |
| Versión analizada | Especificación de requisitos v1.1, ADR-001 y diagramas en `doc/arch/diagramas.md` |
| Estado | Analizada. Recomendación: aceptar con condiciones |

## 1. La solicitud, con las palabras de quien la pidió

> Oye Emiliano, estuve revisando Memorium y tengo un cambio. Los 14 días de gracia son fijos y a mí no me cuadran: en verano me voy hasta tres semanas a lugares sin señal, y si justo se me vence el plazo me puede agarrar fuera. Quiero poder elegir yo cuánto dura mi periodo de gracia, de 7 a 30 días. Es solo cambiar un número, ¿no? Lo necesito antes de usarlo con mi cápsula de verdad.

Lo que pide en una frase: que cada titular decida la duración de su periodo de gracia, entre 7 y 30 días, en lugar de que sea siempre 14.

## 2. Traducción a Memorium

Rodrigo habla de "gracia", "plazo" y "cápsula de verdad". En el proyecto esas palabras tienen nombre y lugar:

| Lo que dijo | Cómo se llama en Memorium | Dónde vive |
|---|---|---|
| "Los 14 días de gracia" | Estado **Periodo de gracia** (RF-010). Los 14 días los propuse yo y se verifican con S-05. | Estado `Gracia` del diagrama de estados (diagrama 7). Hoy dice "pasan 14 días sin respuesta". |
| "Elegir yo cuánto dura" | Atributo nuevo de la cápsula: `periodoGraciaDias`. Es un dato del titular, igual que `plazoInactividadDias` (RF-006). | Clase `Capsula` (diagrama 6), columna `periodo_gracia_dias` en la tabla `CAPSULA` (diagrama 8). |
| "Si se me vence el plazo" | Estado **Esperando confirmaciones**. No es lo mismo que la gracia: el plazo (RF-006) no cambia. | Diagrama 7. No se toca. |
| "Me agarra fuera" | El titular no puede dar señal de vida (RF-012) dentro de la ventana de cancelación. Es el riesgo central de RNF-SEG-003. | Módulo Liberación, con el Reloj. |
| "Mi cápsula de verdad" | Una cápsula `Activa` que ya existiría con 14 días. | Datos ya guardados: hay que migrarlos. |

Módulos de ADR-001 que participan: **Liberación** (calcula el cierre y decide liberar), **Avisos** (cadencia de RF-011) y **Cápsulas** (guarda y edita el valor). **Personas** no cambia.

## 3. Impacto directo

Lo que cambia porque el cambio lo toca de frente:

| Artefacto | Qué cambia |
|---|---|
| RF-010 | "Abre un periodo de gracia de 14 días" pasa a "de N días, según la cápsula". La fecha de cierre es la apertura más N. |
| RF-013 | El criterio dice "al cerrar el día 14". Pasa a "al cerrar el día N". |
| RF-012 | El ejemplo "día 5 de 14" se reescribe en términos de N. La regla no cambia. |
| RF-025 (nuevo) | Configurar el periodo de gracia: entre 7 y 30 días, valor inicial 14. Criterio como RF-006: 6 y 31 se rechazan, 7 y 30 se aceptan. |
| Caso de uso CU-01 | Un campo más en el registro de la cápsula. |
| Diagramas 6, 7 y 8 | Atributo en la clase, texto de la transición ("pasan N días") y columna en el ER. |

## 4. Impacto indirecto

Lo que Rodrigo no mencionó. Ordenado de más a menos caro.

**4.1 Volver a demostrar RNF-SEG-003 (el más caro).** La batería de al menos 50 escenarios de RNF-SEG-003 se pensó con una constante: 14 días. Si la duración es un dato que se puede editar, aparecen casos que antes no existían:

- Que el titular edite `periodoGraciaDias` mientras la cápsula ya está en `Gracia`. Si el cierre se calcula a la hora de revisar (apertura más el valor actual), editar el campo mueve la fecha de liberación de una cápsula en curso. La salida es guardar `fecha_cierre_gracia` en el momento de abrir la gracia y no volver a calcularla.
- Los bordes del rango: 6, 7, 30 y 31 días.
- Una cápsula migrada desde la versión de 14 fijos.

Un error aquí no se arregla después. Una liberación indebida entrega el dinero y la vida privada de alguien que sigue vivo, y no se puede cerrar de nuevo. Por eso pesa más que cualquier otra línea de esta tabla, aunque Rodrigo ni la vea.

**4.2 El campo nuevo es una puerta de ataque (RNF-SEG-005, RF-016).** Hoy RNF-SEG-005 protege que nadie cambie a quién va dirigida la herencia con solo un teléfono desbloqueado. Con este cambio, alguien con ese teléfono podría poner la gracia en 7 días y reducir la ventana del titular real. Hay que meter `periodoGraciaDias` en el alcance de RNF-SEG-005 (dos factores independientes) y ajustar RF-016: el campo solo se edita con la cápsula en estado `Activa`, nunca en `Esperando confirmaciones` ni en `Gracia`.

**4.3 La cadencia de avisos choca con el cierre (RF-011).** La regla ya es relativa: un aviso cada 72 horas mientras el periodo está abierto. Pero con 14 días (336 horas) el último aviso sale a las 288 y no hay coincidencia. Si N es múltiplo de 3 (9, 12, 15, 18, 21, 24, 27 o 30), el último aviso cae exactamente en el instante del cierre: le diría al titular "estás a tiempo" a la misma hora en que la cápsula se libera. La regla tiene que decir que solo se envían avisos antes del cierre, y el criterio de aceptación se reescribe con dos valores de N, no con uno. Esto se prueba con RNF-REN-001 (15 minutos) sin cambios.

**4.4 Cápsulas que ya existen.** Todas deben quedar con 14 por defecto, para que el comportamiento que el titular ya aceptó no cambie sin avisarle. Es una migración de una columna con valor inicial y una restricción de rango.

**4.5 El heredero espera más.** Con 30 días, el heredero de la entrevista (un hermano de 16 años, RNF-USA-002) tarda hasta 16 días más en recibir algo. Es un costo real del cambio. Se mitiga mostrándole el cierre exacto de la gracia en la pantalla de espera, y de todas formas es una decisión del titular, que es quien conoce a su heredero.

**4.6 Lo que no cambia.** El plazo de inactividad (RF-006), el recordatorio a 7 días (RF-020) y la señal a mitad del plazo (RF-024) siguen igual: miden otra cosa. ADR-001 se mantiene: el cambio entra dentro de Liberación y Avisos, no pide separar nada ni abrir una ADR nueva.

## 5. Estimación de esfuerzo

**De dónde sale el número.** Descompuse el cambio en 8 tareas y, a cada una, le puse tres valores en horas: optimista, probable y pesimista. El esperado de cada una es (optimista + 4 x probable + pesimista) / 6, el método PERT. Después sumé. No tengo datos de proyectos anteriores contra los cuales calibrar, así que esto es una estimación mía por descomposición y se corrige con lo que se mida.

| # | Tarea | Opt. | Prob. | Pes. | Esperado |
|---|---|---|---|---|---|
| T1 | Especificación: RF-010, 011, 012, 013, 016, RNF-SEG-005, RF-025 nuevo, trazabilidad y registro de cambios | 2 | 3 | 5 | 3.2 |
| T2 | Diagramas 6, 7, 8 y el texto del flujo de CU-05 | 1 | 1.5 | 2.5 | 1.6 |
| T3 | Datos: columna, valor inicial 14, restricción 7 a 30, migración | 1 | 2 | 3 | 2.0 |
| T4 | Liberación: guardar la fecha de cierre al abrir la gracia, y que el Reloj la use | 3 | 5 | 8 | 5.2 |
| T5 | Avisos: cadencia contra el cierre, regla "solo antes del cierre" | 2 | 4 | 6 | 4.0 |
| T6 | Cápsulas: campo en CU-01, validación, edición solo en `Activa`, doble factor | 4 | 6 | 10 | 6.3 |
| T7 | Pruebas de RNF-SEG-003: bordes, edición en cada estado, migrada, un solo factor | 4 | 6 | 10 | 6.3 |
| T8 | Regresión del resto de la batería y revisión | 2 | 3 | 4 | 3.0 |
| | **Total** | **19** | **30.5** | **48.5** | **31.6** |

**Resultado:** unas **32 horas esperadas**, con rango de 19 a 49. A 4 horas efectivas por día, que es lo que una sola persona puede dedicar mientras estudia (ADR-001: lo construye una persona), son unas 8 jornadas.

**Contra "es solo cambiar un número".** El cambio del dato en sí (T3) es 2 horas, el 6% del total. El resto es lo que el pedido no veía: la fecha congelada y las pruebas de seguridad (T4 y T7 suman 11.5 horas, más de un tercio), la protección del campo (T6) y los avisos (T5).

**Qué haría variar el número.** Si la batería de RNF-SEG-003 todavía no existe cuando entre el cambio, T7 baja, porque los escenarios nuevos se escriben junto con los demás. Si falla un escenario de borde, T4 sube hacia su valor pesimista. El rango es amplio a propósito: T4, T6 y T7 tocan la regla irreversible y ahí un imprevisto cuesta más.

## 6. Alternativas consideradas

| Opción | Esfuerzo (mismo método) | Qué resuelve | Qué deja mal |
|---|---|---|---|
| A. Subir la gracia fija a 30 días para todos | unas 11 horas (rango 7 a 17.5) | Rodrigo queda cubierto sin campo nuevo y sin superficie de ataque. | Todos los herederos esperan 30 días, aunque su titular no viaje. El 14 era una propuesta mía todavía sin verificar (S-05), y esto la reemplazaría por otro número igual de arbitrario. |
| **B. Configurable de 7 a 30 (la solicitud)** | unas 32 horas (rango 19 a 49) | Cada titular decide según su vida. | Cuesta casi el triple que A. |
| C. "Modo viaje" que pausa el contador de inactividad | no se estima | Resolvería el viaje sin tocar la gracia. | Sería una segunda forma de detener el reloj, que es justo lo que RNF-SEG-003 vigila. Toca RF-007 y RF-024. Es más grande y más riesgosa que B. |

## 7. Recomendación

**Aceptar con condiciones** la opción B, para la versión 1.2 de la especificación.

La razón no es que Rodrigo lo pida. Es que su problema es exactamente el riesgo que RNF-SEG-003 busca evitar: un titular vivo, sin señal, al que se le cierra la ventana de cancelación. Una gracia más larga protege contra la peor falla del sistema, y la opción A lo hace mal porque obliga a todos a esperar más.

Condiciones para aceptarlo:

1. Rango de 7 a 30 días, valor inicial 14. Nunca 0.
2. La fecha de cierre se guarda al abrir la gracia y no se recalcula.
3. El campo solo se edita con la cápsula `Activa` y con doble factor (RNF-SEG-005).
4. Los escenarios nuevos (sección 4.1) se agregan a la batería de RNF-SEG-003 antes de dar el cambio por terminado, no después.
5. Los avisos de RF-011 solo se envían antes del cierre.

**Por qué no rechazar:** el costo de 32 horas es alto, pero no hay una alternativa más barata que resuelva el caso sin dejar a los demás titulares peor.
**Por qué no posponer sin fecha:** Rodrigo necesita el cambio antes de usar su cápsula real, y entre más código dependa del 14 fijo, más caro se vuelve quitarlo.

Si el tiempo disponible no alcanza para las 32 horas, el plan de respaldo es la opción A como medida temporal, con RF-025 pendiente para después.

## 8. Seguimiento

| Fecha | Paso | Quién |
|---|---|---|
| 12-11-2026 | Solicitud recibida | Rodrigo Valdespino |
| 17-11-2026 | Análisis de impacto y recomendación | Emiliano Cabañas |
| Pendiente | Decisión de aceptar, y actualización de la especificación a v1.2 con registro de cambios | Emiliano Cabañas y Rodrigo Valdespino |

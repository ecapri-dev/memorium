# ADR-001 · Monolito modular para Memorium

## Estado

Aceptada · 27 de octubre de 2026

## Contexto

Memorium guarda las llaves de acceso al dinero digital de una persona, junto con cartas, fotos y documentos, en una cápsula cifrada que solo se libera cuando se cumplen dos condiciones a la vez: el plazo de inactividad del titular venció y sus guardianes confirmaron el fallecimiento. Después viene un periodo de gracia de 14 días en el que el titular todavía puede cancelar todo.

Lo que más pesa en esta decisión es que una liberación indebida no tiene vuelta atrás. RNF-SEG-003 exige cero liberaciones indebidas: no se puede abrir una cápsula si falta una confirmación, si el titular dio señal de vida o si la cápsula está en alerta o revocada. Revisar esas condiciones y cambiar el estado tiene que pasar como una sola operación. Si dos partes del sistema ven datos distintos aunque sea por unos segundos, una cápsula podría abrirse cuando no debía.

Hay otros datos reales del caso que también cuentan:

- **Quién lo construye.** Lo desarrolla una sola persona. No hay equipos separados que necesiten desplegar su parte sin esperar a los demás.
- **Qué tan claras están las fronteras.** Los módulos ya están definidos (Cápsulas, Personas, Liberación y Avisos) y cada uno tiene sus propios datos. Liberación todavía puede cambiar, porque el número de confirmaciones y la regla del guardián que no hereda (RF-005) salieron de la entrevista y podrían ajustarse.
- **Qué tan grande es la carga.** La meta de la primera versión es 10,000 cápsulas activas (RNF-ESC-001). Todo el proceso se mide en días, no en milisegundos, así que no hace falta escalar una parte del sistema por separado.
- **Qué disponibilidad se pide.** 99.5% al mes (RNF-CON-001). Entre menos piezas se desplieguen, menos puntos pueden fallar.
- **Qué depende de terceros.** Solo Avisos habla con proveedores externos de correo, SMS y WhatsApp, y es la parte que junta más trabajo de golpe cuando vencen varios plazos.

Se consideraron tres opciones:

| Opción | Por qué sí | Por qué no |
|---|---|---|
| Monolito sin módulos | Es lo más rápido de arrancar. | Con el tiempo cualquier parte termina leyendo las tablas de cualquier otra, y la regla de liberación quedaría repartida por todo el código. Eso hace muy difícil probar RNF-SEG-003. |
| Monolito modular | Una sola base de datos y una sola transacción para liberar, con fronteras claras entre módulos. | Hay que respetar la disciplina de no leer tablas ajenas, porque el código lo permite. |
| Microservicios | Cada parte se despliega y escala por su cuenta. | Para una sola persona, operar cuatro servicios cuesta más que el producto. Además, la liberación pasaría a depender de mensajes entre servicios y de consistencia eventual, que es justo el riesgo que RNF-SEG-003 no tolera. |

## Decisión

Memorium se construye como un **monolito modular**: una sola API que se despliega como una unidad y una sola base de datos PostgreSQL, divididas en cuatro módulos.

| Módulo | Qué hace | Con quién habla |
|---|---|---|
| Cápsulas | Guarda la cápsula y sus elementos cifrados. | Personas, para saber quién hereda qué. |
| Personas | Herederos, guardianes y la verificación de identidad (RF-022). | Nadie; los demás le preguntan a él. |
| Liberación | Plazos, confirmaciones, periodo de gracia y la decisión de liberar. Incluye el Reloj que revisa los plazos cada día. | Personas, Cápsulas y Avisos. |
| Avisos | Envía las notificaciones a titulares, guardianes y herederos. | El servicio de mensajería externo. |

Las reglas que vuelven esto un monolito *modular* y no un monolito sin más:

1. Cada módulo es dueño de sus tablas. Ningún módulo lee ni escribe las tablas de otro; si necesita un dato ajeno, se lo pide al módulo dueño por su interfaz.
2. Liberación revisa las condiciones y cambia el estado de la cápsula dentro de una sola transacción.
3. Avisos queda detrás de una interfaz, así que cambiar de proveedor de correo o SMS no toca a los demás módulos.

Los diagramas de contexto, contenedores y componentes de esta decisión están en [diagramas.md](diagramas.md).

## Consecuencias

**Lo que se gana.** Probar RNF-SEG-003 es más directo, porque la regla de liberación vive en un solo módulo y corre en una sola transacción. Se despliega una sola pieza, así que mantener el 99.5% de disponibilidad depende de menos cosas. Y una persona puede entender y operar todo el sistema.

**Lo que se paga.** Si Avisos se satura porque vencieron muchos plazos el mismo día, también le resta recursos al resto de la API, porque comparten el mismo proceso. Además, la frontera entre módulos depende de la disciplina: el código permite leer una tabla ajena aunque la regla lo prohíba, así que hay que cuidarlo en cada revisión.

**Cuándo se revisa esta decisión.** El primer módulo que saldría a un servicio aparte es Avisos. La señal concreta sería que los avisos del periodo de gracia empiecen a llegar tarde respecto a lo que pide RNF-REN-001, o que una falla del proveedor de mensajería alente el resto de la aplicación. En ese momento Avisos se separa con su propia cola de mensajes y los otros tres módulos se quedan como están.

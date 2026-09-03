# Visión del producto

**Autor:** Emiliano Cabañas Prieto
**Fecha de la última versión:** 2026-09-03
**Repositorio:** _(pega aquí la URL de tu repo)_

---

## 1. Descripción del sistema

**Nombre del sistema:** Alerta Vecinal

**Descripción:**

Alerta Vecinal es una app donde los vecinos de una colonia se avisan cuando pasa algo: un robo, un incendio, una emergencia médica. El vecino que ve el problema abre la app, marca de qué se trata y manda la alerta. A los que viven cerca les llega la notificación en segundos.

Quien la recibe puede responder de dos formas: confirmar que él también lo vio, o marcarla como falsa alarma. Todo queda guardado, así que después la colonia puede revisar qué pasó y a qué hora.

---

## 2. Problema y usuarios

**El problema:**

Avisar a los vecinos cuando pasa algo es lento y desordenado. Casi todo se mueve por WhatsApp, donde el mensaje se pierde entre conversaciones o le llega a gente que ya ni vive en la colonia. Y nadie sabe si la alerta es real o si es un rumor que alguien reenvió.

El problema tiene dos caras que se contradicen. La colonia se entera tarde de lo que sí importa y, al mismo tiempo, recibe tantos mensajes que termina ignorándolos todos.

**Cómo se resuelve hoy sin el sistema:**

Los vecinos se las arreglan como pueden. Grupos de WhatsApp, gritar, tocar el claxon, llamarle a la caseta cuando hay. Algunas colonias tienen botones de pánico o alarmas físicas, pero esas solo le sirven a quien esté cerca para oírlas.

**Usuarios del sistema:**

| Tipo de usuario | Qué necesita del sistema | Qué le preocupa |
|---|---|---|
| Vecino que reporta | Levantar una alerta en segundos, sin trámites, aunque esté nervioso o a oscuras | Que no le llegue a nadie. Quedar expuesto como el que reportó |
| Vecino que recibe | Enterarse solo de lo que pasa cerca de su casa, y saber si es real | Que lo saturen de falsas alarmas y acabe ignorándolas todas |
| Administrador o comité vecinal | Ver qué pasa en la colonia, dar de alta a vecinos verificados y moderar abusos | Que la usen para acusar sin pruebas. La responsabilidad de lo que ahí se publique |

**Un conflicto entre usuarios:**

El que reporta quiere que su alerta salga ya. Está en una emergencia y cualquier paso extra le estorba.

El que la recibe quiere lo contrario: que le lleguen puras alertas verificadas. Si lo despiertan tres veces por falsas alarmas, para la cuarta ya silenció la app. Y esa cuarta puede ser la real.

El administrador queda atorado en medio. Si pone filtros, retrasa alertas legítimas. Si no los pone, la app se llena de ruido y nadie le cree. Cualquier verificación que proteja a un usuario le estorba al otro, y esa es la primera decisión de diseño que me toca tomar.

---

## 3. Alcance

### Dentro del alcance

- Registrar vecinos verificando que de verdad vivan en la colonia.
- Levantar una alerta eligiendo tipo (robo, incendio, emergencia médica, persona sospechosa) y ubicación.
- Notificar a los vecinos que estén dentro del radio que definió su colonia.
- Confirmar o desestimar una alerta.
- Guardar el historial de cada alerta: qué pasó, a qué hora, cuántos la confirmaron.
- Dar de alta y de baja vecinos, y revisar los reportes de abuso, desde el panel del administrador.

### Explícitamente fuera del alcance

- **Llamar solo al 911 o a la policía.** Exige convenios con autoridades y una responsabilidad legal que no puedo sostener.
- **Videovigilancia o cámaras en vivo.** Es otro sistema completo, con su propio costo de infraestructura y almacenamiento.
- **Reconocimiento facial o identificar personas en fotos.**
- **Hardware propio** (botones de pánico, sirenas, sensores). La primera versión corre en el celular que el vecino ya trae.
- **Rastreo continuo de la ubicación.** Solo se comparte la ubicación al momento de levantar la alerta.

**Por qué queda fuera:**

El reconocimiento facial lo dejé fuera por una razón de fondo, no por falta de tiempo. Señalar a alguien como sospechoso a partir de una foto puede arruinarle la vida a un inocente, y el sistema no tiene cómo garantizar que esa identificación esté bien. Lo que hace Alerta Vecinal es avisar rápido a quien está cerca, no juzgar quién sale en la imagen. Aparte, meterlo abriría obligaciones de protección de datos personales que un proyecto de un semestre no puede cumplir en serio.

---

## 4. Tipo de sistema y restricciones

**Tipo de sistema:** Web y SaaS, con exigencias de sistema crítico.

**Por qué es de ese tipo:**

Alerta Vecinal se usa por internet desde el celular y atiende a muchas colonias a la vez desde un solo lugar, sin instalar nada en cada una. Eso lo pone de lleno en Web y SaaS, donde pesan la escalabilidad, la disponibilidad, el aislamiento entre colonias y la seguridad de los datos.

Pero hay algo más. Aquí una falla no es una molestia: es un aviso de emergencia que no llegó. Por eso lo diseño con las exigencias de disponibilidad de un sistema crítico, aunque opere como un SaaS.

**Atributos de calidad que impone:**

| Atributo | Por qué importa en mi caso | Qué pasa si no se cumple |
|---|---|---|
| Disponibilidad | Una emergencia no avisa a qué hora va a pasar | Si está caído justo durante un robo, no sirvió de nada, y la colonia ya no vuelve a confiar |
| Tiempo de respuesta | Todo el valor de la alerta está en los primeros segundos | Una notificación que llega cinco minutos tarde da igual que no haberla mandado |
| Confiabilidad de la información | Las falsas alarmas desgastan la atención de los vecinos | Si se llena de ruido, los vecinos silencian la app y la alerta real se pierde ahí |
| Privacidad y protección de datos | Maneja domicilios, ubicaciones y reportes sobre personas | Una fuga expone dónde vive cada familia. Mal usada, se vuelve herramienta de acoso |

**Reglas de negocio que ya identifiqué:**

1. Solo puede levantar alertas un vecino cuyo domicilio ya verificó el administrador de esa colonia. Sin verificar puedes recibir alertas, pero no generarlas.
2. La alerta llega únicamente a los vecinos dentro del radio de su colonia. Los de otra colonia no la ven, aunque vivan a dos calles.
3. Si un vecino levanta varias alertas seguidas y ninguna se confirma, el sistema le suspende la capacidad de alertar hasta que el administrador lo revise.
4. Una alerta pasa a "confirmada" solo cuando otro vecino distinto la respalda. Y una sola desestimación no la cancela: eso lo revisa el administrador.
5. Las fotos de una alerta se borran cuando la alerta se cierra, salvo que el administrador las conserve por un reporte formal.

---

## 5. Ciclo de vida elegido

**Modelo elegido:** Cascada

**Por qué le conviene a este proyecto:**

Mi alcance ya está cerrado y escrito. Puedo comprometerme hoy a que no va a cambiar durante el semestre.

Mi cliente no está disponible. No tengo vecinos reales usando el sistema y dándome retroalimentación cada semana, así que un modelo que vive de esa retroalimentación se quedaría sin con qué alimentarse.

El riesgo más grande de este proyecto era no entender bien la necesidad, y ese ya lo resolví en las primeras semanas al definir el problema y los usuarios. No es un riesgo técnico: mandar notificaciones por radio geográfico es algo resuelto desde hace años.

Y nadie tiene que auditar ni certificar este sistema. No hay una norma ni una autoridad que me exija evidencia formal por cada fase, así que tampoco necesito un modelo construido para producir esa evidencia.

Con el alcance fijo, sin cliente disponible, con el riesgo principal ya cubierto y sin obligación de auditoría, me conviene avanzar en orden: especificar, diseñar, validar.

### Alternativas descartadas

**Alternativa 1:** Ágil

_Por qué la descarté:_ Ágil sirve cuando tienes un cliente presente todo el tiempo, reaccionando a entregas cortas que redefinen la vuelta siguiente. Yo no tengo vecinos dándome retroalimentación semanal. Sin eso, ágil pierde justo el mecanismo que lo hace funcionar.

**Alternativa 2:** Modelo V

_Por qué la descarté:_ Es cierto que mi sistema tiene rasgos de sistema crítico. Pero el modelo V está hecho para sistemas que necesitan evidencia formal y certificación externa, como equipo médico o control aéreo. A Alerta Vecinal nadie lo va a auditar. Exigirme ese nivel de formalidad sería sobre-ingeniería para un proyecto de un semestre.

---

## Antes de entregar

- [x] La descripción del apartado 1 se entiende sin ser del área
- [x] Hay al menos dos tipos de usuario con necesidades distintas
- [x] Identifiqué un conflicto real entre usuarios
- [x] El alcance dice qué queda fuera, no solo qué queda dentro
- [x] Las exclusiones son específicas, no genéricas
- [x] Identifiqué el tipo de sistema y al menos dos atributos de calidad
- [x] Anoté al menos tres reglas de negocio no obvias
- [x] Justifiqué el ciclo de vida contra dos alternativas descartadas
- [ ] El documento está en mi repositorio y se puede leer desde el navegador
- [x] Borré todas las instrucciones en cursiva de la plantilla

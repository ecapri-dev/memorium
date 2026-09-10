# Visión del producto

- **Autor:** Emiliano Cabañas Prieto
- **Fecha de la última versión:** 2026-09-06
- **Repositorio:** https://github.com/ecapri-dev/memorium/

---

## 1. Descripción del sistema

**Nombre del sistema:** Memorium

**Descripción:**

Memorium es una aplicación donde una persona crea capsulas con un beneficiario, con el propocito de que este acceda en determinado tiempo: desde las llaves para acceder a su dinero digital hasta cartas, fotos y documentos personales. Todo eso se guarda cifrado dentro de una cápsula que nadie puede abrir mientras el titular siga vivo.

Para confirmar que el titular falleció, el sistema exige dos señales al mismo tiempo. La primera es que la persona haya dejado de entrar a la aplicación durante el plazo que ella misma configuró. La segunda es que las personas de confianza que nombró lo confirmen. Cuando se cumplen ambas, el sistema todavía espera: avisa al titular por todos los medios que registró y le da un plazo para cancelar, por si hubo un error. Solo si ese plazo vence sin respuesta, la cápsula se abre para los herederos.

Memorium nunca guarda ni mueve el dinero de nadie. Entrega las llaves a quien el titular decidió, y nada más. Y funciona sobre una red pública, de modo que la cápsula siga existiendo aunque la empresa que hizo la aplicación desaparezca.

---

## 2. Problema y usuarios

**El problema:**

Cuando alguien muere con criptomonedas, ese dinero casi siempre se pierde para siempre. No hay un banco al que la familia pueda llegar con un acta de defunción: si nadie tiene la frase de recuperación, los fondos quedan bloqueados y no existe autoridad que pueda desbloquearlos. El dinero sigue ahí, visible para cualquiera, y a la vez fuera del alcance de todos.

Lo mismo pasa con el resto de la vida digital de una persona. Documentos, cuentas, mensajes que hubiera querido dejar y que nadie sabe dónde están.

Y la solución obvia es peor que el problema. Compartir la frase de recuperación en vida significa entregarle a alguien más el control total de tu dinero hoy, confiando en que no lo use antes de tiempo. Quien quiere proteger a su familia termina teniendo que elegir entre dos formas de perder.

**Cómo se resuelve hoy sin el sistema:**

Las personas se las arreglan con soluciones caseras. Escriben la frase de recuperación en un papel y lo esconden en un cajón, una caja fuerte o una caja de seguridad del banco. Se la dicen a un familiar de confianza, o se la dejan en un sobre cerrado con instrucciones. Algunos la incluyen en un testamento ante notario, aunque eso la expone a todo el que intervenga en la sucesión. Y muchos simplemente no hacen nada, porque pensar en el tema incomoda y siempre se puede dejar para después.

**Usuarios del sistema:**

| Tipo de usuario | Qué necesita del sistema | Qué le preocupa |
|---|---|---|
| Titular | Preparar su cápsula y poder cambiarla cuando quiera, con la certeza de que nadie la abrirá mientras viva | Que alguien acceda antes de tiempo; que se abra por error si se va de viaje o pierde el teléfono |
| Heredero | Recibir lo que le corresponde sin trámites imposibles, y saber que hay algo esperándolo | Enterarse tarde o nunca; que el acceso quede bloqueado para siempre por un detalle técnico |
| Guardián | Entender qué se espera de él y poder confirmar el fallecimiento de forma sencilla | Cargar con una responsabilidad que no pidió; que la familia lo presione; que lo culpen si algo sale mal |

**Un conflicto entre usuarios:**

El titular quiere que su cápsula sea imposible de abrir mientras esté vivo. El heredero quiere la certeza de que se va a abrir cuando llegue el momento. Las dos cosas no pueden ser ciertas al mismo tiempo.

Cualquier mecanismo que le dé más certeza al heredero es también una forma de abrir la cápsula antes de tiempo. Y cualquier candado extra que proteja al titular es una forma más de que el heredero se quede sin nada. No es una diferencia de opiniones que se resuelva conversando: es una tensión estructural del sistema, y toda mi decisión de diseño consiste en dónde poner esa raya.

El guardián queda en medio sin haberlo pedido. Si confirma demasiado rápido, puede desbloquear la herencia de alguien que sigue vivo. Si duda o se tarda, deja a la familia esperando algo que ya le corresponde. Y para cuando aparece la duda, el titular ya no está para aclararla.

---

## 3. Alcance

### Dentro del alcance

- Crear una cápsula y guardar dentro llaves, accesos, mensajes, fotos y documentos, todo cifrado.
- Nombrar herederos y definir qué parte de la cápsula recibe cada uno.
- Nombrar guardianes y establecer cuántos de ellos deben confirmar el fallecimiento.
- Registrar las señales de vida del titular dentro del plazo que él mismo configuró.
- Detectar la inactividad y abrir un periodo de gracia, notificando al titular por todos los medios que registró.
- Recibir y contar las confirmaciones de los guardianes.
- Liberar el acceso a los herederos cuando se cumplen las dos condiciones y vence el periodo de gracia.
- Cancelar todo el proceso si el titular da señal de vida antes de que venza el plazo.
- Editar o revocar la cápsula en cualquier momento mientras el titular viva.

### Explícitamente fuera del alcance

- **Guardar o mover los fondos del titular.** Memorium entrega llaves, no dinero.
- **Recuperar el acceso si el titular pierde su propia llave maestra.** Si la pierde, la cápsula se pierde con ella.
- **Sustituir un testamento o validar la herencia ante la ley.**
- **Consultar el registro civil para verificar un acta de defunción oficial.**
- **Soportar otras redes distintas de Solana en la primera versión.**
- **Dar asesoría legal o fiscal sobre los activos heredados.**

**Por qué queda fuera:**

La exclusión que más define al proyecto es la custodia. Memorium no guarda ni mueve el dinero de nadie, y la razón es doble.

Técnicamente, un sistema que custodia fondos se convierte en el único lugar que hay que atacar para robarle a todos sus usuarios de una sola vez. Eso es exactamente el riesgo que este proyecto existe para evitar, así que construirlo así sería contradecirme.

Legalmente, custodiar dinero de terceros es una actividad regulada que un proyecto de un semestre no puede cumplir con seriedad.

Hay una consecuencia bonita de esa decisión: al no tocar los fondos, Memorium puede fallar sin que nadie pierda su dinero por culpa del sistema. Lo peor que puede pasar es que una cápsula no se abra, y eso deja a la familia igual que si Memorium nunca hubiera existido.

---

## 4. Tipo de sistema y restricciones

**Tipo de sistema:** SaaS, con exigencias de sistema crítico.

**Por qué es de ese tipo:**

Memorium se usa por internet y atiende a muchos usuarios a la vez desde un mismo lugar, sin que nadie instale nada. Por su forma de entregarse y de operar es un sistema Web y SaaS, con lo que eso implica de escalabilidad, disponibilidad y aislamiento entre usuarios.

Pero la parte que guarda las condiciones de liberación vive en una red pública, y ahí el código publicado no se corrige como se corrige una página: se queda. Eso le impone exigencias propias de un sistema crítico, porque una falla aquí no incomoda al usuario, le cuesta dinero de forma irreversible o le abre la cápsula antes de tiempo. Por eso el diseño toma prestadas de los sistemas críticos dos cosas: la auditabilidad y la capacidad de degradarse sin causar daño.

**Atributos de calidad que impone:**

| Atributo | Por qué importa en mi caso | Qué pasa si no se cumple |
|---|---|---|
| Seguridad | Una cápsula abierta antes de tiempo no se puede volver a cerrar | Alguien accede al dinero y a la vida privada de una persona que sigue viva |
| Confidencialidad | El sistema guarda llaves privadas, cartas y documentos personales | Una fuga expone el patrimonio y la intimidad de sus usuarios al mismo tiempo |
| Auditabilidad | Cuando la cápsula se abre, el titular ya no está para reclamar si algo salió mal | Nadie puede demostrar si la liberación fue legítima, y la familia se queda sin forma de disputarla |
| Disponibilidad a largo plazo | Una cápsula tiene que seguir existiendo dentro de veinte o treinta años, que es cuando probablemente haga falta | El sistema desaparece y con él justo lo que prometió conservar para siempre |

**Reglas de negocio que ya identifiqué:**

1. Una cápsula solo se libera cuando se cumplen las **dos** condiciones a la vez: que el titular haya dejado de dar señal de vida durante el plazo configurado, y que al menos N de sus M guardianes hayan confirmado el fallecimiento. Con una sola de las dos, no pasa nada.
2. Antes de liberar, el sistema abre un periodo de gracia y notifica al titular por todos los medios que registró. Si el titular responde durante ese periodo, el proceso se cancela por completo y el contador vuelve a cero.
3. Un guardián no puede ser heredero de la misma cápsula. Quien confirma la muerte no puede ser quien se beneficia de esa confirmación.
4. Cualquier edición de la cápsula reinicia el contador de inactividad, porque editarla ya es en sí misma una señal de vida.
5. Si el número de guardianes disponibles baja del mínimo necesario para confirmar, la cápsula entra en estado de alerta y no puede liberarse hasta que el titular nombre reemplazos.
6. La llave que abre la cápsula nunca existe completa en un solo lugar: se guarda repartida en fragmentos, de modo que ni el sistema ni un guardián por su cuenta puedan reconstruirla.

---

## 5. Ciclo de vida elegido

**Modelo elegido:** Modelo V

**Por qué le conviene a este proyecto:**

Mi alcance está cerrado y escrito, con seis exclusiones explícitas, y puedo comprometerme a que no va a cambiar durante el semestre.

El riesgo de este proyecto es de un tipo particular, y es lo que decide el modelo: la probabilidad de falla es baja, pero las consecuencias son graves e irreversibles. Un error en las condiciones de liberación no se arregla en la siguiente versión, porque para entonces la cápsula ya se abrió o ya se perdió.

Y sí hay necesidad de auditoría. Cualquier sistema que maneje llaves de acceso a fondos de terceros necesita que alguien externo verifique formalmente que hace lo que dice hacer, y esa verificación necesita evidencia documentada por cada etapa.

Ese perfil, requisitos estables y verificables, cliente poco disponible, riesgo bajo con consecuencias graves y necesidad de evidencia formal, es exactamente donde el modelo V es la mejor opción. Cada etapa de especificación y diseño queda amarrada a su etapa de verificación correspondiente, en lugar de dejar toda la comprobación para el final.

### Alternativas descartadas

**Alternativa 1:** Ágil

_Por qué la descarté:_ Ágil se sostiene en equivocarse rápido y barato, corrigiendo en la siguiente iteración. Aquí equivocarse no es barato y muchas veces no se puede corregir: si las condiciones de liberación estaban mal, la cápsula ya se abrió antes de tiempo o ya quedó bloqueada para siempre. A eso se suma que no tengo un cliente presente cada semana, que es de donde ágil saca su información.

**Alternativa 2:** Cascada

_Por qué la descarté:_ Era la más tentadora, porque mi alcance está cerrado y cascada le conviene justo a eso. La descarté por un rasgo concreto de mi proyecto: cascada valida hasta el final, y en un sistema donde el error es irreversible y cuesta el dinero de alguien más, dejar toda la verificación para el último momento es el riesgo que no me puedo permitir. El modelo V me conserva el orden de cascada, que es lo que me gustaba, pero le amarra una verificación a cada fase.

---

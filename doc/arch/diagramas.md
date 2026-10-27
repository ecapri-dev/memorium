# Diagramas de arquitectura · Memorium

Ingeniería de Software I (SIS3407) · Unidad 3, diseño de software
Autor: Emiliano Cabañas Prieto

Los diagramas están escritos en Mermaid, así que GitHub los dibuja directamente en esta página y cualquier cambio queda visible en el historial del repositorio. Los tres primeros siguen los niveles 1 a 3 del modelo C4. Los otros cinco usan el mismo sistema para mostrar un flujo, una secuencia, las clases, los estados de la cápsula y las tablas.

## Decisión de arquitectura que siguen los diagramas

Memorium es un monolito modular con cuatro módulos: Cápsulas, Personas, Liberación y Avisos. La regla que se ve en los diagramas es que un módulo no lee las tablas de otro: si necesita un dato ajeno, se lo pregunta al módulo dueño por su interfaz. El porqué de la decisión está en [ADR-001 · Monolito modular](adr-001-monolito-modular.md).

---

## 1. Nivel 1 · Diagrama de contexto

Quién usa Memorium y con qué sistemas externos habla. No aparece ninguna tecnología.

```mermaid
%%{init: {'htmlLabels': false, 'flowchart': {'htmlLabels': false}}}%%
flowchart TD
    titular([Titular])
    guardian([Guardián])
    heredero([Heredero])
    sis[Memorium]
    red[Red pública Solana<br/>condiciones de liberación]
    msj[Servicio de mensajería<br/>correo, SMS y WhatsApp]

    titular -->|arma su cápsula y da señales de vida| sis
    guardian -->|confirma el fallecimiento| sis
    heredero -->|recibe sus elementos| sis
    sis -->|registra condiciones y confirmaciones| red
    sis -->|envía avisos y solicitudes| msj

    style sis fill:#ff5d00,color:#fff
```

La red pública guarda las condiciones de liberación porque ahí no las puede alterar ni el propio operador de Memorium (RNF-SEG-002).

## 2. Nivel 2 · Diagrama de contenedores

Las piezas que se despliegan por separado. El `subgraph` marca la frontera: lo de adentro es Memorium, lo de afuera no.

```mermaid
%%{init: {'htmlLabels': false, 'flowchart': {'htmlLabels': false}}}%%
flowchart TD
    titular([Titular])
    guardian([Guardián])
    heredero([Heredero])

    subgraph sistema[Memorium]
        web[App web<br/>React, cifra en el navegador]
        api[API<br/>Node.js, monolito modular]
        db[(Base de datos<br/>PostgreSQL)]
        obj[(Almacén de cápsulas cifradas<br/>almacenamiento de objetos)]
    end

    red[Red pública Solana]
    msj[Servicio de mensajería]

    titular -->|usa| web
    guardian -->|usa| web
    heredero -->|usa| web
    web -->|HTTPS/JSON| api
    api -->|lee y escribe| db
    api -->|guarda y entrega contenido cifrado| obj
    api -->|RPC| red
    api -->|API del proveedor| msj

    style api fill:#ff5d00,color:#fff
```

El contenido se cifra en el navegador antes de subirlo, así que el almacén solo guarda datos ilegibles (RNF-SEG-001). Son cuatro contenedores porque son las cuatro piezas que se despliegan por separado. El reloj que revisa los plazos no aparece aquí: corre dentro de la API, así que es un componente y está un nivel más adentro.

## 3. Nivel 3 · Diagrama de componentes

Lo que hay dentro de la API. Liberación es el módulo central, por eso va resaltado.

```mermaid
%%{init: {'htmlLabels': false, 'flowchart': {'htmlLabels': false}}}%%
flowchart TD
    web[App web]

    subgraph api[API de Memorium]
        cap[Cápsulas<br/>elementos y cifrado]
        per[Personas<br/>herederos, guardianes y verificación]
        lib[Liberación<br/>plazos, confirmaciones y gracia]
        avi[Avisos<br/>interfaz de envío]
        reloj[Reloj<br/>tarea programada diaria]
    end

    db[(Base de datos)]
    red[Red pública Solana]
    msj[Servicio de mensajería]

    web --> cap
    web --> per
    web --> lib
    reloj -->|plazo vencido| lib
    cap -->|quién hereda qué| per
    lib -->|es guardián y si hereda| per
    lib -->|liberar cápsula| cap
    lib -->|cambio de estado| avi
    lib -->|registra confirmaciones| red
    avi --> msj
    cap --> db
    per --> db
    lib --> db

    style lib fill:#ff5d00,color:#fff
```

Tres detalles. El Reloj es el actor "Tiempo" de los casos de uso: cada día revisa qué plazos vencieron y se lo avisa a Liberación. Liberación le pregunta a Personas si alguien es guardián y si hereda, en lugar de leer sus tablas, que es la regla del ADR-001. Y Avisos está puesto como interfaz y no como un proveedor concreto: es el patrón Adapter, y además recibe los cambios de estado que anuncia Liberación, que es el patrón Observer.

---

## 4. Diagrama de flujo con decisiones

Qué pasa cuando un guardián intenta confirmar el fallecimiento (CU-05).

```mermaid
%%{init: {'htmlLabels': false, 'flowchart': {'htmlLabels': false}}}%%
flowchart TD
    ini([Inicio])
    vida{El titular dio señal de vida?}
    bloq{Está bloqueado por intentos fallidos?}
    ver{Pasó la verificación de identidad?}
    tres{Es su tercer intento fallido?}
    dup{Ya había confirmado antes?}
    reg[Registrar la confirmación]
    min{Se alcanzó el mínimo N con al menos un guardián que no hereda?}
    gracia[Abrir periodo de gracia de 14 días]
    espera[Esperar más confirmaciones]
    cancel[Avisar que el proceso se canceló]
    aviso[Avisar a qué hora termina el bloqueo]
    bloquear[Bloquear 24 horas]
    reint[Pedir que lo intente de nuevo]
    nocuenta[No se cuenta dos veces]
    fin([Fin])

    ini --> vida
    vida -->|Sí| cancel
    vida -->|No| bloq
    bloq -->|Sí| aviso
    bloq -->|No| ver
    ver -->|No| tres
    tres -->|Sí| bloquear
    tres -->|No| reint
    ver -->|Sí| dup
    dup -->|Sí| nocuenta
    dup -->|No| reg
    reg --> min
    min -->|Sí| gracia
    min -->|No| espera
    cancel --> fin
    aviso --> fin
    bloquear --> fin
    reint --> fin
    nocuenta --> fin
    gracia --> fin
    espera --> fin

    style reg fill:#ff5d00,color:#fff
```

Cada rombo sale de un requisito: RF-012 (señal de vida), RF-023 (bloqueo de 24 horas), RF-022 (verificación), RF-009 (no contar dos veces) y RF-005 con RF-010 (mínimo con un guardián que no hereda).

## 5. Diagrama de secuencia

La confirmación que completa el mínimo y abre el periodo de gracia, de punta a punta.

```mermaid
sequenceDiagram
    actor G as Guardián
    participant Web as App web
    participant Lib as Liberación
    participant Per as Personas
    participant Red as Red Solana
    participant Avi as Avisos
    actor T as Titular

    G->>Web: Confirma el fallecimiento
    Web->>Per: verificarIdentidad(guardián, código)
    Per-->>Web: Identidad verificada
    Web->>Lib: registrarConfirmacion(cápsula, guardián)
    Lib->>Per: Es guardián de esta cápsula? Hereda?
    Per-->>Lib: Sí es guardián, no hereda
    Lib->>Red: Registra la confirmación N
    Red-->>Lib: Registrada
    Lib->>Lib: Abre el periodo de gracia de 14 días
    Lib->>Avi: Cambio de estado: periodo de gracia
    Avi->>T: Aviso por cada medio registrado
    Lib-->>Web: Confirmación registrada, N de N
    Web-->>G: Muestra que se abrió el periodo de gracia
```

Las flechas sólidas son llamadas y las punteadas son respuestas. Liberación no decide sola si alguien es guardián: se lo pregunta a Personas.

## 6. Diagrama de clases

Las clases del dominio. Elemento es abstracta y cada tipo sabe presentarse solo, que es el patrón Strategy que se identificó en la actividad anterior.

```mermaid
classDiagram
    class Titular {
        -nombre: string
        -mediosDeContacto: List~string~
        +darSenalDeVida() void
    }
    class Capsula {
        -estado: EstadoCapsula
        -plazoInactividadDias: int
        -minimoConfirmaciones: int
        -ultimaSenalDeVida: Date
        +registrarConfirmacion(guardian) void
        +abrirPeriodoDeGracia() void
        +cancelarLiberacion() void
        +liberar() void
    }
    class Elemento {
        <<abstract>>
        -contenidoCifrado: bytes
        +presentar() string
    }
    class Llave {
        +presentar() string
    }
    class Mensaje {
        +presentar() string
    }
    class Foto {
        +presentar() string
    }
    class Documento {
        +presentar() string
    }
    class Heredero {
        -nombre: string
        -mediosDeContacto: List~string~
    }
    class Guardian {
        -nombre: string
        -telefono: string
        -intentosFallidos: int
        -bloqueadoHasta: Date
        +verificarIdentidad(codigo) bool
    }
    class Confirmacion {
        -fecha: Date
    }

    Titular "1" --> "*" Capsula : crea
    Capsula "1" --> "1..*" Elemento : contiene
    Capsula "1" --> "*" Heredero : nombra
    Capsula "1" --> "*" Guardian : nombra
    Capsula "1" --> "*" Confirmacion : acumula
    Guardian "1" --> "0..1" Confirmacion : emite
    Heredero "1" --> "*" Elemento : recibe
    Elemento <|-- Llave
    Elemento <|-- Mensaje
    Elemento <|-- Foto
    Elemento <|-- Documento
```

El menos es privado y el más es público. `1..*` en Elemento refleja RF-001: una cápsula no se guarda sin al menos un elemento. `0..1` en Confirmación refleja RF-009: cada guardián cuenta una sola vez.

## 7. Diagrama de estados

El ciclo de vida de una cápsula y el evento que la mueve de un estado a otro.

```mermaid
stateDiagram-v2
    state "Esperando confirmaciones" as Esperando
    state "Periodo de gracia" as Gracia
    state "En alerta" as Alerta

    [*] --> Activa: el titular la guarda
    Activa --> Esperando: vence el plazo de inactividad
    Esperando --> Gracia: llega la confirmación N
    Esperando --> Activa: el titular da señal de vida
    Gracia --> Activa: el titular da señal de vida
    Gracia --> Liberada: pasan 14 días sin respuesta
    Activa --> Alerta: quedan menos guardianes que N
    Alerta --> Activa: el titular nombra reemplazos
    Activa --> Revocada: el titular la revoca
    Liberada --> [*]
    Revocada --> [*]
```

Los nombres de los estados son los mismos de los criterios de aceptación (RF-010, RF-012, RF-015 y RF-017). Liberada y Revocada son finales: de ahí no se regresa, que es justo por lo que RNF-SEG-003 exige cero liberaciones indebidas.

## 8. Diagrama entidad-relación

Las tablas de la base de datos y cómo se relacionan.

```mermaid
erDiagram
    TITULAR ||--o{ CAPSULA : crea
    CAPSULA ||--|{ ELEMENTO : contiene
    CAPSULA ||--|{ HEREDERO : nombra
    CAPSULA ||--|{ GUARDIAN : nombra
    HEREDERO ||--o{ ELEMENTO : recibe
    HEREDERO |o--o| GUARDIAN : "también puede ser"
    GUARDIAN ||--o| CONFIRMACION : emite
    CAPSULA ||--o{ CONFIRMACION : acumula
    CAPSULA ||--o{ EVENTO : registra

    TITULAR {
        int id PK
        string nombre
        string correo
        string telefono
    }
    CAPSULA {
        int id PK
        int titular_id FK
        string estado
        int plazo_inactividad_dias
        int minimo_confirmaciones
        date ultima_senal_de_vida
    }
    ELEMENTO {
        int id PK
        int capsula_id FK
        int heredero_id FK
        string tipo "llave, mensaje, foto o documento"
        string ruta_cifrada
    }
    HEREDERO {
        int id PK
        int capsula_id FK
        string nombre
        string contacto
    }
    GUARDIAN {
        int id PK
        int capsula_id FK
        int heredero_id FK "nulo si no hereda"
        string nombre
        string telefono
        date bloqueado_hasta
    }
    CONFIRMACION {
        int id PK
        int capsula_id FK
        int guardian_id FK
        date fecha
    }
    EVENTO {
        int id PK
        int capsula_id FK
        string tipo
        date fecha
        string huella
    }
```

`heredero_id` en GUARDIAN es lo que permite revisar RF-005: si es nulo, ese guardián no hereda nada. La columna `huella` de EVENTO encadena cada evento con el anterior para que una alteración quede en evidencia (RNF-SEG-004).

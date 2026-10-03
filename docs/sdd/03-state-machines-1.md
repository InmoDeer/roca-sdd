# 03 — Máquinas de estado

**Estado:** DRAFT v0.5  
**Última actualización:** 2026-10-02

Etiquetas usadas en este documento: **[Decidido]** confirmado por el autor del SDD; **[Propuesta]** sugerido y pendiente de confirmación; **[TBD]** sin definir.

## 1. Propósito

Este documento define las máquinas de estado conceptuales de ROCA.

El objetivo es separar:

- estado actual del proceso;
- lado del mercado y objetivo comercial;
- operación inmobiliaria;
- Requirement vigente;
- resultado del proceso;
- motivo de cierre;
- eventos que provocan transiciones (ver `04-event-model.md`).

Una modificación de Requirement no debe confundirse con una transición de Opportunity.

## 2. Opportunity

### 2.1 Concepto

Una Opportunity representa la continuidad de **un objetivo comercial principal de una persona, en un lado del mercado**.

**Regla [Decidido]:** una Opportunity tiene un solo objetivo principal. Si una persona desarrolla otro objetivo independiente, se crea otra Opportunity. No se agregan objetivos paralelos dentro de la misma.

Atributos conceptuales:

| Atributo | Descripción |
|---|---|
| `side` | Lado del mercado: `demand` u `oferta` (supply). Ver 2.2 |
| `objective` | Propósito que persigue la persona. Ver 2.4 |
| `status` | Etapa del proceso. Ver 2.6 |
| `outcome` / `reason_code` | Solo al cerrar. Ver 2.8 |
| inmueble (solo `supply`) | Exactamente uno. Ver 2.5 |

La operación (compra, alquiler, venta) **no** es un atributo de Opportunity: vive en Requirement (2.3).

### 2.2 Lado del mercado (`side`) [Decidido]

- **`demand`:** la persona busca un inmueble (comprar, alquilar o indiferente).
- **`supply`:** la persona quiere comercializar un inmueble que posee (vender, alquilar o indiferente).
- Otros procesos no cubiertos: [TBD].

Este atributo **reemplaza** el campo `type` de la v0.2 (Demanda, Servicio de venta, Servicio de alquiler, Captación). La captación deja de ser un tipo y pasa a ser una **etapa** de las oportunidades `supply` (2.6).

Terminología para evitar ambigüedad: `supply` (lado oferta) se distingue de la **oferta económica** que se hace durante una negociación (`offer`, `counteroffer`; ver `04`).

Relación con las personas: el propietario es el **cliente** (quien contrata y paga los servicios) y el interesado es **cliente potencial**. Una misma persona puede tener oportunidades de ambos lados (P4).

### 2.3 Operación [Decidido]

La operación pertenece al Requirement. Valores: `sale`, `rent`, `either` (indiferente).

- Si a la persona le es indiferente vender o alquilar (o comprar o alquilar), es **una sola Opportunity** con operación `either`.
- Un cambio de operación dentro del mismo objetivo es una nueva versión de Requirement, no una nueva Opportunity.
- Al cerrar, el resultado debe registrar la operación que realmente ocurrió.

### 2.4 Objetivo (`objective`)

El objetivo es el propósito que persigue la persona (para qué quiere el inmueble o por qué lo comercializa). Es lo que determina si una Opportunity continúa o se reemplaza.

Valores candidatos [TBD, taxonomía pendiente]: vivir/mudarse, inversión, uso comercial/negocio, comercializar un inmueble propio (`supply`).

**Regla [Decidido]:** una Opportunity continúa mientras se mantengan su lado y su objetivo, aunque cambien zona, presupuesto, características o incluso la operación.

### 2.5 Inmuebles por Opportunity [Decidido]

- **`supply`:** una Opportunity por inmueble. Un propietario con dos inmuebles tiene dos Opportunities `supply`.
- **`demand`:** relación N:M con inmuebles considerados (modelo `OpportunityProperty`, [TBD] en `02`).

### 2.6 Estados

Las etiquetas **[Propuesta]** señalan cambios respecto a la v0.2 que requieren confirmación.

**Demanda:**

`interesado → calificación [Propuesta] → seguimiento → visita → post_visita → negociación [Propuesta de nombre] → cerrada`

**Oferta (`supply`):**

`contactado → propuesta_tasación → seguimiento → captada → en_comercialización [Propuesta de nombre] → cerrada`

Notas:

- `negociación` recoge que las ofertas, contraofertas y "considerando" se registran como actividades dentro de esta etapa, con su subtipo (ver `04`). El nombre del estado es [TBD].
- `calificación` recoge el filtrado de curiosos, encaje, presupuesto y plazo. Es [Propuesta]; puede resolverse como estado o como parte de `interesado`.
- `captada` [Decidido]: el propietario accedió a los servicios (acuerdo de servicios, ver `02`). Es un hito intermedio, no el final: la Opportunity solo se cierra al concretar la operación (vender o alquilar) o al retirar el inmueble. Captar sin cerrar no genera resultado. El nombre `en_comercialización` es [Propuesta].
- **La captación puede ocurrir antes, durante o después de una visita [Decidido].** Una visita puede ser para conversar (el propietario puede decir que lo pensará) o para preparar material audiovisual y datos, normalmente tras captar. Por eso las visitas de `supply` no son estados de la Opportunity: se registran como Visits (§6). `visita` y `post_visita` quedan solo en el flujo de `demand` [Propuesta].
- Estados terminales: ver 2.8. `cerrada` reemplaza a `cerrado`, `captacion_cerrado` y `descartado`.

### 2.7 Significado de los estados

| Estado | Significado |
|---|---|
| interesado / contactado | Existe una intención comercial identificada; la oportunidad puede trabajarse |
| calificación | Se determina encaje, intención, plazo y presupuesto |
| propuesta_tasación | (`supply`) Se prepara la propuesta de servicio, valoración o condiciones |
| seguimiento | Oportunidad activa que requiere acciones posteriores; no implica bajo interés |
| visita | La visita es el siguiente hito o está siendo gestionada. La fecha final la decide el agente |
| post_visita | La visita ocurrió y falta registrar o procesar su resultado |
| negociación | Hay ofertas, contraofertas o condiciones en discusión |
| captada | (`supply`) El propietario accedió a los servicios; existe acuerdo de servicios con comisión pactada |
| en_comercialización | (`supply`) El inmueble está en proceso de venta o alquiler |
| cerrada | Terminal. Lleva `outcome` y `reason_code` |

### 2.8 Cierre: resultado y motivo [Propuesta de estructura]

Se unifica el cierre en un solo estado terminal `cerrada` más:

- `outcome`: `won`, `lost`, `superseded` [TBD, taxonomía];
- `reason_code`;
- fecha;
- actor o evento que produjo el cierre;
- operación real concluida y referencia a su registro (`Operation`), cuando aplique.

`superseded` representa un cambio de objetivo: la Opportunity se reemplaza por otra, no se pierde ni se gana.

Candidatos de `reason_code` (taxonomía definitiva [TBD]): objetivo cambiado, necesidad desaparecida, operación concluida, encontró otra alternativa, no hubo inmueble adecuado, no acepta condiciones, no aceptó trabajar con nosotros, acuerdo no renovado, operación concluida por otro agente o por el propietario, dejó de responder, descartado por criterios, otro.

Si la operación la concluye el propietario u otro agente: en sin exclusividad o semi-exclusividad no hay comisión y la Opportunity se cierra `lost`; con exclusividad vigente sí hay comisión (por defecto 100%) y la Opportunity se cierra `won` [Propuesta] (`02`).

Reapertura: ver 5.

### 2.9 Cambio de objetivo y objetivos simultáneos [Decidido]

**Cambio de objetivo:** se cierra la Opportunity anterior (`superseded`, motivo objetivo cambiado) y se crea una nueva. El historial de la anterior no se transforma ni se elimina.

**Objetivos simultáneos:** cada Opportunity conserva su Requirement, estado, eventos, visitas y resultado.

**Ejemplo de una persona con tres Opportunities abiertas:**

| # | Lado | Contenido | Nota |
|---|---|---|---|
| A | demand | Alquilar un departamento de 3 dormitorios en San Miguel, S/ 2,200–2,400 | Objetivo: vivir |
| B | demand | Comprar un depósito/almacén de 300–350 m² en el Centro de Lima, presupuesto S/ 560,000 | Objetivo: [TBD] (uso propio o inversión) |
| C | supply | Vender su casa en Chorrillos | Un inmueble, una Opportunity |

### 2.10 Vencimiento del acuerdo de servicios [Propuesta]

Cuando un acuerdo de servicios se acerca a su vencimiento, ROCA avisa al agente (anticipación por defecto de 30 días, sin ninguna acción automática). El agente decide:

- **Seguir:** renueva el acuerdo, con nueva vigencia y, si cambian, nuevas condiciones. La Opportunity no cambia.
- **Archivar:** deja de comercializar el inmueble. La Opportunity `supply` se cierra (`lost`, motivo "acuerdo no renovado") y puede reabrirse si el propietario retoma el objetivo (§5).

Si llega la fecha sin decisión, el acuerdo pasa a vencido y deja de dar exclusividad, pero la Opportunity no cambia de estado por sí sola. ROCA sigue recordando la decisión pendiente.

## 3. Requirement y Opportunity

**Requirement cambia cuando cambian las condiciones del mismo objetivo.**  
**Opportunity cambia cuando cambia el objetivo o aparece un objetivo independiente.**

| Situación | Opportunity | Requirement |
|---|---|---|
| Cambia presupuesto | misma | nueva versión |
| Cambia distrito | misma | nueva versión |
| Cambia dormitorios | misma | nueva versión |
| San Miguel → Lince y presupuesto sube a S/ 2,500 (mismo objetivo: vivir) | misma | nueva versión |
| Departamento → oficina manteniendo objetivo de inversión | misma | nueva versión |
| Alquilar → comprar para vivir (mismo objetivo) | misma | nueva versión (operación) |
| Quiere vender o alquilar su inmueble, le es indiferente | una sola | operación `either` |
| "Ya no quiero mudarme; ahora alquilar en Jesús María para Airbnb, hasta S/ 2,300 amoblado" | cierra la anterior (`superseded`) y se crea nueva | nuevo |
| Mudarse + además trasladar negocio | nueva adicional | nuevo |
| Mudarse + además comprar almacén | nueva adicional | nuevo |
| Propietario con dos inmuebles | dos Opportunities `supply` | una por inmueble |
| Persona demanda pasa a vender su casa | nueva Opportunity `supply` | nuevo |
| Retoma un objetivo antes cerrado (volver a querer mudarse tras haber comprado un local) | se reabre la original; la otra sigue abierta | nueva versión si cambian las condiciones |

La regla exacta para clasificar objetivos queda [TBD] hasta definir su taxonomía. Esta tabla debe convertirse en reglas verificables.

## 4. Transiciones preliminares

### Demanda

- created → interesado
- interesado → calificación [Propuesta]
- calificación → seguimiento
- seguimiento → visita
- visita → post_visita
- visita → seguimiento (visita cancelada o no realizada)
- post_visita → seguimiento
- post_visita → negociación
- negociación → seguimiento
- cualquier estado abierto → cerrada

### Oferta (`supply`)

- created → contactado
- contactado → propuesta_tasación
- propuesta_tasación → seguimiento (el propietario lo piensa)
- contactado, propuesta_tasación o seguimiento → captada (el propietario acepta, con o sin visita previa)
- captada → en_comercialización

Las visitas de `supply` no generan transiciones: ocurren antes, durante o después de la captación (§6).
- cualquier estado abierto → cerrada

Las condiciones de entrada/salida y los actores permitidos por transición deben formalizarse.

### Transiciones que requieren cuidado

- **Cambio de Requirement:** no cambia necesariamente el estado. Genera historial/evento.
- **Cambio de objetivo:** no convierte el Requirement existente; cierra y crea.
- **Objetivo paralelo:** no modifica la Opportunity existente.

## 5. Reapertura [Decidido]

Una Opportunity `cerrada` puede **reabrirse**. Se mantiene una sola Opportunity para conservar todo el historial en una única entidad y poder analizar por qué se reabrió, cuántas veces y con qué resultado.

Reglas:

- Toda reapertura genera el evento `opportunity.reopened` con motivo (`reopen_reason`) y disparador.
- Reabrir **no borra** los cierres anteriores: los eventos `opportunity.closed` previos se conservan. Mientras la Opportunity está abierta, `outcome` y `reason_code` vigentes quedan vacíos; el `outcome` final es el del último cierre.
- Se lleva un conteo de ciclos de cierre y reapertura (`04`).
- Estado al que vuelve [Propuesta]: `seguimiento` en `demand`; `en_comercialización` en `supply` cuando el inmueble ya estaba captado (el acuerdo de servicios debe seguir vigente; si venció, se renueva).

**Cuándo se reabre [Decidido]:** solo cuando la persona retoma el mismo objetivo (mismo lado y objetivo) de una Opportunity cerrada. Si el objetivo es nuevo, se crea otra Opportunity. Motivo registrado: `objective_resumed`.

Esto aplica también tras `superseded`. Ejemplo: la persona quería mudarse a una casa más grande (A) y decide mejor ahorrar y comprar un local: se cierra A como `superseded` y se abre B. Más adelante, con el local alquilado y pagándose solo, vuelve a querer mudarse: se reabre A y B sigue abierta.

Al reabrir, el último Requirement de la Opportunity sigue siendo el vigente, pero debe confirmarse con la persona porque las condiciones pueden haber cambiado [Propuesta]. Cualquier cambio genera una nueva versión.

**Quién reabre [Decidido, etapa inicial]:** ROCA propone y el agente aprueba. El agente también puede reabrir directamente. ROCA no reabre por sí sola. Qué reaperturas pasarán a ser automáticas se decidirá más adelante con datos reales (ver `04` §9).

**Reapertura tras `won` [Decidido]:** una Opportunity cerrada con éxito puede reabrirse en dos casos:

- la operación se cae (`operation_fell_through`);
- en alquiler, termina el contrato (`contract_ended`).

**Fin de contrato de alquiler [Decidido en lo esencial]:** ROCA debe anticiparse al vencimiento. Antes de que finalice el contrato, contacta, o avisa al agente para que contacte, tanto al propietario como al inquilino (seguimientos mixtos, con aprobación del agente). Resultados posibles:

- **El inquilino se va:** se reabre la Opportunity `demand` del inquilino para mostrarle opciones, y la Opportunity `supply` del propietario para volver a promocionar su inmueble.
- **El inquilino renueva** [Propuesta]: no se reabre nada; se registra la renovación y se actualizan fechas y monto.

Para esto, el cierre de un alquiler debe conservar las fechas del contrato en un registro de la operación concluida (`Operation`, [Propuesta], ver `02`). Anticipación: por defecto **30 días** antes del fin (el aviso de un mes que suelen fijar los contratos), guardada por contrato (`notice_period_days`) porque el contrato real puede establecer otro plazo [Propuesta de configuración].

**Límites [Decidido]:** no hay límite de reaperturas ni plazo máximo desde el cierre. El conteo de ciclos se conserva para análisis.

La alternativa de crear una nueva Opportunity enlazada a la anterior fue descartada: partiría el historial en dos entidades y dificultaría el análisis.

## 6. Visit

Una Visit es una visita a **un inmueble** en el marco de una Opportunity. Tiene su propia máquina de estado y no reutiliza la de Opportunity.

### 6.1 Atributos

| Atributo | Descripción |
|---|---|
| `opportunity_id` | Obligatorio (invariante 14 de `02`) |
| `property_id` | Normalmente obligatorio [Propuesta]. En visitas `capture` y `production` es el inmueble de la Opportunity `supply` |
| `kind` | `capture` (conversar con el propietario, valorar, proponer), `production` (preparar material audiovisual y levantar datos, normalmente tras captar) o `showing` (mostrar un inmueble a la persona) |
| `tour_id` | Opcional. Agrupa visitas hechas en un mismo recorrido [Propuesta] |
| participantes | Persona interesada, propietario u ocupante que da acceso, agente |
| modalidad | Presencial o virtual [TBD] |

**Recorridos [Propuesta]:** si el agente muestra tres inmuebles en una salida, se registran tres Visits con el mismo `tour_id`. Así el resultado queda por inmueble, que es lo que alimenta `OpportunityProperty`.

### 6.2 Estados

`por_coordinar → programada → confirmada [Propuesta] → realizada`

Estados alternativos: `no_realizada` (nadie se presentó o no se pudo hacer) y `cancelada` (desde cualquier estado abierto).

| Estado | Significado |
|---|---|
| por_coordinar | Se quiere visitar. ROCA recopila rangos de disponibilidad de la persona y escala |
| programada | El agente coordinó con el propietario u ocupante y decidió fecha y hora |
| confirmada | La persona confirmó asistencia (permite medir ausencias) [Propuesta] |
| realizada | La visita ocurrió |
| no_realizada | No ocurrió, con motivo y quién faltó |
| cancelada | Se canceló, con motivo y quién canceló |

**Reglas [Decidido]:** ROCA recopila disponibilidad y no fija la hora; el agente coordina con el propietario y decide la fecha. Reprogramar no es un estado nuevo: se registra el evento de reprogramación y se conserva el conteo. Qué acciones de este flujo (por ejemplo, recordatorios o confirmaciones) serán automáticas queda [TBD], con aprobación primero.

### 6.3 Resultado de la visita (P7)

El resultado es independiente del estado y del cierre de la Opportunity. Tras `realizada`, el resultado está `pendiente` hasta que el agente registre su feedback libre (`registrado`). ROCA lo estructura y el agente lo confirma (`04`).

Campos candidatos [TBD, taxonomía pendiente]:

- Visita `showing`: nivel de interés, siguiente paso (segunda visita, oferta, lo pensará, descartado), objeciones (precio, zona, estado, tamaño), fecha prometida de respuesta.
- Visita `capture`: respuesta del propietario (acepta, lo pensará con fecha prometida de respuesta, rechaza), expectativa de precio, ajustes pedidos, información faltante.
- Visita `production`: material audiovisual producido, datos levantados, información faltante.

### 6.4 Relación con el estado de la Opportunity

**`demand`:**

- Una Opportunity en `visita` tiene al menos una Visit abierta (`por_coordinar`, `programada` o `confirmada`). Puede haber varias abiertas a la vez [Propuesta].
- Cuando una Visit pasa a `realizada` con resultado pendiente, la Opportunity pasa a `post_visita`.
- Al registrarse el resultado, la Opportunity avanza según el siguiente paso: `seguimiento` o `negociación`.
- Si una Visit se cancela o no se realiza y no queda otra abierta, la Opportunity vuelve a `seguimiento`.

**`supply` [Propuesta]:** las visitas no son estados de la Opportunity. Pueden ocurrir antes de captar (para conversar), durante, o después (para preparar material y datos). Tras una visita `capture`, el propietario puede aceptar (la Opportunity pasa a `captada`), pensarlo (queda en `seguimiento` con la fecha prometida de respuesta, que genera un seguimiento) o rechazar (`cerrada`).

**Ambos lados:** programar una visita genera una entrada en la agenda del agente (ver `05-agenda.md`).

## 7. OpportunityProperty

Vínculo entre una Opportunity `demand` y un inmueble considerado. Conserva la historia de cómo evoluciona esa relación (`02` §8). No aplica a `supply`, que ya tiene exactamente un inmueble.

### 7.1 Atributos [Propuesta]

| Atributo | Descripción |
|---|---|
| `opportunity_id`, `property_id` | Pareja única |
| `origin` | `inbound_inquiry` (la persona consultó por ese anuncio), `recommended_by_roca`, `recommended_by_agent`, `requested_by_person` |
| `status` | Ver 7.2 |
| `requirement_version_id` | Versión de Requirement contra la que se evaluó el encaje |
| `match_snapshot` | Campos del Requirement cumplidos e incumplidos y su flexibilidad |
| `reason_code`, `requirement_field` | En rechazos: motivo y campo del Requirement implicado |

### 7.2 Estados [Propuesta]

`candidate → shared → interested → shortlisted → in_negotiation → selected`

Estados de salida: `rejected` y `unavailable`.

| Estado | Significado |
|---|---|
| candidate | ROCA o el agente identificaron el inmueble como compatible, sin mostrarlo aún |
| shared | Se envió o mostró a la persona |
| interested | La persona expresó interés |
| shortlisted | Está en su lista corta |
| in_negotiation | Hay ofertas o contraofertas sobre este inmueble |
| selected | Es el inmueble con el que se concretó la operación |
| rejected | La persona lo descartó, con motivo |
| unavailable | Dejó de estar disponible sin que la persona lo rechazara (por ejemplo, se alquiló a otro) |

Notas:

- **Se puede entrar en cualquier punto.** Un lead que consulta por un anuncio concreto entra directamente en `shared` o `interested` (`origin = inbound_inquiry`).
- **La visita no es un estado.** Se deriva de las Visits vinculadas (cantidad y último resultado), para no duplicar el dato (P12).
- **`rejected` es reversible.** Puede volver a `interested` o `shortlisted` mediante un evento, por ejemplo tras una baja de precio.
- **Motivos candidatos** [TBD]: precio, zona, características, estado del inmueble, tamaño, plazo o disponibilidad, condiciones del propietario, otro. `requirement_field` apunta al criterio del Requirement afectado (por ejemplo, precio → presupuesto).
- **Un solo `selected` por operación.** Al concluir una operación sobre un inmueble, los vínculos de otras Opportunities abiertas con ese inmueble pasan a `unavailable` [Propuesta]. Los mensajes a esas personas requieren aprobación del agente.
- **Efecto sobre la Opportunity:** el estado del vínculo no determina el estado de la Opportunity (invariante 12). Excepción [Propuesta]: una Opportunity en `negociación` tiene al menos un vínculo en `in_negotiation`.
- **Reactivación:** un cambio de precio o de disponibilidad de un inmueble debe evaluarse contra los vínculos `rejected` por precio o disponibilidad y contra los Requirements vigentes de Opportunities abiertas, y generar propuestas de reactivación (`04`).
- **Cooperación entre agentes:** vínculos entre Opportunities e inmuebles de distintos agentes: [TBD].

## 8. Reglas de integridad

1. Una Opportunity tiene un único lado y un único objetivo principal.
2. Una Opportunity `supply` tiene exactamente un inmueble.
3. La operación inmobiliaria pertenece al Requirement, no a la Opportunity.
4. La operación `either` no crea dos Opportunities.
5. Una Opportunity `cerrada` solo vuelve a un estado activo mediante una reapertura registrada (5), que conserva los cierres anteriores en el historial.
6. Crear una nueva versión de Requirement no crea automáticamente una nueva Opportunity.
7. Crear una nueva Opportunity no borra Requirements ni eventos de otras.
8. Una persona puede tener múltiples Opportunities activas simultáneas, en ambos lados.
9. Todo cierre conserva `outcome` y `reason_code` cuando el motivo es conocido.
10. Los eventos deben permitir reconstruir por qué una Opportunity cambió de estado.
11. Una visita no equivale a un cierre.
12. Un inmueble recomendado o visitado no determina por sí solo el estado de la Opportunity.
13. ROCA no infiere un cambio de objetivo solo por cambios en criterios, salvo declaración de la persona o regla explícita.
14. Una oferta económica no cierra la Opportunity por sí sola; el cierre es un evento aparte.
15. Reabrir una Opportunity no elimina ni modifica eventos de cierre anteriores.
16. Una Opportunity solo se reabre si la persona retoma el mismo lado y objetivo; un objetivo nuevo crea otra Opportunity.
17. En la etapa inicial, ROCA propone la reapertura y el agente la aprueba.
18. Una Opportunity `won` solo se reabre si la operación se cae o, en alquiler, si termina el contrato.
19. Una Visit pertenece a una Opportunity y, normalmente, a un inmueble.
20. ROCA no fija la fecha y hora de una visita; la decide el agente.
21. Una Opportunity `demand` en `visita` tiene al menos una Visit abierta. En `supply`, las visitas no determinan el estado de la Opportunity.
22. El resultado de una Visit no cierra la Opportunity por sí solo.
23. Existe como máximo un vínculo `OpportunityProperty` por Opportunity e inmueble.
24. Un vínculo `rejected` conserva su motivo cuando es conocido.
25. Una Opportunity `supply` puede pasar a `captada` con o sin visita previa.
26. Una Opportunity `supply` en `captada` o posterior tiene un acuerdo de servicios, vigente o pendiente de decisión por vencimiento. [Propuesta]

## 9. Pendientes

- Taxonomía de objetivos.
- Confirmación de `calificación` y `negociación` como estados, y del nombre de `en_comercialización`.
- Confirmación de que las Opportunities `supply` no tienen estados de visita.
- Confirmación del tratamiento del vencimiento del acuerdo de servicios (§2.10).
- Taxonomías de `outcome` y `reason_code`.
- Estado de destino definitivo y qué reaperturas serán automáticas (5).
- Modelo de `Operation` (contrato, fechas, partes).
- Condiciones, actores y automatizaciones permitidas por transición.
- Taxonomía de resultados de Visit; confirmación de `confirmada`, `tour_id` y modalidad.
- Máquina de estados de Property.
- Confirmación de los estados de `OpportunityProperty` y su taxonomía de motivos de rechazo.
- Confirmación del modelo de agenda (`05-agenda.md`).
- Confirmación de la máquina de estados de Conversation (`06-conversations.md`).

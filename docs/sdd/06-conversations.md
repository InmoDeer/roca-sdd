# 06 — Conversaciones

**Estado:** DRAFT v0.1  
**Última actualización:** 2026-10-03

Etiquetas: **[Decidido]** confirmado por el autor del SDD; **[Propuesta]** sugerido y pendiente de confirmación; **[TBD]** sin definir.

## 1. Propósito

Conversation es el registro de lo que ocurre en los canales: por ahí entran los leads y las respuestas de propietarios, y de ahí salen los tiempos de respuesta. Este documento define qué es una conversación, cómo se clasifica, quién la atiende y cómo se relaciona con las oportunidades.

## 2. Decisiones

- **Un hilo continuo por persona y canal, enlazado a varias oportunidades** [Decidido]. Un mismo chat de WhatsApp puede tratar varios objetivos de la misma persona.
- **La primera respuesta a un lead nuevo la da ROCA sola**, con datos registrados del inmueble [Decidido]. Es una excepción explícita al criterio de aprobación primero (`04` §9).
- **Los mensajes que no son leads** (spam, conocidos, otros agentes): ROCA propone clasificarlos y el agente confirma [Decidido].
- **Un agente externo puede originar una Opportunity** si contacta con un cliente por un inmueble del autor [Decidido]. Ver sección 8.

## 3. Entidades

### 3.1 Conversation

Hilo continuo de comunicación con una persona a través de un canal (`02`). Puede existir antes de cualquier Opportunity.

| Atributo | Descripción |
|---|---|
| `person_id` | Persona con la que se conversa |
| `channel_id` | Referencia al registro de canales (P13) |
| `external_ref` | Identificador del hilo en el canal, para deduplicar |
| `origin_direction` | `inbound` (la persona escribió primero) u `outbound` (ROCA o el agente escribió primero, por ejemplo captación en Marketplace) |
| `classification` | Ver sección 4 |
| `state` | Ver sección 5 |
| `handled_by` | `roca` o `agent` (sección 6) |
| `last_inbound_at`, `last_outbound_at` | Para derivar quién debe responder |
| oportunidades enlazadas | N:M (sección 7) |

`awaiting` (quién debe el siguiente mensaje: `us`, `them`, `none`) se **deriva** de la dirección del último mensaje y no se guarda.

### 3.2 Message

| Atributo | Descripción |
|---|---|
| `direction` | Entrante o saliente |
| `author` | `person`, `agent` o `ai` (`actor_type`) |
| tipo | Texto, nota de voz, imagen, documento, otros [TBD cuáles se interpretan] |
| `external_ref` | Identificador en el canal |
| `occurred_at` | Cuándo ocurrió, aparte de cuándo se registró (`04` E3) |
| etiquetas de oportunidad | Oportunidades a las que alude (sección 7) |

## 4. Clasificación

`classification` indica qué es la conversación. Valores candidatos [TBD, taxonomía pendiente]: `lead` (interesado), `owner` (propietario), `agent` (otro agente), `known_contact`, `spam`, `other`, `unclassified`.

La clasificación es una **inferencia de ROCA** (`data_origin = inferred`) y pasa a `confirmed` cuando el agente la valida (`04` §4).

Flujo de entrada [Propuesta]:

1. **Mensaje con referencia a un inmueble registrado** (por ejemplo, un lead de portal o de un anuncio), o clasificado como `lead` con confianza suficiente: ROCA procede sola (sección 6).
2. **Mensaje ambiguo o que no parece lead:** ROCA propone una clasificación mediante una Task `decision` y **no responde sola** hasta que el agente confirme.

El umbral de confianza para actuar sola es [TBD].

## 5. Estados [Propuesta]

`new → active → dormant → closed`

| Estado | Significado |
|---|---|
| new | Entró un mensaje y aún no hay respuesta. Es el tiempo que se mide como primera respuesta |
| active | Hay intercambio en curso |
| dormant | Sin actividad durante un período (días: TBD) |
| closed | No requiere más acción (spam, conocido sin interés comercial, resuelta) |

Transiciones:

- `new → active` al enviarse la primera respuesta o al iniciar el intercambio.
- `active → dormant` por inactividad.
- `dormant → active` si llega un mensaje nuevo.
- cualquier estado `→ closed` por clasificación o decisión del agente.
- `closed → active` si llega un mensaje nuevo, salvo que esté clasificada como `spam` o bloqueada [Propuesta].
- Una conversación `outbound` nace en `active`, con la respuesta pendiente de la persona.

Relación con la Opportunity: una conversación `dormant` ligada a una Opportunity abierta genera una Task de seguimiento. Cerrar la Opportunity por "dejó de responder" se apoya en este estado, pero no es automático: lo decide el agente (aprobación primero).

## 6. Quién atiende

`handled_by` evita que ROCA y el agente respondan a la vez.

- **`roca`:** ROCA atiende dentro de su autonomía.
- **`agent`:** el agente atiende. ROCA solo sugiere o redacta borradores.

Cambio a `agent` [Propuesta]:

- ROCA escala cuando el mensaje contiene una negociación de precio, una fecha u hora concreta de visita ("¿mañana?"), una pregunta sin dato registrado o una queja.
- El agente responde directamente desde su teléfono: ROCA lo detecta y pasa a `agent`.
- El agente puede devolver la conversación a `roca`.

### 6.1 Primera respuesta automática [Decidido]

Ante un lead nuevo, ROCA responde sola:

- con **datos registrados** del inmueble y nada más (no inventa ni completa datos faltantes);
- con las preguntas de filtro (qué busca, presupuesto, plazo, encaje);
- sin negociar precio ni fijar horarios (se escala).

Si falta un dato que la persona pide, ROCA no lo inventa: responde con lo que tiene, avisa que lo confirmará y crea una Task `info_missing` [Propuesta].

Alta del lead [Propuesta]:

1. Identificar el canal y la fuente.
2. Comprobar si ya existe la persona y alguna Opportunity (ver `07-identity.md`).
3. Crear Person, Conversation y, si la consulta es sobre un inmueble registrado, una Opportunity `demand` en `interesado` con un vínculo `OpportunityProperty` de origen `inbound_inquiry` (`03` §7).
4. Si el mensaje es genérico y no revela una intención, la conversación queda sin Opportunity hasta identificarla.

## 7. Conversation y Opportunity [Decidido: N:M]

Un hilo puede tratar varias oportunidades y una oportunidad puede desarrollarse en varios hilos (otro canal, o varias personas intermediarias).

Para medir por oportunidad (tiempo de respuesta, interacciones), cada mensaje puede aludir a una o varias oportunidades. ROCA propone la etiqueta (`inferred`) y el agente puede corregirla [Propuesta].

## 8. Otros agentes como origen de Opportunity [Decidido el principio]

Un agente externo que contacta al autor con un cliente suyo por un inmueble del autor genera una Opportunity `demand`.

Propuesta de modelo:

- El agente es una Person con rol de agente (`02`).
- La Opportunity puede tener un **intermediario** (el agente) además de la persona cliente.
- Si el agente no comparte los datos del cliente, la Opportunity queda a nombre del agente, marcada como "actúa por un cliente".
- El reparto de comisión por defecto es 50/50, acordado caso a caso (`02`).

Caso inverso (el autor contacta a otro agente por un inmueble de este para un cliente propio): [TBD].

## 9. Eventos

El catálogo está en `04` §6.1 y siguientes. Principales: `conversation.started`, `message.received`, `message.sent`, `conversation.classified`, `conversation.classification_confirmed`, `conversation.state_changed`, `conversation.handler_changed`, `message.opportunity_tagged`, `ai.escalated_to_agent`.

## 10. Reglas de integridad

- **C1.** Una Conversation pertenece a una persona y a un canal.
- **C2.** Una Conversation puede existir sin Opportunity.
- **C3.** La clasificación es una inferencia hasta que el agente la confirma.
- **C4.** ROCA responde sola solo con datos registrados.
- **C5.** ROCA no responde sola a una conversación sin clasificar con suficiente confianza.
- **C6.** En cada momento una conversación tiene una única parte que la atiende (`handled_by`).
- **C7.** ROCA no negocia precio ni fija horarios en ninguna conversación; escala.
- **C8.** Una persona que pide no ser contactada marca la conversación y la persona como no contactable, y ROCA deja de escribir [Propuesta].
- **C9.** Cerrar una Opportunity por falta de respuesta requiere decisión del agente.

## 11. Pendientes

- Taxonomía de `classification` y umbral de confianza para actuar sola.
- Días de inactividad para `dormant`.
- Qué tipos de mensaje se interpretan (voz, imágenes, documentos).
- Políticas y límites de cada canal para mensajes salientes y primer contacto, a validar con cada conector.
- Control de envíos masivos y manejo de errores de interpretación.
- Confirmación del modelo de identidad y deduplicación (`07-identity.md`), dependencia del alta del lead.
- Caso inverso de agentes (el autor contacta a otro agente).
- Consentimiento y bajas (C8) y requisitos de protección de datos personales.

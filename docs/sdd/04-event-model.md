# 04 — Modelo de eventos

**Estado:** DRAFT v0.7  
**Última actualización:** 2026-10-02

## 1. Propósito

Este documento define cómo ROCA registra lo que ocurre en el proceso comercial y en el propio sistema, de forma que ese historial permita:

- reconstruir qué pasó, cuándo, con qué entidad y por qué (P10);
- explicar cada cambio de estado (invariante 10 de `03`);
- calcular estadísticas reales sobre el negocio del agente;
- alimentar en el futuro las capacidades de IA con datos fiables (P2).

Todo hecho relevante debe producir un evento. Todo resultado (de una visita, de una oportunidad, de una acción de ROCA) debe quedar registrado como evento con motivo estructurado cuando sea posible.

Los puntos no resueltos permanecen como TBD (P11).

## 2. Decisión base: estado almacenado + eventos añadidos

**Decisión:** ROCA guarda el **estado actual** de cada entidad y, además, **añade eventos inmutables** por cada hecho relevante. El estado actual no se reconstruye obligatoriamente a partir de los eventos (no es *event sourcing* completo).

Motivo: P12 (evitar sobreingeniería prematura). Se obtiene trazabilidad y datos para análisis sin el costo de reconstruir todo el estado desde el log.

Consecuencia que debe respetarse: **estado y evento deben cambiar juntos**. No debe existir un cambio de estado sin evento que lo explique (ver E1 en la sección 10).

*Revisable:* si más adelante se necesita reconstruir estados históricos con exactitud, se evaluará una migración hacia event sourcing. TBD.

## 3. Sobre del evento (campos comunes)

Todo evento comparte la misma estructura base. El contenido específico va en `payload`.

| Campo | Descripción | Estado |
|---|---|---|
| `event_id` | Identificador único | Definido |
| `event_type` | Tipo de evento (catálogo de la sección 6) | Definido |
| `schema_version` | Versión del esquema del payload | Definido |
| `occurred_at` | Cuándo ocurrió el hecho en el mundo real | Definido |
| `recorded_at` | Cuándo lo registró ROCA | Definido |
| `organization_id` | Organización propietaria del dato | Definido (modelo multiagente: TBD) |
| `actor_type` | `agent`, `person`, `system`, `ai`, `integration` | Definido |
| `actor_id` | Quién lo produjo, cuando aplica | Definido |
| `subject_type` / `subject_id` | Entidad principal afectada | Definido |
| `person_id`, `opportunity_id`, `property_id`, `conversation_id`, `visit_id` | Referencias de contexto, todas opcionales | Definido |
| `channel_id` | Referencia al registro de canales (WhatsApp, portal, etc.), cuando aplique. Ver 3.1 | Definido |
| `data_origin` | Procedencia del dato (sección 4) | Definido |
| `causation_id` | Evento que causó a este (ej. mensaje → seguimiento creado) | Definido |
| `correlation_id` | Agrupa eventos de un mismo flujo | TBD |
| `payload` | Datos específicos del tipo de evento | Definido |
| `external_ref` | Identificador en el sistema externo (para deduplicar) | TBD |

Notas:

- `occurred_at` y `recorded_at` son distintos a propósito. Un mensaje puede haberse enviado antes de que una integración lo entregue a ROCA, y las métricas de tiempo deben usar `occurred_at`.
- El **contexto relevante en el momento del hecho** debe ir en el payload cuando afecte análisis posteriores. Ejemplo: al recibir un lead sobre una propiedad, guardar el precio vigente en ese instante, no solo la referencia a la propiedad.

### 3.1 Canales como datos [Decidido]

Los canales pueden agregarse, modificarse, actualizarse o eliminarse con el tiempo (P13). Por eso `channel_id` apunta a un **registro de canales** y no a una lista fija de valores. Un canal eliminado o desactivado no borra los eventos históricos que lo referencian.

Estructura del registro de canales: [TBD].

## 4. Procedencia del dato (`data_origin`)

Recoge la distinción ya establecida en `02` §10 y la vuelve verificable:

| Valor | Significado |
|---|---|
| `declared` | La persona o el agente lo dijo o lo escribió explícitamente |
| `observed` | ROCA lo observó como comportamiento (abrió, respondió, tardó) |
| `derived` | Calculado de forma determinística a partir de otros eventos |
| `inferred` | Interpretación de ROCA (clasificación, intención, probabilidad) |
| `confirmed` | Una inferencia que el agente o la persona confirmó expresamente |

**Regla:** una inferencia nunca se convierte en hecho por sí sola. Solo un evento explícito de confirmación la promueve a `confirmed`.

Los eventos con `data_origin = inferred` deben incluir `model_version` y, cuando exista, `confidence` en el payload.

## 5. Tipos de evento por naturaleza

Para análisis posterior es útil distinguir cinco familias:

1. **Hechos del mundo:** algo ocurrió fuera de ROCA (mensaje recibido, llamada, visita realizada).
2. **Acciones del agente:** el agente hizo algo en ROCA (editó, aprobó, registró resultado).
3. **Acciones de ROCA:** ROCA ejecutó algo dentro de su autonomía (envió seguimiento autorizado, creó tarea).
4. **Propuestas e inferencias de ROCA:** ROCA sugirió o interpretó algo (propuesta de respuesta, clasificación de lead).
5. **Decisiones sobre propuestas:** el agente aceptó, editó o rechazó lo propuesto.

Las familias 4 y 5 son especialmente importantes: miden cuánto aporta ROCA y dónde se equivoca (ver sección 8).

## 6. Catálogo preliminar de eventos

Lista inicial. Los nombres y payloads son preliminares y se formalizarán por dominio. Se usa `dominio.acción` en pasado.

### 6.1 Personas y conversaciones

| Evento | Payload esencial |
|---|---|
| `person.created` | origen, canal, datos de contacto |
| `lead.received` | portal, inmueble de referencia, datos declarados (nombre, DNI, teléfono, correo), método de contacto elegido (ver datos, WhatsApp, llamada, llamada por WhatsApp o correo) |
| `person.identifier_added` | tipo, valor normalizado, fuente, verificación |
| `person.identifier_marked_invalid` | identificador, motivo (número sin WhatsApp, correo que rebota, dato falso), portal o canal de origen |
| `person.match_proposed` | nivel (`certain`/`probable`/`possible`), evidencia |
| `person.match_confirmed` / `person.match_rejected` | quién decidió |
| `person.merged` | personas fusionadas, nivel de coincidencia, evidencia, quién la confirmó (ver `07-identity.md`) |
| `person.unmerged` | motivo, quién lo revirtió |
| `opportunity.inquiry_attached` | consulta nueva enlazada a una Opportunity abierta, canal |
| `conversation.started` | canal, origen, propiedad de referencia si existe |
| `message.received` | canal, longitud, adjuntos, referencia externa |
| `message.sent` | canal, autor (`agent`/`ai`), plantilla si aplica, regla de autonomía que lo permitió si lo envió ROCA |
| `message.opportunity_tagged` | oportunidades a las que alude el mensaje (`inferred`, corregible por el agente) |
| `conversation.classified` | clasificación propuesta por ROCA (`inferred`), confianza |
| `conversation.classification_confirmed` | quién confirmó, clasificación final |
| `conversation.state_changed` | `from_state`, `to_state` (`new`/`active`/`dormant`/`closed`), disparador |
| `conversation.handler_changed` | de `roca` a `agent` o a la inversa, motivo (escalamiento, respuesta directa del agente, devolución) |
| `call.logged` | duración, resultado |

### 6.2 Oportunidades

| Evento | Payload esencial |
|---|---|
| `opportunity.created` | lado (`demand`/`supply`), objetivo, canal y publicación de origen, conversación de origen, inmueble (si `supply`) |
| `opportunity.state_changed` | `from_state`, `to_state`, disparador, motivo |
| `opportunity.closed` | `outcome`, `reason_code`, operación real concluida, motivo libre opcional, contexto, número de ciclo |
| `opportunity.reopened` | `reopen_reason`, evento que lo disparó, cierre anterior que se revierte, estado de destino, número de ciclo |
| `opportunity.reopen_proposed` | actor (ROCA), evidencia (mensaje o evento que indica que la persona retomó el objetivo), cierre que se revertiría. La decisión se registra como `ai.action_approved` o `ai.action_rejected` |
| `opportunity.owner_changed` | agente anterior y nuevo (permisos: TBD) |
| `opportunity.superseded_by` | Opportunity que la reemplaza (cambio de objetivo) |
| `opportunity.reactivation_candidate_detected` | evento disparador (ej. cambio de precio), versión de Requirement evaluada. Es solo una señal; la reapertura es un evento aparte (`opportunity.reopened`) |

### 6.3 Requirement

| Evento | Payload esencial |
|---|---|
| `requirement.version_created` | número de versión, campos cambiados (antes/después), motivo declarado si existe |
| `requirement.field_flexibility_changed` | campo, de obligatorio/preferido/flexible/excluido a otro |

### 6.4 Opportunity–Property (relación)

Estos eventos registran cómo evoluciona el vínculo entre una oportunidad y cada inmueble (`02` §8). Son los que más valor aportarán a futuras recomendaciones.

| Evento | Payload esencial |
|---|---|
| `match.candidate_identified` | origen (`recommended_by_roca`, etc.), versión de Requirement evaluada, resumen de encaje (campos cumplidos e incumplidos) |
| `match.recommended` | quién recomendó, versión de Requirement contra la que se evaluó |
| `match.shown` | canal, momento |
| `match.interest_expressed` | declarado u observado |
| `match.rejected` | `reason_code` (precio, zona, características…), campo del Requirement implicado |
| `match.shortlisted` | — |
| `match.visited` | referencia a la visita |
| `match.negotiation_started` | referencia a la negociación |
| `match.reconsidered` | de `rejected` a `interested` o `shortlisted`, evento disparador (ej. baja de precio) |
| `match.selected` | operación concluida asociada |
| `match.unavailable` | motivo (ej. inmueble tomado por otra persona) |

El modelo está en `03` §7 (borrador). Estos eventos deben alinearse con sus estados y motivos de rechazo cuando se confirmen.

### 6.5 Visitas

| Evento | Payload esencial |
|---|---|
| `visit.created` | tipo (`capture`/`production`/`showing`), Opportunity, inmueble, `tour_id` opcional |
| `visit.proposed` | disponibilidad recopilada de la persona |
| `visit.scheduled` | fecha acordada, decidida por el agente |
| `visit.confirmed` | quién confirmó, canal |
| `visit.duplicate_warning` | visita abierta existente para el mismo inmueble y la misma persona o un posible duplicado |
| `visit.rescheduled` | fecha anterior y nueva, quién lo pidió |
| `visit.cancelled` | quién canceló, `reason_code` |
| `visit.not_held` | `reason_code` (no se presentó, imprevisto) |
| `visit.held` | fecha real, asistentes |
| `visit.result_recorded` | nota libre del agente (`declared`), resultado comercial si lo indica, `reason_code` |
| `visit.result_structured` | estructuración de la nota por ROCA (`inferred`); pasa a `confirmed` cuando el agente la valida |

`visit.held` y `visit.result_recorded` son eventos separados a propósito (P7): una visita puede realizarse y su resultado registrarse después. El feedback es libre y estructurable: el agente no rellena un formulario rígido.

### 6.6 Propiedades y publicaciones

| Evento | Payload esencial |
|---|---|
| `property.created` | origen (propietario, enlace, conversación) |
| `property.updated` | campos cambiados (antes/después) |
| `property.price_changed` | precio anterior y nuevo, motivo |
| `property.state_changed` | `from_state`, `to_state` (máquina de Property: TBD) |
| `service_agreement.accepted` | propietario, inmueble, operaciones cubiertas, comisión pactada (porcentaje o meses de renta, y si es la estándar), nivel de exclusividad (`none`/`semi`/`full`), vigencia (por defecto 3 meses en alquiler y 6 en venta), y relación con las visitas (antes, durante o después). **Este evento marca la captación** (invariante 16 de `02`) |
| `service_agreement.expiring_soon` | generado por el sistema antes del vencimiento (por defecto 30 días antes [Propuesta]); no cambia nada por sí solo. El agente decide seguir (`service_agreement.renewed`) o archivar (`opportunity.closed` con motivo `agreement_not_renewed`) |
| `service_agreement.renewed` | nueva vigencia, cambios de condiciones |
| `service_agreement.expired` / `service_agreement.terminated` | motivo |
| `publication.prepared` | destino, versión del contenido |
| `publication.approved` | quién aprobó |
| `publication.published` | destino, referencia externa |
| `publication.updated` / `publication.removed` | motivo |
| `publication.cost_recorded` | monto, moneda, `channel_id`, periodo (gasto en anuncios o promoción) |

### 6.7 Tareas

| Evento | Payload esencial |
|---|---|
| `task.created` | tipo (`follow_up`/`decision`/`record_result`/`info_missing`/`reminder`), origen (evento que la generó), hora fija o fecha límite, creada por regla, ROCA o agente, factores de prioridad vigentes |
| `task.completed` | quién, resultado |
| `task.dismissed` | `reason_code` |
| `task.overdue` | días de retraso |
| `followup.sent` | seguimiento ejecutado por ROCA dentro de su autonomía, canal, regla que lo permitió |
| `followup.notified` | aviso al agente para que haga el seguimiento (llamada o mensaje), motivo |
| `followup.completed_by_agent` | el agente registra que lo hizo, canal y resultado |
| `task.snoozed` | nueva fecha, motivo opcional |
| `task.auto_resolved` | evento que la resolvió |
| `calendar.event_imported` | calendario, evento externo, `external_ref` |
| `calendar.event_changed_externally` | cambio (hora o borrado) y entidad vinculada afectada; por sí solo no modifica la Visit |
| `notification.sent` | tipo, canal (`push`), entidad relacionada |
| `notification.opened` / `notification.acted` | tiempo hasta la apertura o la acción |

### 6.8 Automatización e IA

| Evento | Payload esencial |
|---|---|
| `ai.classification_made` | qué clasificó, valor, `model_version`, `confidence` |
| `ai.action_proposed` | acción propuesta, nivel de autonomía requerido |
| `ai.action_approved` | quién aprobó |
| `ai.action_edited` | qué cambió el agente respecto a la propuesta |
| `ai.action_rejected` | `reason_code` |
| `ai.action_executed` | acción ejecutada de forma autónoma, regla que la permitió |
| `ai.escalated_to_agent` | motivo de escalamiento |

### 6.9 Negociación y ofertas económicas

Las ofertas se registran como actividades dentro de la etapa de negociación, con su subtipo (`03` §2.6).

| Evento | Payload esencial |
|---|---|
| `negotiation.activity_logged` | `kind`: `offer`, `counteroffer`, `considering`, `accepted`, `rejected`, `withdrawn` (lista cerrada: TBD); monto, moneda, condiciones, quién la hace (interesado, propietario, agente), inmueble asociado |

La oferta económica no cierra la Opportunity: el cierre es un evento aparte (`opportunity.closed`).

### 6.10 Operaciones concluidas y contratos

| Evento | Payload esencial |
|---|---|
| `operation.concluded` | tipo (`sale`/`rent`), inmueble, partes, monto, comisión aplicada y su monto, reparto con otros agentes (por defecto 50/50), fechas (en alquiler, inicio y fin), Opportunities enlazadas |
| `operation.fell_through` | operación afectada, `reason_code`, etapa en que se cayó |
| `operation.closed_by_third_party` | quién la cerró (propietario u otro agente), tipo, monto si se conoce, nivel de exclusividad vigente. En sin/semi-exclusividad no genera comisión; en exclusividad sí, por defecto el 100% (variable) |
| `lease.ending_soon` | generado por el sistema según la fecha de fin; por defecto 30 días antes (`notice_period_days` del contrato) |
| `lease.renewed` | nueva fecha de fin, cambio de monto |
| `lease.ended` | resultado: renovado o desocupado |
| `commission.collected` | operación, monto cobrado, fecha (la paga el propietario), parte propia y de otros agentes |

`lease.ending_soon` lo produce el paso del tiempo y no una acción, por lo que su `actor_type` es `system`. Dispara los seguimientos al propietario y al inquilino (`followup.*`) y, si el inquilino se va, las propuestas de reapertura (`opportunity.reopen_proposed`) con `reopen_reason` = `contract_ended` u `operation_fell_through`.

## 7. Eventos de transición y resultados

### 7.1 Transiciones

Todo cambio de estado (Opportunity, Visit, Property) genera un evento `*.state_changed` con:

- estado origen y destino;
- qué lo disparó (evento previo, acción del agente, regla automática);
- actor;
- motivo cuando aplique.

Esto permite medir el tiempo en cada etapa y explicar cualquier cambio (invariante 10 de `03`).

### 7.2 Resultado y motivo estructurado

Cuando un proceso termina (oportunidad, visita, propuesta de IA, tarea descartada), el evento debe llevar:

- `outcome`: resultado en un conjunto cerrado;
- `reason_code`: motivo en un conjunto cerrado y versionado;
- `reason_text`: texto libre opcional.

**Los motivos estructurados son más valiosos que el texto libre** porque son los que permiten agregar. El texto libre complementa, no reemplaza.

Los conjuntos de `outcome` y `reason_code` por entidad son TBD y dependen de las taxonomías pendientes de `03` §9. Candidatos ya identificados en `03` §2.8: objetivo cambiado, necesidad desaparecida, operación cerrada, inmueble captado, encontró otra alternativa, no hubo inmueble adecuado, no acepta condiciones, dejó de responder, descartado por criterios, otro.

## 8. Eventos pensados para estadísticas

El objetivo es que las métricas futuras salgan de los eventos sin tener que reconstruir información que no se guardó. Esta tabla relaciona métricas esperadas con los eventos que las hacen posibles.

| Métrica | Eventos necesarios |
|---|---|
| Tiempo de primera respuesta | `message.received` → primer `message.sent` (usando `occurred_at`) |
| Conversión por fuente/canal | `opportunity.created` (con origen) y `opportunity.closed` (con outcome) |
| Tiempo por etapa | `opportunity.state_changed` |
| Visitas que avanzan | `visit.held`, `visit.result_recorded`, `opportunity.closed` |
| Motivos de pérdida más frecuentes | `opportunity.closed`, `match.rejected` |
| Qué propiedades generan demanda | `conversation.started`/`opportunity.created` con propiedad de referencia, `match.interest_expressed` |
| Efecto de un cambio de precio | `property.price_changed` y consultas posteriores |
| Rendimiento de publicaciones | `publication.published` y leads con esa publicación como origen |
| Utilidad de ROCA | `ai.action_proposed` vs `ai.action_approved` / `ai.action_edited` / `ai.action_rejected` |
| Embudo por inmueble (leads → conversaciones → visitas → oferta → cierre) | `opportunity.created` con inmueble y canal, `conversation.started`, `visit.held`, `negotiation.activity_logged`, `opportunity.closed` |
| Rendimiento por canal vs costo (ej. Marketplace frente a anuncios) | `publication.published`, `publication.cost_recorded`, `opportunity.created` con `channel_id` y `opportunity.closed` |
| Motivos y frecuencia de reapertura | `opportunity.closed`, `opportunity.reopened` |
| Renovación y retención de alquileres | `lease.ending_soon`, `lease.renewed`, `lease.ended` |
| Tiempo de vacancia de un inmueble | `lease.ended` (desocupado) hasta el siguiente `operation.concluded` |
| Operaciones que se caen | `operation.fell_through`, `operation.concluded` |
| Ingresos por canal, tipo de operación o inmueble | `operation.concluded` (comisión aplicada), `commission.collected`, `opportunity.created` con `channel_id` |
| Eficacia de captación | `opportunity.created` (`supply`), `visit.held` (`capture`), `visit.result_recorded` (acepta, lo pensará, rechaza), `service_agreement.accepted` |
| Comisión pactada frente a la estándar | `service_agreement.accepted` |
| Rendimiento por nivel de exclusividad | `service_agreement.accepted`, `operation.concluded`, tiempo hasta el cierre, cierres por otro agente o por el propietario |
| Cierres por terceros | `operation.closed_by_third_party`, nivel de exclusividad |
| Vencimientos de acuerdos | `service_agreement.expiring_soon`, `service_agreement.renewed`, `service_agreement.expired` |
| Seguimientos que ocurren | `followup.notified` vs `followup.completed_by_agent` |
| Cumplimiento de tareas | `task.created`, `task.completed`, `task.dismissed`, `task.overdue` |
| Conversaciones que se enfrían | `conversation.state_changed` (a `dormant`), `opportunity.closed` con motivo "dejó de responder" |
| Escaladas de ROCA al agente | `conversation.handler_changed`, `ai.escalated_to_agent` |
| Calidad de leads por portal | `lead.received`, `person.identifier_marked_invalid` |
| Duplicados y precisión de las fusiones | `person.match_proposed`, `person.match_confirmed`, `person.match_rejected`, `person.merged`, `person.unmerged` |
| Efectividad de los avisos | `notification.sent`, `notification.opened`, `notification.acted` |

### 8.1 Exposición: registrar lo que se ofreció, no solo lo que avanzó

Una tasa de conversión necesita denominador. Por eso deben registrarse los eventos de exposición (`match.recommended`, `match.shown`, `publication.published`) aunque no tengan respuesta. Sin ellos solo se sabe qué funcionó, no qué se intentó.

### 8.2 Correcciones del agente

`ai.action_edited` y `ai.action_rejected` son la señal más directa de dónde ROCA se equivoca. Deben guardar la diferencia entre lo propuesto y lo final.

### 8.3 Expectativas realistas

Con un agente independiente el volumen de eventos será bajo durante meses. Hasta que haya historial suficiente (ver `00` §6), las estadísticas deben presentarse como **descripción de lo ocurrido** (panel de actividad y resultados), no como predicción ni como "aprendizaje" de ROCA.

### 8.4 Seguimientos ejecutados por el agente

Algunos seguimientos los ejecuta ROCA (`followup.sent`) y otros solo los notifica y los hace el agente por llamada o mensaje (`followup.notified`). En el segundo caso ROCA no sabe que ocurrió hasta que el agente lo registra, así que sin `followup.completed_by_agent` esos seguimientos aparecen como no realizados. Cómo reducir esa fricción de registro se propone en `05-agenda.md` (resultado con un toque más nota opcional).

### 8.5 Ciclos de cierre y reapertura

Como una Opportunity puede cerrarse y reabrirse (`03` §5), las métricas deben tratar cada ciclo por separado: el tiempo por etapa se mide por ciclo, el `outcome` vigente es el del último cierre y se cuentan las reaperturas con su motivo. Todos los cierres previos se conservan como eventos.

## 9. Relación con la IA

1. La IA **consume** eventos; no escribe en el historial de hechos. Lo que produce se registra como eventos de la familia 4 (`inferred`), nunca como `declared`.
2. Las inferencias deben ser **versionadas**: si cambia el modelo o el prompt, debe poder saberse con cuál se produjo cada una (`model_version`).
3. Las decisiones del agente sobre propuestas (familia 5) son datos de primera clase.
4. Qué decide el LLM y qué decide código determinístico se especificará en el SDD de IA. Este documento solo fija que **ambos dejan eventos distinguibles** (`actor_type = ai` vs `system`).
5. El uso de históricos de una persona para recomendar a otra requerirá evidencia suficiente y reglas propias (`02` §10). Aislamiento de datos entre organizaciones para cualquier entrenamiento o agregación: **TBD**.
6. **Aprobación primero, automatización después** [Decidido para la etapa inicial]. Qué acciones pasan de "requiere aprobación" a "automática" se decidirá con datos: la tasa de propuestas aprobadas sin edición (`ai.action_approved` frente a `ai.action_edited` y `ai.action_rejected`) es la evidencia natural para promover una acción. Criterio de promoción: [TBD].

## 10. Reglas de integridad

- **E1.** Todo cambio de estado de una entidad debe generar un evento que lo explique.
- **E2.** Los eventos son inmutables. Una corrección se registra como un nuevo evento que referencia al anterior, no modificando el original.
- **E3.** `occurred_at` y `recorded_at` se almacenan siempre por separado.
- **E4.** Todo evento incluye `actor_type` y `data_origin`.
- **E5.** Una inferencia (`inferred`) no se promueve a `confirmed` sin un evento explícito de confirmación.
- **E6.** Todo cierre o descarte incluye `outcome` y `reason_code` cuando el motivo es conocido; si no lo es, se registra explícitamente como desconocido, no se omite.
- **E7.** Los eventos de exposición (recomendado, mostrado, publicado) se registran aunque no reciban respuesta.
- **E8.** Los eventos importados de integraciones deben poder deduplicarse mediante `external_ref`.
- **E9.** El payload debe incluir el contexto que pueda cambiar después y sea relevante para análisis (ej. precio vigente).
- **E10.** Crear una nueva Opportunity no modifica ni elimina eventos de Opportunities anteriores (invariante 20 de `02`).
- **E11.** Una reapertura no modifica ni elimina eventos de cierre anteriores.

## 11. Impacto en documentos anteriores

Resuelto en `03` v0.4 y `02` v0.4:

1. Estado y resultado separados (estado terminal `cerrada` más `outcome` y `reason_code`), pendiente de confirmar su taxonomía.
2. Lado del mercado, objetivo y operación definidos; una Opportunity `supply` por inmueble.
3. Etapas de calificación y negociación propuestas.
4. Captación como hito intermedio y reapertura sobre la misma Opportunity.

Siguen abiertos:

1. **Detalle de la reapertura** (`03` §5): decidida (misma Opportunity, solo si la persona retoma su objetivo; ROCA propone y el agente aprueba). También aplica tras `won` si la operación se cae o termina el contrato de alquiler. No hay límite de reaperturas. Falta definir qué reaperturas serán automáticas.
2. **Resultado y estados de Visit** (`03` §6).
3. **Máquina de estados de Conversation**: borrador en `06-conversations.md`, pendiente de confirmación.
4. **OpportunityProperty** (`03` §7): borrador propuesto; los eventos de la sección 6.4 dependen de su confirmación.
5. **Máquina de estados de Property** y su relación con `captada`.
6. **Modelo de `Operation`** (`02`): los eventos de la sección 6.10 dependen de él.

## 12. Fuera de alcance de esta versión

- Tecnología de almacenamiento, bus de eventos o colas.
- Retención, archivo y borrado de eventos. En particular, cómo conciliar la inmutabilidad con el derecho a eliminar datos personales: **TBD**.
- Orden y consistencia de eventos de integraciones externas con retrasos o duplicados.
- Diseño de tableros, reportes o métricas concretas.
- Formato exacto de cada payload.

## 13. Decisiones pendientes

- Taxonomías cerradas de `outcome` y `reason_code` por entidad.
- Catálogo definitivo de eventos y payloads por dominio.
- Definición de `correlation_id` y `external_ref`.
- Modelo de `OpportunityProperty` y sus eventos asociados.
- Reglas de reapertura de Opportunity.
- Política de retención y borrado frente a inmutabilidad.
- Aislamiento de datos entre organizaciones para analítica y IA.
- Cuándo, si alguna vez, migrar a event sourcing.
- Reaperturas automáticas.
- Modelo de `Operation`.
- Estructura del acuerdo de servicios y de la comisión (vigencia por operación, porcentaje devengado en exclusividad).
- Confirmación del tratamiento del vencimiento del acuerdo (`03` §2.10).
- Confirmación del modelo de agenda (`05-agenda.md`).
- Confirmación del modelo de Conversation (`06-conversations.md`).
- Confirmación del modelo de identidad y deduplicación (`07-identity.md`).
- Estructura del registro de canales.
- Lista cerrada de subtipos de negociación.
- Cómo se registra con poca fricción lo que el agente hace fuera de ROCA (llamadas, mensajes directos).

# 05 — Agenda y tareas

**Estado:** DRAFT v0.1  
**Última actualización:** 2026-10-03

Etiquetas: **[Decidido]** confirmado por el autor del SDD; **[Propuesta]** sugerido y pendiente de confirmación; **[TBD]** sin definir.

## 1. Propósito

Resolver el problema central del autor: perder visitas y seguimientos si no los registra en el momento. ROCA debe reducir la dependencia de la memoria humana.

"Mi día" reúne en una sola vista, con orden de prioridad, lo que hoy el calendario del teléfono mantiene separado: visitas, reuniones, recordatorios, seguimientos y tareas.

## 2. Decisiones

- **Sincronización bidireccional** con el calendario externo [Decidido].
- **Orden de Mi día:** horas fijas primero, luego por dinero en juego [Decidido]. La fórmula del "dinero en juego" es [TBD] (sección 6).
- **Avisos** por notificación push y dentro de Mi día [Decidido].
- **Aprobación primero** (`04` §9): ROCA crea tareas, avisos y recordatorios dentro de su autonomía, pero no ejecuta acciones sensibles; las propone y el agente decide.

## 3. Task [Propuesta]

Una Task es una unidad de trabajo pendiente **para el agente**: contactar, decidir, registrar un resultado, completar información o recordar algo. ROCA las crea a partir de eventos y reglas; el agente las completa, pospone o descarta.

Lo que ROCA ejecuta por sí mismo (por ejemplo, un seguimiento rutinario) no es una Task: es una acción que deja eventos (`followup.sent`). Si requiere aprobación, la Task es la decisión de aprobarla.

### 3.1 Atributos

| Atributo | Descripción |
|---|---|
| `kind` | Ver 3.2 |
| `scheduled_at` | Hora fija (llamada agendada, por ejemplo). Excluyente con `due_at` |
| `due_at` | Fecha límite flexible. Sin hora fija |
| `context` | Entidades relacionadas: Person, Opportunity, Property, Visit, ServiceAgreement u Operation |
| `summary` | Línea de contexto legible (por ejemplo, "Carlos — confirmar visita") |
| `origin` | Evento o regla que la generó (`causation_id`), o creación manual |
| `created_by` | `system`, `ai` o `agent` |
| `status` | Ver sección 4 |
| `priority_factors` | Factores de prioridad vigentes (sección 6) |

### 3.2 Tipos

| Tipo | Uso |
|---|---|
| `follow_up` | Contactar a alguien por llamada o mensaje |
| `decision` | Aprobar o decidir: publicar, enviar un mensaje sensible, reabrir una Opportunity, renovar o archivar un acuerdo |
| `record_result` | Registrar el resultado de una visita o gestión pendiente |
| `info_missing` | Completar información faltante de un inmueble o una persona |
| `reminder` | Recordatorio manual del agente |

### 3.3 Qué genera tareas

| Origen | Tarea |
|---|---|
| Visit `realizada` con resultado pendiente | `record_result` |
| Visita `capture` con respuesta "lo pensará" y fecha prometida | `follow_up` en esa fecha |
| Seguimiento prometido a una persona | `follow_up` |
| `lease.ending_soon` | `follow_up` al propietario y al inquilino |
| `service_agreement.expiring_soon` | `decision` (seguir o archivar) |
| `ai.action_proposed` (publicación, mensaje, reapertura) | `decision` |
| Cambio de precio con candidatos de reactivación | `decision` |
| Información faltante antes de una visita | `info_missing` |
| El agente la crea | `reminder` |

## 4. Estados

`pending → done` o `dismissed`. Una tarea pospuesta sigue `pending` con nueva fecha (`snoozed`). `overdue` se deriva de la fecha y no se guarda.

- **Completar** registra el resultado con un toque (por ejemplo, para un seguimiento: contestó, no contestó, reagendó) más una nota opcional [Propuesta]. Reducir la fricción de registro es clave para la calidad de los datos (`04` §8.4).
- **Descartar** exige un motivo.
- **Resolución automática** [Propuesta]: si el hecho que originó la tarea ocurre solo (por ejemplo, la persona responde), la tarea se cierra como `auto_resolved` con su causa.

## 5. Elementos de hora fija

Una Visit **no genera una Task adicional**. Aparece en Mi día como elemento de hora fija vinculado a la Visit, que es la fuente de verdad de su estado y fecha. Lo mismo para los eventos externos importados del calendario. Las Tasks con `scheduled_at` también son de hora fija.

## 6. Mi día

### 6.1 Estructura [Decidido el criterio de orden]

1. **Horas fijas, en orden cronológico:** visitas, reuniones, tareas con `scheduled_at` y eventos externos (como bloques ocupados).
2. **Tareas flexibles, ordenadas por dinero en juego.**

Cada elemento muestra su línea de contexto (por ejemplo, "María — seguimiento prometido hoy", "Pedro — propietario aún no envió documentos").

### 6.2 Prioridad por dinero [Propuesta de factores, fórmula TBD]

El orden debe ser **determinístico y explicable**: el agente puede ver por qué algo va primero. Factores candidatos:

- tarea vencida o con fecha límite hoy;
- cercanía de la Opportunity al cierre (etapa);
- comisión potencial cuando se conoce (por ejemplo, la pactada en el acuerdo de servicios);
- tiempo desde el último contacto frente a lo prometido.

No se define una fórmula hasta tener datos reales. Los factores vigentes se guardan en cada tarea para poder estudiar después qué priorización funciona. Al inicio no se usa un LLM para ordenar [Propuesta].

## 7. Sincronización con el calendario externo

**Bidireccional [Decidido].** Reglas propuestas:

- **Conector:** la sincronización pasa por un conector (P13). El proveedor es [TBD]. Los calendarios conectados son datos configurables, como los canales.
- **Hacia fuera:** se envían los elementos de hora fija (visitas, reuniones). Las tareas flexibles permanecen en Mi día [Propuesta].
- **Hacia dentro:** los eventos externos aparecen en Mi día como bloques ocupados y ROCA los usa para evitar choques al coordinar visitas. Los eventos personales se muestran solo como "ocupado" [Propuesta; privacidad TBD].
- **Mover un evento vinculado a una Visit** en el calendario externo se registra como reprogramación hecha por el agente. ROCA no avisa a la contraparte sin aprobación.
- **Borrar un evento vinculado a una Visit:** ROCA pregunta si la visita se canceló y no la cancela por sí sola.
- **Duplicados:** se evitan con `external_ref` (`04` E8).
- **Cambios simultáneos en ambos lados:** [TBD].

## 8. Avisos

Canales: **notificación push y Mi día** [Decidido].

Tipos candidatos: recordatorio de visita, seguimiento vencido o por vencer, decisiones pendientes (aprobaciones, vencimientos, reaperturas) y resumen diario [TBD].

Reglas contra el ruido [Propuesta, TBD en detalle]: agrupar avisos, horario silencioso y límite de avisos por día.

Cada aviso genera eventos (`notification.sent`, `opened`, `acted`) para medir si realmente producen acción.

## 9. Eventos

El catálogo está en `04` §6.7 y siguientes. Principales: `task.created`, `task.completed`, `task.dismissed`, `task.snoozed`, `task.auto_resolved`, `task.overdue`, `calendar.event_imported`, `calendar.event_changed_externally`, `notification.sent`, `notification.opened`, `notification.acted`.

## 10. Reglas de integridad

- **T1.** Toda Task tiene un origen: un evento, una regla o una creación manual.
- **T2.** Toda Task referencia al menos una entidad de contexto, salvo un `reminder` manual.
- **T3.** Una Visit no genera una Task duplicada.
- **T4.** Completar una Task registra un resultado y descartarla registra un motivo.
- **T5.** ROCA crea y actualiza Tasks dentro de su autonomía; completar una Task `decision` no ejecuta nada por sí sola.
- **T6.** Un cambio en el calendario externo no cancela ni reprograma nada sin quedar registrado como acción del agente. Cancelar requiere confirmación.
- **T7.** El orden de Mi día es determinístico y explicable.
- **T8.** Una Task resuelta por el propio hecho que la originó se cierra como `auto_resolved` con su causa.

## 11. Pendientes

- Fórmula de prioridad por dinero en juego.
- Anticipación de los recordatorios de visita y de los avisos de vencimiento.
- Proveedor de calendario y regla para cambios simultáneos.
- Privacidad de los eventos personales importados.
- Resumen diario, horario silencioso y límites de avisos.
- Experiencia de registro con un toque para cada tipo de tarea.
- Tareas en equipos: reasignación y supervisión (posterior a V1).
- Tareas recurrentes.

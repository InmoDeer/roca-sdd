# 07 — Identidad de personas y deduplicación

**Estado:** DRAFT v0.1  
**Última actualización:** 2026-10-03

Etiquetas: **[Decidido]** confirmado por el autor del SDD; **[Propuesta]** sugerido y pendiente de confirmación; **[TBD]** sin definir.

## 1. Propósito

Una misma persona llega por varios canales: un portal, WhatsApp, una llamada, Messenger. Si ROCA la registra como varias personas, contacta dos veces a quien ya fue contactado, duplica oportunidades y puede agendar la misma visita dos veces.

Caso real [Decidido como requisito]: una visita agendada con la misma persona por dos medios distintos, WhatsApp (portal) y Messenger (Facebook Marketplace).

## 2. Qué llega por cada canal [Hechos aportados por el autor]

| Fuente | Qué llega | Fiabilidad |
|---|---|---|
| Portales (Urbania, Adondevivir, Babilonia) | **Siempre un correo** con los datos del inmueble y del lead: nombre, DNI, teléfono y correo | Datos declarados, **no siempre reales** |
| Portales, contacto adicional | Según lo que elija la persona: solo ver tus datos, escribirte por WhatsApp, llamarte, llamarte por WhatsApp o enviarte un correo | El canal usado queda como evidencia |
| WhatsApp | Número y mensajes | El número lo verifica el propio canal |
| Messenger / Marketplace | Identificador y nombre de perfil, normalmente sin teléfono | El nombre de perfil puede no ser el real |
| Llamada | Número entrante | Verificado por la línea |

Consecuencia: **el correo del portal es siempre el evento de entrada del lead**, y el contacto por WhatsApp o llamada puede llegar después o no llegar nunca.

## 3. Principio [Decidido]

Al llegar un lead, **primero se busca en la base de datos** si ya existe una persona registrada con ese nombre y/o teléfono, antes de crear una nueva. Propongo ampliar la búsqueda al correo, al DNI y a los identificadores de canal.

## 4. Modelo [Propuesta]

| Elemento | Descripción |
|---|---|
| `PersonIdentifier` | Identificador de una persona: teléfono, correo, DNI, WhatsApp, Messenger, ID de lead de portal. Guarda valor normalizado, fuente, verificación, estado y fechas |
| Verificación | `channel_verified` (lo prueba el propio canal), `declared` (escrito en un formulario o mensaje), `agent_confirmed` |
| Estado | Activo o `invalid` (número inexistente, correo que rebota, dato claramente falso) |
| Alias de nombre | Nombres declarados y nombres de perfil, con su fuente. El nombre vigente lo confirma el agente |
| `PersonLink` | Vínculo propuesto entre dos Persons: nivel de coincidencia, evidencia y estado (`proposed`, `confirmed`, `rejected`) |

Una Person tiene muchos identificadores. Un mismo identificador puede estar compartido por varias Persons (por ejemplo, una pareja) y entonces se marca como `shared`.

## 5. Niveles de coincidencia [Propuesta]

| Nivel | Criterio | Acción |
|---|---|---|
| `certain` | Misma identidad de canal (mismo número de WhatsApp o mismo identificador de Messenger) | Misma Person automáticamente |
| `probable` | Teléfono, correo o DNI declarados que coinciden exactamente con un identificador de una Person existente, tras normalizar, y que no son datos basura, compartidos ni de un agente | Se enlaza de forma provisional, se marca para confirmar y es reversible |
| `possible` | Solo nombre igual o parecido, o coincidencia parcial | Solo propuesta: ROCA crea una Task `decision`; no se fusiona |

El caso típico del portal: el lead declara un teléfono en el correo y luego escribe por WhatsApp desde ese mismo número. Es una coincidencia fuerte, porque el número declarado coincide con uno verificado por el canal.

El caso de la visita duplicada (Messenger y WhatsApp) no se puede resolver por identificadores si la persona nunca da su teléfono en Messenger. Por eso se propone, además:

- que ROCA pueda **preguntar por un dato de contacto** como parte de la calificación;
- que **antes de programar una visita se verifique** si hay otra abierta (regla I6).

## 6. Normalización y datos basura [Propuesta]

- **Teléfonos:** quitar espacios y signos, unificar el prefijo de país y el formato local.
- **Correos:** pasar a minúsculas.
- **Nombres:** ignorar mayúsculas y tildes y comparar sin importar el orden de las palabras. El umbral de parecido es [TBD].
- **Datos basura:** secuencias repetidas o evidentes, correos de relleno o valores que aparecen en muchos leads distintos se marcan como `invalid` o `shared` y no sirven para enlazar.
- **DNI:** dato personal sensible. Se guarda solo si llega y se usa solo como apoyo; no es identificador fuerte porque puede ser falso. Su tratamiento y retención: [TBD].

## 7. Casos especiales [Propuesta]

- **Agentes externos:** el teléfono de un agente aparece en leads de clientes distintos. Los identificadores de una Person clasificada como `agent` no enlazan automáticamente (`06` §8).
- **Teléfonos compartidos:** no enlazan por sí solos si están marcados `shared`.
- **Datos falsos:** un lead con datos inválidos no se descarta; se registra la invalidez como dato de calidad del portal o canal.
- **Un identificador nuevo en una conversación:** si la persona da su teléfono más tarde, ROCA vuelve a buscar coincidencias y propone fusionar si corresponde.

## 8. Fusión

- Al fusionar, se conservan todas las Conversations (siguen siendo un hilo por canal, `06`), todos los identificadores y todos los eventos.
- El agente ve una **línea de tiempo unificada** de la persona entre canales [Propuesta].
- La fusión es **reversible** (`person.unmerged`) y conserva el historial de lo ocurrido.
- Un vínculo rechazado por el agente se guarda para no volver a proponerlo [Propuesta].
- ROCA nunca revela a una persona información de otra. Las fusiones erróneas son un riesgo de privacidad, por eso solo `certain` es automático sin revisión.

## 9. Más allá de la Person: Opportunity y Visit [Propuesta]

La deduplicación debe evitar también duplicados de trabajo, no solo de personas.

- **Opportunity:** si una persona con una Opportunity `demand` abierta y compatible consulta por otro inmueble o por el mismo desde otro canal, la consulta se enlaza a la Opportunity existente (y se crea el vínculo `OpportunityProperty` si falta). Si hay varias abiertas o no es claro, ROCA propone a cuál pertenece.
- **Visit:** antes de crear una visita, ROCA verifica si existe otra abierta para el mismo inmueble con la misma persona o con una Person en posible duplicado, y avisa al agente.
- **Contacto saliente:** ROCA no inicia un primer contacto con una persona que ya tiene una conversación o visita abierta, bajo ningún nivel de coincidencia.

## 10. Leads de portal

- La entrada es el correo del portal, que se interpreta mediante un conector (P13). El formato varía por portal y puede cambiar: cada portal necesita su propia validación.
- Cada lead genera un evento `lead.received` con el portal, el inmueble de referencia, los datos declarados y el método de contacto elegido.
- Si la persona eligió WhatsApp o llamada, ROCA espera ese contacto y responde por ese canal.
- Los leads que solo vieron tus datos y no escribieron: si ROCA los contacta primero y cómo es [TBD], por las políticas de los canales y el contacto no solicitado.
- Cuando un número no existe en WhatsApp o un correo rebota, se registra `person.identifier_marked_invalid`. Con el tiempo esto mide la **calidad de los leads por portal**.

## 11. Eventos

El catálogo está en `04`. Principales: `lead.received`, `person.created`, `person.identifier_added`, `person.identifier_marked_invalid`, `person.match_proposed`, `person.match_confirmed`, `person.match_rejected`, `person.merged`, `person.unmerged`, `opportunity.inquiry_attached`, `visit.duplicate_warning`.

## 12. Reglas de integridad

- **I1.** Antes de crear una Person, ROCA busca coincidencias.
- **I2.** Una Person puede tener varios identificadores y un identificador puede estar compartido por varias Persons.
- **I3.** Solo el nivel `certain` enlaza sin revisión; `probable` es provisional y reversible; `possible` solo propone.
- **I4.** Toda fusión conserva conversaciones, identificadores y eventos, y es reversible.
- **I5.** Los datos declarados nunca se tratan como verificados.
- **I6.** Antes de programar una visita se comprueban visitas abiertas de la misma persona o de posibles duplicados para el mismo inmueble.
- **I7.** ROCA no inicia un primer contacto con una persona con conversación o visita abierta.
- **I8.** Una consulta nueva de una persona con una Opportunity abierta compatible se enlaza a ella; no crea una duplicada.
- **I9.** Los identificadores de agentes y los marcados `shared` o `invalid` no enlazan automáticamente.

## 13. Pendientes

- Confirmar el tratamiento de `probable`: enlace provisional o solo propuesta.
- Umbral de parecido de nombres.
- Tratamiento y retención del DNI y requisitos de protección de datos personales.
- Si ROCA contacta primero a leads de portal que solo vieron los datos.
- Validación técnica de la lectura de correos de cada portal.
- Cómo se presentan al agente los duplicados pendientes (carga de Tasks).
- Identificadores de otros canales futuros.

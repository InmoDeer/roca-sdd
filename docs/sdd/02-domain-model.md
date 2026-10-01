# 02 — Modelo de dominio

**Estado:** DRAFT v0.3  
**Última actualización:** 2026-10-01

## 1. Propósito

Este documento define los conceptos principales del dominio antes de convertirlos en tablas, APIs, componentes o automatizaciones.

El modelo todavía contiene decisiones pendientes. Los puntos no resueltos deben permanecer explícitos hasta ser definidos.

## 2. Entidades principales

### Person

Representa a una persona con la que ROCA tiene o tuvo interacción.

Una persona no tiene un único rol comercial permanente.

### Opportunity

Representa un contexto comercial concreto entre ROCA/agente y una persona.

Una oportunidad representa un objetivo o intención comercial concreta entre ROCA/agente y una persona. Puede representar, por ejemplo:

- mudarse a un departamento en alquiler;
- comprar un inmueble como inversión;
- captar un inmueble de un propietario;
- vender una propiedad.

Una persona puede tener múltiples oportunidades simultáneas cuando mantiene objetivos comerciales independientes.

**Regla central:** la continuidad de una Opportunity depende de la continuidad de su objetivo o intención comercial, no de que sus criterios o requerimientos permanezcan idénticos.

Por tanto:

- si cambia el requerimiento pero se mantiene el mismo objetivo, la Opportunity continúa y se genera una nueva versión de Requirement;
- si cambia el objetivo o intención comercial, debe cerrarse la Opportunity anterior y, cuando corresponda, crear una nueva;
- si la persona mantiene el objetivo anterior y además desarrolla otro objetivo independiente, ambos pueden permanecer como Opportunities simultáneas.

Ejemplos:

- “Quiero alquilar un departamento para vivir” → “sigo queriendo mudarme, pero ahora busco otra zona y tengo mayor presupuesto”: misma Opportunity, nueva versión de Requirement.
- “Quiero comprar un departamento para invertir” → “ahora prefiero comprar una oficina para invertir”: puede continuar la misma Opportunity, porque la intención de inversión se mantiene; cambia el Requirement.
- “Quiero mudarme a un departamento” → “ya no quiero mudarme; prefiero quedarme donde estoy e invertir”: se cierra la primera Opportunity con un motivo que refleje cambio de objetivo y se crea una nueva Opportunity para la inversión.
- “Quiero mudarme a un departamento” y, además, “quiero mudar mi negocio a un nuevo local”: son dos objetivos comerciales independientes y pueden existir dos Opportunities abiertas simultáneamente.
- “Quiero mudarme a un departamento” y, además, “quiero comprar un almacén”: pueden existir dos Opportunities simultáneas si representan procesos comerciales independientes, aunque pertenezcan a la misma persona.

La existencia de una nueva Opportunity no implica que la anterior deba cerrarse si la persona continúa persiguiendo el objetivo anterior.

### Requirement

Representa lo que una persona necesita, busca, prefiere o rechaza dentro de una Opportunity.

Es una entidad dinámica y versionable. Una Opportunity puede tener múltiples versiones de Requirement a lo largo de su vida, conservando el historial de cambios y un Requirement actual.

**Regla:** un Requirement representa la versión vigente de las condiciones para alcanzar el objetivo de su Opportunity. No debe utilizarse para agrupar objetivos comerciales independientes que ocurren simultáneamente.

Puede incluir, según corresponda:

- tipo de operación;
- uno o varios tipos de propiedad;
- presupuesto;
- zonas, distritos o puntos específicos de interés;
- dormitorios;
- cantidad de ambientes;
- metraje;
- cochera;
- mascotas;
- zonificación;
- características comerciales;
- preferencias;
- restricciones;
- otros criterios dependientes del tipo de operación o propiedad.

No todos los campos aplican a todos los requerimientos. Una búsqueda de local comercial puede depender de metraje, ubicación y zonificación, mientras una búsqueda residencial puede depender de dormitorios y mascotas.

Los criterios deberán distinguir, cuando sea relevante, entre:

- obligatorio;
- preferido;
- flexible;
- excluido.

Los cambios no deben destruir las versiones anteriores.

### Property

Representa un inmueble conocido por ROCA.

El inmueble puede atravesar distintas etapas antes y después de estar formalmente captado.

### Conversation

Representa una conversación o hilo de comunicación con una persona a través de un canal.

Una conversación puede existir antes de una oportunidad formal.

### Visit

Representa una visita asociada a una oportunidad.

Puede estar vinculada a un inmueble cuando corresponda.

### Activity / Event

Representa un hecho ocurrido en el sistema o en el proceso comercial.

Ejemplos:

- mensaje recibido;
- mensaje enviado;
- llamada;
- visita programada;
- visita realizada;
- nota agregada;
- inmueble actualizado;
- cambio de precio;
- seguimiento realizado;
- publicación aprobada;
- publicación publicada.

### Task

Representa una acción pendiente que requiere ejecución o supervisión.

### Publication

Representa una publicación de un inmueble en un canal o destino determinado.

Preparar una publicación y publicarla son estados/acciones diferentes.

### Agent / User / Organization

Representan a los actores que utilizan ROCA y la futura estructura organizacional.

El modelo multiagente debe permitir evolucionar desde el agente independiente hacia equipos e inmobiliarias.

## 3. Relaciones conceptuales actuales

- Person 1:N Opportunity
- Opportunity 1:N Requirement
- Opportunity 1:N Conversation
- Opportunity N:M Property
- Opportunity 1:N Visit
- Visit N:1 Property (opcional)
- Property 1:N Publication
- Opportunity 1:N Activity/Event
- Property 1:N Activity/Event

Las relaciones exactas y sus restricciones de integridad deberán formalizarse posteriormente.

## 4. Invariantes preliminares

1. Una persona puede tener múltiples oportunidades simultáneas.
2. Una persona puede tener diferentes roles en diferentes oportunidades.
3. Una oportunidad representa un objetivo/intención comercial, no un inmueble específico.
4. Una oportunidad puede tener múltiples versiones de Requirement.
5. Una oportunidad tiene un Requirement vigente cuando el tipo de oportunidad requiere criterios de búsqueda/condiciones comerciales.
6. Un cambio de Requirement no implica automáticamente una nueva Opportunity.
7. Si cambia el objetivo/intención comercial, debe evaluarse el cierre de la Opportunity anterior y la creación de una nueva.
8. Dos objetivos comerciales independientes de una misma persona pueden tener Opportunities abiertas simultáneamente.
9. Una oportunidad puede existir sin inmueble asociado.
10. Una oportunidad puede estar asociada a múltiples inmuebles.
11. Un inmueble puede estar relacionado con múltiples oportunidades.
12. Una conversación puede existir antes de una oportunidad formal.
13. El origen histórico de una conversación no debe sobrescribir el origen histórico de la persona u oportunidad.
14. Una visita pertenece a una oportunidad.
15. El resultado de una visita es independiente del estado final de la oportunidad.
16. Una propiedad se considera captada cuando existe autorización para publicarla, según la regla de negocio actual.
17. Una publicación preparada no implica que haya sido publicada.
18. El cierre de una oportunidad debe conservar un resultado y motivo de cierre.
19. Los cambios relevantes de Requirement e intención deben poder reconstruirse mediante historial/eventos.
20. El historial de una Opportunity no debe perderse porque posteriormente se cree otra Opportunity para un objetivo diferente de la misma persona.

## 5. Flujos conceptuales actuales

### Propietario

Contacto → Propuesta/Tasación → Seguimiento → Visita → Captación/Cierre o Descartado

La captación efectiva se produce cuando existe autorización para publicar.

Antes de la visita, ROCA puede recopilar información disponible y preparar la propuesta.

La visita puede requerir que ROCA prepare al agente una lista de preguntas o datos pendientes.

### Comprador / Arrendatario

Interesado → Seguimiento → Visita → Post-visita → Cerrado o Descartado

ROCA puede responder preguntas, calificar interés, hacer seguimiento rutinario y escalar situaciones que requieran intervención del agente.

## 6. Demanda y evolución del Requirement

La necesidad de una persona no debe perderse aunque cambie durante el proceso.

ROCA debe poder reconstruir qué buscaba originalmente, qué cambios declaró, cuándo cambió, qué motivo declaró cuando esté disponible, qué inmuebles había considerado y qué Requirement está vigente actualmente.

Un cambio puede ser pequeño, como aumentar el presupuesto, o grande sin cambiar la intención, como pasar de comprar un departamento para inversión a comprar una oficina para inversión.

Un cambio puede ser suficientemente profundo como para representar una nueva Opportunity cuando cambia el objetivo comercial, por ejemplo pasar de alquilar un departamento para vivir a comprar un inmueble para inversión.

Cuando una persona mantiene el objetivo anterior y añade otro objetivo independiente, no se debe sobrescribir ni reemplazar el Requirement ni la Opportunity anteriores. Se crea una Opportunity adicional para el nuevo objetivo.

**Regla exacta para determinar cuándo una modificación constituye una nueva Opportunity: pendiente de formalización mediante la máquina de estados y taxonomía de objetivos.**

## 7. Opportunity ↔ Property

Si una persona expresa una necesidad y todavía no existe un inmueble adecuado, la necesidad no debe perderse.

Debe existir un mecanismo para conservar:

- qué busca;
- condiciones conocidas;
- preferencias;
- restricciones;
- inmuebles vistos o considerados;
- evolución de la necesidad.

La presencia de inmuebles asociados no elimina ni reemplaza el Requirement activo.

Un nuevo inmueble puede aparecer posteriormente y ser evaluado contra Requirements activos de Opportunities abiertas.

**Modelo exacto de relación Opportunity–Property: TBD.**

## 8. Relación Opportunity ↔ Property

Actualmente se considera una relación N:M.

No basta necesariamente con saber que una oportunidad está relacionada con un inmueble. La relación puede necesitar registrar contexto, por ejemplo:

- recomendado;
- visto;
- visitado;
- interesado;
- rechazado;
- rechazado por precio;
- descartado por características;
- seleccionado/shortlist.

La relación debe permitir conservar la historia de cómo evolucionó el vínculo entre una Opportunity y cada Property.

**Modelo exacto: TBD.**

Se evaluará una entidad explícita equivalente a OpportunityProperty.

## 9. Propiedad preliminar

Puede existir información sobre un inmueble antes de que esté formalmente captado.

Por ejemplo, puede conocerse mediante:

- información enviada por un propietario;
- fotografías;
- descripción;
- enlace a una publicación;
- información obtenida durante una conversación.

**Reglas y estado formal de una propiedad preliminar: TBD.**

## 10. Recomendaciones futuras y memoria comercial

Los Requirements activos pueden utilizarse para encontrar inmuebles compatibles.

El historial de Requirements y las interacciones con inmuebles pueden servir posteriormente como información para recomendaciones.

Debe distinguirse entre información declarada explícitamente por la persona, comportamiento observado, información derivada de eventos, inferencias realizadas por ROCA y preferencias confirmadas.

ROCA no debe convertir automáticamente una inferencia en un hecho de la persona.

El uso de históricos para recomendaciones entre personas diferentes requerirá suficiente evidencia y reglas específicas que se definirán posteriormente.

## 11. Decisiones pendientes

Antes de implementar el modelo de datos deben definirse formalmente:

- máquina de estados de Opportunity;
- taxonomía de objetivos/intenciones de Opportunity;
- estados, resultados y motivos de cierre de Opportunity;
- máquina de estados de Property;
- máquina de estados de Visit;
- modelo de eventos;
- permisos y ownership;
- modelo multiagente;
- campos mínimos de cada entidad;
- identidad y deduplicación de personas;
- reglas de archivo y eliminación;
- contratos entre módulos;
- relación exacta Opportunity–Property;
- modelo de necesidades/demanda;
- definición de fuente/origen por entidad.

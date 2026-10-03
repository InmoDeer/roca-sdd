# 02 — Modelo de dominio

**Estado:** DRAFT v0.5  
**Última actualización:** 2026-10-02

## 1. Propósito

Este documento define los conceptos principales del dominio antes de convertirlos en tablas, APIs, componentes o automatizaciones.

El modelo todavía contiene decisiones pendientes. Los puntos no resueltos deben permanecer explícitos hasta ser definidos.

## 2. Entidades principales

### Person

Representa a una persona con la que ROCA tiene o tuvo interacción.

Una persona no tiene un único rol comercial permanente.

Un propietario es **cliente** (quien contrata y paga los servicios de ROCA/agente). Un interesado es **cliente potencial** y puede convertirse después en propietario o en demandante con otro objetivo.

Un agente externo que colabora en una operación o la cierra por su cuenta es también una Person (con rol de agente), no un User de ROCA.

Una persona puede tener varios identificadores (teléfono, correo, DNI, identificadores de canal) y se busca antes de crear una nueva. Modelo en `07-identity.md`.

### Opportunity

Representa un contexto comercial concreto entre ROCA/agente y una persona.

Toda Opportunity pertenece a un **lado del mercado** (`demand` u oferta/`supply`) y tiene un **objetivo** principal. La **operación** (compra, alquiler, venta, o indiferente) no es atributo de la Opportunity: pertenece al Requirement. Ver `03-state-machines.md` §2.

Una oportunidad `supply` está asociada a exactamente un inmueble: un propietario con dos inmuebles tiene dos oportunidades.

Una oportunidad `demand` puede originarse a través de un agente externo (intermediario) que contacta con un cliente por un inmueble del agente [Decidido el principio; modelo en `06-conversations.md` §8].

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

Es un hilo continuo por persona y canal, y puede enlazarse a varias oportunidades [Decidido]. Modelo en `06-conversations.md`.

### Visit

Representa una visita asociada a una oportunidad.

Normalmente está vinculada a un inmueble (ver `03` §6). Un recorrido con varios inmuebles se modela como varias visitas agrupadas [Propuesta].

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

### Task [Propuesta]

Unidad de trabajo pendiente para el agente: contactar, decidir, registrar un resultado, completar información o recordar algo. ROCA las crea a partir de eventos y reglas; el agente las completa, pospone o descarta. Las Visits no generan Tasks: aparecen en la agenda vinculadas. Modelo en `05-agenda.md`.

### ServiceAgreement [Propuesta]

Representa el acuerdo por el cual un propietario accede a los servicios de ROCA/agente para comercializar un inmueble. Su aceptación marca la **captación**.

Incluye, según corresponda: propietario, inmueble, fecha de aceptación, operaciones cubiertas (venta, alquiler o ambas), comisión pactada por operación, nivel de exclusividad y vigencia. Aceptar el acuerdo no implica publicación automática: cada publicación sigue requiriendo aprobación del agente.

**Comisión pactada** (estándar entre paréntesis):

- Venta: porcentaje del precio de venta (3%).
- Alquiler: cantidad de meses de renta, con fracciones (1 mes; también medio mes, 2 meses, etc.).

Si la operación es indiferente (`either`), el acuerdo guarda ambas comisiones. Se registra si el valor pactado es el estándar o fue negociado. **La comisión la paga el propietario [Decidido].** 

**Cuándo se devenga [Decidido]:** la comisión se devenga cuando la operación la concluye el agente. Si la concluye el propietario (semi-exclusividad) u otro agente (sin exclusividad), no se cobra, porque esa otra parte también estaba autorizada para promocionar el inmueble. En exclusividad, si cierra el propietario u otro agente durante la vigencia, el agente cobra por defecto el 100% de la comisión; el porcentaje puede variar según el acuerdo [Decidido].

**Vigencia [Decidido, valores por defecto]:** 3 meses para alquiler y 6 meses para venta, variable según el acuerdo. Se guarda por operación, igual que la comisión [Propuesta]. La vigencia del acuerdo es independiente de la duración del contrato de alquiler.

**Reparto con otros agentes [Decidido]:** según acuerdo interno entre agentes, caso por caso; por defecto 50/50. Se registra en la Operation. Los agentes externos se registran como Person (rol agente) en el CRM [Decidido]: un reparto referencia solo Persons ya registradas, y si falta el registro se crea antes (ROCA lo propone).

**Exclusividad [Decidido]:**

- Sin exclusividad: cualquier agente puede promocionar el inmueble.
- Semi-exclusividad: pueden promocionarlo el agente y el propietario.
- Exclusividad: solo el agente.

Modelo exacto: TBD.

### Operation [Propuesta]

Representa una operación concluida (venta o alquiler): inmueble, partes, monto y, en alquileres, fechas de inicio y fin del contrato.

Enlaza la Opportunity `supply` del propietario y la `demand` de la contraparte. Es necesaria para el resultado `won`, para anticipar el fin de contrato de un alquiler y para las estadísticas.

Registra también la comisión aplicada (copiada del acuerdo al cerrar), su monto, lo efectivamente cobrado, el reparto con otros agentes si participaron (por defecto 50/50) y quién concluyó la operación (agente, propietario u otro agente) [Propuesta].

Modelo exacto: TBD.

### Agent / User / Organization

Representan a los actores que utilizan ROCA y la futura estructura organizacional.

El modelo multiagente debe permitir evolucionar desde el agente independiente hacia equipos e inmobiliarias.

## 3. Relaciones conceptuales actuales

- Person 1:N Opportunity
- Opportunity 1:N Requirement
- Opportunity N:M Conversation (un hilo continuo por persona y canal puede tratar varias oportunidades)
- Opportunity `demand` N:M Property (vía OpportunityProperty)
- Opportunity `supply` N:1 Property (una oportunidad por inmueble)
- Operation N:1 Property [Propuesta]
- Operation N:M Opportunity (la de oferta y la de demanda que la concluyen) [Propuesta]
- ServiceAgreement N:1 Property [Propuesta]
- ServiceAgreement N:1 Opportunity `supply` [Propuesta]
- Opportunity 1:N Visit
- Visit N:1 Property (normalmente obligatoria, ver `03` §6)
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
16. Un inmueble se considera captado cuando el propietario accede a los servicios (acuerdo de servicios). Puede ocurrir antes, durante o después de una visita.
17. Una publicación preparada no implica que haya sido publicada.
18. El cierre de una oportunidad debe conservar un resultado y motivo de cierre.
19. Los cambios relevantes de Requirement e intención deben poder reconstruirse mediante historial/eventos.
20. El historial de una Opportunity no debe perderse porque posteriormente se cree otra Opportunity para un objetivo diferente de la misma persona.
21. Toda Opportunity tiene un lado (`demand` u oferta/`supply`).
22. Una Opportunity `supply` tiene exactamente un inmueble; un propietario con dos inmuebles tiene dos Opportunities.
23. La operación (compra, alquiler, venta, indiferente) pertenece al Requirement. Una operación indiferente es una sola Opportunity.
24. La continuidad de una Opportunity depende de su lado y su objetivo, no de su operación ni de sus criterios.
25. Una Opportunity `won` referencia la operación concluida. [Propuesta]
26. Un acuerdo de servicios guarda la comisión pactada por tipo de operación: porcentaje en venta y meses de renta en alquiler. [Propuesta]
27. Todo acuerdo de servicios tiene un nivel de exclusividad: sin exclusividad, semi-exclusividad o exclusividad. El propietario paga la comisión.
28. En sin exclusividad y semi-exclusividad no hay comisión si la operación la concluye el propietario u otro agente.
29. El reparto de comisión con otros agentes se acuerda caso a caso; por defecto 50/50.
30. En exclusividad, si cierra un tercero durante la vigencia, la comisión se devenga (por defecto 100%, variable).
31. Los agentes externos son Persons registradas; un reparto de comisión referencia solo Persons existentes.
32. Una Opportunity `demand` puede tener un agente intermediario (Person con rol de agente). [Propuesta]

## 5. Flujos conceptuales actuales

### Propietario

Contacto → Propuesta/Tasación → Seguimiento → Captación → Comercialización → Cierre o Descartado

La captación se produce cuando el propietario accede a los servicios (acuerdo de servicios). Puede ocurrir antes, durante o después de una visita.

Las visitas pueden ser para conversar (el propietario puede decir que lo pensará) o para preparar material audiovisual y datos.

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

## 7. Necesidad sin inmueble disponible

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

Esta relación aplica a oportunidades `demand` y se considera N:M. En oportunidades `supply` el inmueble es el sujeto de la oportunidad (exactamente uno) y no se modela como candidato.

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

**Modelo exacto:** borrador en `03` §7 (entidad `OpportunityProperty`), pendiente de confirmación.

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
- confirmación del modelo de identidad y deduplicación (`07-identity.md`);
- reglas de archivo y eliminación;
- contratos entre módulos;
- relación exacta Opportunity–Property;
- modelo de necesidades/demanda;
- definición de fuente/origen por entidad;
- taxonomía de objetivos y valores de `side`;
- registro de canales (los canales son datos configurables, ver P13);
- modelo de Operation (operación concluida, contrato, fechas, partes);
- modelo de ServiceAgreement (vigencia por operación, porcentaje devengado en exclusividad cuando cierra un tercero);
- detalle de la regla de reapertura (la reapertura sobre la misma Opportunity, solo si la persona retoma su objetivo, está decidida en `03` §5).

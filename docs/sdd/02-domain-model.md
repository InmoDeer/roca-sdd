# 02 — Modelo de dominio

**Estado:** DRAFT v0.1  
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

Una oportunidad puede representar, por ejemplo:

- interés de compra;
- interés de alquiler;
- captación de un propietario;
- otra relación comercial que posteriormente se defina.

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
3. Una oportunidad puede existir sin inmueble asociado.
4. Una oportunidad puede estar asociada a múltiples inmuebles.
5. Un inmueble puede estar relacionado con múltiples oportunidades.
6. Una conversación puede existir antes de una oportunidad formal.
7. El origen histórico de una conversación no debe sobrescribir el origen histórico de la persona u oportunidad.
8. Una visita pertenece a una oportunidad.
9. El resultado de una visita es independiente del estado final de la oportunidad.
10. Una propiedad se considera captada cuando existe autorización para publicarla, según la regla de negocio actual.
11. Una publicación preparada no implica que haya sido publicada.
12. El cierre de una oportunidad debe conservar un resultado de cierre.

## 5. Flujos conceptuales actuales

### Propietario

Contacto → Propuesta/Tasación → Seguimiento → Visita → Captación/Cierre o Descartado

La captación efectiva se produce cuando existe autorización para publicar.

Antes de la visita, ROCA puede recopilar información disponible y preparar la propuesta.

La visita puede requerir que ROCA prepare al agente una lista de preguntas o datos pendientes.

### Comprador / Arrendatario

Interesado → Seguimiento → Visita → Post-visita → Cerrado o Descartado

ROCA puede responder preguntas, calificar interés, hacer seguimiento rutinario y escalar situaciones que requieran intervención del agente.

## 6. Demanda sin inmueble

Si una persona expresa una necesidad y todavía no existe un inmueble adecuado, la necesidad no debe perderse.

Debe existir un mecanismo para conservar:

- qué busca;
- condiciones conocidas;
- preferencias;
- restricciones;
- inmuebles vistos o considerados;
- evolución de la necesidad.

**Modelo exacto: TBD.**

Debe evaluarse si esta información pertenece directamente a Opportunity o requiere una entidad/relación explícita de interés.

## 7. Opportunity ↔ Property

Actualmente se considera una relación N:M.

No basta necesariamente con saber que una oportunidad está relacionada con un inmueble. La relación puede necesitar registrar contexto, por ejemplo:

- recomendado;
- visto;
- visitado;
- interesado;
- rechazado;
- rechazado por precio;
- descartado por características.

**Modelo exacto: TBD.**

Se evaluará una entidad explícita equivalente a PropertyInterest u OpportunityProperty.

## 8. Propiedad preliminar

Puede existir información sobre un inmueble antes de que esté formalmente captado.

Por ejemplo, puede conocerse mediante:

- información enviada por un propietario;
- fotografías;
- descripción;
- enlace a una publicación;
- información obtenida durante una conversación.

**Reglas y estado formal de una propiedad preliminar: TBD.**

## 9. Decisiones pendientes

Antes de implementar el modelo de datos deben definirse formalmente:

- máquina de estados de Opportunity;
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

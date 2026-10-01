# 03 — Máquinas de estado

**Estado:** DRAFT v0.1  
**Última actualización:** 2026-10-01

## 1. Propósito

Este documento define las máquinas de estado conceptuales de ROCA.

El objetivo es separar:

- estado actual del proceso;
- objetivo/intención comercial;
- Requirement vigente;
- resultado del proceso;
- motivo de cierre;
- eventos que provocan transiciones.

Una modificación de Requirement no debe confundirse con una transición de Opportunity.

## 2. Opportunity

### 2.1 Concepto

Una Opportunity representa la continuidad de un objetivo o intención comercial.

Su estado describe en qué punto del proceso comercial se encuentra, no qué propiedades específicas está buscando.

El Requirement describe las condiciones actuales para alcanzar ese objetivo.

### 2.2 Estados conceptuales

Para compradores/arrendatarios:

**Interesado → Seguimiento → Visita → Post-visita → Cerrado / Descartado**

Para propietarios/captación:

**Contactado → Propuesta/Tasación → Seguimiento → Visita → Captación/Cerrado / Descartado**

Estos flujos son una primera representación. La máquina formal deberá permitir variantes según tipo de Opportunity sin duplicar el concepto de Opportunity.

### 2.3 Significado de los estados

#### Interesado / Contactado

Existe una intención comercial identificada y la oportunidad puede ser trabajada.

Todavía no necesariamente existe suficiente información para avanzar al siguiente paso.

#### Propuesta / Tasación

Aplica principalmente a oportunidades de propietarios.

ROCA/agente está presentando o preparando la propuesta de servicio, valoración o condiciones para avanzar hacia la captación.

#### Seguimiento

La oportunidad está activa y requiere acciones posteriores.

Puede contener múltiples ciclos de contacto, espera, respuesta, seguimiento automatizado o intervención del agente.

No significa necesariamente que el interés sea bajo.

#### Visita

La visita es el siguiente hito comercial relevante o está siendo gestionada dentro del proceso.

La fecha y hora exactas deben respetar las reglas de autonomía de ROCA. ROCA puede recopilar disponibilidad y proponer/solicitar información, pero el agente conserva la decisión final cuando corresponda.

#### Post-visita

La visita ocurrió y todavía debe registrarse, procesarse o dar seguimiento a su resultado comercial.

El resultado de la visita es independiente del resultado final de la Opportunity.

#### Captación / Cerrado

La oportunidad alcanzó su resultado comercial satisfactorio.

Para una oportunidad de captación, la regla vigente es que el inmueble queda captado cuando existe autorización para publicar.

Para una oportunidad de comprador/arrendatario, el cierre satisfactorio representa la conclusión del proceso comercial definido para esa oportunidad.

#### Descartado

La Opportunity deja de estar activa sin alcanzar el resultado comercial satisfactorio.

El descarte debe conservar un motivo estructurado cuando sea posible.

### 2.4 Resultado y motivo de cierre

El estado terminal por sí solo no es suficiente.

Toda Opportunity cerrada o descartada debe conservar, cuando corresponda:

- resultado;
- motivo de cierre;
- fecha;
- actor o evento que produjo el cierre;
- contexto disponible.

Los motivos deben permitir distinguir situaciones diferentes, por ejemplo:

- objetivo cambiado;
- necesidad desaparecida;
- operación cerrada;
- inmueble captado;
- encontró otra alternativa;
- no hubo inmueble adecuado;
- no acepta condiciones;
- dejó de responder;
- descartado por criterios;
- otro motivo.

**La taxonomía definitiva está pendiente.**

### 2.5 Cambio de objetivo

Un cambio de objetivo no debe registrarse simplemente como una modificación del Requirement.

Ejemplo:

1. Persona quiere mudarse a un departamento.
2. Se crea Opportunity A.
3. La persona decide que ya no quiere mudarse y prefiere permanecer donde está para invertir.
4. Opportunity A se cierra con motivo equivalente a objective_changed.
5. Se crea Opportunity B para la inversión.

El historial de A no se elimina ni se transforma en B.

### 2.6 Objetivos simultáneos

Si la persona mantiene el objetivo original y añade otro objetivo independiente:

1. Opportunity A continúa abierta.
2. Se crea Opportunity B.
3. Cada Opportunity conserva su propio Requirement, estado, eventos, visitas y resultado.

Ejemplo:

- A: mudarse a un departamento.
- B: trasladar su negocio a un nuevo local.

También puede existir:

- C: comprar un almacén.

No debe existir una única Opportunity con Requirements paralelos si estos representan procesos comerciales independientes.

## 3. Requirement y Opportunity

### Regla fundamental

**Requirement cambia cuando cambian las condiciones del mismo objetivo.**

**Opportunity cambia cuando cambia el objetivo o aparece un objetivo comercial independiente.**

Ejemplos:

| Situación | Opportunity | Requirement |
|---|---|---|
| Cambia presupuesto | misma | nueva versión |
| Cambia distrito | misma | nueva versión |
| Cambia dormitorios | misma | nueva versión |
| Departamento → oficina manteniendo objetivo de inversión | misma | nueva versión |
| Mudarse → invertir en lugar de mudarse | nueva | nuevo |
| Mudarse + además trasladar negocio | nueva adicional | nuevo |
| Mudarse + además comprar almacén | nueva adicional | nuevo |

Esta tabla es conceptual y deberá convertirse en reglas verificables.

## 4. Transiciones preliminares

### Comprador / arrendatario

- created → interesado
- interesado → seguimiento
- seguimiento → visita
- visita → post_visita
- post_visita → seguimiento
- post_visita → cerrado
- interesado → descartado
- seguimiento → descartado
- visita → descartado
- post_visita → descartado

### Propietario

- created → contactado
- contactado → propuesta_tasacion
- propuesta_tasacion → seguimiento
- seguimiento → visita
- visita → captacion_cerrado
- contactado → descartado
- propuesta_tasacion → descartado
- seguimiento → descartado
- visita → descartado

Las transiciones exactas y las condiciones de entrada/salida deberán formalizarse.

## 5. Transiciones que requieren cuidado

### Cambio de Requirement

No cambia necesariamente el estado.

Ejemplo:

Seguimiento + Requirement v1 → Seguimiento + Requirement v2

El cambio debe generar historial/evento.

### Cambio de objetivo

No se debe convertir automáticamente el Requirement existente.

Ejemplo:

Opportunity A abierta → Opportunity A cerrada (objective_changed) → Opportunity B creada

### Nuevo objetivo paralelo

No se modifica la Opportunity existente.

Ejemplo:

Opportunity A abierta → Opportunity A abierta + Opportunity B creada

## 6. Visita

La Visit tendrá su propia máquina de estado y no debe reutilizar automáticamente el estado de Opportunity.

Preliminarmente:

**Pendiente/Programación → Programada → Realizada / Cancelada / No realizada**

Una visita realizada puede tener un resultado comercial favorable o desfavorable sin que eso determine automáticamente el cierre de la Opportunity.

La máquina de Visit se definirá en detalle posteriormente.

## 7. Reglas de integridad

1. Una Opportunity terminal no debe volver a un estado activo sin una regla explícita de reapertura.
2. Crear una nueva versión de Requirement no debe crear automáticamente una nueva Opportunity.
3. Crear una nueva Opportunity no debe borrar Requirements ni eventos de Opportunities anteriores.
4. Una Opportunity nueva debe representar un objetivo independiente o un cambio de objetivo.
5. Una persona puede tener múltiples Opportunities activas simultáneamente.
6. El cierre de una Opportunity debe conservar el motivo cuando sea conocido.
7. Los eventos deben permitir reconstruir por qué una Opportunity cambió de estado.
8. Una visita no equivale automáticamente a un cierre.
9. Un inmueble recomendado o visitado no determina por sí solo el estado de la Opportunity.
10. ROCA no debe inferir un cambio de objetivo únicamente por cambios en criterios si la persona no lo ha declarado o si no existe una regla explícita que lo permita.

## 8. Pendientes

Antes de implementar la máquina de estados deben definirse:

- taxonomía formal de tipos/objetivos de Opportunity;
- estados definitivos por tipo de Opportunity;
- estados terminales y reapertura;
- taxonomía de resultados;
- taxonomía de motivos de cierre;
- condiciones de cada transición;
- eventos que producen cada transición;
- quién puede ejecutar cada transición;
- automatizaciones permitidas;
- máquina de estados de Visit;
- máquina de estados de Property;
- relación entre estado de Opportunity y estado de OpportunityProperty.

# 01 — Principios

**Estado:** DRAFT v0.2  
**Última actualización:** 2026-10-02

Estos principios orientan las decisiones de producto, dominio, arquitectura e IA de ROCA.

## P1 — El SDD es la fuente de verdad

Las decisiones importantes deben quedar reflejadas en el SDD.

El código implementa la especificación; no debe ser el lugar donde una regla de negocio aparezca por primera vez de forma accidental.

## P2 — Datos y eventos antes que IA

La inteligencia de ROCA depende de información estructurada y de un historial fiable.

Primero deben existir:

- entidades claras;
- relaciones claras;
- estados;
- eventos;
- permisos;
- trazabilidad.

Después se automatiza e interpreta con IA.

## P3 — Una única fuente de verdad del dominio

Las distintas capacidades de ROCA no deben mantener copias independientes de personas, inmuebles u oportunidades.

Copilot, Sales, Customer, Property y Marketing son capacidades sobre el mismo dominio y los mismos datos.

## P4 — Persona no es oportunidad

Una persona puede tener varias oportunidades simultáneas y diferentes roles en ellas.

El rol comercial pertenece al contexto de la oportunidad, no define permanentemente a la persona.

## P5 — Una conversación puede existir antes que una oportunidad

ROCA debe poder registrar una conversación desde el primer contacto, incluso antes de que exista una oportunidad formal.

El origen y canal del contacto deben conservarse históricamente.

## P6 — Una oportunidad no requiere necesariamente un inmueble

Una persona puede expresar una necesidad sin que exista todavía un inmueble vinculado.

ROCA debe conservar esa demanda para posteriores recomendaciones, cooperación con otros agentes y decisiones de captación.

## P7 — Visita y resultado comercial son conceptos diferentes

Una visita puede realizarse y tener un resultado positivo sin que la oportunidad esté cerrada.

El resultado de la visita y el resultado final de la oportunidad deben poder registrarse independientemente.

## P8 — Automatización con límites explícitos

La automatización debe estar gobernada por reglas claras.

Una acción no es autónoma simplemente porque técnicamente pueda ejecutarse.

## P9 — El agente conserva las decisiones comerciales críticas

ROCA puede preparar, recomendar, recordar, clasificar y ejecutar acciones autorizadas.

No debe asumir decisiones comerciales que requieran criterio o autorización del agente.

## P10 — Trazabilidad

Cuando sea razonablemente posible, las acciones relevantes deben dejar un registro de:

- qué ocurrió;
- cuándo ocurrió;
- sobre qué entidad ocurrió;
- quién o qué lo produjo;
- cuál fue el resultado.

## P11 — Evolución incremental

Si una regla todavía no está suficientemente definida, debe marcarse explícitamente como TBD o TBC.

No se debe inventar una decisión solamente para completar documentación.

## P12 — Evitar sobreingeniería prematura

El diseño inicial debe resolver correctamente el dominio y permitir evolución, pero no introducir infraestructura o abstracciones complejas sin una necesidad demostrada.

La extensibilidad debe lograrse principalmente mediante buenos límites de dominio y contratos claros.

## P13 — Integraciones mediante conectores

El Core de ROCA no se acopla directamente a APIs de proveedores externos.

Toda integración (mensajería, portales, publicidad, almacenamiento, IA, etc.) pasa por una capa de conectores:

ROCA Core → capa de integración/conectores → proveedor externo

Los canales y proveedores pueden agregarse, modificarse, actualizarse o eliminarse sin cambiar el dominio. Por tanto, los canales se tratan como datos (registro de canales), no como una lista fija en el código.

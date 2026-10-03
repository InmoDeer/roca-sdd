# Contexto de traspaso — ROCA

**Estado:** DRAFT v0.2  
**Última actualización:** 2026-10-02

Este documento conserva contexto conversacional que **no está** en `00` a `04`. Es una versión condensada del traspaso generado por la IA anterior, corregida por el autor del SDD el 2026-10-02.

Aviso: las citas textuales provienen del traspaso original y no han sido verificadas contra las conversaciones. Verificarlas antes de tratarlas como literales.

Etiquetas: **A** decidido por el autor; **B** discutido sin decidir; **C** propuesto por una IA y no confirmado; **D** descartado.

## 1. Negocio y operación

**A**

- ROCA nace de una operación inmobiliaria real en Perú, principalmente Lima. El autor es agente inmobiliario independiente desde hace 8 años.
- Su trabajo: buscar y captar inmuebles, hablar con propietarios, publicar, recibir consultas, filtrar interesados, coordinar visitas, hacer seguimientos, cambiar precios y preparar copys.
- Canales y fuentes usados de verdad: Facebook Marketplace, Urbania, Adondevivir, Babilonia, WhatsApp, Messenger, páginas de Facebook/Instagram, leads de portales, Google Drive, Google Photos, notas, portapapeles y memoria.
- Problema central: pérdida de continuidad operativa, no ausencia de CRM. Cita original: *"me pierdo, nunca recuerdo visitas ni seguimientos, si no hago algo al momento se me olvida"*.
- ROCA nació como solución al **catálogo** de inmuebles (información dispersa). El CRM apareció después. Ambos comparten el mismo núcleo de datos; el catálogo no pierde importancia.
- **Visión:** ROCA es la agencia detrás del agente: asume el trabajo operativo para que el agente solo se ocupe de lo que genera dinero (captar, agendar y realizar visitas, negociar y cerrar). Prueba útil para cada función: ¿sirve a ese ciclo o evita perder un lead?
- **Cliente:** el propietario es el cliente de ROCA (quien contrata y paga). El interesado es cliente potencial y puede pasar a ser propietario o tener otro objetivo. *(Corrige el término "proveedor" del traspaso original.)*
- **Captado:** el propietario aceptó trabajar con ROCA/agente (no se exige una autorización de publicación aparte). El propietario paga la comisión. Niveles de exclusividad: sin exclusividad (todos pueden promocionar), semi-exclusividad (el agente y el propietario) y exclusividad (solo el agente). Puede ocurrir antes, durante o después de una visita. Las visitas pueden ser para conversar (el propietario puede decir que lo pensará) o para preparar material audiovisual y datos.
- **Comisión estándar:** 3% en venta; en alquiler, 1 mes de renta (también medio mes, 2 meses, etc.). No hay comisión si la operación la cierra el propietario (semi-exclusividad) u otro agente (sin exclusividad), porque esa parte también estaba autorizada para promocionar. En exclusividad, si cierra un tercero durante la vigencia, cobra el 100% de la comisión por defecto (variable). Reparto con otros agentes: acuerdo interno caso a caso, por defecto 50/50; los agentes externos se registran como personas en el CRM.
- **Vigencia del acuerdo:** variable; por defecto 3 meses en alquiler y 6 meses en venta. Al acercarse el vencimiento ROCA avisa y el agente decide seguir (renovar) o archivar.

**B** (no asumir como parte de V1): LinkedIn, YouTube, Telegram, Instagram como canal integrado, Gmail, Meta Ads, TikTok Ads, Google Ads.

## 2. Modelo de negocio

**B**

- SaaS por suscripción **o** ROCA como infraestructura de Cervo/DEER con agentes afiliados. Sin decisión.
- Sin precios, tiers, comisiones ni plan gratuito definidos.
- "ROCA Personal" fue el nombre de una primera versión de uso propio; no es decisión de arquitectura final.
- **A:** el ROCA desplegado debe ser usado por muchos agentes (multi-tenant).

## 3. Casos reales

**A**

- **Embudo de un inmueble:** *"23 leads, 8 conversaciones, 3 visitas, 1 oferta"*, y finalmente alquilado. El objetivo es conservar la cadena acción → contexto → resultado.
- **Canales:** hipótesis de ejemplo: departamentos de 2 dormitorios en San Miguel de S/ 2,000–2,400 podrían rendir mejor en Marketplace que con anuncios. Es una hipótesis que ROCA debe poder comprobar, no una regla.
- **Curiosos:** ROCA debe filtrar: qué busca, presupuesto, cuándo necesita mudarse, características, interés real, encaje.
- **Plazo:** si alguien quiere mudarse en dos meses, no se responde "agendemos" automáticamente; depende de disponibilidad y demanda del inmueble.
- **Captación por Marketplace:** el autor acortó el mensaje de contacto porque *"la gente no lee"* y algunos propietarios ven mal a las inmobiliarias. No asumir que un mensaje largo funciona mejor.
- **Captación sin visita previa:** el propietario puede enviar fotos, descripción, enlace o texto de otra publicación. ROCA trabaja con esa información y dice al agente qué falta para que la visita sea productiva.
- **Ejemplo de una persona con tres oportunidades:** alquilar un departamento de 3 dormitorios en San Miguel (S/ 2,200–2,400), comprar un depósito/almacén de 300–350 m² en el Centro de Lima (S/ 560,000) y vender su casa en Chorrillos. Detalle en `03` §2.9.

## 4. Reglas de autonomía

**A**

| Acción | Quién |
|---|---|
| Recopilar información, filtrar leads, identificar necesidades, recomendar inmuebles | ROCA |
| Preparar copys, publicaciones, propuestas y borradores | ROCA |
| Responder preguntas sobre inmuebles (solo con datos registrados) | ROCA |
| Actualizar CRM, registrar actividad, detectar pendientes, notificar al agente | ROCA |
| Reactivar leads fríos | ROCA (lógica de selección: TBD) |
| Seguimientos | **Mixtos:** unos puntuales, ejecutados por ROCA; otros solo notifican al agente para que llame o escriba |
| Publicar | Requiere aprobación del agente en cada publicación |
| Fecha y hora de visita | El agente. ROCA recopila rangos de disponibilidad y escala; ante "¿mañana?" no compromete hora |
| Negociar precio | El agente. ROCA puede recopilar presupuesto o disposición y escalar |
| Contratos | ROCA prepara borrador y señala datos pendientes; el agente aprueba |

Citas originales: *"ROCA prepara posts, usuario aprueba cada publicación"*; *"ROCA no negocia ni agenda; recopila rangos y escala; usuario coordina propietario"*.

**A:** primera respuesta a un lead nuevo: ROCA responde sola con datos registrados del inmueble (excepción explícita al criterio de aprobación primero). Mensajes que no son leads: ROCA propone clasificarlos y el agente confirma. Un agente externo que contacta con un cliente por un inmueble del autor puede originar una Opportunity. Las conversaciones son un hilo continuo por persona y canal, enlazado a varias oportunidades (`06-conversations.md`).

**A:** criterio general: **aprobación primero**. ROCA propone y el agente aprueba; qué acciones pasan a automáticas se decide después, con datos reales (por ejemplo, la reapertura de oportunidades).

**A:** ROCA no inventa información de inmuebles ni completa datos faltantes. **A:** algunas capacidades deben razonar (LLM) y otras ser solo automatización determinística.

## 5. Postvisita, datos y agenda

**A**

- Postvisita: feedback libre del agente que ROCA puede estructurar. No se impone un formulario rígido.
- Todo evento se registra con resultados de ser posible (`04-event-model.md`).
- Los clientes reciben links compartibles de propiedades, sin cuenta en ROCA.
- **Fin de contrato de alquiler:** ROCA se anticipa. Antes del vencimiento contacta (o avisa al agente para contactar) al propietario y al inquilino. Si el inquilino se va, se le muestran opciones y el inmueble vuelve a promocionarse. Las Opportunities cerradas con éxito pueden reabrirse por caída de la operación o fin de contrato.
- El agente necesita ver qué tiene pendiente, a quién responder, qué seguimiento toca y qué visita viene, con contexto (ejemplos: "Carlos — confirmar visita", "María — seguimiento prometido hoy").

**B**

- "Mi día": agenda unificada de reuniones, visitas, recordatorios, seguimientos y tareas con prioridad. **A:** se sincroniza bidireccionalmente con el calendario del teléfono; el orden es horas fijas primero y luego por dinero; los avisos van por notificación push y dentro de Mi día. Detalle en `05-agenda.md`; la fórmula de prioridad no está definida.
- Prioridad de inmuebles (qué destacar, dónde invertir en publicidad): deseada, sin fórmula definida. No inventarla.

## 6. Integraciones

**A**

- Regla arquitectónica (principio P13): *"integraciones siempre mediante conectores, nunca APIs externas directas al Core."* Los canales pueden agregarse, cambiar o eliminarse.
- Requeridos como canales: WhatsApp, Messenger, Marketplace, leads de portales.

**B** (no validado técnicamente)

- Proveedor definitivo de WhatsApp/Messenger y cómo se conecta (directo o mediante terceros).
- Marketplace: no asumir que ROCA lo controla por completo; es un problema técnico por resolver.
- Leads de Urbania, Adondevivir y Babilonia: **A:** siempre llegan por correo con los datos del inmueble y del lead (nombre, DNI, teléfono y correo, no siempre reales) y, según lo que elija la persona, también por WhatsApp, llamada, llamada por WhatsApp o correo. **B:** no hay un mecanismo validado para leer esos correos. **A:** debe evitarse contactar repetidamente a la misma persona; caso real: una visita agendada con la misma persona por WhatsApp (portal) y por Messenger (Marketplace). Detalle en `07-identity.md`.
- Importar una publicación existente de un portal para extraer datos: útil conceptualmente, sin resolver técnicamente.
- Chatwoot, n8n y Cerniio: ver sección 8.

## 7. Stack y desarrollo

- **A:** se trabajará con Spec-Driven Development. Motivo: desarrollo previo "sobre la marcha" con agentes que acumuló deuda técnica.
- **A:** el código anterior no cuenta; el desarrollo **empieza de cero**. Esto descarta, por ahora, cualquier herencia del esquema anterior (incluida la limitación de `ownerId`).
- **B:** backend sin decidir. Supabase e InsForge siguen como alternativas. InsForge se consideró sobre todo por la implementación (se afirmó que está diseñado para trabajar con agentes de IA; **no verificado**) y porque podría servir también para producción, según si se usará RAG. No hay decisión de reemplazo.
- **B:** herramientas de desarrollo (Cursor, OpenCode, Codex, Lovable, ChatGPT, modelos locales). ROCA no debe depender conceptualmente de ellas.
- **A:** el autor no es programador tradicional; actúa como Product Owner y supervisor de desarrollo asistido por IA. No hay equipo de ingeniería dedicado. Se planteó una revisión senior antes de lanzar.
- **B:** presupuesto, equipo de desarrollo y plazos sin definir. No inventar límites ni fechas.

## 8. Propuestas de IA no confirmadas (C) y exploraciones (B)

- **B:** Chatwoot (capa de comunicación), n8n (automatización) y Cerniio (social CRM). No se decidió si se conectan directamente o a través de terceros. Si se usan, la lógica de negocio debe permanecer en ROCA y las herramientas ser reemplazables.
- **C:** "cinco cerebros" (cliente, propiedad, ventas, marketing, copiloto) y otros conjuntos de agentes (Comercial, Inventario, Marketing, Seguimiento, Cierre, Analista). No construir entidades rígidas con esos nombres.
- **B:** "Traficker" como posible capa de publicidad y priorización; no es entidad del modelo.

## 9. Descartado (D) y evolución

- **D:** publicar sin aprobación como comportamiento por defecto; que ROCA negocie precios; que ROCA fije visitas por sí sola; que ROCA genere contratos de forma autónoma y definitiva; diseñar ROCA como un CRM genérico.
- **No descartados** (el traspaso original los marcaba como D por error): Chatwoot, InsForge y los seguimientos totalmente automáticos. Los dos primeros son **B**; los seguimientos son mixtos (sección 4).

## 10. Preguntas abiertas

- Modelo de negocio, pricing y primer segmento comercial.
- Backend definitivo y proveedor de IA principal (con fallback).
- Mecanismo técnico de cada integración (sección 6).
- Límite exacto de V1 y orden entre catálogo y CRM.
- Qué mensajes y qué cambios de datos requieren aprobación; operaciones irreversibles; control de envíos masivos; manejo de errores de interpretación.
- Reapertura (`03` §5): decidida sobre la misma Opportunity cuando la persona retoma su objetivo, con aprobación del agente. También se permite tras `won` (operación caída o fin de contrato de alquiler). No hay límite de reaperturas. Falta definir qué reaperturas serán automáticas.
- Taxonomías de objetivos, resultados y motivos.

## 11. Advertencias para quien continúe el SDD

1. "Lo hablamos" no equivale a "el autor decidió".
2. Una ambigüedad real se marca como **OPEN DECISION** y no se rellena con una suposición razonable (P11).
3. ROCA no es un agente autónomo: recopila, entiende, prepara, automatiza lo rutinario, recomienda y escala; el agente conserva las decisiones sensibles.
4. No afirmar que una integración externa funciona hasta que esté validada.

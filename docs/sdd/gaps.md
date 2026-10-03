# Vacíos y mejoras detectadas

**Estado:** DRAFT v0.1  
**Última actualización:** 2026-10-02

Notas de revisión sobre `00` a `04`. No son decisiones: son puntos a evaluar. Se ordenan según su cercanía al ciclo que genera dinero (captar, agendar y realizar visitas, negociar, cerrar).

## Vacíos que afectan al ciclo de ingresos

| # | Vacío | Por qué importa | Sugerencia |
|---|---|---|---|
| 1 | **Acuerdo de servicios.** Definido: captado = el propietario acepta trabajar con nosotros, exclusividad sin/semi/total, vigencia por defecto de 3 meses (alquiler) y 6 (venta), aviso de vencimiento con decisión del agente (seguir o archivar). Propuesto como `ServiceAgreement`. Falta confirmar el modelo | El nivel de exclusividad determina quién más promociona el inmueble | Confirmar el modelo en `02` y el vencimiento en `03` §2.10 |
| 2 | **Comisión.** Definido: la paga el propietario; 3% en venta y 1 mes de renta en alquiler; en sin/semi-exclusividad no se cobra si cierra un tercero; en exclusividad, 100% por defecto (variable); reparto con otros agentes 50/50 por defecto | Es el dato de ingresos; permite analizar rentabilidad por canal, inmueble, exclusividad o tipo de operación | Comisión pactada en `ServiceAgreement` y aplicada y cobrada en `Operation` |
| 3 | **Agenda y recordatorios.** Borrador en `05-agenda.md`. Decidido: sincronización bidireccional, orden mixto (horas fijas y luego dinero) y avisos por push y Mi día. Falta la fórmula de prioridad y el proveedor de calendario | Es el dolor central ("si no hago algo al momento se me olvida"); visitas, vencimientos y seguimientos dependen de ello | Confirmar el borrador antes de implementar seguimientos |
| 4 | **Deduplicación de personas entre canales.** Borrador en `07-identity.md`. Decidido: buscar por nombre y/o teléfono antes de crear. Propuesto: niveles de coincidencia, fusión reversible y verificación antes de programar visitas. Falta decidir el tratamiento de coincidencias probables | Evitar contactar dos veces a quien ya fue contactado y agendar visitas duplicadas | Confirmar el borrador |
| 5 | **Conversation.** Borrador en `06-conversations.md`. Decidido: hilo continuo por persona y canal enlazado a varias oportunidades, respuesta inicial automática de ROCA con datos registrados, clasificación propuesta por ROCA y confirmada por el agente, y otros agentes como posible origen de Opportunity | Tiempo de primera respuesta y entrada de leads | Confirmar el borrador |

## Vacíos de segundo nivel

| # | Vacío | Sugerencia |
|---|---|---|
| 6 | **Reporte al propietario.** El propietario es quien paga y hoy no existe mecanismo para informarle de visitas, feedback y ofertas | Las visitas y resultados ya generan los datos; falta definir el reporte |
| 7 | **Documentos y requisitos de cierre** (verificación del inquilino, contratos en borrador, documentos del inmueble). No hay entidad Document | Definir cuando se especifique la etapa de negociación y cierre |
| 8 | **Acceso al inmueble** (propietario u ocupante que debe permitir la visita, llaves) | Ya recogido como participante en Visit; falta el flujo de coordinación |
| 9 | **Protección de datos personales.** ROCA almacena datos de personas y conversaciones | Verificar los requisitos legales aplicables en Perú antes de lanzar |
| 10 | **Cooperación entre agentes** (inmuebles y oportunidades compartidos). Decidido: reparto 50/50 por defecto y agentes externos registrados como personas en el CRM. Falta: compartir inmuebles y oportunidades entre usuarios | Dejar para después de V1, pero no bloquear el modelo |

## Lo que probablemente sobra por ahora

- **Conceptos propuestos por IA sin confirmar:** "cinco cerebros", "Traficker", fórmulas de prioridad de inmuebles.
- **Catálogo de eventos completo.** Para V1 bastan los eventos del ciclo de ingresos y de las reglas de autonomía. El resto puede añadirse sin migrar si el sobre del evento está bien definido.
- **Integraciones secundarias** (LinkedIn, YouTube, Telegram, TikTok Ads, Google Ads): sin validar y fuera del ciclo central.
- **Automatización de canales completa.** Con "aprobación primero", conviene empezar por preparar y registrar, y automatizar después según datos.

## Criterio sugerido para priorizar V1

Una función entra en V1 si cumple al menos una:

1. Ayuda a captar, agendar, realizar visitas, negociar o cerrar.
2. Evita perder un lead o un seguimiento.
3. Genera datos que se necesitarán después y que no se pueden reconstruir (eventos, motivos, resultados).

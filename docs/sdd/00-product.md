# 00 — Producto

**Estado:** DRAFT v0.1  
**Última actualización:** 2026-10-01

## 1. Qué es ROCA

ROCA es un sistema operativo para agentes inmobiliarios que centraliza inmuebles, personas, oportunidades, conversaciones, actividades, tareas, visitas y publicaciones, incorporando capacidades de IA sobre una única fuente de datos.

La visión de largo plazo es una **“agencia detrás del agente”**: ROCA ayuda al agente a captar propiedades, atender y convertir interesados, mantener el seguimiento, organizar su operación y ejecutar marketing, sin asumir decisiones comerciales que corresponden al agente.

ROCA no debe diseñarse como un chatbot aislado ni como un CRM convencional al que posteriormente se le añade IA. El núcleo es un modelo de dominio y un historial de eventos sobre el cual las distintas capacidades de ROCA pueden operar.

## 2. Problema

La información y el trabajo del agente inmobiliario están dispersos entre portales, WhatsApp, Messenger, archivos, fotografías, notas, portapapeles, memoria y herramientas de publicación.

Esto genera:

- pérdida de contexto;
- respuestas lentas o incompletas;
- seguimiento manual y oportunidades olvidadas;
- dificultad para conocer rápidamente toda la información de un inmueble;
- duplicación del trabajo de publicación;
- poca trazabilidad de lo ocurrido con personas, inmuebles y oportunidades;
- dificultad para detectar qué debe hacer el agente a continuación;
- dificultad para reutilizar la información histórica para recomendaciones futuras.

## 3. Resultado buscado

ROCA debe permitir que el agente:

1. tenga una fuente central de información de sus inmuebles;
2. conserve el contexto histórico de personas y oportunidades;
3. convierta conversaciones en acciones y siguientes pasos;
4. automatice seguimientos rutinarios dentro de reglas definidas;
5. reciba alertas cuando una situación requiera intervención humana;
6. genere materiales de marketing a partir de datos estructurados;
7. consulte y actualice información mediante lenguaje natural;
8. conserve datos suficientemente estructurados para permitir inteligencia futura.

## 4. Alcance inicial

El primer producto está orientado a un **agente inmobiliario independiente**.

Sin embargo, el dominio debe permitir desde el inicio:

- inmuebles compartidos entre agentes;
- oportunidades compartidas entre agentes;
- evolución posterior hacia equipos e inmobiliarias;
- separación entre persona, agente/usuario y organización.

No se debe introducir complejidad empresarial que no sea necesaria para el producto inicial, pero tampoco tomar decisiones que bloqueen innecesariamente la evolución posterior.

## 5. Capacidades principales

- Catálogo y gestión de inmuebles.
- Personas y contexto histórico.
- Oportunidades comerciales.
- Conversaciones.
- Visitas.
- Actividades y eventos.
- Tareas y agenda.
- Publicaciones.
- Marketing.
- Copiloto/IA.
- Integraciones con canales y herramientas externas.

## 6. Límites iniciales de la IA

ROCA no debe, por defecto:

- inventar información;
- negociar precios de forma autónoma;
- comprometer fechas no confirmadas;
- publicar sin la aprobación requerida;
- tomar decisiones comerciales críticas reservadas al agente;
- afirmar que existe aprendizaje avanzado cuando todavía no existe suficiente información histórica para justificarlo.

Los niveles de autonomía y las acciones permitidas se especificarán formalmente en el SDD.

## 7. Principio de evolución

El SDD es un artefacto vivo.

Cuando se incorpora una nueva capacidad, primero deben identificarse sus efectos sobre:

- dominio;
- entidades;
- relaciones;
- estados;
- eventos;
- permisos;
- flujos;
- requisitos;
- integraciones;
- pruebas.

Después se implementa.

La implementación no debe convertirse en la fuente primaria de decisiones de producto.

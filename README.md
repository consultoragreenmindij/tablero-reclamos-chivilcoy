# Tablero de Reclamos — Municipalidad de Chivilcoy

Trabajo Final Integrador — Diplomatura en IA Aplicada a Entornos Digitales de Gestión (FCE-UBA, Cohorte 2026).
Autoras/es: Ivana Jacobs, Eduardo, Horacio.

## Qué hace

Un sistema para que un vecino de Chivilcoy haga un reclamo por WhatsApp (bache, luminaria, poda, etc.) y ese reclamo llegue organizado, clasificado y visible en tiempo real a un tablero de gestión tipo Kanban, sin que nadie tenga que cargar nada a mano.

## Para qué sirve

Hoy, en muchos municipios chicos, los reclamos vecinales se gestionan por WhatsApp informal, cuadernos o planillas sueltas: se pierden, se duplican, nadie sabe qué está pendiente y qué ya se resolvió. Este proyecto propone un circuito mínimo, sin intermediarios innecesarios, para que:

- el vecino reclame por el canal que ya usa (WhatsApp),
- el reclamo quede clasificado por área automáticamente,
- el equipo municipal vea de un vistazo qué está pendiente, en curso y resuelto, por área y con foto.

## Cómo funciona (arquitectura)

```
Vecino (WhatsApp)
      │
      ▼
  WotNot (bot conversacional)
  · pide tipo de reclamo, descripción, ubicación y foto
  · clasifica automáticamente por área responsable
      │
      ▼
  Google Sheet (única fuente de datos)
      │
      ▼
  AppSheet (tablero de gestión)
  · vista Kanban por estado (pendiente / en curso / resuelto)
  · vista de estadísticas
  · sin login para editar, apto para uso diario del equipo municipal
```

**Decisión de diseño deliberada: cero intermediarios extra.** Se evaluó sumar una capa intermedia tipo Apps Script o n8n para "traducir" datos entre el bot y el tablero, y se descartó a propósito. La planilla de Sheets es la única fuente de verdad; el bot escribe ahí directamente y AppSheet lee de ahí directamente. Menos piezas, menos puntos de falla, más fácil de mantener por alguien sin perfil técnico — que es exactamente el contexto real de un equipo municipal chico.

Ver el detalle completo de decisiones tomadas y por qué en [`docs/decisiones.md`](docs/decisiones.md).

## Nivel de implementación

Este proyecto corre **de punta a punta en producción real**, sin simulación ni maqueta: un reclamo real enviado por WhatsApp hoy queda visible en el tablero en segundos. Encuadra como **Nivel 1 (implementación funcional)** en espíritu, aunque el medio no sea código tradicional sino una integración no-code (bot conversacional + planilla + app de gestión). No hay backend propio para desplegar ni notebook que correr: las tres piezas (WotNot, Google Sheets, AppSheet) ya están configuradas y operativas.

## Cómo se usa

- **El vecino**: le escribe al número de WhatsApp del municipio, el bot lo guía con preguntas cortas (tipo de reclamo, descripción, ubicación, foto) y clasifica el área automáticamente.
- **El equipo municipal**: abre la app de AppSheet (web o celular) y ve el tablero Kanban agrupado por estado, con foto y descripción de cada reclamo. Puede cambiar el estado de un reclamo (pendiente → en curso → resuelto) tocando la tarjeta.

### Capturas

**Tablero de Reclamos (vista Kanban, agrupada por estado):**

![Tablero de Reclamos](docs/screenshots/tablero-kanban.png)

**Estadísticas (conteo de reclamos por estado):**

![Estadísticas](docs/screenshots/estadisticas.png)

## Contenido generado con IA

- **Diseño y corrección del circuito completo** (bot → planilla → app): asistido con Claude (Anthropic), incluyendo depuración de la integración AppSheet–Google Sheets, corrección de vistas, formato condicional por estado y resolución de un bug de fuente de datos en la vista de Estadísticas.
- **Identidad visual del tablero**: escudo oficial de la Municipalidad de Chivilcoy integrado en la planilla y en la app.
- El flujo conversacional del bot (WotNot) fue diseñado y configurado por el equipo, sin generación automática de contenido conversacional por IA en esta etapa.

Detalle de metodología, prompts y decisiones en el informe PDF que acompaña esta entrega.

## Estado del proyecto

- ✅ Bot de WhatsApp operativo, clasifica por área automáticamente.
- ✅ Planilla como fuente única de datos.
- ✅ Tablero Kanban en AppSheet, agrupado por estado, con foto y descripción.
- ✅ Vista de estadísticas por estado.
- ⏳ Filtro de seguridad por área (cada responsable ve solo su área): pausado, pendiente de relevar los emails de los responsables municipales.
- ⏳ Compartir acceso formal con todo el equipo de gestión.

## Equipo

Proyecto grupal (hasta 3 integrantes según consigna del TP): Ivana Jacobs, Eduardo y Horacio.

## Informe

El informe completo del TP (estructura a–f exigida por la consigna, incluyendo el análisis con las 4 dimensiones AIBPS) está en [`informe-TP-tablero-reclamos-chivilcoy.pdf`](informe-TP-tablero-reclamos-chivilcoy.pdf), en la raíz de este repositorio.

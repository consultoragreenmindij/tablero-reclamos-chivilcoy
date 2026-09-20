# Registro de decisiones

Este documento muestra el proceso y la evolución del proyecto, no solo el resultado final — decisiones tomadas, alternativas descartadas y por qué.

## 1. Herramienta de tablero: se evaluó Looker Studio, se eligió AppSheet

**Pedido inicial**: que la planilla de reclamos se convierta en un tablero visual, tipo Kanban, fácil de usar día a día por el equipo municipal.

**Se probó Looker Studio primero.** Se descartó: el layout se cortaba en pantalla, no tenía vista Kanban nativa (es una herramienta de reportes/dashboards, no de gestión operativa), y no permite editar el estado de un reclamo desde ahí — solo visualizar.

**Se eligió AppSheet.** Corre sobre la misma planilla, tiene vista Kanban nativa (agrupación por estado con tarjetas), y permite editar el estado de un reclamo tocando la tarjeta — que es justamente el caso de uso diario del equipo.

## 2. Cero intermediarios: se descartó sumar Apps Script o n8n

Principio explícito del equipo: la menor cantidad posible de piezas intermedias entre el bot y el tablero.

Se evaluó agregar una capa de automatización (Apps Script o n8n) para "traducir" o validar datos entre WotNot y AppSheet. Se descartó a propósito: sumar código intermedio contradice el objetivo de simplicidad, y cada pieza nueva es un punto de falla más difícil de mantener para un equipo sin perfil técnico. Arquitectura final: **WotNot → Google Sheet → AppSheet**, sin nada en el medio.

## 3. Clasificación por área: se resuelve en el origen, no en el tablero

La clasificación por área responsable (Alumbrado Público, Obras Públicas, Higiene Urbana, etc.) la resuelve el propio bot de WotNot con un menú de opciones al recibir el reclamo por WhatsApp. AppSheet no necesita ningún mecanismo propio de clasificación — el dato ya llega clasificado.

## 4. Benchmark de mercado: se comparó contra soluciones verticales (ej. Munify)

Se investigaron plataformas SaaS municipales existentes (reclamos vía WhatsApp/app/web, clasificación automática, paneles por área) para entender el estándar del sector. Conclusión: para el alcance de este TP, AppSheet es la opción no-code más cercana a lo que ofrecen esas herramientas verticales, sin sumar intermediarios ni costo. Una plataforma dedicada queda como referencia de escalabilidad si la Municipalidad decide institucionalizar el sistema más allá de este trabajo.

## 5. Corrección de bug: fuente de datos incorrecta en la vista Estadísticas

**Problema detectado**: la vista "Estadísticas" usaba la tabla completa de la planilla en lugar de la vista filtrada, y contaba como reclamos "en blanco" una cantidad de filas vacías generadas por una fórmula de numeración (`=FILA()-1`) que no corresponden a reclamos reales. Esto mostraba una barra de ~51 reclamos "(blank)" dominando el gráfico, cuando los reclamos reales eran muy pocos.

**Corrección**: se cambió la fuente de datos de la vista de la tabla completa "Reclamos" a la vista filtrada "Reclamos con datos" (que excluye filas sin descripción real). Resultado verificado: el gráfico ahora refleja los conteos reales por estado.

## 6. Identidad visual: escudo oficial de la Municipalidad

Se incorporó el escudo oficial de la Municipalidad de Chivilcoy en la planilla (como imagen de referencia) y en la app de AppSheet (ícono de la app), alojado en Google Drive con acceso público por enlace — único formato de URL de Drive que renderiza de forma confiable tanto en Sheets (`=IMAGE()`) como en AppSheet (`drive.google.com/thumbnail?id=...`).

## 7. Pendientes conocidos (no bloqueantes)

- **Filtro de seguridad por área**: pausado. Requiere el email de cada responsable de área, dato que depende de un relevamiento con la Municipalidad aún no completado. Mientras el piloto tenga pocos usuarios, se comparte el tablero sin filtro.
- **Clave de tabla e tipo de columna "Área responsable"**: detalles cosméticos de configuración en AppSheet (la clave de tabla vuelve a `_RowNumber` al guardar; la columna de área quedó como texto en lugar de enum). No afectan el funcionamiento del tablero.

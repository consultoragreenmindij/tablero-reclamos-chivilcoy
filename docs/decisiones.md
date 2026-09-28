# Registro de decisiones

Este documento muestra el proceso y la evolución del proyecto, no solo el resultado final — decisiones tomadas, alternativas descartadas y por qué.

## 1. Herramienta de tablero: se evaluó Looker Studio, se eligió AppSheet

**Pedido inicial**: que la planilla de reclamos se convierta en un tablero visual, fácil de usar día a día por el equipo municipal.

**Se probó Looker Studio primero.** Se descartó: el layout se cortaba en pantalla, es una herramienta de reportes y no de gestión operativa, y no permite editar el estado de un reclamo desde ahí — solo visualizar.

**Se eligió AppSheet.** Corre sobre la misma planilla, agrupa los reclamos por estado en tarjetas y permite cambiar el estado desde la misma pantalla, que es justamente el caso de uso diario del equipo.

**Precisión importante sobre el vocabulario.** Durante buena parte del proyecto esta vista se llamó "Kanban". No lo es: AppSheet produce una lista vertical agrupada por estado, sin columnas paralelas ni arrastre entre ellas. La diferencia se verificó mirando la app real y el nombre se corrigió en toda la documentación. Lo que sí se logró es el objetivo de fondo —distinguir el estado de un vistazo y cambiarlo en un toque— por otro camino: código de color y acciones directas sobre la tarjeta.

## 2. Cero intermediarios: se descartó sumar Apps Script o n8n

Principio rector explícito del proyecto: la menor cantidad posible de piezas intermedias entre el bot y el tablero.

Se evaluó agregar una capa de automatización (Apps Script o n8n) para "traducir" o validar datos entre WotNot y AppSheet. Se descartó a propósito: sumar código intermedio contradice el objetivo de simplicidad, y cada pieza nueva es un punto de falla más difícil de mantener para un equipo sin perfil técnico. Arquitectura final: **WotNot → Google Sheet → AppSheet**, sin nada en el medio.

**También se evaluó migrar el tablero a Trello y se descartó**, por las mismas razones: suma una capa intermedia, genera dos fuentes de verdad, y la sincronización vía Apps Script falla en silencio. Además no resuelve el problema que parecía resolver, porque el control de acceso por área en Trello necesita exactamente los mismos emails de responsables que todavía faltan.

## 3. Clasificación por área: un supuesto que hubo que corregir

**Lo que se asumió durante casi todo el desarrollo.** Que la clasificación por área responsable la resolvía por sí solo el menú del bot de WotNot al recibir el reclamo, y que por lo tanto AppSheet no necesitaba ningún mecanismo propio: el dato ya llegaría clasificado.

**Lo que se comprobó al probar el circuito completo.** El supuesto era falso. El bot guarda en la planilla el *tipo de reclamo* que eligió el vecino (Bacheo/calles, Residuos, Arbolado...), que no es lo mismo que el *área municipal responsable* (Obras Públicas, Higiene Urbana, Espacios Verdes...). Los reclamos cargados hasta ese momento tenían el área completada a mano, y eso mantuvo el problema invisible. La consecuencia era concreta: todo reclamo nuevo llegaba sin área asignada y no aparecía en ninguna de las vistas por área hasta que alguien lo clasificara manualmente.

**Primer intento, fallido.** Se pre-cargó una fórmula por celda en un rango fijo de filas. No funcionó: WotNot **inserta una fila nueva** debajo de la última con datos en lugar de escribir sobre las filas pre-cargadas, de modo que la fila insertada llegaba sin fórmula. El diagnóstico inicial —"el rango se quedó corto"— era erróneo; el problema era la inserción de filas.

**Corrección aplicada.** Una única `ARRAYFORMULA` sobre un rango abierto de la columna traduce el tipo de reclamo al área responsable (Bacheo/calles a Obras Públicas, Residuos a Higiene Urbana, y así) y se recalcula sola para cada fila que el bot inserta, sin tope de filas. Está escrita con búsqueda de subcadena para que resulte insensible a mayúsculas y acentos, porque el texto del menú del bot y el de los datos ya cargados no coincidían exactamente. Se resolvió dentro de la planilla y no con una capa intermedia, para no contradecir la decisión 2.

**Límites conocidos de esta solución.** El mapeo de palabras clave es fijo: sumar un tipo de reclamo nuevo o cambiar el texto del menú del bot obliga a editar la fórmula. Y no se puede pisar a mano el área de un reclamo puntual sin romper el cálculo de toda la columna.

**Por qué se registra el error y no solo la solución.** Es el hallazgo más instructivo del proyecto. Los otros errores se detectaron mirando la configuración de las herramientas; este apareció únicamente al recorrer el circuito entero como lo haría un vecino.

## 4. Benchmark de mercado: se comparó contra soluciones verticales (ej. Munify)

Se investigaron plataformas SaaS municipales existentes (reclamos vía WhatsApp/app/web, clasificación automática, paneles por área) para entender el estándar del sector. Conclusión: para el alcance de este TP, AppSheet es la opción no-code más cercana a lo que ofrecen esas herramientas verticales, sin sumar intermediarios ni costo. Una plataforma dedicada queda como referencia de escalabilidad si la Municipalidad decide institucionalizar el sistema más allá de este trabajo.

## 5. Corrección de bug: fuente de datos incorrecta en la vista Estadísticas

**Problema detectado**: la vista "Estadísticas" usaba la tabla completa de la planilla en lugar de la vista filtrada, y contaba como reclamos "en blanco" una cantidad de filas vacías generadas por una fórmula de numeración (`=FILA()-1`) que no corresponden a reclamos reales. Esto mostraba una barra de ~51 reclamos "(blank)" dominando el gráfico, cuando los reclamos reales eran muy pocos.

**Corrección**: se cambió la fuente de datos de la vista de la tabla completa "Reclamos" a la vista filtrada "Reclamos con datos" (que excluye filas sin descripción real). Resultado verificado en la app: el gráfico refleja los conteos reales por estado.

**Nota sobre las capturas.** Las capturas del repositorio y del informe se rehicieron después de esta corrección. Las anteriores todavía mostraban la barra "(blank)" y contradecían el texto que afirmaba que el bug estaba resuelto.

## 6. Identidad visual y código de color por estado

Se incorporó el escudo oficial de la Municipalidad de Chivilcoy en la planilla (como imagen de referencia) y en la app de AppSheet (ícono de la app), alojado en Google Drive con acceso público por enlace — único formato de URL de Drive que renderiza de forma confiable tanto en Sheets (`=IMAGE()`) como en AppSheet (`drive.google.com/thumbnail?id=...`).

Después se sumó el color institucional (`#0A7A3D`) como color primario, el modo oscuro y una franja verde en el encabezado y el pie de la app.

**El código de color por estado obligó a un rodeo que vale la pena registrar.** El "highlight color" de una regla de formato de AppSheet no pinta el fondo de la tarjeta: dibuja un punto de color antes de **cada** columna formateada. Aplicado a tres columnas producía tres puntos por tarjeta y parecía un error de la app. La solución fue desdoblar cada estado en dos reglas: una que pinta el texto de las tres columnas sin highlight, y otra limitada a la columna del nombre que aporta el único punto. Quedaron seis reglas con nombres explícitos para que la razón del desdoblamiento siga siendo legible más adelante.

También se probó cambiar el tipo de vista a formato de tarjetas ("card") y se descartó: exige mapear cada campo a una posición de la tarjeta y la mayoría de los reclamos no tiene foto, de modo que el resultado era peor que la vista original.

## 7. Alcance: piloto, no servicio en operación

Durante parte de la redacción, la documentación describía el sistema como si estuviera en producción. Se corrigió al verificar el estado real: el bot figura como "Not deployed", los teléfonos de los reclamos de prueba son secuenciales y una de las filas había entrado por canal web, no por WhatsApp. **No se dispone de la línea de WhatsApp oficial de la Municipalidad**, así que el bot se construyó y probó sobre un número propio.

La decisión fue encuadrar el trabajo como piloto verificado de punta a punta y decirlo explícitamente en todos los documentos, en lugar de dejar una ambigüedad que el lector podría tomar por afirmación.

## 8. Pendientes conocidos (no bloqueantes)

- **Despliegue del bot en una línea de WhatsApp abierta a vecinos**: pendiente. El circuito está construido y probado de punta a punta con casos de prueba, pero todavía no se publicó en un número en uso. Es una decisión que excede al desarrollo y corresponde a Intendencia o a la Secretaría de Gobierno.
- **Dependencia de una plataforma paga**: WotNot no ofrece un plan gratuito permanente y el más económico ronda los 29 dólares mensuales. Se decidió no contratarlo para este trabajo. Como resguardo, la configuración completa del flujo del bot está exportada, de modo que el trabajo no se pierde al vencer la cuenta de prueba.
- **Filtro de seguridad por área**: pausado. Requiere el email de cada responsable de área, dato que depende de un relevamiento con la Municipalidad aún no completado. Mientras el piloto tenga pocos usuarios, se comparte el tablero sin filtro.
- **Clave de tabla y tipo de columna "Área responsable"**: detalles cosméticos de configuración en AppSheet (la clave de tabla vuelve a `_RowNumber` al guardar; la columna de área quedó como texto en lugar de enum). No afectan el funcionamiento del tablero.

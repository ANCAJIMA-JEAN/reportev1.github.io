# Seguimiento de reportes · Subgerencia de Producción
## Guía de despliegue y mantenimiento (se hace una sola vez)

Archivos del paquete: `Codigo.gs` (todo el backend en un único archivo), `appsscript.json`, `index.html`, `config.js`.

---
## PARTE A · Google (Sheets + Apps Script + Drive) — unos 15 minutos

1. En Google Drive crea una **hoja de cálculo vacía** y llámala `BD Seguimiento de reportes`.
2. Dentro de ella: **Extensiones → Apps Script**.
3. En el editor: **Configuración del proyecto (engranaje) → marca "Mostrar archivo de manifiesto appsscript.json"**. Vuelve a **Editor**, abre `appsscript.json`, borra su contenido y pega el de este paquete. Guarda.
4. Abre `Código.gs`, **borra todo** y pega el contenido completo de `Codigo.gs`. Guarda (Ctrl+S).
5. Elige la función **`inicializarSistema`** y pulsa **Ejecutar**.
   - Google pedirá permisos (Sheets, Drive, Documentos, correo, conexiones externas). Si dice "Google no ha verificado esta app": **Configuración avanzada → Ir a (proyecto) → Permitir**. Es tu propio script.
   - Al terminar verás un resumen: 15 hojas creadas, 7 áreas cargadas y las carpetas de Drive.
6. Ejecuta **`probarSemanaISO`** (el registro debe decir que todos los casos son correctos) y luego **`validarEstructura`** (debe decir "Estructura correcta").
7. **Implementar → Nueva implementación → tipo "Aplicación web"**: *Ejecutar como: Yo* · *Quién tiene acceso: Cualquier persona*. Pulsa **Implementar** y **copia la URL que termina en `/exec`**.
8. Prueba la URL en el navegador añadiendo `?accion=ping`. Debe responder `{"ok":true,...}`.

## PARTE B · GitHub Pages

1. Crea un repositorio **público** en GitHub (no contiene claves; las páginas de repos privados requieren un plan de pago).
2. Abre `config.js`, reemplaza `PEGUE_AQUI_LA_URL_EXEC` por la URL del paso A7 y guarda.
3. Sube `index.html` y `config.js` a la raíz del repositorio.
4. **Settings → Pages → Source: Deploy from a branch → main / (root) → Save**. En un minuto tendrás la dirección pública del sistema.
5. Abre esa dirección: arriba a la derecha debe decir **Conectado** y deben aparecer las 7 áreas.

---
## PARTE C · Prueba rápida de aceptación (15 minutos, con datos de prueba)

Mantén `Entorno = PRUEBA` en la hoja CONFIGURACION mientras pruebas. Al terminar, borra las filas de prueba de las hojas de datos (deja los encabezados) y cambia `Entorno` a `PRODUCCION`.

| # | Qué hacer | Qué debe pasar |
|---|---|---|
| 1 | Abrir el panel | Aparecen las 7 áreas y se pueden abrir todas |
| 2 | En AREAS, desmarcar `Registro_Habilitado` de un área y recargar | "Nuevo reporte", "Nueva incidencia" y subir fotos quedan bloqueados, con mensaje claro |
| 3 | Desmarcar `Edicion_Habilitada` | No se pueden cambiar estados de pendientes ni de incidencias |
| 4 | Registrar un reporte con fecha de un viernes | La semana ISO aparece calculada antes de guardar; sin avisos de ventana |
| 5 | Registrar otro con fecha de un lunes | Avisa que está fuera de la ventana viernes–domingo, pero lo acepta |
| 6 | Registrar dos reportes del mismo área y semana | El segundo se rechaza y apunta al existente |
| 7 | Registrar pendientes y, en el reporte de la semana siguiente, cambiar su estado | Conservan su ID (`PEN-XXX-000001`); el historial queda en HISTORIAL_PENDIENTES |
| 8 | Registrar una incidencia con 2–3 fotos | Fotos en `02_Evidencias/año/área/incidencia`; metadatos en EVIDENCIAS; miniaturas visibles |
| 9 | Generar un informe por área (PDF y Word) | Dos archivos en `03_Informes_PDF` y `04_Informes_WORD`; abren bien; el Word se edita |
| 10 | Generar un informe consolidado | Incluye las 7 áreas |
| 11 | Revisar `INFORMES_GENERADOS`, `LOG_SISTEMA` y `HISTORIAL_CAMBIOS` | Quedan registradas las operaciones |
| 12 | Abrir el sistema desde el celular | Se ve y funciona |

**Estado de verificación (sé honesto con esto):** toda la lógica (permisos, semana ISO, duplicados, pendientes, historial, incidencias, validación de fotos, informes, correo, auditoría) pasó pruebas automáticas en simuladores de Apps Script y navegador. **Lo que no se pudo probar sin tu cuenta real de Google** y debes confirmar con la tabla anterior: la apariencia final del PDF/Word, la conversión real de Google Docs, la creación real de carpetas/archivos en Drive, las miniaturas, los permisos de Google y el envío real de correo.

---
## PARTE D · Funciones opcionales

### Envío de informes por correo (apagado por defecto)
1. En la hoja **DESTINATARIOS** agrega una fila por persona: `ID_Destinatario` (D1, D2…), `Nombre`, `Correo`, `Cargo`, `Predeterminado`, `Autorizado = TRUE`, `Activo = TRUE`.
2. En **CONFIGURACION** pon `Correo_Habilitado = TRUE`.
3. En el sistema: genera un informe → botón **Correo** → elige destinatarios → **PREPARAR ENVÍO POR CORREO** → **CONFIRMAR Y ENVIAR**.

Reglas de seguridad: **solo se puede escribir a quienes estén en DESTINATARIOS y autorizados** (así nadie puede usar la web para enviar correo a direcciones arbitrarias desde tu cuenta). Los correos completos nunca se muestran en la web. "Envío confirmado por el servicio" significa que Google aceptó el envío; **no** prueba que alguien lo haya recibido o leído. La cuota diaria de correo de Google es limitada (depende del tipo de cuenta; verifícala). Límite del sistema: 10 envíos por hora.

### Clave compartida por área (anti-abuso, opcional)
Como la web no tiene inicio de sesión, cualquiera con el enlace puede escribir en las áreas habilitadas. Para dificultarlo: en CONFIGURACION pon `Clave_Por_Area_Habilitada = TRUE` y, en **Configuración del proyecto → Propiedades del script**, crea una propiedad por área: nombre `CLAVE_AREA-SAN`, `CLAVE_AREA-IYD`, `CLAVE_AREA-OL1`, `CLAVE_AREA-OL2`, `CLAVE_AREA-OL3`, `CLAVE_AREA-API`, `CLAVE_AREA-RIE` y como valor la clave. El sistema la pedirá una vez por sesión. No es autenticación de personas.

### IA para redacción
**No está implementada** (el prompt la define como opcional). Los informes se generan con plantillas y reglas fijas, sin inventar nada. Si la quieres, se agrega después y la clave iría solo en Propiedades del script.

### Respaldos
Menú **Seguimiento de reportes → Activar respaldo semanal automático** (lunes, 02:00–03:00, hora de Lima). Guarda copias del libro en `Respaldos` dentro de la carpeta raíz de Drive y conserva las 12 últimas. También puedes usar **Crear respaldo ahora**. Para restaurar: abre la copia deseada y úsala como libro (o copia las hojas necesarias). Los activadores se ven y se quitan en el editor, icono de reloj.

---
## PARTE E · Mantenimiento

| Tarea | Cómo |
|---|---|
| Cambiar un responsable | Edita la celda `Responsable` en AREAS (no hay que tocar código) |
| Habilitar/bloquear un área | Casillas `Registro_Habilitado` / `Edicion_Habilitada` en AREAS |
| Agregar un área | Agrega una fila en AREAS con `ID_Area` (AREA-XXX), `Nombre_Area`, `Prefijo_Area` (3 letras únicas), `Responsable`, `Estado = Activa`, `Orden`, `Dia_Inicio_Entrega = 5`, `Dia_Limite_Entrega = 7`. Luego menú **Reaplicar formato y validaciones** (crea las casillas) |
| Cambiar la ventana de entrega | Columnas `Dia_Inicio_Entrega` / `Dia_Limite_Entrega` (1 = lunes … 7 = domingo) |
| Agregar un valor de lista (tipo de incidencia, etc.) | Agrégalo en CATALOGOS y ejecuta **Reaplicar formato y validaciones** |
| Cambiar límites de fotos/tamaño/escrituras | Hoja CONFIGURACION |
| Política de acceso a fotos | `Politica_Acceso_Evidencias`: `Restringido` (por defecto) o `Cualquiera con el enlace` (con enlace público, cualquiera que lo obtenga podría ver el archivo) |
| Cambiar el aspecto de los informes | En `Codigo.gs`, sección INFORMES (colores y textos fijos) |
| Revisar qué pasó | Hoja LOG_SISTEMA y HISTORIAL_CAMBIOS (las protecciones son solo advertencias: quien tenga acceso al libro puede editarlas, así que **no son un registro inalterable**) |
| Actualizar el código | Pega el nuevo `Codigo.gs`, ejecuta `inicializarSistema` (no borra datos) y **Implementar → Administrar implementaciones → ✏️ → Versión: Nueva versión → Implementar**. La URL no cambia |
| Actualizar la web | Sube el nuevo `index.html` a GitHub |

## Límites y advertencias (leer una vez)

- **No hay inicio de sesión.** "Registrado por" es un dato declarado, no una identidad verificada. Los permisos por área controlan *qué áreas* pueden escribir, no *quién*.
- **Fotos e informes:** las fotos y los informes se sirven a través del backend, así que son tan privados como la URL de la aplicación web y del sitio. La política "Restringido" solo evita compartir los archivos directamente desde Drive.
- **Cuotas de Google:** una operación de Apps Script puede tardar hasta 6 minutos; el correo, las lecturas y las escrituras tienen cuotas diarias.
- **Informes:** se arman por código en Google Docs (no desde una plantilla con marcadores) y no incluyen numeración de páginas; no se garantiza que una tabla no se parta entre páginas.
- **Pendientes en informes:** muestran su estado a la fecha de generación, no al cierre de ese periodo.
- **Semana ISO:** un reporte entregado de lunes a jueves se registra en la semana de esa fecha (se avisa si cae fuera de viernes–domingo).

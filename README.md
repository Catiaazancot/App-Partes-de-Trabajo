# App-Partes-de-Trabajo
## Prompt PoC

Quiero desarrollar una **aplicación web responsive y mobile-first en español** que funcione como Prueba de Concepto (PoC) de una futura plataforma SaaS para mecánicos aeronáuticos.

El objetivo principal de esta PoC es demostrar que un mecánico puede fotografiar o subir un parte de trabajo / Technical Logbook (TLB), utilizar Inteligencia Artificial y reconocimiento de documentos para extraer automáticamente la información relevante del trabajo realizado, revisar y corregir esa información y guardarla en su registro personal de experiencia profesional.

La aplicación debe ser funcional de extremo a extremo, pero deliberadamente sencilla. No quiero sobrearquitectura ni funcionalidades innecesarias.

Prioridades:

1. mínima complejidad;
2. mínimo coste operativo;
3. excelente experiencia móvil;
4. demostrar claramente el caso de uso;
5. arquitectura suficientemente ordenada para evolucionar posteriormente;
6. evitar desarrollar ahora funcionalidades propias de un SaaS comercial completo.

# 1. CONCEPTO

El usuario objetivo es un mecánico aeronáutico que necesita mantener un registro de su experiencia profesional.

Actualmente recibe o trabaja con documentos de mantenimiento de aeronaves de diferentes compañías y formatos. El usuario tiene que trasladar manualmente la información relevante a su logbook de experiencia.

La aplicación debe simplificar este proceso.

Flujo principal:

**Fotografiar/subir documento → analizar con IA → extraer campos → revisar → corregir si es necesario → guardar experiencia → consultar histórico → filtrar → generar reporte PDF.**

El valor diferencial de la aplicación está principalmente en evitar que el usuario tenga que introducir manualmente cada experiencia.

# 2. PRINCIPIO FUNDAMENTAL DE LA EXTRACCIÓN

Los documentos pueden proceder de diferentes compañías y tener estructuras completamente diferentes.

No diseñar el sistema esperando posiciones fijas de los campos.

La extracción debe ser semántica: analizar el contenido completo del documento e identificar qué información corresponde a cada campo independientemente de su posición.

Los documentos pueden contener:

* texto impreso;
* texto manuscrito;
* tablas;
* abreviaturas aeronáuticas;
* casillas;
* selecciones mediante marcas;
* círculos alrededor de opciones;
* fotografías tomadas con perspectiva;
* documentos girados;
* sombras;
* calidad de imagen variable.

La aplicación nunca debe inventar información.

Si un campo no puede identificarse con suficiente seguridad, devolverlo vacío o marcarlo como "Revisar".

El usuario siempre tiene la última palabra antes de guardar una experiencia.

# 3. CAMPOS OBLIGATORIOS A EXTRAER

Intentar obtener exactamente estos seis campos principales:

### Fecha

Fecha en la que se realizó el trabajo.

Normalizar internamente la fecha a un formato consistente.

### Matrícula

Aircraft Registration / Reg / Registration.

Ejemplos posibles:
EC-JFH
EI-EMK
G-MDBD
PT-MUJ

### Localización

Lugar/base/aeropuerto donde se realizó el trabajo.

Puede aparecer mediante código IATA/ICAO o texto.

Ejemplos:
ACE
AUH

No inventar el nombre completo del aeropuerto si únicamente aparece el código.

### Tipo de avión

Aircraft Type / A/C Type / Model.

Ejemplos:
B737-800
B787-8
A330
A320
A321
A318/A319/A320/A321

Mantener el máximo detalle razonablemente identificable en el documento.

### Tarea realizada

Task / Operation performed / Rectification / Action Taken / Work Performed.

Este campo es especialmente importante.

Cuando un documento contenga tanto una descripción del defecto como una rectificación o acción realizada, priorizar la **acción, rectificación o trabajo ejecutado**, no simplemente el defecto inicial.

Conservar suficiente detalle técnico para que posteriormente pueda utilizarse como evidencia de experiencia profesional.

No resumir agresivamente el contenido técnico.

### ATA

ATA Chapter relacionado con la tarea.

Ejemplos:
05
12
21
24
27
29
32
36
52
57

Si aparece como "57-WINGS", conservar preferentemente el valor completo disponible.

No inferir un ATA únicamente a partir de la descripción si el documento no proporciona información suficiente. Si no puede determinarse con seguridad, dejarlo pendiente de revisión.

# 4. POSIBILIDAD DE VARIAS EXPERIENCIAS EN UNA MISMA IMAGEN

Una fotografía o documento puede contener MÁS DE UNA tarea o registro.

La IA debe detectar si existen varios trabajos claramente diferenciados.

Si encuentra varios, devolver un array de experiencias.

Ejemplo:

[
{
fecha: "...",
matricula: "...",
localizacion: "...",
tipo_avion: "...",
tarea: "...",
ata: "..."
},
{
fecha: "...",
matricula: "...",
localizacion: "...",
tipo_avion: "...",
tarea: "...",
ata: "..."
}
]

La interfaz debe informar:

"Se han detectado 3 experiencias en este documento."

Y permitir revisar cada una individualmente antes de guardarlas.

# 5. NIVEL DE CONFIANZA

Si técnicamente es sencillo con las herramientas disponibles, almacenar también un nivel de confianza por campo:

* alta;
* media;
* baja.

No mostrar porcentajes artificialmente precisos.

En interfaz:

Alta → dato detectado normalmente.

Media → mostrar indicador discreto "Revisar".

Baja o no identificado → destacar el campo y solicitar al usuario que lo complete.

Esta funcionalidad es secundaria. No complicar la PoC si la implementación supone mucho desarrollo adicional.

# 6. AUTENTICACIÓN

Utilizar el sistema de autenticación nativo de Base44 si puede implementarse de forma sencilla.

Permitir:

* registro;
* inicio de sesión;
* cierre de sesión.

Cada usuario solo debe poder acceder a su propio perfil, documentos y experiencias.

No crear roles administrativos complejos.

Si alguna parte de autenticación avanzada complica innecesariamente la PoC, priorizar email + contraseña.

# 7. MODELO DE DATOS

Crear únicamente las entidades necesarias.

## Usuario / Perfil

Campos:

* id
* nombre
* apellidos
* email
* teléfono
* número de licencia EASA
* fecha de expiración de licencia
* número de empleado, opcional
* fecha de creación

Estos datos se consideran información fija del usuario y se utilizarán posteriormente en el reporte de experiencia.

## Experiencia

Campos:

* id
* user_id
* fecha
* matricula
* localizacion
* tipo_avion
* tarea_realizada
* ata
* created_at
* source_type
* estado_validacion

source_type puede distinguir, por ejemplo:

* IA/documento
* manual

estado_validacion:

* pendiente
* validado

Si se implementan niveles de confianza sin complicar excesivamente la solución, añadir la información necesaria.

## Documento

Crear esta entidad únicamente si es necesaria para el funcionamiento técnico.

Campos mínimos:

* id
* user_id
* archivo
* fecha_subida
* estado_procesamiento

No crear todavía un gestor documental complejo.

# 8. ALMACENAMIENTO DE IMÁGENES

Para esta PoC las fotografías son únicamente el origen utilizado para extraer la información.

No necesitamos construir un archivo histórico sofisticado de imágenes.

Si Base44 necesita almacenar temporalmente el archivo para procesarlo, hacerlo de la forma más sencilla posible.

La arquitectura debe permitir que en el producto futuro se decida si conservar o eliminar las imágenes originales.

No añadir ahora costes o complejidad innecesarios por almacenamiento permanente.

# 9. PANTALLAS

Crear las siguientes pantallas.

## LOGIN / REGISTRO

Pantalla sencilla y profesional.

Después de iniciar sesión, acceder al dashboard.

## DASHBOARD

Debe transmitir inmediatamente el propósito de la aplicación.

Encabezado:

"Mi experiencia aeronáutica"

Mostrar:

* nombre del usuario;
* número de licencia si existe;
* total de experiencias registradas;
* número de tipos de aeronave diferentes;
* última experiencia registrada.

Elemento principal:

Botón destacado:

"+ Añadir experiencia"

Debajo:

"Últimas experiencias"

Mostrar aproximadamente las cinco últimas.

No crear gráficos innecesarios.

## AÑADIR EXPERIENCIA

Esta es la pantalla más importante de toda la PoC.

Título:

"Añadir experiencia"

Dos opciones principales:

### Hacer una foto

En móvil debe intentar abrir directamente la cámara si técnicamente es posible desde navegador.

### Subir documento

Permitir subir:

* JPG;
* JPEG;
* PNG;
* PDF si la plataforma permite procesarlo sin añadir complejidad significativa.

Mostrar:

"Sube o fotografía tu parte de trabajo. Analizaremos el documento para extraer automáticamente la información de tu experiencia."

Después:

Botón:

"Analizar documento"

Mostrar durante el procesamiento:

"Analizando parte de trabajo..."

"Estamos identificando la información relevante."

Evitar animaciones largas o artificiales.

## REVISAR EXPERIENCIA

Después del procesamiento mostrar:

"Revisa la información detectada"

Texto:

"Hemos extraído estos datos del documento. Comprueba que sean correctos antes de guardarlos."

Mostrar los seis campos:

Fecha
Matrícula
Localización
Tipo de avión
ATA
Tarea realizada

TODOS deben ser editables.

Tarea realizada debe ser un textarea amplio.

Los campos no reconocidos deben quedar vacíos y claramente indicados para revisión.

Si se detectaron varias experiencias, utilizar tarjetas o un sistema sencillo que permita revisar todas.

Acciones:

"Guardar experiencia"

"Cancelar"

Opcional:

"Volver a analizar"

Después de guardar:

mostrar confirmación:

"Experiencia guardada correctamente."

Y ofrecer:

"Ver mi experiencia"

"Añadir otra"

# 10. MI EXPERIENCIA

Crear una pantalla llamada:

"Mi experiencia"

Mostrar todos los registros del usuario.

En escritorio utilizar tabla.

En móvil utilizar tarjetas legibles.

Columnas/datos principales:

Fecha
Matrícula
Localización
Tipo de avión
ATA
Tarea realizada

Permitir ordenar por fecha.

Añadir filtros:

* Desde
* Hasta
* Tipo de avión
* Matrícula
* ATA

Añadir búsqueda textual sobre tarea realizada.

Botón:

"Limpiar filtros"

Mostrar:

"X experiencias encontradas"

Permitir abrir una experiencia y editarla.

Permitir eliminarla, solicitando confirmación antes.

# 11. GENERACIÓN DEL REPORTE

Desde "Mi experiencia" debe existir un botón principal:

"Generar reporte"

El reporte debe utilizar las experiencias actualmente filtradas.

Antes de generar mostrar una pantalla/resumen:

"Generar reporte de experiencia"

Incluir:

Datos del usuario:

* nombre y apellidos;
* email;
* teléfono;
* licencia EASA;
* fecha de expiración de licencia.

Periodo:

* fecha mínima;
* fecha máxima.

Número de experiencias incluidas.

Tabla con:

* Fecha
* Matrícula
* Tipo de avión
* Localización
* ATA
* Tarea realizada

Generar un PDF profesional, limpio y preparado para entregar a una empresa.

Título:

"Registro de Experiencia Aeronáutica"

Subtítulo:

"Maintenance Experience Report"

No copiar visualmente ninguna marca comercial existente.

Puede inspirarse funcionalmente en un logbook profesional, pero debe tener identidad propia.

Incluir al final:

"Declaro que la información incluida en este registro corresponde a mi experiencia profesional."

Dejar espacio para:

Fecha

Firma

El objetivo de la PoC es que el usuario pueda descargar este PDF.

Si la generación PDF real supone una limitación técnica importante de Base44, implementar la solución más sencilla y fiable posible que permita generar o imprimir/guardar un documento equivalente en PDF.

# 12. PERFIL

Crear pantalla:

"Mi perfil"

Campos editables:

Nombre
Apellidos
Email
Teléfono
Número de licencia EASA
Fecha de expiración
Número de empleado

Botón:

"Guardar cambios"

Estos datos alimentarán automáticamente el encabezado del reporte.

# 13. ALTA MANUAL

Aunque la principal funcionalidad sea la extracción mediante IA, permitir también:

"Añadir manualmente"

Mostrar exactamente los mismos seis campos.

Esto sirve como fallback cuando un documento sea imposible de interpretar.

# 14. NAVEGACIÓN

Navegación sencilla.

En escritorio:

* Inicio
* Mi experiencia
* Añadir experiencia
* Mi perfil

En móvil utilizar navegación adaptada, priorizando especialmente:

"Añadir experiencia"

El proceso de registrar una experiencia debe requerir el mínimo número posible de pasos.

# 15. DISEÑO

Crear una identidad provisional.

No utilizar logos ni identidad visual de Ryanair, Skyhook, EASA ni ninguna aerolínea.

Estética:

* profesional;
* aeronáutica;
* tecnológica;
* sobria;
* moderna;
* limpia;
* alta legibilidad;
* mobile-first.

Paleta sugerida:

* azul marino oscuro como color principal;
* blanco;
* grises muy claros;
* azul técnico como color de acción.

Evitar:

* gradientes excesivos;
* estética futurista exagerada;
* exceso de gráficos;
* iconografía decorativa;
* animaciones innecesarias;
* interfaces densas.

Utilizar iconos simples relacionados con:

* cámara;
* documento;
* avión;
* historial;
* perfil;
* descarga.

# 16. RESPONSIVE

La aplicación debe funcionar especialmente bien desde móvil.

El caso de uso principal ocurrirá desde el teléfono del mecánico.

Priorizar:

* botones grandes;
* campos fáciles de editar;
* cámara accesible;
* carga rápida;
* revisión cómoda;
* navegación sencilla.

También debe funcionar correctamente en escritorio.

# 17. DATOS DEMO

Crear algunos registros demo para que la aplicación no aparezca vacía inicialmente en entorno de demostración.

Utilizar datos ficticios y claramente de demostración.

Ejemplo:

Fecha: 27/11/2025
Matrícula: EC-DEMO
Localización: ACE
Tipo avión: B737-800
ATA: 32
Tarea realizada: "INSPECCIÓN FUNCIONAL DEL SISTEMA DE TREN DE ATERRIZAJE"

No utilizar nombres, emails, números de licencia ni otros datos personales reales procedentes de los documentos proporcionados.

# 18. IA / PROCESAMIENTO DE DOCUMENTOS

Implementar el procesamiento utilizando la alternativa más sencilla disponible dentro de Base44 o mediante una integración de IA compatible.

El procesamiento debe aceptar una imagen y devolver exclusivamente información estructurada.

Utilizar conceptualmente este schema:

{
"experiencias": [
{
"fecha": null,
"matricula": null,
"localizacion": null,
"tipo_avion": null,
"tarea_realizada": null,
"ata": null
}
]
}

Reglas del modelo:

* analizar todo el documento;
* no asumir posiciones fijas;
* no inventar valores;
* interpretar abreviaturas aeronáuticas;
* distinguir defecto de acción realizada;
* priorizar rectification/action taken/operation performed;
* detectar varias tareas si existen;
* conservar el texto técnico relevante;
* devolver null cuando un dato no pueda identificarse;
* nunca guardar automáticamente sin revisión humana.

# 19. SEGURIDAD MÍNIMA

Cada experiencia debe pertenecer a un usuario.

Un usuario no debe poder consultar experiencias de otro usuario.

No exponer claves API en frontend.

Las llamadas a servicios externos de IA deben realizarse desde backend/server-side cuando corresponda.

Aplicar las protecciones estándar de Base44 sin desarrollar una arquitectura empresarial compleja.

# 20. NO IMPLEMENTAR TODAVÍA

NO desarrollar en esta PoC:

* Stripe;
* pagos;
* suscripciones;
* planes Free/Pro;
* facturación;
* panel administrativo;
* empresas;
* supervisores;
* roles complejos;
* validación oficial por compañías;
* notificaciones;
* emails comerciales;
* analytics avanzados;
* marketplace;
* aplicación móvil nativa;
* App Store;
* Google Play;
* infraestructura empresarial específica;
* almacenamiento documental avanzado;
* entrenamiento específico para cada aerolínea;
* integraciones con sistemas de Ryanair u otras aerolíneas.

No añadir funcionalidades no solicitadas solo porque parezcan útiles.

# 21. CRITERIO DE ÉXITO DE LA POC

La PoC se considerará funcional si podemos demostrar este escenario:

1. Un usuario entra en la aplicación.
2. Accede a "Añadir experiencia".
3. Fotografía o sube un TLB real.
4. La aplicación analiza el documento mediante IA.
5. Obtiene Fecha, Matrícula, Localización, Tipo de avión, Tarea realizada y ATA siempre que sean identificables.
6. El usuario revisa los resultados.
7. Corrige cualquier error.
8. Guarda la experiencia.
9. La experiencia aparece en "Mi experiencia".
10. Puede filtrar sus registros.
11. Puede generar un reporte profesional con la experiencia seleccionada.

Este flujo tiene prioridad absoluta sobre cualquier otra funcionalidad.

# 22. FORMA DE TRABAJAR

Construye primero la estructura completa de la aplicación y el flujo principal.

No dediques tiempo inicialmente a perfeccionar funcionalidades secundarias.

Si alguna característica requiere una integración externa o configuración que no puedas completar automáticamente, crea correctamente la interfaz y el flujo y explícame exactamente qué configuración necesito realizar.

Antes de añadir complejidad, prioriza siempre la solución más sencilla que permita demostrar el caso de uso.

Quiero terminar con una PoC que pueda enseñar a un cliente real, no simplemente con un mockup visual.

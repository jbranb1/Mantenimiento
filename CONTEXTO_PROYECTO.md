# Contexto del proyecto: Control Industrial

## Propósito

Aplicación web en español para llevar localmente el inventario de máquinas de una planta industrial y administrar el catálogo de tipos de máquina. La implementación funcional está concentrada en [`index.html`](./index.html), que contiene el HTML, los estilos CSS y el JavaScript de la aplicación.

Este documento resume el comportamiento y las decisiones visibles en la implementación actual para servir como punto de partida en futuras sesiones. Si el código cambia, el código fuente sigue siendo la referencia definitiva y este contexto debe actualizarse junto con los cambios relevantes.

## Estado actual

- Aplicación de una sola página, sin backend ni proceso de compilación aparente.
- Al cargar la página se muestra una pantalla de bienvenida; el usuario debe seleccionar **Entrar al sistema** para acceder. Después aparece el área de trabajo con una ilustración industrial adaptable, sin módulos seleccionados, hasta que el usuario elija uno. La bienvenida aparece en cada carga y no guarda la elección.
- Dos módulos activos: **Máquinas** y **Tipos de máquina**.
- **Repuestos** y **Mantenimientos** se muestran como módulos próximamente disponibles; no tienen funcionalidad implementada.
- Los datos se persisten en `localStorage` del navegador. No se sincronizan entre navegadores, dispositivos o perfiles.
- La interfaz y los mensajes están en español.

## Funcionalidad

### Máquinas

- Registrar, editar y eliminar máquinas.
- Campos: código, marca, tipo de máquina, capacidad y fotografía opcional.
- El código se recorta y se normaliza a mayúsculas al guardar; no se permiten códigos duplicados sin distinguir mayúsculas/minúsculas.
- La selección de tipo se alimenta del catálogo de tipos registrados.
- Buscar por código, marca, tipo o capacidad. La búsqueda no distingue mayúsculas/minúsculas.
- Ver una fotografía ampliada en un cuadro de diálogo.
- Exportar el inventario a un libro `.xlsx` usando ExcelJS. El botón se habilita cuando hay máquinas registradas.
- Eliminar requiere confirmación.

### Tipos de máquina

- Registrar, editar y eliminar tipos, cada uno con nombre y descripción.
- El nombre no puede duplicarse sin distinguir mayúsculas/minúsculas.
- Buscar por nombre o descripción.
- No se permite borrar un tipo que esté asignado a una máquina.
- Al cambiar el nombre de un tipo, también se actualiza el tipo en las máquinas que lo usaban.
- Al iniciar por primera vez se proponen dos tipos predeterminados: **Motor de potencia** y **Cilindro neumático**.
- Si hay máquinas guardadas cuyo tipo no aparece en el catálogo, el inicio incorpora esos tipos al catálogo con una descripción de recuperación.

### Fotografías

- Se aceptan archivos de imagen de hasta 15 MB.
- Antes de guardar, la imagen se reduce para que su lado mayor mida como máximo 1200 px y se convierte a JPEG con calidad 0.78.
- La imagen optimizada se guarda como data URL junto con la máquina en `localStorage`.
- Durante el procesamiento se deshabilita el botón de guardar y se muestra una vista previa.

## Datos y persistencia

Claves actuales de `localStorage`:

- `control-industrial-maquinas-v1`: arreglo de máquinas.
- `control-industrial-tipos-maquina-v1`: arreglo de tipos de máquina.
- `control-industrial-visitas-v1`: contador de visitas (entero). Se incrementa una vez por sesión de pestaña (marca en `sessionStorage`: `control-industrial-visita-registrada`) y se muestra bajo el reloj de la bienvenida. Es local a cada navegador/perfil, no global; si el almacenamiento falla, el contador se oculta.

Forma de los registros que construye la aplicación:

```text
Máquina: { id, code, brand, type, capacity, photoData }
Tipo:    { id, name, description }
```

Los identificadores se crean con `crypto.randomUUID()` cuando está disponible, con `Date.now()` como alternativa. Los tipos predeterminados tienen identificadores fijos.

La carga de máquinas espera un arreglo JSON y, ante datos malformados o incompatibles, inicia con una lista vacía. Los errores de escritura se presentan mediante avisos en la interfaz; no hay almacenamiento remoto ni recuperación desde un servidor.

## Interfaz y dependencias

- Diseño adaptable para escritorio y móvil.
- Pantalla inicial de bienvenida con acceso explícito al sistema y un reloj discreto (fecha y hora actuales, actualizado cada segundo) en la esquina inferior izquierda; al entrar se muestra el espacio de trabajo vacío, sin módulo seleccionado. El usuario puede elegir **Máquinas** o **Tipos de máquina** en el menú; el logotipo regresa al espacio vacío.
- El espacio de trabajo inicial presenta `assets/inicio-industrial.png` como ilustración adaptable y ofrece accesos directos a los cuatro módulos del panel lateral. **Máquinas** y **Tipos de máquina** son accesibles; **Repuestos** y **Mantenimientos** permanecen deshabilitados mientras estén próximamente disponibles.
- Solo la pantalla de bienvenida muestra, en pantallas de 1200 px o más, un carrusel de tres ilustraciones (`assets/bienvenida-*.png`) en el lado derecho: ~480 px de ancho, opacidad 60 %, colocado en una capa detrás del texto de bienvenida, rotación cada 5 s con fade de 700 ms, indicadores clicables y pausa al pasar el cursor o enfocar los controles. En pantallas menores se oculta para conservar el mensaje de bienvenida.
- En móviles, cada módulo activo incluye un botón **Volver al inicio** para regresar a la pantalla de accesos.
- Paleta industrial: menú en grafito, superficies en blanco y gris acero, y acciones principales en ámbar con texto oscuro. Los indicadores de éxito y error conservan sus colores de estado.
- Navegación entre módulos dentro de la misma página; actualiza el título y el estado activo del menú.
- El panel de navegación se puede contraer y expandir con un botón en la barra superior. En móvil se abre como panel lateral y se cierra al seleccionar un módulo, tocar fuera o presionar Escape; esta preferencia no se guarda.
- Bootstrap 5.3.8, Bootstrap Icons 1.13.1, ExcelJS 4.4.0 y fuentes DM Sans / IBM Plex Mono se cargan desde CDN.
- Algunas funciones dependen de APIs del navegador como `localStorage`, `FileReader`, `canvas`, `<dialog>` y `crypto.randomUUID`.

## Guía para cambios futuros

1. Leer este archivo primero para recuperar el contexto y consultar en [`index.html`](./index.html) el fragmento concreto que se va a modificar.
2. Mantener sincronizados ambos módulos cuando un cambio afecte a máquinas y tipos (por ejemplo, opciones del selector, renombrado o eliminación).
3. Preservar la validación del navegador, la unicidad de códigos/nombres, los mensajes visibles y los estados vacíos de las tablas.
4. Si se cambia la estructura o las claves de `localStorage`, considerar los datos existentes: una actualización puede dejar datos incompatibles o requerir una migración.
5. Tener presente que este almacenamiento es local al navegador; no describirlo como base de datos compartida ni como respaldo.
6. Actualizar este documento si cambian el alcance, los modelos de datos, las reglas de negocio, las dependencias o la persistencia.

## Límites conocidos del contexto

Este resumen documenta la implementación inspeccionada de `index.html`; no implica que se hayan ejecutado pruebas funcionales ni que exista un backend, configuración de despliegue o historial de decisiones adicional.

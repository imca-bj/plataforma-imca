# Agenda operativo · Documento de referencia

Estado: funcionando en pruebas. Pendiente de diseño (Lovable) y de carga de escuelas desde Excel.

## 1. Qué controla y en qué se diferencia de la agenda de atención

- **Agenda de atención:** citas de atención psicológica y expedientes de pacientes.
- **Agenda operativo:** pláticas y eventos que el IMCA da fuera de la oficina (escuelas, centros de trabajo, jornadas y eventos). Registra a quién se dio, dónde, qué pasó y el reporte con grupos y personas atendidas.

Cada actividad es una **plática** o un **evento**.

## 2. Pantallas

| Pantalla | Qué hace | Quién la ve |
|---|---|---|
| Agenda | Vista por día. Programar, abrir y cambiar estado. Filtro "Todo el equipo" / "Solo las mías". Muestra convocados en cada tarjeta. | Todos (operativos solo ven las suyas) |
| Detalle de la actividad | Datos generales, convocados, estado y botón "Generar reporte" o "Continuar reporte". | Según Agenda |
| Reporte | Pantalla propia con el formulario del reporte. Botón para regresar a los datos. | Según el reporte |
| Programar plática / evento | Formulario con los campos de cada tipo. | Administrador, coordinador y supervisión |
| Historial | Actividades de un mes. Descarga CSV si tiene permiso. | Todos (operativos solo ven las suyas) |
| Dashboard | Pláticas, eventos, realizadas, reportes enviados, personas y grupos atendidos. | Solo quien tenga "ve dashboard" |
| Catálogos | Temas, tipos de evento, turnos, público, qué llevar, cómo se darán y escuelas. | Solo administrador |

## 3. Recorrido habitual

1. **Programa:** administrador, coordinador o supervisión crea la plática o el evento.
2. **Convoca:** marca a los convocados. Solo ellos ven la actividad.
3. **Realiza:** el operativo convocado marca Realizada o No realizada.
4. **Reporta:** desde el día de la actividad, el operativo genera el reporte, lo guarda como borrador las veces que quiera y lo envía al terminar.
5. **Revisa:** administrador o coordinación consulta el historial y el dashboard.
6. **Anula:** solo Ana Luisa y Alberto pueden anular un reporte.

## 4. Campos de la plática

- Clase: Plática.
- Fecha y horario: uno o varios horarios (horario partido). Cada horario tiene hora de y hora a.
- Tipo de lugar: Secundaria, Preparatoria, Universidad o Centro de trabajo.
  - Secundaria, Preparatoria o Universidad: campo con lista de escuelas de ese nivel (búsqueda por texto). Si no hay lista cargada para ese nivel, se escribe a mano.
  - Centro de trabajo: texto libre.
- Turno: Matutino, Vespertino, Ambos, Modular.
- ¿A quién se le dará?: Alumnos, Padres y madres, Trabajadores.
- Plática (tema): catálogo de temas.
- Grupos esperados: número.
- Ubicación: enlace de Google Maps (campo para pegar), Supermanzana y Colonia.
- ¿Cómo se darán las pláticas?: una sola opción (Pláticas simultáneas, Biblioteca, Un solo salón, Varios salones).
- Qué llevar: varias opciones.
- Contacto en el lugar (opcional): Nombre, Puesto, Teléfono.
- Notas (opcional).
- Convocados: lista de personal marcado como convocable.

## 5. Campos del evento

- Clase: Evento.
- Fecha y horario: igual que la plática.
- Tipo de evento: Feria de la legalidad, El futbol nos une, Cancún nos une, Feria de la Paz, Jornada de bienestar, Otro.
- Lugar del evento: texto libre.
- Ubicación: enlace de Google Maps (campo para pegar).
- Convocados.
- Qué llevar.
- Contacto en el lugar (opcional).
- Notas (opcional).

No tiene tipo de lugar, turno, público, tema, grupos esperados, supermanzana, colonia ni cómo se darán.

## 6. Campos del reporte

- Número de pláticas.
- Grupos atendidos: grado, grupo, hombres y mujeres (un renglón por grupo, se pueden agregar varios).
- ¿Hubo incidencia?: Sí / No. Si es Sí, las observaciones son obligatorias al enviar.
- Observaciones.
- Fotos: hasta 3.
- Documentos: hasta 3.
- Folio automático (PLA/año/0000).

## 7. Estados

**Actividad:** Programada, En proceso, Realizada, Cancelada, No realizada.
- Administrador, coordinador y supervisión cambian cualquier estado.
- El operativo convocado solo marca Realizada o No realizada.

**Reporte:** Borrador, Enviado, Anulado.

## 8. Reglas de reportes

- El reporte se llena **desde el mismo día de la plática o evento**, no antes. Aplica a plática y a evento.
- Se puede guardar como borrador las veces que se necesite. No se pierde lo capturado.
- Al enviar, el reporte queda enviado.
- Después de enviado, solo se puede editar **el mismo día de la actividad**. Al día siguiente queda cerrado para todos, excepto para anular.
- Solo Ana Luisa y Alberto pueden anular.

## 9. Permisos

| Persona | Rol | Ve | Crea y edita | Convoca | Cambia estado | Reporta | Anula | Dashboard | Descarga |
|---|---|---|---|---|---|---|---|---|---|
| Alberto | Administrador | Todo | Todo | Sí | Sí | Sí | Sí | Sí | Sí |
| Ana Luisa | Administrador | Todo | Todo | Sí | Sí | Sí | Sí | Sí | Sí |
| Juan Carlos | Administrador | Todo | Todo | Sí | Sí | Sí | No | Sí | Sí |
| Guillermo | Coordinador | Todo | Sí | Sí | Sí | Sí | No | Sí | No |
| Rosy | Coordinador | Todo | Sí | Sí | Sí | Sí | No | Sí | No |
| Luis | Supervisión | Todo | Sí | Sí | Sí | Sí | No | No | No |
| Carolina | Técnica | Todo | No | No | No | No | No | Sí | Sí |
| Operativos (10) | Operativo | Solo donde está convocado | No | No | Solo Realizada / No realizada | Sí, solo sus actividades | No | No | No |

Juan Carlos, Luis y Carolina pueden ser convocados. Operativos y Carolina figuran en la lista de convocados según lo que se marque al programar.

## 10. Acceso

- Entran con su correo institucional (@imcabj.gob.mx).
- En el primer inicio de sesión se pide su Gmail una sola vez. Se guarda para el calendario.
- Google Calendar no está conectado en este módulo todavía.

## 11. Pendiente

- Diseño visual (Lovable).
- Carga de la lista completa de escuelas de Cancún desde Excel.
- Formato de descarga de reportes (CSV hoy; PDF por confirmar).
- Conexión con Google Calendar (por ahora no).
- Quitar un archivo ya subido a un reporte (no está hecho).

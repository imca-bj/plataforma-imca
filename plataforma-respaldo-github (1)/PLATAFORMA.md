# Plataforma IMCA

Documento vivo. Es la fuente de verdad del proyecto: si algo cambia, se actualiza aquí.

## Objetivo

Concentrar en un solo sitio web (dominio propio, Netlify) las apps internas del IMCA, con un solo inicio de sesión y permisos por rol.

## Alcance

**Dentro de la plataforma**
- Agenda general
- Agenda de atención terapéutica
- Reportes de atención
- Pláticas
- Vacaciones, incapacidades y faltas
- Control de gasolina
- Módulos pequeños que vayan surgiendo

**Fuera de la plataforma (proyectos propios)**
- Expedientes: proyecto de Supabase independiente, con sus propios accesos y datos.
- Finanzas: proyecto de Supabase independiente, con su propio control de acceso.

## Stack

- **Frontend:** HTML, CSS y JavaScript en un sitio estático, desplegado en Netlify.
- **Backend y base de datos:** Supabase (Postgres, Auth, Row Level Security, Storage, Edge Functions, pg_cron).
- **Código:** GitHub. Cada cambio se registra y se puede revertir. Netlify despliega desde GitHub.
- **Estado actual de las apps:** Google Apps Script + Google Sheets (agenda, pláticas, reportes, vacaciones y gasolina por migrar).

## Principios

1. Migrar de una app a la vez. La versión en Apps Script sigue activa hasta que la nueva se valida con el equipo.
2. No cambiar la forma de trabajar de una app que ya funciona sin avisar y pedir permiso antes.
3. Permisos por rol desde el inicio. Ningún usuario ve datos de otro rol o departamento sin permiso.
4. Los datos sensibles (nombres de usuarios, atención, situación de calle) se manejan con el mínimo necesario en pantalla y en notificaciones.
5. Los datos viven en cuentas institucionales, no en cuentas personales.

## Roles (base)

Tomados de las apps actuales. Por revisar al migrar cada módulo.

- **admin:** acceso total del módulo.
- **agenda / coordinación:** agenda, confirma y cancela citas.
- **terapeuta:** ve y gestiona solo su agenda y sus usuarios.
- **consulta:** ve información y reportes, no modifica lo de otros.
- **operativo:** atiende reportes asignados a su departamento.

## Módulos

| Módulo | Estado | Notas |
|---|---|---|
| Base (login, roles, menú, dominio) | Por construir | Primer paso |
| Control de gasolina | Nuevo | Primer módulo de prueba: no hay datos que migrar |
| Pláticas | Apps Script activa | Migrar después de la base |
| Vacaciones | Propuesta | Datos laborales: permisos estrictos |
| Reportes de atención | Apps Script activa | Lo consultan para el flujo de expedientes |
| Agenda general | Apps Script activa | Migrar con cuidado; hoy también usa Google Calendar |
| Agenda de atención terapéutica | Apps Script activa | Datos clínicos: mismo cuidado que expedientes |

## Notificaciones (por decidir)

- **Push web** para terapeutas, si instalan la app en la pantalla de inicio. Gratuito.
- **Google Calendar**, como hoy, con alerta una hora antes. Funciona, pero depende de Google.
- **WhatsApp Business API** para recordatorios a usuarios. Requiere verificación con Meta y pago por mensaje.
- Pendiente de decidir: canal, contenido mínimo del aviso (solo iniciales) y quién lo autoriza.

## Pendientes abiertos

- Crear el repositorio en GitHub y conectarlo a Netlify.
- Crear el proyecto de Supabase de la plataforma.
- Definir permisos definitivos por rol para vacaciones y agenda terapéutica.
- Definir canal de notificaciones.
- Inventariar qué servicios de Google usa cada app actual (Drive, Gmail, Calendar, PDFs).
- Revisar con Jurídico o Transparencia el alojamiento y tratamiento de datos personales.

## Decisiones tomadas

- Expedientes y finanzas quedan aparte (nov 2026).
- La agenda no está ligada a expedientes; son apps independientes.
- Migración por módulos, empezando por la base y control de gasolina.

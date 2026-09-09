# Marbella Nails

Sitio web y panel de administración para **Marbella Nails**, estudio de manicuría y cosmetología de Mar en Itaembe Guazú, Posadas, Misiones.

- 🌐 Sitio en producción: https://pagina-de-estetica-vert.vercel.app/
- 🔒 Panel admin: https://pagina-de-estetica-vert.vercel.app/admin.html (acceso restringido, no listado en el menú)

Todo el proyecto son **dos archivos HTML autocontenidos** (HTML + CSS + JS en el mismo archivo, sin build ni dependencias de instalación) más las imágenes fuente sin usar en `img/`.

---

## `index.html` — Sitio principal

### Diseño
- Tema claro (marca real: dorado + vino sobre fondo crudo) con **modo oscuro opcional**, toggle en el header, se guarda la preferencia en `localStorage`.
- Mobile-first, totalmente responsive.
- Tipografía `Fraunces` (títulos) + `Work Sans` (texto), Google Fonts.

### Secciones
1. **Header** — logo, navegación, botón de tema, menú hamburguesa en mobile.
2. **Hero** — imagen + llamado a reservar.
3. **Servicios** — Manicuría (Semipermanente, Nivelación/Kapping, Extensiones Softgel, Polygel, Press On) y Cosmetología (Limpieza básica, Limpieza profunda, Personalizado).
4. **Turnero (reserva de turnos)** — wizard paso a paso dentro de un `<dialog>`:
   1. Categoría (Manicuría / Cosmetología)
   2. Servicio
   3. Nombre y email
   4. Fecha (calendario propio hecho a medida, sin `<input type="date">` nativo)
   5. Horario disponible (bloquea automáticamente los horarios ya reservados)
   6. Resumen y confirmación
   - Al confirmar: guarda la reserva en **Firestore**, envía **email de confirmación automático** (EmailJS) y abre **WhatsApp** con el mensaje pre-armado para coordinar con Mar.
   - Revisa disponibilidad en tiempo real contra Firestore para no permitir turnos superpuestos (incluye un re-chequeo justo antes de guardar, por si alguien reservó el mismo horario segundos antes).
5. **Cuidado de uñas** — tips.
6. **Onicofagia** — sección informativa.
7. **El Estudio** — fotos reales del espacio.
8. **Trabajos** — galería de trabajos reales con filtro por técnica y lightbox.
9. **Ubicación** — mapa de Google Maps embebido.
10. **FAQ** — acordeón.
11. **Footer**.

### Integraciones externas
| Servicio | Uso |
|---|---|
| **Firebase Firestore** | Guarda las reservas (`colección reservas`) y consulta disponibilidad en tiempo real. |
| **EmailJS** | Envía el email de confirmación automático al cliente. |
| **WhatsApp** (`wa.me`) | Abre un chat con el mensaje de la reserva pre-cargado para coordinar con Mar. |

---

## `admin.html` — Panel de administración

Pensado para que Mar vea y gestione las reservas sin tocar código. No está enlazado desde el sitio público (solo accesible por URL directa) y lleva `<meta name="robots" content="noindex, nofollow">` para no aparecer en buscadores.

### Login
- **Firebase Authentication** real (no hay contraseña hardcodeada).
- Dos formas de entrar: **email + contraseña** o **"Continuar con Google"**.
- Lista blanca de acceso: solo la cuenta configurada en `CORREO_PERMITIDO` puede ver el panel — cualquier otra cuenta que inicie sesión es rechazada y desconectada automáticamente.
- Mismo modo oscuro/claro que el sitio principal (comparten la preferencia guardada en `localStorage`).

### Funciones del panel
- Lista todas las reservas agrupadas por fecha, con encabezados en español ("Hoy — martes 9 de septiembre", etc.).
- Filtro: **Próximas** / **Hoy** / **Todas**.
- Botón **Actualizar** para recargar sin refrescar la página.
- **Cancelar reserva** — borra el turno de Firestore (requiere estar autenticada; protegido por las reglas de seguridad de Firestore).

---

## Seguridad de datos (Firestore)

Reglas configuradas en la consola de Firebase (Firestore → Reglas):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /reservas/{doc} {
      allow read: if true;
      allow create: if request.resource.data.keys().hasAll(['fecha','hora','duracion','servicio','nombre'])
        && request.resource.data.fecha is string
        && request.resource.data.hora is string
        && request.resource.data.duracion is number;
      allow delete: if request.auth != null;
      allow update: if false;
    }
  }
}
```

- Cualquiera puede **leer** (necesario para calcular disponibilidad desde el sitio público) y **crear** una reserva con el formato correcto.
- Solo un usuario **autenticado** (login del panel admin) puede **borrar**.
- Nadie puede **modificar** una reserva ya creada.

---

## Estructura del repo

```
index.html         Sitio principal (HTML + CSS + JS, imágenes en base64 embebidas)
admin.html          Panel de administración
_localserver.js      Servidor estático simple en Node.js para probar en localhost:5500
img/                 Fotos originales de respaldo (no se usan directamente, ya están embebidas en index.html)
```

### Correr en local

```
node _localserver.js
```

Abre en `http://localhost:5500` (sitio) y `http://localhost:5500/admin.html` (panel).

---

## Pendientes antes de entregar a Mar

- [ ] Cambiar el número de WhatsApp de prueba por el real de Mar (`WHATSAPP_NUMBER` en `index.html`).
- [ ] Cambiar `CORREO_PERMITIDO` en `admin.html` por el email real de Mar.
- [ ] Cargar precios reales por servicio.
- [ ] Agregar sección "Sobre Mar" con bio real.
- [ ] Agregar políticas de cancelación y métodos de pago.

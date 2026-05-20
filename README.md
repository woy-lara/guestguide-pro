# GuestGuide Pro

> Plataforma SaaS multi-tenant de concierge digital para hoteles. Conecta huéspedes con experiencias locales mediante códigos QR, validación de vouchers y comisiones automáticas.

🌎 **Demo en vivo**: https://woy-lara.github.io/guestguide-pro/
🏔️ **Ubicación piloto**: Boquete, Panamá

---

## El problema

Los hoteles boutique pierden ingresos por dos vías:

1. **No monetizan las experiencias locales** que sus huéspedes contratan por fuera (tours, spa, gastronomía).
2. **El concierge tradicional no escala** — depende de un humano memorizando ofertas y comisiones.

Los proveedores turísticos pierden visibilidad porque **no tienen acceso directo** al huésped dentro del hotel.

## La solución

Una plataforma de tres lados:

| Rol | Lo que obtiene |
|---|---|
| **SaaS Admin** | Gestiona la red completa de hoteles + proveedores, métricas, facturación |
| **Hotel** | Builder visual de su microsite, QR codes por ubicación, comisiones por venta |
| **Proveedor turístico** | Ficha pública, vouchers validados por QR, métricas de demanda |

El huésped escanea un QR en el lobby → ve experiencias → reserva → paga al hotel → hotel recibe comisión → proveedor recibe voucher validado.

## Features principales

- 🏨 **Multi-tenant** con 3 roles distintos y vistas independientes
- 📱 **Mobile preview integrada** — ves la app del huésped sin salir del dashboard
- 🔲 **QR codes reales** con tracking de escaneos, métricas y deep-links
- 📊 **Analytics** con charts SVG custom (sin dependencias pesadas)
- 🌍 **i18n** Español / Inglés en la app del huésped
- 📜 **Audit log** completo con seed de 60 días
- 🔔 **Centro de notificaciones** contextual por rol
- ⌘K **Command palette** para navegación rápida
- 💾 **Backup/restore** del estado completo (export/import JSON)
- 🎨 **Hotel builder** con 9 tabs (Datos, Diseño, Equipo, Habitaciones, Servicios, Políticas, Marketing, PMS, QRs)
- 🤖 **AI Assistant Hub** para sugerencias y auditorías
- 🗺️ **Mapas** de hoteles y proveedores (Leaflet)

## Stack

- **HTML único** (single-file SPA) — sin build, sin backend, sin frameworks
- **Tailwind CSS** (CDN) + design tokens custom (paleta sage/olive)
- **Vanilla JavaScript** — sin React, sin Vue, sin nada
- **Lucide** para íconos
- **Leaflet** para mapas
- **qrcode-generator** para QR reales

Todo el estado vive en memoria (objeto `DATABASE`). La persistencia es vía `localStorage` (preferencias) o export/import JSON manual.

## Cómo correr local

```bash
# Clona el repo
git clone https://github.com/woy-lara/guestguide-pro.git
cd guestguide-pro

# Sirve con cualquier static server
python3 -m http.server 3000
# o
npx serve .
```

Abre http://localhost:3000 y empieza a explorar. El selector de rol en la esquina superior te permite cambiar de SaaS Admin a Hotel o Proveedor.

## Estructura del proyecto

```
guestguide-pro/
├── index.html       # Todo está aquí (~10,700 líneas)
├── CLAUDE.md        # Guía para asistentes de IA / colaboradores
├── README.md        # Este archivo
└── .gitignore
```

Sí, leíste bien: **un solo archivo HTML**. Es deliberado — el objetivo en esta fase es validar la propuesta, no escalar.

## Estado actual

Este es un **prototipo funcional para recolección de feedback**. No está en producción. No conecta con sistemas reales de pago, PMS, ni email. Todo lo que se ve está respaldado por datos seed realistas.

### Roadmap inmediato
- [ ] Pantalla de login simulado + selección visual de rol
- [ ] Modo oscuro
- [ ] Onboarding modal para primeros visitantes
- [ ] Settings page mínima
- [ ] Empty states mejorados
- [ ] Hotel Builder tabs restantes en EN (Team, Rooms, Services, Policies, Marketing, PMS)

### Para validación real
- [ ] Backend mínimo (Supabase / Firebase)
- [ ] Auth real (magic link via email)
- [ ] Persistencia de datos
- [ ] Integración con un PMS real para piloto

## Feedback bienvenido

Si abriste esto y tienes 5 minutos:

1. Cambia entre roles desde el dropdown superior
2. Abre el **Hotel Builder** y juega con la pestaña "Apariencia"
3. Mira el **mobile preview** mientras editas — refleja cambios en vivo
4. Prueba el **Command Palette** (⌘K)
5. Dime qué te confunde, qué te gustó, qué falta

Abre un issue o escríbeme directo.

## Autor

Carlos Lara — Boquete, Panamá

---

_Construido con paciencia y café de Finca Lérida._

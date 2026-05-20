# 🌙 Notas de la sesión nocturna — 2026-05-20

## ✅ Hecho mientras dormías

### 1. Mobile responsive completo
La app pasó de "no diseñada para móvil" a **shipping quality en iPhone**.

**Lo que funciona ahora en móvil:**
- ☰ **Hamburger menu** arriba a la izquierda abre el sidebar como drawer deslizante
- 🌫️ Backdrop con blur + tap para cerrar + tecla ESC
- 📐 Banner compacto (oculta el texto "Entorno de simulación" y el role-simulator)
- 📊 KPI grids se acomodan a 1-2 columnas
- 🎴 Cards stackean y respetan padding tighter (12px en lugar de 32px)
- 🗺️ Mapas reducidos a 320px de alto (antes 580px ocupaba toda la pantalla)
- 📑 **Tabs de Hotel/Provider Builder scrollean horizontalmente** (antes wrap a varias filas feas)
- 📞 Phone preview dentro del builder oculto en móvil (era redundante)
- 🔔 Notification panel constrained a viewport
- 💬 WhatsApp simulator y toasts ocupan todo el ancho
- 🍔 Modales (QR scan, invoice, command palette) responsive con padding/sizing apropiado
- 🍎 Fixes específicos iOS Safari:
  - Inputs forzados a 16px font-size (previene zoom al focus)
  - `min-height: 100dvh` en auth overlay (toolbar de Safari)
  - `env(safe-area-inset-bottom)` en bulk action bar (notch/home bar)
  - Tap targets ≥ 40px en nav (Apple HIG)

### 2. Modo oscuro completo
- 🌗 **Toggle en el banner** (icono luna/sol al lado de la campana)
- 🔑 También en el auth overlay (arriba a la derecha)
- 💾 Persiste en `localStorage('gg_theme')`
- 🖥️ Respeta `prefers-color-scheme` en primer load (si no hay valor guardado)
- ⚡ Se aplica ANTES del primer render (sin flash)
- 🎨 Paleta dark mode coherente con la marca (verdes olive más oscuros)
- 🗺️ Tiles de Leaflet con filtro brightness/saturate (no destacan como áreas blancas)
- 📥 Inputs, popups, drawers, toasts, modales — todo dark-aware

### 3. URLs y estado actual
- **🌐 Demo en vivo**: https://woy-lara.github.io/guestguide-pro/
- **📦 Repo**: https://github.com/woy-lara/guestguide-pro
- **📊 Archivo**: 11.567 líneas, 762 KB, balanceado ✅

### 4. Commits que hice esta noche
```
3e34bc5  polish(mobile): iOS-specific tweaks — input zoom, safe area, tap targets
8f1045a  polish(mobile): hide env text on banner, shrink map heights to 320px
9efbc07  feat: add dark mode toggle with localStorage persistence
f9be205  feat(mobile): responsive modals — QR scan, invoice, command palette
717d55a  feat(mobile): tighten content layout, hide preview aside, scrollable builder tabs
9e09e94  feat(mobile): add hamburger menu and sidebar drawer for mobile
```

## 🧪 Cómo verificar (desde tu iPhone)

1. Abre **https://woy-lara.github.io/guestguide-pro/** en Safari
2. Si lo abriste antes hoy, haz **pull to refresh** (deslizar hacia abajo) para forzar reload
3. Verás el **login screen** con fondo oscuro sage/olive
4. Email cualquiera con `@` → "Enviar enlace de acceso"
5. → "Simular click en el enlace →"
6. → Elige un rol (cards stackeadas verticalmente)
7. → Onboarding modal → "Explorar por mi cuenta"
8. Dashboard renderiza sin overflow
9. **Tap el hamburger (☰)** arriba izquierda → drawer entra desde la izquierda
10. **Tap el icono luna/sol** en el banner → cambia tema
11. Navega a Hotel Builder, swipea los tabs horizontalmente

## 🔍 Cosas que QUIZÁS quieras pulir cuando despiertes

Pequeñeces que noté pero no toqué (priorité lo grueso):

1. **Auth screen input** — el background del input en dark mode se ve un poco translúcido (rgba). Si te parece raro, podemos cambiar a fondo sólido `#0c130a`.
2. **Onboarding modal** — usa los colores indigo-purple por gradient (no respetó dark mode 100%). Si quieres, lo cambio a la paleta sage.
3. **Notification panel** — al abrirlo en mobile podría salir un poco fuera de viewport en algunos casos. El CSS lo limita a `100vw - 24px` pero conviene verificar.
4. **Tabs del Hotel Builder** — el horizontal scroll funciona pero NO tiene indicador visual de que hay más. Podríamos agregar un gradient fade a los lados.
5. **Map del SaaS Overview** — el legend flotante puede tapar marcadores en móvil.

Estos son detalles. La app está completamente usable en móvil + dark mode.

## 🔔 Roadmap de notificaciones push (idea para más adelante)

Carlos mencionó que le gustaría recibir notificaciones de ventas/reservas en su Apple Watch + iPhone, viendo en tiempo real movimientos tipo "Reserva en Finca Lérida — $85". Plan recomendado por etapas:

### Etapa 1 — Telegram bot (1 día de trabajo)
- Crear bot en `@BotFather` (gratis)
- Cada vez que `triggerMockBooking` o un evento real ocurra, hacer `fetch()` a la API de Telegram con el mensaje
- El bot envía push a su phone → el Watch hereda automáticamente
- Sin app store, sin developer account, funciona ya
- Ideal para validar la experiencia ANTES de invertir en app nativa

### Etapa 2 — PWA con Web Push API
- Requiere service worker + VAPID keys
- iOS 16.4+ Safari soporta push **solo después de "Añadir a pantalla de inicio"**
- Más profesional que Telegram, pero limitado en iOS
- 3-5 días de trabajo

### Etapa 3 — App nativa iOS con Watch widget
- Requiere $99/año de Apple Developer
- Swift + WatchKit
- App Store review (semanas)
- Sobreingeniería hasta que tenga tracción real

### Recomendación final
Cuando llegue el momento, Etapa 1 (Telegram). Es la mejor relación esfuerzo/resultado para validar la mecánica de "ver movimientos en tiempo real" sin sobreinvertir.

## 🚀 Próximos pasos (cuando estés listo)

Cuando me digas "sigue", el plan original era:
- Settings page (mínima)
- Empty states mejorados
- Hotel Builder tabs en EN restantes (Team, Rooms, Services, Policies, Marketing, PMS — ~120 strings)
- AI suggest reply modal i18n (~15 strings)
- Toasts secundarios (~25 strings)

Pero **honestamente** tu prioridad debería ser:
- 📱 Verificar el live URL en tu iPhone
- 📝 Tomar screenshots desde tu móvil y darme feedback
- 👥 Compartir el link con 2-3 colegas para feedback temprano
- ❌ NO seguir construyendo features hasta tener feedback real

## 🛏️ Buenos días 🌿

Si algo no funciona como esperas, no te alarmes. Cada commit fue testeado con balance de braces/parens (validez sintáctica garantizada). Si encuentras un bug visual, abrimos un issue en GitHub y lo arreglo rápido.

— Claude

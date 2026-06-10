# Plan Ok — Qué falta para que funcione del todo

> Investigación y plan de producción. Actualizado: 2026-06-09.
> App: https://woy-lara.github.io/guestguide-pro/ (single-file, GitHub Pages)

## Dónde estamos hoy

Lo que YA funciona (frontend completo):

- Dashboards por rol (admin, hotelero, proveedor) con módulos Restaurante y Spa
- Login simulado: admin con clave maestra, clientes por email autorizado o enlace personalizado (`?c=hacienda-los-molinos`)
- Visores públicos para huéspedes (`?menu=`, `?spa=`) con QR
- Datos demo + cliente piloto real (Hacienda Los Molinos, menú de 61 platos)

**La limitación de fondo**: no hay backend. Todo vive en el navegador de cada
persona (localStorage). Si creas un cliente en tu laptop, ese cliente NO existe
en el celular del hotelero — cada dispositivo carga los datos semilla del
archivo. El login es disuasivo, no seguro (cualquiera con conocimientos
técnicos lo salta desde la consola). **El salto a backend es lo que convierte
esto en un producto real.**

## Lo que necesita un app para funcionar de verdad

| # | Pieza | Para qué | Opción recomendada | Costo inicial |
|---|-------|----------|--------------------|---------------|
| 1 | **Backend + base de datos** | Que los datos vivan en un solo lugar y todos vean lo mismo | **Supabase** (Postgres + Auth + API) | $0 (free tier) |
| 2 | **Autenticación real** | Magic links reales por email, sesiones seguras server-side | Supabase Auth (incluido) | $0 |
| 3 | **Seguridad** | Que cada hotel SOLO vea sus datos | RLS de Postgres + checklist (abajo) | $0 |
| 4 | **Dominio propio** | planok.app o similar — profesionalismo + emails | Namecheap/Porkbun | ~$12–30/año |
| 5 | **Envío de emails** | Magic links, reporte diario 7am | Resend (free: 100/día) | $0 |
| 6 | **WhatsApp real** | Órdenes y reservas al staff (hoy simulado) | Twilio WhatsApp API o Meta Business | ~$0.005–0.05/msg |
| 7 | **Legal** | Términos, privacidad (PII de huéspedes) | Plantillas + revisión local | bajo |
| 8 | **Pagos** (cuando cobres) | Suscripciones de hoteles | Stripe (o PayPal/local Panamá) | % por transacción |
| 9 | **Monitoreo** | Enterarte de errores antes que el cliente | Sentry free tier | $0 |
| 10 | **Backups** | No perder datos de clientes | Incluido en Supabase + export semanal | $0 |

### Supabase free tier (verificado 2026)

500 MB de base de datos, 50,000 usuarios activos/mes en Auth, 1 GB de archivos,
500k invocaciones de edge functions — de sobra para arrancar con 5–20 hoteles.

⚠️ **Trampa del free tier**: el proyecto se PAUSA tras 7 días sin actividad.
Para producción 24/7 con clientes reales, el plan **Pro ($25/mes)** es el
costo real de operar. Estrategia: desarrollar gratis, pasar a Pro cuando el
primer hotel pague.

**Presupuesto realista de arranque: ~$12 (dominio) hoy → ~$25–40/mes cuando
haya clientes pagando.**

## Checklist de seguridad (OBLIGATORIO antes de escribir lógica de negocio)

Este es el entregable acordado: se cumple ANTES de la primera línea de backend.

- [ ] **Keys solo en servidor**: ninguna API key en el HTML/JS público. Las
      llamadas sensibles pasan por edge functions con env vars.
- [ ] **RLS activado en TODAS las tablas** desde el día 1: cada hotel ve solo
      sus filas (`hotel_id = auth.uid()` o tabla de membresías).
- [ ] **Roles en el servidor**: admin/hotelero/proveedor se verifican en
      Postgres (claims/tabla de roles), nunca confiando en el cliente.
- [ ] **Anon key pública ≠ permisos**: la anon key de Supabase puede ser
      pública SOLO si RLS está bien configurado. Probarlo intentando leer
      datos de otro hotel desde la consola.
- [ ] **PII de huéspedes minimizada**: guardar solo lo necesario (nombre,
      habitación); nada de pasaportes/tarjetas. Definir retención (borrar al
      checkout + 30 días).
- [ ] **HTTPS en todo** (GitHub Pages y Supabase ya lo dan).
- [ ] **Validación server-side**: precios y totales se calculan en el
      servidor, nunca se aceptan del cliente.
- [ ] **Rate limiting** en endpoints públicos (anti-spam de órdenes).
- [ ] **Backups automáticos** verificados + un restore de prueba.
- [ ] **Logs de auditoría** en el servidor (ya existe el patrón en frontend).

## Roadmap por fases

### Fase 0 — HOY ✅
Rebrand Plan Ok + login admin con clave + flujo cliente + enlaces
personalizados + fallback de kiosk. *(hecho en commit `6a16d3c`)*

### Fase 1 — Fundación backend (1–2 sesiones)
1. Crear cuenta y proyecto en Supabase (lo haces tú, 10 min, gratis)
2. Diseñar el esquema: `hotels`, `users`, `menu_items`, `spa_services`,
   `orders`, `audit_log` (traducción directa del `DATABASE` actual)
3. Aplicar el checklist de seguridad (RLS + roles)
4. Script de migración: seed actual → Postgres

### Fase 2 — Auth real (1 sesión)
Magic links reales por email (Supabase Auth + Resend). El login simulado se
reemplaza; los enlaces `?c=slug` pasan a ser invitaciones reales.

### Fase 3 — Datos vivos (1–2 sesiones)
El frontend lee/escribe de Supabase en vez de localStorage. Aquí el app
"funciona del todo": lo que editas en tu laptop lo ve el hotelero en su celular.

### Fase 4 — Canales reales
Dominio + emails transaccionales + reporte diario 7am (cron) + WhatsApp API.

### Fase 5 — Crecimiento
Webhooks del PMS (Cloudbeds/Mews — preguntar a Hacienda qué usan), pagos con
Stripe, módulo Transporte.

## Lo que solo TÚ puedes hacer (cuando quieras arrancar Fase 1)

1. Crear cuenta en [supabase.com](https://supabase.com) (gratis, con tu email o GitHub)
2. Decidir el dominio (¿planok.app? ¿planok.com.pa?) — yo verifico disponibilidad
3. Preguntar a Hacienda Los Molinos **qué PMS usan** (para la Fase 5)
4. Cambiar la clave maestra de admin (te explico cómo en el chat)

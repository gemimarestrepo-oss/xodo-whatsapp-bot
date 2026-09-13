# XODÓ Chatbot de WhatsApp — guía para publicarlo

Este proyecto es el backend que recibe mensajes en tu WhatsApp real, responde
preguntas automáticas y toma pedidos. Tú solo entras al panel (`/panel`) para
confirmar o rechazar cada pedido — el cliente recibe el aviso solo.

## Paso 1 — Sube este código a GitHub

1. Crea una cuenta en [github.com](https://github.com) si no tienes.
2. Crea un repositorio nuevo, ej. `xodo-whatsapp-bot` (puede ser privado).
3. Sube estos archivos ahí (arrastrándolos desde la web de GitHub es suficiente,
   no necesitas usar comandos).

**Importante:** no subas el archivo `.env` (con tu token real) a GitHub — el
`.gitignore` ya lo excluye para protegerte.

## Paso 2 — Despliega en Render (gratis)

1. Crea una cuenta en [render.com](https://render.com) (puedes entrar con tu
   cuenta de GitHub).
2. Clic en **New +** → **Web Service**.
3. Conecta tu repositorio `xodo-whatsapp-bot`.
4. Configuración:
   - **Build Command:** `npm install`
   - **Start Command:** `npm start`
   - **Plan:** Free
5. En la sección **Environment**, agrega estas variables (con tus valores
   reales, no los de ejemplo):
   - `VERIFY_TOKEN` → inventa una palabra, ej. `xodo2026secreto`
   - `WHATSAPP_TOKEN` → tu token de acceso de Meta
   - `PHONE_NUMBER_ID` → `1219128987960030`
   - `ADMIN_KEY` → inventa otra clave para entrar a tu panel, ej. `majoxodo123`
6. Clic en **Create Web Service**. Cuando termine, Render te da una URL como
   `https://xodo-whatsapp-bot.onrender.com`.

## Paso 3 — Conecta el webhook en Meta

1. Ve a tu app en developers.facebook.com → **Casos de uso** → WhatsApp →
   **Configuración** → **Webhooks**.
2. **Callback URL:** `https://xodo-whatsapp-bot.onrender.com/webhook`
3. **Verify token:** el mismo que pusiste en `VERIFY_TOKEN` en Render.
4. Clic en **Verificar y guardar**.
5. En la lista de campos de webhook, suscríbete a **messages**.

## Paso 4 — Pruébalo

Escríbele por WhatsApp a tu número de prueba (el que te dio Meta) y deberías
recibir respuesta del bot. Prueba con "hola", "precios", "pedido".

## Paso 5 — Revisar pedidos

Entra a `https://xodo-whatsapp-bot.onrender.com/panel?clave=TU_ADMIN_KEY`
(la clave que pusiste en `ADMIN_KEY`). Ahí ves cada pedido con botones
**Confirmar** / **Rechazar** — al usarlos, el cliente recibe automáticamente
un WhatsApp con el resultado.

## Antes de pasar a tu número real (322 693 2728)

Con el número de prueba solo tú puedes escribirle al bot. Para que cualquier
cliente pueda hacerlo desde tu número real, Meta te va a pedir:

- **Verificar tu negocio** en Meta Business Suite (documentos legales: NIT,
  cámara de comercio, etc.) — toma de 1 a 7 días hábiles.
- Generar un **token de acceso permanente** (el que tienes ahora dura 24
  horas) — esto se hace desde Business Manager, con un **usuario del
  sistema**; te guío en ese paso cuando llegues ahí.

## Nota sobre el plan gratis de Render

El plan free "duerme" el servidor si no recibe tráfico por un rato, y tarda
unos segundos en despertar con el primer mensaje del día. Para un volumen
pequeño de pedidos esto no es un problema; si tu negocio crece y quieres que
responda siempre al instante, el plan pago de Render (desde ~7 USD/mes)
elimina esa espera.

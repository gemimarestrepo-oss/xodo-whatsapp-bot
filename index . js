require('dotenv').config();
const express = require('express');
const fs = require('fs');
const path = require('path');

const app = express();
app.use(express.json());

// ---------- Configuracion (variables de entorno) ----------
const {
  VERIFY_TOKEN,       // palabra clave que tu inventas, la usas en Meta y aqui
  WHATSAPP_TOKEN,     // el "Identificador de acceso" de Meta
  PHONE_NUMBER_ID,    // el "Phone Number ID" de Meta
  ADMIN_KEY,          // clave que tu inventas para entrar al panel /panel
  PORT = 3000,
} = process.env;

const GRAPH_URL = `https://graph.facebook.com/v20.0/${PHONE_NUMBER_ID}/messages`;
const PRICE = 4500;
const FLAVORS = ['Limón', 'Natural', 'Limón-Pimienta'];
const ORDERS_FILE = path.join(__dirname, 'orders.json');

// ---------- Almacenamiento simple ----------
// Estado de conversacion por numero de telefono (en memoria: se reinicia si el server se reinicia)
const conversations = new Map();

function loadOrders() {
  try {
    return JSON.parse(fs.readFileSync(ORDERS_FILE, 'utf8'));
  } catch {
    return [];
  }
}
function saveOrders(list) {
  fs.writeFileSync(ORDERS_FILE, JSON.stringify(list, null, 2));
}

function fmtCOP(n) {
  return '$' + n.toLocaleString('es-CO');
}

// ---------- Enviar mensajes de WhatsApp ----------
async function sendText(to, body) {
  await fetch(GRAPH_URL, {
    method: 'POST',
    headers: {
      Authorization: `Bearer ${WHATSAPP_TOKEN}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      messaging_product: 'whatsapp',
      to,
      type: 'text',
      text: { body },
    }),
  }).then(async (r) => {
    if (!r.ok) console.error('Error enviando WhatsApp:', await r.text());
  });
}

// ---------- Respuestas automaticas (FAQ) ----------
const FAQS = [
  { kws: ['sabor', 'sabores', 'portafolio', 'productos', 'papas'],
    a: 'Nuestro portafolio tiene 3 sabores artesanales:\n• Papa Sabor Limón\n• Papa Sabor Natural\n• Papa Sabor Limón-Pimienta\n\nPresentación: bolsa de 80 g, aprox. 3 porciones.' },
  { kws: ['precio', 'cuanto cuesta', 'vale', 'valor', 'cuesta'],
    a: `Precio al por mayor: ${fmtCOP(PRICE)} por unidad (bolsa de 80 g). Despacho desde Medellín, Colombia.` },
  { kws: ['ingrediente', 'alergen', 'alergia', 'gluten'],
    a: 'XODÓ Chips es apto para consumo humano y libre de alérgenos, elaborado con papas criollas y fritura artesanal.' },
  { kws: ['vence', 'vencimiento', 'dura', 'caduca', 'vida util', 'vida útil'],
    a: 'Vida útil de 4 meses en condiciones adecuadas de almacenamiento.' },
  { kws: ['invima', 'registro', 'sanitario', 'legal', 'norma'],
    a: 'Contamos con notificación sanitaria vigente NSA-0014904-2023 y cumplimos con las normas de rotulado del empaque.' },
  { kws: ['empaque', 'bolsa', 'presentacion', 'presentación', 'conserva'],
    a: 'Empaque premium con sellado hermético de alta barrera: conserva frescura y crocancia por más tiempo que el empaque tradicional.' },
  { kws: ['diferencia', 'competencia', 'por que', 'ventaja', 'mejor'],
    a: 'Frente al mercado tradicional: innovación constante, sellado hermético de alta barrera, frescura y vida útil extendida.' },
  { kws: ['distribuidor', 'mayorista', 'revender', 'negocio', 'margen', 'invertir'],
    a: 'Producto terminado, empacado y listo para distribución inmediata: baja inversión, entregas en corto tiempo, marca y soporte completo, y buen margen de rentabilidad.' },
  { kws: ['asesor', 'humano', 'persona', 'hablar con alguien', 'contacto'],
    a: 'Con gusto te contactamos directamente: 322 693 2728 · xodoproductos@gmail.com · @xodo_official.' },
  { kws: ['gracias', 'chao', 'adios', 'adiós', 'hasta luego'],
    a: '¡Con el alma, gracias a ti! Cualquier otra pregunta, aquí estoy. 🌿' },
];

function matchFaq(text) {
  const t = text.toLowerCase();
  for (const f of FAQS) {
    if (f.kws.some((k) => t.includes(k))) return f.a;
  }
  return null;
}

const MENU_TEXT =
  '¡Hola! Soy el asistente de XODÓ Chips 🌿\n\n' +
  'Puedes preguntarme por sabores, precios, empaque, registro Invima, etc.\n' +
  'O escribe *"pedido"* para hacer un pedido.';

// ---------- Flujo de pedido (maquina de estados por numero) ----------
function getState(from) {
  if (!conversations.has(from)) {
    conversations.set(from, { step: 'MENU', draft: { items: [] } });
  }
  return conversations.get(from);
}

async function handleIncoming(from, text) {
  const state = getState(from);
  const t = text.trim();
  const tl = t.toLowerCase();

  switch (state.step) {
    case 'FLAVOR': {
      const flavor =
        FLAVORS.find((f) => tl.includes(f.toLowerCase().split('-')[0])) || t;
      state.draft.currentFlavor = flavor;
      state.step = 'QTY';
      await sendText(from, `¿Cuántas bolsas (80 g) de ${flavor} deseas?`);
      return;
    }
    case 'QTY': {
      const qty = parseInt(t.replace(/\D/g, ''), 10);
      if (!qty || qty <= 0) {
        await sendText(from, 'Escribe solo el número de bolsas, por ejemplo: 20');
        return;
      }
      state.draft.items.push({ flavor: state.draft.currentFlavor, qty });
      state.step = 'MORE';
      await sendText(from, '¿Deseas agregar otro sabor? (Sí / No)');
      return;
    }
    case 'MORE': {
      if (tl.startsWith('s')) {
        state.step = 'FLAVOR';
        await sendText(from, `¿Qué otro sabor deseas? (${FLAVORS.join(', ')})`);
      } else {
        state.step = 'NAME';
        await sendText(from, '¿A nombre de quién va el pedido?');
      }
      return;
    }
    case 'NAME':
      state.draft.name = t;
      state.step = 'CITY';
      await sendText(from, '¿Ciudad y dirección de entrega?');
      return;
    case 'CITY':
      state.draft.city = t;
      state.step = 'PHONE';
      await sendText(from, '¿Número de contacto para confirmar la entrega?');
      return;
    case 'PHONE':
      state.draft.phone = t;
      state.step = 'PAYMENT';
      await sendText(from, '¿Cómo prefieres pagar? (Transferencia / Efectivo contraentrega / Otro)');
      return;
    case 'PAYMENT': {
      state.draft.payment = t;
      const total = state.draft.items.reduce((s, i) => s + i.qty * PRICE, 0);
      state.draft.total = total;
      const itemsTxt = state.draft.items
        .map((i) => `${i.qty} × ${i.flavor} = ${fmtCOP(i.qty * PRICE)}`)
        .join('\n');
      state.step = 'CONFIRM';
      await sendText(
        from,
        `Resumen de tu pedido:\n${itemsTxt}\n\nTotal: ${fmtCOP(total)}\n${state.draft.name} · ${state.draft.city}\n${state.draft.phone} · ${state.draft.payment}\n\n¿Confirmas? (Confirmar / Cancelar)`
      );
      return;
    }
    case 'CONFIRM': {
      if (tl.startsWith('confirmar')) {
        const order = {
          ...state.draft,
          id: 'pedido_' + Date.now(),
          from,
          status: 'pendiente',
          createdAt: new Date().toLocaleString('es-CO'),
        };
        const list = loadOrders();
        list.unshift(order);
        saveOrders(list);
        await sendText(
          from,
          '¡Pedido enviado! XODÓ lo revisará y te confirma muy pronto por este mismo WhatsApp. Gracias por tu compra. 🌿'
        );
      } else {
        await sendText(from, 'Pedido cancelado. Cuando quieras, escribe "pedido" para empezar de nuevo.');
      }
      conversations.set(from, { step: 'MENU', draft: { items: [] } });
      return;
    }
    default: {
      if (/pedido|comprar|ordenar|quiero comprar/.test(tl)) {
        conversations.set(from, { step: 'FLAVOR', draft: { items: [] } });
        await sendText(from, `Perfecto, armemos tu pedido. ¿Qué sabor deseas? (${FLAVORS.join(', ')})`);
        return;
      }
      const faq = matchFaq(t);
      await sendText(from, faq || MENU_TEXT);
      return;
    }
  }
}

// ---------- Webhook de WhatsApp ----------

// Verificacion (Meta llama esto UNA vez al configurar el webhook)
app.get('/webhook', (req, res) => {
  const mode = req.query['hub.mode'];
  const token = req.query['hub.verify_token'];
  const challenge = req.query['hub.challenge'];
  if (mode === 'subscribe' && token === VERIFY_TOKEN) {
    return res.status(200).send(challenge);
  }
  return res.sendStatus(403);
});

// Mensajes entrantes
app.post('/webhook', async (req, res) => {
  res.sendStatus(200); // responder rapido a Meta, procesar despues
  try {
    const entry = req.body.entry?.[0];
    const change = entry?.changes?.[0];
    const message = change?.value?.messages?.[0];
    if (!message) return; // puede ser un evento de "status" (entregado/leido), lo ignoramos
    const from = message.from;
    const text = message.text?.body || '';
    if (text) await handleIncoming(from, text);
  } catch (e) {
    console.error('Error procesando webhook:', e);
  }
});

// ---------- Panel de revision ("Panel Majo") ----------
function checkAdmin(req, res) {
  if (req.query.clave !== ADMIN_KEY) {
    res.status(401).send('Clave incorrecta. Agrega ?clave=TU_CLAVE a la URL.');
    return false;
  }
  return true;
}

app.get('/panel', (req, res) => {
  if (!checkAdmin(req, res)) return;
  const orders = loadOrders();
  const rows = orders
    .map((o) => {
      const itemsTxt = o.items.map((i) => `${i.qty} × ${i.flavor}`).join(', ');
      const actions =
        o.status === 'pendiente'
          ? `<form method="POST" action="/panel/orders/${o.id}/confirm?clave=${req.query.clave}" style="display:inline">
               <button style="background:#3C6E4F;color:#fff;border:none;border-radius:8px;padding:6px 12px;margin-right:6px;">Confirmar</button>
             </form>
             <form method="POST" action="/panel/orders/${o.id}/reject?clave=${req.query.clave}" style="display:inline">
               <button style="background:#fff;color:#B23A2E;border:1px solid #B23A2E;border-radius:8px;padding:6px 12px;">Rechazar</button>
             </form>`
          : `<b>${o.status}</b>`;
      return `<tr style="border-bottom:1px solid #eee;">
        <td style="padding:8px;">${o.name}</td>
        <td style="padding:8px;">${itemsTxt}</td>
        <td style="padding:8px;">${fmtCOP(o.total)}</td>
        <td style="padding:8px;">${o.city}<br>${o.phone}</td>
        <td style="padding:8px;">${o.createdAt}</td>
        <td style="padding:8px;">${actions}</td>
      </tr>`;
    })
    .join('');
  res.send(`
    <html><head><meta name="viewport" content="width=device-width, initial-scale=1">
    <style>body{font-family:sans-serif;background:#F4EFE4;padding:20px;} table{width:100%;border-collapse:collapse;background:#fff;border-radius:10px;overflow:hidden;} th{text-align:left;padding:8px;background:#221F1C;color:#fff;}</style>
    </head><body>
    <h2>Pedidos XODÓ</h2>
    <table><tr><th>Nombre</th><th>Pedido</th><th>Total</th><th>Entrega</th><th>Fecha</th><th>Acción</th></tr>
    ${rows || '<tr><td style="padding:12px;">Todavía no hay pedidos.</td></tr>'}
    </table>
    </body></html>
  `);
});

app.post('/panel/orders/:id/confirm', async (req, res) => {
  if (!checkAdmin(req, res)) return;
  const list = loadOrders();
  const order = list.find((o) => o.id === req.params.id);
  if (order) {
    order.status = 'confirmado';
    saveOrders(list);
    await sendText(
      order.from,
      `¡Tu pedido fue confirmado! ✅ Total: ${fmtCOP(order.total)}. Pronto coordinamos la entrega en ${order.city}. Gracias por tu compra en XODÓ. 🌿`
    );
  }
  res.redirect(`/panel?clave=${req.query.clave}`);
});

app.post('/panel/orders/:id/reject', async (req, res) => {
  if (!checkAdmin(req, res)) return;
  const list = loadOrders();
  const order = list.find((o) => o.id === req.params.id);
  if (order) {
    order.status = 'rechazado';
    saveOrders(list);
    await sendText(
      order.from,
      `Hola ${order.name}, tuvimos un inconveniente con tu pedido y no lo pudimos confirmar. Escríbenos al 322 693 2728 para resolverlo. Disculpa las molestias.`
    );
  }
  res.redirect(`/panel?clave=${req.query.clave}`);
});

app.get('/', (req, res) => res.send('XODO WhatsApp bot activo ✅'));

app.listen(PORT, () => console.log(`Servidor corriendo en puerto ${PORT}`));

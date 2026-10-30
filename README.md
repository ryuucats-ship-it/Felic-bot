# Felic-bot
TEST BOT WA BY RAY'Z V

{
  "name": "bot-wa-multifungsi",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "start": "node index.js"
  },
  "engines": {
    "node": ">=18"
  },
  "dependencies": {
    "express": "^5.1.0"
  }
}
https://graph.facebook.com/${VERSION}/${PHONE_ID}/messageschange.value?.messagesentry.changesreq.body.entryb.lengthapp.postprocess.env.PORTreq.rawBody
const express = require("express");
const crypto = require("node:crypto");

const app = express();
app.use(express.json({
  verify: (req, res, buf) => {
    req.rawBody = Buffer.from(buf);
  }
}));

const PORT = process.env.PORT || 3000;
const TOKEN = process.env.WEBHOOK_VERIFY_TOKEN;
const APP_SECRET = process.env.APP_SECRET;
const ACCESS_TOKEN = process.env.WHATSAPP_ACCESS_TOKEN;
const PHONE_ID = process.env.PHONE_NUMBER_ID;
const VERSION = process.env.GRAPH_API_VERSION || "v23.0";

app.get("/", (req, res) => {
  res.send("Bot WhatsApp siap menerima koneksi.");
});

app.get("/webhook", (req, res) => {
  const mode = req.query["hub.mode"];
  const token = req.query["hub.verify_token"];
  const challenge = req.query["hub.challenge"];

  if (mode === "subscribe" && token === TOKEN && TOKEN) {
    return res.status(200).send(challenge);
  }

  return res.sendStatus(403);
});

app.post("/webhook", async (req, res) => {
  const signature = req.get("x-hub-signature-256");

  if (!APP_SECRET || !signature || !req.rawBody) {
    return res.sendStatus(401);
  }

  const expected = "sha256=" +
    crypto.createHmac("sha256", APP_SECRET)
      .update(req.rawBody)
      .digest("hex");

  const a = Buffer.from(signature);
  const b = Buffer.from(expected);

  if (a.length !== b.length ||
      !crypto.timingSafeEqual(a, b)) {
    return res.sendStatus(401);
  }

  // Balas webhook secepatnya.
  res.sendStatus(200);

  const entries = req.body.entry || [];

  for (const entry of entries) {
    for (const change of entry.changes || []) {
      const messages = change.value?.messages || [];

      for (const message of messages) {
        if (message.type !== "text") continue;

        const from = message.from;
        const text = message.text.body.trim();

        try {
          const reply = getReply(text);
          if (reply) await sendMessage(from, reply);
        } catch (error) {
          console.error("Gagal memproses pesan:", error.message);
        }
      }
    }
  }
});

function getReply(input) {
  const text = input.toLowerCase().trim();

  if (["menu", ".menu", "help"].includes(text)) {
    return [
      "🤖 BOT MULTIFUNGSI",
      "",
      "1. menu - Daftar fitur",
      "2. ping - Cek status",
      "3. hitung 12+5 - Kalkulator",
      "4. tebak - Tebak-tebakan",
      "",
      "Kirim salah satu perintah di atas!"
    ].join("\n");
  }

  if (text === "ping" || text === ".ping") {
    return "Pong! 🟢 Bot berhasil merespons.";
  }

  if (text === "tebak" || text === ".tebak") {
    return [
      "🧩 TEBAK-TEBAKAN",
      "Apa yang punya banyak gigi",
      "tetapi tidak bisa menggigit?",
      "",
      "Jawaban: sisir 😭"
    ].join("\n");
  }

  const calc = text.match(
    /^hitung\s+(-?\d+(?:\.\d+)?)\s*([+\-*/])\s*(-?\d+(?:\.\d+)?)$/
  );

  if (calc) {
    const a = Number(calc[1]);
    const op = calc[2];
    const b = Number(calc[3]);
    let result;

    if (op === "+") result = a + b;
    if (op === "-") result = a - b;
    if (op === "*") result = a * b;

    if (op === "/") {
      if (b === 0) return "Tidak bisa membagi dengan nol.";
      result = a / b;
    }

    if (!Number.isFinite(result)) {
      return "Hasil tidak valid.";
    }

    return `🧮 Hasil: ${result}`;
  }

  return 'Perintah tidak dikenal. Ketik "menu" untuk melihat fitur.';
}

async function sendMessage(to, body) {
  const response = await fetch(
    `https://graph.facebook.com/${VERSION}/${PHONE_ID}/messages`,
    {
      method: "POST",
      headers: {
        "Authorization": `Bearer ${ACCESS_TOKEN}`,
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        messaging_product: "whatsapp",
        recipient_type: "individual",
        to,
        type: "text",
        text: { body }
      })
    }
  );

  if (!response.ok) {
    console.error("WhatsApp API error:", await response.text());
  }
}

app.listen(PORT, () => {
  console.log(`Bot berjalan pada port ${PORT}`);
});

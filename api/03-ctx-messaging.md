# ctx.send

API de envio. Todo método retorna um `MessageHandle` (thenable) com ações encadeáveis.

```js
// chat atual (runtime only)
await ctx.send.text("Olá!");
await ctx.send.text("https://exemplo.com", { linkPreview: false });
await ctx.send.text("Olá @user", { mentions: ["5511999999999@s.whatsapp.net"] });

await ctx.send.image("/tmp/foto.jpg", "legenda");
await ctx.send.image("/tmp/foto.jpg", "secreta", { viewOnce: true });

await ctx.send.video("/tmp/video.mp4", "legenda");
await ctx.send.audio("/tmp/audio.ogg");                      // como voz (padrão)
await ctx.send.audio("/tmp/audio.ogg", { asVoice: false });   // como áudio normal
await ctx.send.sticker("/tmp/sticker.webp");
await ctx.send.sticker(bufferGeradoEmMemoria);
await ctx.send.file("/tmp/relatorio.pdf", "relatorio-2025.pdf");
await ctx.send.poll("Qual sabor?", ["Morango", "Chocolate"], { allowMultipleAnswers: true });

// qualquer chat, por ID (setup + runtime)
await ctx.send.to(chatId).text("notificação");
```

### Opções por método

| Método | Opções |
|---|---|
| `text` | `{ linkPreview?, mentions?: string[] }` |
| `image` / `video` | `{ viewOnce? }` |
| `audio` | `{ asVoice? (padrão true), viewOnce? }` |
| `poll` | `{ allowMultipleAnswers? }` |
| `sticker` / `file` | sem opções extra |

### Ações pós-envio (chaining)

```js
await ctx.send.text("Pronto!").react("✅");

const handle = await ctx.send.image("/tmp/x.jpg"); // handle = Message enviada
await handle.delete();
```

`delete(forEveryone? = true)` e `react(emoji)` retornam `Promise`.

> `pin(duration?)` também existe na interface, mas atualmente **não tem efeito** — o Baileys (a
> biblioteca por trás da conexão com o WhatsApp) ainda não suporta fixar mensagens, então chamar
> `.pin()` só registra um aviso no log e não faz nada. Deixamos o método aí pra quando isso mudar.

> **send vs reply:** `ctx.send.*` manda sem citar nada. `ctx.msg.reply.*` cita a mensagem que
> disparou o handler — prefira `reply` em grupos.

---

# ctx.msg

Contexto da mensagem que disparou o handler (runtime only).

```js
ctx.msg.body;         // string — texto completo
ctx.msg.type;         // "chat" | "image" | "video" | "audio" | ...
ctx.msg.fromMe;       // true se enviada pelo próprio bot
ctx.msg.sender;       // ID de quem enviou
ctx.msg.senderName;   // nome de exibição
ctx.msg.command;      // primeira palavra, sem prefixo, minúscula
ctx.msg.args;         // string[] — palavras após o comando
ctx.msg.hasMedia;     // boolean
ctx.msg.isGif;        // boolean
ctx.msg.hasReply;     // true se é resposta a outra msg
ctx.msg.hasPrefix;    // true se começa com CMD_PREFIX
```

### Detectando comandos

```js
// Mensagem: "!ping"  (CMD_PREFIX = "!")
if (ctx.msg.is("ping")) {
  await ctx.send.text("pong!");
}
```

### Lendo argumentos

```js
// Mensagem: "!video https://youtube.com/watch?v=..."
const url = ctx.msg.args[0];
if (!url) {
  await ctx.msg.reply.text("Informe uma URL.");
  return;
}
```

### Respondendo com quote

`ctx.msg.reply` é um sender completo — mesmos métodos de `ctx.send`, mas citando a mensagem
original:

```js
await ctx.msg.reply.text("Aqui está sua resposta!");
await ctx.msg.reply.image("/tmp/foto.jpg", "Sua imagem.");
```

### Baixando mídia

```js
if (ctx.msg.hasMedia) {
  const media = await ctx.msg.downloadMedia(); // { mimetype, data(base64) } | null
  if (!media) return void await ctx.msg.reply.text("Não consegui baixar.");
  const buf = Buffer.from(media.data, "base64");
}
```

### Mensagem citada

```js
if (ctx.msg.hasReply) {
  const quoted = await ctx.msg.getReply(); // { key, message, pushName } | null — não é um ctx.msg completo
  console.log(quoted?.pushName);
}
```

`getReply()` retorna a mensagem original em formato bruto (o mesmo formato interno do Baileys),
não um `ctx.msg` completo — então **não tem** `.hasMedia`/`.downloadMedia()`/`.reply` prontos.
Hoje não existe um atalho público pra baixar mídia de uma mensagem citada; isso é uma limitação
conhecida. Se seu plugin precisa disso, o campo `quoted.message` tem a estrutura crua do Baileys
(ex: `quoted.message.imageMessage`), mas mexer nisso diretamente significa depender de detalhes
internos que podem mudar.

### Contato do remetente

`ctx.msg.getContact()` resolve `@lid` automaticamente — prefira isso a
`ctx.contacts.get(ctx.msg.sender)` dentro de um handler. Mesma shape de
[objeto de contato](/docs/05-ctx-contacts/#objeto-de-contato-normalizado).

```js
const contact = await ctx.msg.getContact();
await ctx.msg.reply.text(`oi ${contact.mention.text}`, contact.mention);
```

### Evitando loops

O bot recebe as próprias mensagens também:

```js
export default async function (ctx) {
  if (ctx.msg.fromMe) return;
  // ...
}
```

---
title: ctx.events
description: Assina eventos brutos do WhatsApp (mensagens, chats, contatos, grupos, conexão) fora do ciclo normal de mensagem.
sidebar:
  order: 8.1
---

Assina eventos brutos do WhatsApp diretamente no `WaContract` — útil pra reagir a coisas que não
chegam como uma mensagem comum (atualização de grupo, contato, status de conexão, etc.). Setup
only: assine em `plugin.setup(ctx)`, não em `default(ctx)`.

```js
export async function setup(ctx) {
  const unsubscribe = ctx.events.on("group-participants.update", (payload) => {
    ctx.log.info("participantes mudaram", payload);
  });

  // chame unsubscribe() se precisar parar de escutar antes do plugin recarregar
}
```

`ctx.events.once(evento)` devolve uma Promise que resolve no próximo disparo do evento (e some a
inscrição sozinha depois):

```js
const payload = await ctx.events.once("connection.update");
```

Só é possível assinar um conjunto fixo de eventos — passar qualquer outro nome lança um erro
explícito citando essa lista. Toda inscrição feita via `ctx.events.on(...)` é automaticamente
removida quando o plugin é recarregado ou desativado — não é preciso limpar manualmente no
encerramento do plugin.

## Eventos disponíveis

### `messages.upsert`
Mensagem nova (ou reenviada em batch de histórico) chegando.

```ts
{ messages: BotMessage[], type: "notify" | "append" }
```
`type: "notify"` é mensagem em tempo real; `"append"` é sincronização de histórico. Cada item de
`messages` é o mesmo shape usado em `ctx.msg` durante o fluxo normal de comando.

### `messages.update`
Edição, reação, ou mudança de status (ex.: recibo de leitura) numa mensagem existente.

```ts
{ updates: Array<{ key: BotQuotedRef, update: Record<string, unknown> }> }
```
`key` identifica a mensagem afetada (`key.id`, `key.remoteJid`, `key.fromMe`, `key.participant` —
os mesmos campos que `ctx.msg.quoted` usa pra referenciar uma mensagem). `update` é um objeto solto
com o que mudou — o shape varia conforme o tipo de update (edição de texto, reação, status), então
trate como dado bruto e verifique as chaves antes de usar.

### `messages.delete`
Mensagem apagada (por qualquer participante, ou o chat inteiro sendo limpo).

```ts
{ keys: BotQuotedRef[], all?: { jid: string } | null }
```
Se `all` vier preenchido, o evento é "todo o histórico de `all.jid` foi apagado" (limpeza de
conversa) e `keys` pode vir vazio — trate esse caso separado de um delete pontual.

### `messaging-history.set`
Disparado uma vez, na sincronização inicial de histórico após conectar.

```ts
{ chats: BotChatSummary[], contacts: BotContactSummary[], messages: BotMessage[] }
```
`BotChatSummary` é `{ id, name? }`, `BotContactSummary` é `{ id, name? }` (versões enxutas — sem
estado ao vivo de admin/negócio, que só vêm por `ctx.chat`/`ctx.contacts`).

### `chats.upsert` / `chats.update` / `chats.delete`
Conversa nova, conversa alterada (nome, etc.) e conversa removida da lista.

```ts
// chats.upsert
{ chats: BotChatSummary[] }
// chats.update
{ updates: Array<{ id: string, name?: string }> }
// chats.delete
{ ids: string[] }
```

### `contacts.upsert` / `contacts.update`
Contato novo salvo e contato existente alterado (ex.: nome mudou).

```ts
// contacts.upsert
{ contacts: BotContactSummary[] }
// contacts.update
{ updates: BotContactSummary[] }
```

### `group-participants.update`
Alguém entrou, saiu, foi promovido/rebaixado a admin, ou teve metadado de membro alterado num
grupo.

```ts
{
  id: string,               // JID do grupo
  author: string,           // LID de quem acionou o evento
  participants: string[],   // LIDs afetados, normalizados
  action: "add" | "remove" | "promote" | "demote" | "modify",
}
```
`participants` e `author` vêm sempre em formato **LID** (`@lid`), mesmo que o WhatsApp tenha
entregue o número de telefone (`phoneNumber`) junto por baixo dos panos — o driver usa esse dado
só pra alimentar o cache interno LID↔PN, mas não repassa no payload do evento. Se seu plugin
precisa do número de quem entrou/saiu, consulte `ctx.contacts` depois de receber o evento — o
número pode vir `null` se o WhatsApp nunca expôs essa correspondência pro bot.

`"modify"` cobre ajustes de metadado de membro (ex.: cartão de contato) sem entrada/saída real —
trate separado de `add`/`remove` se seu plugin conta membros.

### `groups.upsert` / `groups.update`
Grupo novo (bot foi adicionado) e metadado de grupo alterado (nome, descrição, config).

```ts
// groups.upsert
{ groups: Array<{ id: string, subject?: string }> }
// groups.update
{ updates: Array<{ id: string }> }
```
`groups.update` só confirma **quais** grupos mudaram (`id`) — pra saber o que mudou, busque o
metadado atual (`ctx.chat` no grupo em questão) depois de receber o evento.

### `group.join-request`
Pedido de entrada num grupo com aprovação de admin ativada.

```ts
{
  id: string,           // JID do grupo
  author: string,       // quem processou (aprovou/rejeitou), quando aplicável
  participant: string,  // quem pediu pra entrar
  action: "created" | "revoked" | "rejected",
  method: "invite_link" | "linked_group_join" | "non_admin_add" | "unknown",
}
```

### `blocklist.set` / `blocklist.update`
Lista de bloqueio carregada inteira (no connect) e alteração incremental nela.

```ts
// blocklist.set
{ blocklist: string[] }   // lista completa de JIDs bloqueados
// blocklist.update
{ blocklist: string[], type: "add" | "remove" }
```

### `connection.update`
Mudança no estado da conexão com o WhatsApp.

```ts
{
  connection: "open" | "close" | "connecting",
  lastDisconnect?: { statusCode?: number },
}
```
Use `connection === "close"` + `lastDisconnect?.statusCode` pra diagnosticar desconexões (ex.:
detectar logout vs queda de rede). Pra só saber quando o bot reconectou, prefira
`await ctx.events.once("connection.update")` filtrando `connection === "open"` dentro do callback.

> Se precisar de um evento fora dessa lista, abra uma issue — adicionar um evento novo é uma
> mudança de contrato (`WaContract`), não só da API de plugin.

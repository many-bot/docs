# ctx.events (setup only)

```js
export async function setup(ctx) {
  const off = ctx.events.on("group-participants.update", async (update) => {
    if (update.action === "add") {
      await ctx.send.to(update.id).text("Bem-vindo! 👋");
    }
  });
  setTimeout(off, 60 * 60 * 1000); // para de ouvir em 1h

  await ctx.events.once("connection.update"); // resolve na próxima mudança de conexão
  ctx.log.success("Conexão atualizada!");

  ctx.events.on("connection.update", (update) => {
    if (update.connection === "close") ctx.log.warn("Bot desconectou.");
  });
}
```

`on(event, handler)` → retorna `off()`. `once(event)` → `Promise` que resolve na próxima
ocorrência, tipada com o payload daquele evento. `cleanup()` → remove todos os listeners do
plugin (chamado automaticamente pelo kernel ao descarregar).

> **`messages.upsert`:** o kernel já consome esse evento internamente pra despachar plugins
> (é a fonte de `ctx.msg`). Ouvir `messages.upsert` via `ctx.events.on` te dá acesso ao evento
> bruto do Baileys, incluindo mensagens de chats fora da lista de permitidos — filtre
> manualmente se precisar disso, e prefira `ctx.msg` no handler normal em vez de reimplementar
> a lógica de despacho aqui.

## Eventos disponíveis

Esses são os eventos nativos do socket Baileys (`sock.ev`), repassados via `ctx.events`:

| Evento | Payload (resumo) | Descrição |
|---|---|---|
| `connection.update` | `{ connection, qr?, lastDisconnect? }` | Conexão mudou (abrindo, `qr` recebido, fechada). |
| `creds.update` | credenciais atualizadas | Sessão de auth mudou — o kernel já persiste isso. |
| `messaging-history.set` | histórico inicial (chats/contatos/mensagens) | Sincronização de histórico ao conectar. |
| `chats.upsert` / `chats.update` / `chats.delete` | `Chat[]` / updates parciais / IDs | Chats criados, atualizados ou removidos. |
| `presence.update` | `{ id, presences }` | Presença (online/digitando) de um chat mudou. |
| `contacts.upsert` / `contacts.update` | `Contact[]` / updates parciais | Contatos criados ou atualizados. |
| `messages.upsert` | `{ messages, type }` | Nova(s) mensagem(ns) — inclui as do próprio bot. ⚠️ veja nota acima. |
| `messages.update` | updates parciais de mensagem | Status/conteúdo de mensagem mudou. |
| `messages.delete` | IDs ou `{ jid, all }` | Mensagem(ns) apagada(s). |
| `messages.media-update` | updates de mídia | Upload/download de mídia concluído ou falhou. |
| `messages.reaction` | `{ key, reaction }[]` | Reação enviada, atualizada ou removida. |
| `message-receipt.update` | `{ key, receipt }[]` | Confirmação de entrega/leitura. |
| `groups.upsert` / `groups.update` | metadados de grupo | Grupo criado ou metadados (nome, descrição, foto) mudaram. |
| `group-participants.update` | `{ id, participants, action }` | Membro entrou, saiu, foi promovido ou removido. `action`: `"add" \| "remove" \| "promote" \| "demote"`. |
| `group.join-request` | `{ id, participant, action }` | Pedido de entrada em grupo com aprovação obrigatória. |
| `blocklist.set` / `blocklist.update` | lista de JIDs bloqueados | Lista de bloqueio carregada ou atualizada. |
| `call` | `{ id, from, status, isVideo }[]` | Chamada recebida, aceita, rejeitada ou encerrada. |
| `labels.edit` / `labels.association` | labels do WhatsApp Business | Labels criadas/editadas ou associadas a um chat. |

> Payloads exatos vêm de `BaileysEventMap` (`@whiskeysockets/baileys`). O pacote `@manybot/types`
> já tipa `ctx.events.on`/`once` sobre esse mapa — o `handler` recebe o payload correto
> automaticamente pra qualquer evento nativo do Baileys, sem precisar importar nada extra no
> seu plugin.

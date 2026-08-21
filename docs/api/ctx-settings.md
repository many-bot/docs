---
title: ctx.settings
description: Configurações por chat (ou globais) que o usuário ajusta pelo próprio WhatsApp, incluindo comunidades.
sidebar:
  order: 15
---

Armazenamento de configurações por chat (ou global), persistido em disco — pense em "preferências
que o usuário ajusta pelo próprio WhatsApp", diferente de [`ctx.storage`](/docs/api/ctx-storage/)
(dados livres do plugin).

```js
// no chat atual
ctx.settings.set("boasVindas", true);
const ativo = ctx.settings.get("boasVindas", false); // com fallback
ctx.settings.getAll();      // todas as chaves deste chat
ctx.settings.delete("boasVindas");
ctx.settings.deleteAll();

// configuração global do bot, não ligada a nenhum chat
ctx.settings.global.set("modoManutencao", true);
ctx.settings.global.get("modoManutencao", false);

// configuração de outro chat específico
ctx.settings.forChat(outroChatId).set("idioma", "en");
```

### Comunidades

Grupos que fazem parte da mesma comunidade do WhatsApp podem compartilhar configurações:

```js
ctx.settings.link(communityId);          // associa o chat atual a uma comunidade
ctx.settings.unlink();                   // remove a associação do chat atual
ctx.settings.getCommunityId();           // string | null
ctx.settings.getCommunityChats();        // string[] — chats associados à mesma comunidade
```

Veja a tabela completa de métodos em [ctx — opções e assinaturas](/docs/api/ctx-options/#ctxsettings).

> No `setup()`, só `ctx.settings.global` está disponível — sem chat atual, os métodos que operam
> no "chat atual" (incluindo `forChat`/`link`/`unlink`) não fazem sentido ali.

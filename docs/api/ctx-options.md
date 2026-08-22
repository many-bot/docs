---
title: "ctx: opções e assinaturas"
description: Tabelas de referência rápida — opções por método de envio, e assinaturas dos métodos utilitários (i18n, scheduler, storage, settings, plugins).
sidebar:
  order: 20
  label: Opções e assinaturas
---

Referência rápida das opções/assinaturas de cada método, reunidas aqui pra não repetir a mesma
tabela em cada página de endpoint. Pra exemplos e explicação de uso, veja a página do endpoint
correspondente.

## ctx.send

Opções por método — veja [ctx.send](/docs/api/ctx-send/) para exemplos.

| Método | Opções |
|---|---|
| `text` | `{ linkPreview?, mentions?: string[] }` |
| `image` / `video` / `gif` | `{ viewOnce?, mentions?: string[] }` |
| `audio` | `{ asVoice? (padrão true), viewOnce? }` |
| `poll` | `{ allowMultipleAnswers? }` |
| `sticker` / `file` | sem opções extra |

## ctx.i18n

Veja [ctx.i18n](/docs/api/ctx-i18n/) para exemplos.

| Método | Assinatura | Descrição |
|---|---|---|
| `t` | `(key, context?)` | Traduz chave dos locales do **core**. |
| `createT` | `(import.meta.url)` | Retorna `{ t, lang }` — `t()` escopado aos locales do plugin, `lang` é o idioma ativo no momento. |
| `reload` | `()` | Recarrega traduções do disco. |
| `getCurrentLang` | `()` | `"pt"`, `"en"`, etc. |

## ctx.scheduler

Veja [ctx.scheduler](/docs/api/ctx-scheduler/) para exemplos.

| Método | Assinatura | Descrição |
|---|---|---|
| `schedule(expression, fn)` | `(string, () => Promise<void>) => { stop(): void }` | Registra uma tarefa cron; retorna um handle pra cancelar. |

## ctx.storage

Veja [ctx.storage](/docs/api/ctx-storage/) para exemplos.

| Prop/Método | Descrição |
|---|---|
| `dir` | Caminho absoluto do diretório de dados. |
| `resolve(relativePath)` | Resolve caminho dentro de `dir`, criando subpastas. |

## ctx.settings

Veja [ctx.settings](/docs/api/ctx-settings/) para exemplos.

| Método | Descrição |
|---|---|
| `get(key, default?)` / `getAll()` | Lê uma chave (ou todas) do chat atual. Sem 2º argumento, retorna `undefined` (não `null` — diferente de `ctx.config.get()`). |
| `set(key, value)` / `delete(key)` / `deleteAll()` | Escreve/remove no chat atual. `value` precisa ser serializável em JSON. |
| `global` | Mesmos métodos acima (`get`/`set`/`delete`/`getAll`/`deleteAll`), mas escopados ao bot inteiro, sem chat associado. |
| `forChat(chatId)` | Mesmos métodos acima, escopados a outro chat específico. |
| `link(communityId)` / `unlink()` | Associa/desassocia o chat atual a uma comunidade. |
| `getCommunityId()` / `getCommunityChats()` | Consulta a associação atual. |

## ctx.plugins

Veja [ctx.plugins](/docs/api/ctx-plugins/) para exemplos.

| Método | Descrição |
|---|---|
| `get(name)` | API pública ou `null`. Dependência opcional. |
| `require(name)` | API pública ou lança erro. Dependência obrigatória. |
| `exists(name)` | `true` se o plugin está ativo. |

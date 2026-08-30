---
title: Configuração (manybot.toml)
description: Todas as chaves de manybot.toml — CLIENT_ID, CMD_PREFIX, CHATS, LANGUAGE, PHONE_NUMBER, LOGIN_METHOD — e o que recarrega sozinho.
sidebar:
  order: 2
---

Na primeira execução, o ManyBot cria `~/.manybot/manybot.toml` (no Windows,
`C:\Users\SeuUsuário\.manybot\manybot.toml`) com valores padrão comentados.
Edite esse arquivo para configurar o bot — não existe comando `manybot config`,
é só um arquivo de texto.

> 🧪 Comandos têm seu próprio arquivo, `~/.manybot/commands.yaml` — separado deste e em YAML,
> não TOML. É a nova arquitetura de comandos, ainda **experimental**: veja
> [`commands.yaml`](/docs/commands-yaml/).

## Chaves disponíveis

| Chave          | Padrão      | Descrição                                                                 |
|----------------|-------------|----------------------------------------------------------------------------|
| `CLIENT_ID`    | `"manybot"` | Identificador da sessão — separa os dados de login em `~/.manybot/sessions/<CLIENT_ID>`. Rode duas instâncias do ManyBot (dois números diferentes) na mesma máquina usando um `CLIENT_ID` distinto em cada `manybot.toml`. |
| `CMD_PREFIX`   | `"!"`       | Prefixo de comando lido por `ctx.msg.is()`/`ctx.msg.command`. Pode ser mais de um caractere (ex: `"bot "`). |
| `CHATS`        | `[]`        | Lista de JIDs permitidos. Vazio = **sem restrição**, o bot responde em qualquer conversa que participe. Veja [abaixo](#chats--restringindo-onde-o-bot-responde). |
| `LANGUAGE`     | `"en"`      | Idioma da interface e das traduções (`t()`, `ctx.i18n`, `ctx.t`). Atualmente `en`, `es` e `pt`. |
| `PHONE_NUMBER` | `""`        | Número (com DDI) usado para conectar via código de pareamento. Só é lido se `LOGIN_METHOD = "phone"`. |
| `LOGIN_METHOD` | `""`        | `"qr"` ou `"phone"`. Vazio = pergunta interativamente na primeira conexão e salva a escolha aqui automaticamente. |
| `EXCLUDE_CHATS` | `[]`       | Lista de JIDs a **ignorar**, mesmo que passem pelo filtro de `CHATS`. Útil pra excluir um chat específico sem restringir todo o resto. |
| `SECURITY_LEVEL` | `"medium"` | `"low"` / `"medium"` / `"high"` — quão cauteloso o bot é pra não parecer automatizado. Níveis mais altos deixam o bot mais lento (menos chats simultâneos, atrasos maiores), mas reduzem o risco do WhatsApp sinalizar a conta. Valor inválido cai pro padrão. |
| `LOG_LEVEL` | `"normal"` | `"normal"` (tudo) / `"clean"` (esconde ruído de rotina, mantém sucesso/avisos/erros) / `"minimal"` (só avisos e erros). Valor inválido cai pro padrão. |
| `OWNER_NUMBER` | `""`     | Número (ou JID) tratado como dono do bot — é contra esse valor que a permissão `owner: true` do [`commands.yaml`](/docs/commands-yaml/) checa. String vazia é tratada como "não configurado". |
| `ADMIN_JID` | `""`        | Número/JID que recebe alertas via WhatsApp quando o bot sobe (alertas críticos, aviso de update). Vazio desliga esse canal — o log e a notificação do SO continuam funcionando mesmo assim. |
| `SMTP_HOST` / `SMTP_PORT` / `SMTP_SEC` / `SMTP_USER` / `SMTP_PASS` / `SMTP_FROM` / `SMTP_TO` | `""` / `587` / `"starttls"` / `""` / `""` / `""` / `""` | Canal de alerta por e-mail, opcional — deixe `SMTP_HOST` em branco pra desligar. `SMTP_SEC`: `"starttls"` (porta 587, upgrade após conectar), `"ssl"` (porta 465, criptografado desde o início) ou `"none"`. |
| `SMTP_INSECURE` | `false`  | Pula validação do certificado TLS — necessário pra proxies SMTP locais com certificado autoassinado (ex: Proton Mail Bridge, Mailhog, Mailpit). Deixe `false` pra um provedor remoto real. |
| `UPDATE_CHECK_ENABLED` | `true` | Checa no npm se existe uma versão nova do ManyBot: ao iniciar, e depois a cada `UPDATE_CHECK_INTERVAL_HOURS`. |
| `UPDATE_CHECK_INTERVAL_HOURS` | `24` | Intervalo (em horas) entre checagens de atualização, quando `UPDATE_CHECK_ENABLED = true`. |
| `STATUS_ENABLED` | `true`  | Liga uma página HTTP local mostrando se o bot está online ou offline. |
| `STATUS_PORT` | `8080`     | Porta da página de status, quando `STATUS_ENABLED = true`. |

> As chaves de driver (`driver_primary`, `driver_baileys_enabled`, `driver_fallback_cooldown_ms`,
> `driver_verify_window_ms`) controlam a seleção/fallback de driver de conexão. Hoje só o driver
> Baileys existe, então na prática só valem os padrões — não há necessidade de mexer nelas.

> Essas são as chaves que o próprio ManyBot lê. Plugins podem definir e ler
> chaves adicionais no mesmo arquivo via `ctx.config.get("MINHA_CHAVE")` — veja
> [ctx.config](/docs/api/ctx-config/).

## O que recarrega sozinho

O ManyBot observa `manybot.toml` e aplica a maioria das mudanças **sem
reiniciar** — inclusive `CMD_PREFIX` e `CHATS`, lidos direto do arquivo a cada
mensagem.

`LANGUAGE` é a exceção: o sistema de traduções carrega o idioma uma única vez
e não escuta esse arquivo. Mudar `LANGUAGE` em runtime não troca o idioma dos
textos já carregados — é preciso reiniciar o bot, ou um plugin chamar
`ctx.i18n.reload()` manualmente (veja [ctx.i18n](/docs/api/ctx-i18n/)).

## CHATS — restringindo onde o bot responde

Por padrão (`CHATS = []`) o ManyBot processa mensagens de **qualquer** chat —
grupo ou privado — em que a conta esteja. Para restringir a uma lista
específica, preencha `CHATS` com os JIDs desejados:

```toml
CHATS = ["5511999999999@c.us", "120363000000000000@g.us"]
```

### Descobrindo o JID de um chat

Rode o bot com a flag `--getid`:

```bash
manybot --getid
```

Isso abre uma conexão separada (não interfere com o bot já rodando), sincroniza
a lista de chats e mostra uma seleção interativa — use as setas e espaço para
marcar um ou mais chats, Enter para confirmar. Os JIDs selecionados são
copiados para a área de transferência (e também impressos no terminal), prontos
para colar em `CHATS`.

> Grupos usam o sufixo `@g.us`; conversas privadas usam `@c.us` — esse é o formato de **conversa**
> que o ManyBot expõe em `ctx.chat.id` e no `CHATS` deste arquivo. Identidade de **pessoa**
> (`ctx.msg.sender`, `id` nos objetos de contato) é diferente: usa `@lid`, não `@c.us` — veja a
> nota completa em [ctx.msg](/docs/api/ctx-msg/).

## Arquivos legados

Versões antigas usavam `manybot.conf`/`manyplug.conf` (formato próprio, não
TOML). Na primeira execução após atualizar, o ManyBot migra o `.conf`
automaticamente para `.toml` e renomeia o original para `.conf.bak`. Esses
arquivos `.conf` estão congelados — não recebem chaves novas — então não há
motivo para criar um do zero hoje.

## Veja também

O ManyPlug tem seu próprio arquivo, `~/.manybot/manyplug.toml` (lista de
plugins ativos e preferências do CLI) — veja
[Configuração](/docs/manyplug-cli/#configuração) na página do ManyPlug CLI.

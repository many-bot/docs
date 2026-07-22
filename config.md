# Configuração (manybot.toml)

Na primeira execução, o ManyBot cria `~/.manybot/manybot.toml` (no Windows,
`C:\Users\SeuUsuário\.manybot\manybot.toml`) com valores padrão comentados.
Edite esse arquivo para configurar o bot — não existe comando `manybot config`,
é só um arquivo de texto.

## Chaves disponíveis

| Chave          | Padrão      | Descrição                                                                 |
|----------------|-------------|----------------------------------------------------------------------------|
| `CLIENT_ID`    | `"manybot"` | Identificador da sessão — separa os dados de login em `~/.manybot/sessions/<CLIENT_ID>`. Rode duas instâncias do ManyBot (dois números diferentes) na mesma máquina usando um `CLIENT_ID` distinto em cada `manybot.toml`. |
| `CMD_PREFIX`   | `"!"`       | Prefixo de comando lido por `ctx.msg.is()`/`ctx.msg.command`. Pode ser mais de um caractere (ex: `"bot "`). |
| `CHATS`        | `[]`        | Lista de JIDs permitidos. Vazio = **sem restrição**, o bot responde em qualquer conversa que participe. Veja [abaixo](#chats--restringindo-onde-o-bot-responde). |
| `LANGUAGE`     | `"en"`      | Idioma da interface e das traduções (`t()`, `ctx.i18n`, `ctx.t`). Atualmente `en`, `es` e `pt`. |
| `PHONE_NUMBER` | `""`        | Número (com DDI) usado para conectar via código de pareamento. Só é lido se `LOGIN_METHOD = "phone"`. |
| `LOGIN_METHOD` | `""`        | `"qr"` ou `"phone"`. Vazio = pergunta interativamente na primeira conexão e salva a escolha aqui automaticamente. |

> Essas são as chaves que o próprio ManyBot lê. Plugins podem definir e ler
> chaves adicionais no mesmo arquivo via `ctx.config.get("MINHA_CHAVE")` — veja
> [ctx.config](/docs/08-ctx-utilities/#ctxconfig).

## O que recarrega sozinho

O ManyBot observa `manybot.toml` e aplica a maioria das mudanças **sem
reiniciar** — inclusive `CMD_PREFIX` e `CHATS`, lidos direto do arquivo a cada
mensagem.

`LANGUAGE` é a exceção: o sistema de traduções carrega o idioma uma única vez
e não escuta esse arquivo. Mudar `LANGUAGE` em runtime não troca o idioma dos
textos já carregados — é preciso reiniciar o bot, ou um plugin chamar
`ctx.i18n.reload()` manualmente (veja [ctx.i18n](/docs/08-ctx-utilities/#ctxi18n)).

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

> Grupos usam o sufixo `@g.us`; conversas privadas usam `@c.us` (esse é o
> formato que o ManyBot expõe em `ctx.chat.id`, `ctx.msg.sender` e nos objetos
> de contato — veja a nota sobre isso em [ctx.msg](/docs/03-ctx-messaging/)).

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

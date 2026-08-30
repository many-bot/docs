---
title: commands.yaml
description: A nova arquitetura de comandos do ManyBot — identidade, permissões, argumentos, indicador de carregamento e menu centralizados num único arquivo. Experimental.
sidebar:
  order: 6.5
  label: commands.yaml 🧪
---

> ⚠️ **Experimental — em testes, não é 100% funcional ainda.** `commands.yaml` é a nova
> arquitetura de comandos do ManyBot. O núcleo já está implementado e coberto por testes
> (parser, registry, permissões, deprecação automática, menu nativo, encadeamento de funções,
> indicador de carregamento), mas dois pontos seguem deliberadamente pendentes: (1) o kernel
> ainda **não obriga** que todo comando esteja no `commands.yaml` — um plugin sem entrada
> nenhuma continua funcionando do jeito antigo, e isso só deve virar obrigatório quando o YAML
> passar a ser o caminho primário; (2) ainda não existe um aviso quando a mesma função é
> referenciada no YAML **e** chamada diretamente por outro plugin (uso duplo/conflito). Fora
> isso, o schema pode ganhar campos novos ou mudar de comportamento sem aviso em versões `5.x`
> — não é recomendado depender dela em produção ainda. Quando amadurecer o suficiente, essa
> arquitetura deve **substituir** o sistema atual de comandos/plugins (leitura de
> `ctx.msg.command`, menu manual, etc.) na **ManyBot 6.0**. Até lá, os dois modelos convivem:
> um plugin pode continuar decidindo sozinho se responde a uma mensagem (como hoje), ou ter sua
> identidade/permissões declaradas aqui.

## O que muda

Hoje, um plugin decide sozinho se uma mensagem é "seu" comando (`ctx.msg.is("figurinha")`),
cuida da própria checagem de permissão, e não existe um menu automático — cada bot precisa de um
plugin de menu configurado manualmente. Com `commands.yaml`:

- **Identidade e apelidos** (`cmd`, `aliases`) saem do código do plugin e vão pro YAML.
- **Permissões** (admin, dono, apenas grupo/DM, lista de chats permitidos, cooldown,
  whitelist/blacklist) são aplicadas pelo kernel *antes* do plugin rodar — o plugin não precisa
  checar isso sozinho.
- **Descrição e manual** (`desc`, `manual`) alimentam um **menu nativo**, gerado automaticamente
  conforme comandos são registrados.
- **Indicador de "processando..."** (`loading:`) também é decidido pelo kernel — reação,
  "digitando...", "gravando áudio...", spinner por edição de mensagem, ou nenhum.
- **Argumentos obrigatórios** podem ser declarados (`arguments:`) e o kernel valida antes de
  chamar o plugin, respondendo sozinho com uma dica de uso (gerada automaticamente a partir dos
  tipos declarados) quando falta algo.
- **Uma função ou várias em sequência** (`function:`/`functions:`) — o kernel pode chamar mais
  de uma função do plugin, em ordem, para o mesmo comando.
- **Renomear ou remover** um comando é detectado automaticamente (o histórico fica salvo em
  disco, sobrevive a reinícios) e pode notificar quem tentar o nome antigo por um período de
  transição, em vez de simplesmente parar de responder.

O `commands.yaml` fica em `~/.manybot/commands.yaml`, ao lado do `manybot.toml` — mas, diferente
da configuração geral do bot (que é TOML), aqui é YAML.

## Estrutura do arquivo

Por padrão, comandos são declarados **direto na raiz** do arquivo — as únicas chaves de topo
reservadas são `defaults`, `menu`, `categories`, `manuals`, `import`, `loading_presets`,
`loading`, `prefix`, `notify_changes`, `notify_period_days`, `deprecation_message`,
`permission_messages` e `commands`; qualquer outra chave de primeiro nível é interpretada como
um comando, e o nome da chave vira o `id` interno estável desse comando.

Também é aceito envolver os comandos numa chave `commands:` — as duas formas são equivalentes e
podem inclusive ser combinadas (uma chave dentro de `commands:` e outra na raiz), desde que não
colidam:

```yaml
defaults:
  notifyPeriodDays: 7

commands:
  figurinha:
    cmd: figurinha
    plugin: many-media
```

Se a mesma chave aparecer nos dois lugares (raiz e dentro de `commands:`), a colisão é reportada
como erro no log e a versão de dentro de `commands:` é ignorada.

`prefix:` na raiz é aceito pelo parser mas hoje é **apenas informativo** — o prefixo de
verdade usado em runtime continua vindo de `manybot.toml`. Não espere que mudar `prefix:` aqui
mude o prefixo do bot.

## Exemplo mínimo

```yaml
figurinha:
  cmd: figurinha
  aliases: [f, sticker]
  plugin: many-media
  category: midia
  desc: "Transforma uma imagem/vídeo em figurinha"
```

## Estrutura de um comando

```yaml
<id-estável>:
  cmd: <palavra digitada>
  aliases: [<lista>]
  plugin: <nome-do-plugin>       # OU text: "resposta fixa" (sem plugin)
  function: <nome-da-função>     # ou functions: [fn1, fn2, ...] — ver "Encadeamento de funções"
  category: <id-da-categoria>
  group: <id-de-grupo>           # 🚧 aceito no schema, mas hoje sem efeito no menu — ver nota abaixo
  desc: "descrição curta"        # opcional — sobrescreve a desc do plugin
  manual: "texto longo"          # opcional — cai pra manuals.<id> se omitido
  deprecatedMessage: "..."       # opcional — mensagem de depreciação específica deste comando
  notifyChanges: true            # opcional — sobrescreve defaults.notifyChanges só pra este comando
  loading: <preset-ou-inline>    # opcional — indicador de "processando...", ver seção própria
  permissions: { ... }
  messages: { ... }
  arguments: [ ... ]             # ou args: [...]
  subcommands: [ ... ]
```

A chave de topo (`figurinha` no exemplo) é um **id interno estável**, desacoplado do `cmd:` (a
palavra que o usuário digita). É esse id estável que permite detectar renomeações: mudar só o
`cmd:` de um comando já registrado (mantendo o mesmo id) é tratado como um rename, não como
remover um comando e criar outro.

`cmd` é obrigatório — sem ele, o comando não carrega e um aviso é registrado no log. `desc` e
`manual` são totalmente opcionais: sem `desc`, o campo simplesmente fica de fora do menu; sem
`manual` (quando alguém pede o manual explicitamente), o kernel cai pro `desc`.

Um comando é ou `plugin:` (roteia pra uma ou mais funções de plugin) ou `text:` (resposta fixa)
— ou, sem nenhuma função própria, um **comando-contêiner** que só agrupa `subcommands:` (ver
seção própria abaixo). Declarar `plugin:` sem `function`/`functions` e sem `subcommands:` não
carrega nada — o kernel loga um aviso e ignora a entrada.

`plugin:` aceita tanto a chave completa do registry (`owner/repo`) quanto só o nome curto do
plugin (resolvido automaticamente contra os plugins ativos — se o nome curto bater com mais de
um `owner/repo`, a ambiguidade não é resolvida sozinha, e o valor original é mantido, gerando um
comando "órfão" com aviso no log). Também aceita a forma inline `plugin: many-media.removerFundo`
(ponto separando plugin e função) como atalho pra `plugin: many-media` + `function: removerFundo`.

> 🚧 O campo `group:` no schema é pensado pra agrupar comandos diferentes numa mesma entrada do
> menu — mas ele ainda **não tem efeito nenhum** hoje (não é lido em nenhum lugar do
> renderizador do menu). Evite depender dele até que o agrupamento seja implementado de fato.

### Texto e manual longos a partir de arquivo

`text:`/`manual:` aceitam conteúdo inline ou uma referência `file:./caminho` (relativo à pasta de
config, `~/.manybot/`). O conteúdo é lido literalmente, sem nenhum parser — a formatação nativa
do WhatsApp (`*negrito*`, `_itálico_`, etc.) já funciona direto no arquivo. Se o arquivo não for
encontrado, o kernel loga um aviso e usa a própria referência (`file:./...`) como texto.

```yaml
regras:
  cmd: regras
  text: "file:./textos/regras.txt"
```

### Descrição e manual por idioma

`desc`/`manual` aceitam string simples ou um mapa por idioma, usando o mesmo sistema de i18n do
resto do ManyBot ([`ctx.i18n`](/docs/api/ctx-i18n/)):

```yaml
figurinha:
  cmd: figurinha
  plugin: many-media
  desc:
    pt: "Transforma uma imagem/vídeo em figurinha"
    en: "Turns an image/video into a sticker"
```

## Encadeamento de funções (`functions:`)

Além de um único `function: "nome"`, um comando pode declarar uma **cadeia** de funções do
mesmo plugin, executadas em ordem:

```yaml
banir:
  cmd: banir
  plugin: many-mod
  functions: [logarTentativa, executarBanimento]
```

Cada função recebe a mesma forma `(ctx, { args, subcommand })`. Por padrão, o retorno de uma
função não interrompe a cadeia (mesmo `undefined`/vazio deixa a próxima rodar) — para
interromper deliberadamente, uma função pode devolver o sentinela `STOP_CHAIN` (exportado de
`commandsConfig.ts`). Uma cadeia vazia (comando sem `function`/`functions` e sem `subcommands`
com handler próprio) significa que o comando é só metadado — nada é executado.

## Indicador de carregamento (`loading:`)

Enquanto o plugin processa, o kernel pode mostrar um indicador de "processando..." sem que o
plugin precise cuidar disso:

```yaml
figurinha:
  cmd: figurinha
  plugin: many-media
  loading:
    type: reaction
    icon: "⏳"
    onSuccess: "✅"
    onError: "❌"
```

Tipos reconhecidos:

- `reaction` — reage com um emoji na mensagem original (`icon`, `onSuccess`/`on_success`,
  `onError`/`on_error`).
- `typing` — presença nativa "digitando..." do WhatsApp. Não aceita propriedades extras.
- `recording_audio` — presença nativa "gravando áudio...". Mesma regra do `typing`.
- `spinner` — edita uma mensagem enviada pelo próprio bot, ciclando por uma lista de `frames` a
  cada `intervalMs`/`interval_ms` (mínimo 1000ms; valores menores são arredondados pra cima).
  Aceita também `onSuccess`/`onError`.
- `none` — desliga explicitamente qualquer indicador.

Uma propriedade não reconhecida para o tipo declarado invalida o `loading:` inteiro (falha
fechada — o kernel loga erro e trata como se nada tivesse sido declarado ali).

`loading:` aceita tanto um objeto inline quanto o **nome de um preset** definido em
`loading_presets:` (chave de topo):

```yaml
loading_presets:
  padrao:
    type: reaction
    icon: "⏳"

figurinha:
  cmd: figurinha
  plugin: many-media
  loading: padrao
```

A cadeia de herança (do mais específico pro mais genérico) é:

```
comando/subcomando  →  categoria (categories.<id>.loading)  →  defaults.loading (ou loading: no topo)
```

O primeiro nível que declarar um `loading:` (inline ou preset já resolvido) vence; níveis
omitidos simplesmente passam adiante pro próximo da cadeia. Um subcomando sem `loading:` próprio
herda o `loading:` já resolvido do comando pai.

## Permissões

```yaml
banir:
  cmd: banir
  plugin: many-mod
  permissions:
    admin: true          # quem manda precisa ser admin do grupo
    botAdmin: true        # o bot precisa ser admin do grupo
    scope: group          # group | dm | any (padrão: any)
    owner: false           # true = só o número configurado como OWNER_NUMBER
    dono: "5511999999999"  # opcional — dono específico deste comando, tem prioridade sobre owner/OWNER_NUMBER
    allowedChats: []        # lista fechada de chats (grupos e DMs) onde o comando pode rodar
    cooldownSeconds: 5
    whitelist:
      groups: ["120363...@g.us"]
      users: ["5511999999999@c.us"]
    blacklist:
      groups: []
      users: []
  messages:
    senderNotAdmin: "Só admins podem usar esse comando."
    botNotAdmin: "Preciso ser admin do grupo pra fazer isso."
    ownerOnly: "Esse comando é restrito ao dono do bot."
    donoOnly: "Esse comando é restrito a um número específico."
    wrongScope: "Esse comando só funciona em grupos."
    allowedChats: "Esse comando não está liberado neste chat."
    blacklist: "Você não pode usar esse comando aqui."
    cooldown: "Calma — espera {{seconds}}s antes de tentar de novo."
```

A ordem de checagem é fixa: `dono` → `owner` → `scope` → `allowedChats` → `blacklist` →
`whitelist` → admin do bot (`botAdmin`) → admin de quem enviou (`admin`) → cooldown (só é
consumido se todas as checagens anteriores passarem). Se `whitelist` e `blacklist` derem match ao
mesmo tempo, **`blacklist` vence**. `dono` (JID/número específico deste comando) tem prioridade
sobre o `owner` genérico — se `dono` estiver definido, só ele é checado nesse passo, o
`OWNER_NUMBER` global do bot nem entra.

A mensagem de `cooldown` aceita os placeholders `{{seconds}}` e `{{time}}` — ambos resolvem pro
mesmo valor (segundos restantes).

Padrões definidos em `defaults.permissions`/`defaults.messages` valem pra todo comando que não
sobrescrever o campo; um plugin também pode declarar seus próprios padrões de permissão, mas o
que estiver no `commands.yaml` sempre vence.

### Formas alternativas (flat) dos mesmos campos

Além da forma canônica acima, o parser aceita nomes alternativos "achatados" (o mesmo YAML de
referência do projeto usa essa variante):

| Forma canônica | Forma flat equivalente |
|---|---|
| `scope: group` | `group_only: true` |
| `scope: dm` | `dm_only: true` |
| `whitelist: { groups: [...] }` | `whitelist_groups: [...]` (mescla com a forma nested, se as duas existirem) |
| `blacklist: { users: [...] }` | `blacklist_users: [...]` (idem) |
| (sem equivalente canônico) | `allowed_chats: [...]` — mesmo campo que `allowedChats` |
| (sem equivalente canônico) | `hidden_outside_scope: true` — mesmo campo que `hiddenOutsideScope`, esconde o comando do menu fora do escopo dele |

Declarar `group_only: true` e `dm_only: true` ao mesmo tempo é inválido — o parser loga um aviso
e ignora o escopo (fica como se nada tivesse sido declarado).

`defaults`/topo também aceitam um bloco `permission_messages:` com nomes próprios, que é
traduzido internamente pros mesmos campos de `messages:` acima:

```yaml
permission_messages:
  admin_only: "Só admins."       # -> senderNotAdmin E botNotAdmin
  dono_only: "Só o dono."        # -> ownerOnly E donoOnly
  group_only: "Só em grupo."     # -> wrongScope
  cooldown: "Espera {{seconds}}s."
  blacklist: "Bloqueado aqui."
  allowed_chats: "Não liberado aqui."
```

Também é possível sobrescrever `defaults.notifyChanges`/`notifyPeriodDays`/`notifyMessage` via
chaves de topo equivalentes: `notify_changes`, `notify_period_days`, `deprecation_message`. Se as
duas formas aparecerem (bloco `defaults:` e chave de topo), a chave de topo vence — é um overlay
raso, sem merge profundo (mesma regra do `import:`).

## Argumentos obrigatórios

```yaml
banir:
  cmd: banir
  plugin: many-mod
  arguments:
    - name: alvo
      type: mention
      required: true
```

`args:` funciona como sinônimo de `arguments:`, e `optional: true` como o inverso de
`required: true` — se nenhum dos dois for informado, o argumento é opcional por padrão.

Tipos reconhecidos hoje: `mention`, `url`, `media_direct`, `media_reply` (`media_direct_or_reply`
também é aceito e normalizado silenciosamente para `media_reply`), `number`, `duration`,
`choice` (use `choices: [...]` junto), `boolean`, `quoted_text`, `reply`. Faltando um argumento
`required: true`, o kernel responde sozinho com uma dica de uso, gerada automaticamente a partir
dos tipos declarados — o plugin nem chega a rodar. Exemplo de dica gerada para o `banir` acima:
`!banir @<user>`. Cada tipo tem sua própria notação na dica (`--nome=<a|b|c>` pra `choice`,
`--nome[=true|false]` pra `boolean`, `<n>` pra `number`, etc.); argumentos opcionais aparecem
entre colchetes. Parsing de texto livre continua responsabilidade do plugin; o kernel só
reconhece essas formas estruturadas.

## Subcomandos

Cada sub-ação de um comando (ex: `!figurinha remover-fundo`) é declarada como um item de uma
**lista** `subcommands:` — não como um mapa:

```yaml
figurinha:
  cmd: figurinha
  aliases: [f]
  plugin: many-media
  subcommands:
    - cmd: remover-fundo
      aliases: [rf]
      function: removerFundo   # ou functions: [...] — omitido = herda a cadeia de funções do pai
      desc: "Remove o fundo da figurinha"
```

Se `subcommands:` não for uma lista, o kernel ignora todos os subcomandos daquele comando (com um
aviso no log). O `id` de cada subcomando não é escolhido por quem escreve o YAML — é derivado
automaticamente como `<id-do-pai>::<cmd>` (ex.: `figurinha::remover-fundo`), útil pra reconhecer
o subcomando em logs. Dois subcomandos com o mesmo `cmd` (comparado sem diferenciar
maiúsculas/minúsculas) dentro do mesmo pai: o segundo é ignorado com aviso no log.

Um subcomando herda a cadeia de `functions` do pai quando não declara a sua própria, e herda o
`loading:` já resolvido do pai quando não declara o seu. Permissões seguem a mesma lógica de
sempre (própria > padrão da função-default do plugin > `defaults.permissions`), com o `scope` do
pai como fallback quando o subcomando não define o seu. Aliases **não** são herdados do pai — a
lista de `aliases:` de um subcomando é sempre a que ele mesmo declara (vazia, se omitida).
Sessão/estado próprios do fluxo (timeout, histórico de mídia recebida, etc.) continuam vivendo
inteiramente dentro do plugin — não fazem parte do `commands.yaml`.

## Comandos-contêiner (subcomandos sem handler próprio)

Um comando pode existir só pra agrupar `subcommands:`, sem `function`/`functions` própria — útil
quando o comando "pai" nunca deve ser chamado sozinho (ex.: `!todo` sem nada depois não faz
sentido, só `!todo add`/`!todo list` fazem):

```yaml
todo:
  cmd: todo
  plugin: many-utils
  subcommands:
    - cmd: add
      function: addItem
    - cmd: list
      function: listItems
```

Esse tipo de comando nunca é despachado diretamente — o kernel sempre roteia pra um dos
`subcommands:`. Um token de subcomando desconhecido depois do `!todo` cai no fallback padrão
("subcomando desconhecido", com a lista de subcomandos válidos), da mesma forma que aconteceria
com um comando normal que tem subcomandos.

## Deprecação automática

Renomear o `cmd:` de um comando (mantendo o mesmo id) ou remover a chave por completo marca o
nome antigo como **deprecated** por um período de transição (`notifyPeriodDays`, padrão 7 dias).
O histórico de cmd por id, e as deprecações ativas, ficam persistidos em disco (no mesmo banco
SQLite usado pelas configurações do bot) — então isso sobrevive a reinícios do processo; a janela
de aviso continua contando a partir do momento real da mudança, não é reiniciada a cada boot.
Durante essa janela:

- quem digitar o nome antigo recebe um aviso configurável (placeholders `{{old}}`, `{{new}}`,
  `{{days}}` — chave dupla, igual ao resto do sistema de i18n do ManyBot);
- registrar um novo comando reaproveitando esse nome antigo é bloqueado, pra evitar confusão.

```yaml
defaults:
  notifyChanges: true
  notifyPeriodDays: 7
  notifyMessage: "O comando {{old}} virou {{new}}. Esse aviso some em {{days}} dias."
```

`notifyChanges` também pode ser sobrescrito por comando individual, assim como a mensagem: um
comando pode declarar seu próprio `deprecatedMessage`, que tem prioridade sobre
`defaults.notifyMessage` quando presente. Sem nenhum dos dois definidos, o kernel cai pra um
texto padrão embutido (traduzido via i18n).

> 🐛 **Legado a migrar:** o comando `figurinha` ainda implementa sua própria depreciação
> manualmente (não via `commands.yaml`) — essa migração não foi feita ainda.

## Menu

```yaml
menu:
  title: "🤖 Meu Bot — Menu"
  intro: "Use {prefix}<comando> pra executar ou {prefix}help <comando> pra ver o manual."
  footer: "Feito com ManyBot"
  cmd: menu
  aliases: [help, man, menu, bot, "?"]
  notFoundFallback: false
  welcomeMessage: "Oi! Digite {prefix}menu pra ver o que eu faço."
  welcomeWindowDays: 3
  pageSize: 15

categories:
  midia:
    label: "Mídia"
    order: 1
  moderacao:
    label: "Moderação"
    order: 2
    scope: group           # comandos dessa categoria assumem scope: group por padrão
    hiddenInScope: dm       # categoria some do menu quando visualizada numa DM
    loading: padrao          # default de loading pra comandos dessa categoria sem loading: próprio
```

Note que `{prefix}` em `intro`/`footer`/`welcomeMessage` usa chave **simples** — diferente dos
placeholders `{{old}}`/`{{new}}`/`{{days}}`/`{{seconds}}`/`{{time}}` das seções acima. Não
misture os dois formatos.

`welcomeMessage` é mostrado automaticamente pra um usuário na primeira mensagem dele dentro da
janela `welcomeWindowDays` — substitui a antiga flag permanente `seen_users`. Duas checagens
extras evitam um "welcome fantasma": a mensagem recebida precisa ter texto não vazio, e precisa
ter chegado há no máximo 60s (protege contra o buffer de eventos do Baileys reclassificando
recibos antigos como mensagens novas depois de uma reconexão). `notFoundFallback` controla se o
bot responde algo quando um comando com prefixo não bate com nada conhecido (mesmo depois dos
plugins legados rodarem); como plugins legados não conseguem avisar que "trataram" a mensagem,
isso pode gerar uma resposta duplicada — por isso vem desligado por padrão.

`hiddenInScope` (por categoria) e `hiddenOutsideScope`/`hidden_outside_scope` (por
comando/subcomando, dentro de `permissions:`) já são levados em conta pelo menu de verdade — uma
categoria some inteira do menu quando ele é renderizado no escopo declarado em
`hiddenInScope`, e um comando some do menu fora do escopo declarado em `hiddenOutsideScope`
(exceto quando o escopo resolvido for `any`).

## Importando outros arquivos

`commands.yaml` pode importar outros arquivos YAML, cada um contribuindo com seções de nível
raiz inteiras (`menu:`, `categories:`, `manuals:`, ou comandos completos declarados na raiz):

```yaml
import:
  - menu.yaml
  - manual.yaml
```

`import:` também aceita uma string única (um só arquivo), não só uma lista. A resolução acontece
**antes** de desembrulhar um eventual `commands:` no arquivo principal — então uma chave
importada pode colidir tanto com a raiz quanto com o conteúdo de dentro de `commands:`.

Não existe merge profundo: cada chave de nível raiz só pode ser declarada em **um** lugar (o
arquivo principal ou um dos importados). Se dois arquivos declararem a mesma chave, a primeira
declaração vence e a segunda é ignorada com um erro no log — isso evita que uma importação
sobrescreva silenciosamente algo já definido. Um `import:` aninhado dentro de um arquivo
importado é ignorado (não é recursivo). Um arquivo de import ausente ou malformado (erro de
leitura, YAML inválido, ou raiz que não é um objeto) gera um erro no log e é pulado, sem derrubar
o carregamento do resto do `commands.yaml`.

## Comandos declarados pelo próprio plugin

Além de declarar tudo no `commands.yaml`, um plugin pode exportar seus próprios comandos padrão
direto no código:

```js
export const commands = {
  figurinha: {
    cmd: "figurinha",
    aliases: ["f", "sticker"],
    desc: "Transforma uma imagem/vídeo em figurinha",
    category: "midia",
    handler: async (ctx) => { /* ... */ },
  },
};
```

Cada entrada aceita tanto uma função "pura" (só `handler`, sem identidade própria — só fica
acessível se o `commands.yaml` do usuário fornecer o `cmd`) quanto um objeto completo com
identidade própria (`cmd`/`aliases`/`desc`/`category`/`manual`/`permissions` já definidos no
plugin). Quando o mesmo id também existe no `commands.yaml` do usuário, o YAML **sempre
sobrescreve** o que o plugin declarou — o mesmo princípio já vale hoje para permissões, e se
aplica à identidade inteira do comando. Uma cadeia `functions: [a, b]` declarada no YAML pode
encadear várias entradas desse mesmo mapa `commands`, desde que todas pertençam ao mesmo plugin.

## Consultando comandos de dentro de um plugin

`ctx.commands` deixa um plugin checar se um comando existe (ou ler sua descrição/manual) sem
precisar de `ctx.plugins.require()` do plugin dono — veja
[ctx.commands](/docs/api/ctx-commands/) 🧪.

## Ainda não decidido / em aberto

- Se/quando o agrupamento via `group:` (ver nota acima) será implementado no menu.
- Aviso de uso duplo/conflito quando a mesma função de plugin é referenciada no `commands.yaml`
  **e** chamada diretamente por outro plugin via `ctx.plugins.require()`/`.get()` — a abordagem
  cogitada é um check em runtime comparando o nome da função pedida contra o conjunto de nomes
  registrados como `function:`/`functions:` no command registry, mas ainda não foi implementada
  nem validada.
- Aplicação obrigatória de "todo comando precisa estar no `commands.yaml`" está deliberadamente
  adiada — hoje é opt-in, e plugins sem entrada nenhuma continuam funcionando do jeito antigo
  (`ctx.msg.is(...)` dentro do próprio `default()`). Isso só deve virar obrigatório quando o
  YAML passar a ser o caminho primário de fato.
- Migração da depreciação manual do comando `figurinha` para o mecanismo automático do
  `commands.yaml` (ver nota na seção de deprecação acima).


---
title: commands.yaml
description: A nova arquitetura de comandos do ManyBot — identidade, permissões, argumentos e menu centralizados num único arquivo. Experimental.
sidebar:
  order: 6.5
  label: commands.yaml 🧪
---

> ⚠️ **Experimental — em testes, não é 100% funcional ainda.** `commands.yaml` é a nova
> arquitetura de comandos do ManyBot, ainda em desenvolvimento. Ela já funciona para boa parte
> dos casos descritos nesta página, mas partes específicas podem ter comportamento incompleto ou
> mudar sem aviso em versões `5.x` — não é recomendado depender dela em produção ainda. Quando
> amadurecer o suficiente, essa arquitetura deve **substituir** o sistema atual de
> comandos/plugins (leitura de `ctx.msg.command`, menu manual, etc.) na **ManyBot 6.0**. Até lá,
> os dois modelos convivem: um plugin pode continuar decidindo sozinho se responde a uma
> mensagem (como hoje), ou ter sua identidade/permissões declaradas aqui.

## O que muda

Hoje, um plugin decide sozinho se uma mensagem é "seu" comando (`ctx.msg.is("figurinha")`),
cuida da própria checagem de permissão, e não existe um menu automático — cada bot precisa de um
plugin de menu configurado manualmente. Com `commands.yaml`:

- **Identidade e apelidos** (`cmd`, `aliases`) saem do código do plugin e vão pro YAML.
- **Permissões** (admin, apenas grupo/DM, apenas o dono, cooldown, whitelist/blacklist) são
  aplicadas pelo kernel *antes* do plugin rodar — o plugin não precisa checar isso sozinho.
- **Descrição e manual** (`desc`, `manual`) alimentam um **menu nativo**, gerado automaticamente
  conforme comandos são registrados.
- **Argumentos obrigatórios** podem ser declarados (`arguments:`) e o kernel valida antes de
  chamar o plugin, respondendo sozinho com uma dica de uso quando falta algo.
- **Renomear ou remover** um comando é detectado automaticamente e pode notificar quem tentar o
  nome antigo por um período de transição, em vez de simplesmente parar de responder.

O `commands.yaml` fica em `~/.manybot/commands.yaml`, ao lado do `manybot.toml` — mas, diferente
da configuração geral do bot (que é TOML), aqui é YAML.

## Estrutura do arquivo

Comandos são declarados **direto na raiz** do arquivo — não existe uma chave `commands:` de
agrupamento. As únicas chaves de topo reservadas (não tratadas como comando) são `defaults`,
`menu`, `categories`, `manuals` e `import`; qualquer outra chave de primeiro nível é interpretada
como um comando, e o nome da chave vira o `id` interno estável desse comando.

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
  function: <nome-da-função>     # opcional — quando o plugin exporta mais de uma
  category: <id-da-categoria>
  desc: "descrição curta"        # opcional — sobrescreve a desc do plugin
  manual: "texto longo"          # opcional — cai pra manuals.<id> se omitido
  deprecatedMessage: "..."       # opcional — mensagem de depreciação específica deste comando
  permissions: { ... }
  messages: { ... }
  arguments: [ ... ]
  subcommands: [ ... ]
```

A chave de topo (`figurinha` no exemplo) é um **id interno estável**, desacoplado do `cmd:` (a
palavra que o usuário digita). É esse id estável que permite detectar renomeações: mudar só o
`cmd:` de um comando já registrado (mantendo o mesmo id) é tratado como um rename, não como
remover um comando e criar outro.

`cmd` é obrigatório — sem ele, o comando não carrega e um aviso é registrado no log. `desc` e
`manual` são totalmente opcionais: sem `desc`, o campo simplesmente fica de fora do menu; sem
`manual` (quando alguém pede o manual explicitamente), o kernel usa um texto placeholder.

Um comando é ou `plugin:` (roteia pra uma função de plugin) ou `text:` (resposta fixa, sem
plugin nenhum) — não os dois.

> 🚧 Existe também um campo `group:` no schema, pensado pra agrupar comandos diferentes numa
> mesma entrada do menu — mas ele ainda **não tem efeito nenhum** hoje (não é lido em nenhum
> lugar do renderizador do menu). Evite depender dele até que o agrupamento seja implementado de
> fato.

### Texto e manual longos a partir de arquivo

`text:`/`manual:` aceitam conteúdo inline ou uma referência `file:./caminho` (relativo à pasta de
config, `~/.manybot/`). O conteúdo é lido literalmente, sem nenhum parser — a formatação nativa
do WhatsApp (`*negrito*`, `_itálico_`, etc.) já funciona direto no arquivo.

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

## Permissões

```yaml
banir:
  cmd: banir
  plugin: many-mod
  permissions:
    admin: true          # quem manda precisa ser admin do grupo
    botAdmin: true        # o bot precisa ser admin do grupo
    scope: group          # group | dm | any (padrão: any)
    owner: false           # true = só o número configurado como dono
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
    wrongScope: "Esse comando só funciona em grupos."
    cooldown: "Calma — espera {{seconds}}s antes de tentar de novo."
```

A ordem de checagem é fixa: dono (`owner`) → escopo (`scope`) → `blacklist` → `whitelist` →
admin do bot (`botAdmin`) → admin de quem enviou (`admin`) → cooldown (só é consumido se todas as
checagens anteriores passarem). Se `whitelist` e `blacklist` derem match ao mesmo tempo,
**`blacklist` vence**.

A mensagem de `cooldown` aceita os placeholders `{{seconds}}` e `{{time}}` — ambos resolvem pro
mesmo valor (segundos restantes).

Padrões definidos em `defaults.permissions`/`defaults.messages` valem pra todo comando que não
sobrescrever o campo; um plugin também pode declarar seus próprios padrões de permissão, mas o
que estiver no `commands.yaml` sempre vence.

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

Tipos reconhecidos hoje: `mention`, `url`, `media_direct`, `media_reply`, `number`, `duration`,
`choice` (use `choices: [...]` junto), `boolean`, `quoted_text`, `reply`. Faltando um argumento
`required: true`, o kernel responde sozinho com uma dica de uso — o plugin nem chega a rodar.
Parsing de texto livre continua responsabilidade do plugin; o kernel só reconhece essas formas
estruturadas.

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
      function: removerFundo   # opcional — herda a function do pai se omitido
      desc: "Remove o fundo da figurinha"
```

Se `subcommands:` não for uma lista, o kernel ignora todos os subcomandos daquele comando (com um
aviso no log). O `id` de cada subcomando não é escolhido por quem escreve o YAML — é derivado
automaticamente como `<id-do-pai>::<cmd>` (ex.: `figurinha::remover-fundo`), útil pra reconhecer
o subcomando em logs.

Um subcomando herda as permissões do comando pai por padrão, mas pode sobrescrevê-las
independentemente. Sessão/estado próprios do fluxo (timeout, histórico de mídia recebida, etc.)
continuam vivendo inteiramente dentro do plugin — não fazem parte do `commands.yaml`.

## Deprecação automática

Renomear o `cmd:` de um comando (mantendo o mesmo id) ou remover a chave por completo marca o
nome antigo como **deprecated** por um período de transição (`notifyPeriodDays`, padrão 7 dias).
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
`defaults.notifyMessage` quando presente.

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
```

Note que `{prefix}` em `intro`/`footer`/`welcomeMessage` usa chave **simples** — diferente dos
placeholders `{{old}}`/`{{new}}`/`{{days}}`/`{{seconds}}`/`{{time}}` das seções acima. Não
misture os dois formatos.

`welcomeMessage` é mostrado automaticamente pra um usuário na primeira mensagem dele dentro da
janela `welcomeWindowDays` — substitui a antiga flag permanente `seen_users`. `notFoundFallback`
controla se o bot responde algo quando um comando com prefixo não bate com nada conhecido (mesmo
depois dos plugins legados rodarem); como plugins legados não conseguem avisar que "trataram" a
mensagem, isso pode gerar uma resposta duplicada — por isso vem desligado por padrão.

## Importando outros arquivos

`commands.yaml` pode importar outros arquivos YAML, cada um contribuindo com seções de nível
raiz inteiras (`menu:`, `categories:`, `manuals:`, ou comandos completos declarados na raiz):

```yaml
import:
  - menu.yaml
  - manual.yaml
```

Não existe merge profundo: cada chave de nível raiz só pode ser declarada em **um** lugar (o
arquivo principal ou um dos importados). Se dois arquivos declararem a mesma chave, a primeira
declaração vence e a segunda é ignorada com um erro no log — isso evita que uma importação
sobrescreva silenciosamente algo já definido. Um `import:` aninhado dentro de um arquivo
importado é ignorado (não é recursivo). Um arquivo de import ausente ou malformado gera um erro
no log e é pulado, sem derrubar o carregamento do resto do `commands.yaml`.

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
aplica à identidade inteira do comando.

## Consultando comandos de dentro de um plugin

`ctx.commands` deixa um plugin checar se um comando existe (ou ler sua descrição/manual) sem
precisar de `ctx.plugins.require()` do plugin dono — veja
[ctx.commands](/docs/api/ctx-commands/) 🧪.

## Ainda não decidido / em aberto

- Se/quando o agrupamento via `group:` (ver nota acima) será implementado no menu.
- Como o kernel vai expor os comandos padrão embutidos de um plugin (via `export const
  commands`, seção acima) pra que apareçam automaticamente no menu sem entrada nenhuma no
  `commands.yaml` do usuário — o mecanismo exato de descoberta automática ainda não está fechado.
- Aplicação obrigatória de "todo comando precisa estar no `commands.yaml`" está deliberadamente
  adiada — hoje é opt-in, e plugins sem entrada nenhuma continuam funcionando do jeito antigo
  (`ctx.msg.is(...)` dentro do próprio `default()`).


# ManyPlug CLI

O gerenciador de plugins oficial do ManyBot.

```bash
manyplug <comando> [opções]
# ou, mais curto:
mp <comando> [opções]
```

`manyplug` e `mp` são exatamente o mesmo comando — use o que preferir.

---

## init

Cria o esqueleto de um novo plugin, pluginpack ou profile.

```bash
manyplug init <nome>
```

`init` sempre pergunta interativamente o **nome do autor** (usado pra montar a `key` do plugin,
`autor/nome`) — não tem flag pra isso. Se o diretório `<nome>` já existir, pergunta antes de
sobrescrever.

### Opções

- `-c`, `--category <cat>` — categoria do plugin: `integration`, `games`, `media`, `utility`,
  `admin`, `fun`, `moderation`, `ai`, `education`, `social`, `economy`, `automation`, `tools`
  (padrão: `utility`; categoria inválida gera aviso e cai pro padrão)
- `-t`, `--type <tipo>` — `plugin`, `pluginpack` ou `profile` (padrão: `plugin`)
- `--lang <lang>` — linguagem do plugin: `js` ou `ts`. Se omitido (ou inválido), é perguntado
  interativamente — Enter sem digitar nada assume `js`. Não se aplica a `--type profile`
  (profiles não têm código).

### Estrutura gerada (plugin, JavaScript)

```
<nome>/
├── index.js
├── manyplug.json
├── package.json        # devDependencies: @manybot/types
├── README.md
├── .gitignore
└── locale/
    ├── pt.json
    └── en.json
```

### Estrutura gerada (plugin, TypeScript)

Com `--lang ts` (ou escolhendo `ts` no prompt), a estrutura muda um pouco:

```
<nome>/
├── src/
│   └── index.ts
├── manyplug.json      # "main": "dist/index.js"
├── package.json       # inclui "scripts.build" e devDependencies: typescript, @manybot/types
├── tsconfig.json
├── README.md
├── .gitignore         # já ignora dist/
└── locale/
    ├── pt.json
    └── en.json
```

O ManyBot só carrega arquivos `.js` já compilados — ele não transpila TypeScript. Antes de
`manyplug install --local .`, rode:

```bash
npm install
npm run build
```

Isso gera `dist/index.js`, que é o arquivo que `manyplug.json` aponta em `main`. Repita o build
sempre que editar `src/index.ts`.

Em ambos os casos (JS ou TS), o pacote `@manybot/types` (adicionado como `devDependency` pelo
`init`) descreve o formato do `ctx` — importe dele (`import('@manybot/types').PluginContext` via
JSDoc, ou `import type { PluginContext } from "@manybot/types"` em TS) pra ter autocomplete no
seu editor. Veja a [Referência da API](/docs/00-index) pra saber o que cada parte de `ctx` faz.

### `--type pluginpack` / `--type profile`

Com `--type pluginpack`, o `init` também gera uma subpasta `example-plugin/` com um plugin de
exemplo completo (mesma estrutura de um plugin normal), mostrando como organizar os demais. Com
`--type profile`, gera só o `manyplug.json` com `"plugins": []` pra você preencher — sem
`index.js`/`package.json`, já que profiles não têm código. Ambos vêm com um `README.md` já
explicando o formato.

---

## install

Instala plugins, pluginpacks ou profiles do [mpindex](https://manybot.stxerr.dev/manyplug/mpindex.json)
ou de um caminho local. Alias: `i`.

```bash
manyplug install <autor/plugin> [plugin2...]
manyplug i <autor/plugin>
```

```bash
# local
manyplug install --local <caminho>
```

### Opções

- `-l`, `--local <caminho>` — instala de um diretório local com `manyplug.json` válido
- `-w`, `--watch` — observa mudanças e reinstala automaticamente (requer `--local`)
- `-b`, `--branch <branch>` — instala de uma branch específica
- `-y`, `--yes` — pula confirmação

> Plugins sem `key` no `manyplug.json` são instalados em `manydev/<nome>`. Adicione `"key": "autor/nome"` para evitar isso.

Aceita o nome curto (`manyplug install figurinha`) se for único no registry — havendo mais de um
autor com um plugin de mesmo nome, o comando lista as opções e pede a key completa
(`autor/figurinha`). Instalar um plugin cujo nome curto já pertence a outro plugin diferente
(outro autor) já instalado é recusado, pra evitar ambiguidade.

O download é feito num diretório temporário e só é movido pro lugar final depois de validado —
uma instalação cancelada ou que falhe no meio não deixa um plugin quebrado pra trás. Se o
registry tiver mais de um espelho pro mesmo plugin (ex: Codeberg e GitHub), o `install` tenta o
próximo automaticamente se um falhar.

### Pluginpacks e profiles

Além de plugins individuais, `install` também entende dois outros tipos de pacote, identificados
pelo campo `"type"` do `manyplug.json`:

- **pluginpack** — um repositório com vários plugins dentro, cada um em sua própria subpasta com
  seu próprio `manyplug.json`. Instalar o pack instala cada plugin individualmente — depois de
  instalado, cada um funciona exatamente como se tivesse sido instalado sozinho.
- **profile** — só uma lista de plugins (`"plugins": ["autor/nome", ...]`) pro ManyPlug baixar,
  sem código nenhum próprio. Instalar um profile busca cada plugin listado no registry.

Ambos são criados com `manyplug init --type pluginpack` ou `--type profile`.

---

## link

Cria um symlink de um plugin (ou pluginpack) local dentro da pasta de plugins — como `npm link`.
Edições no código-fonte valem na hora, sem precisar reinstalar. Alias: `ln`.

```bash
manyplug link [caminho]   # padrão: .
manyplug ln
```

Pluginpacks linkam cada subplugin individualmente. Profiles não têm código próprio pra linkar e
não são suportados por esse comando — use `install` nesse caso.

---

## search

Busca plugins no registry por nome, key, categoria ou descrição. Alias: `s`.

```bash
manyplug search <busca>
manyplug s figurinha
```

### Opções

- `-c`, `--category <cat>` — filtra por categoria

Plugins já instalados aparecem marcados como `[installed]` no resultado; entradas do tipo
pluginpack ou profile aparecem marcadas como `[pluginpack]`/`[profile]`.

---

## update

Reinstala todos os plugins não-locais com as versões mais recentes do registry. Alias: `up`.

```bash
manyplug update
```

### Opções

- `-y`, `--yes` — pula confirmação

> Plugins locais (instalados com `--local`) e sem `key` são ignorados.

---

## list

Lista plugins instalados. Por padrão, mostra só os ativos. Alias: `ls`.

```bash
manyplug list
manyplug ls --all
```

### Opções

- `-a`, `--all` — inclui plugins desativados

A listagem mostra nome, versão, categoria e status — que pode ser `enabled`, `disabled` ou
`incomplete` (o arquivo de entrada declarado em `main` não existe mais no disco).

---

## enable / disable

Ativa ou desativa plugins instalados. Aceita múltiplos nomes de uma vez. Aliases: `en` e `dis`.

```bash
manyplug enable <plugin> [plugin2...]
manyplug en <plugin>

manyplug disable <plugin> [plugin2...]
manyplug dis <plugin>
```

```bash
# por profile
manyplug enable -p myprofile
manyplug disable --profile myprofile
```

### Opções

- `-a`, `--all` — ativa/desativa todos os plugins instalados
- `-p`, `--profile <profile>` — ativa/desativa todos os plugins instalados através desse profile.
  Aceita o nome curto ou a key completa (`autor/profile`); se o nome curto for ambíguo entre
  profiles de autores diferentes, o comando lista as opções e pede a key completa. Tem prioridade
  sobre nomes de plugin e sobre `--all` — se usado junto, os outros argumentos são ignorados.

> Só plugins instalados **através** do profile (`manyplug install autor/profile`) ficam
> marcados como pertencentes a ele — instalar o plugin avulso depois não cria essa marcação.

O ManyBot detecta a mudança sozinho e aplica em segundos — não precisa reiniciar o bot.

---

## remove

Remove plugins instalados. Oferece a opção de apagar os dados do plugin também. Alias: `rm`.

```bash
manyplug remove <plugin> [plugin2...]
manyplug rm <plugin>
```

### Opções

- `-y`, `--yes` — pula confirmação de remoção do plugin (a confirmação de remoção dos dados
  continua sendo perguntada)
- `-Y` — pula **as duas** confirmações (plugin e dados) sozinho, sem precisar de `-y` junto

---

## validate

Valida um plugin, pluginpack ou profile local. Alias: `val`.

```bash
manyplug validate [caminho]   # padrão: .
```

Checagens no `manyplug.json`:
- Campos obrigatórios (`name`, `version`, `category`) e tipos de todos os campos conhecidos.
- Campo não reconhecido no manifesto → aviso.
- `name`/`key` seguem o formato descrito em [manyplug.json](/docs/how-to-make-a-plugin/#manyplugjson); se `key` estiver presente, sua parte depois da `/` precisa bater com `name`.
- `manybotVersion`, se presente, é comparado com a versão do ManyBot instalada.
- Entry point (`main`) existe no disco.
- Pluginpacks: cada subpasta precisa ter seu próprio `manyplug.json` válido — pack sem nenhum
  subplugin é erro. Profiles: `plugins` precisa ser uma lista não vazia de keys.

Checagens de dependências:
- `package.json` → cada dependência declarada tem uma pasta correspondente em `node_modules`
  (aviso se faltando — rode `npm install`).
- `manyplug.json`'s `dependencies` (outros plugins) → compara com o que o código realmente usa via
  `ctx.plugins.require()`: completa automaticamente uma dependência detectada no código mas
  ausente do manifesto, e avisa sobre uma dependência declarada mas nunca usada.
- `externalDependencies` → cada `command` é procurado no `PATH`. Faltando: erro se `optional`
  for `false`/omitido (bloqueia, sai com código 1); aviso se `optional: true`.
- O código também é escaneado por chamadas a `exec`/`execSync`/`execFile`/`execFileSync`/`spawn`/
  `spawnSync` com um binário literal (ex: `execSync("ffmpeg ...")`) — mesmo que esse binário não
  esteja em `externalDependencies`, o `validate` avisa se ele não estiver no `PATH`.

Checagens de `locale/`:
- Todos os arquivos de idioma existem e são JSON válido.
- As mesmas chaves existem em todos os idiomas — chave presente num arquivo e ausente em outro
  gera aviso, pra pegar traduções esquecidas.

Checagens de uso de `ctx` no código:
- Desestruturação (`const { x } = ctx`) e acesso direto (`ctx.x`, `ctx.x.y`) são comparados com as
  chaves e métodos reais da API — nome errado ou digitado errado gera aviso.
- Chamar `ctx.send(...)` ou `ctx.msg.reply(...)` diretamente (em vez de `ctx.send.text(...)`,
  `ctx.msg.reply.text(...)`, etc.) é detectado e sinalizado.

Qualquer **erro** (não aviso) faz o comando sair com código 1 — útil pra travar CI/hooks de
publicação.

---

## info

Mostra detalhes de um plugin instalado: nome, key, versão, categoria, autor, licença, repo, tipo,
status, arquivo de entrada (`main`), caminho no disco, tamanho, diretório de dados (e tamanho, ou
"nenhum"), descrição, dependências (outros plugins, com versão) e dependências externas.

```bash
manyplug info <plugin>
```

Aceita nome curto (`meu-plugin`) ou chave completa (`autor/meu-plugin`) — mesma resolução usada
por `enable`/`disable`/`remove`/`install`: se o nome curto for ambíguo entre plugins de autores
diferentes, é pedida a key completa.

---

## version

Exibe ou atualiza a versão no `manyplug.json` do plugin atual.

```bash
manyplug version           # exibe a versão atual
manyplug version 1.2.0     # atualiza para 1.2.0
```

> Pode ser qualquer string, não precisa seguir semver. Diferente de `validate`, não aceita um
> caminho — sempre opera no `manyplug.json` do diretório atual (`cd` até a pasta do plugin antes
> de rodar).

---

## help

```bash
manyplug help
manyplug help <comando>
```

Rodar `manyplug` sem nenhum argumento também mostra essa ajuda. Note que `-h`/`--help` **não**
funcionam (ao contrário da maioria das CLIs) — use `help` mesmo.

### `-v`, `--version`

Mostra a versão instalada do ManyPlug e sai — funciona em qualquer lugar, não precisa estar
numa pasta de plugin (diferente de `manyplug version`, que é sobre o `manyplug.json` do plugin
atual).

```bash
manyplug --version
manyplug -v
```

---

## Configuração

Na primeira execução de qualquer comando, o ManyPlug cria `~/.manybot/manyplug.toml` — nele fica a
lista de plugins ativos (gerenciada por `enable`/`disable`) e algumas preferências:

| Chave       | Padrão                                              | Descrição                                             |
|-------------|------------------------------------------------------|--------------------------------------------------------|
| `LANGUAGE`  | `"auto"`                                              | Idioma da interface. `"auto"` detecta do sistema; pode ser fixado (ex: `"pt"`). |
| `REGISTRY`  | `https://manybot.stxerr.dev/manyplug/mpindex.json`    | URL do registry usado por `install`/`search`/`update`. |
| `CONFIRM`   | `true`                                                | Se `false`, pula confirmações que normalmente pedem `-y`. |

O idioma também pode ser sobrescrito por comando com a variável de ambiente `MANYPLUG_LANG`
(útil pra scripts e CI), sem precisar editar o arquivo.

> O ManyPlug também mantém um cache interno em `~/.manybot/registry.json` (metadados de
> instalação — o que foi instalado local/linkado/via qual profile). Não é pra editar à mão; se
> parecer corrompido ou desatualizado, é seguro apagar — ele é reconstruído na próxima operação.

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

### Opções

- `-c`, `--category <cat>` — categoria do plugin: `integration`, `games`, `media`, `utility`,
  `admin`, `fun`, `moderation`, `ai`, `education`, `social`, `economy`, `automation`, `tools`
  (padrão: `utility`)
- `-t`, `--type <tipo>` — `plugin`, `pluginpack` ou `profile` (padrão: `plugin`)
- `--lang <lang>` — linguagem do plugin: `js` ou `ts`. Se omitido, é perguntado interativamente.

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

Plugins já instalados aparecem marcados como `[installed]` no resultado.

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

- `-y`, `--yes` — pula confirmação de remoção do plugin
- `-Y` — pula também a confirmação de remoção dos dados

---

## validate

Valida o `manyplug.json` de um plugin, pluginpack ou profile local — campos obrigatórios, tipos,
entry point, locale, dependências externas e uso de `ctx` no código. Alias: `val`.

```bash
manyplug validate [caminho]   # padrão: .
```

---

## info

Mostra detalhes de um plugin instalado: versão, categoria, tipo, status, tamanho, dados e dependências.

```bash
manyplug info <plugin>
```

Aceita nome curto (`meu-plugin`) ou chave completa (`autor/meu-plugin`).

---

## version

Exibe ou atualiza a versão no `manyplug.json` do plugin atual.

```bash
manyplug version           # exibe a versão atual
manyplug version 1.2.0     # atualiza para 1.2.0
```

> Pode ser qualquer string, não precisa seguir semver.

---

## help

```bash
manyplug help
manyplug help <comando>
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

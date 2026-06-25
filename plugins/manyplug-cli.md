# ManyPlug CLI
O gerenciador de plugins oficial do ManyBot.

---

## install

Instala plugins do [mpindex](https://manybot.stxerr.dev/manyplug/mpindex.json) ou de um caminho local.

```bash
manyplug install <autor/plugin> [plugin2...]
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

---

## update

Reinstala todos os plugins não-locais com as versões mais recentes do registry.

```bash
manyplug update
```

### Opções
- `-y`, `--yes` — pula confirmação

> Plugins locais (instalados com `--local`) e sem `key` são ignorados.

---

## list, ls

Lista plugins instalados. Por padrão, mostra só os ativos.

```bash
manyplug list
manyplug ls --all
```

### Opções
- `-a`, `--all` — inclui plugins desativados

---

## enable / disable

Ativa ou desativa plugins instalados. Aceita múltiplos nomes de uma vez.

```bash
manyplug enable <plugin> [plugin2...]
manyplug disable <plugin> [plugin2...]
```

### Opções
- `-a`, `--all` — desativa/ativa todos os plugins instalados

---

## remove, rm

Remove plugins instalados. Oferece a opção de apagar os dados do plugin também.

```bash
manyplug remove <plugin> [plugin2...]
manyplug rm <plugin>
```

### Opções
- `-y`, `--yes` — pula confirmação de remoção do plugin
- `-Y` — pula todas as confirmações, incluindo a dos dados

---

## init

Cria o esqueleto de um novo plugin para desenvolvimento.

```bash
manyplug init <nome>
```

Estrutura gerada:

```
<nome>/
├── index.js
├── manyplug.json
├── package.json
├── README.md
├── .gitignore
└── locale/
    ├── pt.json
    └── en.json
```

### Opções
- `-c`, `--category <cat>` — categoria do plugin: `games`, `media`, `utility`, `service`, `admin`, `fun` (padrão: `utility`)
- `--service` — marca como plugin de serviço em segundo plano

---

## validate, val

Valida o `manyplug.json` de um plugin local — campos obrigatórios, tipos, entry point, locale e dependências externas.

```bash
manyplug validate [caminho]   # padrão: .
```

---

## version

Exibe ou atualiza a versão no `manyplug.json` do plugin atual.

```bash
manyplug version           # exibe a versão atual
manyplug version 1.2.0     # atualiza para 1.2.0
```

> Pode ser qualquer string, não precisa seguir semver.

---

## info

Mostra detalhes de um plugin instalado: versão, categoria, tipo, status, tamanho, dados e dependências.

```bash
manyplug info <plugin>
```

Aceita nome curto (`meu-plugin`) ou chave completa (`autor/meu-plugin`).

---

## help

```bash
manyplug help
manyplug help <comando>
```

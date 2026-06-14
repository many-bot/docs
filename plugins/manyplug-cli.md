# ManyPlug CLI

O gerenciador de plugins oficial do ManyBot.

Abaixo você encontra informações sobre todos os comandos e como usar cada.

---

## install

### O que faz?

Instala plugins do [manyplug-repo](https://github/many-bot/manyplug-repo) (obsoleto) ou do [mpindex](/manyplug/mpindex.json)
 ou de um repositório Git.

### Uso

*Atual** (mpindex) - recomendado.

```bash
manyplug install <usuário/plugin> [...opções]
```

**Legacy** (manyplug-repo) - depende do **Git** instalado no sistema e será desativado em breve.

```bash
manyplug install <nome no manyplug-repo>
```

### Opções

- `-l`, `--local <caminho>` - instala de um arquivo local de um plugin válido
- `-y`, `--yes` - instala sem confirmação
- `--needed` - instala somente se já não estiver instalado

---

## list, ls

### O que faz?

Lista plugins instalados e ativos.

### Uso:

```bash
manyplug list
```

### Opções:

- `-a`, `--all` - lista até plugins desativados

---

## enable

### O que faz?

Ativa plugins instalados.

### Uso:

```bash
manyplug enable <plugin>
```

---

## disable

### O que faz?

Desativa plugins instalados.

### Uso:

```bash
manyplug disable <plugin>
```

---

## remove, rm

### O que faz?

Remove plugins instalados.

### Uso:

```bash
manyplug remove <plugin>
```

ou

```bash
manyplug rm <plugin>
```

## Opções

- `-y`, `--yes` - remove sem confirmação
- `--remove-deps` - além do plugin, remove dependências - era útil apenas nas versões 3.x e anteriores do ManyBot, cujo o manyplug já não suporta mais.

---

## sync (obsoleto)

### O que faz?

Sincroniza registry local - entrando em desuso em novas versões.

### Uso:

```bash
manyplug sync
```

## Opções

- `-f`, `--force` - sobrescreve o registro mesmo se nada mudou

---

## init

### O que faz?

Cria um template de plugin para desenvolvimento.

Estrutura padrão:

```bash
plugin
├── index.js
├── locale
│   ├── en.json
│   └── pt.json
├── manyplug.json
├── package.json
└── README.md
```

### Uso:

```bash
manyplug init <nome> 
```

## Opções

- `-c`, `--category` - define uma categoria para o plugin (válidos: `games`, `media`, `utility`, `service`, `admin` e `fun`)

---

## update

### O que faz?

Atualiza todos os plugins de uma só vez. 

### Uso:

```bash
manyplug update 
```

## Opções

- `-y`, `--yes` - atualiza sem confirmação

## Observação

Plugins que não estão no `mpindex` ou no `manyplug-repo` não são atualizados automaticamente.

---

## validate, val

### O que faz?

Valida se um plugin local está com a estrutura correta.

### Uso:

```bash
manyplug validate [caminho] 
```

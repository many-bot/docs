# Como fazer um plugin

Aqui você vai aprender tudo o que precisa para começar a desenvolver plugins para o ManyBot.

## Índice:

- [Estrutura padrão de um plugin](#estrutura-padrao-de-um-plugin)
- [Publicando seu plugin](#publicando-seu-plugin)

---

# Estrutura padrão de um plugin

```bash
plugin
├── index.js
├── manyplug.json
├── package.json
├── README.md
└── locale
    ├── en.json
    ├── es.json
    └── pt.json
```

> O comando `init` do `manyplug` monta essa estrutura manualmente. Veja [manyplug init](/docs/manyplug-cli/#init).

## index.js

Aquivo principal do plugin, é aqui onde começa a execução.

Aqui você exporta a função `default` e recebe a [API do ManyBot](/docs/api-reference).

Exemplo:

```js
export default async function (ctx) {
    const { msg } = ctx.msg;
    const pfx = ctx.config.get("CMD_PREFIX");

    if (msg.is(pfx + "hello")) {
        msg.reply("Hello!");
    }
}
```

## manyplug.json

Arquivo essencial caso queira publicar seu plugin. Todos os metadados do seu plugin ficam aqui.

Exemplo:

```json
{
    "name": "meu-plugin",
    "key": "eu/meu-plugin",
    "author": {
        "name": "eu",
        "email": "meu@email.com",
        "website": "www.meusite.com"
    },
    "description": "Meu plugin legal pro ManyBot.",
    "version": "1.0.0",
    "category": "utility",
    "service": false,
    "dependencies": {
        "dependencia-npm": ">=10"
    },
    "externalDependencies": {
        "programa": {
            "command": "programa",
            "optional": false
        }
    }
}
```

### Campos

#### `name`
Nome do plugin. Usado como identificador legível e no registro. Deve ser único por autor.

#### `key`
Chave global do plugin no formato `autor/nome`. Usada para referenciá-lo no `mpindex` e em dependências de outros plugins.

#### `author`
Handle do autor. O único campo obrigatório é o "name", o resto é opcional.

#### `description`
Descrição curta do que o plugin faz. Exibida na listagem do registro.

#### `version`
Versão atual do plugin, formato flexível, use o que mais gostar (SemVer, CalVer, etc).

#### `category`
Categoria do plugin. Valores possíveis: `integration`, `games`, `media`, `utility`, `service`, `admin` e `fun`

#### `service`
`true` se o plugin roda em background como serviço (sem comandos diretos). `false` se é acionado exclusivamente por comandos.

#### `dependencies`
Dependências npm necessárias. O formato é `"pacote": "range semver"`. Serão instaladas automaticamente pelo ManyPlug.

#### `externalDependencies`
Programas externos necessários no sistema (ex: `yt-dlp`, `ffmpeg`). Cada entrada aceita:
- `command`: comando usado para verificar se o programa está disponível
- `optional`: se `false`, o plugin não carrega sem ele; se `true`, degrada graciosamente

## package.json

Arquivo mínimo, a única coisa que é obrigatório ter é:

```
{
    "type": "module"
}
```

## README.md

Arquivo Markdown simples explicando seu plugin. Não é obrigatório, mas é altamente recomendado para ajudar
 outros usuários a entender como utilizar.

É preferencial o Inglês, mas isso não é tão restrito.

Exemplo real:

```
# ManyMedia
Download videos and audio from YouTube, Reddit, Instagram, and other yt-dlp supported sites — either sending the file directly to chat or uploading to a storage server and sharing the link.

## Features
- **Multi-site support**: YouTube, Reddit, Instagram, SoundCloud, TikTok, and any other yt-dlp compatible site
- **Audio extraction**: Downloads and extracts MP3 at best quality
- **Flexible delivery**: Send file directly to chat, or upload to a storage server and reply with the link
- **Upload retry**: Failed uploads are retried up to 4 times with a 3-second delay
- **Queued processing**: Downloads run in a queue to prevent resource contention
- **Automatic cleanup**: Temporary files removed after delivery

## Requirements
- `yt-dlp` installed and available in `PATH`
- `ffmpeg` it is needed when convering filetypes (e.g. mp4 to mp3) and post-processing, but for downloading is optional
- `cookies.txt` file in the project root (**required** — used for YouTube, Reddit, and other sites that need authentication)

## Usage
!video https://youtube.com/watch?v=...
!video https://www.reddit.com/r/...
!video https://www.instagram.com/reel/...
!audio https://youtube.com/watch?v=...

## Configuration
Add to `manybot.conf`:

| Key | Default | Description |
|-----|---------|-------------|
| `UPL_MEDIA_TO_SRV` | `no` | Set to `yes` to upload to the storage server and reply with a link instead of sending the file directly |
| `MEDIA_SRV_API_KEY` | — | API key for the storage server (required when `UPL_MEDIA_TO_SRV=yes`) |

### Example
UPL_MEDIA_TO_SRV=yes
MEDIA_SRV_API_KEY=your_api_key_here

When `UPL_MEDIA_TO_SRV=no` (default), the file is sent directly to the chat with no external upload.  
When `UPL_MEDIA_TO_SRV=yes`, the file is uploaded to `https://api.stxerr.dev/upload` and the reply contains the download link.

> Make sure to have your API key to use upload. If you don't have one, request it by sending an email to me@stxerr.dev.

## cookies.txt Setup
`cookies.txt` is required for Reddit, YouTube, and other sites that enforce login or rate limits. Without it, downloads from these sites will fail.

**1. Install a browser extension** to export cookies in Netscape format:
- Chrome/Edge: [Get cookies.txt LOCALLY](https://chromewebstore.google.com/detail/get-cookiestxt-locally/cclelndahbckbenkjhflpdbgdldlbecc)
- Firefox: [cookies.txt](https://addons.mozilla.org/en-US/firefox/addon/cookies-txt/)

**2. Log in** to Reddit (and YouTube if needed) in your browser.

**3. Export** the cookies using the extension and save the file as `cookies.txt` in the root of the ManyBot project (`/opt/manybot/cookies.txt`).

> Cookies expire over time. If downloads start failing again, re-export and replace the file.

## Localization
Available in:
- English (`locale/en.json`)
- Portuguese (`locale/pt.json`)
- Spanish (`locale/es.json`)
```

## locale

Diretório com traduções do seu plugin, exemplo:

en.json:
```
{
  "hello": "Hello!"
}
```

pt.json:
```
{
  "hello": "Olá!"
}
```

e no código:

```
export default async function (ctx) {
    const { msg } = ctx.msg;
    const pfx     = ctx.config.get("CMD_PREFIX");
    const { t }   = ctx.i18n.createT(import.meta.url);

    if (msg.is(pfx + "hello")) {
        msg.reply(t("hello"));
    }
}
```

O ManyBot vai automaticamente pegar a mensagem correta dependendo do idioma configurado.

---

# Publicando seu plugin

Agora é hora do mundo conhecer sua criação! Para isso, siga os passos:

## 1. Teste seu plugin

Antes de tudo, para ter absoluta certeza, teste antes de prosseguir para confirmar de que tudo funciona corretamente.

Para isso:

```
manyplug install --path /caminho/do/seu/plugin
```

O ManyPlug vai instalar para você no lugar correto.

Depois disso execute o ManyBot e verifique se os comportamentos são esperados.

## 2. Enviando uma solicitação

Envie um email para [manybot@pm.me](mailto:manybot@pm.me), com a seguinte estrutura:

```
Assunto: [PLUGIN-REQUEST] <Nome do seu plugin>

# O que ele faz?

Escreva aqui o que seu plugin faz, resumidadamente

# Repositório(s)

GitHub: https://github.com/.../...
Codeberg: https://codeberg.org/.../...
```

O email pode ser tanto em inglês quanto em português.

Certifique-se de ter um README explicativo no repositório para que os desenvolvedores
 entendam o que seu plugin faz.

## 3. Publicação

Após seu plugin ser revisado e testado por nós, vamos te enviar um email de volta com
 assuntos como:

- Se eu plugin foi aceito ou não e o motivo
- Se tem algo a mais que precisa saber
- Feeback sobre 
- Etc.

Nosso atendimento é 100% humano e buscamos ser o mais amigável possível. Sem medo!

Depois do seu plugin ser aceito, ele será incluído no index oficial (mpindex), e
 ficará mais ou menos dessa forma:

```
"synt-xerror/manymedia": {
    "repos": {
        "codeberg": {
            "branchs": { 
                "master": "https://codeberg.org/many-bot/manybot/archive/master.tar.gz"
                "dev": "https://codeberg.org/many-bot/manybot/archive/dev.tar.gz"
            }
        },
        "github": {
            "branchs": { 
                "master": "https://github.com/many-bot/manybot/archive/refs/heads/master.tar.gz"
                "dev": "https://github.com/many-bot/manybot/archive/refs/heads/dev.tar.gz"
            }
        },
    }
}
```

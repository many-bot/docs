---
title: Primeiros passos
description: Do zero a um ManyBot respondendo mensagens em poucos minutos — instalação, conexão e primeiros testes.
sidebar:
  order: 1
---

Caso você só queira um bot simples pra tarefas básicas (como baixar vídeos/áudios,
fazer figurinhas, etc), recomendamos usar a instância oficial, adicionando o número:
**+55 16 99459-1903**.

Se o seu objetivo é hospedar o seu próprio bot personalizado, esta página te leva do
zero a um bot respondendo mensagens em poucos minutos.

> Em caso de dúvidas, você pode nos contatar via [manybot@pm.me](mailto:manybot@pm.me)
> ou nas nossas comunidades do [WhatsApp](https://chat.whatsapp.com/KfOuIwhpQjN8fcZTMHmaGQ) e [Discord](https://discord.gg/gC7aKChXmA).

## Índice

- [O que você vai instalar](#o-que-voce-vai-instalar)
- [Linux](#instalacao-linux)
- [Windows](#instalacao-windows)
- [Android (Termux)](#instalacao-android-termux)
- [Conectando o bot](#conectando-o-bot)
- [Colocando o bot pra responder algo](#colocando-o-bot-pra-responder-algo)
- [Problemas comuns (Windows)](#problemas-comuns-windows)

> **FreeBSD** e **macOS** não foram testados. Caso queira compatibilidade com algum outro
> sistema, crie uma issue no [GitHub](https://github.com/many-bot/manybot) ou no
> [Codeberg](https://codeberg.org/many-bot/manybot), ou envie um email com a sugestão para
> [manybot@pm.me](mailto:manybot@pm.me).

---

## O que você vai instalar

Duas ferramentas, ambas via `npm`:

- **[ManyBot](https://www.npmjs.com/package/@manybot/manybot)** — o bot em si. Sozinho ele não faz
  nada além de conectar: toda funcionalidade (figurinhas, downloads, moderação, etc.) vem de plugins.
- **[ManyPlug](https://www.npmjs.com/package/@manybot/manyplug)** — o gerenciador de plugins, usado
  pra instalar e ativar o que o bot vai efetivamente fazer. Pode ser chamado tanto de `manyplug`
  quanto do alias mais curto `mp`.

É necessário **Node.js 24 ou mais recente**.

---

## Instalação Linux

Instale o Node.js e o NPM de acordo com sua distribuição Linux (pode ser necessário acesso _root_):

| Distro               | Comando                           |
| --------------------- | --------------------------------- |
| Debian / Ubuntu        | `sudo apt install nodejs npm`     |
| Fedora                 | `sudo dnf install nodejs npm`     |
| Arch Linux             | `sudo pacman -S nodejs npm`       |
| openSUSE               | `sudo zypper install nodejs npm`  |
| Alpine                 | `sudo apk add nodejs npm`         |
| Void Linux             | `sudo xbps-install -S nodejs npm` |
| Gentoo                 | `sudo emerge nodejs npm`          |

> Verifique em qual distribuição o seu sistema se baseia —
> por exemplo, Linux Mint é baseado em Ubuntu, logo o comando de instalação é o mesmo.
>
> Se o pacote da sua distro instalar uma versão do Node abaixo da 20, use o
> [NodeSource](https://github.com/nodesource/distributions) ou o [nvm](https://github.com/nvm-sh/nvm)
> pra pegar uma versão mais recente.

Confirme a versão instalada:

```bash
node -v   # precisa ser 24.x ou mais recente
```

---

Verifique o prefixo global do NPM e veja se tem acesso a ele com:

```bash
npm config get prefix
```

Exemplo de saída:

```
/usr
```

Caso o diretório mostrado não seja acessível pelo seu usuário e necessite de permissões de outro usuário,
você pode mudar o prefixo para algo acessível usando o comando:

```bash
npm config set prefix ~/.npm-global # ou outro diretório
```

Verifique:

```bash
npm config get prefix
```

Depois certifique-se de que o PATH inclui o novo diretório.

No Bash:

```bash
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.bashrc
```

No Zsh:

```zsh
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.zshrc
```

Depois disso feche e abra seu terminal novamente para reiniciar o shell.

---

Agora instale o **ManyBot** e o **ManyPlug**:

```bash
npm install -g @manybot/manybot @manybot/manyplug
```

Confirme que os dois foram instalados:

```bash
npm list -g @manybot/manybot @manybot/manyplug
manyplug --version   # ou: mp --version
```

> `manybot` não tem uma flag `--version` — rodá-lo já tenta conectar (veja a próxima seção).

---

## Instalação Windows

Baixe e instale o Node.js pelo [site oficial](https://nodejs.org) — pegue a versão **LTS mais
recente** (24 ou superior). Durante a instalação, certifique-se de marcar a opção **"Add to PATH"**.

Após a instalação, abra o **Prompt de Comando** ou **PowerShell** e verifique se está tudo certo:

```powershell
node -v
npm -v
```

Em seguida, instale o **ManyBot** e o **ManyPlug**:

```powershell
npm install -g @manybot/manybot @manybot/manyplug
```

Confirme que os dois foram instalados:

```powershell
npm list -g @manybot/manybot @manybot/manyplug
manyplug --version
```

---

## Instalação Android (Termux)

Use o [Termux](https://termux.dev/) — instale pela [F-Droid](https://f-droid.org/packages/com.termux/)
ou pelo [GitHub](https://github.com/termux/termux-app/releases) (**não** use a versão da Play
Store, ela está desatualizada e sem suporte).

Recomendamos o uso do Node.js LTS no Termux, que tem mais suporte que a versão travada dele.

```bash
pkg update && pkg upgrade
pkg install nodejs-lts
```

Confirme a versão instalada:

```bash
node -v   # precisa ser 24.x ou mais recente
```

Instale o **ManyBot** e o **ManyPlug** normalmente:

```bash
npm install -g @manybot/manybot @manybot/manyplug
```

### Mantendo o bot vivo em segundo plano

Android mata processos em segundo plano de forma agressiva pra economizar bateria — isso derruba
o bot se você trocar de app ou apagar a tela sem esses ajustes:

- Roda `termux-wake-lock` antes de subir o bot, pra impedir o sistema de suspender o Termux.
  `termux-wake-unlock` desfaz.
- Nas configurações do Android, tira o Termux da otimização de bateria (geralmente em
  **Configurações → Apps → Termux → Bateria → Sem restrições**) — o caminho exato varia por
  fabricante.
- Considere um gerenciador de processos como o [pm2](https://pm2.keymetrics.io/) (funciona no
  Termux igual em qualquer Linux) pra reconectar sozinho se o processo cair mesmo assim.

> **Cuidado com dependências nativas em plugins:** o Termux roda numa arquitetura e numa libc
> (Bionic, não glibc) diferente da maioria das distros Linux. Pacotes npm com binários nativos
> pré-compilados (`sqlite3`, `bcrypt`, `sharp`, `canvas`, etc.) frequentemente não têm binário
> pronto pra essa combinação e caem pra compilar do zero — o que exige toolchain de compilação
> extra (`pkg install clang make python`) e, mesmo assim, pode falhar ou travar a instalação do
> plugin em vários dispositivos. Veja [boas práticas de plugins](/docs/best-practices) antes de
> escolher dependências, especialmente se pretende publicar o plugin pra outras pessoas usarem.

---

## Conectando o bot

Com tudo instalado, rode pela primeira vez:

```bash
manybot
```

Na primeira execução, o ManyBot:

1. Cria a pasta de configuração em `~/.manybot/` (no Windows, `C:\Users\SeuUsuário\.manybot\`),
   com o arquivo `manybot.toml` dentro — é nele que fica toda a configuração do bot (veja a
   [referência completa](/docs/config)).
2. Pergunta como você quer conectar: **escanear um QR Code** ou **receber um código de pareamento**
   no número de telefone. Essa escolha fica salva, então só é perguntada uma vez.
   - **QR Code**: abra o WhatsApp no celular, vá em **Configurações → Dispositivos conectados →
     Conectar um dispositivo** e escaneie o código exibido no terminal.
   - **Código de pareamento**: informe o número com DDI ao ser perguntado, e digite o código
     recebido no WhatsApp, na mesma tela de **Conectar um dispositivo**.
3. Depois de conectado, o bot fica online e escutando mensagens de **qualquer conversa** em que a
   conta participe — veja [`CHATS`](/docs/config#chats--restringindo-onde-o-bot-responde) se
   quiser restringir a chats específicos.

> Os arquivos de sessão ficam salvos em `~/.manybot/sessions/manybot/` (o nome da subpasta segue
> `CLIENT_ID`, `"manybot"` por padrão). Não compartilhe essa pasta com ninguém — quem tiver acesso
> a ela pode controlar sua conta do WhatsApp.

Pra manter o bot rodando em segundo plano (e reconectar sozinho se cair), use um gerenciador de
processos como o [pm2](https://pm2.keymetrics.io/) — isso é opcional, o `manybot` sozinho já
mantém a conexão ativa enquanto o terminal estiver aberto.

---

## Colocando o bot pra responder algo

Sozinho, o ManyBot não tem nenhum comando embutido — **tudo é plugin**. Com o bot conectado (deixe
esse terminal aberto), abra um segundo terminal e instale algo:

```bash
manyplug search figurinha
manyplug install <autor/plugin-que-encontrou>
```

Ou, se preferir só testar o fluxo sem instalar nada de terceiros, crie um plugin de exemplo:

```bash
manyplug init meu-teste --category utility
cd meu-teste
manyplug install --local .
```

Isso instala um plugin que responde `Pong!` para a mensagem `!ping`.

O ManyBot detecta a mudança automaticamente — **não precisa reiniciar** — e passa a carregar o
plugin em poucos segundos. Envie a mensagem correspondente (ex: `!ping`, com o prefixo padrão `!`)
em qualquer conversa com o número conectado, e o bot deve responder.

> `CMD_PREFIX`, `CHATS` e a maior parte das outras chaves em `~/.manybot/manybot.toml` são
> aplicadas automaticamente ao editar o arquivo, sem reiniciar o bot. A exceção é `LANGUAGE`
> (idioma das traduções), que exige reiniciar — veja [Configuração](/docs/config) pra detalhes de
> cada chave.

A partir daqui:
- Veja [Configuração](/docs/config) pra todas as chaves de `manybot.toml`, incluindo como
  restringir o bot a chats específicos.
- Veja [sobre os plugins](/docs/about-plugins) e o [ManyPlug CLI](/docs/manyplug-cli) pra explorar
  mais o gerenciador.
- Veja [como fazer um plugin](/docs/how-to-make-a-plugin) se quiser desenvolver o seu próprio.

## Problemas comuns (Windows)

### "O arquivo C:\Program Files\nodejs\npm.ps1 não pode ser carregado porque a execução de scripts está desabilitada neste sistema."

Para resolver você precisa definir a política de execução do PowerShell. Execute no PowerShell como administrador:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Digite `S` na próxima tela.

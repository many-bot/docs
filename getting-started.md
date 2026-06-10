# Primeiros passos
Caso você só queira um bot simples pra tarefas básicas (como baixar vídeos/audios,
 fazer figurinhas, etc), recomendamos que use a instância oficial, adicionando o número:
 **+55 16 99459-1903**.

No entanto, se seu objetivo é hospedar o seu próprio bot personalizado, siga os passos
 a seguir de acordo com seu sistema operacional.
> Em caso de dúvidas sobre os passos a seguir, você pode nos contatar via [manybot@pm.me](mailto:manybot@pm.me)
 ou na nossa [comunidade do WhatsApp](https://chat.whatsapp.com/KfOuIwhpQjN8fcZTMHmaGQ) e [Discord](https://discord.gg/gC7aKChXmA).

## Índice
- [Linux](#instalacao-linux)
- [Windows](#instalacao-windows)
- [Problemas comuns (Windows)](#problemas-comuns-windows)

> Outros sistemas como **FreeBSD**, **macOS** e **Android** não foram testados.
 Caso queira compatibilidade com algum outro sistema, crie uma issue no
 [GitHub](https://github.com/many-bot/manybot) ou no [Codeberg](https://codeberg.org/many-bot/manybot),
 ou envie um email com a sugestão para [manybot@pm.me](mailto:manybot@pm.me).

---

## Instalação Linux

Instale o Node.js e o NPM de acordo com sua distribuição Linux (pode ser necessário acesso _root_):

| Distro               | Comando                           |
| -------------------- | --------------------------------- |
| Debian / Ubuntu      | `sudo apt install nodejs npm`     |
| Fedora               | `sudo dnf install nodejs npm`     |
| Arch Linux           | `sudo pacman -S nodejs npm`       |
| openSUSE             | `sudo zypper install nodejs npm`  |
| Alpine               | `sudo apk add nodejs npm`         |
| Void Linux           | `sudo xbps-install -S nodejs npm` |
| Gentoo               | `sudo emerge nodejs npm`          |

> Verifique em qual distribuição o seu sistema se baseia —
 por exemplo, Linux Mint é baseado em Ubuntu, logo o comando de instalação é o mesmo.

---

Após a instalação correta, verifique o prefixo global do NPM e veja se tem acesso a ele com:

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

Após concluir a instalação e configuração do Node.js e NPM, agora você pode instalar
 o **ManyBot** e o **ManyPlug CLI** (gerenciador de plugins).

ManyBot:

```bash
npm install -g @manybot/manybot
```

ManyPlug:

```bash
npm install -g @manybot/manyplug
```

Depois disso inicie pela primeira vez:

```bash
manybot
```

Ao iniciar, o ManyBot vai gerar os arquivos de configuração no diretório atual e exibir um QR Code.
 Escaneie-o com o WhatsApp em **Dispositivos conectados → Conectar um dispositivo**.
 Após a leitura, o bot estará online.

> Os arquivos de sessão ficam salvos localmente. Não compartilhe a pasta de sessão com ninguém.

---

## Instalação Windows

Baixe e instale o Node.js pelo [site oficial](https://nodejs.org). Durante a instalação,
 certifique-se de marcar a opção **"Add to PATH"**.

Após a instalação, abra o **Prompt de Comando** ou **PowerShell** e verifique se está tudo certo:

```powershell
node -v
npm -v
```

Em seguida, instale o **ManyBot** e o **ManyPlug CLI**:

```powershell
npm install -g @manybot/manybot
npm install -g @manybot/manyplug
```

Depois disso inicie pela primeira vez:

```powershell
manybot
```

Assim como no Linux, será exibido um QR Code — escaneie-o com o WhatsApp em
 **Dispositivos conectados → Conectar um dispositivo**.

## Problemas comuns (Windows)

### "O arquivo C:\Program Files\nodejs\npm.ps1 não pode ser carregado porque a execução de scripts está desabilitada neste sistema."

Para resolver você precisa definir a política de execução do PowerShell. Execute no Powershell como administrador:

```powershell
 Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Digite 'S' na próxima tela.

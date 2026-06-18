# ManyBot Docs

Essa é a documentação oficial da ManyBot, aqui você vai encontrar tudo o que quer saber sobre o projeto,
 como instalar, como configurar, como desenvolver, manter e como contribuir para o desenvolvimento.

## Colaboração

A ManyBot é de código-aberto, sob a [Licença Pública Geral GNU 3.0](https://www.gnu.org/licenses/gpl-3.0.pt-br.html) (GPLv3).
 Isso significa que qualquer um pode inspecionar o código, modificar, redistribuir e colaborar.

Caso seja um desenvolvedor interessado em ajudar, abaixo você vai aprender a como ajudar no desenvolvimento do ManyBot e
em projeto relacionados.

**Repositórios do ManyBot:**

- **ManyBot**: o código-fonte do bot em si (distribuído sob GPLv3)
    - GitHub: https://github.com/many-bot/manybot
    - Codeberg: https://codeberg.org/many-bot/manybot
    - CGit: https://git.stxerr.dev/manybot.git

- **ManyBot Docs**: toda a documentação do ManyBot
    - GitHub: https://github.com/many-bot/docs
    - Codeberg: https://codeberg.org/many-bot/docs
    - CGit: https://git.stxerr.dev/manybot-docs.git

- **ManyBot Website**: website oficial do projeto
    - GitHub: https://github.com/many-bot/website
    - Codeberg: https://codeberg.org/many-bot/website
    - CGit: https://git.stxerr.dev/manybot-website.git

- **ManyPlug CLI**: gerenciador de plugins de linha de comando (distribuído sob MIT)
    - GitHub: https://github.com/many-bot/manyplug
    - Codeberg: https://codeberg.org/many-bot/manyplug
    - CGit: https://git.stxerr.dev/manyplug.git

- **ManyPlug Repository**: repositório de plugins do ManyBot
    - GitHub: https://github.com/many-bot/manyplug-repo
    - Codeberg: https://codeberg.org/many-bot/manyplug-repo
    - CGit: https://git.stxerr.dev/manyplug-repo.git

### Como contribuir

Depende de como gosta de contribuir. A seguir vamos demonstrar dois métodos: **pull requests** e **git patches**.

#### Pull requests

A forma mais comum nos dias de hoje.

1. Vá para o nosso repositório no [GitHub](https://github.com/many-bot) ou no [Codeberg](https://codeberg.org/many-bot).
2. Faça um fork do repositório 
3. Clone seu fork:
```
git clone https://...
```
4. Crie uma branch:
```
git checkout -b minha-correcao
```
5. Faça as alterações.
6. Commit:
```
git add .
git commit -m "Corrige problema X"
```
7. Envie para seu fork:
```
git push origin minha-correcao
```
8. Abra a página do seu fork no GitHub/Codeberg.
9. Clique em "Criar Pull Request". 
9. Escolha:
- Base: manybot/master
- Compare: seu-fork/minha-correcao
10. Escreva uma descrição e envie.

#### Git patches

Patches são o jeito clássico e ainda muito usado em projetos como o *Linux Kernel Organization*.

1. Clone o repositório:
```
git clone https://...
```
2. Crie uma branch:
```
git checkout -b minha-correcao
```
3. Faça as alterações e commit:
```
git add .
git commit -m "Corrige problema X"
```
4. Gere o patch:
```
git format-patch main --stdout > minha-correcao.patch
```
5. Envie o patch por e-mail para [devel@stxerr.dev](mailto:devel@stxerr.dev).

Você pode fazer isso diretamente pelo terminal com `git send-email`:
```
git send-email --to=devel@stxerr.dev minha-correcao.patch
```

> Para configurar o `git send-email`, consulte a documentação do seu provedor de e-mail ou use um servidor SMTP local como o `msmtp`.

Patches recebidos serão revisados e aplicados com `git am`:
```
git am minha-correcao.patch
```

## Guias e tutoriais

Iremos em breve fazer guias e tutoriais que te ensinam a utilizar e desenvolver com o ManyBot. Caso 
 queira ajudar nisso, abra a branch `draft/guides` do repositório.

# Dúvidas?

Entre em contato via email (manybot@pm.me) ou entre na nossa comunidade do [WhatsApp](https://chat.whatsapp.com/KfOuIwhpQjN8fcZTMHmaGQ) ou [Discord](https://discord.gg/gC7aKChXmA).

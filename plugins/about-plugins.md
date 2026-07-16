# Sobre os plugins

Cada funcionalidade do ManyBot é um plugin independente. Isso permite que desenvolvedores façam
suas próprias funcionalidades para bots sem tocar no core do sistema — o ManyBot sozinho não tem
nenhum comando embutido, tudo o que ele faz vem de plugins instalados e ativos.

## Linguagens suportadas

- **JavaScript** — funciona direto, sem nenhum passo extra
- **TypeScript** — precisa ser compilado antes (o ManyBot só carrega `.js`); `manyplug init --lang ts`
  já monta a estrutura de build pra isso

O pacote de tipos oficial é o `@manybot/types`, publicado no npm — `manyplug init` já adiciona
como `devDependency` do plugin, dando autocomplete pro `ctx` tanto em JS (via JSDoc) quanto em TS.
Veja [como fazer um plugin](/docs/how-to-make-a-plugin) para detalhes.

## Download e instalação

Para baixar e instalar plugins, use a ferramenta oficial do ManyBot pra isso, o
[ManyPlug](https://www.npmjs.com/package/@manybot/manyplug) (`manyplug` ou `mp`).

Para encontrar plugins, use `manyplug search <termo>` ou procure no
[index oficial](https://manybot.stxerr.dev/manyplug/mpindex.json).

## Diretório padrão de plugins

Após instalados, você os verá em `~/.manybot/plugins/` (ou em `C:\Users\SeuUsuário\.manybot\plugins`
no Windows), organizados por autor — ex: `~/.manybot/plugins/synt-xerror/figurinha`.

Mais informações sobre o CLI do ManyPlug na [próxima página dessa seção](/docs/manyplug-cli).

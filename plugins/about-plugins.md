# Sobre os plugins

Cada funcionalidade do ManyBot é um plugin independente. Isso permite com que desenvolvedores façam
 suas próprias funcionalidades para bots sem tocar no core do sistema.

## Download e instalação

Para baixar e instalar plugins, você pode usar a ferramenta oficial do ManyBot para isso, o
 [ManyPlug](https://www.npmjs.com/package/@manybot/manyplug).

Para encontrar plugins, você pode procurar no [index oficial](/plugins) e copiar o comando de instalação
 presente em cada plugin.

## Registro e index

O ManyPlug pega de dois registros diferentes: [manyplug-repo](https://github.com/many-bot/manyplug-repo)
 e [mpindex](/mpindex.json). A diferença é importante entender:

- manyplug-repo: antigo repositório, não é mais atualizado desde o dia 10 de julho de 2026, será
 removido em futuras updates.
 - Como identificar: se você instalou um plugin usando apenas o nome dele (ex. `figurinha`), você
    está usando o manyplug-repo.

- mpindex: index de plugins baseado em repositórios Git independentes (não ficam em um único repositório
 como o manyplug-repo).
 - Como identificar: se você instalou um plugin usando uma chave com autor e nome (e. `synt-xerror/figurinha`),
    você está usando o mpindex.

Prefira sempre instalar plugins do mpindex, visto que o anterior não é mais mantido e é desatualizado.
 Mantemos o `manyplug-repo` apenas por compatibilidade temporária.

## Diretório padrão de plugins

Após instalados, você os verá em `~/.manybot/plugins/` (ou em `C:\Users\SeuUsuário\.manybot\plugins` no Windows).

> Substitua `<plugin>` pelo nome do plugin no qual usou para instalá-lo (ex. `synt-xerror/manymedia`).

Mais informações sobre o CLI do ManyPlug na [próxima página dessa seção](/docs/manyplug-cli)


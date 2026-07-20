# Boas práticas em plugins

Um apanhado de recomendações gerais pra escrever plugins que funcionam bem pra quem instala —
seja num servidor Linux, num Windows, ou num celular Android via Termux.

## Índice

- [Dependências nativas — cuidado, especialmente pensando em Android](#dependencias-nativas-cuidado-especialmente-pensando-em-android)
- [Não bloqueie o event loop](#nao-bloqueie-o-event-loop)
- [Guarde estado onde ele sobrevive a reinício](#guarde-estado-onde-ele-sobrevive-a-reinicio)
- [Não brigue com o guardOptions sem motivo](#nao-brigue-com-o-guardoptions-sem-motivo)
- [Evite loops com fromMe](#evite-loops-com-fromme)
- [Trate `null` como resposta normal, não exceção](#trate-null-como-resposta-normal-não-excecao)
- [Mantenha `manyplug.json` enxuto](#mantenha-manyplugjson-enxuto)
- [Não hardcode número/JID](#nao-hardcode-numerojid)
- [Locale, mesmo que só em português](#locale-mesmo-que-so-em-português)

---

## Dependências nativas — cuidado, especialmente pensando em Android

O ManyBot roda em Linux, Windows **e Android (via Termux)** — e cada plataforma tem sua própria
arquitetura/libc. Pacotes npm que dependem de um **addon nativo compilado** (qualquer coisa que
usa `node-gyp`, `node-pre-gyp`, `prebuild-install`, etc. por baixo) publicam binários
pré-compilados só pras combinações de SO/arquitetura mais comuns — e o Termux (Bionic libc, não
glibc) quase nunca está nessa lista. O `npm install` então cai pra compilar do zero na hora,
o que:

- exige toolchain de compilação instalado no dispositivo (`clang`, `make`, `python`, headers) —
  nada disso vem por padrão;
- é lento e, em celulares mais fracos, pode travar ou estourar memória;
- às vezes **simplesmente não compila**, porque a lib depende de headers de sistema que o Bionic
  não tem.

Isso significa que quem instalar seu plugin num servidor Linux normal nem percebe nada — mas
metade da sua base de usuários no Android vai ter a instalação falhando, e vai parecer que o
`manyplug install` está quebrado, quando na real é a dependência escolhida.

**Antes de adicionar uma dependência ao `manyplug.json`, pergunte: ela tem alternativa pura
JS/WASM?** Alguns exemplos comuns:

| Ao invés de... | Prefira | Por quê |
|---|---|---|
| `sqlite3` / `better-sqlite3` | [`node:sqlite`](https://nodejs.org/api/sqlite.html) (nativo do Node, disponível a partir da 22, estável o suficiente na 24 que o ManyBot já exige) | Não é dependência nenhuma — já vem embutido no runtime, sem compilar nada. |
| `bcrypt` | `bcryptjs` | Implementação pura JS, mesma API na prática. |
| `sharp` / `canvas` (processamento de imagem pesado) | [`jimp`](https://www.npmjs.com/package/jimp) pra casos simples (redimensionar, converter, marca d'água) | Pura JS/WASM, sem addon nativo. Se o plugin realmente precisa de algo no nível do `sharp` (ex: processamento em massa, filtros pesados), documente isso claramente como requisito no README em vez de assumir que "só instala". |
| Bindings nativos de FFmpeg (`fluent-ffmpeg` chamando lib nativa embutida, `ffmpeg-static`, etc.) | Chamar o binário `ffmpeg` do sistema via `externalDependencies` (veja [manyplug.json](/docs/how-to-make-a-plugin/#manyplugjson)) | O binário do sistema já existe como pacote no Termux (`pkg install ffmpeg`) e nas distros — não precisa embutir/compilar nada, só declarar a dependência externa. |

De forma geral: **prefira sempre um binário externo declarado via `externalDependencies`**
(que o usuário instala pelo gerenciador de pacotes do próprio sistema — `apt`, `pkg`, etc.) **a
uma lib npm que embute/compila esse binário por baixo dos panos**. É mais transparente sobre o
que realmente vai ser instalado, e deixa a cargo do gerenciador de pacotes de cada plataforma
resolver a compatibilidade — que é o trabalho dele, não do seu plugin.

Se não tiver alternativa e a dependência nativa for mesmo necessária, pelo menos avisa isso bem
visível no `README.md` do plugin, pra quem for instalar num Android já saber de antemão que pode
dar problema, em vez de descobrir no meio de um `manyplug install` que travou.

---

## Não bloqueie o event loop

Plugins rodam em sequência — **um plugin travado atrasa a resposta de todos os outros** pra
aquela mensagem, porque o kernel despacha um por um. Trabalho pesado feito direto no handler
(parse de arquivo grande, laço síncrono longo, criptografia pesada, redimensionar imagem sem
`await`) trava o processo inteiro, não só seu plugin.

- Downloads e afins: use `ctx.download.enqueue()` (fila serializada, já pensada pra isso) em vez
  de baixar direto no handler.
- Qualquer coisa que legitimamente demore mais que alguns segundos: responda algo tipo
  "processando..." e faça o trabalho pesado depois, mandando o resultado quando terminar via
  `ctx.send.to(chatId)` — não deixe o handler pendurado esperando.
- Lembre que o `timeout` padrão do `guardOptions` desativa o plugin se ele não terminar em 2
  minutos — é um sinal de que a tarefa deveria ser assíncrona/enfileirada, não um limite pra
  tentar espremer.

---

## Guarde estado onde ele sobrevive a reinício

Um `Map()` no escopo do módulo (padrão comum pra "sessões", filas, placares) some inteiro
quando o bot reinicia — o que acontece com mais frequência num celular do que num servidor
(queda de bateria, Android matando o processo, troca de rede). Se o dado importa de verdade
(placar de um jogo, configuração que o usuário ajustou, fila de tarefas pendente), persista com
`ctx.storage`/`ctx.settings` em vez de só memória. Reserve memória em runtime pra estado
genuinamente descartável (timeout de uma sessão de 2 minutos, cache de curtíssimo prazo).

---

## Não brigue com o guardOptions sem motivo

`timeout`, `typing`, `cooldown` e `jitter` existem pra mitigar detecção/banimento da conta do
WhatsApp. Desativar os quatro de uma vez "porque incomoda no teste" é comum durante
desenvolvimento, mas se isso for pro plugin publicado, todo mundo que instalar herda esse risco
maior sem saber. Desative só a opção específica que seu plugin realmente não combina com (ex:
`typing: false` num plugin de serviço que nunca aparece "digitando" porque não responde
diretamente a mensagens) — não o pacote inteiro por padrão.

---

## Evite loops com fromMe

Já é mencionado na [anatomia de um plugin](/docs/01-plugins-basic/), mas vale reforçar: o bot
recebe as próprias mensagens também. Qualquer handler que reage a mensagem "solta" (não só
comando com prefixo) sem checar `ctx.msg.fromMe` corre risco de responder a si mesmo em loop.

---

## Trate `null` como resposta normal, não exceção

Várias APIs devolvem `null` em cenários legítimos e relativamente comuns, não só em erro:
`ctx.msg.getContact()`, `ctx.contacts.get()`, `ctx.msg.getReply()`, `ctx.contacts.getPfpUrl()`.
Assumir que o retorno sempre vem preenchido e acessar propriedade direto (`contact.pushname` sem
checar `contact` antes) é a causa mais comum de plugin quebrando silenciosamente — o kernel
desativa automaticamente um plugin que lança erro não tratado, então um `null` não tratado pode
tirar seu plugin do ar até você reinstalar.

---

## Mantenha `manyplug.json` enxuto

Cada entrada em `dependencies` é instalada via `npm` na máquina de **cada pessoa** que instalar
seu plugin — servidor, desktop ou celular. Dependência a mais não é só peso: é mais uma
superfície de instalação falhando (ver seção de dependências nativas acima), mais tempo de
`manyplug install`, e mais coisa pra manter atualizada. Antes de adicionar uma lib pra resolver
algo pequeno, considere se não dá pra fazer com o que o Node/`ctx.utils` já oferece.

---

## Não hardcode número/JID

Números de admin, grupo de log, etc. — se estiverem fixos no código, seu plugin só funciona pra
você. Leia de `ctx.config.get(...)` (uma chave no `manybot.toml` do usuário) ou de
`ctx.settings`/`ctx.settings.global` (configurável pelo próprio WhatsApp) em vez de escrever o
número direto no `index.js`.

---

## Locale, mesmo que só em português

Não é obrigatório, mas mensagens de erro/resposta soltas direto no código (sem `ctx.i18n`) viram
retrabalho se um dia você quiser suportar outro idioma, ou se o plugin for usado por gente que
prefere `en`. Veja [locale](/docs/how-to-make-a-plugin/#locale).

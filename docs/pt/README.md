# Vue Test Utils

Utilitários de teste de componente para Vue 3.

## Instalação e Uso

- yarn: `yarn add @vue/test-utils --dev`
- npm: `npm install @vue/test-utils --save-dev`

Começar com a [documentação](https://test-utils.vuejs.org/pt).

## É Nosso Objetivo Migrar da Vue 2 + Vue Test Utils v1?

Podemos [consultar o guia de migração](https://test-utils.vuejs.org/pt/migration/). Este continua em andamento. Se encontrarmos um problema ou algo que não funciona, mas que anteriormente funcionava na primeira versão da Vue Test Utils, podemos abrir um problema.

## Documentação

Consultar a [documentação](https://test-utils.vuejs.org/pt/).

## Desenvolvimento

Começamos por executar `pnpm install`. Podemos executar os testes com `pnpm test`. É tudo!

## Colaboração

Todos os ficheiros da documentação podem ser encontrados na `packages/docs`. Contém os ficheiros de markdown em Inglês, enquanto as traduções são armazenadas nas suas sub-pastas `<lang>` correspondentes:

- [`fr`](https://github.com/vuejs/test-utils/tree/main/packages/docs/fr): Tradução francesa.

Além disto, a sub-pasta `.vitepres` contém a configuração e o tema, incluindo a informação i18n.

- `pnpm docs:dev`: Iniciar o servidor de desenvolvimento da documentação
- `pnpm docs:build`: Construir a documentação.

Para adicionar ou manter as traduções, seguimos as [Diretrizes de Tradução do Ecossistema da Vue](https://github.com/vuejs-translations/guidelines/blob/main/README_ECOSYSTEM.md).

- `pnpm docs:translation:status [<lang>]`: Mostrar o estado da tradução para a nossa língua. Se não especificarmos uma língua, será mostrado o estado de todas as línguas.
- `pnpm docs:translation:compare <lang>`: Comparar a documentação com o último ponto de verificação da nossa língua.
- `pnpm docs:translation:update <lang> [<commit>]`: Atualizar o ponto de verificação para a nossa língua. O ponto de verificação será definido pela sequência de caracteres aleatórios de compromisso mais recente. No entanto, também podemos especificar uma sequência de caracteres aleatórios de compromisso manualmente.

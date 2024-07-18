# Instalação {#Installation}

```bash
npm install --save-dev @vue/test-utils

# ou
yarn add --dev @vue/test-utils
```

## Uso {#Usage}

A Vue Test Utils é agnóstica em relação a abstração - podemos usá-la com qualquer executor de teste que quisermos. A maneira mais fácil de experimentá-la é usando a [Jest](https://jestjs.io/), uma ferramenta popular de execução de testes.

Para carregar os ficheiros `.vue` com a Jest, precisaremos da `vue-jest`. A `vue-jest` versão 5 é a que suporta a Vue 3. Esta ainda está na versão alfa, assim como o resto do ecossistema da Vue.js 3, então se encontrarmos um erro de programação, podemos reportá-lo [nesta ligação](https://github.com/vuejs/vue-jest/) e especificar que estamos usando a `vue-jest` versão 5.

Nós podemos instalá-la com `vue-jest@next`. Depois precisamos configurá-la com a opção [`transform`](https://jestjs.io/docs/en/configuration#transform-objectstring-pathtotransformer--pathtotransformer-object) da Jest.

Se não quisermos configurá-la nós próprios, podemos obter um repositório mínimo com tudo configurado [nesta ligação](https://github.com/lmiller1990/vtu-next-demo).

Precisamos continuar a leitura para aprendermos mais sobre a Vue Test Utils.

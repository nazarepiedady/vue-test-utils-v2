# Começar {#Getting-Started}

Nós estamos na Vue Test Utils, a biblioteca oficial de utilitário de teste para Vue.js!

Esta é a documentação da Vue Test Utils versão 2, que tem como alvo a Vue 3.

Em suma:

- [Vue Test Utils 1](https://github.com/vuejs/vue-test-utils/) tem como objetivo a [Vue 2](https://github.com/vuejs/vue/).
- [Vue Test Utils 2](https://github.com/vuejs/test-utils/) tem como objetivo a [Vue 3](https://github.com/vuejs/core/).

## Que é a Vue Test Utils? {#What-is-Vue-Test-Utils-}

A Vue Test Utils (VTU) é um conjunto de funções utilitárias destinadas a simplificar o teste de componentes da Vue.js. Este fornece alguns métodos para montar e interagir com os componentes da Vue duma maneira isolada.

Eis um exemplo:

```js
import { mount } from '@vue/test-utils'

// O componente a testar
const MessageComponent = {
  template: '<p>{{ msg }}</p>',
  props: ['msg']
}

test('displays message', () => {
  const wrapper = mount(MessageComponent, {
    props: {
      msg: 'Hello world'
    }
  })

  // Asserir o texto desenhado do componente
  expect(wrapper.text()).toContain('Hello world')
})
```

A Vue Test Utils é comummente usada com uma ferramenta de execução de teste. As ferramentas de execução de teste populares incluem:

- [Vitest](https://vitest.dev/). Baseada no Terminal, tem uma interface de navegador experimental.
- [Cypress](https://cypress.io/). Baseada no navegador, suporta, Vite, Webpack.
- [Playwright](https://playwright.dev/docs/test-components) (experimental). Baseada no navegador, suporta a Vite.
- [WebdriverIO](https://webdriver.io/docs/component-testing/vue). Baseada no navegador, suporta a Vite, Webpack, suporta vários navegadores.

A Vue Test Utils é uma biblioteca minimalista e sem opinião. Para algo mais rico em funcionalidades, ergonómica e com mais opinião podemos considerar o [Teste de Componente da Cypress](https://docs.cypress.io/guides/component-testing/overview) que tem um ambiente de desenvolvimento de recarga instantânea, ou a [Biblioteca de Teste](https://testing-library.com/docs/vue-testing-library/intro/) que enfatiza os seletores baseados na acessibilidade quando fazemos asserções. Ambas ferramentas usam a Vue Test Utils nos bastidores e expõem a mesma API.

## Que se Segue? {#What-Next-}

Para vermos a Vue Test Utils em ação, [podemos realizar o Curso Intensivo](../guide/essentials/a-crash-course), onde construímos uma aplicação de controlo de tarefas usando a abordagem de testar primeiro.

A documentação está dividida em duas secções principais:

- **Fundamentos**, para cobrir os casos de uso comuns que enfrentaremos quando testarmos os componentes da Vue.
- **Vue Test Utils em Profundidade**, para explorar outras funcionalidades avançadas da biblioteca.

Nós também podemos explorar a [API](../api/) completa.

Alternativamente, se preferirmos aprender através de vídeo, existe [um número de palestras disponíveis nesta ligação](https://www.youtube.com/playlist?list=PLC2LZCNWKL9ahK1IoODqYxKu5aA9T5IOA).

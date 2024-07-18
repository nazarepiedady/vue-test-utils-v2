# Testar a Vuex {#Testing-Vuex}

A Vuex é apenas um detalhe de implementação; não é necessário qualquer tratamento especial para testar componentes que utilizam a Vuex. Dito isto, existem algumas técnicas que podem tornar os nossos testes mais fáceis de ler e escrever. Iremos analisá-las nesta secção.

Este guia pressupõe que estamos familiarizados com a Vuex. A Vuex 4 é a versão que funciona com a Vue.js 3. [Ler a documentação](https://next.vuex.vuejs.org/).

## Um Exemplo Simples {#A-Simple-Example}

Eis um armazém de estados de Vuex simples, e um componente que depende da presença de um armazém de estados da Vuex:

```js
import { createStore } from 'vuex'

const store = createStore({
  state() {
    return {
      count: 0
    }
  },
  mutations: {
    increment(state: any) {
      state.count += 1
    }
  }
})
```

O armazém de estado simplesmente armazena uma contagem, aumentando-o quando a mutação `increment` é confirmada. Este é o componente que testaremos:

```js
const App = {
  template: `
    <div>
      <button @click="increment" />
      Count: {{ count }}
    </div>
  `,
  computed: {
    count() {
      return this.$store.state.count
    }
  },
  methods: {
    increment() {
      this.$store.commit('increment')
    }
  }
}
```

## Testar com um Armazém de Estado de Vuex Verdadeiro {#Testing-with-a-Real-Vuex-Store}

Para testar completamente se este componente e o armazém de estado da Vuex funcionam, clicaremos no `<button>` e asseriremos que a contagem é aumentada. Nas nossas aplicações de Vue, normalmente em `main.js`, instalamos a Vuex desta maneira:

```js
const app = createApp(App)
app.use(store)
```

Isto deve-se ao fato de a Vuex ser uma extensão. As extensões são aplicadas ao chamar `app.use` e passando uma extensão.

A Vue Test Utils também permite-nos instalar extensões, utilizando a opção de montagem `global.plugins`:

```js
import { createStore } from 'vuex'

const store = createStore({
  state() {
    return {
      count: 0
    }
  },
  mutations: {
    increment(state: any) {
      state.count += 1
    }
  }
})

test('vuex', async () => {
  const wrapper = mount(App, {
    global: {
      plugins: [store]
    }
  })

  await wrapper.find('button').trigger('click')

  expect(wrapper.html()).toContain('Count: 1')
})
```

Depois de instalar a extensão, usamos `trigger` para clicar no botão e asserir que a `count` é aumentada. Este tipo de teste, que abrange a interação entre diferentes sistemas (neste caso, a componente e o armazém de estado), é conhecido como teste de integração.

## Testar com um Armazém de Estado Simulado {#Testing-with-a-Mock-Store}

Em contrapartida, um teste unitário pode isolar e testar o componente e o armazém de estado separadamente. Isto pode ser útil se tivermos uma aplicação muito grande com um armazém de estado complexo. Para este caso de uso, podemos simular as partes do armazém de estado em estamos interessados usando a `global.mocks`:

```js
test('vuex using a mock store', async () => {
  const $store = {
    state: {
      count: 25
    },
    commit: jest.fn()
  }

  const wrapper = mount(App, {
    global: {
      mocks: {
        $store
      }
    }
  })

  expect(wrapper.html()).toContain('Count: 25')
  await wrapper.find('button').trigger('click')
  expect($store.commit).toHaveBeenCalled()
})
```

Em vez de usar um armazém de estado de Vuex verdadeiro e instalá-lo através da `global.plugins`, criamos o nosso próprio armazém de estado simulado, implementando apenas as partes da Vuex usadas no componente (neste caso, as funções `state` e `commit`).

Embora possa parecer conveniente testar o armazém de estado isoladamente, observemos que este não nos dará nenhum aviso se quebrarmos o nosso armazém de estado de Vuex. Consideremos cuidadosamente se queremos simular o armazém de estado da Vuex ou utilizar o armazém de estado verdadeiro, e compreender as vantagens e desvantagens.

## Testar a Vuex em Isolamento {#Testing-Vuex-in-Isolation}

Podemos querer testar as nossas mutações ou ações de Vuex em total isolamento, especialmente se forem complexas. Não precisamos da Vue Test Utils para isto, já que um armazém de estado de Vuex é apenas código de JavaScript normal. Eis como podemos testar a mutação `increment` sem a Vue Test Utils:

```js
test('increment mutation', () => {
  const store = createStore({
    state: {
      count: 0
    },
    mutations: {
      increment(state) {
        state.count += 1
      }
    }
  })

  store.commit('increment')

  expect(store.state.count).toBe(1)
})
```

## Predefinir o Estado da Vuex {#Presetting-the-Vuex-State}

Por vezes, pode ser útil ter o armazém de estado da Vuex num estado específico para um teste. Uma técnica útil que podemos utilizar, além da `global.mocks`, é criar uma função que embrulha a `createStore` e receba um argumento para semear o estado inicial. Neste exemplo, estendemos a `increment` para receber um argumento adicional, que será adicionado a `state.count`. Se isto não for fornecido, apenas incrementamos a `state.count` por 1:

```js
const createVuexStore = (initialState) =>
  createStore({
    state: {
      count: 0,
      ...initialState
    },
    mutations: {
      increment(state, value = 1) {
        state.count += value
      }
    }
  })

test('increment mutation without passing a value', () => {
  const store = createVuexStore({ count: 20 })
  store.commit('increment')
  expect(store.state.count).toBe(21)
})

test('increment mutation with a value', () => {
  const store = createVuexStore({ count: -10 })
  store.commit('increment', 15)
  expect(store.state.count).toBe(5)
})
```

Ao criar uma função `createVuexStore` que recebe um estado inicial, podemos facilmente definir o estado inicial. Isto permite-nos testar todos os casos extremos, simplificando, ao mesmo tempo, os nossos testes.

O [Vue Testing Handbook](https://lmiller1990.github.io/vue-testing-handbook/testing-vuex.html) tem mais exemplos para testar a Vuex. Nota: os exemplos referem-se a Vue.js 2 e a Vue Test Utils 1. As ideias e os conceitos são os mesmos, e o **Vue Testing Handbook** será atualizado para a Vue.js 3 e a Vue Test Utils 2 num futuro próximo.

## Testar com o uso da API de Composição {#Testing-using-the-Composition-API}

A Vuex é acessada através duma função `useStore` quando se utiliza a interface de programação de aplicação de composição. [Ler mais sobre isto neste artigo](https://next.vuex.vuejs.org/guide/composition-api.html).

A `useStore` pode ser usada com uma chave de injeção opcional e única, conforme discutido na [documentação da Vuex](https://next.vuex.vuejs.org/guide/typescript-support.html#typing-usestore-composition-function).

Tem o seguinte aspeto:

```js
import { createStore } from 'vuex'
import { createApp } from 'vue'

// criar um símbolo globalmente único
// para a chave de injeção.
const key = Symbol()

const App = {
  setup () {
    // utilizar uma chave única para acessar
    // o armazém de estado.
    const store = useStore(key)
  }
}

const store = createStore({ /* ... */ })
const app = createApp({ /* ... */ })

// especificar a chave como segundo argumento
// ao chamar `app.use(store)`.
app.use(store, key)
```

Para evitar a repetição da passagem do parâmetro chave sempre que a `useStore` for usada, a documentação da Vuex recomenda extrair esta lógica numa função auxiliar e reutilizar esta função em vez da função predefinida `useStore`. [Ler mais sobre isto neste artigo](https://next.vuex.vuejs.org/guide/typescript-support.html#typing-usestore-composition-function). A abordagem que fornece um armazém de estado usando a Vue Test Utils depende da maneira como a função `useStore` é usada no componente.

### Testar Componentes que Utilizam `useStore` sem uma Chave de Injeção {#Testing-Components-that-Utilize-useStore-without-an-Injection-Key}

Sem uma chave de injeção, os dados do armazém de estado podem simplesmente ser injetados no componente através da opção de montagem global `provide`. O nome do armazém de estado injetado deve ser o mesmo que o componente, por exemplo, “store”.

#### Exemplo de Fornecimento da `useStore` sem chave {#Example-for-providing-the-unkeyed-useStore}

```js
import { createStore } from 'vuex'

const store = createStore({
  // ...
})

const wrapper = mount(App, {
  global: {
    provide: {
      store: store
    },
  },
})
```

### Testar Componentes que Utilizam `useStore` com uma Chave de Injeção {#Testing-Components-that-Utilize-useStore-with-an-Injection-Key}

Ao utilizar o armazém de estado com uma chave de injeção, a abordagem anterior não funcionará. A instância do armazém de estado não será retornada por `useStore`. Para acessar o armazém de estado correto, é necessário fornecer o identificador.

Tem de ser a chave exata passada para `useStore` na função `setup` do componente ou para `useStore` na função auxiliar personalizada. Uma vez que os símbolos de JavaScript são únicos e não podem ser recriados, é melhor exportar a chave do armazém de estado verdadeiro.

Podemos utilizar `global.provide` com a chave correta para injetar o armazém de estado, ou `global.plugins` para instalar o armazém de estado e especificar a chave:

#### Fornecer a `useStore` com Chave usando a `global.provide` {#Providing-the-Keyed-useStore-using-global-provide}

```js
// store.js
export const key = Symbol()
```

```js
// app.spec.js
import { createStore } from 'vuex'
import { key } from './store'

const store = createStore({ /* ... */ })

const wrapper = mount(App, {
  global: {
    provide: {
      [key]: store
    },
  },
})
```

#### Fornecer a `useStore` com Chave usando a `global.plugins` {#Providing-the-Keyed-useStore-using-global-plugins}

```js
// store.js
export const key = Symbol()
```

```js
// app.spec.js
import { createStore } from 'vuex'
import { key } from './store'

const store = createStore({ /* ... */ })

const wrapper = mount(App, {
  global: {
    // para passar opções às extensões,
    // usamos a sintaxe de vetor.
    plugins: [[store, key]]
  },
})
```



## Conclusão {#Conclusion}

- Usar `global.plugins` instalar a Vuex como uma extensão.
- Usar `global.mocks` para simular um objeto global, como a Vuex, para casos de uso avançados.
- Considerar testar mutações e ações complexas da Vuex isoladamente.
- Embrulhar `createStore` com uma função que recebe um argumento para configurar cenários de teste específicos.

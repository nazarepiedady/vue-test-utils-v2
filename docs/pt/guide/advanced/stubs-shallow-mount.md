# Esboços e Montagens Superficiais {#Stubs-and-Shallow-Mount}

A Vue Test Utils fornece alguns recursos avançados para _esboçar_ componentes e diretivas. Um _esboço_ é onde substituímos uma implementação existente de um componente ou diretiva personalizada por uma implementação fictícia que não faz nada, o que pode simplificar um teste complexo. Veremos um exemplo.

## Esboçar um Único Componente Filho {#Stubbing-a-single-child-component}

Um exemplo comum é quando gostaríamos de testar algo num componente que aparece muito alto na hierarquia de componentes.

Neste exemplo, temos um `<App>` desenha uma mensagem, bem como um componente `FetchDataFromApi` que faz uma chamada à interface de programação de aplicação e desenha o seu resultado:

```js
const FetchDataFromApi = {
  name: 'FetchDataFromApi',
  template: `
    <div>{{ result }}</div>
  `,
  async mounted() {
    const res = await axios.get('/api/info')
    this.result = res.data
  },
  data() {
    return {
      result: ''
    }
  }
}

const App = {
  components: {
    FetchDataFromApi
  },
  template: `
    <h1>Welcome to Vue.js 3</h1>
    <fetch-data-from-api />
  `
}
```

Não queremos fazer uma chamada à interface de programação de aplicação neste teste em particular, apenas queremos asserir que a mensagem é desenhada. Neste caso, poderíamos utilizar a `stubs`, que aparece na opção de montagem `global`:

```js
test('stubs component with custom template', () => {
  const wrapper = mount(App, {
    global: {
      stubs: {
        FetchDataFromApi: {
          template: '<span />'
        }
      }
    }
  })

  console.log(wrapper.html())
  // <h1>Welcome to Vue.js 3</h1><span></span>

  expect(wrapper.html()).toContain('Welcome to Vue.js 3')
})
```

Notemos que o modelo de marcação mostra `<span></span>` onde estava o `<fetch-data-from-api />`? Nós o substituímos por um esboço — neste caso, fornecemos a nossa própria implementação passando um `template`.

Também podemos obter um esboço predefinido, em vez fornecermos no nosso próprio esboço:

```js
test('stubs component', () => {
  const wrapper = mount(App, {
    global: {
      stubs: {
        FetchDataFromApi: true
      }
    }
  })

  console.log(wrapper.html())
  /*
    <h1>Welcome to Vue.js 3</h1>
    <fetch-data-from-api-stub></fetch-data-from-api-stub>
  */

  expect(wrapper.html()).toContain('Welcome to Vue.js 3')
})
```

Isto esboçará _todos_ os componentes `<FetchDataFromApi />` da árvore de apresentação, independentemente do nível em que estes aparecem. É por isto que está na opção de montagem `global`.

:::tip DICA
Para esboçar, podemos utilizar a chave em `components` ou o nome do nosso componente. Se ambos forem fornecidos em `global.stubs`, a chave será usada primeiro.
:::

## Esboçar Todos os Componentes Filhos {#Stubbing-all-children-components}

Por vezes, podemos querer esboçar _todos_ os componentes personalizados. Por exemplo, podemos ter um componente como este:

```js
const ComplexComponent = {
  components: { ComplexA, ComplexB, ComplexC },
  template: `
    <h1>Welcome to Vue.js 3</h1>
    <ComplexA />
    <ComplexB />
    <ComplexC />
  `
}
```

Imaginemos que cada um dos `<Complex>` faz algo complicado, e só estamos interessados em testar se o `<h1>` desenha a saudação correta. Poderíamos fazer algo como:

```js
const wrapper = mount(ComplexComponent, {
  global: {
    stubs: {
      ComplexA: true,
      ComplexB: true,
      ComplexC: true
    }
  }
})
```

Mas isto é muito complicado. A VTU tem uma opção de montagem `shallow` que esboçará automaticamente todos os componentes filhos:

```js {3}
test('shallow stubs out all child components', () => {
  const wrapper = mount(ComplexComponent, {
    shallow: true
  })

  console.log(wrapper.html())
  /*
    <h1>Welcome to Vue.js 3</h1>
    <complex-a-stub></complex-a-stub>
    <complex-b-stub></complex-b-stub>
    <complex-c-stub></complex-c-stub>
  */
})
```

:::tip DICA
Se usamos a VTU V1, talvez nos lembremos disto como `shallowMount`. Este método também ainda está disponível — é o mesmo que escrever `shallow: true`.
:::

## Esboçar Todos os Componentes Filhos com Exceções {#Stubbing-all-children-components-with-exceptions}

Por vezes, queremos esboçar _todos_ os componentes personalizados, _exceto_ um específico. Consideremos um exemplo:

```js
const ComplexA = {
  template: '<h2>Hello from real component!</h2>'
}

const ComplexComponent = {
  components: { ComplexA, ComplexB, ComplexC },
  template: `
    <h1>Welcome to Vue.js 3</h1>
    <ComplexA />
    <ComplexB />
    <ComplexC />
  `
}
```

Ao utilizar a opção de montagem `shallow`, todos os componentes filhos serão automaticamente esboçados. Se quisermos optar explicitamente por não fazer o esboço de um componente específico, podemos fornecer o seu nome em `stubs` com o valor definido como `false`:

```js {3}
test('shallow allows opt-out of stubbing specific component', () => {
  const wrapper = mount(ComplexComponent, {
    shallow: true,
    global: {
      stubs: { ComplexA: false }
    }
  })

  console.log(wrapper.html())
  /*
    <h1>Welcome to Vue.js 3</h1>
    <h2>Hello from real component!</h2>
    <complex-b-stub></complex-b-stub>
    <complex-c-stub></complex-c-stub>
  */
})
```

## Esboçar um Componente Assíncrono {#Stubbing-an-async-component}

No caso de quisermos esboçar um componente assíncrono, existem dois comportamentos. Por exemplo, podemos ter componentes como este:

```js
// AsyncComponent.js
export default defineComponent({
  name: 'AsyncComponent',
  template: '<span>AsyncComponent</span>'
})

// App.js
const App = defineComponent({
  components: {
    MyComponent: defineAsyncComponent(() => import('./AsyncComponent'))
  },
  template: '<MyComponent/>'
})
```

O primeiro comportamento é usar a chave definida no nosso componente, que carrega o componente assíncrono. Neste exemplo, utilizámos a chave `MyComponent`. Não é necessário utilizar `async/await` no caso de teste, porque o componente foi esboçado antes de ser resolvido:

```js
test('stubs async component without resolving', () => {
  const wrapper = mount(App, {
    global: {
      stubs: {
        MyComponent: true
      }
    }
  })

  expect(wrapper.html()).toBe('<my-component-stub></my-component-stub>')
})
```

O segundo comportamento consiste em utilizar o nome do componente assíncrono. Neste exemplo, utilizámos o nome `'AsyncComponent'`. Agora é necessário usar `async/await`, porque o componente assíncrono precisa ser resolvido e, em seguida, pode ser esboçado pelo nome definido no componente assíncrono.

**Nos certificamos de que definimos um nome no nosso componente assíncrono!**

```js
test('stubs async component with resolving', async () => {
  const wrapper = mount(App, {
    global: {
      stubs: {
        AsyncComponent: true
      }
    }
  })

  await flushPromises()

  expect(wrapper.html()).toBe('<async-component-stub></async-component-stub>')
})
```

## Esboçar uma Diretiva {#Stubbing-a-directive}

Por vezes, as diretivas fazem coisas muito complexas, como executar muita manipulação do modelo de objeto do documento, o que pode resultar em erros nos nossos testes (devido ao fato de a `jsdom` não se assemelhar a todo o comportamento do modelo de objeto do documento). Um exemplo comum são as diretivas de dicas de ferramentas de várias bibliotecas, que normalmente dependem muito da medição da posição ou tamanho dos nós do modelo de objeto do documento.

Neste exemplo, temos outro `<App>` que desenha uma mensagem com dica de utilização de ferramenta:

```js
// diretiva de dica de utilização de ferramenta
// declarada algures, com o nome `Tooltip`

const App = {
  directives: {
    Tooltip
  },
  template: '<h1 v-tooltip title="Welcome tooltip">Welcome to Vue.js 3</h1>'
}
```

Não queremos que o código da diretiva `Tooltip` seja executado neste teste, apenas queremos asserir que a mensagem é desenhada. Neste caso, poderíamos utilizar a `stubs`, que aparece na opção de montagem `global` passando `vTooltip`:

```js
test('stubs component with custom template', () => {
  const wrapper = mount(App, {
    global: {
      stubs: {
        vTooltip: true
      }
    }
  })

  console.log(wrapper.html())
  // <h1>Welcome to Vue.js 3</h1>

  expect(wrapper.html()).toContain('Welcome to Vue.js 3')
})
```

:::tip DICA
A utilização do esquema de nomenclatura `vCustomDirective` para diferenciar entre componentes e diretivas é inspirada na [mesma abordagem](https://pt.vuejs.org/api/sfc-script-setup#using-custom-directives) usada em `<script setup>`.
:::

Por vezes, precisamos duma parte da funcionalidade da diretiva (normalmente porque algum código depende desta). Assumiremos que a nossa diretiva adiciona a classe de folha de estilo em cascata `with-tooltip` quando executada, e que este é um comportamento importante para o nosso código. Neste caso, podemos trocar `true` pela nossa implementação de simulação da diretiva:

```js
test('stubs component with custom template', () => {
  const wrapper = mount(App, {
    global: {
      stubs: {
        vTooltip: {
          beforeMount(el: Element) {
            console.log('directive called')
            el.classList.add('with-tooltip')
          }
        }
      }
    }
  })

  // 'directive called' registada na consola

  console.log(wrapper.html())
  // <h1 class="with-tooltip">Welcome to Vue.js 3</h1>

  expect(wrapper.classes('with-tooltip')).toBe(true)
})
```

Acabamos de trocar a nossa implementação de diretiva pela nossa própria!

:::warning AVISO
O esboço de diretivas não funcionará em componentes funcionais ou `<script setup>` devido à falta do nome da diretiva dentro da função [`withDirectives`](https://pt.vuejs.org/api/render-function#withdirectives). Temos de considerar a simulação de um módulo de diretiva através da nossa abstração de testes se precisarmos de simular uma diretiva utilizada num componente funcional. Consultar o endereço https://github.com/vuejs/core/issues/6887 pela proposta de desbloqueio desta funcionalidade.
:::

## Ranhuras Predefinidas e `shallow` {#Default-Slots-and-shallow}

Uma vez que a `shallow` esboça todo o conteúdo dos componentes, nenhum `<slot>` será desenhado ao utilizar a `shallow`. Embora isto não seja um problema na maioria dos casos, existem alguns cenários em que isto não é ideal:

```js
const CustomButton = {
  template: `
    <button>
      <slot />
    </button>
  `
}
```

E podemos utilizá-lo desta maneira:

```js
const App = {
  props: ['authenticated'],
  components: { CustomButton },
  template: `
    <custom-button>
      <div v-if="authenticated">Log out</div>
      <div v-else>Log in</div>
    </custom-button>
  `
}
```

Se usarmos a `shallow`, a ranhura não será desenhada, uma vez que a função de interpretação em `<custom-button />` é esboçada. Isto significa que não poderemos verificar se o texto correto é desenhado!

Para este caso de uso, podemos utilizar `config.renderStubDefaultSlot`, que desenhará o conteúdo da ranhura predefinida, mesmo quando utilizamos a `shallow`:

```js {1,4,8}
import { config, mount } from '@vue/test-utils'

beforeAll(() => {
  config.global.renderStubDefaultSlot = true
})

afterAll(() => {
  config.global.renderStubDefaultSlot = false
})

test('shallow with stubs', () => {
  const wrapper = mount(AnotherApp, {
    props: {
      authenticated: true
    },
    shallow: true
  })

  expect(wrapper.html()).toContain('Log out')
})
```

Uma vez que este comportamento é global, e não numa base `mount` por `mount`, temos de lembrar-nos de o ativar ou desativar antes e depois de cada teste.

:::tip DICA
Também podemos ativar isto globalmente ao importar `config` no nosso ficheiro de configuração, e definir `renderStubDefaultSlot` como `true`. Infelizmente, devida a limitações técnicas, este comportamento não é estendido a outras ranhuras para além da ranhura predefinida.
:::

## `mount`, `shallow`, e `stubs`: qual deles e quando? {#mount-shallow-and-stubs-which-one-and-when-}

Como regra geral, **quanto mais os nossos testes assemelharem-se à maneira como o nosso software é utilizado**, mais confiança podem dar-nos.

Os testes que usam `mount` desenharão toda a hierarquia de componentes, o que é mais próximo do que o utilizador experimentará num navegador verdadeiro.

Por outro lado, os testes usando a `shallow` são focados num componente específico. A `shallow` pode ser útil para testar componentes avançados em completo isolamento. Se tivermos apenas um ou dois componentes irrelevantes para os nossos testes, consideremos utilizar a `mount` em combinação com a `stubs` ao invés de `shallow`. Quanto mais esboçarmos, menos parecido com a produção o nosso teste se torna.

Precisamos ter em mente que, independentemente de fazermos uma montagem completa ou uma reprodução superficial, bons testes focam-se em entradas (`props` e interação com o utilizador, como com a `trigger`) e saídas (os elementos do modelo de objeto do documento desenhados e eventos), não em detalhes de implementação.

Por isto, independentemente do método de montagem que escolhermos, é bom ter em mente estas diretrizes.

## Conclusão {#Conclusion}

- Usar a `global.stubs` para substituir um componente ou diretiva por um componente fictício para simplificar os nossos testes.
- Usar `shallow: true` (ou `shallowMount`) para esboçar todos os componentes filhos
- Usar a `global.renderStubDefaultSlot` para desenhar o `<slot>` predefinido para um componente esboçado.

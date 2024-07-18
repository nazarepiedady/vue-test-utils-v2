# Comportamento Assíncrono {#Asynchronous-Behavior}

Devemos ter notado que algumas outras partes do guia utilizam `await` ao chamar alguns métodos no `wrapper`, como `trigger` e `setValue`. O que é isso?

Sabemos que a Vue atualiza de maneira reativa: quando alteramos um valor, o modelo de objeto do documento é automaticamente atualizado para refletir o valor mais recente. [A Vue faz estas atualizações de maneira assíncrona](https://v3.vuejs.org/guide/change-detection.html#async-update-queue). Em contraste, um executor de testes como Jest é executado _de maneira síncrona_. Isto pode causar alguns resultados surpreendentes nos testes.

Veremos algumas estratégias para garantir que a Vue está a atualizar o modelo de objeto do documento como esperado quando executamos os nossos testes.

## Um Simples Exemplo — Atualizar com a `trigger` {#A-Simple-Example-Updating-with-trigger}

Reutilizaremos o componente `<Counter>` de [manipulação de evento](../essentials/event-handling) com uma alteração; agora desenhamos o `count` no `template`:

```js
const Counter = {
  template: `
    <p>Count: {{ count }}</p>
    <button @click="handleClick">Increment</button>
  `,
  data() {
    return {
      count: 0
    }
  },
  methods: {
    handleClick() {
      this.count += 1
    }
  }
}
```

Escreveremos um teste para verificar se `count` está a aumentar:

```js
test('increments by 1', () => {
  const wrapper = mount(Counter)

  wrapper.find('button').trigger('click')

  expect(wrapper.html()).toContain('Count: 1')
})
```

Surpreendentemente, isto falha! A razão é que, embora `count` seja aumentado, a Vue não atualizará o modelo de objeto do documento até o próximo tique ciclo de eventos. Por este motivo, a asserção (`expect()...`) será chamada antes da Vue atualizar o modelo de objeto do documento.

:::tip DICA
Se quisermos saber mais sobre este comportamento central da JavaScript, podemos ler sobre o [Ciclo de Evento e suas microtarefas e microtarefas](https://javascript.info/event-loop#macrotasks-and-microtasks).
:::

Deixando de lado os detalhes de implementação, como podemos resolver isso? a Vue fornece uma maneira de esperarmos até que o modelo de objeto do documento seja atualizado: `nextTick`.

```js {1,7}
import { nextTick } from 'vue'

test('increments by 1', async () => {
  const wrapper = mount(Counter)

  wrapper.find('button').trigger('click')
  await nextTick()

  expect(wrapper.html()).toContain('Count: 1')
})
```

Agora o teste passará porque garantimos que o próximo “tique” foi executado e o modelo de objeto do documento foi atualizado antes da execução da asserção.

Como a `await nextTick()` é comum, a Vue Test Utils fornece um atalho. Os métodos que fazem com que o modelo de objeto do documento seja atualizado, tais como `trigger` e `setValue` retornam `nextTick()`, por isto podemos simplesmente `await` estes diretamente:

```js {4}
test('increments by 1', async () => {
  const wrapper = mount(Counter)

  await wrapper.find('button').trigger('click')

  expect(wrapper.html()).toContain('Count: 1')
})
```

## Resolver Outros Comportamentos Assíncronos {#Resolving-Other-Asynchronous-Behavior}

A `nextTick` é útil para garantir que alguma alteração nos dados reativos seja refletida no modelo de objeto do documento antes de continuar o teste. No entanto, às vezes podemos querer garantir que outros comportamentos assíncronos não relacionados a Vue também sejam concluídos.

Um exemplo comum é uma função que retorna uma `Promise`. Talvez tenhamos simulado o nosso cliente de protocolo de hipertexto da `axios` usando a `jest.mock`:

```js
jest.spyOn(axios, 'get').mockResolvedValue({ data: 'some mocked data!' })
```

Neste caso, a Vue não tem conhecimento da promessa não resolvida, então chamar a `nextTick` não funcionará — a nossa asserção pode ser executada antes de ser resolvida. Para cenários como este, a Vue Test Utils expõe a [`flushPromises`](../../api/#flushPromises), que faz com que todas as promessas pendentes sejam resolvidas imediatamente.

Veremos um exemplo:

```js{1,12}
import { flushPromises } from '@vue/test-utils'
import axios from 'axios'

jest.spyOn(axios, 'get').mockResolvedValue({ data: 'some mocked data!' })

test('uses a mocked axios HTTP client and flushPromises', async () => {
  // algum componente que faz um protocolo de hipertexto chamada
  // em `created` usando `axios`
  const wrapper = mount(AxiosComponent)

  // a promessa da `axios` é resolvida imediatamente
  await flushPromises()

  // depois da linha acima, o pedido da `axios` foi resolvido
  // com os dados simulados.
})
```

:::tip DICA
Se quisermos saber mais sobre como testar pedidos em componentes, precisamos de consultar o guia [Fazer Requisições de Protocolo de Transferência de Hipertexto](http-requests.md).
:::

## Testar a `setup` assíncrona {#Testing-asynchronous-setup}

Se o componente que queremos testar utiliza uma `setup` assíncrona, então devemos montar o componente num componente `Suspense` (como fazemos quando o utilizamos na nossa aplicação).

Por exemplo, este componente `Async`:

```js
const Async = defineComponent({
  async setup() {
    // esperar algo
  }
})
```

deve ser testado da seguinte maneira:

```js
test('Async component', async () => {
  const TestComponent = defineComponent({
    components: { Async },
    template: '<Suspense><Async/></Suspense>'
  })

  const wrapper = mount(TestComponent)
  await flushPromises();
  // ...
})
```

Nota: para a testar a instância subjacente `vm` dos nossos componentes `Async`, utilizamos o valor de retorna de `wrapper.findComponent(Async)`. Uma vez que um novo componente é definido e montado neste cenário, o embrulhador retornado por `mount(TestComponent)` contém o seu próprio `vm` (vazio).

## Conclusão {#Conclusion}

- A Vue atualiza o modelo de objeto do documento de maneira assíncrona; em vez disto, o executor de testes executa o código de maneira assíncrona.
- Usar `await nextTick()` para garantir que o modelo de objeto do documento foi atualizado antes do teste continuar.
- As funções que podem atualizar o modelo de objeto do documento (como `trigger` e `setValue`) retornam `nextTick`, então precisamos `await` estas.
- Usar `flushPromises` da Vue Test Utils para resolver quaisquer promessas não resolvidas de dependências que não sejam da Vue (como requisições de interface de programação de aplicação).
- Usar `Suspense` para testar componentes com uma `setup` assíncrono numa função de teste assíncrona.

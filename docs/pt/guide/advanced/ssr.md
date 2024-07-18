# Testar a Interpretação do Lado do Servidor {#Testing-Server-side-Rendering}

A Vue Test Utils fornece `renderToString` para testar aplicações de Vue que usam a interpretação do lado do servidor (SSR). Este guia guiar-lo-á através do processo de teste duma aplicação de Vue que usa a interpretação do lado do servidor.

## `renderToString` {#renderToString}

A `renderToString` é uma função que desenha um componente de Vue a uma sequência de caracteres. É uma função assíncrona que retorna uma promessa e aceita os mesmos parâmetros que `mount` ou `shallowMount`.

Consideremos um componente simples que usa uma função gatilho `onServerPrefetch`:

```ts
function fakeFetch(text: string) {
  return Promise.resolve(text)
}

const Component = defineComponent({
  template: '<div>{{ text }}</div>',
  setup() {
    const text = ref<string | null>(null)

    onServerPrefetch(async () => {
      text.value = await fakeFetch('onServerPrefetch')
    })

    return { text }
  }
})
```

Podemos escrever um teste para este componente usando `renderToString`:

```ts
import { renderToString } from '@vue/test-utils'

it('renders the value returned by onServerPrefetch', async () => {
  const contents = await renderToString(Component)
  expect(contents).toBe('<div>onServerPrefetch</div>')
})
```

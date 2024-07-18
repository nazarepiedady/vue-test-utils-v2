# Escrever Componentes Fáceis de Testar {#Write-components-that-are-easy-to-test}

A Vue Test Utils ajuda-nos a escrever testes para os componentes da Vue. No entanto, não existe muito o que possa ser feito pela VTU.

Segue-se uma lista de sugestões para escrever código que seja mais fácil de testar, e para escrever testes que sejam significativos e simples de manter.

A lista seguida fornece orientações gerais e pode ser útil em cenários comuns.

## Não testar detalhes de implementação {#Do-not-test-implementation-details}

Pensemos em termos de entradas e saídas a partir da perspetiva do utilizador. Grosso modo, isto é tudo o que devemos ter em conta quando escrevemos um teste para um componente de Vue:

| **Entradas**    | Exemplos                                                  |
| --------------- | --------------------------------------------------------- |
| Interações      | Clicar, escrever…, qualquer interação “humana”            |
| Propriedades    | Os argumentos que um componente recebe                    |
| Fluxos de dados | Dados recebidos de chamadas de API, subscrições de dados… |

| **Saídas**         | Exemplos                                         |
| ------------------ | ------------------------------------------------ |
| Elementos do DOM   | Qualquer nó _observável_ desenhado no documento  |
| Eventos            | Eventos emitidos (usando `$emit`)                |
| Efeitos Colaterais | Tais como `console.log` ou chamadas de API       |

**Tudo o resto são detalhes de implementação**.

Notemos que esta lista não inclui elementos como métodos internos, estados intermédios ou mesmo dados.

A regra geral é que um **um teste não deve falhar durante ao refazer**, ou seja, quando alteramos a sua implementação interna sem alterar o seu comportamento. Se isto acontecer, o teste pode basear-se nos detalhes de implementação.

Por exemplo, suponhamos um componente básico `Counter` que possui um botão para incrementar um contador:

```vue
<template>
  <p class="paragraph">Times clicked: {{ count }}</p>
  <button @click="increment">increment</button>
</template>

<script>
export default {
  data() {
    return { count: 0 }
  },
  methods: {
    increment() {
      this.count++
    }
  }
}
</script>
```

Poderíamos escrever o seguinte teste:

```js
import { mount } from '@vue/test-utils'
import Counter from './Counter.vue'

test('counter text updates', async () => {
  const wrapper = mount(Counter)
  const paragraph = wrapper.find('.paragraph')

  expect(paragraph.text()).toBe('Times clicked: 0')

  await wrapper.setData({ count: 2 })

  expect(paragraph.text()).toBe('Times clicked: 2')
})
```

Notemos como atualizamos neste exemplo os seus dados internos e também nos baseamos em detalhes (do ponto de vista do utilizador) como as classes de CSS.

:::tip DICA
Notemos que alterar os dados ou o nome da classe de CSS faria com que o teste falhasse. No entanto, o componente continuaria a funcionar como esperado. Isto é conhecido como um **falso positivo**.
:::

Em vez disto, o teste seguinte tenta manter as entradas e saídas listadas acima:

```js
import { mount } from '@vue/test-utils'

test('text updates on clicking', async () => {
  const wrapper = mount(Counter)

  expect(wrapper.text()).toContain('Times clicked: 0')

  const button = wrapper.find('button')
  await button.trigger('click')
  await button.trigger('click')

  expect(wrapper.text()).toContain('Times clicked: 2')
})
```

Bibliotecas como a [Vue Testing Library](https://github.com/testing-library/vue-testing-library/) são construídas com base nestes princípios. Se estivermos interessados nesta abordagem, devemos consultá-la.

## Construir componentes menores e mais simples {#Build-smaller-simpler-components}

Uma regra geral é que se um componente fizer menos coisas, será mais fácil de testar.

A criação de componentes menores torna-nos mais fáceis de compor e mais fáceis entender. Segue-se uma lista de sugestões para tornar os componentes mais simples.

### Extrair chamadas de interface de programação de aplicação {#Extract-API-calls}

Normalmente, realizaremos várias requisições com o protocolo de transferência de hipertexto ao longo da nossa aplicação. De uma perspetiva de teste, as requisições do protocolo de transferência de hipertexto fornecem entradas para o componente, e um componente também pode enviar requisições de protocolo de transferência de hipertexto.

:::tip DICA
Precisamos consultar o guia [Fazer requisições com o protocolo de transferência de hipertexto](../advanced/http-requests) se não estivermos familiarizados com o teste de chamadas de interface de programação de aplicação.
:::

### Extrair métodos complexos {#Extract-complex-methods}

Por vezes, um componente pode apresentar um método complexo, realizar cálculos pesados ou usar várias dependências.

A sugestão nesta situação é **extrair este método e importá-lo ao componente**. Desta maneira, podemos testar o método isoladamente usando a Jest ou qualquer outra executor de testes.

Isto tem a vantagem adicional de se obter um componente que é mais fácil de entender porque a lógica complexa é encapsulada noutro ficheiro.

Além disto, se o método complexo for difícil de configurar ou lento, podemos querer simular o método para tornar o teste mais simples e rápido. Exemplos sobre como [fazer requisições de protocolo de transferência de hipertexto]() são um bom exemplo — a [`axios`](https://axios-http.com/docs/intro) é uma biblioteca bastante complexa!

## Escrever os testes antes de escrever o componente {#Write-tests-before-writing-the-component}

Não podemos escrever código não se pode testar se escrevermos testes de antemão!

O nosso [Curso Intensivo](../essentials/a-crash-course) oferece um exemplo de como escrever testes antes do código conduz a componentes testáveis. Também ajuda-nos a detetar e testar casos extremos.

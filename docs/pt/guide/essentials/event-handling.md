# Manipulação de Evento {#Event-Handling}

Os componente da Vue interagem uns com os outros através de propriedades e emitindo eventos chamando `$emit`. Neste guia, veremos como verificar se os eventos são emitidos corretamente usando a função `emitted()`.

Este artigo também está disponível como um [vídeo curto](https://www.youtube.com/watch?v=U_j-nDur4oU&list=PLC2LZCNWKL9ahK1IoODqYxKu5aA9T5IOA&index=14).

## O componente  `Counter` {#The-Counter-component}

Eis um componente `<Counter>` simples. Inclui um botão que, quando clicado, incrementa uma variável de contagem interna e emite o seu valor:

```js
const Counter = {
  template: '<button @click="handleClick">Increment</button>',
  data() {
    return {
      count: 0
    }
  },
  methods: {
    handleClick() {
      this.count += 1
      this.$emit('increment', this.count)
    }
  }
}
```

Para testar completamente este componente, devemos verificar se um evento `increment` com o último valor `count` é emitido.

## Asserir os eventos emitidos {#Asserting-the-emitted-events}

Para o fazer, basear-nos-emos no método `emitted()`. **Retorna um objeto com todos os eventos que o componente emitiu**, e os seus argumentos num vetor. Veremos como funciona:

```js
test('emits an event when clicked', () => {
  const wrapper = mount(Counter)

  wrapper.find('button').trigger('click')
  wrapper.find('button').trigger('click')

  expect(wrapper.emitted()).toHaveProperty('increment')
})
```

> Se ainda não vimos `trigger` antes, não precisamos preocupar-nos. É usado para simular a interação do utilizador. Podemos obter mais informações no artigo sobre [Formulários](./forms).

A primeira coisa a notar é que `emitted()` retorna um objeto, onde cada chave corresponde a um evento emitido. Neste caso, `increment`.

Este teste deve passar. Nos certificámos de que emitimos um evento com o nome apropriado.

## Asserir os argumentos do evento {#Asserting-the-arguments-of-the-event}

Isto é bom — mas podemos fazer melhor! Precisamos verificar se emitimos os argumentos corretos quando `this.$emit('increment', this.count)` é chamado.

O nosso próximo passo é asserir que o evento contém o valor `count`. O fazemos passando um argumento para `emitted()`:

```js {9}
test('emits an event with count when clicked', () => {
  const wrapper = mount(Counter)

  wrapper.find('button').trigger('click')
  wrapper.find('button').trigger('click')

  // `emitted()` aceita um argumento.
  // Retorna um vetor com todas as
  // ocorrências de `this.$emit('increment')`
  const incrementEvent = wrapper.emitted('increment')

  // "Clicamos" duas vezes, então o vetor de
  // `increment` deve ter dois valores.
  expect(incrementEvent).toHaveLength(2)

  // Asserir o resultado do primeiro clique.
  // Notar que o valor é um vetor.
  expect(incrementEvent[0]).toEqual([1])

  // Depois, o resultado do segundo.
  expect(incrementEvent[1]).toEqual([2])
})
```

Recapitularemos e analisaremos a saída de `emitted()`. Cada uma destas chaves contém os diferentes valores emitidos durante o teste:

```js
// console.log(wrapper.emitted('increment'))
;[
  [1], // a primeira vez que é chamada, `count` é 1
  [2] // a segunda vez que é chamada, `count` é 2
]
```

## Asserir eventos complexos {#Asserting-complex-events}

Imaginemos que agora o nosso componente `<Counter>` precisa emitir um objeto com informação adicional. Por exemplo, precisamos dizer a qualquer componente pai que ouve o evento `@increment` se `count` é par ou ímpar:

```js {12-15}
const Counter = {
  template: `<button @click="handleClick">Increment</button>`,
  data() {
    return {
      count: 0
    }
  },
  methods: {
    handleClick() {
      this.count += 1

      this.$emit('increment', {
        count: this.count,
        isEven: this.count % 2 === 0
      })
    }
  }
}
```

Como fizemos anteriormente, precisamos acionar o evento `click` no elemento `<button>`. Em seguida, usamos `emitted('increment')` para garantir que os valores corretos são emitidos:

```js
test('emits an event with count when clicked', () => {
  const wrapper = mount(Counter)

  wrapper.find('button').trigger('click')
  wrapper.find('button').trigger('click')

  // "Clicamos" duas vezes, então o vetor de
  // `increment` deve ter dois valores.
  expect(wrapper.emitted('increment')).toHaveLength(2)

  // Depois, podemos garantir que cada elemento de
  // `wrapper.emitted('increment')` contém um vetor
  // com o objeto esperado.
  expect(wrapper.emitted('increment')[0]).toEqual([
    {
      count: 1,
      isEven: false
    }
  ])

  expect(wrapper.emitted('increment')[1]).toEqual([
    {
      count: 2,
      isEven: true
    }
  ])
})
```

Testar as cargas úteis de eventos complexos, como objetos, não é diferente de testar valores simples como números ou sequências de caracteres.

## API de Composição {#Composition-API}

Se estivermos usando a API de Composição, estaremos chamando `context.emit()` ao invés de `this.$emit()`. A `emitted()` captura eventos de ambos, então podemos testar nosso componente usando as mesmas técnicas descritas neste artigo.

## Conclusão {#Conclusion}

- Usar `emitted()` para acessar eventos emitidos a partir de um componente de Vue.
- A `emitted(eventName)` retorna um vetor, onde cada elemento representa um evento emitido.
- Os argumentos são armazenados em `emitted(eventName)[index]` num vetor na mesma ordem em que são emitidos.

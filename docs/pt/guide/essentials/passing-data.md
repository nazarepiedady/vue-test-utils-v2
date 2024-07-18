# Passar Dados aos Componentes {#Passing-Data-to-Components}

A Vue Test Utils fornece várias maneiras de definir dados e propriedades num componente, para permitir-nos testar completamente o comportamento do componente em diferentes cenários.

Nesta secção, exploramos as opções de montagem `data` e `props`, bem como a `VueWrapper.setProps()` para atualizar dinamicamente as propriedades que um componente recebe.

## O Componente `Password` {#The-Password-Component}

Demonstraremos as funcionalidades acima construindo um componente `<Password>`. Este componente verifica se uma palavra-passe cumpre determinados critérios, como o comprimento e a complexidade. Começaremos com o seguinte e adicionaremos funcionalidades, bem como testes para garantir que as funcionalidades funcionam corretamente:

```js
const Password = {
  template: `
    <div>
      <input v-model="password">
    </div>
  `,
  data() {
    return {
      password: ''
    }
  }
}
```

O primeiro requisito que acrescentaremos é um comprimento mínimo.

## Usar `props` para definir um comprimento mínimo {#Using-props-to-set-a-minimum-length}

Queremos reutilizar este componente em todos os nossos projetos, cada um dos quais pode ter requisitos diferentes. Por esta razão, faremos de `minLength` uma **propriedade** que passamos ao `<Password>`.

Mostraremos um erro se `password` for menor que `minLength`. Podemos fazer isto criando uma propriedade computada `error` e a desenharemos condicionalmente usando `v-if`:

```js
const Password = {
  template: `
    <div>
      <input v-model="password">
      <div v-if="error">{{ error }}</div>
    </div>
  `,
  props: {
    minLength: {
      type: Number
    }
  },
  computed: {
    error() {
      if (this.password.length < this.minLength) {
        return `Password must be at least ${this.minLength} characters.`
      }
      return
    }
  }
}
```

Para testar isto, precisamos definir a `minLength`, bem como uma `password` que é menor que este número. Podemos fazer isto usando as opções de montagem `data` e `props`. Por fim, asseriremos que a mensagem de erro correta é desenhada:

```js
test('renders an error if length is too short', () => {
  const wrapper = mount(Password, {
    props: {
      minLength: 10
    },
    data() {
      return {
        password: 'short'
      }
    }
  })

  expect(wrapper.html()).toContain('Password must be at least 10 characters')
})
```

A escrita de um teste para uma regra `maxLength` é deixado como um exercício ao leitor! Outra maneira de escrever isto seria usando a `setValue` para atualizar a entrada com uma palavra-passe que é muito curta. Podemos obter mais informações no artigo sobre [Formulários](./forms).

## Usar `setProps` {#Using-setProps}

Por vezes, podemos precisar de escrever um teste para um efeito colateral da alteração duma propriedade. Este simples componente `<Show>` desenha uma saudação se a propriedade `show` for `true`.

```vue
<template>
  <div v-if="show">{{ greeting }}</div>
</template>

<script>

export default {
  props: {
    show: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      greeting: 'Hello'
    }
  }
}
</script>
```

Para testar isto completamente, podemos querer verificar se `greeting` é desenhado por padrão. Podemos atualizar a propriedade `show` utilizando `setProps()`, o que esconde `greeting`:

```js
import { mount } from '@vue/test-utils'
import Show from './Show.vue'

test('renders a greeting when show is true', async () => {
  const wrapper = mount(Show)
  expect(wrapper.html()).toContain('Hello')

  await wrapper.setProps({ show: false })

  expect(wrapper.html()).not.toContain('Hello')
})
```

Também usamos a palavra-chave `await` quando chamamos `setProps()`, para garantir que o modelo de objeto do documento foi atualizado antes da execução das asserções.

## Conclusão {#Conclusion}

- Usar as opções de montagem `props` e `data` para pré-definir o estado de um componente.
- Usar `setProps()` para atualizar uma propriedade durante um teste.
- Usar a palavra-chave `await` antes de `setProps()` para garantir que a Vue atualizará o modelo de objeto do documento antes que o teste continue.
- Interagir diretamente com o nosso componente pode dar-nos uma maior cobertura. Considerar usar `setValue` ou `trigger` em combinação com `data` para garantir que funciona corretamente.

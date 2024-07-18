# Ranhuras {#Slots}

A Vue Test Utils fornece algumas funcionalidades úteis para testar componentes que usam `slots`.

## Um Exemplo Simples {#A-Simple-Example}

Podemos ter um componente genérico `<layout>` que usa uma ranhura predefinida para desenhar algum conteúdo. Por exemplo:

```js
const Layout = {
  template: `
    <div>
      <h1>Welcome!</h1>
      <main>
        <slot />
      </main>
      <footer>
        Thanks for visiting.
      </footer>
    </div>
  `
}
```

Talvez queiramos escrever um teste para garantir que o conteúdo da ranhura predefinida é desenhado. A VTU fornece a opção de montagem `slots` para este propósito:

```js
test('layout default slot', () => {
  const wrapper = mount(Layout, {
    slots: {
      default: 'Main Content'
    }
  })

  expect(wrapper.html()).toContain('Main Content')
})
```

Passa! Neste exemplo, passamos algum conteúdo de texto para a ranhura predefinida. Se quisermos ser ainda mias específicos, e verificar se o conteúdo da ranhura predefinida é desenhado dentro de `<main>`, podemos mudar a asserção:

```js
test('layout default slot', () => {
  const wrapper = mount(Layout, {
    slots: {
      default: 'Main Content'
    }
  })

  expect(wrapper.find('main').text()).toContain('Main Content')
})
```

## Ranhuras Nomeadas {#Named-Slots}

Podemos ter um componente `<layout>` mais complexo com algumas ranhuras nomeadas. Por exemplo:

```js
const Layout = {
  template: `
    <div>
      <header>
        <slot name="header" />
      </header>

      <main>
        <slot name="main" />
      </main>
      <footer>
        <slot name="footer" />
      </footer>
    </div>
  `
}
```

A VTU também suporta isto. Podemos escrever um teste da seguinte maneira. Notemos que, neste exemplo, passamos HTML em vez de conteúdo de texto para as ranhuras:

```js
test('layout full page layout', () => {
  const wrapper = mount(Layout, {
    slots: {
      header: '<div>Header</div>',
      main: '<div>Main Content</div>',
      footer: '<div>Footer</div>'
    }
  })

  expect(wrapper.html()).toContain('<div>Header</div>')
  expect(wrapper.html()).toContain('<div>Main Content</div>')
  expect(wrapper.html()).toContain('<div>Footer</div>')
})
```

## Várias Ranhuras {#Multiple-Slots}

Também podemos passar um vetor de ranhuras:

```js
test('layout full page layout', () => {
  const wrapper = mount(Layout, {
    slots: {
      default: [
        '<div id="one">One</div>',
        '<div id="two">Two</div>'
      ]
    }
  })

  expect(wrapper.find('#one').exists()).toBe(true)
  expect(wrapper.find('#two').exists()).toBe(true)
})
```

## Uso Avançado {#Advanced-Usage}

Também podemos passar uma função de interpretação, um objeto com modelo de marcação ou até mesmo um componente de ficheiro único importado a partir de um ficheiro `vue` para uma opção de montagem da ranhura:

```js
import { h } from 'vue'
import Header from './Header.vue'

test('layout full page layout', () => {
  const wrapper = mount(Layout, {
    slots: {
      header: Header,
      main: h('div', 'Main Content'),
      sidebar: { template: '<div>Sidebar</div>' },
      footer: '<div>Footer</div>'
    }
  })

  expect(wrapper.html()).toContain('<div>Header</div>')
  expect(wrapper.html()).toContain('<div>Main Content</div>')
  expect(wrapper.html()).toContain('<div>Footer</div>')
})
```

[Consultar os testes](https://github.com/vuejs/test-utils/blob/9d3c2a6526f3d8751d29b2f9112ad2a3332bbf52/tests/mountingOptions/slots.spec.ts#L124-L167) para obter mais exemplos e casos de uso.

## Ranhuras Delimitadas {#scoped-slots}

As [ranhuras delimitadas](https://pt.vuejs.org/guide/components/slots#scoped-slots) e vínculos também são suportados.

```js
const ComponentWithSlots = {
  template: `
    <div class="scoped">
      <slot name="scoped" v-bind="{ msg }" />
    </div>
  `,
  data() {
    return {
      msg: 'world'
    }
  }
}

test('scoped slots', () => {
  const wrapper = mount(ComponentWithSlots, {
    slots: {
      scoped: `<template #scoped="scope">
        Hello {{ scope.msg }}
        </template>
      `
    }
  })

  expect(wrapper.html()).toContain('Hello world')
})
```

Quando usamos modelos de marcação de sequência de caracteres para o conteúdo da ranhura, **se não explicitamente definido com o uso de um marcador de embrulho `<template #scoped="scopeVar">`**, o âmbito da ranhura torna-se disponível como um objeto `params` quando a ranhura é avaliada.

```js
test('scoped slots', () => {
  const wrapper = mount(ComponentWithSlots, {
    slots: {
      // não é fornecido nenhum marcador de modelo de
      // marcação de embrulho, o âmbito da ranhura é
      // exposto como "params".
      scoped: `Hello {{ params.msg }}`
    }
  })

  expect(wrapper.html()).toContain('Hello world')
})
```

## Conclusion {#Conclusion}

- Usar a opção de montagem `slots` para testar se os componentes que usam `<slot>` desenham o conteúdo corretamente.
- O conteúdo pode ser uma sequência de caracteres, uma função de interpretação, ou um componente de ficheiro único importado.
- Usar `default` para a ranhura predefinida, e o nome correto para as ranhuras nomeadas.
- Ranhuras delimitadas e a abreviação `#` também são suportadas.

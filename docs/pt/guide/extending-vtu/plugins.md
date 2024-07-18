# Extensões {#Plugins}

As extensões adicionam funcionalidades de nível global à interface de programação de aplicação da Vue Test Utils. Esta é a maneira oficial de estender a interface de programação de aplicação da Vue Test Utils com lógica, métodos ou funcionalidades personalizadas.

Alguns casos de uso para extensões:

1. Atribuir pseudónimos aos métodos públicos existentes.
2. Anexar os combinadores à instância do embrulhador (objeto `Wrapper`).
3. Anexar a funcionalidade ao embrulhador (objeto `Wrapper`).

## Extensão Embrulhadora {#Wrapper-Plugin}

### Usar uma Extensão {#Using-a-Plugin}

Instalar as extensões ao chamar o método `config.plugins.VueWrapper.install()`. Isto tem de ser feito antes de chamarmos `mount`.

O método `install()` receberá uma instância de `WrapperAPI` contendo ambas as propriedades públicas e privadas da instância:

```js
// ficheiro `setup.js`
import { config } from '@vue/test-utils'

// extensão definida localmente,
// consultar “Escrever uma Extensão”
import MyPlugin from './myPlugin'

// Instalar uma extensão no `VueWrapper`
config.plugins.VueWrapper.install(MyPlugin)
```

Podemos, opcionalmente, introduzir algumas opções:

```js
config.plugins.VueWrapper.install(MyPlugin, { someOption: true })
```

A nossa extensão deve ser instalada uma vez. Se usarmos a Jest, isto deve estar na `setupFiles` ou `setupFilesAfterEnv` do nosso ficheiro de configuração da Jest.

Algumas extensões chamam `config.plugins.VueWrapper.install()` quando sõa importados. Isto é comum se estendemos várias interfaces ao mesmo tempo. Sigamos as instruções da extensão que instalamos.

Consultar o [Guia da Comunidade da Vue](https://vue-community.org/guide/ecosystem/testing.html) ou [awesome-vue](https://github.com/vuejs/awesome-vue#test) para obter uma coleção de extensões e bibliotecas contribuídas pela comunidade.

### Escrever uma Extensão {#Writing-a-Plugin}

Uma extensão de Vue Test Utils é simplesmente uma função que recebe a instância de `VueWrapper` ou `DOMWrapper` montada e pode modificá-la.

#### Extensão Básica {#Basic-Plugin}

Abaixo está uma extensão simples para adicionar um pseudónimo conveniente para mapear `wrapper.element` para `wrapper.$el`:

```js
// setup.js
import { config } from '@vue/test-utils'

const myAliasPlugin = (wrapper) => {
  return {
    $el: wrapper.element // pseudónimo simples
  }
}

// Chamar `install` no tipo que queremos estender
// Podemos escrever uma extensão para qualquer valor
// dentro de `config.plugins`,
config.plugins.VueWrapper.install(myAliasPlugin)
```

E na nossa especificação, poderemos usar a nossa extensão após `mount`:

```js
// component.spec.js
const wrapper = mount({ template: `<h1>🔌 Plugin</h1>` })
console.log(wrapper.$el.innerHTML) // 🔌 Plugin
```

#### Extensão do Identificador de Teste de Dados {#Data-Test-ID-Plugin}

A extensão abaixo adiciona um método `findByTestId` à instância de `VueWrapper`. Isto encoraja o uso duma estratégia de seletor que depende de atributos exclusivos para teste nos nossos componentes `.vue`.

Uso:

`MyComponent.vue`:

```vue
<template>
  <MyForm class="form-container" data-testid="form">
    <MyInput data-testid="name-input" v-model="name" />
  </MyForm>
</template>
```

`MyComponent.spec.js`:

```js
const wrapper = mount(MyComponent)
wrapper.findByTestId('name-input') // retorna `VueWrapper` ou `DOMWrapper`
 ```

Implementação da extensão:

```js
import { config } from '@vue/test-utils'

const DataTestIdPlugin = (wrapper) => {
  function findByTestId(selector) {
    const dataSelector = `[data-testid='${selector}']`
    const element = wrapper.element.querySelector(dataSelector)
    return new DOMWrapper(element)
  }

  return {
    findByTestId
  }
}

config.plugins.VueWrapper.install(DataTestIdPlugin)
```

## Extensão de Esboços {#Stubs-Plugin}

A `config.plugins.createStubs` permite sobrescrever a criação predefinida de esboço fornecida pela VTU.

Alguns casos de utilização são:

* Queremos adicionar mais lógica aos esboços (por exemplo, ranhuras nomeadas).
* Queremos utilizar diferentes esboços para vários componentes (por exemplo, componentes de esboço duma biblioteca).

### Uso {#Usage}

```ts
config.plugins.createStubs = ({ name, component }) => {
  return defineComponent({
    render: () => h(`custom-${name}-stub`)
  })
}
```

Esta função será chamada sempre que a VTU gerar um esboço a partir de:

```ts
const wrapper = mount(Component, {
  global: {
    stubs: {
      ChildComponent: true
    }
  }
})
```

ou:

```ts
const wrapper = shallowMount(Component)
```

Mas não será chamada, quando definirmos explicitamente um esboço:

```ts
const wrapper = mount(Component, {
  global: {
    stubs: {
      ChildComponent: { template: '<child-stub/>' }
    }
  }
})
```

## Usar a Extensão com a TypeScript {#Using-the-plugin-with-TypeScript}

Para utilizar a nossa extensão embrulhadora personalizada com a [TypeScript](https://www.typescriptlang.org/), temos de declarar a nossa função embrulhadora personalizada. Portanto, adicionamos um ficheiro chamado `vue-test-utils.d.ts` com o seguinte conteúdo:


```ts
import { DOMWrapper } from '@vue/test-utils';

declare module '@vue/test-utils' {
  interface VueWrapper {
    findByTestId(testId: string): DOMWrapper[];
  }
}
```

## Disponibilizar a Nossa Extensão {#Featuring-Your-Plugin}

Se sentirmos falta de alguma funcionalidade, consideremos escrever uma extensão para estender a Vue Test Utils e submetê-la para ser apresentada no [Guia da Comunidade da Vue](https://vue-community.org/guide/ecosystem/testing.html) ou [awesome-vue](https://github.com/vuejs/awesome-vue#test).

# Testar `v-model` {#Testing-v-model}

Quando escrevemos componentes que dependem da interação da `v-model` (evento `update:modelValue`), precisamos manipular o `event` e as `props`.

Consultar a [discussão sobre a "integração da vmodel"](https://github.com/vuejs/test-utils/discussions/279) por algumas soluções da comunidade.

Consultar a [documentação do evento VModel da VueJS](https://pt.vuejs.org/guide/components/v-model).

## Um Exemplo Simples {#A-Simple-Example}

Eis um simples componente `Editor`:

```js
const Editor = {
  props: {
    label: String,
    modelValue: String
  },
  emits: ['update:modelValue'],
  template: `<div>
    <label>{{label}}</label>
    <input :value="modelValue" @input="$emit('update:modelValue', $event.target.value)">
  </div>`
}
```

Este componente comporta-se-á apenas como um componente de entrada:

```js
const App {
  components: {
    Editor
  },
  template: `<editor v-model="text" label="test" />`,
  data(){
    return {
      text: 'test'
    }
  }
}
```

Agora, quando digitamos na entrada, este atualizará o `text` no nosso componente.

Para testar este comportamento:

```js
test('modelValue should be updated', async () => {
  const wrapper = mount(Editor, {
    props: {
      modelValue: 'initialText',
      'onUpdate:modelValue': (e) => wrapper.setProps({ modelValue: e })
    }
  })

  await wrapper.find('input').setValue('test')
  expect(wrapper.props('modelValue')).toBe('test')
})
```

# Várias `v-model` {#Multiple-v-model}

Em algumas situações, podemos ter várias `v-model` direcionadas a propriedades específicas.

Exemplo de um Editor de Dinheiro, podemos ter as propriedades `currency` e `modelValue`.

```js
const MoneyEditor = {
  template: `<div> 
    <input :value="currency" @input="$emit('update:currency', $event.target.value)"/>
    <input :value="modelValue" type="number" @input="$emit('update:modelValue', $event.target.value)"/>
  </div>`,
  props: ['currency', 'modelValue'],
  emits: ['update:currency', 'update:modelValue']
}
```

Podemos testar ambos:

```js
test('modelValue and currency should be updated', async () => {
  const wrapper = mount(MoneyEditor, {
    props: {
      modelValue: 'initialText',
      'onUpdate:modelValue': (e) => wrapper.setProps({ modelValue: e }),
      currency: '$',
      'onUpdate:currency': (e) => wrapper.setProps({ currency: e })
    }
  })

  const [currencyInput, modelValueInput] = wrapper.findAll('input')
  await modelValueInput.setValue('test')
  await currencyInput.setValue('£')

  expect(wrapper.props('modelValue')).toBe('test')
  expect(wrapper.props('currency')).toBe('£')
})
```

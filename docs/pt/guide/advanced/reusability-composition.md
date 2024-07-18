# Reutilização e Composição {#Reusability-Composition}

Principalmente:

- `global.mixins`.
- `global.directives`.

## Testar Funções de Composição {#Testing-composables}

Quando trabalhamos com a interface de programação de aplicação de composição e criamos funções de composição, muitas vezes queremos testar apenas a função de composição. Comecemos com um exemplo simples:

```typescript
export function useCounter() {
  const counter = ref(0)

  function increase() {
    counter.value += 1
  }

  return { counter, increase }
}
```

Neste caso, não precisamos realmente da `@vue/test-utils`. Eis o teste correspondente:

```typescript
test('increase counter on call', () => {
  const { counter, increase } = useCounter()

  expect(counter.value).toBe(0)

  increase()

  expect(counter.value).toBe(1)
})
```

Para funções de composição mais complexas, que usam as funções gatilhos do ciclo de vida como `onMounted` ou a manipulação de `provide` e `inject`, podemos criar um componente auxiliar de teste simples. A seguinte função de composição busca os dados do utilizador dentro da função gatilho `onMounted`:

```typescript
export function useUser(userId) {
  const user = ref()
  
  function fetchUser(id) {
    axios.get(`users/${id}`)
      .then(response => (user.value = response.data))
  }

  onMounted(() => fetchUser(userId))

  return { user }
}
```

Para testar esta função de composição, podemos criar um simples `TestComponent` nos testes. O `TestComponent` deve utilizar a função de composição exatamente da mesma maneira que os componentes reais o utilizariam:

```typescript
// Simular uma requisição de interface de programação de aplicação
jest.spyOn(axios, 'get').mockResolvedValue({ data: { id: 1, name: 'User' } })

test('fetch user on mount', async () => {
  const TestComponent = defineComponent({
    props: {
      // Definir propriedades, para testar a função de composição
      // com diferentes argumentos de entrada
      userId: {
        type: Number,
        required: true
      }
    },
    setup (props) {
      return {
        // Chamar a função de composição e expor todos valores de
        // retorno na nossa instância de componente para podermos
        // acessá-los com `wrapper.vm`
        ...useUser(props.userId)
      }
    }
  })

  const wrapper = mount(TestComponent, {
    props: {
      userId: 1
    }
  })

  expect(wrapper.vm.user).toBeUndefined()

  await flushPromises()

  expect(wrapper.vm.user).toEqual({ id: 1, name: 'User' })
})
```

## `provide` e `inject` {#Provide-inject}

A Vue oferece uma maneira de passar propriedades para todos os componentes filhos com `provide` e `inject`. A melhor maneira de testar este comportamento é testar a árvore inteira (pai + filhos). Mas, por vezes, isto não é possível, porque a árvore é demasiado complexa ou porque só queremos testar uma única função de composição.

### Testar a `provide` {#Testing-provide}

Assumamos o seguinte componente que queremos testar:

```vue
<template>
  <div>
    <slot />
  </div>
</template>

<script setup>
provide('my-key', 'some-data')
</script>
```

Neste caso, podemos desenhar um componente filho real e testar o uso correto de `provide` ou podemos criar um componente auxiliar de teste simples e passá-lo para a ranhura predefinida:

```typescript
test('provides correct data', () => {
  const TestComponent = defineComponent({
    template: '<span id="provide-test">{{value}}</span>',
    setup () {
      const value = inject('my-key')
      return { value }
    }
  })

  const wrapper = mount(ParentComponent, {
    slots: {
      default: () => h(TestComponent)
    }
  })

  expect(wrapper.find('#provide-test').text()).toBe('some-data')
})
```

Se o nosso componente não contém uma ranhura, podemos usar um [`stub`](./stubs-shallow-mount.md#stubbing-a-single-child-component) e substituir um componente filho pelo nosso auxiliar de teste:

```vue
<template>
  <div>
    <SomeChild />
  </div>
</template>

<script setup>
import SomeChild from './SomeChild.vue'

provide('my-key', 'some-data')
</script>
```

E o teste:

```typescript
test('provides correct data', () => {
  const TestComponent = defineComponent({
    template: '<span id="provide-test">{{value}}</span>',
    setup () {
      const value = inject('my-key')
      return { value }
    }
  })

  const wrapper = mount(ParentComponent, {
    global: {
      stubs: {
        SomeChild: TestComponent
      }
    }
  })

  expect(wrapper.find('#provide-test').text()).toBe('some-data')
})
```

### Testar a `inject` {#Testing-inject}

Quando o nosso componente usa `inject` e precisamos passar dados com a `provide`, então podemos utilizar a opção `global.provide`:

```vue
<template>
  <div>
    {{ value }}
  </div>
</template>

<script setup>
const value = inject('my-key')
</script>
```

O teste unitário poderia ser simplesmente parecido com:

```typescript
test('renders correct data', () => {
  const wrapper = mount(MyComponent, {
    global: {
      provide: {
        'my-key': 'some-data'
      }
    }
  })

  expect(wrapper.text()).toBe('some-data')
})
```

## Conclusão {#Conclusion}

- Testar funções de composição simples sem um componente e `@vue/test-utils`
- Criar um componente auxiliar de teste para testar funções de composições mais complexos
- Criar um componente auxiliar de teste para testar se o nosso componente fornece os dados corretos com `provide`
- Usar `global.provide` para passar dados para o nosso componente que usa `inject`

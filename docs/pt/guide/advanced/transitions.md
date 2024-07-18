# Transições {#Transitions}

Em geral, podemos querer testar o modelo de objeto do documento resultante após uma transição, e é por isto que a Vue Test Utils simula `<transition>` e `<transition-group>` por padrão.

Segue-se um componente simples que alterna um conteúdo embrulhado numa transição de desvanecimento:

```vue
<template>
  <button @click="show = !show">Toggle</button>

  <transition name="fade">
    <p v-if="show">hello</p>
  </transition>
</template>

<script>
import { ref } from 'vue'

export default {
  setup() {
    const show = ref(false)

    return {
      show
    }
  }
}
</script>

<style lang="css">
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
</style>
```

Uma vez que a Vue Test Utils esboça as transições embutidas, podemos testar o componente acima como testaríamos qualquer outro componente:

```js
import Component from './Component.vue'
import { mount } from '@vue/test-utils'

test('works with transitions', async () => {
  const wrapper = mount(Component)

  expect(wrapper.find('hello').exists()).toBe(false)

  await wrapper.find('button').trigger('click')

  // Depois de clicar o botão, o
  // elemento `<p>` existe e está visível
  expect(wrapper.get('p').text()).toEqual('hello')
})
```

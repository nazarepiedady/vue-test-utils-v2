# Testar o Teletransporte {#Testing-Teleport}

A Vue 3 vem com um novo componente embutido: `<Teleport>`, que permite os componentes “teletransportarem” o seu conteúdo para fora do seu próprio `<template>`. A maioria dos testes escritos com a Vue Test Utils está delimitada ao componente passado a `mount`, o que introduz alguma complexidade quando se trata de testar um componente teletransportado para fora do componente onde este é inicialmente desenhado.

Eis algumas estratégias e técnicas para testar componentes usando `<Teleport>`.

:::tip DICA
Se quisermos testar o resto do nosso componente, ignorando o teletransporte, podemos esboçar o `teleport` ao passar `teleport: true` na [opção global `stubs`](../../api/#global-stubs).
:::

## Exemplo {#Example}

Neste exemplo, testamos um componente `<Navbar>`. Este desenha um componente `<Signup>` num `<Teleport>`. A propriedade `target` do `<Teleport>` é um elemento localizado fora do componente `<Navbar>`.

Este é o componente `Navbar.vue`:

```vue
<template>
  <Teleport to="#modal">
    <Signup />
  </Teleport>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import Signup from './Signup.vue'

export default defineComponent({
  components: {
    Signup
  }
})
</script>
```

Este simplesmente teletransporta um `<Signup>` para outro lugar. É simples para o propósito deste exemplo.

O `Signup.vue` é um formulário que validade se `username` é maior que 8 caracteres. Se for, quando submetido, este emite um evento `signup` com a `username` como a carga. Testar isto será o nosso objetivo:

```vue
<template>
  <div>
    <form @submit.prevent="submit">
      <input v-model="username" />
    </form>
  </div>
</template>

<script>
export default {
  emits: ['signup'],
  data() {
    return {
      username: ''
    }
  },
  computed: {
    error() {
      return this.username.length < 8
    }
  },
  methods: {
    submit() {
      if (!this.error) {
        this.$emit('signup', this.username)
      }
    }
  }
}
</script>
```

## Montar o Componente {#Mounting-the-Component}

Começamos com um teste mínimo:

```ts
import { mount } from '@vue/test-utils'
import Navbar from './Navbar.vue'
import Signup from './Signup.vue'

test('emits a signup event when valid', async () => {
  const wrapper = mount(Navbar)
})
```

Executar este teste dar-nos-á um aviso: `[Vue warn]: Failed to locate Teleport target with selector "#modal"`. Criaremos isto:

```ts {5-15}
import { mount } from '@vue/test-utils'
import Navbar from './Navbar.vue'
import Signup from './Signup.vue'

beforeEach(() => {
  // criar o destino do teletransporte
  const el = document.createElement('div')
  el.id = 'modal'
  document.body.appendChild(el)
})

afterEach(() => {
  // limpar
  document.body.outerHTML = ''
})

test('teleport', async () => {
  const wrapper = mount(Navbar)
})
```

Utilizamos a Jest para este exemplo, o que não reinicia o modelo de objeto do documento em cada teste. Por esta razão, é bom limpar após cada teste com `afterEach`.

## Interagir com o Componente Teletransportado {#Interacting-with-the-Teleported-Component}

A próxima coisa a fazer é preencher a entrada do nome do utilizador. Infelizmente, não podemos utilizar `wrapper.find('input')`. Por que não? Uma rápida `console.log(wrapper.html())` mostra-nos:

```html
<!--teleport start-->
<!--teleport end-->
```

Vemos alguns comentários usados pela Vue para lidar com o `<Teleport>` — mas nenhum `<input>`. Isto acontece porque o componente `<Signup>` (e a sua marcação de hipertexto) não é mais desenhada dentro do `<Navbar>` — este foi teletransportado para fora.

Apesar da marcação de hipertexto de fato ser teletransportada para fora, o modelo de objeto do documento virtual associado ao `<Navbar>` mantém uma referência ao componente original. Isto significa que podemos usar `getComponent` e `findComponent`, que operam no modelo de objeto do documento virtual, não no modelo de objeto do documento normal:

```ts {12}
beforeEach(() => {
  // ...
})

afterEach(() => {
  // ...
})

test('teleport', async () => {
  const wrapper = mount(Navbar)

  wrapper.getComponent(Signup) // já está!
})
```

A `getComponent` retorna um `VueWrapper`. Agora podemos usar métodos como `get`, `find` e `trigger`.

Terminaremos o teste:

```ts {4-8}
test('teleport', async () => {
  const wrapper = mount(Navbar)

  const signup = wrapper.getComponent(Signup)
  await signup.get('input').setValue('valid_username')
  await signup.get('form').trigger('submit.prevent')

  expect(signup.emitted().signup[0]).toEqual(['valid_username'])
})
```

Passa!

O teste completo:

```ts
import { mount } from '@vue/test-utils'
import Navbar from './Navbar.vue'
import Signup from './Signup.vue'

beforeEach(() => {
  // criar o destino do teletransporte
  const el = document.createElement('div')
  el.id = 'modal'
  document.body.appendChild(el)
})

afterEach(() => {
  // limpar
  document.body.outerHTML = ''
})

test('teleport', async () => {
  const wrapper = mount(Navbar)

  const signup = wrapper.getComponent(Signup)
  await signup.get('input').setValue('valid_username')
  await signup.get('form').trigger('submit.prevent')

  expect(signup.emitted().signup[0]).toEqual(['valid_username'])
})
```

Podemos esboçar o teletransporte ao usar `teleport: true`:

```ts
import { mount } from '@vue/test-utils'
import Navbar from './Navbar.vue'

test('teleport', async () => {
  const wrapper = mount(Navbar, {
    global: {
      stubs: {
        teleport: true
      }
    }
  })
})
```

## Conclusão {#Conclusion}

- Criar um destino de teletransporte com `document.createElement`.
- Encontrar componentes teletransportados usando a `getComponent` ou a `findComponent` que operam no nível do modelo de objeto do documento virtual.

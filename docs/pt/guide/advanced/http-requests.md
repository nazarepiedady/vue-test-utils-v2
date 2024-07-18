# Fazer Requisições de Protocolo Hipertexto {#Making-HTTP-requests}

As ferramentas de execução de teste modernas já fornecem muitos recursos excelentes quando se trata de testar requisições de hipertexto. Assim, a Vue Test Utils não apresenta nenhuma ferramenta única para fazer isto.

No entanto, é uma funcionalidade importante para testar, e existe alguns problemas que queremos destacar.

Nesta secção, exploramos alguns padrões para executar, simular, e asserir requisições de hipertexto.

## Uma lista de publicações de blogue {#A-list-of-blog-posts}

Começaremos por um caso de utilização básico. O componente `PostList` a seguir desenha uma lista de publicações de blogue obtidas duma interface de programação de aplicação externa. Para obter estas publicações, o componente apresenta um elemento `button` que aciona a requisição:

```vue
<template>
  <button @click="getPosts">Get posts</button>
  <ul>
    <li v-for="post in posts" :key="post.id" data-test="post">
      {{ post.title }}
    </li>
  </ul>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      posts: null
    }
  },
  methods: {
    async getPosts() {
      this.posts = await axios.get('/api/posts')
    }
  }
}
</script>
```

Há várias coisas que precisamos de fazer para testar corretamente este componente.

O nosso primeiro objetivo é testar este componente **sem chegar à interface de programação de aplicação**. Isto criaria um teste frágil e potencialmente lento.

Em segundo lugar, temos de garantir que o componente fez a chamada correta com os parâmetros corretos. Não obteremos resultados desta interface de programação de aplicação, mas ainda assim precisamos de garantir que pedimos os recursos corretos.

Além disto, temos de certificar-nos de que o modelo de objeto do documento foi atualizado em conformidade e apresenta os dados. Fazemos isto utilizando a função `flushPromises()` da `@vue/test-utils`:

```js
import { mount, flushPromises } from '@vue/test-utils'
import axios from 'axios'
import PostList from './PostList.vue'

const mockPostList = [
  { id: 1, title: 'title1' },
  { id: 2, title: 'title2' }
]

// As linhas seguintes dizem a Jest para simular qualquer
// chamada a `axios.get` e para retornar `mockPostList`
// em seu lugar
jest.spyOn(axios, 'get').mockResolvedValue(mockPostList)

test('loads posts on button click', async () => {
  const wrapper = mount(PostList)

  await wrapper.get('button').trigger('click')

  // Asseriremos que chamamos `axios.get` a quantidade certa de
  // vezes e com os parâmetros corretos.
  expect(axios.get).toHaveBeenCalledTimes(1)
  expect(axios.get).toHaveBeenCalledWith('/api/posts')

  // Aguardar até que o modelo de objeto do
  // documento ser atualizado.
  await flushPromises()

  // Por fim, nos certificamos de que desenhamos o
  // conteúdo da interface de programação de aplicação.
  const posts = wrapper.findAll('[data-test="post"]')

  expect(posts).toHaveLength(2)
  expect(posts[0].text()).toContain('title1')
  expect(posts[1].text()).toContain('title2')
})
```

Prestemos atenção que adicionamos o prefixo `mock` à variável `mockPostList`. Se não, receberemos o erro: "A fábrica de módulo de `jest.mock()` não tem permissão para referir-se a nenhuma variável fora do âmbito.". Isto é específico da Jest, e podemos ler mais sobre este comportamento [na sua documentação](https://jestjs.io/docs/es6-class-mocks#calling-jestmock-with-the-module-factory-parameter).

Observemos também como aguardamos `flushPromises` e depois interagimos com o componente. Fazemos isto para garantir que o modelo de objeto do documento foi atualizado antes da execução das asserções.

:::tip Alternativas à `jest.mock()`
Há várias maneiras de definir simulações em Jest. A usada no exemplo acima é a mais simples. Para alternativas mais poderosas, talvez queiramos verificar a [`axios-mock-adapter`](https://github.com/ctimmerm/axios-mock-adapter) ou a [`msw`](https://github.com/mswjs/msw), entre outros.
:::

### Asserir o estado de carregamento {#Asserting-loading-state}

Agora, teste componente `PostList` é bastante útil, mas falta-lhe algumas outras funcionalidades fantásticas. Vamos expandi-lo para apresentar uma mensagem elegante enquanto carrega as nossas publicações!

Além disto, também desativaremos o elemento `<button>` durante o carregamento. Não queremos que os utilizadores continuem a enviar pedidos durante a obtenção!:

```vue {2,4,19,24,28}
<template>
  <button :disabled="loading" @click="getPosts">Get posts</button>

  <p v-if="loading" role="alert">Loading your posts…</p>
  <ul v-else>
    <li v-for="post in posts" :key="post.id" data-test="post">
      {{ post.title }}
    </li>
  </ul>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      posts: null,
      loading: null
    }
  },
  methods: {
    async getPosts() {
      this.loading = true

      this.posts = await axios.get('/api/posts')

      this.loading = null
    }
  }
}
</script>
```

Escreveremos um teste para garantir que todos os elementos relacionados com o carregamento são desenhados a tempo:

```js
test('displays loading state on button click', async () => {
  const wrapper = mount(PostList)

  // Reparemos que executamos as seguintes asserções antes de clicar no botão
  // Neste caso, o componente deve estar num estado de “não carregamento”.
  expect(wrapper.find('[role="alert"]').exists()).toBe(false)
  expect(wrapper.get('button').attributes()).not.toHaveProperty('disabled')

  // Agora o acionaremos como de costume.
  await wrapper.get('button').trigger('click')

  // Asserimos o “estado de carregamento” antes de
  // eliminarmos todas as promessas.
  expect(wrapper.find('[role="alert"]').exists()).toBe(true)
  expect(wrapper.get('button').attributes()).toHaveProperty('disabled')

  // Tal como fizemos anteriormente, aguardamos até
  // o modelo de objeto do documento ser atualizado.
  await flushPromises()

  // Depois disto, voltamos a um estado de “não carregamento”.
  expect(wrapper.find('[role="alert"]').exists()).toBe(false)
  expect(wrapper.get('button').attributes()).not.toHaveProperty('disabled')
})
```

## Requisições de Hipertexto da Vuex {#HTTP-requests-from-Vuex}

Um cenário típico para aplicações mais complexas é acionar uma ação de Vuex que realiza a requisição de protocolo de hipertexto.

Isto não é diferente do exemplo acima descrito. Podemos querer carregar o armazém de estado como está e simular serviços como `axios`. Desta maneira, simulamos os limites do nosso sistema, alcançando assim mais confiança nos nossos testes.

Podemos consultar a documentação [Testar a Vuex](vuex.md) para obter mais informações sobre como testar a Vuex com a Vue Test Utils.

## Conclusão {#Conclusion}

- A Vue Test Utils não requer ferramentas especiais para testar requisições de protocolo de hipertexto. A única coisa a ter em conta é que testamos um comportamento assíncrono.
- Os testes não devem depender de serviços externos. Utilizemos ferramentas de simulação como `jest.mock` para evitar isto.
- A `flushPromises()` é uma ferramenta útil para garantir que o modelo de objeto do documento é atualizado após uma operação assíncrona.
- Acionar diretamente as requisições de hipertexto através da interação com o componente torna o nosso teste mais resistente.

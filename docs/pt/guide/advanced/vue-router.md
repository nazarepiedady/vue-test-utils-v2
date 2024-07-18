# Testar a Vue Router {#Testing-Vue-Router}

Este artigo apresentará duas maneiras de testar uma aplicação usando a Vue Router:

1. Utilizar a Vue Router verdadeira, a qual é mais parecida com a produção, mas também pode levar à complexidade quando se testam aplicações maiores.
2. Utilizar um roteador simulado, o que permite um controlo mais preciso do ambiente de teste.

Notemos que a Vue Test Utils não fornece nenhuma função especial para ajudar a testar componentes que dependem da Vue Router.

## Usar um Roteador Simulado {#Using-a-Mocked-Router}

Podemos usar um roteador simulado para não termos que preocupar-nos com os detalhes de implementação da Vue Router nos nossos testes unitários.

Em vez de utilizar uma instância verdadeira de Vue Router, podemos criar uma versão simulada que apenas implementa as funcionalidades em que estamos interessados. Podemos fazer isto usando uma combinação de `jest.mock` (se usarmos a Jest), e `global.components`.

Quando simulamos uma dependência, geralmente é porque **não estamos interessados em testar o seu comportamento**. Não queremos testar se clicar em `<router-link>` navega para a página correta — é claro que sim! No entanto, podemos estar interessados em garantir que o `<a>` tem o atributo `to` correto.

Veremos um exemplo mais realista! Este componente mostra um botão que redireciona um utilizador autenticado para página de edição de publicações (baseado nos parâmetros da rota atual). Um utilizador não autenticado deve ser redirecionado para uma rota `/404`:

```js
const Component = {
  template: `<button @click="redirect">Click to Edit</button>`,
  props: ['isAuthenticated'],
  methods: {
    redirect() {
      if (this.isAuthenticated) {
        this.$router.push(`/posts/${this.$route.params.id}/edit`)
      } else {
        this.$router.push('/404')
      }
    }
  }
}
```

Poderíamos usar um roteador verdadeiro, navegar para a rota correta para este componente e, depois de clicar no botão, verificar se a página correta é desenhada... No entanto, isto é muita configuração para um teste relativamente simples. Na sua essência, o teste que queremos escrever é “se autenticado, redirecionar para X, caso contrário redirecionar para Y”. Veremos como podemos fazer isto ao simular o roteamento usando a propriedade `globals.mocks`:

```js
import { mount } from '@vue/test-utils';

test('allows authenticated user to edit a post', async () => {
  const mockRoute = {
    params: {
      id: 1
    }
  }
  const mockRouter = {
    push: jest.fn()
  }

  const wrapper = mount(Component, {
    props: {
      isAuthenticated: true
    },
    global: {
      mocks: {
        $route: mockRoute,
        $router: mockRouter
      }
    }
  })

  await wrapper.find('button').trigger('click')

  expect(mockRouter.push).toHaveBeenCalledTimes(1)
  expect(mockRouter.push).toHaveBeenCalledWith('/posts/1/edit')
})

test('redirect an unauthenticated user to 404', async () => {
  const mockRoute = {
    params: {
      id: 1
    }
  }
  const mockRouter = {
    push: jest.fn()
  }

  const wrapper = mount(Component, {
    props: {
      isAuthenticated: false
    },
    global: {
      mocks: {
        $route: mockRoute,
        $router: mockRouter
      }
    }
  })

  await wrapper.find('button').trigger('click')

  expect(mockRouter.push).toHaveBeenCalledTimes(1)
  expect(mockRouter.push).toHaveBeenCalledWith('/404')
})
```

Utilizamos a `global.mocks` para fornecer as dependências necessárias (`this.$route` e `this.$router`) para definir um estado ideal para cada teste.

Pudemos então usar `jest.fn()` para supervisionar quantas vezes, e com quais argumentos, `this.$router.push` foi chamado. O melhor de tudo é que não temos de lidar com a complexidade ou com as limitações da Vue Router no nosso teste! Só nos preocupámos em testar a lógica da aplicação.

:::tip DICA
Podemos querer testar todo o sistema duma maneira completa. Poderíamos considerar uma abstração como a [Cypress](https://www.cypress.io/) para testes completos do sistema utilizando um navegador verdadeiro.
:::

## Usar um Roteador Verdadeiro {#Using-a-Real-Router}

Agora que já vimos como usar um roteador simulado, daremos uma olhada em como usar a Vue Router verdadeira.

Criaremos uma aplicação básica de blogue que usa a Vue Router. As publicações são listadas na rota `/posts`:

```js
const App = {
  template: `
    <router-link to="/posts">Go to posts</router-link>
    <router-view />
  `
}

const Posts = {
  template: `
    <h1>Posts</h1>
    <ul>
      <li v-for="post in posts" :key="post.id">
        {{ post.name }}
      </li>
    </ul>
  `,
  data() {
    return {
      posts: [{ id: 1, name: 'Testing Vue Router' }]
    }
  }
}
```

A raiz da aplicação apresenta um `<router-link>` que conduz a `/posts`, onde listamos as publicações.

O roteador verdadeiro tem o seguinte aspeto. Notemos que exportamos as rotas separadamente da rota, para podermos instanciar um novo roteador para cada teste individual mais tarde:

```js
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  {
    path: '/',
    component: {
      template: 'Welcome to the blogging app'
    }
  },
  {
    path: '/posts',
    component: Posts
  }
];

const router = createRouter({
  history: createWebHistory(),
  routes: routes,
})

export { routes };

export default router;
```

A melhor maneira de ilustrar como testar uma aplicação utilizando a Vue Router é deixar os avisos guiarem-nos. O teste mínimo que se segue é suficiente para ajudar-nos a avançar:

```js
import { mount } from '@vue/test-utils'

test('routing', () => {
  const wrapper = mount(App)
  expect(wrapper.html()).toContain('Welcome to the blogging app')
})
```

O teste falha. Também imprime dois avisos:

```bash
console.warn node_modules/@vue/runtime-core/dist/runtime-core.cjs.js:39
  [Vue warn]: Failed to resolve component: router-link

console.warn node_modules/@vue/runtime-core/dist/runtime-core.cjs.js:39
  [Vue warn]: Failed to resolve component: router-view
```

Os componentes `<router-link>` e `<router-view>` não foram encontrados. Precisamos de instalar a Vue Router! Uma vez que a Vue Router é uma extensão, a instalamos usando a opção de montagem `global.plugins`:

```js {12,13,14}
import { mount } from '@vue/test-utils'
import { createRouter, createWebHistory } from 'vue-router'
// Esta importação deve apontar para o nosso ficheiro de
// rotas declarado acima.
import { routes } from "@/router"

const router = createRouter({
  history: createWebHistory(),
  routes: routes,
})

test('routing', () => {
  const wrapper = mount(App, {
    global: {
      plugins: [router]
    }
  })
  expect(wrapper.html()).toContain('Welcome to the blogging app')
})
```

Estes dois avisos já desapareceram — mas agora temos outro aviso:

```js
console.warn node_modules/vue-router/dist/vue-router.cjs.js:225
  [Vue Router warn]: Unexpected error when starting the router: TypeError: Cannot read property '_history' of null
```

Embora não seja totalmente claro no aviso, está relacionado ao fato de a **Vue Router 4 manipular o roteamento de maneira assíncrona**.

A Vue Router fornece uma função `isReady` que diz-nos quando o roteador está pronto. Podemos então `await` para garantir que a navegação inicial aconteceu:

```js {13,14}
import { mount } from '@vue/test-utils'
import { createRouter, createWebHistory } from 'vue-router'
import { routes } from "@/router"

const router = createRouter({
  history: createWebHistory(),
  routes: routes,
})

test('routing', async () => {
  router.push('/')

  // Depois desta linha, o roteador está pronto
  await router.isReady()

  const wrapper = mount(App, {
    global: {
      plugins: [router]
    }
  })
  expect(wrapper.html()).toContain('Welcome to the blogging app')
})
```

O teste agora passa! Foi muito trabalhoso, mas agora nos certificamos de que a aplicação navega corretamente para a rota inicial.

Agora navegaremos para `/posts` e nos certificaremos que o roteamento funciona como esperado:

```js {21,22}
import { mount } from '@vue/test-utils'
import { createRouter, createWebHistory } from 'vue-router'
import { routes } from "@/router"

const router = createRouter({
  history: createWebHistory(),
  routes: routes,
})

test('routing', async () => {
  router.push('/')
  await router.isReady()

  const wrapper = mount(App, {
    global: {
      plugins: [router]
    }
  })
  expect(wrapper.html()).toContain('Welcome to the blogging app')

  await wrapper.find('a').trigger('click')
  expect(wrapper.html()).toContain('Testing Vue Router')
})
```

Novamente, outro erro enigmático:

```js
console.warn node_modules/@vue/runtime-core/dist/runtime-core.cjs.js:39
  [Vue warn]: Unhandled error during execution of native event handler
    at <RouterLink to="/posts" >

console.error node_modules/@vue/runtime-core/dist/runtime-core.cjs.js:211
  TypeError: Cannot read property '_history' of null
```

Novamente, devido à nova natureza assíncrona da Vue Router 4, precisamos `await` o roteamento concluir antes de fazer qualquer asserção.

Neste caso, no entanto, não existe nenhuma função gatilho `hasNavigated` que possamos aguardar. Uma alternativa é usar a função `flushPromises` exportada da Vue Test Utils:

```js {1,22}
import { mount, flushPromises } from '@vue/test-utils'
import { createRouter, createWebHistory } from 'vue-router'
import { routes } from "@/router"

const router = createRouter({
  history: createWebHistory(),
  routes: routes,
})

test('routing', async () => {
  router.push('/')
  await router.isReady()

  const wrapper = mount(App, {
    global: {
      plugins: [router]
    }
  })
  expect(wrapper.html()).toContain('Welcome to the blogging app')

  await wrapper.find('a').trigger('click')
  await flushPromises()
  expect(wrapper.html()).toContain('Testing Vue Router')
})
```

_Finalmente_ passa. Ótimo! No entanto, tudo isto é muito manual — e isto é para uma aplicação pequena e banal. Está é a razão pela qual usar um roteador simulado é uma abordagem comum ao tester componentes `.vue` usando a Vue Test Utils. No caso de preferirmos continuar a utilizar um roteador verdadeiro, não esqueçamos que cada teste deve utilizar a sua própria instância do roteador, da seguinte maneira:

```js {1,19}
import { mount, flushPromises } from '@vue/test-utils'
import { createRouter, createWebHistory } from 'vue-router'
import { routes } from "@/router"

let router;
beforeEach(async () => {
  router = createRouter({
    history: createWebHistory(),
    routes: routes,
  })
});

test('routing', async () => {
  router.push('/')
  await router.isReady()

  const wrapper = mount(App, {
    global: {
      plugins: [router]
    }
  })
  expect(wrapper.html()).toContain('Welcome to the blogging app')

  await wrapper.find('a').trigger('click')
  await flushPromises()
  expect(wrapper.html()).toContain('Testing Vue Router')
})
```

## Usar um Roteador Simulado com a API de Composição {#Using-a-mocked-router-with-Composition-API}

A Vue Router 4 permite trabalhar com o roteador e a rota dentro da função `setup` com a interface de programação de aplicação de composição.

Consideremos o mesmo componente de demonstração reescrito utilizando a interface de programação de aplicação de composição:

```js
import { useRouter, useRoute } from 'vue-router'

const Component = {
  template: `<button @click="redirect">Click to Edit</button>`,
  props: ['isAuthenticated'],
  setup (props) {
    const router = useRouter()
    const route = useRoute()

    const redirect = () => {
      if (props.isAuthenticated) {
        router.push(`/posts/${route.params.id}/edit`)
      } else {
        router.push('/404')
      }
    }

    return {
      redirect
    }
  }
}
```

Desta vez, para testar o componente, usaremos a capacidade da Jest de simular um recurso importado, `vue-router` e simular tanto o roteador quanto a rota diretamente:

```js
import { useRouter, useRoute } from 'vue-router'

jest.mock('vue-router', () => ({
  useRoute: jest.fn(),
  useRouter: jest.fn(() => ({
    push: () => {}
  }))
}))

test('allows authenticated user to edit a post', () => {
  useRoute.mockImplementationOnce(() => ({
    params: {
      id: 1
    }
  }))

  const push = jest.fn()
  useRouter.mockImplementationOnce(() => ({
    push
  }))

  const wrapper = mount(Component, {
    props: {
      isAuthenticated: true
    },
    global: {
      // Esboços da `router-link` e `router-view` para o caso
      // de serem desenhados no nosso modelo de marcação.
      stubs: ["router-link", "router-view"],
    }
  })

  await wrapper.find('button').trigger('click')

  expect(push).toHaveBeenCalledTimes(1)
  expect(push).toHaveBeenCalledWith('/posts/1/edit')
})

test('redirect an unauthenticated user to 404', () => {
  useRoute.mockImplementationOnce(() => ({
    params: {
      id: 1
    }
  }))

  const push = jest.fn()
  useRouter.mockImplementationOnce(() => ({
    push
  }))

  const wrapper = mount(Component, {
    props: {
      isAuthenticated: false
    }
    global: {
      // Esboços da `router-link` e `router-view` para o caso
      // de serem desenhados no nosso modelo de marcação.
      stubs: ["router-link", "router-view"],
    }
  })

  await wrapper.find('button').trigger('click')

  expect(push).toHaveBeenCalledTimes(1)
  expect(push).toHaveBeenCalledWith('/404')
})
```

## Usar um Roteador Verdadeiro com API de Composição {#Using-a-real-router-with-Composition-API}

A utilização de um roteador verdadeiro com a interface de programação de aplicação de composição funciona da mesma maneira que a utilização de um roteador verdadeiro com a interface de programação de aplicação de opções. Lembremos que, tal como acontece com a interface de programação de aplicação de opções, é considerado uma boa prática instanciar um novo objeto roteador para cada teste, em vez de importar o roteador diretamente da nossa aplicação:

```js
import { mount } from '@vue/test-utils'
import { createRouter, createWebHistory } from 'vue-router'
import { routes } from "@/router"

let router;

beforeEach(async () => {
  router = createRouter({
    history: createWebHistory(),
    routes: routes,
  })

  router.push('/')
  await router.isReady()
});

test('allows authenticated user to edit a post', async () => {
  const wrapper = mount(Component, {
    props: {
      isAuthenticated: true
    },
    global: {
      plugins: [router],
    }
  })

  const push = jest.spyOn(router, 'push')
  await wrapper.find('button').trigger('click')

  expect(push).toHaveBeenCalledTimes(1)
  expect(push).toHaveBeenCalledWith('/posts/1/edit')
})
```

Para aqueles que preferem uma abordagem não manual, a biblioteca [`vue-router-mock`](https://github.com/posva/vue-router-mock) criada por [@posva](https://github.com/posva) também está disponível como uma alternativa.

## Conclusão {#Conclusion}

- Podemos utilizar uma instância de `router` verdadeira nos nossos testes.
- Existem algumas advertências, no entanto: a Vue Router 4 é assíncrona, e precisamos de ter isto em conta quando escrevemos testes.
- Para aplicações mais complexas, consideremos a possibilidade de simular a dependência do roteador e concentrar-nos em testar a lógica subjacente.
- Precisamos de utilizar a funcionalidade de esboço ou simulação do nosso executor de testes sempre que possível.
- Utilizar `global.mocks` para simular dependências globais, como `this.$route` e `this.$router`.

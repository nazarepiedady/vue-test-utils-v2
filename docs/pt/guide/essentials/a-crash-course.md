# Um Curso Intensivo {#A-Crash-Course}

Neste curso intensivo, estudaremos a Vue Test Utils (VTU) ao construir uma simples aplicação de gestão de tarefas e escrevendo os testes a medida que seguimos. Este guia abordará como:

- Montar os componentes
- Encontrar os elementos
- Preencher os formulários
- Acionar os eventos

## Começar {#Getting-Started}

Nós começaremos com um simples componente `TodoApp` com uma única tarefa:

```vue
<template>
  <div></div>
</template>

<script>
export default {
  name: 'TodoApp',

  data() {
    return {
      todos: [
        {
          id: 1,
          text: 'Learn Vue.js 3',
          completed: false
        }
      ]
    }
  }
}
</script>
```

## O primeiro teste - uma tarefa é desenhada {#The-first-test-a-todo-is-rendered}

O primeiro teste que escreveremos verifica se uma tarefa é desenhada. Veremos o teste primeiro, e depois discutiremos cada parte:

```js
import { mount } from '@vue/test-utils'
import TodoApp from './TodoApp.vue'

test('renders a todo', () => {
  const wrapper = mount(TodoApp)

  const todo = wrapper.get('[data-test="todo"]')

  expect(todo.text()).toBe('Learn Vue.js 3')
})
```

Nós começamos por importar `mount` - esta é a maneira principal de desenhar um componente na VTU. Nós declaramos um teste usando a função `test` com uma descrição curta do teste. As funções `test` e `expect` estão disponíveis globalmente na maioria das ferramentas de execução de teste (este exemplo usa a [Jest](https://jestjs.io/en/)). Se `test` e `expect` parecerem confusas, a documentação da Jest tem um [exemplo mais simples](https://jestjs.io/docs/en/getting-started) de como usá-las e de como funcionam.

Em seguida, chamamos `mount` e passamos o componente como primeiro argumento - isto é algo que quase todos os testes que escrevemos farão. Por convenção, atribuímos o resultado a uma variável chamada `wrapper`, uma vez que `mount` fornece um "embrulhador" simples em torno da aplicação com alguns métodos convenientes para testes.

Finalmente, usamos outra função global comum para muitas ferramentas de execução de testes - incluindo a Jest - `expect`. A ideia é que estamos asserindo, ou _esperando_, que o resultado real corresponda ao que pensamos que deve ser. Neste caso, estamos procurando um elemento com o seletor `data-test="todo"` - no modelo de objeto do documento, isto parecer-se-á com `<div data-test="todo">...</div>`. Em seguida, chamamos o método `text` para obter o conteúdo, que esperamos que seja `'Learn Vue.js 3'`.

> Usar seletores `data-test` não é obrigatório, mas pode tornar os nossos testes menos frágeis. As classes e os identificadores tendem a mudar ou deslocar-se à medida que uma aplicação cresce - ao usar `data-test`, é claro aos outros programadores quais elementos são usados nos testes, e não devem ser alterados.

## Fazer o teste passar {#Making-the-test-pass}

Se executarmos teste teste agora, este falha com a seguinte mensagem de erro: `Unable to get [data-test="todo"]`. Isto porque não estamos desenhando nenhum item de tarefa, então a chamada `get()` está falhando em retornar um embrulhador (precisamos lembrar que a VTU embrulha todos os componentes e elementos do modelo de objeto do documento num "embrulhador" com alguns métodos convenientes). Atualizaremos o `<template>` no `TodoApp.vue` para desenhar o vetor `todos`:

```vue
<template>
  <div>
    <div v-for="todo in todos" :key="todo.id" data-test="todo">
      {{ todo.text }}
    </div>
  </div>
</template>
```

Com esta alteração, o teste está a ser aprovado. Parabéns! Escrevemos o nosso primeiro teste de componente.

## Adicionar uma nova tarefa {#Adding-a-new-todo}

A próxima funcionalidade que adicionaremos é a possibilidade de o utilizador criar uma nova tarefa. Para isto, precisamos dum formulário com um campo de entrada para o utilizador escrever algum texto. Quando o utilizador submete o formulário, esperamos que a nova tarefa seja desenhada. Daremos uma olhadela ao teste:

```js
import { mount } from '@vue/test-utils'
import TodoApp from './TodoApp.vue'

test('creates a todo', () => {
  const wrapper = mount(TodoApp)
  expect(wrapper.findAll('[data-test="todo"]')).toHaveLength(1)

  wrapper.get('[data-test="new-todo"]').setValue('New todo')
  wrapper.get('[data-test="form"]').trigger('submit')

  expect(wrapper.findAll('[data-test="todo"]')).toHaveLength(2)
})
```

Como de costume, começamos usando `mount` para desenhar o elemento. Também estamos asserindo que apenas 1 tarefa é desenhada — isto torna claro que estamos adicionando uma tarefa adicional, como sugere a linha final do teste.

Para atualizar o `<input>`, usamos `setValue` — isto permite-nos definir o valor do campo de entrada.

Depois de atualizar o `<input>`, usamos o método `trigger` para simular o utilizador submetendo o formulário. Finalmente, asserimos que o número de itens da lista de tarefas aumentou de 1 para 2.

Se executarmos este teste, é óbvio que falhará. Atualizaremos o `TodoApp.vue` para ter os elementos `<form>` e `<input>` e fazer o teste passar:

```vue
<template>
  <div>
    <div v-for="todo in todos" :key="todo.id" data-test="todo">
      {{ todo.text }}
    </div>

    <form data-test="form" @submit.prevent="createTodo">
      <input data-test="new-todo" v-model="newTodo" />
    </form>
  </div>
</template>

<script>
export default {
  name: 'TodoApp',

  data() {
    return {
      newTodo: '',
      todos: [
        {
          id: 1,
          text: 'Learn Vue.js 3',
          completed: false
        }
      ]
    }
  },

  methods: {
    createTodo() {
      this.todos.push({
        id: 2,
        text: this.newTodo,
        completed: false
      })
    }
  }
}
</script>
```

Nós estamos usando `v-model` para vincular ao `<input>` e `@submit` para ouvir o envio do formulário. Quando o formulário é submetido, `createTodo` é chamada e insere uma nova tarefa no vetor `todos`.

Embora isto pareça bom, a execução do teste exibe um erro:

```
expect(received).toHaveLength(expected)

    Expected length: 2
    Received length: 1
    Received array:  [{"element": <div data-test="todo">Learn Vue.js 3</div>}]
```

O número de tarefas não aumentou. O problema é que a Jest executa os testes de maneira síncrona, terminando o teste assim que a função final é chamada. A Vue, no entanto, atualiza o modelo de objeto do documento de maneira assíncrona. Precisamos marcar o teste como `async`, e chamar `await` em quaisquer métodos que possam fazer com que o modelo de objeto do documento mude. O `trigger` é um destes métodos, assim como `setValue` — podemos simplesmente acrescentar `await` e o testo deve funcionar como esperado:

```js
import { mount } from '@vue/test-utils'
import TodoApp from './TodoApp.vue'

test('creates a todo', async () => {
  const wrapper = mount(TodoApp)

  await wrapper.get('[data-test="new-todo"]').setValue('New todo')
  await wrapper.get('[data-test="form"]').trigger('submit')

  expect(wrapper.findAll('[data-test="todo"]')).toHaveLength(2)
})
```

Agora o teste passa finalmente!

## Completar uma tarefa {#Completing-a-todo}

Agora que podemos criar as tarefas, daremos ao utilizador a capacidade de marcar um item de tarefa como concluído ou não concluído com uma caixa de verificação. Como anteriormente, começaremos com o teste de falha:

```js
import { mount } from '@vue/test-utils'
import TodoApp from './TodoApp.vue'

test('completes a todo', async () => {
  const wrapper = mount(TodoApp)

  await wrapper.get('[data-test="todo-checkbox"]').setValue(true)

  expect(wrapper.get('[data-test="todo"]').classes()).toContain('completed')
})
```

Este teste é semelhante aos dois anteriores; procuramos um elemento e interagimos com este da mesma maneira (usamos `setValue` novamente, já que estamos interagindo com um `<input>`).

Finalmente, fazemos uma asserção. Nós aplicaremos uma classe `completed` às tarefas concluídas — podemos então usar isto para adicionar algum estilo para indicar visualmente o estado duma tarefa.

Podemos fazer o teste passar atualizando o `<template>` para incluir o `<input type="checkbox">` e um vínculo de classe sobre o elemento de tarefa:

```vue
<template>
  <div>
    <div
      v-for="todo in todos"
      :key="todo.id"
      data-test="todo"
      :class="[todo.completed ? 'completed' : '']"
    >
      {{ todo.text }}
      <input
        type="checkbox"
        v-model="todo.completed"
        data-test="todo-checkbox"
      />
    </div>

    <form data-test="form" @submit.prevent="createTodo">
      <input data-test="new-todo" v-model="newTodo" />
    </form>
  </div>
</template>
```

Parabéns! Escrevemos os nossos primeiros testes de componente.

## Organizar, Atuar, Asserir {#Arrange-Act-Assert}

Podemos ter notado algumas linhas entre o código em cada um dos testes. Veremos o segundo teste novamente, em detalhes:

```js
import { mount } from '@vue/test-utils'
import TodoApp from './TodoApp.vue'

test('creates a todo', async () => {
  const wrapper = mount(TodoApp)

  await wrapper.get('[data-test="new-todo"]').setValue('New todo')
  await wrapper.get('[data-test="form"]').trigger('submit')

  expect(wrapper.findAll('[data-test="todo"]')).toHaveLength(2)
})
```

O teste está dividido em três fases distintas, separadas por novas linhas. As três etapas representam as três fases dum teste: **organizar**, **atuar**, e **asserir**.

Na fase de _organização_, configuramos o cenário para o teste. Um exemplo mais complexo pode exigir a criação dum armazém de Vuex, ou o preenchimento duma base de dados.

Na fase de _atuação_, representamos o cenário, simulando como um utilizador interagiria com o componente ou a aplicação.

Na fase de _asserção_, fazemos asserções sobre como esperamos que seja o estado atual do componente.

Quase todos os testes seguirão estas três fases. Não precisamos separá-las com novas linhas como este guia faz, mas é bom ter estas três fases em mente quando escrevemos os nossos testes.

## Conclusão {#Conclusion}

- Usar `mount()` para desenhar um componente.
- Usar `get` e `findAll` para consultar o modelo de objeto do documento.
- `trigger()` e `setValue()` são auxiliares para simular a entrada do utilizador.
- A atualização o modelo de objeto do documento é uma operação assíncrona, precisamos usar `async` e `await`.
- O teste normalmente consiste em 3 fases; organizar, atuar, e asserir.

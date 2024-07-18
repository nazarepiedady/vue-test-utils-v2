# Interpretação Condicional {#Conditional-Rendering}

A Vue Test Utils tem uma gama de funcionalidades para interpretar e fazer asserções sobre o estado de um componente, visando verificar se este comporta-se corretamente. Este artigo explorará a maneira de interpretar componentes, bem como verificar se estão a interpretar o conteúdo corretamente.

Este artigo também está disponível como um [vídeo curto](https://www.youtube.com/watch?v=T3CHtGgEFTs&list=PLC2LZCNWKL9ahK1IoODqYxKu5aA9T5IOA&index=15).

## Encontrar Elementos {#Finding-Elements}

Uma das funcionalidades básicas da Vue é a capacidade de inserir e remover elementos dinamicamente com a diretiva `v-if`. Veremos como testar um componente que usa a diretiva `v-if`.

```js
const Nav = {
  template: `
    <nav>
      <a id="profile" href="/profile">My Profile</a>
      <a v-if="admin" id="admin" href="/admin">Admin</a>
    </nav>
  `,
  data() {
    return {
      admin: false
    }
  }
}
```

No componente `<Nav>`, é exibida uma hiperligação para o perfil do utilizador. Além disto, se o valor `admin` for `true`, revelamos uma hiperligação para a secção de administração. Há três cenários que devemos verificar se estão a comportar-se corretamente.

1. A hiperligação `/profile` deve ser exibida.
2. Quando o utilizador é um administrador, a hiperligação `/admin` deve ser exibida.
3. Quando o utilizador não é um administrador, a hiperligação `/admin` não deve ser exibida.

## Usar `get()` {#Using-get-}

O `wrapper` tem um método `get()` que procura por um elemento existente. Utiliza a sintaxe [`querySelector`](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector).

Podemos asserir o conteúdo da hiperligação do perfil usando `get()`:

```js
test('renders a profile link', () => {
  const wrapper = mount(Nav)

  // Eis que asserimos implicitamente que
  // o elemento #profile existe.
  const profileLink = wrapper.get('#profile')

  expect(profileLink.text()).toEqual('My Profile')
})
```

Se `get()` não retornar um elemento que corresponda ao seletor, levantará um erro e nosso teste falhará. A `get()` retorna um `DOMWrapper` se um elemento for encontrado. Um `DOMWrapper` é um embrulhador fino em torno do elemento do modelo do objeto do documento que implementa a [interface de programação de aplicação do embrulhador](/api/#wrapper-methods) — é por isto que conseguimos fazer `profileLink.text()` e acessar o texto.

Existe um outro tipo de embrulhador — um `VueWrapper` — retornado por [`getComponent`](/api/#getcomponent) que funciona da mesma maneira.

## Usar `find()` e `exists()` {#Using-find-and-exists-}

A `get()` trabalha com o pressuposto de que os elementos existem e lança um erro quando não existem. _Não_ se recomenda o seu uso para asserir a existência.

Para o fazer, usamos `find()` e `exists()`. O próximo teste assere que se `admin` é `false` (a qual é por padrão), a hiperligação de administração não está presente:

```js
test('does not render an admin link', () => {
  const wrapper = mount(Nav)

  // Usar `wrapper.get` lançaria e faria o teste falhar.
  expect(wrapper.find('#admin').exists()).toBe(false)
})
```

Precisamos notar que estamos chamando `exists()` sobre o valor retornado por `.find()`. A `find()`, assim como a `mount()`, também retorna um `wrapper`. A `mount()` tem alguns métodos adicionais, porque embrulha um componente de Vue, `find` retorna apenas um nó do modelo de objeto do documento normal, mas muitos dos métodos são partilhados entre ambos. Alguns outros métodos incluem `classes()`, que obtém as classes que um nó do modelo de objeto do documento possui, e `trigger()` para simular a interação do utilizador. Podemos encontrar uma lista dos métodos suportados [nesta hiperligação](../../api/#wrapper-methods).

## Usar `data` {#Using-data}

O teste final é para asserir que a hiperligação de administração é desenhada quando `admin` é `true`. É `false` por padrão, mas podemos sobrepor isto usando o segundo argumento para `mount()`, as [opções de montagem](../../api/#mount-options).

Para `data`, usamos a opção corretamente chamada de `data`:

```js
test('renders an admin link', () => {
  const wrapper = mount(Nav, {
    data() {
      return {
        admin: true
      }
    }
  })

  // Mais uma vez, ao usar `get` asserimos implicitamente que
  // o elemento existe.
  expect(wrapper.get('#admin').text()).toEqual('Admin')
})
```

Se tivermos outras propriedades em `data`, não precisamos nos preocupar — a Vue Test Utils combinará as duas. A `data` nas opções de montagem terão prioridade sobre quaisquer valores predefinidos.

Para saber quais outras opções de montagem existem, consultar [`Passing Data`](../essentials/passing-data) ou consultar [`mounting options`](../../api/#mount-options).

## Verificar a Visibilidade dos Elementos {#Checking-Elements-visibility}

Por vezes, apenas queremos esconder ou mostrar um elemento mantendo-o no modelo de objeto do documento. A Vue oferece a `v-show` para cenários como este. (Podemos consultar as diferenças entre `v-if` e `v-show` [nesta hiperligação](https://pt.vuejs.org/guide/conditional#v-if-vs-v-show)).

Este é o aspeto de um componente com `v-show`:

```js
const Nav = {
  template: `
    <nav>
      <a id="user" href="/profile">My Profile</a>
      <ul v-show="shouldShowDropdown" id="user-dropdown">
        <!-- dropdown content -->
      </ul>
    </nav>
  `,
  data() {
    return {
      shouldShowDropdown: false
    }
  }
}
```

Neste cenário, o elemento não é visível, mas é sempre desenhado. A `get()` ou `find()` sempre retornará um `Wrapper` — `find()` com `.exists()` sempre retornará `true` — porque o **elemento continua no modelo de objeto do documento**.

## Usar `isVisible()` {#Using-isVisible-}

A função `isVisible()` permite verificar a existência de elementos ocultos. Em particular `isVisible()` verificará se:

- um elemento ou os seus ancestrais têm estilo `display: none`, `visibility: hidden`, `opacity :0`
- um elemento ou os seus ancestrais estão localizados dentro de um marcador `<details>` colapsado
- um elemento ou os seus ancestrais têm o atributo `hidden`

Para qualquer um destes casos, `isVisible()` retorna `false`.

Os cenários de teste usando `v-show` parecer-se-ão com:

```js
test('does not show the user dropdown', () => {
  const wrapper = mount(Nav)

  expect(wrapper.get('#user-dropdown').isVisible()).toBe(false)
})
```

## Conclusão {#Conclusion}

- Usar `find()` com `exists()` para verificar se um elemento está no modelo de objeto do documento.
- Usar `get()` se esperamos que o elemento esteja no modelo de objeto do documento.
- A opção de montagem `data` pode ser usada para definir valores padrão num componente.
- Usar `get()` com `isVisible()` para verificar a visibilidade de um elemento que está no modelo de objeto do documento.

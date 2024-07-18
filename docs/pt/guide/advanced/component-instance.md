# Instância do Componente {#Component-Instance}

A [`mount`](/api/#mount) retorna um `VueWrapper` com vários métodos convenientes para testar componentes de Vue. Por vezes, podemos querer acessar à instância da Vue subjacente. Podemos acessar-lhe com a propriedade `vm`.

## Um Simples Exemplo {#A-Simple-Example}

Eis um componente simples que combina as propriedades e dados para desenhar uma saudação:

```ts
test('renders a greeting', () => {
  const Comp = {
    template: `<div>{{ msg1 }} {{ msg2 }}</div>`,
    props: ['msg1'],
    data() {
      return {
        msg2: 'world'
      }
    }
  }

  const wrapper = mount(Comp, {
    props: {
      msg1: 'hello'
    }
  })

  expect(wrapper.html()).toContain('hello world')
})
```

Daremos uma olhada no que está disponível na `vm` com `console.log(wrapper.vm)`:

```js
{
  msg1: [Getter/Setter],
  msg2: [Getter/Setter],
  hasOwnProperty: [Function]
}
```

Podemos ver ambas `msg1` e `msg2`! Coisas como as propriedades `methods` e `computed` também aparecerão, se estiverem definidas. Quando escrevemos um teste, embora é geralmente recomendado asserir contra o modelo de objeto do documento (usando algo como `wrapper.html()`), em algumas raras circunstâncias podemos precisar do acesso à instância subjacente da Vue.

## Uso com `getComponent` e `findComponent` {#Usage-with-getComponent-and-findComponent}

A `getComponent` e `findComponent` retorna um `VueWrapper` — muito parecido com o que é obtido pela `mount`. Isto significa que também podemos acessar a todas as mesmas propriedades, incluindo a `vm`, no resultado de `getComponent` ou `findComponent`.

Eis um exemplo simples:

```js
test('asserts correct props are passed', () => {
  const Foo = {
    props: ['msg'],
    template: `<div>{{ msg }}</div>`
  }

  const Comp = {
    components: { Foo },
    template: `<div><foo msg="hello world" /></div>`
  }

  const wrapper = mount(Comp)

  expect(wrapper.getComponent(Foo).vm.msg).toBe('hello world')
  expect(wrapper.getComponent(Foo).props()).toEqual({ msg: 'hello world' })
})
```

Uma maneira mais completa de testar isto seria asserindo contra o conteúdo desenhado. Ao fazer isto, garantimos que a propriedade correta é passada *e* desenhada.

:::warning Tipo `WrapperLike` quando se usa um seletor de CSS
Quando usamos `wrapper.findComponent('.foo')` por exemplo, então a VTU retornará o tipo `WrapperLike`. Isto ocorre porque os componentes funcionais precisariam de um `DOMWrapper` ou um `VueWrapper`. Podemos forçar o retorno de um `VueWrapper` fornecendo o tipo correto do componente:

```typescript
wrapper.findComponent('.foo') // retorna WrapperLike
wrapper.findComponent<typeof FooComponent>('.foo') // retorna VueWrapper
wrapper.findComponent<DefineComponent>('.foo') // retorna VueWrapper
```
:::

## Conclusão {#Conclusion}

- Usar `vm` para acessar a instância interna da Vue
- A `getComponent` e `findComponent` retornam um embrulhador da Vue. Estas instâncias da Vue também estão disponíveis através da `vm`.

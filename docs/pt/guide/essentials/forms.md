# Manipulação de Formulário {#Form-Handling}

Os formulários na Vue podem ser tão simples quanto os formulários da HTML para complicadas árvores agrupadas de elementos de formulários de componentes de Vue personalizados. Iremos gradualmente percorrer as maneiras de interagir com os elementos do formulário, definindo valores e acionando eventos.

Os métodos que mais usaremos são `setValue()` e `trigger()`.

## Interagir com elementos do formulário {#Interacting-with-form-elements}

Vejamos um formulário muito básico:

```vue
<template>
  <div>
    <input type="email" v-model="email" />

    <button @click="submit">Submit</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      email: ''
    }
  },
  methods: {
    submit() {
      this.$emit('submit', this.email)
    }
  }
}
</script>
```

### Definir valores ao elemento {#Setting-element-values}

A maneira mais comum de vincular uma entrada ao `data` na Vue é usando `v-model`. Como provavelmente já sabemos, esta lida com os eventos que cada elemento de formulário emite e das propriedades que aceita, facilitando o trabalho com elementos de formulário.

Para alterar o valor duma entrada na VTU, podemos usar o método `setValue()`. Este aceita um parâmetro, geralmente uma `String` ou um `Boolean`, e retorna uma `Promise`, a qual é resolvida após a Vue ter atualizado o modelo de objeto do documento.

```js
test('sets the value', async () => {
  const wrapper = mount(Component)
  const input = wrapper.find('input')

  await input.setValue('my@mail.com')

  expect(input.element.value).toBe('my@mail.com')
})
```

Como podemos ver, `setValue` define a propriedade `value` no elemento de entrada para o que passamos a este.

Usamos `await` para nos certificarmos que a Vue completou a atualização e alteração foi refletida no modelo de objeto do documento, antes de fazermos quaisquer asserções.

### Acionar eventos {#Triggering-events}

O acionamento de eventos é a segunda ação mais importante quando se trabalha com formulários e elementos de ação. Daremos uma olhada no nosso `button`, do exemplo anterior.

```html
<button @click="submit">Submit</button>
```

Para acionar um evento de clique, podemos usar o método `trigger`.

```js
test('trigger', async () => {
  const wrapper = mount(Component)

  // acionar o elemento
  await wrapper.find('button').trigger('click')

  // asserir que alguma ação foi realizada,
  // como um evento emitido. 
  expect(wrapper.emitted()).toHaveProperty('submit')
})
```

> Se ainda não vimos `emitted()` antes, não precisamos nos preocupar. Esta é usada para asserir os eventos emitidos de um componente. Podemos saber mais em [Manipulação de Evento](./event-handling).

Acionamos o ouvinte de evento `click`, para o componente executar o método `submit`. Tal como fizemos com `setValue`, usamos `await` para garantir que a ação é refletida pela Vue.

Podemos então asserir que alguma ação aconteceu. Neste caso, que emitimos o evento correto.

Combinaremos estes dois para testar se o nosso formulário simples emite as entradas do utilizador.

```js
test('emits the input to its parent', async () => {
  const wrapper = mount(Component)

  // definir o valor
  await wrapper.find('input').setValue('my@mail.com')

  // acionar o elemento
  await wrapper.find('button').trigger('click')

  // asserir que o evento `submit` é emitido,
  expect(wrapper.emitted('submit')[0][0]).toBe('my@mail.com')
})
```

## Fluxos de trabalho avançados {#Advanced-workflows}

Agora que sabemos o básico, mergulharemos exemplos mais complexos.

### Trabalhar com vários elementos do formulário {#Working-with-various-form-elements}

Vimos que `setValue` funciona com elementos de entrada, mas é muito mais versátil, já que pode definir o valor em vários tipos de elementos de entrada.

Veremos um formulário mais complicado, que tem mais tipos de entradas.

```vue
<template>
  <form @submit.prevent="submit">
    <input type="email" v-model="form.email" />

    <textarea v-model="form.description" />

    <select v-model="form.city">
      <option value="new-york">New York</option>
      <option value="moscow">Moscow</option>
    </select>

    <input type="checkbox" v-model="form.subscribe" />

    <input type="radio" value="weekly" v-model="form.interval" />
    <input type="radio" value="monthly" v-model="form.interval" />

    <button type="submit">Submit</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      form: {
        email: '',
        description: '',
        city: '',
        subscribe: false,
        interval: ''
      }
    }
  },
  methods: {
    async submit() {
      this.$emit('submit', this.form)
    }
  }
}
</script>
```

O nosso componente de Vue estendido é um pouco mais longo, tem mais alguns tipos de entrada e agora tem o manipulador `submit` movido para um elemento `<form/>`.

Da mesma maneira que definimos o valor no `input`, podemos defini-lo em todas as outras entradas no formulário.

```js
import { mount } from '@vue/test-utils'
import FormComponent from './FormComponent.vue'

test('submits a form', async () => {
  const wrapper = mount(FormComponent)

  await wrapper.find('input[type=email]').setValue('name@mail.com')
  await wrapper.find('textarea').setValue('Lorem ipsum dolor sit amet')
  await wrapper.find('select').setValue('moscow')
  await wrapper.find('input[type=checkbox]').setValue()
  await wrapper.find('input[type=radio][value=monthly]').setValue()
})
```

Como podemos ver, `setValue` é um método muito versátil. Esta pode trabalhar com todos os tipos de elementos de formulários.

Usamos `await` em todo o lado, para nos certificarmos de que cada alteração foi aplicada antes de acionarmos a seguinte. Isto é recomendado para garantir que fizemos asserções quando o modelo de objeto do documento é atualizado.

:::tip DICA
Se não passarmos um parâmetro para `setValue` para as entradas `OPTION`, `CHECKBOX`, ou `RADIO`, estas serão definidas como `checked`.
:::

Definimos valores no nosso formulários, agora é altura de submeter o formulário e fazer algumas afirmações.

### Acionar ouvintes de eventos complexos {#Triggering-complex-event-listeners}

Os ouvintes de eventos nem sempre são simples eventos de `click`. A Vue permite-lhe ouvir todos os tipos de eventos do modelo de objeto do documento, adicionar modificadores especiais como `.prevent` e muito mais. Veremos como as podemos testar.

No nosso formulário acima, movemos o evento do elemento `button` para o elemento `form`. Esta é uma boa prática a ser seguida, uma vez que nos permite submeter um formulário ao pressionar a tecla `enter`, a qual é uma abordagem mais nativa.

Para acionar o manipulador `submit`, usamos o método `trigger` novamente.

```js {14,16-22}
test('submits the form', async () => {
  const wrapper = mount(FormComponent)

  const email = 'name@mail.com'
  const description = 'Lorem ipsum dolor sit amet'
  const city = 'moscow'

  await wrapper.find('input[type=email]').setValue(email)
  await wrapper.find('textarea').setValue(description)
  await wrapper.find('select').setValue(city)
  await wrapper.find('input[type=checkbox]').setValue()
  await wrapper.find('input[type=radio][value=monthly]').setValue()

  await wrapper.find('form').trigger('submit.prevent')

  expect(wrapper.emitted('submit')[0][0]).toStrictEqual({
    email,
    description,
    city,
    subscribe: true,
    interval: 'monthly'
  })
})
```

Para testar o modificar de evento, copiamos e colamos diretamente a nossa sequência de caracteres de evento `submit.prevent` em `trigger`. A `trigger` pode ler o evento passado e todos os seus modificadores, e aplicar seletivamente o que for necessário.

:::tip DICA
Os modificadores de eventos nativos como `.prevent` e `.stop` são específicos da Vue e como tal não precisamos testá-los, os recursos internos da Vue já fazem isso.
:::

Em seguida, fazemos uma asserção simples, se o formulário emitiu o evento e carga correta.

#### Submissão de formulário nativa {#Native-form-submission}

O acionamento de um evento `submit` num elemento `<form>` imita o comportamento do navegador durante o evento de um formulário. Se quiséssemos acionar a submissão do formulário mais naturalmente, poderíamos acionar um evento de `click` no botão de submissão. Uma vez que elementos de formulário não conectados ao `document` não podem ser submetidos, segundo a [especificação da HTML](https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#form-submission-algorithm), precisamos usar [`attachTo`](../../api/#attachto) para conectar o elemento do embrulhador.

#### Vários modificadores sobre o mesmo evento {#Multiple-modifiers-on-the-same-event}

Suponhamos que temos um formulário muito detalhado e complexo, com uma manipulação especial da interação. Como podemos testar isso?

```html
<input @keydown.meta.c.exact.prevent="captureCopy" v-model="input" />
```

Suponhamos que temos uma entrada que manipula quando o utilizador clica `cmd` + `c`, e queremos intercetá-lo e impedi-lo de copiar. Para testar isso é tão fácil quanto copiar e colar o evento do componente para o método `trigger()`.

```js
test('handles complex events', async () => {
  const wrapper = mount(Component)

  await wrapper.find(input).trigger('keydown.meta.c.exact.prevent')

  // executar as nossas asserções
})
```

A Vue Test Utils lê o evento e aplica as propriedades corretas ao objeto do evento. Neste caso, corresponderá a algo como isto:

```js
{
  // ... outras propriedades
  "key": "c",
  "metaKey": true
}
```

#### Adicionar dado adicional a um evento {#Adding-extra-data-to-an-event}

Digamos que o nosso código precisa de algo dentro do objeto `event`. Podemos testar estes cenários passando dado adicional como segundo parâmetro.

```vue
<template>
  <form>
    <input type="text" v-model="value" @blur="handleBlur" />
    <button>Submit</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      value: ''
    }
  },
  methods: {
    handleBlur(event) {
      if (event.relatedTarget.tagName === 'BUTTON') {
        this.$emit('focus-lost')
      }
    }
  }
}
</script>
```

```js
import Form from './Form.vue'

test('emits an event only if you lose focus to a button', () => {
  const wrapper = mount(Form)

  const componentToGetFocus = wrapper.find('button')

  wrapper.find('input').trigger('blur', {
    relatedTarget: componentToGetFocus.element
  })

  expect(wrapper.emitted('focus-lost')).toBeTruthy()
})
```

Neste exemplo, assumimos que o nosso código verifica dentro do objeto `event`, se o `relatedTarget` é um botão ou não. Podemos simplesmente passar uma referência a este elemento, imitando o que aconteceria se o utilizador clicasse num `button` após escrever algo no `input`.

## Interagir com as entradas do componente de Vue {#Interacting-with-Vue-Component-inputs}

As entradas não são apenas elementos simples. Usamos frequentemente os componentes de Vue que se comportam como entradas. Estes podem adicionar marcação, estilos e muitas funcionalidades num formato fácil de usar.

Testar formulários que usam tais entradas pode ser assustador no início, mas com algumas regras simples, torna-se rapidamente um passeio no parque.

A seguir temos um componente que embrulha um elemento `label` e um `input`:

```vue
<template>
  <label>
    {{ label }}
    <input
      type="text"
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
    />
  </label>
</template>

<script>
export default {
  name: 'CustomInput',

  props: ['modelValue', 'label']
}
</script>
```

Este componente de Vue também emite de volta tudo o que digitamos. Para o usar, fazemos:

```html
<custom-input v-model="input" label="Text Input" class="text-input" />
```

Como acima, a maioria destas entradas com a Vue tem um `button` ou `input` real neles. Podemos facilmente entrar este elemento e atuar sobre este:

```js
test('fills in the form', async () => {
  const wrapper = mount(CustomInput)

  await wrapper.find('.text-input input').setValue('text')

  // continuar com as asserções ou
  // ações como submeter o formulário,
  // asserir o modelo de objeto do documento…
})
```

### Testar componentes de entrada complexos {#Testing-complex-Input-components}

O que acontece se o nosso componente de entrada não for assim tão simples? Podemos estar a usar uma biblioteca de interface do utilizador, como a Vuetify. Se dependermos de procurar dentro da marcação para encontrar o elemento correto, os nossos testes podem falhar se a biblioteca externa decidir alterar os seus elementos internos.

Nestes casos, podemos definir o valor diretamente, usando a instância do componente e `setValue`.

Suponhamos que temos um formulário que usa a área de texto Vuetify:

```vue
<template>
  <form @submit.prevent="handleSubmit">
    <v-textarea v-model="description" ref="description" />
    <button type="submit">Send</button>
  </form>
</template>

<script>
export default {
  name: 'CustomTextarea',
  data() {
    return {
      description: ''
    }
  },
  methods: {
    handleSubmit() {
      this.$emit('submitted', this.description)
    }
  }
}
</script>
```

Podemos usar `findComponent` para encontrar a instância do componente e, em seguida, definir o seu valor.

```js
test('emits textarea value on submit', async () => {
  const wrapper = mount(CustomTextarea)
  const description = 'Some very long text...'

  await wrapper.findComponent({ ref: 'description' }).setValue(description)

  wrapper.find('form').trigger('submit')

  expect(wrapper.emitted('submitted')[0][0]).toEqual(description)
})
```

## Conclusão {#Conclusion}

- Usar `setValue` para definir o valor em entradas do modelo de objeto do documento e componentes de Vue.
- Usar `trigger` para acionar eventos do modelo de objeto do documento, com e sem modificações.
- Adicionar dado de evento adicional ao `trigger` usando o segundo parâmetro.
- Asserir que o modelo de objeto do documento foi alterado, e que os eventos corretos foram emitidos. Tentar não asserir dados sobre a instância do componente.

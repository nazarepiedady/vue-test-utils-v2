# Questões Frequentes {#FAQ}

[[toc]]

## Simulando Datas e Temporizadores com a Vitest {#Mocking-Dates-and-Timers-with-Vitest}

O agendador da Vue depende da hora do sistema. Temos de certificar-nos de montar os componentes *depois* de chamar `vi.setSystemTime`, já que a Vue depende dos seus efeitos colaterais. A montagem dos componentes antes de chamar `vi.setSystemTime` pode causar quebras na reatividade.

Consultar a [vuejs/test-utils#2074](https://github.com/vuejs/test-utils/issues/2074).

## Aviso da Vue: Falha na definição da propriedade {#Vue-warn-Failed-setting-prop}

```
[Vue warn]: Failed setting prop "prefix" on <component-stub>: value foo is invalid.
TypeError: Cannot set property prefix of #<Element> which has only a getter
```

Este aviso é mostrado no caso de estivermos usando `shallowMount` ou `stubs` com um nome de propriedade que é partilhado com o [`Element`](https://developer.mozilla.org/en-US/docs/Web/API/Element).

Nomes de propriedades comuns que são partilhados com o `Element`:

* `attributes`
* `children`
* `prefix`

Consultar: [`Element`](https://developer.mozilla.org/en-US/docs/Web/API/Element)

**Soluções possíveis**

1. Usar `mount` ao invés de `shallowMount` para desenhar sem os esboços
2. Ignorar o aviso através da simulação de `console.warn`
3. Renomear a propriedade para não entrar em conflito com as propriedades do `Element`.

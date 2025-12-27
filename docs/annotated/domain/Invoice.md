# `src/domain/Invoice.ts`

## Papel na arquitetura

Value Object do domínio, representando uma fatura gerada.

## Código original

```ts
export default class Invoice {

	constructor (readonly date: Date, readonly amount: number) {
	}
}
```

## Anotações técnicas

- **Imutabilidade**: propriedades são `readonly`, evitando mutações após criação.
- **Value Object**: não há identidade própria — o que importa são os valores `date` e `amount`.

## Boas práticas

- **Simplicidade**: mantém o modelo de domínio minimalista.
- **Confiabilidade**: objetos imutáveis reduzem efeitos colaterais.

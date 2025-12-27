# `src/domain/Payment.ts`

## Papel na arquitetura

Entidade que representa um pagamento feito por um cliente em um contrato.

## Código original

```ts
export default class Payment {

	constructor (
		readonly idPayment: string,
		readonly date: Date,
		readonly amount: number
	) {
	}
}
```

## Anotações técnicas

- **Identidade própria**: `idPayment` diferencia pagamentos mesmo com mesma data/valor.
- **Imutabilidade**: `readonly` garante consistência do registro de pagamento.

## Boas práticas

- **Modelagem explícita**: objetos de domínio evitam trabalhar com estruturas genéricas (ex.: objetos livres).

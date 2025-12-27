# `src/domain/InvoiceGenerationFactory.ts`

## Papel na arquitetura

Fábrica responsável por escolher qual **estratégia de geração** será usada.

## Código original

```ts
import AccrualBasisStrategy from "./AccrualBasisStrategy";
import CashBasisStrategy from "./CashBasisStrategy";

export default class InvoiceGenerationFactory {

	static create (type: string) {
		if (type === "cash") {
			return new CashBasisStrategy();
		}
		if (type === "accrual") {
			return new AccrualBasisStrategy();
		}
		throw new Error("Invalid type");
	}
}
```

## Anotações técnicas

- **Factory Method**: concentra a decisão de qual estratégia instanciar.
- **Validação de entrada**: lança erro quando o tipo não é suportado.
- **Extensibilidade**: para adicionar um novo tipo, basta expandir a fábrica (OCP).

## Boas práticas

- **Coesão**: a regra de escolha fica isolada em um único ponto.

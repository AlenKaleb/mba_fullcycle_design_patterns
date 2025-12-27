# `src/domain/CashBasisStrategy.ts`

## Papel na arquitetura

Estratégia de geração **por regime de caixa**: gera faturas somente para pagamentos efetivamente realizados no mês/ano.

## Código original

```ts
import Contract from "./Contract";
import Invoice from "./Invoice";
import InvoiceGenerationStrategy from "./InvoiceGenerationStrategy";

export default class CashBasisStrategy implements InvoiceGenerationStrategy {

	generate(contract: Contract, month: number, year: number): Invoice[] {
		const invoices: Invoice[] = [];
		for (const payment of contract.getPayments()) {
			if (payment.date.getMonth() + 1 !== month || payment.date.getFullYear() !== year) continue;
			invoices.push(new Invoice(payment.date, payment.amount));
		}
		return invoices;
	}

}
```

## Anotações técnicas

- **Iteração de pagamentos**: utiliza `contract.getPayments()` para isolar o estado interno.
- **Filtro por competência**: valida mês/ano com `getMonth() + 1` (JavaScript indexa meses a partir de 0).
- **Geração de `Invoice`**: cada pagamento vira uma fatura com a mesma data e valor.

## Boas práticas

- **Algoritmo encapsulado**: regras de caixa ficam isoladas da entidade `Contract`.
- **Legibilidade**: separação clara entre filtragem e criação das faturas.

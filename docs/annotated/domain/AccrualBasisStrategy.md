# `src/domain/AccrualBasisStrategy.ts`

## Papel na arquitetura

Estratégia de geração **por regime de competência**: parcela o contrato e gera faturas por mês, independentemente do pagamento.

## Código original

```ts
import Contract from "./Contract";
import Invoice from "./Invoice";
import InvoiceGenerationStrategy from "./InvoiceGenerationStrategy";
import moment from "moment";

export default class AccrualBasisStrategy implements InvoiceGenerationStrategy {

	generate(contract: Contract, month: number, year: number): Invoice[] {
		const invoices: Invoice[] = [];
		let period = 0;
		while (period <= contract.periods) {
			const date = moment(contract.date).add(period++, "months").toDate();
			if (date.getMonth() + 1 !== month || date.getFullYear() !== year) continue;
			const amount = contract.amount/contract.periods;
			invoices.push(new Invoice(date, amount));
		}
		return invoices;
	}

}
```

## Anotações técnicas

- **Uso de `moment`**: facilita cálculo de datas ao adicionar meses ao início do contrato.
- **Loop por períodos**: `while (period <= contract.periods)` percorre cada mês do contrato.
- **Cálculo de parcela**: `contract.amount/contract.periods` distribui o valor total igualmente.
- **Filtro por competência**: mantém apenas faturas do mês/ano solicitados.

## Boas práticas

- **Separação de algoritmo**: competência fica em classe própria, permitindo evolução sem impacto no resto do domínio.
- **Reuso de contrato**: evita duplicar regras de data dentro do `Contract`.

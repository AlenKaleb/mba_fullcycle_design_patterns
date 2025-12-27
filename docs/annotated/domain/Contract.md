# `src/domain/Contract.ts`

## Papel na arquitetura

Entidade central do domínio. Representa um contrato com **parcelas** e **pagamentos**, oferecendo comportamento rico (cálculo de saldo e geração de faturas).

## Código original

```ts
import Invoice from "./Invoice";
import InvoiceGenerationFactory from "./InvoiceGenerationFactory";
import InvoiceGenerationStrategy from "./InvoiceGenerationStrategy";
import Payment from "./Payment";
import moment from "moment";

export default class Contract {
	private payments: Payment[];

	constructor (
		readonly idContract: string,
		readonly description: string,
		readonly amount: number,
		readonly periods: number,
		readonly date: Date
	) {
		this.payments = [];
	}

	addPayment (payment: Payment) {
		this.payments.push(payment);
	}

	getPayments () {
		return this.payments;
	}

	getBalance () {
		let balance = this.amount;
		for (const payment of this.payments) {
			balance -= payment.amount;
		}
		return balance;
	}

	generateInvoices (month: number, year: number, type: string) {
		const invoiceGenerationStrategy = InvoiceGenerationFactory.create(type);
		return invoiceGenerationStrategy.generate(this, month, year);
	}
}
```

## Anotações técnicas

- **Entidade rica**: `Contract` guarda o estado (`payments`) e expõe comportamentos (`getBalance`, `generateInvoices`).
- **Encapsulamento**: `payments` é privado e só pode ser modificado por `addPayment`.
- **Cálculo de saldo**: `getBalance` deduz pagamentos do valor total — lógica de domínio, não de infraestrutura.
- **Strategy + Factory**: `generateInvoices` delega a geração de faturas a uma estratégia escolhida pela `InvoiceGenerationFactory`.
- **Import `moment` não usado**: indica dependência não utilizada, algo que pode ser removido para manter o domínio enxuto.

## Boas práticas

- **Coesão**: regras de negócio ficam na entidade de domínio.
- **Extensibilidade**: novas estratégias de geração de faturas podem ser adicionadas sem alterar esta classe (OCP).

# `src/application/usecase/GenerateInvoices.ts`

## Papel na arquitetura

Caso de uso responsável por **gerar faturas** a partir de contratos persistidos e formatar a saída por meio de um *Presenter*. Também publica eventos para efeitos colaterais.

## Código original

```ts
import ContractDatabaseRepository from "../../infra/repository/ContractDatabaseRepository";
import ContractRepository from "../repository/ContractRepository";
import Presenter from "../presenter/Presenter";
import JsonPresenter from "../../infra/presenter/JsonPresenter";
import Usecase from "./Usecase";
import Mediator from "../../infra/mediator/Mediator";

export default class GenerateInvoices implements Usecase {

	constructor (
		readonly contractRepository: ContractRepository, 
		readonly presenter: Presenter = new JsonPresenter(),
		readonly mediator: Mediator = new Mediator()
	) {
	}

	async execute (input: Input): Promise<any> {
		const output: Output[] = [];
		const contracts = await this.contractRepository.list();
		for (const contract of contracts) {
			const invoices = contract.generateInvoices(input.month, input.year, input.type);
			for (const invoice of invoices) {
				output.push({ date: invoice.date, amount: invoice.amount });
			}
		}
		await this.mediator.publish("InvoicesGenerated", output);
		return this.presenter.present(output);
	}
}

type Input = {
	month: number,
	year: number,
	type: string,
	format?: string
}

export type Output = {
	date: Date,
	amount: number
}
```

## Anotações técnicas

- **Dependências por interface**: `ContractRepository` e `Presenter` são abstrações, permitindo troca de implementação.
- **Dependências concretas com default**: `JsonPresenter` e `Mediator` são usados por padrão, mas podem ser substituídos (injeção de dependência).
- **Carregamento de contratos**: `list()` abstrai origem de dados (banco, API, etc.).
- **Delegação ao domínio**: `contract.generateInvoices(...)` mantém as regras no domínio.
- **Normalização de saída**: converte `Invoice` para `Output` simples (DTO de aplicação).
- **Evento de domínio**: `publish("InvoicesGenerated", output)` aciona efeitos secundários (ex.: e-mail) sem acoplamento.
- **Presenter**: `present(...)` formata a saída (JSON, CSV, etc.).

## Boas práticas

- **Camada de aplicação como orquestradora**: não contém regra de negócio detalhada.
- **DIP + IoC**: uso de interfaces e injeção de dependências.
- **Desacoplamento de side effects**: mediador separa geração de faturas de notificações.

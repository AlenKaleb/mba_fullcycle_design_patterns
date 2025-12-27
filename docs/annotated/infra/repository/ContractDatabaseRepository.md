# `src/infra/repository/ContractDatabaseRepository.ts`

## Papel na arquitetura

Implementação concreta do `ContractRepository` utilizando acesso a banco de dados relacional.

## Código original

```ts
import AccrualBasisStrategy from "../../domain/AccrualBasisStrategy";
import Contract from "../../domain/Contract";
import ContractRepository from "../../application/repository/ContractRepository";
import DatabaseConnection from "../database/DatabaseConnection";
import Payment from "../../domain/Payment";

export default class ContractDatabaseRepository implements ContractRepository {

	constructor (readonly connection: DatabaseConnection) {
	}

	async list(): Promise<Contract[]> {
		const contracts: Contract[] = [];
		const contractsData = await this.connection.query("select * from branas.contract", []);
		for (const contractData of contractsData) {
			const contract = new Contract(contractData.id_contract, contractData.description, parseFloat(contractData.amount), contractData.periods, contractData.date);
			const paymentsData = await this.connection.query("select * from branas.payment where id_contract = $1", [contract.idContract]);
			for (const paymentData of paymentsData) {
				contract.addPayment(new Payment(paymentData.id_payment, paymentData.date, parseFloat(paymentData.amount)));
			}
			contracts.push(contract);
		}
		return contracts;
	}

}
```

## Anotações técnicas

- **Injeção de `DatabaseConnection`**: permite trocar o driver de banco sem alterar o repositório.
- **Consulta de contratos**: `select * from branas.contract` recupera contratos base.
- **Hidratação de entidades**: cria `Contract` e associa `Payment`s, mantendo domínio independente de SQL.
- **Conversão de tipos**: `parseFloat` normaliza tipos numéricos vindos do banco.
- **Consulta por contrato**: busca pagamentos com `id_contract` para manter agregação consistente.
- **Import não utilizado**: `AccrualBasisStrategy` é importado mas não usado — pode ser removido para reduzir ruído.

## Boas práticas

- **Repository Pattern**: encapsula a persistência.
- **Modelagem de agregados**: `Contract` carrega seus pagamentos.

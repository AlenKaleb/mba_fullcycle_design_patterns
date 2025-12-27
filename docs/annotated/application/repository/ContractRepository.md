# `src/application/repository/ContractRepository.ts`

## Papel na arquitetura

Interface do **Repository Pattern** para acesso a contratos.

## Código original

```ts
import Contract from "../../domain/Contract";

export default interface ContractRepository {
	list (): Promise<Contract[]>;
}
```

## Anotações técnicas

- **Contrato único**: expõe `list()` para obter contratos, sem revelar detalhes de persistência.
- **Retorno tipado**: `Contract[]` garante objetos de domínio na camada de aplicação.

## Boas práticas

- **Isolamento de infraestrutura**: aplicação não depende de SQL ou ORM.
